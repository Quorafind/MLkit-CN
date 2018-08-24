# 在iOS中使用自定义的TensorFlow版本

如果您是经验丰富的ML开发人员，并且预构建的TensorFlow Lite库不能满足您的需求，则可以使用ML Kit 自定义[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)版本。例如，您可能想要添加自定义操作。 

## 先决条件

- 一个可用的[TensorFlow Lite](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/lite/README.md#building-tensorflow-lite-and-the-demo-app-from-source)构建环境
- 检出(checkout)0.1.7的Tensorflow Lite

你可以通过使用git检出正确版本：

```
$ git checkout -b work
$ git reset --hard tflite-v0.1.7
$ git cherry-pick f1f1d5172fe5bfeaeb2cf657ffc43ba744187bee
```

## 构建Tensorflow Lite库

1. 按照[标准说明](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/lite/g3doc/ios.md)构建Tensorflow Lite（随您的修改）

2. 构建框架：

   ```
   $ tensorflow/contrib/lite/lib_package/create_ios_frameworks.sh
   ```

您可以在这里发现生成好的框架：

`tensorflow/contrib/lite/gen/ios_frameworks/tensorflow_lite.framework.zip `

**注意：这里有一些XCode 9.3 的[build issues reported](https://github.com/tensorflow/tensorflow/issues/18356) ** 

## 创建一个本地pod

1. 为您的本地pod创建一个目录

2. 在您创建的目录中运行`pod lib create TensorFlowLite`

3. 在`TensorFlowLite`目录中创建一个`Frameworks`目录

4. 解压缩上面生成的`tensorflow_lite.framework.zip`文件

5. 复制解压`tensorflow_lite.framework`到`TensorFlowLite/Frameworks`

6. 修改生成`TensorFlowLite/TensorFlowLite.podspec`引用的库：

   ```
       Pod::Spec.new do |s|
         s.name             = 'TensorFlowLite'
         s.version          = '0.1.7' # Version must match.
         s.ios.deployment_target = '9.0'
         
         # ... 让其它改变生效
         
         internal_pod_root = Pathname.pwd
         s.frameworks = 'Accelerate'
         s.libraries = 'c++'
         s.vendored_frameworks = 'Frameworks/tensorflow_lite.framework'
   
         s.pod_target_xcconfig = {
           'SWIFT_VERSION' => '4.0',
           'INTERNAL_POD_ROOT' => "#{internal_pod_root}",
           'HEADER_SEARCH_PATHS' => "$(inherited) '${INTERNAL_POD_ROOT}/Frameworks/tensorflow_lite.framework/Headers'",
           'OTHER_LDFLAGS' => "-force_load '${INTERNAL_POD_ROOT}/Frameworks/tensorflow_lite.framework/tensorflow_lite'"
         }
       end
   ```

## 在您的项目中引用自定义的pod

您可以通过直接从您的应用程序中的 `Podfile` 引用自定义pod：

```
pod 'Firebase/MLModelInterpreter'
pod 'TensorFlowLite', :path => 'path/to/your/TensorflowLite'
```

有关管理专用pod的其他选项，请参阅 Cocoapods文档中的[Private Pods](https://guides.cocoapods.org/making/private-cocoapods.html)。请注意，版本必须完全匹配，并且在从私有存储库包含pod时应引用此版本，例如`pod 'TensorFlowLite', "0.1.7"`。 