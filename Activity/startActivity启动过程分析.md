#### 简介
- startActivity() ---> 用来启动一个Activity，那么我们的分析入口就从startActivity()方法入手
###### Activity启动流程的梳理（基于Android 10.0）

- 对于Activity的启动流程一共涉及到两次跨进程通信，分别是以下两种
  - 从App端进程到服务端进程 (App ---> AMTS)
  - 从服务端进程到App进程（AMTS ---> APP）
  - 以上两次跨进程通信都是通过Binder完成的
- 从APP端到AMTS
  - 发送一个启动Activity的请求，通过IActivityTaskManger调用到在服务端实现类中的方法，完成了一次跨进程通信
  - 发送的请求中，携带着数据
- 在AMTS中（system_server 服务进程）做如下几件事情
  - 处理栈顶的Activity ---> 通过跨进程通信从服务端到App端（IApplicationThread），调用到了栈顶Activity的onPause生命周期方法
  - 然后开启一个新的Activity ，新Activity所处的进程和线程存在的话就不用创建了，不存在的话还需要创建对应的进程从而也会创建新的Application
- 从AMTS到App端
  - 此次跨进程通信 主要使用了 ActivityThread中的IApplicationThread ，通过在调用了App进程中实现类中的方法，
  - 收到服务端的请求后，通过Handler（消息机制）从Binder线程切换到主线程，从而去调用各种生命周期方法
- 当然了上述只是一个大概的流程，其中在服务端做的事情是特别多的，比如携带持有数据的Intent，等等。

###### Activity启动流程中涉及到的类

- App端
  - Activity
  - Instrumentation
- ATMS
  - ActivityTaskManagerService：IActivityTaskManager的服务端实现类，用于和App端进行通信
  - ActivityStartController
  - ActivityStarter
  - RootActivityContainer
  - ActivityStack
  - ClientLifecycleManager
  - ClientTransaction
- App端
  - ActivityThread  ： UI线程的管理类
  - ApplicationThread：IApplicationThread的服务端实现类，内部实现的方法用于和ATMS所在的进程端进行通信
  - ClientTransactionHandler
  - TransactionExecutor
  - Instrumentation
  - Activity

- 其他
  - ActivityInfo：用于存储AndroidManifest中设置的Activity信息和receiver节点的信息
  - ActivityRecord：是存在与系统进程中，用于存储Activity的实例信息
  - ActivityClientRecord：是存在App进程中，用于存储Activity的实例信息
  - 在整个跨进程传递过程中有一个Token类型的字段，这个Token类型是继承IApplicationThread.Stub，在App进程中，是作为key用来存储ActivityClientRecord的实例，在系统进程中Token类中会持有ActivityRecord的一个弱引用。
  - 在每次进行跨进程通信的时候，都会传递这个Token类型的字段，通过这个Token类型字段就会将 ActivityRecord，ActivityClientRecord和Activity进行关联的。

#### 代码流程分析
###### startActivity
```java
###  MainActivity.kt
startActivity(intent)
### Activity.java
 // 这里的Bundle是一个null
 @Override
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }

// 这里无论options是否为空，都会调用到startActivityForResult()方法
 @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }

//  最终都会回调到这个startActivityForResult()方法中，看代码
  public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        //如果是第一次启动，mParent为null
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                    // Instrumentation # execStartActivity()
                    // 在这里调用 Instrumentation的 execStartActivity()方法
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                mStartedActivity = true;
            }
            cancelInputsAndStartExitTransition(options);
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {  
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }
```
###### Instrumentation
- 是应用程序检测代码的基类
```java
### execStartActivity
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        // IApplicationThread ActivityThread # ApplicationThread
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;

        .......省略代码.......

        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //获取到ATMS的实例，调用了它的startActivity方法
            // getService()返回的是一个IActivityTaskManager类型的对象
            // IActivityTaskManger是一个AIDL文件，它的服务端是ActivityTaskManagerService
            // 最终调用了ActivityTaskManagerService中的startActivity()方法，是作为一次跨进程通信的，
            // 这里就从App进程 传递到了ATMS所在的系统进程
            int result = ActivityTaskManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```
- 通过IActivityTaskManager，调用了startActivity()进行了一次跨进程通信，从app端跨到系统进程中,以上分析过程都是在App进程中进行的。从ActivityTaskManagerService的startActivity()开始，是在系统进程中执行的。

###### ActivityTaskManagerService
```java
### startActivity
// 上述startActivity()方法中，最终进行一系列调用，调用到了如下方法
 int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
            boolean validateIncomingUser) {
        enforceNotIsolatedCaller("startActivityAsUser");

        userId = getActivityStartController().checkTargetUser(userId, validateIncomingUser,
                Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");

        // TODO: Switch to user app stacks here.
        /**
         * getActivityStartController() --> ActivityStartController
         * 调用了obtainStarter()方法 获取到ActivityStarter对象
         *  **注意** ActivityStarter 的 setMayWait()方法
         * 最终执行了execute()方法
         * 我个人理解：ActivityStarter类是用来解析 Intent和flag携带的数据
         */
        return getActivityStartController().obtainStarter(intent, "startActivityAsUser")
                .setCaller(caller)
                .setCallingPackage(callingPackage)
                .setResolvedType(resolvedType)
                .setResultTo(resultTo)
                .setResultWho(resultWho)
                .setRequestCode(requestCode)
                .setStartFlags(startFlags)
                .setProfilerInfo(profilerInfo)
                .setActivityOptions(bOptions)
                .setMayWait(userId)
                .execute();

    }
```
- 上述方法最终调用到了ActivityStarter中的execute方法
###### ActivityStarter
```java
### execute()
 int execute() {
        try {
            // TODO(b/64750076): Look into passing request directly to these methods to allow
            // for transactional diffs and preprocessing.
            /**
             * 在ActivityTaskManagerService中有调用过ActivityStarter的setMayWait()
             * ActivityStarter setMayWait(int userId) {
             *         mRequest.mayWait = true;
             *         mRequest.userId = userId;
             *
             *         return this;
             *     }
             *
             *     接着调用 startActivityMayWait()
             */
            if (mRequest.mayWait) {
                return startActivityMayWait(mRequest.caller, mRequest.callingUid,
                        mRequest.callingPackage, mRequest.realCallingPid, mRequest.realCallingUid,
                        mRequest.intent, mRequest.resolvedType,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.startFlags,
                        mRequest.profilerInfo, mRequest.waitResult, mRequest.globalConfig,
                        mRequest.activityOptions, mRequest.ignoreTargetSecurity, mRequest.userId,
                        mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup,
                        mRequest.originatingPendingIntent, mRequest.allowBackgroundActivityStart);
            } else {
                return startActivity(mRequest.caller, mRequest.intent, mRequest.ephemeralIntent,
                        mRequest.resolvedType, mRequest.activityInfo, mRequest.resolveInfo,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.callingPid,
                        mRequest.callingUid, mRequest.callingPackage, mRequest.realCallingPid,
                        mRequest.realCallingUid, mRequest.startFlags, mRequest.activityOptions,
                        mRequest.ignoreTargetSecurity, mRequest.componentSpecified,
                        mRequest.outActivity, mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup,
                        mRequest.originatingPendingIntent, mRequest.allowBackgroundActivityStart);
            }
        } finally {
            onExecutionComplete();
        }
    }

### startActivityMayWait()

   private int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, int requestRealCallingPid, int requestRealCallingUid,
            Intent intent, String resolvedType, IVoiceInteractionSession voiceSession,
            IVoiceInteractor voiceInteractor, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, WaitResult outResult,
            Configuration globalConfig, SafeActivityOptions options, boolean ignoreTargetSecurity,
            int userId, TaskRecord inTask, String reason,
            boolean allowPendingRemoteAnimationRegistryLookup,
            PendingIntentRecord originatingPendingIntent, boolean allowBackgroundActivityStart) {
        // Refuse possible leaked file descriptors
         
        .......省略代码........

        /**
         *  通过ActivityStackSupervisor # resolveIntent()方法 获取到ResolveInfo
         */

        ResolveInfo rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId,
                0 /* matchFlags */,
                        computeResolveFilterUid(
                                callingUid, realCallingUid, mRequest.filterCallingUid));
        if (rInfo == null) {
            UserInfo userInfo = mSupervisor.getUserInfo(userId);
            if (userInfo != null && userInfo.isManagedProfile()) {
                // Special case for managed profiles, if attempting to launch non-cryto aware
                // app in a locked managed profile from an unlocked parent allow it to resolve
                // as user will be sent via confirm credentials to unlock the profile.
                UserManager userManager = UserManager.get(mService.mContext);
                boolean profileLockedAndParentUnlockingOrUnlocked = false;
                long token = Binder.clearCallingIdentity();
                try {
                    UserInfo parent = userManager.getProfileParent(userId);
                    profileLockedAndParentUnlockingOrUnlocked = (parent != null)
                            && userManager.isUserUnlockingOrUnlocked(parent.id)
                            && !userManager.isUserUnlockingOrUnlocked(userId);
                } finally {
                    Binder.restoreCallingIdentity(token);
                }
                if (profileLockedAndParentUnlockingOrUnlocked) {
                    rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId,
                            PackageManager.MATCH_DIRECT_BOOT_AWARE
                                    | PackageManager.MATCH_DIRECT_BOOT_UNAWARE,
                            computeResolveFilterUid(
                                    callingUid, realCallingUid, mRequest.filterCallingUid));
                }
            }
        }
        // Collect information about the target of the Intent.
        /**
         * 通过ActivityStackSupervisor # resolveActivity获取到ActivityInfo
         * 这是获取到要启动的Activity的信息
         */
        ActivityInfo aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, profilerInfo);

             .........省略代码............

            final ActivityRecord[] outRecord = new ActivityRecord[1];
            /**
             *  调用了startActivity()
              */
            int res = startActivity(caller, intent, ephemeralIntent, resolvedType, aInfo, rInfo,
                    voiceSession, voiceInteractor, resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags, options,
                    ignoreTargetSecurity, componentSpecified, outRecord, inTask, reason,
                    allowPendingRemoteAnimationRegistryLookup, originatingPendingIntent,
                    allowBackgroundActivityStart);

     ...... 省略代码 .......
            return res;
        }
    }

//标示
    private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            SafeActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, TaskRecord inTask, String reason,
            boolean allowPendingRemoteAnimationRegistryLookup,
            PendingIntentRecord originatingPendingIntent, boolean allowBackgroundActivityStart) {

        if (TextUtils.isEmpty(reason)) {
            throw new IllegalArgumentException("Need to specify a reason.");
        }
        mLastStartReason = reason;
        mLastStartActivityTimeMs = System.currentTimeMillis();
        mLastStartActivityRecord[0] = null;

        /**
         * 继续调用了同名的重载方法 startActivity()
         */
        mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
                callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
                inTask, allowPendingRemoteAnimationRegistryLookup, originatingPendingIntent,
                allowBackgroundActivityStart);

        if (outActivity != null) {
            // mLastStartActivityRecord[0] is set in the call to startActivity above.
            outActivity[0] = mLastStartActivityRecord[0];
        }

        return getExternalResult(mLastStartActivityResult);
    }

 private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
                IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
                int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
                ActivityRecord[] outActivity, boolean restrictedBgActivity) {
        int result = START_CANCELED;
        final ActivityStack startedActivityStack;
        try {
            mService.mWindowManager.deferSurfaceLayout();
            /**
             * 重点1
             */
            result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                    startFlags, doResume, options, inTask, outActivity, restrictedBgActivity);
        } finally {
            final ActivityStack currentStack = r.getActivityStack();
            startedActivityStack = currentStack != null ? currentStack : mTargetStack;

            if (ActivityManager.isStartResultSuccessful(result)) {
                if (startedActivityStack != null) {
                    // If there is no state change (e.g. a resumed activity is reparented to
                    // top of another display) to trigger a visibility/configuration checking,
                    // we have to update the configuration for changing to different display.
                    final ActivityRecord currentTop =
                            startedActivityStack.topRunningActivityLocked();
                    if (currentTop != null && currentTop.shouldUpdateConfigForDisplayChanged()) {
                        mRootActivityContainer.ensureVisibilityAndConfig(
                                currentTop, currentTop.getDisplayId(),
                                true /* markFrozenIfConfigChanged */, false /* deferResume */);
                    }
                }
            } else {
                // If we are not able to proceed, disassociate the activity from the task.
                // Leaving an activity in an incomplete state can lead to issues, such as
                // performing operations without a window container.
                final ActivityStack stack = mStartActivity.getActivityStack();
                if (stack != null) {
                    stack.finishActivityLocked(mStartActivity, RESULT_CANCELED,
                            null /* intentResultData */, "startActivity", true /* oomAdj */);
                }

                // Stack should also be detached from display and be removed if it's empty.
                if (startedActivityStack != null && startedActivityStack.isAttached()
                        && startedActivityStack.numActivities() == 0
                        && !startedActivityStack.isActivityTypeHome()) {
                    startedActivityStack.remove();
                }
            }
            mService.mWindowManager.continueSurfaceLayout();
        }
        postStartActivityProcessing(r, result, startedActivityStack);

        return result;
    }

// 在这个startActivity方法中，
// startActivityUnchecked()重点

```
- 看下startActivityUnchecked()方法
```java
  // 在该方法内部会调用到如下代码
 if (mDoResume) {
      /**
       * 重点
        */
        // 这里 mRootActivityContainer ---> RootActivityContainer (Activity容器的根节点)
      mRootActivityContainer.resumeFocusedStacksTopActivities();
}
```
- RootActivityContainer # resumeFocusedStacksTopActivities()
```java
// 在该方法中 会调用到ActivityStack的resumeTopActivityUncheckedLocked()的方法
 final ActivityStack focusedStack = display.getFocusedStack();
                if (focusedStack != null) {
                    /**
                     * 重点核心
                     * ActivityStack的resumeTopActivityUncheckedLocked()
                     */
                    focusedStack.resumeTopActivityUncheckedLocked(target, targetOptions);
                }
```
###### ActivityStack
```java
### resumeTopActivityUncheckedLocked()
// 确保处于栈顶的Activity是处于resume的生命周期，这个方法的目的在于通知处于栈顶的Activity要执行onPause方法
 boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        if (mInResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
         
            mInResumeTopActivity = true;
            /**
             * 重点 核心
             */
            result = resumeTopActivityInnerLocked(prev, options);
            final ActivityRecord next = topRunningActivityLocked(true);
            if (next == null || !next.canTurnScreenOn()) {
                checkReadyForSleep();
            }
        } finally {
            mInResumeTopActivity = false;
        }

        return result;
    }
### resumeTopActivityInnerLocked(prev, options);

// 在该方法中会执行如下核心代码，调用到了startPausingLocked ---> 通知栈顶的Activity要执行onPause方法的流程
if (mResumedActivity != null) {
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: Pausing " + mResumedActivity);
            /**
             * 重点 核心
             */
            pausing |= startPausingLocked(userLeaving, false, next, false);
        }
// 在这个方法还有一个核心重点方法，该方法是执行新Activity的创建流程
mStackSupervisor.startSpecificActivityLocked(next, true, true); 

### startPausingLocked()
// 在此方法中，会执行如下核心的代码
if (prev.attachedToProcess()) {
            if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Enqueueing pending pause: " + prev);
            try {
                EventLogTags.writeAmPauseActivity(prev.mUserId, System.identityHashCode(prev),
                        prev.shortComponentName, "userLeaving=" + userLeaving);

                /**
                 * 核心 重点
                 *
                 * mService是ActivityTaskManagerService
                 * getLifecycleManager() ---> ClientLifecycleManager
                 * =======> ClientLifecycleManger # scheduleTransaction()
                 */
                mService.getLifecycleManager().scheduleTransaction(prev.app.getThread(),
                        prev.appToken, PauseActivityItem.obtain(prev.finishing, userLeaving,
                                prev.configChangeFlags, pauseImmediately));
            } catch (Exception e) {
                // Ignore exception, if process died other code will cleanup.
                Slog.w(TAG, "Exception thrown during pause", e);
                mPausingActivity = null;
                mLastPausedActivity = null;
                mLastNoHistoryActivity = null;
            }
        } 
```

###### ClientLifecycleManager

```java
### scheduleTransaction()
  
   void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
        /**
         * IAplicationThread AIDL文件，服务端是ActivityThread中的ApplicationThread
         */
        final IApplicationThread client = transaction.getClient();
        // transaction 是 ClientTransaction类型的，会去执行ClientTransaction#schedule()方法
        /**
         * ClientTransaction # schedule()
         * public void schedule() throws RemoteException {
         *          //这里面mClient是IApplicationThread类型的
         *          //所以综上分析，就调用到了ActivityThread#ApplicationThraad中的scheduleTransaction(this)
         *         mClient.scheduleTransaction(this);
         *     }
         */
        transaction.schedule();
  			//当获取到的client不是一个人Binder，那么将会移除远程请求。
        if (!(client instanceof Binder)) {
            // If client is not an instance of Binder - it's a remote call and at this point it is
            // safe to recycle the object. All objects used for local calls will be recycled after
            // the transaction is executed on client in ActivityThread.
            transaction.recycle();
        }
    }
```

- 到这里就进行了一次跨进程通行，从服务端进程切换到APP进程中，通过IApplicationThread

###### ActivityThrerad # ApplicationThread

```java
### scheduleTransaction()
 /**
         * 重点 核心
         * @param transaction
         * @throws RemoteException
         */
        @Override
        public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
            //核心 重点 调用了父类的 scheduleTransaction()
            // 由于scheduleTransaction() 方法 ，ActivityThread并没有实现，调用的是父类中的
            ActivityThread.this.scheduleTransaction(transaction);
        }
// ActivityThread extends ClientTransactionHandler

// 所以上述ActivityThread.this.scheduleTransaction(transaction); 方法执行就调用到了ClientTransactionHandler中的该方法

### ClientTransactionHandler

    void scheduleTransaction(ClientTransaction transaction) {
        transaction.preExecute(this);
        // sendMessage 是一个抽象方法 在ActivityThread中实现了
        sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
    }

### ActivityThread 中的 sendMessage
// 该方法中，通过调用了 mH的sendMessage()发送了一个消息
 private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        if (DEBUG_MESSAGES) {
            Slog.v(TAG,
                    "SCHEDULE " + what + " " + mH.codeToString(what) + ": " + arg1 + " / " + obj);
        }
        Message msg = Message.obtain();
        msg.what = what;
        msg.obj = obj;
        msg.arg1 = arg1;
        msg.arg2 = arg2;
        if (async) {
            msg.setAsynchronous(true);
        }
        /**
         * mH ----> H extends Handler
         * 由handler发送了一个消息
         */
        mH.sendMessage(msg);
    }
// 看下 ActivityThread 中的 H（Handler实现）的handleMessage() what == EXECUTE_TRANSACTION

 /**
  * 分析 触发执行栈顶activity的onPause方法
  */
    case EXECUTE_TRANSACTION:
    final ClientTransaction transaction = (ClientTransaction) msg.obj;
    //调用了TransactionExecutor # execute()
    mTransactionExecutor.execute(transaction);
    if (isSystem()) {
      transaction.recycle();
    }
    // TODO(lifecycler): Recycle locally scheduled transactions.
    break;

```

###### TransactionExecutor

```java
### execute()
    /**
     * 重点 核心 ActivityThread 中的 H（Handler）接受到消息后，调用到了 TransactionExecutor中的execute()
     */
    public void execute(ClientTransaction transaction) {
        if (DEBUG_RESOLVER) Slog.d(TAG, tId(transaction) + "Start resolving transaction");
				
  			//拿到Token字段，这个字段可以理解为 存储Activity实例对应的key，
        final IBinder token = transaction.getActivityToken();
        if (token != null) {
            final Map<IBinder, ClientTransactionItem> activitiesToBeDestroyed =
                    mTransactionHandler.getActivitiesToBeDestroyed();
            final ClientTransactionItem destroyItem = activitiesToBeDestroyed.get(token);
            if (destroyItem != null) {
                if (transaction.getLifecycleStateRequest() == destroyItem) {
                    // It is going to execute the transaction that will destroy activity with the
                    // token, so the corresponding to-be-destroyed record can be removed.
                    activitiesToBeDestroyed.remove(token);
                }
                if (mTransactionHandler.getActivityClient(token) == null) {
                    // The activity has not been created but has been requested to destroy, so all
                    // transactions for the token are just like being cancelled.
                    Slog.w(TAG, tId(transaction) + "Skip pre-destroyed transaction:\n"
                            + transactionToString(transaction, mTransactionHandler));
                    return;
                }
            }
        }

        if (DEBUG_RESOLVER) Slog.d(TAG, transactionToString(transaction, mTransactionHandler));

     
        executeCallbacks(transaction);
				
        /**
         * 核心 重点
         */
        executeLifecycleState(transaction);
        mPendingActions.clear();
        if (DEBUG_RESOLVER) Slog.d(TAG, tId(transaction) + "End resolving transaction");
    }

// executeLifecycleState(transaction);
/** Transition to the final state if requested by the transaction. */
    private void executeLifecycleState(ClientTransaction transaction) {
        final ActivityLifecycleItem lifecycleItem = transaction.getLifecycleStateRequest();
        if (lifecycleItem == null) {
            // No lifecycle request, return early.
            return;
        }

        final IBinder token = transaction.getActivityToken();
        final ActivityClientRecord r = mTransactionHandler.getActivityClient(token);
        if (DEBUG_RESOLVER) {
            Slog.d(TAG, tId(transaction) + "Resolving lifecycle state: "
                    + lifecycleItem + " for activity: "
                    + getShortActivityName(token, mTransactionHandler));
        }

        if (r == null) {
            // Ignore requests for non-existent client records for now.
            return;
        }

       /**
        * 重点分析
        */
        cycleToPath(r, lifecycleItem.getTargetState(), true /* excludeLastState */, transaction);

        // Execute the final transition with proper parameters.
        lifecycleItem.execute(mTransactionHandler, token, mPendingActions);
        lifecycleItem.postExecute(mTransactionHandler, token, mPendingActions);
    }

private void cycleToPath(ActivityClientRecord r, int finish, boolean excludeLastState,
            ClientTransaction transaction) {
        final int start = r.getLifecycleState();
        if (DEBUG_RESOLVER) {
            Slog.d(TAG, tId(transaction) + "Cycle activity: "
                    + getShortActivityName(r.token, mTransactionHandler)
                    + " from: " + getStateName(start) + " to: " + getStateName(finish)
                    + " excludeLastState: " + excludeLastState);
        }
        final IntArray path = mHelper.getLifecyclePath(start, finish, excludeLastState);
  			//重点分析
        performLifecycleSequence(r, path, transaction);
    }

// 该方法中 根据不同的state值，回调到 ActivityThread中的不同handle方法（处理生命周期的方法）
private void performLifecycleSequence(ActivityClientRecord r, IntArray path,
            ClientTransaction transaction) {
        final int size = path.size();
        for (int i = 0, state; i < size; i++) {
            state = path.get(i);
            if (DEBUG_RESOLVER) {
                Slog.d(TAG, tId(transaction) + "Transitioning activity: "
                        + getShortActivityName(r.token, mTransactionHandler)
                        + " to state: " + getStateName(state));
            }
            switch (state) {
                case ON_CREATE:
                    mTransactionHandler.handleLaunchActivity(r, mPendingActions,
                            null /* customIntent */);
                    break;
                case ON_START:
                    mTransactionHandler.handleStartActivity(r, mPendingActions);
                    break;
                case ON_RESUME:
                    mTransactionHandler.handleResumeActivity(r.token, false /* finalStateRequest */,
                            r.isForward, "LIFECYCLER_RESUME_ACTIVITY");
                    break;
                case ON_PAUSE:
                    mTransactionHandler.handlePauseActivity(r.token, false /* finished */,
                            false /* userLeaving */, 0 /* configChanges */, mPendingActions,
                            "LIFECYCLER_PAUSE_ACTIVITY");
                    break;
                case ON_STOP:
                    mTransactionHandler.handleStopActivity(r.token, false /* show */,
                            0 /* configChanges */, mPendingActions, false /* finalStateRequest */,
                            "LIFECYCLER_STOP_ACTIVITY");
                    break;
                case ON_DESTROY:
                    mTransactionHandler.handleDestroyActivity(r.token, false /* finishing */,
                            0 /* configChanges */, false /* getNonConfigInstance */,
                            "performLifecycleSequence. cycling to:" + path.get(size - 1));
                    break;
                case ON_RESTART:
                    mTransactionHandler.performRestartActivity(r.token, false /* start */);
                    break;
                default:
                    throw new IllegalArgumentException("Unexpected lifecycle state: " + state);
            }
        }
    }
}

// 咱这个过程是分析 onPause状态的 此时调用了ActivityThread的handlePauseActivity()方法
```

###### ActivityThread # handlePauseActivity()

```java
@Override
public void handlePauseActivity(IBinder token, boolean finished, boolean userLeaving,
                                int configChanges, PendingTransactionActions pendingActions, String reason) {
  // 根据Token拿到ActivityClientRecord实例
  // ActivityClientRecord 在App进程用来保存Activity的实例的，
  ActivityClientRecord r = mActivities.get(token);
  if (r != null) {
    if (userLeaving) {
      performUserLeavingActivity(r);
    }

    r.activity.mConfigChangeFlags |= configChanges;
     //执行performPauseActivity()
    performPauseActivity(r, finished, reason, pendingActions);

    // Make sure any pending writes are now committed.
    if (r.isPreHoneycomb()) {
      QueuedWork.waitToFinish();
    }
    mSomeActivitiesChanged = true;
  }
}

// perfromPauseActivity()
// 在performPauseActivity()方法，执行了perfromPauseActivityIfNeeded()方法，
private void performPauseActivityIfNeeded(ActivityClientRecord r, String reason) {
  //验证状态
        if (r.paused) {
            return;
        }

        // Always reporting top resumed position loss when pausing an activity. If necessary, it
        // will be restored in performResumeActivity().
        reportTopResumedActivityChanged(r, false /* onTop */, "pausing");

        try {
            r.activity.mCalled = false;
          	//执行了 callActivityOnPause()
          	// r --> ActivityClientRecord 保存了Activity的实例
          	// r.activity 保存的Activity的实例
          	// mInstrumentation ---> Instrumentation 
          	
            mInstrumentation.callActivityOnPause(r.activity);
            if (!r.activity.mCalled) {
                throw new SuperNotCalledException("Activity " + safeToComponentShortString(r.intent)
                        + " did not call through to super.onPause()");
            }
        } catch (SuperNotCalledException e) {
            throw e;
        } catch (Exception e) {
            if (!mInstrumentation.onException(r.activity, e)) {
                throw new RuntimeException("Unable to pause activity "
                        + safeToComponentShortString(r.intent) + ": " + e.toString(), e);
            }
        }
        r.setState(ON_PAUSE);
    }
```

###### Instrumentation # callActivityOnPause()

```java
//这里调用了activity的perfromPasue()方法
public void callActivityOnPause(Activity activity) {
        activity.performPause();
    }
    
```

###### Activity # perfromPause()

```java
final void performPause() {
        dispatchActivityPrePaused();
        mDoReportFullyDrawn = false;
        mFragments.dispatchPause();
        mCalled = false;
        /**
         * 核心 重点 执行到onPause()
         */
        onPause();
        writeEventLog(LOG_AM_ON_PAUSE_CALLED, "performPause");
        mResumed = false;
        if (!mCalled && getApplicationInfo().targetSdkVersion
                >= android.os.Build.VERSION_CODES.GINGERBREAD) {
            throw new SuperNotCalledException(
                    "Activity " + mComponent.toShortString() +
                    " did not call through to super.onPause()");
        }
        dispatchActivityPostPaused();
    }
```

- 可以看到这里执行到了onPause方法，这样就会执行到具体Activity中的onPause()方法了

- 以上就是栈顶Activity执行pause的流程，在ActivityStack中resumeTopActivityInnerLocked()方法中，还有调用

  ```java
  // mStackSupervisor ----> ActivityStackSupervisor
  mStackSupervisor.startSpecificActivityLocked(next, true, true);
  ```

###### ActivityStackSupervisor

```java
### startSpecificActivityLocked
  
void startSpecificActivityLocked(ActivityRecord r, boolean andResume, boolean checkConfig) {
        // Is this activity's application already running?
        final WindowProcessController wpc =
                mService.getProcessController(r.processName, r.info.applicationInfo.uid);

        boolean knownToBeDead = false;
        // 当我们应用的进程和线程存在的话
        if (wpc != null && wpc.hasThread()) {
            try {
                /**
                 * 重点分析1 ---> 走这里
                 */
                realStartActivityLocked(r, wpc, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Exception when starting activity "
                        + r.intent.getComponent().flattenToShortString(), e);
            }

            // If a dead object exception was thrown -- fall through to
            // restart the application.
            knownToBeDead = true;
        }

        // Suppress transition until the new activity becomes ready, otherwise the keyguard can
        // appear for a short amount of time before the new process with the new activity had the
        // ability to set its showWhenLocked flags.
        if (getKeyguardController().isKeyguardLocked()) {
            r.notifyUnknownVisibilityLaunched();
        }

        try {
            if (Trace.isTagEnabled(TRACE_TAG_ACTIVITY_MANAGER)) {
                Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "dispatchingStartProcess:"
                        + r.processName);
            }
            // Post message to start process to avoid possible deadlock of calling into AMS with the
            // ATMS lock held.
            /** 重点分析2：
             * 当应用的进程和线程不选在的话，就发送一个消息去开启一个应用进程
             * 应用进程都是从zygote进程通过socket通信来fork一份出来的
             * 使用socket通信的目的在于安全
             * 这里的mService.mH 就是ActivityThread 中的h
             */
            final Message msg = PooledLambda.obtainMessage(
                    ActivityManagerInternal::startProcess, mService.mAmInternal, r.processName,
                    r.info.applicationInfo, knownToBeDead, "activity", r.intent.getComponent());
            mService.mH.sendMessage(msg);
        } finally {
            Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
        }
    }
```

- 当我们要启动的Activity所处的进程和线程都存在的话，看下重点分析1

```java
### realStartActivityLocked(r, wpc, andResume, checkConfig);

 boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc,
            boolean andResume, boolean checkConfig) throws RemoteException {
   
   							.....省略代码.....
                  
                // Create activity launch transaction.
                // 创建Activity启动的事务 
                // proc ---> WindowProcessController proc.getThread() ---> IApplicationThread AIDL 远程的服                                   务端实现是在APP进程中实现的，
                final ClientTransaction clientTransaction = ClientTransaction.obtain(
                        proc.getThread(), r.appToken);

                final DisplayContent dc = r.getDisplay().mDisplayContent;
                // ClientTransaction 添加了一个回调
                clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
                        System.identityHashCode(r), r.info,
                        // TODO: Have this take the merged configuration instead of separate global
                        // and override configs.
                        mergedConfiguration.getGlobalConfiguration(),
                        mergedConfiguration.getOverrideConfiguration(), r.compat,
                        r.launchedFromPackage, task.voiceInteractor, proc.getReportedProcState(),
                        r.icicle, r.persistentState, results, newIntents,
                        dc.isNextTransitionForward(), proc.createProfilerInfoIfNeeded(),
                                r.assistToken));

                // Set desired final state.
                final ActivityLifecycleItem lifecycleItem;
                if (andResume) {
                    lifecycleItem = ResumeActivityItem.obtain(dc.isNextTransitionForward());
                } else {
                    lifecycleItem = PauseActivityItem.obtain();
                }
                clientTransaction.setLifecycleStateRequest(lifecycleItem);

                // Schedule transaction.
                // getLifecycleManager --> ClientLifecycleManager 执行了scheduleTransaction()方法
   							// 在这里调用到了ClientLifecycleManager中的scheduleTransaction()
   							// 在scheduleTransaction()方法中又调用了 ClientTransaction中的schedule()方法
   							// 在该方法中 通过跨进程通信 从系统进程到APP进程 通过IApplicationThread
   							// 我们的IApplicationThread的远程服务端实现是在App进程中的ActivityThread中实现的
   							// 根据上文分析栈顶Activity执行onPause的流程，到这里就是很清楚了，在App进程中通过发送消息到ui线程中
   							// 发送了一个EXECUTE_TRANSACTION类型的消息
   							// 在handleMessage()中 调用了TransactionExecutor # execute()方法
   							// 然后经过一些列调用 最终调用到了 ActivityThread中的handleLaunchActivity()中
                mService.getLifecycleManager().scheduleTransaction(clientTransaction);
   
                 .....省略代码.....
                
       }
```

###### ActivityThread # handleLaunchActivity()

````java
public Activity handleLaunchActivity(ActivityClientRecord r,
                                         PendingTransactionActions pendingActions, Intent customIntent) {
     	......省略代码......
       
        /**
         * 初始化Activity的WindowManager，每一个Activity都会对应一个窗口
         */

        WindowManagerGlobal.initialize();

        // Hint the GraphicsEnvironment that an activity is launching on the process.
        GraphicsEnvironment.hintActivityLaunch();

        /**
         * 特别重点的方法
         * 创建Activity并且显示它
         * 在这个方法中会创建DecorView
         */
        final Activity a = performLaunchActivity(r, customIntent);

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            if (!r.activity.mFinished && pendingActions != null) {
                pendingActions.setOldState(r.state);
                pendingActions.setRestoreInstanceState(true);
                pendingActions.setCallOnPostCreate(true);
            }
        } else {
            /**
             * 如果因为某些原因发生了错误，就结束掉当前的Activity
             */
            // If there was an error, for any reason, tell the activity manager to stop us.
            try {
                ActivityTaskManager.getService()
                        .finishActivity(r.token, Activity.RESULT_CANCELED, null,
                                Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }

        return a;
    }

### performLaunchActivity()
  
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
  			// ActivityClientRecord 中保存着Activity的信息
        ActivityInfo aInfo = r.activityInfo;
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);
        }

        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                    mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }

        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {

            java.lang.ClassLoader cl = appContext.getClassLoader();
            /**
             * 此处通过反射创建指定的Activity，
             *
             *  public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
             *             @Nullable Intent intent)
             *             throws InstantiationException, IllegalAccessException, ClassNotFoundException {
             *         return (Activity) cl.loadClass(className).newInstance();
             *     }
             */
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                        "Unable to instantiate activity " + component
                                + ": " + e.toString(), e);
            }
        }

        try {
            // 创建进程中的Application对象 如果没有存在的话
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (localLOGV) Slog.v(TAG, "Performing launch of " + r);
            if (localLOGV) Slog.v(
                    TAG, r + ": app=" + app
                            + ", appName=" + app.getPackageName()
                            + ", pkg=" + r.packageInfo.getPackageName()
                            + ", comp=" + r.intent.getComponent().toShortString()
                            + ", dir=" + r.packageInfo.getAppDir());

            if (activity != null) {
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (r.overrideConfig != null) {
                    config.updateFrom(r.overrideConfig);
                }
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
                /**
                 * 此处调用了attach方法建立Activity与Context之间的联系，
                 * 并且创建了PhoneWindow对象
                 调用到了attach方法
                 */
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        //将NonConfigurationInstance对象赋给新的Activity对象
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);

                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                checkAndBlockForNetworkAccess();
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }

                activity.mCalled = false;
                if (r.isPersistable()) {
                    /**
                     * 通过Instrumentation#callActivityOnCreate 调用了新创建的Activity 的onCreate方法
                     */
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                if (!activity.mCalled) {
                    throw new SuperNotCalledException(
                            "Activity " + r.intent.getComponent().toShortString() +
                                    " did not call through to super.onCreate()");
                }
                r.activity = activity;
            }
            r.setState(ON_CREATE);

            // updatePendingActivityConfiguration() reads from mActivities to update
            // ActivityClientRecord which runs in a different thread. Protect modifications to
            // mActivities to avoid race.
            synchronized (mResourcesManager) {
                mActivities.put(r.token, r);
            }

        } catch (SuperNotCalledException e) {
            throw e;

        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                        "Unable to start activity " + component
                                + ": " + e.toString(), e);
            }
        }

        return activity;
    }
````

- 上述方法中 先调用了attach方法，建立了Activity与Context的关系，同时初始化好了PhoneWindow，然后再去执行了Activity的onCreate()方法

###### Instrumentation#callActivityOnCreate

```java
public void callActivityOnCreate(Activity activity, Bundle icicle,
            PersistableBundle persistentState) {
        prePerformCreate(activity);
        //调用了Activity的performCreate()方法
  			// 在Activity的performCreate()方法中 执行了onCreate()方法，至此创建一个新的Activity，同时执行了onCreate方法。
        activity.performCreate(icicle, persistentState);
        postPerformCreate(activity);
    }
```

