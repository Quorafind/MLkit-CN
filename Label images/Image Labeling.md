# 图像标签

使用ML Kit的图像标签API，您可以识别图像中的实体，而无需向设备上的API或基于云的API提供任何其他上下文元数据。

图像标签可让您深入了解图像的内容。当你使用API时，你会得到一个被识别的实体列表：人物，事物，地点，活动等等。出现的每个标签都带有一个分数，表示ML模型与其相关性的置信度。利用这些信息，您可以执行诸如自动元数据生成和内容审核等任务。

[iOS](https://github.com/Quorafind/MLkit-CN/blob/master/Label%20images/Label%20Images%20with%20ML%20Kit%20on%20iOS.md) [Android](https://github.com/Quorafind/MLkit-CN/blob/master/Label%20images/Label%20Images%20with%20ML%20Kit%20on%20iOS.md)

## 在设备上的API和云API之间进行选择

|                    |                        设备上                         |                             云端                             |
| ------------------ | :---------------------------------------------------: | :----------------------------------------------------------: |
| 价格               |                         免费                          | 每个月免费1000次使用：更多请参阅 [定价](https://firebase.google.com/pricing) |
| 标签范围           | 400多个标签可以覆盖照片中最常见的物体或行为。见下文。 | 许多类别中有超过10,000个标签。见下文。另外，请尝试 [Cloud Vision API演示](https://cloud.google.com/vision/docs/drag-and-drop)，了解您提供的图像可以找到哪些标签。 |
| 知识图谱实体ID支持 |                           √                           |                              √                               |

## 设备上的标签示例

基于设备的API支持400多种标签，如以下示例： 

| 类别 | 标签示例                     |
| ---- | ---------------------------- |
| 人   | `Crowd` `Selfie` `Smile`     |
| 活动 | `Dancing` `Eating` `Surfing` |
| 事情 | `Car` `Piano` `Receipt`      |
| 动物 | `Bird` `Cat` `Dog`           |
| 植物 | `Flower` `Fruit` `Vegetable` |
| 地方 | `Beach` `Lake` `Mountain`    |

## 云端的标签示例

基于云的API支持10,000多种标签，如以下示例： 

| 类别       | 标签示例                                 | 类别       | 标签示例                                      |
| ---------- | ---------------------------------------- | ---------- | --------------------------------------------- |
| 艺术和娱乐 | `Sculpture` `Musical Instrument` `Dance` | 天文物体   | `Comet` `Galaxy` `Star`                       |
| 工商业     | `Restaurant` `Factory` `Airline`         | 颜色       | `Red` `Green` `Blue`                          |
| 设计       | `Floral` `Pattern` `Wood Stain`          | 喝         | `Coffee` `Tea` `Milk`                         |
| 活动       | `Meeting` `Picnic` `Vacation`            | 虚构人物   | `Santa Claus` `Superhero` `Mythical creature` |
| 餐饮       | `Casserole` `Fruit` `Potato chip`        | 家庭和花园 | `Laundry basket` `Dishwasher` `Fountain`      |
| 行为       | `Wedding` `Dancing` `Motorsport`         | 物料       | `Ceramic` `Textile` `Fiber`                   |
| 媒体       | `Newsprint` `Document` `Sign`            | 运输方式   | `Aircraft` `Motorcycle` `Subway`              |
| 职业       | `Actor` `Florist` `Police`               | 生物       | `Plant` `Animal` `Fungus`                     |
| 组织       | `Government` `Club` `College`            | 地方       | `Airport` `Mountain` `Tent`                   |
| 知识       | `Robot` `Computer` `Solar panel`         | 事物       | `Bicycle` `Pipe` `Doll`                       |

## Google知识图实体ID

此外，ML Kit会返回每个标签的文字描述，还会返回标签的Google知识图谱实体ID。此ID是一个字符串，用于唯一标识由标签表示的实体，并且是和[知识图谱搜索API](https://developers.google.com/knowledge-graph/)使用的ID所相同的ID 。您可以使用此字符串来识别跨语言的实体，并且得到不仅限于文本说明的格式。

## 结果例子

![](https://github.com/Quorafind/MLkit-CN/blob/master/Label%20images/1024px-Valais_Cup_2013_-_OM-FC_Porto_13-07-2013_-_Brice_Samba_en_extension.jpg)	

Photo: Clément Bucco-Lechat / Wikimedia Commons / CC BY-SA 3.0 

| On-device      |            | Cloud          |                         |
| -------------- | ---------- | -------------- | ----------------------- |
| 描述           | Stadium    | 描述           | sport venue             |
| 知识图谱实体ID | /m/019cfy  | 知识图谱实体ID | /m/0bmgjqz              |
| 置信度         | 0.9205354  | 置信度         | 0.9860726               |
| 描述           | Sports     | 描述           | player                  |
| 知识图谱实体ID | /m/06ntj   | 知识图谱实体ID | /m/02vzx9               |
| 置信度         | 0.7531109  | 置信度         | 0.9797604               |
| 描述           | Event      | 描述           | stadium                 |
| 知识图谱实体ID | /m/081pkj  | 知识图谱实体ID | /m/019cfy               |
| 置信度         | 0.66905296 | 置信度         | 0.9635762               |
| 描述           | Leisure    | 描述           | soccer specific stadium |
| 知识图谱实体ID | /m/04g3r   | 知识图谱实体ID | /m/0404y4               |
| 置信度         | 0.59904146 | 置信度         | 0.95806926              |
| 描述           | Soccer     | 描述           | football player         |
| 知识图谱实体ID | /m/02vx4   | 知识图谱实体ID | /m/0gl2ny2              |
| 置信度         | 0.56384534 | 置信度         | 0.9510419               |
| 描述           | Net        | 描述           | sports                  |
| 知识图谱实体ID | /m/02qdwbp | 知识图谱实体ID | /m/06ntj                |
| 置信度         | 0.54679185 | 置信度         | 0.9253524               |
| 描述           | Plant      | 描述           | soccer player           |
| 知识图谱实体ID | /m/05s2s   | 知识图谱实体ID | /m/0pcq81q              |
| 置信度         | 0.524364   | 置信度         | 0.9033665               |
| ... etc.       |            | 描述           | arena                   |
|                |            | 知识图谱实体ID | /m/018lrm               |
|                |            | 置信度         | 0.8897188               |
|                |            | ... etc.       |                         |