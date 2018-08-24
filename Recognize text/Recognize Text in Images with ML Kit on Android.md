# 在安卓上使用ML Kit识别图像中的文本

您可以使用ML Kit来识别图像中的文本，使用设备上的模型或云上的模型。请参阅[概述](https://github.com/Quorafind/MLkit-CN/blob/master/ML%20kit%20for%20Firebase.md)以了解每种方法的优点。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-android/tree/master/mlkit)上的ML Kit快速入门示例，或者尝试使用[codelab](http://g.co/codelabs/mlkit-android)。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/android/setup)来开始您的工作。

2. 在app-level的`build.gradle` 文件中为ML kit添加依赖:

   ```java
   dependencies {
     // ...
   
     implementation 'com.google.firebase:firebase-ml-vision:15.0.0'
   }
   ```

3.  可选但建议：如果您使用设备上的API，请将应用配置为在从Play商店安装应用后自动将ML模型下载到设备： 

   ```java
   <meta-data
       android:name="com.google.firebase.ml.vision.DEPENDENCIES"
       android:value="text" />
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

- [在设备上识别文本](#在设备上识别文本)
- [基于云端的文本识别](#基于云端的文本识别)

## 在设备上识别文本

为了使用设备上的文本识别模型，请运行如下文本识别器。

1. ### 运行文本识别器

   为了识别图像中的文本，从任一个`Bitmap`，`media.Image`，`ByteBuffer`，字节阵列，或在设备上的文件中创建一个`FirebaseVisionImage`对象。然后，传递`FirebaseVisionImage`对象到`FirebaseVisionTextDetector`的`detectInImage`方法。

   1. 从图片中创建`FirebaseVisionImage`对象. 

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
        private int getRotationCompensation(Activity activity, String cameraId)
                throws CameraAccessException {
            // 得到设备当前与原始的角度的旋转差值
            // 然后照片一定要旋转回去相对的差值
            int deviceRotation = activity.getWindowManager().getDefaultDisplay().getRotation();
            int rotationCompensation = ORIENTATIONS.get(deviceRotation);
        
            // 在大多数的设备上，传感器的方向是90度。但是对于
            // 少数设备，这个值是270度。那么对于这些270度的设备
            // 必须让照片旋转额外的180 ((270 + 270) % 360) 度.
            CameraManager cameraManager = (CameraManager) getSystemService(CAMERA_SERVICE);
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
        
        ...
        
        // 得到正在使用CameraManager的camera的ID。随后:
        int rotation = getRotationCompensation(this, MY_CAMERA_ID);
        ```

        然后，将`media.Image`对象和旋转值传递给`FirebaseVisionImage.fromMediaImage()`： 

        ```java
        FirebaseVisionImage image = FirebaseVisionImage.fromMediaImage(mediaImage,rotation);
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
      FirebaseVisionTextDetector detector = FirebaseVision.getInstance()
          .getVisionTextDetector();
      ```

   3. 最后，将图像传递给`detectInImage`方法： 

      ```java
      Task<FirebaseVisionText> result =
              detector.detectInImage(image)
                      .addOnSuccessListener(new OnSuccessListener<FirebaseVisionText>() {
                          @Override
                          public void onSuccess(FirebaseVisionText firebaseVisionText) {
                              // 任务成功完成
                              // ...
                          }
                      })
                      .addOnFailureListener(
                              new OnFailureListener() {
                                  @Override
                                  public void onFailure(@NonNull Exception e) {
                                      // 任务失败并且报异常
                                      // ...
                                  }
                              });
      ```

2. ### 从识别的文本块中提取文本

   如果文本识别操作成功，程序则会将`FirebaseVisionTextBlock`的数组传递给成功侦听器（success listener）。每个`FirebaseVisionTextBlock`对象代表在图像中识别到的文本的矩形块（例如一段打印文本）。对于每一个`FirebaseVisionTextBlock`，您都可以获取块的边界坐标和块中包含的文本。另外，对于`FirebaseVisionTextBlock`，您都可以获得组成该块的文本行，以及构成每行文本的元素（如字符或标点符号）： 

   ```java
   for (FirebaseVisionText.Block block: firebaseVisionText.getBlocks()) {
       Rect boundingBox = block.getBoundingBox();
       Point[] cornerPoints = block.getCornerPoints();
       String text = block.getText();
   
       for (FirebaseVisionText.Line line: block.getLines()) {
           // ...
           for (FirebaseVisionText.Element element: line.getElements()) {
               // ...
           }
       }
   }
   ```


## 基于云端的文本识别

要使用基于云的文本识别模型，请配置并运行文本检测器，如下所述。 

1. ### 配置文本检测器

   默认情况下，云识别器会使用模型的稳定版本并返回多达10个结果。如果要更改这些设置中的任何一个，请用`FirebaseVisionCloudDetectorOptions` 对象指定它们。

   例如，要更改这些默认设置，按照以下示例构建一个`FirebaseVisionCloudDetectorOptions`对象：

   ```java
   FirebaseVisionCloudDetectorOptions options =
       new FirebaseVisionCloudDetectorOptions.Builder()
           .setModelType(FirebaseVisionCloudDetectorOptions.LATEST_MODEL)
           .setMaxResults(15)
           .build();
   ```

   要使用默认设置，可以在下一步中使用`FirebaseVisionCloudDetectorOptions.DEFAULT`。 

2. ### 运行文本检测器

   为了识别图像中的文本，从任一个`Bitmap`，`media.Image`，`ByteBuffer`或者字节阵列创建一个`FirebaseVisionImage`对象，抑或在设备上的文件中选取一个。然后，传递`FirebaseVisionImage`对象到无论是`FirebaseVisionCloudTextDetector`或（如果图像是一个文件的话）`FirebaseVisionCloudDocumentTextDetector`的`detectInImage` 方法。 

   1. 从您的图像中创建一个`FirebaseVisionImage`对象。

      - 从 `Bitmap`对象创建`FirebaseVisionImage`对象：

        ```java
        FirebaseVisionImage image = FirebaseVisionImage.fromBitmap(bitmap);
        ```

        `Bitmap`物体表示的图像必须是直立的，不需要旋转。 

      - 要从`media.Image`对象创建`FirebaseVisionImage`对象（例如从设备的相机捕捉图像时），首先要确定图像必须旋转的角度，以补偿设备的旋转和相机传感器在设备中的方向差：

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
        
        
            // 返回相关的FirebaseVisionImageMetadata的旋转值。
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
        FirebaseVisionImage image = FirebaseVisionImage.fromMediaImage(mediaImage, rotation);
        ```

      - 要从一个字节数组或`ByteBuffer`创建一个`FirebaseVisionImage`对象 ，首先按照上面的描述计算图像旋转。

        然后，创建一个包含图像高度，宽度，颜色编码格式和旋转的`FirebaseVisionImageMetadata`对象：

        ```java
        FirebaseVisionImageMetadata metadata = new FirebaseVisionImageMetadata.Builder()
                .setWidth(1280)
                .setHeight(720)
                .setFormat(FirebaseVisionImageMetadata.IMAGE_FORMAT_NV21)
                .setRotation(rotation)
                .build();
        ```

        使用`buffer`或数组以及元数据对象来创建一个 `FirebaseVisionImage`对象： 

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

   2. 获取`FirebaseVisionCloudTextDetector`或`FirebaseVisionCloudDocumentTextDetector`的一个实例： 

      ```java
      FirebaseVisionCloudTextDetector detector = FirebaseVision.getInstance()
              .getVisionCloudTextDetector();
      // 或者，修改默认设定:
      // FirebaseVisionCloudTextDetector detector = FirebaseVision.getInstance()
      //         .getVisionCloudTextDetector(options);
      ```

   3. 最后，将图像传递给`detectInImage`方法： 

      ```java
      Task<FirebaseVisionCloudText> result = detector.detectInImage(image)
              .addOnSuccessListener(new OnSuccessListener<FirebaseVisionCloudText>() {
                  @Override
                  public void onSuccess(FirebaseVisionCloudText firebaseVisionCloudText) {
                      // 任务成功完成
                      // ...
                  }
              })
              .addOnFailureListener(new OnFailureListener() {
                  @Override
                  public void onFailure(@NonNull Exception e) {
                      // 任务失败并发出异常
                      // ...
                  }
              });
      ```

### 3.从识别的文本块中提取文本

如果文本识别操作成功，则会将`FirebaseVisionCloudText`对象传递给成功侦听器（success listener）。该对象包含图像中识别的文本。 

您还可以获取有关文本结构的信息。文本被组织成页面，块，段落，单词和符号。对于组织成的每个单位，您都可以获取其详细信息，例如其维度及其包含的语言。 

例如： 

```java
String recognizedText = firebaseVisionCloudText.getText();

for (FirebaseVisionCloudText.Page page: firebaseVisionCloudText.getPages()) {
    List<FirebaseVisionCloudText.DetectedLanguage> languages =
            page.getTextProperty().getDetectedLanguages();
    int height = page.getHeight();
    int width = page.getWidth();
    float confidence = page.getConfidence();

    for (FirebaseVisionCloudText.Block block: page.getBlocks()) {
        Rect boundingBox = block.getBoundingBox();
        List<FirebaseVisionCloudText.DetectedLanguage> blockLanguages =
                block.getTextProperty().getDetectedLanguages();
        float blockConfidence = block.getConfidence();
        // 还有更多: 文段, 词语, 符号
    }
}
```

