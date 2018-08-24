# 在iOS上使用ML Kit识别地标

您可以使用ML kit来识别图片中的著名地标。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-ios/tree/master/mlkit)上的ML Kit快速入门示例。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLVision'
   ```

​       而后每次您要安装或者升级您的Pods的时候，请确保使用您的Xcode项目的`.xcworkspace`来打开它。

1. 在您的程序中，引入Firebase：

   Swift：

   ```swift
   import Firebase
   ```

   Objective-C：

   ```objective-c
   @import Firebase;
   ```

2. 如果您想使用基于云的模型，并且尚未将项目升级到Blaze计划，请在[Firebase控制台](https://console.firebase.google.com/)中执行此操作。只有Blaze计划的项目才能使用Cloud Vision API。 

3. 如果您想使用基于云的模型，您也需要开启Cloud Vision API：

   - 在云API列表管理平台中打开[Cloud Vision API](https://console.cloud.google.com/apis/library/vision.googleapis.com/)。
   - 确保您的Firebase项目已经在当前菜单页面中被置于顶端。
   - 如果API依旧还是显示为enabled，请点击Enable。

## 配置地标识别器

默认情况下，云识别器使用`STABLE`版本的模型并返回多达10个结果。如果您想要更改这些设置中的任何一个，请使用下例中的`VisionCloudDetectorOptions`对象指定它们： 

Swift：

```swift
let options = VisionCloudDetectorOptions()
options.modelType = .latest
options.maxResults = 20
```

Objective-C：

```objective-c
  FIRVisionCloudDetectorOptions *options =
      [[FIRVisionCloudDetectorOptions alloc] init];
  options.modelType = FIRVisionCloudModelTypeLatest;
  options.maxResults = 20;
```

在下一步中，创建Cloud识别器对象时传递该`VisionCloudDetectorOptions`对象。 

### 运行地标识别器

为了能够识别图像中的地标，将图像传递为`UIImage`或者`CMSampleBufferRef`到`VisionCloudLandmarkDetector`的`detect(in:)`方法：

1. #### 得到一个`VisionCloudLandmarkDetector`实例：

   Swift：

   ```swift
   lazy var vision = Vision.vision()
   let landmarkDetector = vision.cloudLandmarkDetector(options: options)
   // 或者使用默认设定:
   // let landmarkDetector = vision?.cloudLandmarkDetector()
   ```

   Objective-C：

   ```objective-c
   FIRVision *vision = [FIRVision vision];
   FIRVisionCloudLandmarkDetector *landmarkDetector = [vision cloudLandmarkDetector];
   // 或者使用默认设定:
   // FIRVisionCloudLandmarkDetector *landmarkDetector =
   //     [vision cloudLandmarkDetectorWithOptions:options];
   ```

2. #### 使用`UIImage`或者`CMSampleBufferRef`创建一个`VisionImage`对象:

   **使用`UIImage`：**

   1. 如有必要，旋转图像以使其`imageOrientation` 属性为`.up`。

   2. `VisionImage`使用正确旋转的对象创建一个对象 `UIImage`。不要指定任何旋转元数据 - 默认值`.topLeft`，必须使用。

      Swift：

      ```swift
      let image = VisionImage(image: uiImage)
      ```

      Objective-C：

      ```objective-c
      FIRVisionImage *image = [[FIRVisionImage alloc] initWithImage:uiImage];
      ```

   **使用`CMSampleBufferRef`：**

   1. 创建一个`VisionImageMetadata`对象，该对象指定包含在`CMSampleBufferRef`缓冲区中的图像数据的方向 。

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

3. #### 然后，将图像传递给该`detect(in:)`方法：

   Swift：

   ```swift
   landmarkDetector.detect(in: visionImage) { (landmarks, error) in
     guard error == nil, let landmarks = landmarks, !landmarks.isEmpty else {
       // ...
       return
     }
   
     // 识别地标
     // ...
   }
   ```

   Objective-C：

   ```objective-c
   [landmarkDetector detectInImage:image
                        completion:^(NSArray<FIRVisionCloudLandmark *> *landmarks,
                                     NSError *error) {
     if (error != nil) {
       return;
     } else if (landmarks != nil) {
       // 得到地标
     }
   }];
   ```

## 获取著名地标的有关信息

如果地标识别成功，则会将一个`VisionCloudLandmark` 对象数组传递给完成处理程序。从每个对象中，您可以获取在图像中识别的地标的信息。 

例如：

Swift：

```swift
for landmark in landmarks {
  let landmarkDesc = landmark.landmark
  let boundingPoly = landmark.frame
  let entityId = landmark.entityId

  // 一个地标可以有多个定位，例如图片中的地址还有就是所描绘地标的位置。
  for location in landmark.locations {
    let latitude = location.latitude
    let longitude = location.longitude
  }

  let confidence = landmark.confidence
}
```

Objective-C：

```objective-c
for (FIRVisionCloudLandmark *landmark in landmarks) {
   NSString *landmarkDesc = landmark.landmark;
   CGRect frame = landmark.frame;
   NSString *entityId = landmark.entityId;

  // 一个地标可以有多个定位，例如图片中的地址还有就是所描绘地标的位置。
   for (FIRVisionLatitudeLongitude *location in landmark.locations) {
     double latitude = [location.latitude doubleValue];
     double longitude = [location.longitude doubleValue];
   }

   float confidence = [landmark.confidence floatValue];
}
```