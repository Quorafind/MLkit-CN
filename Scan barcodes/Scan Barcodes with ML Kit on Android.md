# 在安卓上使用ML Kit扫描条形码

您可以使用ML kit来识别并且解码条码。

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

3. 可选但建议：如果您使用设备上的API，请将应用配置为在从Play商店安装应用后自动将ML模型下载到设备： 

   ```java
   <meta-data
       android:name="com.google.firebase.ml.vision.DEPENDENCIES"
       android:value="text" />
   <!-- 为了能够使用多个模型: android:value="text,model2,model3" -->
   ```

    如果您未启用安装时模型下载，则将在您首次运行设备上识别器的时候开始下载该模型。在下载完成之前发出的请求不会有任何结果。

## 配置条形码识别器

如果您知道您希望读取哪种条形码格式，可以通过将其配置为仅检测这些格式来提高条形码识别器的识别速度。

例如，假设您只希望识别Aztec和QR码，您可以构建一个 `FirebaseVisionBarcodeDetectorOptions`  对象，如下所示：

```java
FirebaseVisionBarcodeDetectorOptions options =
    new FirebaseVisionBarcodeDetectorOptions.Builder()
        .setBarcodeFormats(FirebaseVisionBarcode.FORMAT_QR_CODE,
                           FirebaseVisionBarcode.FORMAT_AZTEC)
        .build();
```

支持以下格式：

- Code 128 (`FORMAT_CODE_128`)
- Code 39 (`FORMAT_CODE_39`)
- Code 93 (`FORMAT_CODE_93`)
- Codabar (`FORMAT_CODABAR`)
- EAN-13 (`FORMAT_EAN_13`)
- EAN-8 (`FORMAT_EAN_8`)
- ITF (`FORMAT_ITF`)
- UPC-A (`FORMAT_UPC_A`)
- UPC-E (`FORMAT_UPC_E`)
- Data Matrix (`FORMAT_DATA_MATRIX`)
- QR Code (`FORMAT_QR_CODE`)
- PDF417 (`FORMAT_PDF417`)
- Aztec (`FORMAT_AZTEC`)

### 运行人脸识别器

识别图像中的人脸，从任一个`Bitmap`，`media.Image`，`ByteBuffer`或者字节阵列创建一个`FirebaseVisionImage`对象，抑或在设备上的文件中选取。然后，传递`FirebaseVisionImage`对象到 `FirebaseVisionBarcodeDetector`的`detectInImage`方法。 

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
     
     // 或者:
     FirebaseVisionImage image = FirebaseVisionImage.fromByteArray(array, metadata);
     ```

   - 要从文件创建`FirebaseVisionImage`对象，请将应用context和文件URI传递给`FirebaseVisionImage.fromFilePath()`： 

     ```java
     FirebaseVisionImage image = FirebaseVisionImage.fromFilePath(context, uri);
     ```

2. 得到一个 `FirebaseVisionBarcodeDetector`实例:

   ```java
   FirebaseVisionBarcodeDetector detector = FirebaseVision.getInstance()
       .getVisionBarcodeDetector(options);
   ```

3. 最后，将图像传递给`detectInImage`方法： 

   ```java
   Task<List<FirebaseVisionBarcode>> result =
       detector.detectInImage(image, metadata)  // or detectInBuffer(buffer, metadata)
       .addOnSuccessListener(
           new OnSuccessListener<List<FirebaseVisionBarcode>>() {
             @Override
             public void onSuccess(List<FirebaseVisionBarcode> barcodes) {
               // 任务成功
               // ...
             }
           })
       .addOnFailureListener(
           new OnFailureListener() {
             @Override
             public void onFailure(@NonNull Exception e) {
               // 任务失败并且显示异常
               // ...
             }
           });
   ```

## 从条形码获取信息

如果条形码识别操作成功，会将一个`FirebaseVisionBarcode`对象数组 传递给成功侦听器（success listener）。每个`FirebaseVisionBarcode`对象代表图像中检测到的条形码。对于每个条形码，您可以在输入图像中获取其边界坐标以及由条形码编码的原始数据。此外，如果条形码检测器能够确定条形码编码的数据类型，则可以获取包含解析数据的对象。 

例如：

```java
for(int i = 0; i < barcodes.size(); i++) {
  FirebaseVisionBarcode barcode = barcodes.valueAt(i);

  Rect bounds = barcode.getBoundingBox();
  Point[] corners = barcode.getCornerPoints();

  String rawValue = barcode.getRawValue();

  int valueType = barcode.getValueType();
  // 完整的类型支持请看API文档
  switch (valueType) {
    case FirebaseVisionBarcode.TYPE_WIFI:
      String ssid = barcode.getWifi().getSsid();
      String password = barcode.getWifi().getPassword();
      int type = barcode.getWifi().getEncryptionType();
      break;
    case FirebaseVisionBarcode.TYPE_URL:
      String title = barcode.getUrl().getTitle();
      String url = barcode.getUrl().getUrl();
      break;
  }
}
```

