## Bitmap的内存模型

**Android的Bitmap内存管理也是随着版本的迭代在不断的演进**

###### API10之前

- API10之前的时候也就是Android2.3.3之前，Bitmap的像素数据是保存在Native内存中的，而Bitmap对象是放在堆内存中（Dalvik Heap中） 
- Native内存中的像素数据不会按照预期的方式进行同步回收，有可能导致Native内存升高。

######  API10 到 API26

- 也就是从Android3.0到Android8.0的时候，Android把Bitmap的像素数据也放到堆内存中了，这样像素数据也会随着Bitmap的对象的回收而一起回收。

###### API26之后

- 也就是从Android8.0之后，Android把像素数据再次放到Native内存中，但是这回做了改进，native层的Bitmap像素数据可以做好和Java层的对象一起快速释放。

## Bitmap的像素格式(也就是一个像素占用的内存大小)

###### ARGB_8888(Android默认使用的像素格式)

- 颜色信由有Alpha、Red、Green、Blue四部分组成，每个部分占了8位，共32位，占了4个字节

###### ARGB_4444

- 颜色信息由Alpha、Red、Green、Blue四部分组成，每个部分占了4位，共16位，占了2个字节

###### RGB_565

- 颜色信息由Red、Green、Blue三部分组成，R占了5位、G占了6位、B占了5位，共16位，占了2个字节

###### ALPHA_8

- 颜色信息只有透明度，共8位 占了1个字节。

## Bitmap占有的内存

#### getByteCount()

- 该方法是在API12加入的，代表着Bitmap的像素数据需要最少的内存。
- 在API19开始getAlloctaionByteCount()方法替代了getByteCount()。

#### getAllocationByteCount()

- API19之后加入的，代表着内存中为Bitmap分配的内存大小。

#### Bitmap的内存回收

- 在Android3.0之前，需要手动调用Bitmap.recycler()进行Bitmap的回收
- 在Android3.0之后，不需要手动回收Bitmap了。

#### Bitmap的内存复用

###### 怎么复用

- 在Android3.0之后引入了新的字段 BitmapFactory.Options.inBitmap和inMutab 

- 如果字段inMutable设置为true，那么解码的方法会尝试服用一个存在的Bitmap。

- Bitmap的内存复用意味着减少了一个内存的回收以及申请，这样性能会更好。

  ```java
  @RequiresApi(api = Build.VERSION_CODES.KITKAT)
      private void reuseBitmap() {
          BitmapFactory.Options options = new BitmapFactory.Options();
         //关键点1
          options.inMutable = true;
          options.inDensity = 320;
          options.inTargetDensity = 320;
          Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.scal_density, options);
          Log.d(TAG, "reuseBitmap: " + bitmap);
          Log.d(TAG, "reuseBitmap: width = " + bitmap.getWidth() + ",height = " + bitmap.getHeight());
          Log.d(TAG, "reuseBitmap: byte_count = "+bitmap.getByteCount()+",allocation_byte_count ="+bitmap.getAllocationByteCount());
          // 关键点2
          options.inBitmap = bitmap;
          options.inDensity = 320;
          options.inTargetDensity = 160;
          Bitmap bitmap2 = BitmapFactory.decodeResource(getResources(), R.drawable.scal_density, options);
          Log.d(TAG, "reuseBitmap: " + bitmap2);
          Log.d(TAG, "reuseBitmap: width = " + bitmap2.getWidth() + ",height = " + bitmap2.getHeight());
          Log.d(TAG, "reuseBitmap: byte_count = "+bitmap2.getByteCount()+",allocation_byte_count ="+bitmap2.getAllocationByteCount());
      }
  
  //运行结果
  
  2020-03-04 20:15:23.955 29831-29831/? D/MainActivity: reuseBitmap: android.graphics.Bitmap@57148f
  2020-03-04 20:15:23.955 29831-29831/? D/MainActivity: reuseBitmap: width = 720,height = 360
  2020-03-04 20:15:23.955 29831-29831/? D/MainActivity: reuseBitmap: byte_count = 1036800,allocation_byte_count =1036800
  2020-03-04 20:15:23.964 29831-29831/? D/MainActivity: reuseBitmap: android.graphics.Bitmap@57148f
  2020-03-04 20:15:23.964 29831-29831/? D/MainActivity: reuseBitmap: width = 360,height = 180
  2020-03-04 20:15:23.964 29831-29831/? D/MainActivity: reuseBitmap: byte_count = 259200,allocation_byte_count =1036800
  
   //同一个对象
    // 复用后的bitmap getAllocationByteCount 和 getByteCount 也是不一样的。
  ```

  

###### 复用也是有版本的问题的

- 在Android4.4之前只有格式为jpg、png，同等宽高，inSampleSize为1的Bitmap才可以复用。
- 在Android4.4之后开始，被复用的Bitmap内存大于需要新申请内存的Bitmap的内存就可以。

#### 说下getByteCount()和getAllocationByteCount()的区别

- 两者一般情况下是相等的。
- 通过复用Bitmap来解码图片的时候，如果被复用的Bitmap的内存比待内存分配的Bitmap要大的时候
  - getByteCode表示新的Bitmap的像素大小也就是占用的内存大小。
  - getAllocationByteCode表示要被复用的Bitmap的真实内存大小

## 计算Bitmap占用的内存

> 让人想到的就是： width x height x 一个像素占用的内存大小

#### BitmapFactory.decodeResource()

- 使用这个API进行图片的解码，bitmap在内存中占用的大小 = width x scale x height x scale x 一个像素占用的内存大小; 
- 其中 scale(缩放系数) = mTargetDensity/inDensity;
- 这里面的width和height，可以理解为图片的原始宽和高，当图片放进不同的drawable文件夹中的时候，内存中Bitmap的宽和高 与图片的实际宽和高有可能不一样大了。
- 正常情况下我们所说的公式 width和height 是Bitmap的，
- 一张图片在相同app中的不同的资源文件夹下 占用的内存可能不一样大。

#### BitmapFactory.decodeFile()

- 该方法从磁盘中加载到内存中不涉及到缩放（inDenisty和inTargetDensity）的影响。

## Bitmap的压缩

#### 质量压缩(Bitmap.compress())

- 它在不改变图片的像素的提前下，改变图片的位深和透明度，来进行压缩图片。
- 但是经过质量压缩后磁盘中的File会变小，但是解码后的Bitmap在内存中的大小是不变的。

```java
/**
     * 质量压缩
     * @param bitmap
     * @return
     */
    private Bitmap compressBitmap(Bitmap bitmap) {
        ByteArrayOutputStream ous = new ByteArrayOutputStream();
        bitmap.compress(Bitmap.CompressFormat.JPEG, 100, ous);
        int options = 100;
        while (ous.toByteArray().length / 1024 > 100) {
            ous.reset();
          	options -= 10;
            bitmap.compress(Bitmap.CompressFormat.JPEG, options, ous);
            
        }
        ByteArrayInputStream bIn = new ByteArrayInputStream(ous.toByteArray());
        Bitmap compressBitmap = BitmapFactory.decodeStream(bIn, null, null);
        return compressBitmap;
    }
```

###### 质量压缩后的Bitmap，再次保存到本地变大了？

```java
public static boolean saveBitmapToJPG(Bitmap bitmap, File file) {
    if (bitmap == null)
        return false;
    FileOutputStream fos = null;
     if (file.exists()){
         file.delete();
     }
    try {
        fos = new FileOutputStream(file);
        bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
        fos.flush();
        fos.close();
        return true;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return false;
}


```

- 原因是压缩后的Bitmap经过了一次编码，导致保存的Bitmap过大

###### 可以这样搞

```java
 compressPic(BitmapFactory.decodeResource(getResources(),R.drawable.scal_density));
        String path = Environment.getExternalStorageDirectory().getPath();
        File file = new File(path, "/test/2.jpg");
        if (!file.exists()) {
            file.getParentFile().mkdirs();
        }

        try {
            FileOutputStream fileOutputStream = new FileOutputStream(file);
            fileOutputStream.write(ous.toByteArray());
            fileOutputStream.flush();
            fileOutputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
```



#### 采样率压缩(BitmapFactory.Options.inSampleSize)

- 可以真实压缩解码后的Bitmap在内存中的大小。

- 首先我们设置inJustDecodeBounds = true 不加载Bitmap到内存中，但是可以获取Bitmap的原始宽高。

- 根据Bitmap的宽高和想要显示的大小，计算合适的inSampleSize设置黑Options.inSampleSize = 

- 然后将inJustDecodeBounds = false ,意味着可以进行加载Bitmap到内存中了。

  ```java
    private static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
          int outWidth = options.outWidth;
          int outHeight = options.outHeight;
          int inSampleSize = 1;
  
          if (outWidth > reqWidth || outHeight > reqHeight) {
              final int halfHeight = outHeight / 2;
              final int halfWidth = outWidth / 2;
  
              while ((halfWidth / inSampleSize) > reqWidth && (halfHeight / inSampleSize) > reqHeight) {
                  inSampleSize *= 2;
              }
          }
  
          return inSampleSize;
      }
  
      /**
       * 采样率压缩法
       * @param path
       * @param reqWidth
       * @param reqHeight
       * @return
       */
      private static Bitmap decodeBitmapFromFile(String path, int reqWidth, int reqHeight) {
          BitmapFactory.Options options = new BitmapFactory.Options();
          //设置读取图片的时候仅仅读取图片的长和宽
          options.inJustDecodeBounds = true;
          Bitmap bitmap = BitmapFactory.decodeFile(path, options);
          //计算合适的缩放比
          options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
          options.inPreferredConfig = Bitmap.Config.RGB_565;
          options.inJustDecodeBounds = true;
          Bitmap realBitmap = BitmapFactory.decodeFile(path, options);
          return realBitmap;
      }
  ```

  



