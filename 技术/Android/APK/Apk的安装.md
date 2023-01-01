## APK的安装

#### APK的组成

- META-INF：存放签名的信息，应用安装验证签名的时候会验证该文件里面的信息，
- res：这里面的资源是不经过编译原样打包进来的。
- AndroidManifest.xml：程序全局配置文件。该文件是每个应用程序都必须定义和包含的文件。
  - 它里面描述了程序的名字、版本、权限、引用的库文件等信息。
- classes.dex：Dalvik字节码文件，Android会将所有的class文件全部放到这一个文件里。
- resources.arsc：编译后的二进制资源文件，保存资源文件的索引，由aapt生成。
- lib：如果存在的话，存放的是ndk编出来的so库。

#### APK的打包过程

- 编译器将源代码转换成DEX文件，将资源文件转换成已编译资源。
- APK打包器将DEX文件和已编译资源合并成单个APK。在安装APK在Android设备之前，必须将APK进行签名。
- 最终生成的APK文件之前还会使用zipalign工具来优化文件。

#### APK的安装过程

##### 弹出安装界面（PackageInstallerActivity）

###### buildUi

- 当一个未安装apk被点击后，会弹出安装界面，点击确认后，会进入PackageInstallerActivity # buildUi中的确认事件
- 点击APK后，弹出的安装界面，主要是由buildUi构成，上面有取消和安装的两个按钮，点击安装后会调用startInstall进行安装
- buildUi的源码

###### startInstall

- 主要是组装了一个Intent，并且跳转到InstallInstalling这个Activity
- startInstall的源码

###### InstallInstalling # onCreate

- 从PackageInstallerActivity跳转到InstallInstallingActivity后，会执行onCreate方法

- onCreate源码

  ```java
  @Override
      protected void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
  
          ApplicationInfo appInfo = getIntent()
                  .getParcelableExtra(PackageUtil.INTENT_ATTR_APPLICATION_INFO);
          mPackageURI = getIntent().getData();
  
          if ("package".equals(mPackageURI.getScheme())) {
              try {
                  getPackageManager().installExistingPackage(appInfo.packageName);
                  launchSuccess();
              } catch (PackageManager.NameNotFoundException e) {
                  launchFailure(PackageManager.INSTALL_FAILED_INTERNAL_ERROR, null);
              }
          } else {
              final File sourceFile = new File(mPackageURI.getPath());
              PackageUtil.AppSnippet as = PackageUtil.getAppSnippet(this, appInfo, sourceFile);
  
              mAlert.setIcon(as.icon);
              mAlert.setTitle(as.label);
              mAlert.setView(R.layout.install_content_view);
              mAlert.setButton(DialogInterface.BUTTON_NEGATIVE, getString(R.string.cancel),
                      (ignored, ignored2) -> {
                          if (mInstallingTask != null) {
                              mInstallingTask.cancel(true);
                          }
  
                          if (mSessionId > 0) {
                              getPackageManager().getPackageInstaller().abandonSession(mSessionId);
                              mSessionId = 0;
                          }
  
                          setResult(RESULT_CANCELED);
                          finish();
                      }, null);
              setupAlert();
              requireViewById(R.id.installing).setVisibility(View.VISIBLE);
  
              /** 
               * 第一步： 
               * 当savedInstanceState 不为null的时候
               * 从savedInstanceState中拿到此前保存的 mSessionId和mInstallId
               * mSessionId：是安装包的会话ID
               * mInstallId：是等待安装的事件ID
               */
              if (savedInstanceState != null) {
                  mSessionId = savedInstanceState.getInt(SESSION_ID);
                  mInstallId = savedInstanceState.getInt(INSTALL_ID);
  
                  // Reregister for result; might instantly call back if result was delivered while
                  // activity was destroyed
                  try {
                      /**
                       *  第二步：
                       *  以mInstallId为标示 向InstallEventRecriver中组册一个观察者
                       *  其中launchFinishBasedOnResult 是接受安装的回调事件
                       *  无论成功还是失败，都会回调关闭当前的页面
                       */
  
                      InstallEventReceiver.addObserver(this, mInstallId,
                              this::launchFinishBasedOnResult);
                  } catch (EventResultPersister.OutOfIdsException e) {
                      // Does not happen
                  }
              } else {
                  /**
                   *   第三步：
                   *   创建SessionParams，用它来来代表安装会话的参数，组装params
                   */
  
                  PackageInstaller.SessionParams params = new PackageInstaller.SessionParams(
                          PackageInstaller.SessionParams.MODE_FULL_INSTALL);
                  params.setInstallAsInstantApp(false);
                  params.setReferrerUri(getIntent().getParcelableExtra(Intent.EXTRA_REFERRER));
                  params.setOriginatingUri(getIntent()
                          .getParcelableExtra(Intent.EXTRA_ORIGINATING_URI));
                  params.setOriginatingUid(getIntent().getIntExtra(Intent.EXTRA_ORIGINATING_UID,
                          UID_UNKNOWN));
                  params.setInstallerPackageName(getIntent().getStringExtra(
                          Intent.EXTRA_INSTALLER_PACKAGE_NAME));
                  params.setInstallReason(PackageManager.INSTALL_REASON_USER);
  								/**
  								* 第四步：
  								* 根据 mPackageURI对APK安装包进行解析，并将解析的参数赋值给SessionParams
  								*
  								*/
                  File file = new File(mPackageURI.getPath());
                  try {
                      PackageParser.PackageLite pkg = PackageParser.parsePackageLite(file, 0);
                      params.setAppPackageName(pkg.packageName);
                      params.setInstallLocation(pkg.installLocation);
                      params.setSize(
                              PackageHelper.calculateInstalledSize(pkg, false, params.abiOverride));
                  } catch (PackageParser.PackageParserException e) {
                      Log.e(LOG_TAG, "Cannot parse package " + file + ". Assuming defaults.");
                      Log.e(LOG_TAG,
                              "Cannot calculate installed size " + file + ". Try only apk size.");
                      params.setSize(file.length());
                  } catch (IOException e) {
                      Log.e(LOG_TAG,
                              "Cannot calculate installed size " + file + ". Try only apk size.");
                      params.setSize(file.length());
                  }
  
                  try {
                    /*
                    * 向InstalleventReceiver 注册一个观察者返回一个mInstallId。
                    *
                    */
                      mInstallId = InstallEventReceiver
                              .addObserver(this, EventResultPersister.GENERATE_NEW_ID,
                                      this::launchFinishBasedOnResult);
                  } catch (EventResultPersister.OutOfIdsException e) {
                      launchFailure(PackageManager.INSTALL_FAILED_INTERNAL_ERROR, null);
                  }
  
                  try {
                      mSessionId = getPackageManager().getPackageInstaller().createSession(params);
                  } catch (IOException e) {
                      launchFailure(PackageManager.INSTALL_FAILED_INTERNAL_ERROR, null);
                  }
              }
  
              mCancelButton = mAlert.getButton(DialogInterface.BUTTON_NEGATIVE);
  
              mSessionCallback = new InstallSessionCallback();
          }
      }
  ```

- 从onCreate方法执行完毕后，下一个重要流程就是InstallInstalling中的onResume()方法

###### InstallInstalling # onResume()

- 将APK以IO形式写入到PackageInsstaller # Session 中
- 调用了 PakcageInstaller # commit方法，进行apk的安装
- onResume()源码

##### PackageInstaller # commit

- commit源码

  ```java
     public void commit(@NonNull IntentSender statusReceiver) {
              try {
                  /**
                   * mSession ---> IPackageInstallerSession
                   * IPackageInstallerSession AIDL文件，服务端是PackageInstallerSession 进行了一次跨进程通信
                   * PackageInstallerSession # commit()
                   */
                  mSession.commit(statusReceiver, false);
              } catch (RemoteException e) {
                  throw e.rethrowFromSystemServer();
              }
          }
  ```

###### PackageInstallerSession # commit()

- commit() 内部通过Handler发送了一个消息，
- 处理消息的地方，调用了commitNonStagedLocked()
- commitNonStagedLocked() 该方法的的内部调用了PackageManagerService # installStage()

