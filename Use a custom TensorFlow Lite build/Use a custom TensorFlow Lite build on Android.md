# 在安卓中使用自定义的TensorFlow版本

如果您是经验丰富的ML开发人员，并且预设的TensorFlow Lite库不能满足您的需求，则可以使用ML Kit 自定义[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)版本。例如，您可能想要添加自定义操作。 

## 预设条件

- 一个可用的[TensorFlow Lite](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/lite/README.md#building-tensorflow-lite-and-the-demo-app-from-source)构建环境

## 为Android捆绑自定义的TensorFlow Lite

构建Tensorflow Lite AAR： 

```java
$ bazel build --cxxopt='--std=c++11' -c opt        \
  --fat_apk_cpu=x86,x86_64,arm64-v8a,armeabi-v7a   \
  //tensorflow/contrib/lite/java:tensorflow-lite
```

这将在`bazel-genfiles/tensorflow/contrib/lite/java/`中生成一个AAR文件。将自定义Tensorflow Lite AAR发布到您的本地 [Maven](https://maven.apache.org/download.cgi)存储库： 

```
$ mvn install:install-file -Dfile=bazel-genfiles/tensorflow/contrib/lite/java/tensorflow-lite.aar -DgroupId=org.tensorflow \
  -DartifactId=tensorflow-lite -Dversion=0.1.100 -Dpackaging=aar
```

最后，在您的应用程序中`build.gradle`，用您的自定义的版本覆盖Tensorflow Lite版本：

```java
implementation 'org.tensorflow:tensorflow-lite:0.1.100'
```