# 使用CloudKit仪表板管理数据库

您可以使用CloudKit仪表板执行许多数据库管理任务。例如，您可以使用CloudKit仪表板修改架构和记录。容器的数据库存在于开发环境和生产环境中。您可以执行的操作取决于您是在开发环境还是生产环境中。

转到[CloudKit仪表板](https://icloud.developer.apple.com/dashboard)，登录，然后单击选项以浏览CloudKit仪表板功能。

![../Art/Dashboard1.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/Dashboard1_2x.png)

### 关于开发和生产环境

开发环境用于创建模式并添加用于测试的记录。生产环境由商店销售的应用程序访问。开发中的应用程序可以访问开发环境或生产环境。但是，商店中销售的应用程序只能访问生产环境。

在开发环境中，CloudKit会根据您保存到数据库的记录自动为您创建架构。此功能允许您迭代和优化您的架构，而无需显式创建它。您还可以使用CloudKit仪表板来修改和添加记录。

第一次部署架构时，架构将复制到生产环境（记录不会复制到生产环境中）。下次部署架构时，架构将与生产架构合并。为防止冲突，您无法删除先前部署到生产环境的开发模式中的字段或记录类型。

在生产环境中，您无法更改架构，但可以在公共数据库中添加，修改和删除记录。

![../Art/developer\_workflow\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/developer_workflow_2x.png)

当您通过Xcode运行CloudKit应用程序时，它会自动配置为使用开发环境。从Xcode导出应用程序进行测试时，可以指定开发环境或生产环境。将应用程序提交到商店时，会将其配置为使用生产环境。

### 选择您的容器

CloudKit仪表板中的所有功能都适用于当前选定的容器。使用左上角的弹出菜单切换容器。CloudKit仪表板显示属于您所属的所有Apple Developer Program团队的所有容器。在执行本章中的任何任务之前，请务必选择您正在开发的应用程序使用的容器。

### 重置开发环境

如果使用即时模式使用记录填充数据库（如[初始化容器中所述）](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW3)，则可以在应用程序运行之间重置开发环境。如果您从未部署过开发环境，则重置开发环境会删除所有记录和记录类型。否则，它会删除所有记录并将架构返回到生产环境的状态。

**重置开发环境**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击开发环境中的“重置...”。
2. 在出现的对话框中，阅读警告，选中复选框，然后单击“重置”。

![../Art/Reset2017.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/Reset2017_2x.png)

### 创建和删除记录类型

在开发环境中，您可以使用CloudKit仪表板创建，修改和删除记录类型。

**创建记录类型**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击“开发”环境下的“数据”。
2. 在选项卡栏中选择“记录类型”。
3. 单击“创建新类型”按钮。
4. 在“新记录类型”字段中输入名称。

   ![../Art/CreateRecord1.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/CreateRecord1_2x.png)

5. 要添加字段，请单击“添加字段”，输入字段名称，然后从弹出菜单中选择字段类型。
6. 要删除字段，请单击字段行中的“删除”按钮（x）。

   如果部署了该字段，则会禁用“删除”按钮。

7. 单击保存记录类型。

您只能在开发环境中删除记录类型，并且只能在未部署该记录类型时删除记录类型。删除记录类型时，其所有关联记录也将从数据库中删除。

**删除记录类型**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击“开发”环境中的“数据”。
2. 在选项卡栏中选择“记录类型”。
3. 选择要删除的记录类型。
4. 单击“删除记录类型”按钮。

   如果部署了记录类型，则禁用删除选项。

5. 在出现的对话框中，单击“删除”。

### 添加，修改和删除记录

在开发和生产环境中，您可以使用CloudKit仪表板在公共数据库中添加，修改和删除记录。

**创建记录**

1. 在[CloudKit仪表板中，](https://icloud.developer.apple.com/dashboard)单击“开发”或“生产”环境中的“数据”。
2. 从选项卡栏中选择记录。
3. 单击“创建新记录...”以开始新记录。

   CloudKit仪表板分配随机UUID作为记录名称。

4. 在文本字段中输入值。
5. 对于日期/时间类型，请在单独的文本字段中输入日历日期和时间值。![../Art/CreateRecord3.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/CreateRecord3_2x.png)
6. 对于`Asset`值，将文件拖到框中，或单击“选择文件”以上载文件。
7. 对于位置类型，请在单独的文本字段中输入纬度和经度。

   纬度范围为-90到90，经度范围为-180到180。

8. 单击保存。

**查看，修改或删除记录**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击“开发”或“生产”环境中的“数据”。
2. 从选项卡栏中选择记录。
3. 查询要查看，修改或删除的记录。
4. 选择要查看，编辑或删除的记录。

   记录字段显示在详细信息区域中。

5. 要编辑记录，请在文本字段中输入新值，然后单击“保存”。
6. 要删除记录，请单击“删除”按钮，然后在出现的对话框中单击“删除”。

### 搜索记录

在开发和生产环境中，您可以搜索具有字符串字段的记录。

**搜索记录**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击“开发”或“生产”环境中的“数据”。
2. 从选项卡栏中选择记录。
3. 选择要查询的记录类型。
4. 为查询添加过滤器或排序条件。
5. 单击查询记录按钮。

### 概括

本章介绍如何使用CloudKit仪表板管理数据库。你学会了如何：

* 将开发环境重置为已知状态
* 创建和删除记录类型
* 创建和编辑记录

