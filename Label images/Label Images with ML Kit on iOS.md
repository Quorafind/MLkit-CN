# 在iOS上使用ML Kit标注图片

您可以使用ML Kit来标注图片，使用设备上的模型或云上的模型。请参阅[概述](https://github.com/Quorafind/MLkit-CN/blob/master/ML%20kit%20for%20Firebase.md)以了解每种方法的优点。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-ios/tree/master/mlkit)上的ML Kit快速入门示例，或者尝试使用[codelab](http://g.co/codelabs/mlkit-ios)。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLVision'
   
   # 如果使用设备上的API:
   pod 'Firebase/MLVisionLabelModel'
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

   如果您想要仅仅开启使用设备上的模型，您可以跳过这一步。

现在您已经可以开始使用设备上的模型或者基于云端的模型识别图像中的文本了。

- [在设备上标注图片](#在设备上标注图片)
- [基于云端的图片标注](#基于云端的图片标注)

## 在设备上标注图片

要使用基于云的图像标签模型，请按以下所述配置和运行图像标注器。 

1. ### 配置图像标注器

   默认情况下，设备上的图片标注仅返回置信度为0.5或以上的标签信息。如果您想更改此设置，请按照以下示例创建一个`VisionLabelDetectorOptions`对象：

   Swift：

   ```swift
   let options = VisionLabelDetectorOptions(
     confidenceThreshold: Constants.labelConfidenceThreshold
   )
   ```

   [ViewController.swift](https://github.com/firebase/quickstart-ios/blob/7bd8f039ea86668d8d19eea0b7712c5341e8733b/mlkit/MLKitExample/ViewController.swift#L225-L227)

   Objective-C：

   ```objective-c
   FIRVisionLabelDetectorOptions *options =
       [[FIRVisionLabelDetectorOptions alloc] initWithConfidenceThreshold:0.6f];
   ```

2. ### 运行图片标注器

   1. 为了能够识别图像中的文本，将图像传递为`UIImage`或者`CMSampleBufferRef`到`VisionLabelDetector`的`detect(in:)`方法：

      1. #### 得到一个`VisionTextDetector`实例：

         Swift：

         ```swift
         lazy var vision = Vision.vision()
         let labelDetector = vision.labelDetector(options: options)  // 检测错误
         // 或者使用默认设定:
         // let labelDetector = vision?.labelDetector()
         ```

         Objective-C：

         ```objective-c
         FIRVision *vision = [FIRVision vision];
         FIRVisionLabelDetector *labelDetector = [vision labelDetector];
         // 或者使用默认设定:
         // FIRVisionLabelDetector *labelDetector =
         //     [vision labelDetectorWithOptions:options];
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
         labelDetector.detect(in: visionImage) { (labels, error) in
           guard error == nil, let labels = labels, !labels.isEmpty else {
             // 错误，您需要检查控制台的报错
             // ...
             return
           }
         
           // 被标注过的照片
           // ...
         }
         ```

         Objective-C：

         ```objective-c
         [labelDetector detectInImage:image
                           completion:^(NSArray<FIRVisionLabel *> *labels,
                                        NSError *error) {
           if (error != nil || labels.count == 0) {
             return;
           }
           // 得到标签，从FIRVisionLabel取得标签.
         }];
         ```

3. ### 从标注过的对象中获取信息

   如果图像标注成功，则会将`VisionLabel`对象数组传递给完成处理程序。您可以从每个对象中获取在图像中识别到的特征的有关信息。 

   例如：

   Swift：

   ```
   for label in labels {
     let labelText = label.label
     let entityId = label.entityID
     let confidence = label.confidence
   }
   ```

   Objective-C：

   ```
   for (FIRVisionLabel *label in labels) {
     NSString *labelText = label.label;
     NSString *entityId = label.entityID;
     float confidence = label.confidence;
   }
   ```

## 基于云端的图片标注

要使用基于云端的图像标注模型，请按以下所述，对图像标注器进行配置和运行。

1. ### 配置图像标注器

   默认情况下，云检测器使用`STABLE`版本的模型。如果您想要使用最新版本的模型，请按照以下示例创建一个`VisionCloudDetectorOptions`对象： 

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

   在下一步中，`VisionCloudDetectorOptions`创建Cloud识别器对象时传递该对象。

2. ### 运行图像标注器

   为了识别和标注在图片中的实体，将图像作为`UIImage`或传递`CMSampleBufferRef`给`VisionCloudLabelDetector`的 `detectText(in:)` 方法： 

   1. #### 获取`VisionCloudLabelDetector` 实例：

      Swift：

      ```Swift
      let labelDetector = Vision.vision().cloudLabelDetector()
      // 或者改变默认设定:
      // let labelDetector = Vision.vision().cloudLabelDetector(options: options)
      ```

      Objective-C：

      ```objective-c
      FIRVision *vision = [FIRVision vision];
      FIRVisionCloudLabelDetector *labelDetector = [vision cloudLabelDetector];
      // 或者，修改默认设定:
      // FIRVisionCloudLabelDetector *labelDetector =
      //     [vision cloudLabelDetectorWithOptions:options];
      ```

   2. #### `VisionImage`使用`UIImage`或 创建一个对象`CMSampleBufferRef`。

      **使用`UIImage`：** 

      1. 如有必要，旋转图像以使其`imageOrientation` 属性为`.up`。

      2. `VisionImage`使用正确旋转的对象创建一个对象 `UIImage`。不要指定任何旋转元数据 - 默认值`.topLeft`，必须使用。

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
         metadata.orientation = .rightTop   // Row为0在右边，column为0则是在顶端
         ```

         Objective-C：

         ```
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
      labelDetector.detect(in: visionImage) { (labels: [VisionCloudLabel]?, error: Error?) in
        guard error == nil, let labels = labels, !labels.isEmpty else {
          // ...
      
          return
        }
      
        // 标注后的图片
        // ...
      }
      ```

      Objective-C：

      ```objective-c
      [labelDetector detectInImage:image
                        completion:^(NSArray<FIRVisionCloudLabel *> *labels,
                                     NSError *error) {
        if (error != nil || labels.count == 0) {
          return;
        }
        // 得到标签.将图片标签信息加入到FIRVisionCloudLabel.
      }];
      ```

3. ### 获取对象的标签有关信息

   如果图像标签成功，则会将`VisionCloudLabel`对象数组传递给完成处理程序(completion handler)。您可以从每个对象中获取有关图像中识别的实体的信息。 

   例如：

   Swift：

   ```
   for label in labels {
     let labelText = label.label
     let entityId = label.entityId
     let confidence = label.confidence
   }
   ```

   Objective-C：

   ```
   for (FIRVisionCloudLabel *label in labels) {
     NSString *labelText = label.label;
     NSString *entityId = label.entityId;
     float confidence = [label.confidence floatValue];
   }
   ```