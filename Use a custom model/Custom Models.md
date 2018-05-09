# 自定义模型

如果您是一位经验丰富的ML开发人员，而且ML Kit的预训练的模型不能满足您的需求，您可以通过ML Kit使用定 的[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)模型。

使用Firebase托管您的TensorFlow Lite模型或将其与您的应用程序打包在一起。然后，使用ML Kit SDK来使用您的自定义模型的最佳版本构建应用。如果您使用Firebase托管您的模型，ML Kit会自动更新您的用户的所用版本。

[iOS](https://github.com/Quorafind/MLkit-CN/blob/master/Use%20a%20custom%20model/Use%20a%20TensorFlow%20Lite%20model%20for%20inference%20with%20ML%20Kit%20on%20iOS.md) [Android]()

## 核心功能

| TensorFlow Lite模型托管 | 使用Firebase托管您的模型，以减少应用程序的大小，并确保您的应用程序始终使用您的模型的最新版本 |
| ----------------------- | ------------------------------------------------------------ |
| 设备端的ML推断          | 通过使用ML Kit SDK运行您的自定义TensorFlow Lite模型，在iOS或Android应用程序中执行推理。该模型可以与云中托管的应用程序捆绑在一起，或者两者兼而有之。 |
| 自动模型后备            | 指定多个模型来源; 当云托管模型不可用时，请使用本地存储的模型 |
| 自动模型更新            | 配置您的应用程序自动下载新版本模型的条件：用户设备空闲时，正在充电或具有Wi-Fi连接 |

## 实现步骤

| **训练您的TensorFlow模型**                  | 使用TensorFlow构建和训练定制模型。或者，重新训练现有模型，解决类似于您想要实现的问题。请参阅TensorFlow Lite [开发人员指南](https://www.tensorflow.org/mobile/tflite/devguide)。 |
| ------------------------------------------- | ------------------------------------------------------------ |
| **将模型转换为TensorFlow Lite**             | 使用TensorFlow优化转换器（TOCO），通过freezing gragh的办法。将模型从标准TensorFlow格式转换为TensorFlow Lite格式。请参阅TensorFlow Lite [开发人员指南](https://www.tensorflow.org/mobile/tflite/devguide)。 |
| **使用Firebase托管您的TensorFlow Lite模型** | 可选：当您使用Firebase托管您的TensorFlow Lite模型并将ML Kit SDK包含在您的应用程序中时，ML Kit可让您的用户随时了解最新版本的模型。您可以将ML Kit配置为在用户的设备闲置或充电或具有Wi-Fi连接时自动下载型号更新。 |
| **使用TensorFlow Lite模型进行推断**         | 在您的iOS或Android应用中使用ML Kit的自定义模型API来执行Firebase托管或应用捆绑的模型的推理。 |