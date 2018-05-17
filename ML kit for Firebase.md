# ML kit for Firebase

在您的应用中使用机器学习来解决真实世界的问题。

ML kit是一种手机平台SDK，是一种能够将谷歌专业的机器学习知识带到应用中的极其简单易用的封装包。无论您是否有机器学习的经验，您都可以在几行代码中实现您想要的功能。甚至，您无需对神经网络或者模型优化有多深入的了解，也能完成您想要做的事情。另一方面，如果您是一位经验丰富的ML开发人员，ML kit甚至提供了便利的API，可帮助您在移动应用中使用自定义的TensorFlow Lit模型。

## 核心功能

| 为工程应用的常见用例     | ML Kit附带了一套用于常见移动用例的随时可用的API：文本识别、人脸识别、地标识别、条形码识别和图像标注等。只需将数据传递给ML kit库，它就能够给您提供您想要的信息。 |
| ------------------------ | ------------------------------------------------------------ |
| 在手机设备上或者云端运行 | ML kit所选用的API可以在设备上运行或者在云上运行。我们的设备上提供的API即使在没有网络连接的情况下也可以快速地处理您的数据。另一方面，我们基于云端的API则是利用了Google  Cloud Platform提供的强大的机器学习功能。可以为您提供更高的准确度。 |
| 装载自有模型             | 而如果ML kit提供的API并不符合您的需求，您可以随时使用您的现有TensorFlow Lite模型。只需要将您的模型上传到Firebase中，我们就会负责托管并将其投放到您的应用当中去。ML kit在这个过程中充当了您的自定义模型的API层，使其更易于运行和使用。 |

## 它怎么运行？

ML kit通过在单个SDK中集成了Google的Cloud Vision API、TensorFlow Lite和Android神经网络API等ML技术，可以轻松地将ML技术应用到您的应用中。无论您是需要基于云处理的强大功能，专为移动设备优化的设备的模型的实时功能，还是定制的TensorFlow Lite模型的灵活性，ML Kit都可以让您通过几行代码实现。

### 有哪些特性在设备上可用抑或在云端可用？

|      特性      | 设备 | 云端 |
| :------------: | :--: | :--: |
|    文本识别    |  √   |  √   |
|    人脸识别    |  √   |  ×   |
|   条形码识别   |  √   |  ×   |
|    图像标记    |  √   |  √   |
|    地标识别    |  ×   |  √   |
| 自定义模型装载 |  √   |  ×   |

## 实现方法

| 整合SDK                  | 通过Gradle或者CocoaPods来快速整合您需要的SDK。               |
| ------------------------ | ------------------------------------------------------------ |
| 准备您的输入数据         | 例如，如果您使用的是视觉功能，请实现相机捕抓图像并生成必要的元数据（如图像旋转）的功能，或者提示用户从其相册中选择一张照片。 |
| 将您的数据提供到ML模型中 | 通过将ML模型应用到您的数据中，您可以生成诸如检测到的面部的表情状态或者图像中识别的对象和概念的洞察（Insight），具体取决于您所使用的功能。使用这些洞察为您的应用中的功能提供支持，如照片点缀（PS），自动元数据生成或者您可以想象的任何功能。 |

## 下一步

- 探索准备好的API：[文字识别](https://github.com/Quorafind/MLkit-CN/blob/master/Recognize%20text/Introduction.md)、[人脸识别](https://github.com/Quorafind/MLkit-CN/blob/master/Detect%20faces/Introduction.md)、[条形码扫描](https://github.com/Quorafind/MLkit-CN/blob/master/Scan%20barcodes/Introduction.md)、[图像标签](https://github.com/Quorafind/MLkit-CN/blob/master/Label%20images/Image%20Labeling.md)以及[地标识别](https://github.com/Quorafind/MLkit-CN/blob/master/Recognize%20landmarks/Landmark%20Recognition.md)。
- 关于如何在您的个人应用中使用您的[个性化化模型](https://github.com/Quorafind/MLkit-CN/blob/master/Use%20a%20custom%20model/Custom%20Models.md)。
- 看一下在github上的[iOS](https://github.com/firebase/quickstart-ios/tree/master/mlkit)和[Android](https://github.com/firebase/quickstart-android/tree/master/mlkit)实例。
- 为[iOS](http://g.co/codelabs/mlkit-ios)或者[Android](http://g.co/codelabs/mlkit-android)的codeLabs。
