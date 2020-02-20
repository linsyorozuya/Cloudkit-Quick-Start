---
description: 架构 - schema
---

# 通过保存记录创建数据库架构

在开发期间，使用CloudKit API创建架构很容易。当你将记录对象保存到数据库时，会自动为您创建关联的记录类型及其字段。此功能称为即时架构，仅在您使用开发环境时才可用。例如，在开发期间，您可以使用存储在属性列表中的测试记录填充CloudKit数据库。

本章介绍了一些CloudKit API并包含代码片段。`#import <CloudKit/CloudKit.h>`在每个使用CloudKit类和方法的实现文件的顶部添加。有关CloudKit API的详细信息，请阅读[_Cloud Kit Framework Reference_](https://developer.apple.com/documentation/cloudkit)。

### 关于设计架构

设计CloudKit架构以存储要保留的应用程序对象模型部分。如果要从头开始实现应用程序，请使用模型 - 视图 - 控制器设计模式将用户界面视图与模型对象分开。然后设计一个模式，该模式可以有效地存储要存储在iCloud中的对象模型的各个部分。

#### 将数据分成记录类型

CloudKit架构由一个或多个具有名称，字段和其他元数据的记录类型组成。字段类型与允许的属性列表类型类似，但有一些附加内容。

* `Asset`类型用于与记录分开存储的批量数据。
* `Location`类型用于有效查询地理坐标。
* `Reference`类型表示记录之间的一对一和一对多关系的类型。记录的最大大小为1MB，因此对大数据使用`Asset`类型（不是`Bytes`类型）。

该表显示了CloudKit仪表板中显示的可能字段类型及其等效的CloudKit框架类。

| 字段类型 | 类 | 描述 |
| :--- | :--- | :--- |
| Asset | CKAsset | 与记录关联但单独存储的大文件 |
| Bytes | NSData | 与记录一起存储的字节缓冲区的包装器 |
| Date/Time | NSDate | 单个时间点 |
| Double | NSNumber | 一双 |
| INT（64） | NSNumber | 一个整数 |
| Location | CLLocation | 地理坐标和高度 |
| Reference | CKReference | 从一个对象到另一个对象的关系 |
| String | NSString | 不可变的文本字符串 |
| List | NSArray | 任何上述字段类型的数组 |

使用这些字段设计记录类型来存储应用程序的持久数据。例如，主 - 细节用户界面的草图在主界面（左侧的列）中显示艺术作品标题的集合，并在详细界面（右侧区域）中显示所选艺术作品的属性。

![../Art/masterdetail\_sketch\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/masterdetail_sketch_2x.png)

在代码中，底层对象模型包括的`Artwork`和`Artist`类，其中`Artwork`具有一对一关系`Artist`。

在该模式中，有一个在这个对象模型中的对象和记录类型之间有一个一对一映射`Artwork`和`Artist`。`Artwork`记录类型的`artist`字段是对`Artist`记录的引用的引用类型。该`image`字段是包含URL 的`Asset`类型，该`location`字段是具有经度和纬度属性的`Location`类型。 `Artwork`和`Artist`中的所有其他字段都是简单的字符串和日期类型。

![../Art/schema\_design\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/schema_design_2x.png)

#### 确定您的记录名称

确定用于为记录创建唯一名称的启发式方法。记录名称与记录区域（_record zone_ 数据库的分区）相结合是记录标识符（_record identifier_），表示数据库中记录的位置。记录名称可以是另一个数据源使用的外键，也可以是在记录区域内使其唯一的字符串组合。例如，记录的记录名称`Artwork`可以将艺术家的名字和姓氏与目录号组合在一起，如字符串中所示`115 Chen, Mei`。如果使用CloudKit仪表板创建记录，则会自动为记录分配唯一ID。

### 以编程方式创建记录

首先创建一个记录标识符，[`CKRecordID`](https://developer.apple.com/documentation/cloudkit/ckrecordid)该类的实例，指定记录名称和记录区域。然后创建一个记录，[`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)该类的实例，传递记录标识符。使用键值编码样式方法设置记录的字段。

**在代码中创建记录**

   1.创建指定唯一记录名称的记录ID。

```swift
let artworkRecordID = CKRecordID（recordName："115"）
CKRecordID * artworkRecordID = [[CKRecordID alloc] initWithRecordName：@"115"];
```

   2.创建一个记录对象

```swift
let artworkRecord = CKRecord（recordType："Artwork"，recordID：artworkRecordID）
CKRecord * artworkRecord = [[CKRecord alloc] initWithRecordType：@"Artwork" recordID：artworkRecordID];
```

   3.设置记录的字段。

```swift
artworkRecord [“title”] ="MacKerricher State Park" as NSString
artworkRecord [“artist”] ="Mei Chen" as NSString
artworkRecord [“address”] ="Fort Bragg，CA" as NSString
artworkRecord [@“title”] = @"MacKerricher State Park";
artworkRecord [@“artist”] = @"Mei Chen";
artworkRecord [@“address”] = @"Fort Bragg，CA";
```

### 保存记录

首先选择要保存记录的数据库（公共，私有或自定义），然后保存记录。如果记录不存在记录类型，则会为您创建记录类型。

**保存记录**

   1.在应用程序的默认容器中获取数据库。 

      要获取公共数据库：

```swift
let myContainer = CKContainer.default()
let myContainer = CKContainer.default()
myContainer = [CKContainer defaultContainer]; 
CKDatabase  publicDatabase = [myContainer publicCloudDatabase];
```

      要获取私人数据库：

```swift
let myContainer = CKContainer.default()
let privateDatabase = myContainer.privateDatabase
CKContainer *myContainer = [CKContainer defaultContainer];
CKDatabase *privateDatabase = [myContainer privateCloudDatabase];
```

      要获取自定义容器：

```swift
let myContainer = CKContainer(identifier: "iCloud.com.example.ajohnson.GalleryShared")
CKContainer *myContainer = [CKContainer containerWithIdentifier:@"iCloud.com.example.ajohnson.GalleryShared"];     
```

      要创建由多个应用共享的自定义容器，请阅读 [应用之间的共享容器](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/EnablingiCloudandConfiguringCloudKit/EnablingiCloudandConfiguringCloudKit.html#//apple_ref/doc/uid/TP40014987-CH2-SW8)。

   2.保存记录。

```swift
publicDatabase.save(artworkRecord) {
    (record, error) in
    if let error = error {
        // Insert error handling
        return
    }
    // Insert successfully saved record code
}

[publicDatabase saveRecord:artworkRecord completionHandler:^(CKRecord *artworkRecord, NSError *error){
   if (error) {
      // Insert error handling
      return
   }
   // Insert successfully saved record code
}];
```

         如果记录类型不存在，CloudKit框架将使用您设置的字段创建它

在单击Xco​​de中的“运行”按钮之前，请在设备上输入iCloud凭据，如下一节中所述。

### 在运行应用程序之前输入iCloud凭据

在开发中，当您在模拟器或设备上通过Xcode运行应用程序时，您需要输入iCloud凭据以读取公共数据库中的记录。在生产中，默认权限允许未经过身份验证的用户读取公共数据库中的记录，但不允许他们写入记录。

因此，在运行应用程序并将记录保存到数据库之前，请在iOS上的“设置”或Mac上的“系统偏好设置”中输入iCloud帐户。还启用iCloud Drive。稍后，编写必要的错误处理，以便在需要iCloud凭据时向用户显示对话框，如[警告用户输入iCloud凭据中所述](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW8)。

要在iOS模拟器中运行您的应用程序，请在选择模拟器之前在iOS模拟器中输入iCloud凭据，然后单击Xco​​de中的“运行”按钮。您需要为在Xcode的Scheme弹出菜单中选择的每个iOS Simulator执行这些步骤。

**在iOS模拟器中输入iCloud凭据**

1. 选择Xcode&gt;打开开发人员工具&gt; iOS模拟器。
2. 在iOS模拟器中，选择“硬件”&gt;“主页”。
3. 启动“设置”应用，然后单击“iCloud”。

   ![../Art/3\_icloud\_settings\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/3_icloud_settings_2x.png)

4. 输入Apple ID和密码。
5. 单击“登录”。

   等待iOS验证iCloud帐户。

6. 要启用iCloud Drive，请单击iCloud Drive开关。

   如果未显示此开关，则表示已启用iCloud Drive。

有关如何创建iCloud帐户，请阅读[创建iCloud帐户以进行开发](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/EnablingiCloudandConfiguringCloudKit/EnablingiCloudandConfiguringCloudKit.html#//apple_ref/doc/uid/TP40014987-CH2-SW7)。

### 提醒用户输入iCloud凭据

在保存记录之前，通过验证用户是否已登录其iCloud帐户来改善用户体验。如果用户未登录，则会发出警报，指示用户如何输入其iCloud凭据并启用iCloud Drive。在`else`下面的子句中插入保存记录的代码。

```swift
[[CKContainer defaultContainer] accountStatusWithCompletionHandler:^(CKAccountStatus accountStatus, NSError *error) {
    if (accountStatus == CKAccountStatusNoAccount) {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Sign in to iCloud"
                                                                       message:@"Sign in to your iCloud account to write records. On the Home screen, launch Settings, tap iCloud, and enter your Apple ID. Turn iCloud Drive on. If you don't have an iCloud account, tap Create a new Apple ID."
                                                                preferredStyle:UIAlertControllerStyleAlert];
        [alert addAction:[UIAlertAction actionWithTitle:@"Okay"
                                                  style:UIAlertActionStyleCancel
                                                handler:nil]];
        [self presentViewController:alert animated:YES completion:nil];
    }
    else {
        // Insert your just-in-time schema code here
    }
}];
To learn about errors that may occur when saving records, read CloudKit Framework Constants Reference.
```

要了解保存记录时可能发生的错误，请阅读 _CloudKit Framework Constants Reference_。

### 运行你的应用程序

在Xcode中，运行您的应用程序以执行保存记录的代码并在数据库中创建架构。

### 验证您的步骤

使用CloudKit仪表板验证记录类型是否已添加到架构中，以及记录是否已添加到数据库中。

#### 使用CloudKit仪表板查看记录类型

验证记录类型是否具有正确的字段名称和类型。

**查看记录类型**

1. 登录[CloudKit仪表板](https://icloud.developer.apple.com/dashboard)。
2. 从列表中选择应用程序使用的容器。
3. 在“开发”或“生产”环境中选择“数据”。
4. 在选项卡栏中，单击“记录类型”。
5. 选择记录类型。

   字段名称和类型显示在右侧的详细信息区域中。

   `Users`是系统保留的记录类型。不能被删除，但你可以添加字段。

![../Art/2017RecordTypes.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/2017RecordTypes_2x.png)

#### 在查看记录之前启用RecordName索引

默认情况下，使用实时架构创建的记录类型的所有元数据索引是被禁用的。需要启用recordName查询索引才能在CloudKit仪表板中查看关联的记录。

**启用recordName查询索引**

1. 在选项卡栏中，单击“索引”并选择记录类型。
2. 单击“添加索引”，然后选择“recordName”字段。
3. 单击保存记录类型。

#### 使用CloudKit仪表板查看记录

验证您保存的记录是否包含所有数据。

**查看记录**

1. 在[CloudKit仪表板](https://icloud.developer.apple.com/dashboard)的选项卡栏中，单击“记录”。
2. 通过添加过滤器或排序条件来定义查询。
3. 单击查询记录。
4. 在第二列中，选择recordName。

   记录键值对出现在右侧的详细信息区域中。

![../Art/2017ViewRecord.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/2017ViewRecord_2x.png)

### 概括

在本章中，您学习了如何：

* 通过以编程方式保存记录来创建模式
* 在多个应用之间共享容器ID
* 使用CloudKit仪表板查看您创建的记录类型和记录

