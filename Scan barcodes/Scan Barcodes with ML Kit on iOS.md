# 在iOS上使用ML Kit扫描条形码

您可以使用ML kit来识别并且解码条码。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-ios/tree/master/mlkit)上的ML Kit快速入门示例。

## 在开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLVision'
   pod 'Firebase/MLVisionBarcodeModel'
   ```

​       而后每次您要安装或者升级您的Pods的时候，请确保使用您的Xcode项目的`.xcworkspace`来打开它。

3. 在您的程序中，引入Firebase：

   Swift：

   ```swift
   import Firebase
   ```

   Objective-C：

   ```objective-c
   @import Firebase;
   ```

## 配置条形码识别器

如果您知道您希望读取哪种条形码格式，可以通过将其配置为仅检测这些格式来提高条形码识别器的识别速度。

 Swift：

例如，假设只希望识别Aztec和QR码，您可以构建一个`VisionBarcodeDetectorOptions` 对象，如下所示：

```Swift
let format = VisionBarcodeFormat.all
// 或者, e.g.: VisionBarcodeFormat.qrCode | VisionBarcodeFormat.aztec
let options = VisionBarcodeDetectorOptions(formats: format)
```

Objective-C：

例如，假设只希望识别Aztec和QR码，您可以构建一个 `FIRVisionBarcodeDetectorOptions`   对象，如下所示：

```objective-c
FIRVisionBarcodeDetectorOptions *options =
    [[FIRVisionBarcodeDetectorOptions alloc]
     initWithFormats: FIRVisionBarcodeFormatQRCode | FIRVisionBarcodeFormatAztec];
```

支持以下格式：

Swift：

- Code128
- Code39
- Code93
- CodaBar
- EAN13
- EAN8
- ITF
- UPCA
- UPCE
- DataMatrix
- QRCode
- PDF417
- Aztec

Objective-C：

- Code 128 (`FIRVisionBarcodeFormatCode128`)
- Code 39 (`FIRVisionBarcodeFormatCode39`)
- Code 93 (`FIRVisionBarcodeFormatCode93`)
- Codabar (`FIRVisionBarcodeFormatCodaBar`)
- EAN-13 (`FIRVisionBarcodeFormatEAN13`)
- EAN-8 (`FIRVisionBarcodeFormatEAN8`)
- ITF (`FIRVisionBarcodeFormatITF`)
- UPC-A (`FIRVisionBarcodeFormatUPCA`)
- UPC-E (`FIRVisionBarcodeFormatUPCE`)
- Data Matrix (`FIRVisionBarcodeFormatDataMatrix`)
- QR Code (`FIRVisionBarcodeFormatQRCode`)
- PDF417 (`FIRVisionBarcodeFormatPDF417`)
- Aztec (`FIRVisionBarcodeFormatAztec`)

## 运行条形码识别器

为了能够识别图像中的条形码，将图像传递为`UIImage`或者`CMSampleBufferRef`到`VisionBarcodeDetector`的`detect(in:)`方法：

1. ### 得到一个`VisionTextDetector`实例：

   Swift：

   ```Swift
   lazy var vision = Vision.vision()
   let barcodeDetector = vision.barcodeDetector(options: options)  // 检查错误。
   // 或者使用默认设置:
   // let barcodeDetector = vision?.barcodeDetector()
   ```

   Objective-C：

   ```objective-c
   FIRVision *vision = [FIRVision vision];
   FIRVisionBarcodeDetector *detector = [vision barcodeDetector];
   // 或者修改默认设置:
   // FIRVisionBarcodeDetector *detector =
   //     [vision barcodeDetectorWithOptions:options];
   ```

2. ### 使用`UIImage` 或者 `CMSampleBufferRef`创建 `VisionImage` 对象。

   使用`UIImage` :

   - 如果有必要，旋转图像，使得它的`imageOrientation`的属性是`.up`。

   - `VisionImage`使用正确旋转的对象创建一个对象 `UIImage`。不要指定任何旋转元数据 - 默认值`.topLeft`，必须使用。 

     Swift：

     ```swift
     let image = VisionImage(image: uiImage)
     ```

     Objective-C：

     ```objective-c
     FIRVisionImage *image = [[FIRVisionImage alloc] initWithImage:uiImage];
     ```

   使用`CMSampleBufferRef`： 

   1. 创建一个`VisionImageMetadata`对象，该对象指定包含在`CMSampleBufferRef`缓冲区中的图像数据的方向 。 

      例如，如果图像数据必须顺时针旋转90度才能保持直立： 

      Swift：

      ```swift
      let metadata = VisionImageMetadata()
      metadata.orientation = .rightTop  // Row为0在右边，column为0则是在顶端
      ```

      Objective-C：

      ```objective-c
      // Row为0在右边，column为0则是在顶端
      FIRVisionImageMetadata *metadata = [[FIRVisionImageMetadata alloc] init];
      metadata.orientation = FIRVisionDetectorImageOrientationRightTop;
      ```

   2. `VisionImage`使用`CMSampleBufferRef`对象和旋转元数据创建一个对象 ： 

      Swift：

      ```swift
      let image = VisionImage(buffer: bufferRef)
      image.metadata = metadata
      ```

      Objective-C：

      ```objective-c
      FIRVisionImage *image = [[FIRVisionImage alloc] initWithBuffer:buffer];
      image.metadata = metadata;
      ```

3. ### 然后，将图像传递给该`detect(in:)`方法：

   Swift：

   ```swift
   barcodeDetector.detect(in: visionImage) { (barcodes, error) in
     guard error == nil, let barcodes = barcodes, !barcodes.isEmpty else {
       // 错误，您应该检查控制台。
       // ...
       return
     }
   
     // 识别并且解读条形码
     // ...
   }
   ```

   Objective-C：

   ```objective-c
   [detector detectInImage:image
                completion:^(NSArray<FIRVisionBarcode *> *barcodes,
                             NSError *error) {
                  if (error != nil) {
                    return;
                  } else if (barcodes != nil) {
                    // 识别条形码
                    // ...
                  }
                }];
   ```

## 从条形码获取信息

如果条形码识别操作成功，则识别器将返回一个`VisionBarcode`对象数组 。每个`VisionBarcode`对象代表图像中检测到的条形码。对于每个条形码，您可以在输入图像中获取其边界坐标以及由条形码编码的原始数据。此外，如果条形码识别器能够确定条形码编码的数据类型，则可以获取包含解析数据的对象。 

Swift：

```Swift
for barcode in barcodes {
  let corners = barcode.cornerPoints

  let displayValue = barcode.displayValue
  let rawValue = barcode.rawValue

  let valueType = barcode.valueType
  switch valueType {
  case .wiFi:
    let ssid = barcode.wifi!.ssid
    let password = barcode.wifi!.password
    let encryptionType = barcode.wifi!.type
  case .URL:
    let title = barcode.url!.title
    let url = barcode.url!.url
  default:
    // 所有支持的数据类型请查看API列表
  }
}
```

Objective-C：

```objective-c
 for (FIRVisionBarcode *barcode in barcodes) {
   NSArray *corners = barcode.cornerPoints;

   NSString *displayValue = barcode.displayValue;
   NSString *rawValue = barcode.rawValue;

   FIRVisionBarcodeValueType valueType = barcode.valueType;
   switch (valueType) {
     case FIRVisionBarcodeValueTypeWiFi:
       // ssid = barcode.wifi.ssid;
       // password = barcode.wifi.password;
       // encryptionType = barcode.wifi.type;
       break;
     case FIRVisionBarcodeValueTypeURL:
       // url = barcode.URL.url;
       // title = barcode.URL.title;
       break;
     // ...
     default:
       break;
   }
 }
```