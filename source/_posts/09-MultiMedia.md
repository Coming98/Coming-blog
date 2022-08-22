---
title: 09-MultiMedia
mathjax: false
date: 2022-08-22 17:40:39
summary: Android 多媒体的操作
categories: Android Dev.
tags:
  - Android
---

# Notification

Application: 应用程序不在前台运行且希望向用户发出提示信息时

通知渠道: 对通知信息的细分, 可以按照通知渠道控制一类通知的重要程度(是否响铃、是否振动或者是否要关闭这个渠道的通知)

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208221141721.png)

## Quick Start

1. 获取通知管理对象: `NotificationManager`

```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
```

2. 创建通知渠道: `NotificationChannel`

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    // channelId can be defined arbitrarily
    // channelName is for display
    // importance: NotificationManager.IMPORTANCE_HIGH, IMPORTANCE_DEFAULT, IMPORTANCE_LOW, IMPORTANCE_MIN, different notification importance means different notify action and user can change it at will
    val channel = NotificationChannel(channelId: String, channelName: String, importance)
    manager.createNotificationChannel(channel)
}
```

3. 设定通知触发的意图: 这里以触发一个 Activity 为例

> PendingIntent can be understood as an intent to delay execution
> Params list is: context, 0, intent object, action (set 0 in basic use)

```kotlin
val intent = Intent(this, NotificationActivity::class.java)
val pendingIntent = PendingIntent.getActivity(this, 0, intent, 0)
```

4. 创建通知对象: `NotificationCompat.Builder(context, channelId: String)`

```kotlin
val notification = NotificationCompat.Builder(this, "normal")
                .setContentTitle("This is content title")
                .setContentText("This is content text")
                .setSmallIcon(R.drawable.small_icon) // 只能使用纯 alpha 图层的图片进行设置, 在系统状态栏显示
                .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon)) // 下拉状态栏后显示
                .setContentIntent(pendingIntent) // PendingIntent
                .setAutoCancel(true) // 表示当点击这个通知的时候，通知会自动取消
                .build()
```

5. 触发通知: `manager.notify(id: Int, notification)`

```kotlin
// 需要保证每个通知指定的 id 是不同的
manager.notify(1, notification)
```

## 长文字通知

默认情况下, 通知内容多余一行将会被 `...` 截断, 可以通过 setStyle 中的 NotificationCompat.BigTextStyle() 进行富文本化

```kotlin
.setStyle(NotificationCompat.BigTextStyle().bigText(content)) // replace the .setContentText
```

## 图片通知

通过 setStyle 中的 NotificationCompat.BigPictureStyle() 进行展示

```kotlin
.setStyle(NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(resources, R.drawable.big_image)))
```

# Take Photo

调用摄像头媒体进行拍照并展示照片: 
- 定义一个 File 对象用于存储拍下的照片, 避免涉及运行时权限, 选择存储在 SD 卡的应用关联缓存目录下 `getExternalCacheDir()`
- 构建目标照片的 Uri: 通过 `getUriForFile()` 进行获取


```kotlin
buttonTakePhoto.setOnClickListener {
    // File 对象, 存放摄像头拍下的照片
    // 存放在手机 SD 卡的应用关联缓存目录下 `/sdcard/Android/data/<package name>/cache`
    outputImage = File(externalCacheDir, "output_image.jpg")
    if (outputImage.exists()) {
        outputImage.delete()
    }
    outputImage.createNewFile()
    imageUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        // 将 File 对象转换成一个封装过的 Uri 对象: context, 任意唯一字符串, file
        // FileProvider 是一种特殊的 ContentProvider，它使用了和 ContentProvider 类似的机制来对数据进行保护，可以选择性地将封装过的 Uri 共享给外部，从而提高了应用的安全性
        FileProvider.getUriForFile(this, "com.example.a16cameraalbumtest.fileprovider", outputImage)
    } else {
        Uri.fromFile(outputImage)
    }
    // 启动相机程序
    val intent = Intent("android.media.action.IMAGE_CAPTURE")
    // 图片输出地址
    intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri)
    // 获取相机拍摄后的流文件
    ImageCaptureActivityForResult.launch(intent)
}
```

拍照完成后相机 Activity 将数据流返回, 通过 contentResolver 与 Uri 进行获取

```kotlin
private val ImageCaptureActivityForResult = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
    result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val bitmap = BitmapFactory.decodeStream(contentResolver.openInputStream(imageUri))
        // 拍照时可能旋转了手机
        imageView.setImageBitmap(rotateIfRequired(bitmap))
    }
}

private fun rotateIfRequired(bitmap: Bitmap): Bitmap {
    val exif = ExifInterface(outputImage.path)
    val orientation = exif.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL)
    return when (orientation) {
        ExifInterface.ORIENTATION_ROTATE_90 -> rotateBitmap(bitmap, 90)
        ExifInterface.ORIENTATION_ROTATE_180 -> rotateBitmap(bitmap, 180)
        ExifInterface.ORIENTATION_ROTATE_270 -> rotateBitmap(bitmap, 270)
        else -> bitmap
    }
}
private fun rotateBitmap(bitmap: Bitmap, degree: Int): Bitmap {
    val matrix = Matrix()
    matrix.postRotate(degree.toFloat())
    val rotatedBitmap = Bitmap.createBitmap(bitmap, 0, 0, bitmap.width, bitmap.height, matrix, true)
    bitmap.recycle()
    return rotatedBitmap
}
```

声明注册 provider: meta-data 中用于指定 uri 的共享路径通过 xml 资源进行赋值

```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="com.example.a16cameraalbumtest.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
<!-- external-path 用于指定 Uri 的共享路径, name 属性的值可以随便填写, path 属性表示共享的具体路径 -->
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="/" />
</paths>
```

# Select Photos

从相册中选择照片, 打开系统的文件选择器 `Intent.ACTION_OPEN_DOCUMENT` 指定文件类别为 `Intent.CATEGORY_OPENABLE`

```kotlin
buttonSelectPhoto.setOnClickListener {
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
    intent.addCategory(Intent.CATEGORY_OPENABLE)
    // 指定要显示的图片
    intent.type = "image/*"
    SelectPhotoForResult.launch(intent)
}
```

选定照片后通过返回的 Uri 获取资源对象

```kotlin
private val SelectPhotoForResult = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
    if (result.resultCode == Activity.RESULT_OK && result.data != null) {
        result.data?.data.let { uri ->
            if (uri != null) {
                val bitmap = getBitmapFromUri(uri)
                imageView.setImageBitmap(bitmap)
            }
        }
    }
}

private fun getBitmapFromUri(uri: Uri) = contentResolver.openFileDescriptor(uri, "r")?.use {
    BitmapFactory.decodeFileDescriptor(it.fileDescriptor)
}
```

# Play Audio

主要通过 MediaPlayer 类实现的:
- setDataSource(): 设置要播放的音频文件的位置
- prepare(): 开始播放之前调用, 完成准备工作
- start(): 开始/继续播放
- pause()
- reset(): 将 MediaPlayer 重置为刚刚创建的状态
- seekTo(): 从指定位置开始播放音频
- stop()
- release()
- isPlaying()
- getDuration(): 获取载入的音频文件的时长

1. 获取 MediaPlayer 对象 并进行初始化

```kotlin
private val mediaPalyer = MediaPlayer()
private fun initMediaPlayer() {
    val assetManager = assets // getAssets()
    // 打开目标文件的句柄
    val fd = assetManager.openFd("music.mp3")
    // file descriptor
    // the offset into the file where the data to be played starts, in bytes
    // the length in bytes of the data to be played
    mediaPalyer.setDataSource(fd.fileDescriptor, fd.startOffset, fd.length)
    mediaPalyer.prepare()
}
```

2. 按钮逻辑操作

```kotlin
when(v?.id) {
    R.id.play -> {
        if (!mediaPalyer.isPlaying) {
            mediaPalyer.start()
        }
    }
    R.id.pause -> {
        if (mediaPalyer.isPlaying) {
            mediaPalyer.pause()
        }
    }
    R.id.stop -> {
        mediaPalyer.reset()
        initMediaPlayer()
    }
}
```

3. Activity 被销毁时记得释放 MediaPlayer

```kotlin
override fun onDestroy() {
    super.onDestroy()
    mediaPalyer.stop()
    mediaPalyer.release()
}
```

# Play Video

主要通过 VideoView 类实现的, 通过命名可以看出, 这个类将 Video 的显示与控制集一身:
- setVideoPath(): 设置要播放的视频文件
- resume(): 将视频从头开始播放
- suspend(): 释放 VideoView
- start, pause, seekTo, isPlaying, getDuration

Tips: VideoView 不支持直接播放 assets 目录下的视频资源, 但是支持播放 `res/raw/*` 目录下的视频资源

1. 设置媒体对象

```kotlin
videoView = findViewById(R.id.videoView)

val uri = Uri.parse("android.resource://$packageName/${R.raw.video}")
videoView.setVideoURI(uri)
videoView.setMediaController(MediaController(this))
```

2. 逻辑控制

```kotlin
when(v?.id) {
    R.id.play -> {
        if (!videoView.isPlaying) {
            videoView.start()
        }
    }
    R.id.pause -> {
        if (videoView.isPlaying) {
            videoView.pause()
        }
    }
    R.id.replay -> {
        if (videoView.isPlaying) {
            videoView.resume()
        }
    }
}
```
3. 事后释放

```kotlin
override fun onDestroy() {
    super.onDestroy()
    videoView.suspend()
}
```
