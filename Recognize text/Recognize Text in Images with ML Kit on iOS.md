# 在iOS上使用ML Kit识别图像中的文本

您可以使用ML Kit来识别图像中的文本，使用设备上的模型或云上的模型。请参阅[概述](https://github.com/Quorafind/MLkit-CN/blob/master/ML%20kit%20for%20Firebase.md)以了解每种方法的优点。

有关此API使用的示例，请参阅[GitHub](https://github.com/firebase/quickstart-ios/tree/master/mlkit)上的ML Kit快速入门示例，或者尝试使用[codelab](http://g.co/codelabs/mlkit-ios)。

## 在您开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLVision'
   
   # If using the on-device API:
   pod 'Firebase/MLVisionTextModel'
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

4.  如果您想使用基于云的模型，并且尚未将项目升级到Blaze计划，请在[Firebase控制台](https://console.firebase.google.com/)中执行此操作。只有Blaze计划的项目才能使用Cloud Vision API。 

5. 如果您想使用基于云的模型，您也需要开启Cloud Vision API：

   + 在云API列表管理平台中打开[Cloud Vision API](https://console.cloud.google.com/apis/library/vision.googleapis.com/)。
   + 确保您的Firebase项目已经在当前菜单页面中被置于顶端。
   + 如果API依旧还是显示为enabled，请点击Enable。

   如果您想要仅仅开启使用设备上的模型，您可以跳过这一步。

现在您已经可以开始使用设备上的模型或者基于云端的模型识别图像中的文本了。

- [在设备上识别文本](#在设备上识别文本)
- [基于云端的文本识别](#基于云端的文本识别)

## 在设备上识别文本

为了使用设备上的文本识别模型，请运行如下文本识别器。

1. ### 运行文本识别器

   为了能够识别图像中的文本，将图像传递为`UIImage`或者`CMSampleBufferRef`到`VisionTextDetector`的`detect(in:)`方法：

   1. #### 得到一个`VisionTextDetector`实例：

      Swift：

      ```swift
      lazy var vision = Vision.vision()
      let textDetector = vision.textDetector()  // 检测错误.
      ```

      Objective-C：

      ```objective-c
      FIRVision *vision = [FIRVision vision];
      FIRVisionTextDetector *detector = [vision textDetector];
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
      textDetector.detect(in: visionImage) { (features, error) in
        guard error == nil, let features = features, !features.isEmpty else {
          //错误，您应该检查控制台来寻找错误
          // ...
          return
        }
      
        // 识别并且输出文本
        print("Detected text has: \(features.count) blocks")
        // ...
      }
      ```

      Objective-C：

      ```objective-c
      [detector detectInImage:image
                   completion:^(NSArray<FIRVisionText *> *features,
                                NSError *error) {
        if (error != nil) {
          return;
        } else if (features != nil) {
          // Recognized text
        }
      }];
      ```

2. ### 从识别的文本块中提取文本

   如果文本识别操作成功，它将返回一个`VisionText`对象数组 。每个`VisionText`对象都代表一个矩形的文本块，一行文本或一个文本单词。

   对于每一个`VisionText`，您都可以获取块的边界坐标和块中包含的文本：

   Swift：

   ```swift
   for feature in features {
     let value = feature.text
     let corners = feature.cornerPoints
   }
   ```

   Objective-C：

   ```objective-c
   for (id <FIRVisionText> feature in features) {
     NSString *value = feature.text;
     NSArray<NSValue *> *corners = feature.cornerPoints;
   }
   ```

   另外，如果`VisionText`，是`VisionTextBlock`，你可以得到构成块的文本行，如果是 `VisionTextLine`，你可以得到组成每行文本的元素： 

   Swift：

   ```swift
   // 内含文本列的块
   if let block = feature as? VisionTextBlock {
     for line in block.lines {
       // ...
       for element in line.elements {
         // ...
       }
     }
   }
   
   // 包含文本元素的Lines
   else if let line = feature as? VisionTextLine {
     for element in line.elements {
       // ...
     }
   }
   
   // 文本元素都是可输出文本
   else if let element = feature as? VisionTextElement {
     // ...
   }
   ```

   Objective-C：

   ```objective-c
   // 内含文本列的块
   if ([feature isKindOfClass:[FIRVisionTextBlock class]]) {
    FIRVisionTextBlock *block = (FIRVisionTextBlock *)feature;
    for (FIRVisionTextLine *line in block.lines) {
      // ...
      for (FIRVisionTextElement *element in line.elements) {
        // ...
      }
    }
   }
   
   // 包含文本元素的Lines
   else if ([feature isKindOfClass:[FIRVisionTextLine class]]) {
    FIRVisionTextLine *line = (FIRVisionTextLine *)feature;
    for (FIRVisionTextElement *element in line.elements) {
      // ...
    }
   }
   
   // 文本元素都是可输出文本
   else if ([feature isKindOfClass:[FIRVisionTextElement class]]) {
    // ...
   }
   ```

   ## 基于云端的文本识别

要使用基于云的文本识别模型，请配置并运行文本识别器，如下所述。 

>  ML Kit的基于云的API目前仅可用于预览。对于投入生产的基于云的API，请考虑直接使用Cloud Vision API。 

1. ### 配置文本识别器

   默认情况下，云检测器使用稳定版本的模型。如果您想要使用最新版本的模型，请按照以下示例创建一个`VisionCloudDetectorOptions`对象： 

   Swift：

   ```swift
   let options = VisionCloudDetectorOptions()
   options.modelType = .latest
   // options.maxResults 对此API没有任何影响
   ```

   Objective-C：

   ```objective-c
   FIRVisionCloudDetectorOptions *options =
       [[FIRVisionCloudDetectorOptions alloc] init];
   options.modelType = FIRVisionCloudModelTypeLatest;
   ```

   在下一步中，`VisionCloudDetectorOptions`创建Cloud识别器对象时传递该对象。 

2. ### 运行文本识别器

   要识别图像中的文本，请创建一个`VisionCloudTextDetector`对象，或者，如果图像是文档，则为`VisionCloudDocumentTextDetector`对象。然后，将图像作为`UIImage`或传递`CMSampleBufferRef`给该`detectText(in:)` 方法： 

   1. #### 获取`VisionCloudTextDetector`或`VisionCloudDocumentTextDetector`的实例： 

      Swift：

      ```Swift
      lazy var vision = Vision.vision()
      let textDetector = vision.cloudTextDetector(options: options)
      // 或者用默认设置:
      // let textDetector = vision?.cloudTextDetector()
      ```

      Objective-C：

      ```objective-c
      FIRVision *vision = [FIRVision vision];
      FIRVisionCloudTextDetector *detector = [vision cloudTextDetector];
      // 或者更改默认设置:
      // FIRVisionCloudTextDetector *detector =
      //     [vision cloudTextDetectorWithOptions:options];
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
      textDetector.detect(in: visionImage) { (cloudText, error) in
        guard error == nil, let cloudText = cloudText else {
          // ...
          return
        }
      
        // 识别并且输出文本
        // ...
      }
      ```

      Objective-C：

      ```objective-c
      [detector detectInImage:image
                   completion:^(FIRVisionCloudText *cloudText,
                                NSError *error) {
                     if (error != nil) {
                       return;
                     } else if (cloudText != nil) {
                       // Recognized text
                     }
                   }];
      ```

3. ### 从识别的文本块中提取文本

   如果文本识别操作成功，则会将`VisionCloudText`对象传递给成功侦听器（Success Listener）。该对象包含图像中识别的文本。

   例如：

   Swift

   ```swift
   let recognizedText = cloudText.text
   ```

   Objective-C：

   ```objective-c
   NSString *recognizedText = cloudText.text;
   ```

   您还可以获取有关文本结构的信息。文本被组织成页面，块，段落，单词和符号。对于组织的每个单位，您都可以获取信息，例如其维度及其包含的语言。 

   例如： 

   Swift：

   ```swift
   for page in cloudText.pages {
     let width = page.width
     let height = page.height
     let langs = page.textProperty?.detectedLanguages
     if let blocks = page.blocks {
       for block in blocks {
         let blockFrame = block.frame
       }
     }
   }
   ```

   Objective-C：

   ```objective-c
   for (FIRVisionCloudPage *page in cloudText.pages) {
    int width = [page.width intValue];
    int height = [page.height intValue];
    NSArray<FIRVisionCloudDetectedLanguage *> *langs = page.textProperty.detectedLanguages;
    for (FIRVisionCloudBlock *block in page.blocks) {
      CGRect frame = block.frame;
      // 等等.
    }
   }
   ```



