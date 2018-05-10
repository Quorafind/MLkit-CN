# 在iOS上使用ML Kit识别人脸

你可以在iOS上使用ML Kit来在图像和视频中识别人脸。

请参阅GitHub上的[ML Kit快速入门示例](https://github.com/firebase/quickstart-ios/tree/master/mlkit)，了解正在使用的此API的示例。 

## 在开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLVision'
   pod 'Firebase/MLVisionFaceModel'
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

4.  从项目设置的**Build Settings > Build Options** 部分禁用项目的Bitcode生成。 

## 在设备上识别人脸

### 设置人脸识别器

在您对图像进行人脸识别以前，如果您想要修改任何关于人脸识别的默认设定，请通过`VisionFaceDetectorOptions` 对象来指定这些设定。您可以修改以下设定：

| 设置         | 参数 | 参数2 | 解释 |
| ------------ | -------------------------------- | ---------------------------------------------------------- | ------------ |
| 识别模式     | `fast` (默认“快速”) | `accurate` | 取决于您在人脸识别中是喜欢速度或准确性。 |
| 识别特征点   | `none` (默认“无”) | `all`        | 是否尝试识别面部“特征点”：眼睛，耳朵，鼻子，脸颊，嘴巴。 |
| 辨认脸部     | `none` (默认“无”) | `all`        | 是否将人脸分类为“微笑”和“睁眼”等类别。 |
| 脸部最小识别 | `CGFloat` (默认： `0.1`)         |                        | 要检测的脸部相对于图像的最小尺寸。 |
| 允许人脸追踪 | `false` (默认) | `true`          | 是否分配人脸ID，可用于跟踪图像中的人脸。 |

例如，为了修改以上所有的默认设定，在下边的例子中构建一个 `VisionFaceDetectorOptions`  对象：

Swift：

```swift
let options = VisionFaceDetectorOptions()
options.modeType = .accurate
options.landmarkType = .all
options.classificationType = .all
options.minFaceSize = CGFloat(0.1)
options.isTrackingEnabled = true
```

Objective-C：

```objective-c
FIRVisionFaceDetectorOptions *options = [[FIRVisionFaceDetectorOptions alloc] init];
options.modeType = FIRVisionFaceDetectorModeAccurate;
options.landmarkType =  FIRVisionFaceDetectorLandmarkAll;
options.classificationType = FIRVisionFaceDetectorClassificationAll;
options.minFaceSize = (CGFloat) 0.2f;
options.isTrackingEnabled = YES;
```

### 运行人脸识别器

为了在图片中识别人脸，需要将照片作为 `UIImage` 或者 `CMSampleBufferRef` 传递到 `VisionFaceDetector`的 `detect(in:)`方法: 

1. `VisionFaceDetector` 的一个例子:

   Swift：

   ```swift
   lazy var vision = Vision.vision()
   let faceDetector = vision.faceDetector(options: options)  // Check console for errors.
   // 或者使用以下默认设定:
   // let faceDetector = vision?.faceDetector()
   ```

   Objective-C：

   ```Objective-C
   FIRVision *vision = [FIRVision vision];
   FIRVisionFaceDetector *detector = [vision faceDetector];
   // 或者为了修改默认设定:
   // FIRVisionFaceDetector *detector =
   //     [vision faceDetectorWithOptions:options];
   ```

2. 使用`UIImage` 或者 `CMSampleBufferRef`创建 `VisionImage` 对象。

   使用`UIImage` :

   1. 如果有必要，旋转图像，使得它的`imageOrientation`的属性是`.up`。

   2. `VisionImage`使用正确旋转的对象创建一个对象 `UIImage`。不要指定任何旋转元数据 - 默认值`.topLeft`，必须使用。 

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

3. 然后，将图像传递给该`detect(in:)`方法：

   Swift：

   ```swift
   faceDetector.detect(in: visionImage) { (faces, error) in
     guard error == nil, let faces = faces, !faces.isEmpty else {
       // 错误，您应该检查控制台。
       // ...
       return
     }
   
     // 脸部识别
     // ...
   }
   ```

   Objective-C：

   ```objective-c
   [detector detectInImage:image
                completion:^(NSArray<FIRVisionFace *> *faces,
                             NSError *error) {
                  if (error != nil) {
                    return;
                  } else if (faces != nil) {
                    // 识别人脸
                  }
                }];
   ```

### 获取检测到的面部有关信息

如果人脸检测操作成功，则人脸识别器将一组`VisionFace`对象传递给完成处理程序（completion handler）。每个 `VisionFace`对象代表在图像中检测到的脸部。对于每个人脸，您可以在输入图像中获得其边界坐标以及您配置人脸识别器查找的任何其他信息。例如：

Swift：

```swift
for face in faces {
  let frame = face.frame
  if face.hasHeadEulerAngleY {
    let rotY = face.headEulerAngleY  // 头部转向右rotY角
  }
  if face.hasHeadEulerAngleZ {
    let rotZ = face.headEulerAngleZ  // 头部转向上rotZ角
  }

  // 如果特征点识别开启了（嘴，耳，眼，脸颊，还有鼻子可以检测）：
  if let leftEye = face.landmark(ofType: .leftEye) {
    let leftEyePosition = leftEye.position
  }

  // 如果辨认开启了：
  if face.hasSmilingProbability {
    let smileProb = face.smilingProbability
  }
  if face.hasRightEyeOpenProbability {
    let rightEyeOpenProb = face.rightEyeOpenProbability
  }

  // 如果脸部追踪开启了：
  if face.hasTrackingID {
    let trackingId = face.trackingID
  }
}
```

 Objective-C：

```objective-c
for (FIRVisionFace *face in faces) {
  // 图像中脸的边界
  CGRect frame = face.frame;

  if (face.hasHeadEulerAngleY) {
    CGFloat rotY = face.headEulerAngleY;  // 头部转向右rotY角
  }
  if (face.hasHeadEulerAngleZ) {
    CGFloat rotZ = face.headEulerAngleZ;  // 头部转向上rotZ角
  }

  // 如果特征点识别开启了（嘴，耳，眼，脸颊，还有鼻子可以检测）
  FIRVisionFaceLandmark *leftEar = [face landmarkOfType:FIRFaceLandmarkTypeLeftEar];
  if (leftEar != nil) {
    FIRVisionPoint *leftEarPosition = leftEar.position;
  }

  // 如果辨认开启了
  if (face.hasSmilingProbability) {
    CGFloat smileProb = face.smilingProbability;
  }
  if (face.hasRightEyeOpenProbability) {
    CGFloat rightEyeOpenProb = face.rightEyeOpenProbability;
  }

  // 如果脸部追踪开启了
  if (face.hasTrackingID) {
    NSInteger trackingID = face.trackingID;
  }
}
```