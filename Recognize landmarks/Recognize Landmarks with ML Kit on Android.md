# 在安卓上使用ML Kit识别地标

您可以使用ML kit来识别图片中的著名地标。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-android/tree/master/mlkit)上的ML Kit快速入门示例。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/android/setup)来开始您的工作。

2. 在app-level的`build.gradle` 文件中为ML kit添加依赖:

   ```java
   dependencies {
     // ...
   
     implementation 'com.google.firebase:firebase-ml-vision:15.0.0'
   }
   ```

3. 如果您想使用基于云的模型，并且尚未将项目升级到Blaze计划，请在[Firebase控制台](https://console.firebase.google.com/)中执行此操作。只有Blaze计划的项目才能使用Cloud Vision API。 

4. 如果您想使用基于云的模型，您也需要开启Cloud Vision API：

   - 在云API列表管理平台中打开[Cloud Vision API](https://console.cloud.google.com/apis/library/vision.googleapis.com/)。
   - 确保您的Firebase项目已经在当前菜单页面中被置于顶端。
   - 如果API依旧还是显示为enabled，请点击Enable。

## 配置地标识别器

默认情况下，云识别器使用`STABLE`版本的模型并返回多达10个结果。如果您想要更改这些设置中的任何一个，请使用下例中的 `FirebaseVisionCloudDetectorOptions` 对象指定它们。 

例如，要更改这两个默认设置，请按照以下示例构建一个`FirebaseVisionCloudDetectorOptions` 对象：

```java
FirebaseVisionCloudDetectorOptions options =
    new FirebaseVisionCloudDetectorOptions.Builder()
        .setModelType(FirebaseVisionCloudDetectorOptions.LATEST_MODEL)
        .setMaxResults(15)
        .build();
```

要使用默认设置，可以在下一步中使用`FirebaseVisionCloudDetectorOptions.DEFAULT` 。 

## 运行地标识别器

为了进行地标识别，从任一个`Bitmap`，`media.Image`，`ByteBuffer`，字节阵列，或在设备上的文件中创建一个`FirebaseVisionImage`对象。然后，传递`FirebaseVisionImage`对象到 `FirebaseVisionCloudLandmarkDetector` 的`detectInImage`方法。

1. 从您的图像中创建一个 `FirebaseVisionImage`  对象。

   - 从 `Bitmap`对象创建`FirebaseVisionImage`对象：

     ```java
     FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);
     ```

     `Bitmap` 对象指示的图像必须是直立的，不能旋转。 

   - 要从 `media.Image`对象创建`FirebaseVisionImage`对象（例如从设备的相机捕捉图像时），首先要确定图像必须旋转的角度，以补偿设备的旋转和相机传感器在设备中的方向： 

     ```java
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
     // Or: 
     // FirebaseVisionImage image = FirebaseVisionImage.fromByteArray(byteArray, metadata);
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

2. 得到一个 `FirebaseVisionCloudLandmarkDetector` 的实例：

   ```java
   FirebaseVisionCloudLandmarkDetector detector = FirebaseVision.getInstance()
           .getVisionCloudLandmarkDetector();
   // 或者，您可以使用默认设定:
   // FirebaseVisionCloudLandmarkDetector detector = FirebaseVision.getInstance()
   //         .getVisionCloudLandmarkDetector(options);
   ```

3. 最后传递图片到 `detectInImage` 方法：

   ```java
   Task<List<FirebaseVisionCloudLandmark>> result = detector.detectInImage(image)
           .addOnSuccessListener(new OnSuccessListener<List<FirebaseVisionCloudLandmark>>() {
               @Override
               public void onSuccess(List<FirebaseVisionCloudLandmark> firebaseVisionCloudLandmarks) {
                   // 任务成功
                   // ...
               }
           })
           .addOnFailureListener(new OnFailureListener() {
               @Override
               public void onFailure(@NonNull Exception e) {
                   // 任务失败并且报异常
                   // ...
               }
           });
   ```

## 获取著名地标的有关信息

如果地标识别操作成功，`FirebaseVisionCloudLandmark`对象列表 将传递给成功侦听器(success listener)。每个`FirebaseVisionCloudLandmark`对象代表在图像中识别的地标。对于每个地标，您可以输入图像，得到它的图像中的坐标，地标名称，经度和纬度，[知识图谱实体ID](https://developers.google.com/knowledge-graph/) （如果可用）以及匹配的置信度得分。例如： 

```java
for (FirebaseVisionCloudLandmark landmark: firebaseVisionCloudLandmarks) {

    Rect bounds = landmark.getBoundingBox();
    String landmarkName = landmark.getLandmark();
    String entityId = landmark.getEntityId();
    float confidence = landmark.getConfidence();

    // 多个位置是可能的，例如，所描绘的地标的位置和照片的拍摄位置。
    for (FirebaseVisionLatLng loc: landmark.getLocations()) {
        double latitude = loc.getLatitude();
        double longitude = loc.getLongitude();
    }
}
```