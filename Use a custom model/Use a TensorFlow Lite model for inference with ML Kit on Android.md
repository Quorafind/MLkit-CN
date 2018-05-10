# 在安卓上通过ML Kit使用TensorFlow Lite模型进行推理

您可以通过ML Kit使用[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)模型执行设备上的推理 。

此API需要Android SDK level 16（Jelly Bean）或更新的版本。

有关此API使用的示例，请参阅GitHub上的[ML Kit快速入门示例](https://github.com/firebase/quickstart-android/tree/master/mlkit)，或者尝试使用[codelab](http://g.co/codelabs/mlkit-android-custom-model)。

## 在你开始之前

1. 如果您还没有将Firebase添加到您的程序当中，那您可以从[开始指南](https://firebase.google.com/docs/ios/setup)来开始您的工作。

2. 在应用级的`build.gradle` 文件中为ML kit添加依赖：

   ```java
   dependencies {
     // ...
   
     implementation 'com.google.firebase:firebase-ml-model-interpreter:15.0.0'
   }
   ```

3. 转换您想要的TensorFlow模型为TensorFlow Lite(tflite)格式。请查看[TOCO：TensorFlow Lite 优化转换器](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite/toco)

## 托管或捆绑您的模型

在您的应用程序中使用TensorFlow Lite模型进行推理之前，您必须将该模型提供给ML Kit。ML Kit可以使用Firebase远程托管的，本地存储在设备上的或两者兼而有之的TensorFlow Lite模型。

通过在Firebase上托管模型并在本地支持，您可以确保在模型可用时使用最新版本的模型，但是当Firebase托管的模型不可用时，您的应用程序的ML功能仍可使用。

### 模型安全

无论您如何向ML Kit提供您的TensorFlow Lite模型，ML Kit都将它们以标准序列化protobuf格式存储在本地存储中。

理论上，这意味着任何人都可以复制你的模型。但是，实际上，大多数模型都是特定于应用程序的，并且通过优化进行混淆，以至于风险与拆分和重用代码的竞争对手相似。因而，在应用中使用自定义模型之前，您应该意识到这种风险。

在Android API level 21(Lollipop) 或者更新版本中，模型将会被下载到一个[不会自动备份](https://developer.android.com/reference/android/content/Context#getNoBackupFilesDir())的目录当中。

在Android API level 20 或者更加旧的版本中，这个模型将会被下载到app-private内部存储中指定的一个名为`com.google.firebase.ml.custom.models`的目录当中。如果您启用了文件备份`BackupAgent` ，您可以选择排除掉该目录。

### 在Firebase上托管的模型

要在Firebase上托管您的TensorFlow Lite模型，请执行以下操作：

1. 在[Firebase控制台](https://console.firebase.google.com/)的**ML Kit**部分中，点击**自定义**标签。

2. 点击**添加自定义模型**（或**添加其他模型**）。

3. 指定将用于在Firebase项目中标识您的模型的名称，然后上传该`.tflite`文件。

4. 在您的应用的manifest当中，您需要声明`INTERNET`权限：

   ```java
   <uses-permission android:name="android.permission.INTERNET" />
   ```

5. 如果您的版本是Android SDK level 18(Jellybean)或者更老的版本的时候，您还需要声明以下权限：

   ```java
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                    android:maxSdkVersion="18" />
   <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
                    android:maxSdkVersion="18" />
   ```

将自定义模型添加到Firebase项目后，您可以使用指定的名称在应用中引用该模型。在任何时候，您都可以上传`.tflite`模型的新文件，然后您的应用将下载新模型，并在应用下次重新启动时开始使用它。您可以定义您的应用程序尝试更新模型所需的设备条件（请参阅下文）。

### 使模型可在本地使用

为了使您的TensorFlow Lite模型在本地可用，您可以将模型与应用程序捆绑在一起，也可以在应用程序中从自己的服务器中下载模型。

要将TensorFlow Lite模型与您的应用程序捆绑在一起，请将该`.tflite`文件复制到您应用程序的`assets/`文件夹中。（您可能需要先通过右键单击`app/`文件夹来创建文件夹，然后单击**新建>文件夹>资源文件夹(Assets Folder)**。）

然后，将以下内容添加到您的项目`build.gradle`文件中：

```java
android {

    // ...

    aaptOptions {
        noCompress "tflite"
    }
}
```

然后这个`.tflite`文件将会包括在应用程序的包中，并且在ML kit中变成可用的预设。

如果您选择您希望让模型托管在您自有的服务器上，您可以下载模型到本地存储中您的应用程序的指定位置去，然后这个模型也能够为ML kit所用。

## 加载模型

要使用TensorFlow Lite模型进行推理，请首先指定`.tflite`文件的位置 。

如果您使用Firebase托管您的模型，请注册一个 `FirebaseCloudModelSource` 对象，指定您上传模型时分配给模型的名称，以及最初ML Kit应该下载模型以及何时可用更新的条件。

```java
FirebaseModelDownloadConditions.Builder conditionsBuilder =
        new FirebaseModelDownloadConditions.Builder().requireWifi();
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    // Enable advanced conditions on Android Nougat and newer.
    conditionsBuilder = conditionsBuilder
            .requireCharging()
            .requireDeviceIdle();
}
FirebaseModelDownloadConditions conditions = conditionsBuilder.build();

// 通过指定您上传其到Firebase时的模型的名称构建一个 FirebaseCloudModelSource 对象
FirebaseCloudModelSource cloudSource = new FirebaseCloudModelSource.Builder("my_cloud_model")
        .enableModelUpdates(true)
        .setInitialDownloadConditions(conditions)
        .setUpdatesDownloadConditions(conditions)
        .build();
FirebaseModelManager.getInstance().registerCloudModelSource(cloudSource);
```

如果您想将模型与应用程序捆绑在一起，或者在运行时从您自己的托管服务器下载模型，请创建一个`FirebaseLocalModelSource`对象，指定`.tflite`模型的文件名以及文件是否为预设（如果已捆绑）或在本地存储中（如果在运行时已下载）。 

```java
FirebaseLocalModelSource localSource = new FirebaseLocalModelSource.Builder("my_local_model")
        .setAssetFilePath("mymodel.tflite")  // 或者setFilePath， 如果您是从自己的服务器下载的话
        .build();
FirebaseModelManager.getInstance().registerLocalModelSource(localSource);
```

然后，使用您的Cloud源，本地源或两者的名称创建一个`FirebaseModelOptions`对象，并使用它来获取以下`FirebaseModelInterpreter`实例： 

```java
FirebaseModelOptions options = new FirebaseModelOptions.Builder()
        .setCloudModelName("my_cloud_model")
        .setLocalModelName("my_local_model")
        .build();
FirebaseModelInterpreter firebaseInterpreter =
        FirebaseModelInterpreter.getInstance(options);
```

如果您同时指定了云模型源和本地模型源，那么模型推理器将在云模型可用时使用云模型，如果云模型不可用，则会回退到本地模型。 

## 指定模型的输入和输出

接下来，您必须通过创建`FirebaseModelInputOutputOptions`对象来指定模型输入和输出的格式 。

TensorFlow Lite模型将输入作为输出并生成一个或多个多维数组。这些数组包含`UInt8`， `Int32`，`Int64`，或`Float32`。您必须为ML Kit配置模型使用的数组的number和dimensions（“shape”）。

例如，一个图像分类模型可能需要输入一个1x640x480x3字节的数组，代表一个640x480真彩色（24位）图像，并产生一个1000个`Float32`值的列表，每个值决定了图像是处于概率模型预测的1000个类别中的一个成员类别中的概率。

```java
FirebaseModelInputOutputOptions inputOutputOptions =
    new FirebaseModelInputOutputOptions.Builder()
        .setInputFormat(0, FirebaseModelDataType.BYTE, new int[]{1, 640, 480, 3})
        .setOutputFormat(0, FirebaseModelDataType.FLOAT32, new int[]{1, 1000})
        .build();
```

## 对输入数据进行推理

最后，要使用该模型执行推理，请与您的模型的输入创建一个`FirebaseModelInputs` 对象，还有模型的输入和输出规范，并将这三样参数一并传递给模型推理器的`run`方法： 

```java
byte[][][][] input = new byte[1][640][480][3];
input = getYourInputData();
FirebaseModelInputs inputs = new FirebaseModelInputs.Builder()
    .add(input)  // add() 尽可能企及您的模型输入数量
    .build();
Task<FirebaseModelOutputs> result =
    firebaseInterpreter.run(inputs, inputOutputOptions)
        .addOnSuccessListener(
          new OnSuccessListener<FirebaseModelOutputs>() {
            @Override
            public void onSuccess(FirebaseModelOutputs result) {
              // ...
            }
          })
        .addOnFailureListener(
          new OnFailureListener() {
            @Override
            public void onFailure(@NonNull Exception e) {
              // 任务如若出错则报异常
              // ...
            }
          });
```

您可以通过调用`getOutput()`传递给成功侦听器(success listener)的对象的方法来获取输出。例如： 

```java
float[][] output = result.<float[][]>getOutput(0);
float[] probabilities = output[0];
```

您如何使用输出取决于您使用的模型。例如，如果您正在执行分类，那么作为下一步，您可能会将结果的索引映射到它们所代表的标签。 