# 在iOS上通过ML Kit使用TensorFlow Lite模型进行推理

您可以使用ML Kit配合[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)模型执行在设备上的推理 。

ML Kit只能在运行iOS 9或更新版本的设备上使用TensorFlow Lite模型。

请参阅GitHub上的[ML Kit快速入门示例](https://github.com/firebase/quickstart-ios/tree/master/mlkit)，了解正在使用的此API的示例。

## 在开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 将ML kit库放进您的Podfile中：

   ```objc
   pod 'Firebase/Core'
   pod 'Firebase/MLModelInterpreter'
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

4. 转换您想要的TensorFlow模型为TensorFlow Lite(tflite)格式。请查看[TOCO：TensorFlow Lite 优化转换器](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite/toco)

## 托管或捆绑您的模型

在您的应用程序中使用TensorFlow Lite模型进行推理之前，您必须将该模型提供给ML Kit。ML Kit可以使用Firebase远程托管的，本地存储在设备上的或两者兼而有之的TensorFlow Lite模型。

通过在Firebase上托管模型并在本地支持，您可以确保在模型可用时使用最新版本的模型，但是当Firebase托管的模型不可用时，您的应用程序的ML功能仍可使用。

### 模型安全

无论您如何向ML Kit提供您的TensorFlow Lite模型，ML Kit都将它们以标准序列化protobuf格式存储在本地存储中。

理论上，这意味着任何人都可以复制你的模型。但是，实际上，大多数模型都是特定于应用程序的，并且通过优化进行混淆，以至于风险与拆分和重用代码的竞争对手相似。因而，在应用中使用自定义模型之前，您应该意识到这种风险。

### 在Firebase上托管的模型

要在Firebase上托管您的TensorFlow Lite模型，请执行以下操作：

1. 在[Firebase控制台](https://console.firebase.google.com/)的**ML Kit**部分中，点击**自定义**标签。
2. 点击**添加自定义模型**（或**添加其他模型**）。
3. 指定将用于在Firebase项目中标识您的模型的名称，然后上传该`.tflite`文件。

将自定义模型添加到Firebase项目后，您可以使用指定的名称在应用中引用该模型。在任何时候，您都可以上传`.tflite`模型的新文件，然后您的应用将下载新模型，并在应用下次重新启动时开始使用它。您可以定义您的应用程序尝试更新模型所需的设备条件（请参阅下文）。

### 使模型可在本地使用

为了让您的TensorFlow Lite模型在本地可用，您可以将模型与应用程序捆绑在一起，也可以在运行时从您自己的服务器上下载模型。

要将TensorFlow Lite模型与您的应用程序捆绑在一起，请将该`.tflite`文件添加到您的Xcode项目中，注意选择**复制捆绑软件资源**。该`.tflite`文件将包含在应用程序包中，并可用于ML Kit。

如果您将模型托管在自己的服务器上，则可以在您应用程序的适当位置将模型下载到本地存储。然后，该模型将作为本地文件提供给ML Kit。

## 加载模型

要使用TensorFlow Lite模型进行推理，请首先指定`.tflite`文件的位置 。

如果您使用Firebase托管您的模型，请注册一个`CloudModelSource`对象，指定您上传模型时分配给模型的名称，以及最初ML Kit应该下载模型以及何时可用更新的条件。

Swift：

```swift
let conditions = ModelDownloadConditions(wiFiRequired: true, idleRequired: true)
let cloudModelSource = CloudModelSource(
  modelName: "my_cloud_model",
  enableModelUpdates: true,
  initialConditions: conditions,
  updateConditions: conditions
)
let registrationSuccessful = ModelManager.modelManager().register(cloudModelSource)
```

Objective-C：

```objective-c
FIRModelDownloadConditions *conditions =
    [[FIRModelDownloadConditions alloc] initWithWiFiRequired:YES
                                                idleRequired:YES];
FIRCloudModelSource *cloudModelSource =
    [[FIRCloudModelSource alloc] initWithModelName:@"my_cloud_model"
                                enableModelUpdates:YES
                                 initialConditions:conditions
                                  updateConditions:conditions];
  BOOL registrationSuccess =
      [[FIRModelManager modelManager] registerCloudModelSource:cloudModelSource];
```

如果将模型与您的应用程序捆绑在一起，或者在运行时从您自己的主机下载模型，请注册一个`LocalModelSource`对象，指定`.tflite`模型的本地路径，并为本地源分配一个唯一名称，以在您的应用程序中识别它。 

Swift：

```swift
guard let modelPath = Bundle.main.path(
  forResource: "my_model",
  ofType: "tflite"
) else {
  // 不恰当的路径
  return
}
let localModelSource = LocalModelSource(modelName: "my_local_model",
                                        path: modelPath)
let registrationSuccessful = ModelManager.modelManager().register(localModelSource)
```

Objective-C：

```objective-c
NSString *modelPath = [NSBundle.mainBundle pathForResource:@"my_model"
                                                    ofType:@"tflite"];
FIRLocalModelSource *localModelSource =
    [[FIRLocalModelSource alloc] initWithModelName:@"my_local_model"
                                              path:modelPath];
BOOL registrationSuccess =
      [[FIRModelManager modelManager] registerLocalModelSource:localModelSource];
```

然后，用Cloud源，本地源或两者兼有创建一个`ModelOptions`对象，并使用它来获取一个`ModelInterpreter`实例。如果您只有一个来源，请指定`nil`为您不使用的来源类型。 

Swift：

```
let options = ModelOptions(
  cloudModelName: "my_cloud_model",
  localModelName: "my_local_model"
)
let interpreter = ModelInterpreter(options: options)
```

Objective-C：

```objective-c
FIRModelOptions *options = [[FIRModelOptions alloc] initWithCloudModelName:@"my_cloud_model"
                                                            localModelName:@"my_local_model"];
FIRModelInterpreter *interpreter = [FIRModelInterpreter modelInterpreterWithOptions:options];
```

如果您同时指定了云模型源和本地模型源，那么模型解释器将在云模型可用时使用云模型，如果云模型不可用，则会回退到本地模型。 

## 指定模型的输入和输出

接下来，您必须通过创建`ModelInputOutputOptions`对象来指定模型输入和输出的格式 。

TensorFlow Lite模型将输入作为输出并生成一个或多个多维数组。这些数组包含`UInt8`， `Int32`，`Int64`，或`Float32`。您必须为ML Kit配置模型使用的数组的number和dimensions（“shape”）。

例如，一个图像分类模型可能需要输入一个1x640x480x3字节的数组，代表一个640x480真彩色（24位）图像，并产生一个1000个`Float32`值的列表，每个值决定了图像是处于概率模型预测的1000个类别中的一个成员类别中的概率。

Swift：

```swift
let ioOptions = ModelInputOutputOptions()
do {
  try ioOptions.setInputFormat(index: 0, type: .uInt8, dimensions: [1, 640, 480, 3])
  try ioOptions.setOutputFormat(index: 0, type: .float32, dimensions: [1, 1000])
} catch let error as NSError {
  print("Failed to set input or output format with error: \(error.localizedDescription)")
}
```

Objective-C：

```objective-c
FIRModelInputOutputOptions *ioOptions = [[FIRModelInputOutputOptions alloc] init];
NSError *error;
[ioOptions setInputFormatForIndex:0
                             type:FIRModelElementTypeUInt8
                       dimensions:@[@1, @640, @480, @3]
                            error:&error];
if (error != nil) { return; }
[ioOptions setOutputFormatForIndex:0
                              type:FIRModelElementTypeFloat32
                        dimensions:@[@1, @1000]
                             error:&error];
if (error != nil) { return; }
```

## 对输入数据进行推理

最后，要使用模型执行推理，请使用模型输入创建一个`ModelInputs` 对象，并将模型输入和模型的输入和输出选项传递给模型解释器的`run(inputs:options:)`方法。为获得最佳性能，请将模型输入作为`Data`（`NSData`）对象传递。 

Swift：

```swift
let input = ModelInputs()
do {
  var data: Data  // or var data: Array
  // Store input data in `data`
  // ...
  try input.addInput(data)
  // 对输入的指数重复有必要
} catch let error as NSError {
  print("Failed to add input: \(error.localizedDescription)")
}

interpreter.run(inputs: input, options: ioOptions) { outputs, error in
  guard error == nil, let outputs = outputs else { return }
  // 实现输出
  // ...
}
```

Objective-C：

```objective-c
FIRModelInputs *inputs = [[FIRModelInputs alloc] init];
NSData *data;  // Or NSArray *data;
// ...
[inputs addInput:data error:&error];  // 重复的必要
if (error != nil) { return; }
[interpreter runWithInputs:inputs
                   options:ioOptions
                completion:^(FIRModelOutputs * _Nullable outputs,
                             NSError * _Nullable error) {
  if (error != nil || outputs == nil) {
    return;
  }
  // 实现输出
  // ...
}];
```

您可以通过调用`output(index:)`返回的对象的方法来获取输出。例如： 

Swift：

```
// Get first and only output of inference with a batch size of 1
let probabilities = try? outputs.output(index: 0)
```

Objective-C：

```
// Get first and only output of inference with a batch size of 1
NSError *outputError;
[outputs outputAtIndex:0 error:&outputError];
```

您如何使用输出取决于您使用的模型。例如，如果您正在执行分类，那么作为下一步，您可能会将结果的索引映射到它们所代表的标签。 