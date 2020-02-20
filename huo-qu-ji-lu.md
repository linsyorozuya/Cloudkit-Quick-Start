---
description: 谓词 - predicate      标识符 - Identifier
---

# 获取记录

将记录保存到数据库后，可以使用不同的机制检索它们。按记录ID获取单个记录，或使用谓词查询多条记录。（谓词定义了搜索记录的逻辑条件。）通常，您获取要在启动时显示给用户的记录子集，然后订阅对用户感兴趣的更改。

如果使用`Location`字段类型，还可以获取地理区域内的记录，如[按位置获取记录中](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/AddingAssetsandLocations/AddingAssetsandLocations.html#//apple_ref/doc/uid/TP40014987-CH6-SW1)所述。

### 按标识符获取记录

如果您知道要获取的记录的记录ID，则可以按单个记录ID获取。例如，此代码片段获取一个名为的记录`115`。

```swift
CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
CKRecordID *artworkRecordID = [[CKRecordID alloc] initWithRecordName:@"115"];
[publicDatabase fetchRecordWithID:artworkRecordID completionHandler:^(CKRecord *artworkRecord, NSError *error) {
   if (error) {
      // Error handling for failed fetch from public database
   }
   else {
      // Display the fetched record
   }
}];
```

### 获取和修改记录

您可以获取，修改和保存对单个记录所做的更改。此代码片段获取`Artwork`记录，更改其`date`字段值，并将其保存到数据库。

```swift
// Fetch the record from the database
CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
CKRecordID *artworkRecordID = [[CKRecordID alloc] initWithRecordName:@"115"];
[publicDatabase fetchRecordWithID:artworkRecordID completionHandler:^(CKRecord *artworkRecord, NSError *error) {
   if (error) {
      // Error handling for failed fetch from public database
   }
   else {
      // Modify the record and save it to the database
      NSDate *date = artworkRecord[@"date"];
      artworkRecord[@"date"] = [date dateByAddingTimeInterval:30.0 * 60.0];
      [publicDatabase saveRecord:artworkRecord completionHandler:^(CKRecord *savedRecord, NSError *saveError) {
         // Error handling for failed save to public database
      }];
   }
}];

```

### 使用谓词查询记录

如果您有许多记录并在iCloud中存储大文件，则您不太可能希望在设备上本地存储所有记录。而是使用查询获取一部分数据。查询组合了记录类型，谓词和排序描述符，其中谓词包含索引的字段。您使用[`CKQuery`](https://developer.apple.com/documentation/cloudkit/ckquery)对象在代码中构建查询。

例如，此代码片段将获取具有指定标题的所有图稿。

```swift
CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"title = %@", @"Santa Cruz Mountains"];
CKQuery *query = [[CKQuery alloc] initWithRecordType:@"Artwork" predicate:predicate];
[publicDatabase performQuery:query inZoneWithID:nil completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        // Error handling for failed fetch from public database
    }
    else {
        // Display the fetched records
    }
}];
```

在Gallery应用程序中，将获取具有指定标题的图稿。

![../Art/5\_fetching\_by\_attribute\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/5_fetching_by_attribute_2x.png)

### 概括

在本章中，您学习了如何：

* 按标识符获取记录
* 获取，修改和保存单个记录
* 使用查询和谓词获取多个记录 

