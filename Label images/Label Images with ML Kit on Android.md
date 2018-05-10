# 在安卓上使用ML Kit标注图片

您可以使用ML Kit来识别图像中的文本，使用设备上的模型或云上的模型。请参阅[概述](https://github.com/Quorafind/MLkit-CN/blob/master/ML%20kit%20for%20Firebase.md)以了解每种方法的优点。

有关此API使用的示例，请参阅 [GitHub](https://github.com/firebase/quickstart-android/tree/master/mlkit) 上的 ML Kit 快速入门示例，或者尝试使用 [codelab](http://g.co/codelabs/mlkit-android) 。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/android/setup)来开始您的工作。

2. 在app-level的 `build.gradle` 文件中为ML kit添加依赖:

   ```java
   dependencies {
     // ...
   
     implementation 'com.google.firebase:firebase-ml-vision:15.0.0'
     implementation 'com.google.firebase:firebase-ml-vision-image-label-model:15.0.0'
   }
   ```

3. 可选但建议：如果您使用设备上的API，请将应用配置为在从Play商店安装应用后自动将ML模型下载到设备： 

   ```java
   <meta-data
       android:name="com.google.firebase.ml.vision.DEPENDENCIES"
       android:value="label" />
   <!-- 为了能够使用多个模型: android:value="text,model2,model3" -->
   ```

    如果您未启用安装时模型下载，则将在您首次运行设备上识别器的时候开始下载该模型。在下载完成之前发出的请求不会有任何结果。

4. 如果您想使用基于云的模型，并且尚未将项目升级到Blaze计划，请在[Firebase控制台](https://console.firebase.google.com/)中执行此操作。只有Blaze计划的项目才能使用Cloud Vision API。 

5. 如果您想使用基于云的模型，您也需要开启Cloud Vision API：

   - 在云API列表管理平台中打开[Cloud Vision API](https://console.cloud.google.com/apis/library/vision.googleapis.com/)。
   - 确保您的Firebase项目已经在当前菜单页面中被置于顶端。
   - 如果API依旧还是显示为enabled，请点击Enable。

   如果您想要仅仅开启使用设备上的模型，您可以跳过这一步。

现在您已经可以开始使用设备上的模型或者基于云端的模型识别图像中的文本了。

- [在设备上标注图片](#在设备上标注图片)
- [基于云端的图片标注](#基于云端的图片标注)

## 在设备上标注图片

要使用基于云端的图片标注模型，请按以下所述配置和运行图像标注器。

1. ### 配置图像标注器

   默认情况下，图片标注后至多会返回10个标签。如果您想要更改该设置，请按照以下实例创建一个`FirebaseVisionLabelDetectorOptions `对象。

   ```
   FirebaseVisionLabelDetectorOptions options =
           new FirebaseVisionLabelDetectorOptions.Builder()
                   .setConfidenceThreshold(0.8f)
                   .build();
   ```

2. ### 运行图像标注器

   为了识别图像中的文本，从任一个`Bitmap`，`media.Image`，`ByteBuffer`，字节阵列，或在设备上的文件中创建一个`FirebaseVisionImage`对象。然后，传递`FirebaseVisionImage`对象到`FirebaseVisionLabelDetector`的`detectInImage`方法。

   1. 从图像中创建一个`FirebaseVisionImage`对象。如果您使用`Bitmap`则图像标签器运行速度最快或者如果您使用camera2 API，一种名为`media.Image`的JPEG格式，也可最快。当然如果可能，建议您使用（后者）这种格式。

      - 从 `Bitmap`对象创建`FirebaseVisionImage`对象：

        ```java
        FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);
        ```

        `Bitmap` 对象指示的图像必须是直立的，不能旋转。 

      - 要从 `media.Image`对象创建`FirebaseVisionImage`对象（例如从设备的相机捕捉图像时），首先要确定图像必须旋转的角度，以补偿设备的旋转和相机传感器在设备中的方向： 

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

        ```java
        FirebaseVisionImage image = FirebaseVisionImage.fromMediaImage(mediaImage,rotation);
        ```

      - 要从一个字节数组或`ByteBuffer`创建一个`FirebaseVisionImage`对象，首先按照上面的描述计算图像旋转角度。

        然后，创建一个包含图像高度，宽度，颜色编码格式和旋转度的`FirebaseVisionImageMetadata`对象：

        ```
        FirebaseVisionImageMetadata metadata = new FirebaseVisionImageMetadata.Builder()
                .setWidth(1280)
                .setHeight(720)
                .setFormat(FirebaseVisionImageMetadata.IMAGE_FORMAT_NV21)
                .setRotation(rotation)
                .build();
        ```

        使用缓冲区或数组以及元数据对象来创建一个 `FirebaseVisionImage`对象： 

        ```
        FirebaseVisionImage image = FirebaseVisionImage.fromByteBuffer(buffer, metadata);
        // Or: FirebaseVisionImage image = FirebaseVisionImage.fromByteArray(byteArray, metadata);
        ```

      - 要从文件创建`FirebaseVisionImage`对象，请将应用context和文件URI传递给`FirebaseVisionImage.fromFilePath()`： 

        ```
        FirebaseVisionImage image;
        try {
            image = FirebaseVisionImage.fromFilePath(context, uri);
        } catch (IOException e) {
            e.printStackTrace();
        }
        ```

   2. 获取一个 `FirebaseVisionLabelDetector`实例:

      ```
      FirebaseVisionLabelDetector detector = FirebaseVision.getInstance()
              .getVisionLabelDetector();
      // 或者设置您想要的最小置信度:
      FirebaseVisionLabelDetector detector = FirebaseVision.getInstance()
              .getVisionLabelDetector(options);
      ```

   3. 最后，传递图片到 `detectInImage`  方法：

      ```java
      Task<List<FirebaseVisionLabel>> result =
              detector.detectInImage(image)
                      .addOnSuccessListener(
                              new OnSuccessListener<List<FirebaseVisionLabel>>() {
                                  @Override
                                  public void onSuccess(List<FirebaseVisionLabel> labels) {
                                      // 任务完成
                                      // ...
                                  }
                              })
                      .addOnFailureListener(
                              new OnFailureListener() {
                                  @Override
                                  public void onFailure(@NonNull Exception e) {
                                      // 任务失败并出异常
                                      // ...
                                  }
                              });
      ```

3. ### 获取被标注的对象的相关信息

   如果图像标注操作成功，则会将`FirebaseVisionLabel` 对象列表传递给成功侦听器(success listener)。每个`FirebaseVisionLabel` 对象都表示图像中标记的内容。对于每个标签，您可以获取标签的文本说明， [知识图谱实体标识ID](https://developers.google.com/knowledge-graph/) （如果可用）以及匹配物体/行为的置信度分数。例如： 

   ```java
   for (FirebaseVisionLabel label: labels) {
       String text = label.getLabel();
       String entityId = label.getEntityId();
       float confidence = label.getConfidence();
   }
   ```

## 基于云端的图片标注

为了使用基于云端的图片标注模型，请按下边描述配置和运行图片标注器

1. ### 配置图像标注器

   默认情况下，云端识别器使用模型的`STABLE`版本，并返回多达10个的结果。如果您要修改这些设置中的任何一个，请用`FirebaseVisionCloudDetectorOptions`  对象指定它们。

   例如，要更改这两个默认设置，请按照以下示例构建一个`FirebaseVisionCloudDetectorOptions `对象。

   ```
   FirebaseVisionCloudDetectorOptions options =
       new FirebaseVisionCloudDetectorOptions.Builder()
           .setModelType(FirebaseVisionCloudDetectorOptions.LATEST_MODEL)
           .setMaxResults(15)
           .build();
   ```

   要使用默认设置，您可以在下一步中使用 `FirebaseVisionCloudDetectorOptions`。

2. ###  运行图像标注器

   为了进行图片标注，从任一个`Bitmap`，`media.Image`，`ByteBuffer`，字节阵列，或在设备上的文件中创建一个`FirebaseVisionImage`对象。然后，传递`FirebaseVisionImage`对象到`FirebaseCloudVisionLabelDetector`的`detectInImage`方法。

   1. #### 从图像中创建一个`FirebaseVisionImage`对象。如果您使用`Bitmap`则图像标签器运行速度最快或者如果您使用camera2 API，一种名为`media.Image`的JPEG格式，也可最快。当然如果可能，建议您使用（后者）这种格式。

      - 从 `Bitmap`对象创建`FirebaseVisionImage`对象：

        ```
        FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);
        ```

        `Bitmap` 对象指示的图像必须是直立的，不能旋转。 

      - 要从 `media.Image`对象创建`FirebaseVisionImage`对象（例如从设备的相机捕捉图像时），首先要确定图像必须旋转的角度，以补偿设备的旋转和相机传感器在设备中的方向： 

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

        ```java
        FirebaseVisionImage image = FirebaseVisionImage.fromMediaImage(mediaImage,rotation);
        ```

      - 要从一个字节数组或`ByteBuffer`创建一个`FirebaseVisionImage`对象，首先按照上面的描述计算图像旋转角度。

        然后，创建一个包含图像高度，宽度，颜色编码格式和旋转度的`FirebaseVisionImageMetadata`对象：

        ```
        FirebaseVisionImageMetadata metadata = new FirebaseVisionImageMetadata.Builder()
                .setWidth(1280)
                .setHeight(720)
                .setFormat(FirebaseVisionImageMetadata.IMAGE_FORMAT_NV21)
                .setRotation(rotation)
                .build();
        ```

        使用缓冲区或数组以及元数据对象来创建一个 `FirebaseVisionImage`对象： 

        ```
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

   2. #### 获得一个 `FirebaseVisionCloudLabelDetector` 实例：

      ```
      FirebaseVisionCloudLabelDetector detector = FirebaseVision.getInstance()
              .getVisionCloudLabelDetector();
      // 或者，修改默认设置:
      // FirebaseVisionCloudLabelDetector detector = FirebaseVision.getInstance()
      //         .getVisionCloudLabelDetector(options);
      ```

   3. #### 最后，将图像传递至`detectInImage `方法：

      ```
      Task<List<FirebaseVisionCloudLabel>> result =
              detector.detectInImage(image)
                      .addOnSuccessListener(
                              new OnSuccessListener<List<FirebaseVisionCloudLabel>>() {
                                  @Override
                                  public void onSuccess(List<FirebaseVisionCloudLabel> labels) {
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

3. ### 获取被标注的对象的相关信息

   如果图像标注操作成功，则会将`FirebaseVisionLabel` 对象列表传递给成功侦听器(success listener)。每个`FirebaseVisionLabel` 对象都表示图像中标记的内容。对于每个标签，您可以获取标签的文本说明， [知识图谱实体标识ID](https://developers.google.com/knowledge-graph/) （如果可用）以及匹配物体/行为的置信度分数。例如： 

   ```
   for (FirebaseVisionCloudLabel label: labels) {
       String text = label.getLabel();
       String entityId = label.getEntityId();
       float confidence = label.getConfidence();
   }
   ```