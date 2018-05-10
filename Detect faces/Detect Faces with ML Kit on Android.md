# 在安卓上使用ML Kit识别人脸

你可以在安卓上使用ML Kit来在图像和视频中识别人脸。

请参阅GitHub上的[ML Kit快速入门示例](https://github.com/firebase/quickstart-android/tree/master/mlkit)，了解正在使用的此API的示例。 

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/android/setup)来开始您的工作。

2. 在应用级的`build.gradle` 文件中为ML kit添加依赖:

   ```java
   dependencies {
     // ...
   
     implementation 'com.google.firebase:firebase-ml-vision:15.0.0'
   }
   ```

3. 可选但建议：如果您使用设备上的API，请将应用配置为在从Play商店安装应用后自动将ML模型下载到设备： 

   ```java
   <meta-data
       android:name="com.google.firebase.ml.vision.DEPENDENCIES"
       android:value="text" />
   <!-- 为了能够使用多个模型: android:value="text,model2,model3" -->
   ```

    如果您未启用安装时模型下载，则将在您首次运行设备上识别器的时候开始下载该模型。在下载完成之前发出的请求不会有任何结果。

## 在设备上识别人脸

### 设置人脸识别器

在您对图像进行人脸识别以前，如果您想要修改任何关于人脸识别的默认设定，请通过`FirebaseVisionFaceDetectorOptions ` 对象来指定这些设定。您可以修改以下设定：

| 设置         | 参数                        | 参数2                 | 解释                                                     |
| ------------ | --------------------------- | --------------------- | -------------------------------------------------------- |
| 识别模式     | `FAST_MODE` (默认)          | `ACCURATE_MODE`       | 取决于您在人脸识别中是喜欢速度或准确性。                 |
| 识别特征点   | `NO_LANDMARKS` (默认)       | `ALL_LANDMARKS`       | 是否尝试识别面部“特征点”：眼睛，耳朵，鼻子，脸颊，嘴巴。 |
| 辨认脸部     | `NO_CLASSIFICATIONS` (默认) | `ALL_CLASSIFICATIONS` | 是否将人脸分类为“微笑”和“睁眼”等类别。                   |
| 脸部最小识别 | `float` (默认: `0.1f`)      |                       | 最小识别多大的脸部，与照片大小有关。                     |
| 允许人脸追踪 | `false` (默认)              | `true`                | 是否分配人脸ID，可用于跟踪图像中的人脸。                 |

例如，为了修改以上所有的默认设定，在下边的例子中构建一个 `FirebaseVisionFaceDetectorOptions `  对象：

```java
FirebaseVisionFaceDetectorOptions options =
    new FirebaseVisionFaceDetectorOptions.Builder()
        .setModeType(FirebaseVisionFaceDetectorOptions.ACCURATE_MODE)
        .setLandmarkType(FirebaseVisionFaceDetectorOptions.ALL_LANDMARKS)
        .setClassificationType(FirebaseVisionFaceDetectorOptions.ALL_CLASSIFICATIONS)
        .setMinFaceSize(0.2f)
        .setTrackingEnabled(true)
        .build();
```

### 运行人脸识别器

识别图像中的人脸，从任一个`Bitmap`，`media.Image`，`ByteBuffer`或者字节阵列创建一个`FirebaseVisionImage`对象，抑或在设备上的文件中选取。然后，传递`FirebaseVisionImage`对象到 `FirebaseVisionFaceDetector`的`detectInImage`方法。 

1. 从图像中创建一个`FirebaseVisionImage`对象。 

   - 从 `Bitmap`对象创建`FirebaseVisionImage` 对象：

     ```java
     FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);
     ```

     `Bitmap`物体 表示的图像必须是直立的，不要任何额外的旋转。 

   - 要从`media.Image`对象创建`FirebaseVisionImage`对象 （例如从设备的相机捕捉图像时），首先要确定图像必须旋转的角度，以补偿设备的旋转和相机传感器在设备中的方向差： 

     ```
     private static final SparseIntArray ORIENTATIONS = new SparseIntArray();
     static {
         ORIENTATIONS.append(Surface.ROTATION_0, 90);
         ORIENTATIONS.append(Surface.ROTATION_90, 0);
         ORIENTATIONS.append(Surface.ROTATION_180, 270);
         ORIENTATIONS.append(Surface.ROTATION_270, 180);
     }
     
     /**
      * 得到当前图像需要补偿的角度
      */
     @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
     private int getRotationCompensation(String cameraId, Activity activity, Context context)
             throws CameraAccessException {
         // 得到设备当前与原始的角度的旋转差值
         // 然后照片一定要旋转回去相对的差值
         int deviceRotation = activity.getWindowManager().getDefaultDisplay().getRotation();
         int rotationCompensation = ORIENTATIONS.get(deviceRotation);
     
         // 在大多数的设备上，传感器的方向是90度。但是对于
         // 少数设备，这个值是270度。那么对于这些270度的设备
         // 必须让照片旋转额外的180 ((270 + 270) % 360) 度.
         CameraManager cameraManager = (CameraManager) context.getSystemService(CAMERA_SERVICE);
         int sensorOrientation = cameraManager
                 .getCameraCharacteristics(cameraId)
                 .get(CameraCharacteristics.SENSOR_ORIENTATION);
         rotationCompensation = (rotationCompensation + sensorOrientation + 270) % 360;
     
         // 返回相关的FirebaseVisionImageMetadata rotation值。
         int result;
         switch (rotationCompensation) {
             case 0:
                 result = FirebaseVisionImageMetadata.ROTATION_0;
                 break;
             case 90:
                 result = FirebaseVisionImageMetadata.ROTATION_90;
                 break;
             case 180:
                 result = FirebaseVisionImageMetadata.ROTATION_180;
                 break;
             case 270:
                 result = FirebaseVisionImageMetadata.ROTATION_270;
                 break;
             default:
                 result = FirebaseVisionImageMetadata.ROTATION_0;
                 Log.e(TAG, "Bad rotation value: " + rotationCompensation);
         }
         return result;
     }
     ```

     然后，将`media.Image`对象和旋转值传递给`FirebaseVisionImage.fromMediaImage()`： 

     ```
     FirebaseVisionImage image = FirebaseVisionImage.fromMediaImage(mediaImage, rotation);
     ```

   - 要从一个字节数组或`ByteBuffer`创建一个`FirebaseVisionImage`对象，首先按照上面的描述计算图像旋转角度。

     然后，创建一个包含图像高度，宽度，颜色编码格式和旋转度的`FirebaseVisionImageMetadata`对象：

     ```java
     FirebaseVisionImageMetadata metadata = new FirebaseVisionImageMetadata.Builder()
             .setWidth(1280)
             .setHeight(720)
             .setFormat(FirebaseVisionImageMetadata.IMAGE_FORMAT_NV21)
             .setRotation(rotation)
             .build();
     ```

     使用缓冲区或数组以及元数据对象来创建一个 `FirebaseVisionImage`对象： 

     ```java
     FirebaseVisionImage image = FirebaseVisionImage.fromByteBuffer(buffer, metadata);
     // 或者: FirebaseVisionImage image = FirebaseVisionImage.fromByteArray(byteArray, metadata);
     ```

   - 要从文件创建`FirebaseVisionImage`对象，请将应用context和文件URI传递给`FirebaseVisionImage.fromFilePath()`： 

     ```java
     FirebaseVisionImage image;
     try {
         image = FirebaseVisionImage.fromFilePath(context, uri);
     } catch (IOException e) {
         e.printStackTrace();
     }
     ```

2. 获取一个`FirebaseVisionTextDetector`实例：  

   ```java
   FirebaseVisionFaceDetector detector = FirebaseVision.getInstance()
       .getVisionFaceDetector(options);
   ```

   **注意：请多多检查控制台显示的错误**

3. 最后，将图像传递给`detectInImage`方法： 

   ```java
   Task<List<FirebaseVisionFace>> result =
           detector.detectInImage(image)
                   .addOnSuccessListener(
                           new OnSuccessListener<List<FirebaseVisionFace>>() {
                               @Override
                               public void onSuccess(List<FirebaseVisionFace> faces) {
                                   // 任务成功
                                   // ...
                               }
                           })
                   .addOnFailureListener(
                           new OnFailureListener() {
                               @Override
                               public void onFailure(@NonNull Exception e) {
                                   // 任务失败并且报错
                                   // ...
                               }
                           });
   ```

### 获取检测到的面部有关信息

如果人脸检测操作成功，则人脸识别器将一组`FirebaseVisionFace` 对象传递给成功监听器（success listener ）。每个 `FirebaseVisionFace` 对象代表在图像中检测到的脸部。对于每个人脸，您可以在输入图像中获得其边界坐标以及您配置人脸识别器查找的任何其他信息。例如：

```java
for (FirebaseVisionFace face : faces) {
    Rect bounds = face.getBoundingBox();
    float rotY = face.getHeadEulerAngleY();  // 头部转向右rotY角
    float rotZ = face.getHeadEulerAngleZ();  // 头部转向上rotZ角

    // 如果特征点识别开启了（嘴，耳，眼，脸颊，还有鼻子可以检测）：
    FirebaseVisionFaceLandmark leftEar = face.getLandmark(FirebaseVisionFaceLandmark.LEFT_EAR);
    if (leftEar != null) {
        FirebaseVisionPoint leftEarPos = leftEar.getPosition();
    }

    // 如果辨认功能开启了:
    if (face.getSmilingProbability() != FirebaseVisionFace.UNCOMPUTED_PROBABILITY) {
        float smileProb = face.getSmilingProbability();
    }
    if (face.getRightEyeOpenProbability() != FirebaseVisionFace.UNCOMPUTED_PROBABILITY) {
        float rightEyeOpenProb = face.getRightEyeOpenProbability();
    }

    // 如果脸部追踪开启了:
    if (face.getTrackingId() != FirebaseVisionFace.INVALID_ID) {
        int id = face.getTrackingId();
    }
}
```