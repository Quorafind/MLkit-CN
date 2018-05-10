# 地标识别

借助ML Kit的地标识别API，您可以识别图像中的一些广为人知的地标。

当您将图片传递给此API时，您会看到其中已识别的地标，以及每个地标的地理坐标和发现地标的图像区域。您可以使用此信息自动生成图像元数据，根据用户共享的内容为用户创建个性化体验等等。

[iOS](https://github.com/Quorafind/MLkit-CN/blob/master/Recognize%20landmarks/Recognize%20Landmarks%20with%20ML%20Kit%20on%20iOS.md) [Android](https://github.com/Quorafind/MLkit-CN/blob/master/Recognize%20landmarks/Recognize%20Landmarks%20with%20ML%20Kit%20on%20Android.md)

## 核心功能

| 识别著名的地标           | 获取自然或者已建地标的名称和地理坐标以及地标所在图像的区域。试用 [Cloud Vision API演示](https://cloud.google.com/vision/docs/drag-and-drop)，查看您提供的图像中可以找到哪些地标。 |
| ------------------------ | ------------------------------------------------------------ |
| 获取Google知识图谱实体ID | 知识图谱实体ID是被识别的地标的唯一可识别字符串，并且是由[知识图谱搜索API](https://developers.google.com/knowledge-graph/)所使用的相同的ID 。您可以使用此字符串来识别跨语言的实体，并且得到不仅止于文本说明的格式。 |
| 小批量使用免费           | 每个月免费使用此功能1000次：更多详情请参阅 [定价](https://firebase.google.com/pricing) |

## 结果实例

![](https://github.com/Quorafind/MLkit-CN/blob/master/Recognize%20landmarks/680px-Bruegge_View_from_Rozenhoedkaai.jpg)		

Photo: Arcalino / Wikimedia Commons / CC BY-SA 3.0 

| 结果               |                                              |
| ------------------ | -------------------------------------------- |
| **描述**           | Brugge                                       |
| **地理坐标**       | 51.207367, 3.226933                          |
| **知识图谱实体ID** | /m/0drjd2                                    |
| **边界坐标**       | (20, 342), (651, 342), (651, 798), (20, 798) |
| **可信概率**       | 0.77150935                                   |