---
description: 参考字段 - reference field、谓词 - predicate
---

# 添加参考字段

使用架构中的引用字段来表示模型对象之间的关系; 例如，表示分层数据或表示所有权。本章介绍如何添加对记录的引用，保存和获取带引用的记录，以及指定所有权以便自动删除相关记录。

### 关于在架构中建立模型关系

您可以使用引用字段类型来表示模型对象之间的一对一和一对多关系。在您的代码中，引用字段是一个[`CKReference`](https://developer.apple.com/documentation/cloudkit/ckreference)对象，它封装了目标记录的记录ID，并被添加到源记录中。要在架构中表示一对一关系，请将引用字段添加到源记录类型。

![../Art/to\_one\_relationship\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/to_one_relationship_2x.png)

要表示模型对象之间的一对多关系，如果引用是从子记录到父记录（即向子记录添加引用字段），则效率更高。子记录是源，父记录是此架构中的目标。

例如，从`Artist`到`Artwork`存在一对多关系，反过来是从`Artwork`到`Artist`是一对一的关系。

![../Art/gallery\_object\_model\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/gallery_object_model_2x.png)

要在架构中表示这些关系，将引用字段添加到名为`artist`的相应`Artwork`记录类型中。此参考字段将包含`Artist`记录的记录ID 。

![../Art/to\_many\_relationship\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/to_many_relationship_2x.png)

类似地，为了表示在对象模型中一个一对多关系从`Artist`到`Collection`，添加参考字段到`Collection`记录。获取这些记录后，可以在模型对象之间创建适当的一对一和一对多关系。

### 创建参考字段

在开发期间，保存包含引用字段的记录以生成架构。通过向[`CKReference`](https://developer.apple.com/documentation/cloudkit/ckreference)源记录添加对象来表示架构中的一对一关系。

**创建从一个记录到另一个记录的引用**

1. 创建或获取目标记录的记录ID。

   ```swift
   CKRecordID *artistRecordID = [[CKRecordID alloc] initWithRecordName:@"Mei Chen"];
   ```

2. 通过将目标的记录ID作为参数传递来创建引用对象。

   ```swift
   CKReference *artistReference = [[CKReference alloc] initWithRecordID:artistRecordID action:CKReferenceActionNone];
   ```

3. 将引用对象添加到源记录。

   ```swift
   CKRecord *artworkRecord;
   …
   artworkRecord[@"artist"] = artistReference;
   ```

保存源记录以创建记录类型，如[初始化容器](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW3)中所述。如果要保存包含引用的多个记录，请在一次操作中保存所有记录，如“[保存和获取多个记录的批处理操作](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/MaintainingaLocalCacheofCloudKitRecords/MaintainingaLocalCacheofCloudKitRecords.html#//apple_ref/doc/uid/TP40014987-CH12-SW1)”中所述。CloudKit将确保在源记录之前保存目标记录。

### 使用参考字段获取记录

不要在模型 - 视图 - 控制器设计模式中使用记录（[`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)对象）作为模型对象。而是从提取的记录创建单独的模型对象，尤其是当提取的记录包含引用时。您的应用程序负责解释记录之间的引用并在模型对象之间创建适当的关系。获取源记录时，不会自动获取目标记录。您的应用负责根据需要获取目标记录。根据引用所表现的关系类型，有不同的方法来获取目标和源记录。

如果可能，批量提取以解析模型对象之间的关系，如[“保存和获取多个记录的批处理操作”](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/MaintainingaLocalCacheofCloudKitRecords/MaintainingaLocalCacheofCloudKitRecords.html#//apple_ref/doc/uid/TP40014987-CH12-SW1)所述。

#### 解决一对一的关系

对于一对一关系，从源记录中获取引用字段并获取关联的目标记录。

**获取一对一关系的目标**

1. 获取参考字段。

   ```swift
   CKRecord *artworkRecord;
   …
   CKReference *referenceToArtist = artworkRecord[@"artist"];
   ```

2. 从引用中获取目标记录ID。

   ```swift
   CKRecordID *artistRecordID = artistReference.recordID;
   ```

3. 获取目标记录。

   ```swift
   [publicDatabase fetchRecordWithID:artistRecordID completionHandler:^(CKRecord *artistRecord, NSError *error) {
       if (error) {
           // Failed to fetch record
       }
       else {
           // Successfully fetched record
       }
   }];
   ```

   将代码添加到[`fetchRecordWithID:completionHandler:`](https://developer.apple.com/documentation/cloudkit/ckdatabase/1449126-fetch)完成处理程序参数。例如，添加代码以创建相应对象`Artwork`和`Artist`对象之间的一对一关系。

#### 解决一对多关系

对于一对多关系，使用谓词一次获取父记录的所有子项。您可以获取将父项作为其目标记录的所有记录。

**获取一对多关系的孩子**

1. 从[`CKRecordID`](https://developer.apple.com/documentation/cloudkit/ckrecordid)先前获取的父记录ID（）和父项的模型对象开始。

   例如，`Artist`从`Artist`记录创建模型对象。

   ```swift
   __block Artist *artist = [[Artist alloc] initWithRecord:artistRecord];
   ```

   使用，`__block`以便稍后可以在完成处理程序中访问父对象。

2. 创建谓词对象以获取子记录。

   ```swift
   NSPredicate *predicate = [NSPredicate predicateWithFormat:@“artist = %@”, artistRecordID];
   ```

3. 创建一个查询对象，指定要搜索的记录类型。

   ```swift
   CKQuery *query = [[CKQuery alloc] initWithRecordType:@“Artwork” predicate:predicate];
   ```

   在您的代码中，替换`@“Artwork”`为子记录类型的名称。

4. 执行提取。

   ```swift
   CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
   [publicDatabase performQuery:query inZoneWithID:nil completionHandler:^(NSArray *results, NSError *error) {
       if (error) {
           // Failed to fetch children of parent
       }
       else {
           // Create model objects for each child and set the one-to-many relationship from the parent to its children
       }
   }];
   ```

   将代码添加到`else`创建模型对象之间的对应关系的语句中

### 用于保存和获取多个记录的批处理操作

将多个记录批量保存和提取放到单个操作中会更高效。如果记录类型包含引用字段，这一点尤为重要。您可以使用[`CKModifyRecordsOperation`](https://developer.apple.com/documentation/cloudkit/ckmodifyrecordsoperation)对象在一个操作中保存新的源和目标记录。您可以创建对没有标识符的目标记录的引用。只要将源记录和目标记录一起保存，CloudKit就可以确保在相关[`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)对象的图形中，在源记录之前保存目标记录。您还可以使用[`CKFetchRecordsOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordsoperation)对象在一个操作中获取一组源记录的所有目标。

**在单个操作中获取多个记录**

1. 添加要提取的记录的所有记录ID到一个数组中。

   例如，要获取多个一对一关系，请将引用字段的所有目标记录ID添加到数组中。

2. 通过将记录ID数组作为参数传递来创建获取记录操作对象。

   ```swift
   CKFetchRecordsOperation *fetchRecordsOperation = [[CKFetchRecordsOperation alloc] initWithRecordIDs:fetchRecordIDs];
   ```

3. （可选）提供每个记录完成处理程序。

   ```swift
   fetchRecordsOperation.perRecordCompletionBlock = ^(CKRecord *record, CKRecordID *recordID, NSError *error) {
       if (error) {
           // Retain the record IDs for failed fetches
       }
       else {
           // Create a model object and set any relationships to other models
       }
   };
   ```

   如果要保存成功的单个记录中的数据，请提供每个记录完成处理程序。完成处理程序应为成功获取的记录创建模型对象，并且由于操作可能失败，因此应保留失败的获取的记录ID。

4. 设置整个操作的完成处理程序。

   ```swift
   fetchRecordsOperation.fetchRecordsCompletionBlock = ^(NSDictionary *recordsByRecordID, NSError *error) {
       if (error) {
           // Failed to fetch all or some of the records
       }
       else {
           // Update all associated views
       }
   };
   ```

   如果在步骤3中将记录ID保存在每个记录完成处理程序中，则可以尝试再次获取失败的记录。

5. 开始操作。

   ```swift
   fetchRecordsOperation.database = [[CKContainer defaultContainer] publicCloudDatabase];
   [fetchRecordsOperation start];
   ```

### 指定所有权以自动删除相关记录

您可以指定在删除目标记录时是否删除引用的源记录。例如，您可能希望在删除父级时删除一对多关系的子级。父记录_拥有_子记录。如果孩子拥有其他记录，他们也将被删除，导致一连串的删除。![../Art/cascade\_deletes\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/cascade_deletes_2x.png)

您在创建引用对象时指定删除操作。要在删除目标时删除源记录，请将目标记录和[`CKReferenceActionDeleteSelf`](https://developer.apple.com/documentation/cloudkit/ckrecord_reference_action/deleteself)操作参数作为[`initWithRecord:action:`](https://developer.apple.com/documentation/cloudkit/ckreference/1515312-initwithrecord)方法传递。在“图库”示例中，指定删除艺术家时应删除属于艺术家的图稿。

```swift
CKReference *referenceToArtist = [[CKReference alloc] initWithRecord:artistRecord action:CKReferenceActionDeleteSelf];
artworkRecord[@"artist"] = referenceToArtist;
```

或者，检查CloudKit仪表板中的每条记录的DeleteSelf框。

当删除[`CKReferenceActionDeleteSelf`](https://developer.apple.com/documentation/cloudkit/ckrecord_reference_action/deleteself)引用的第一个目标记录时，CloudKit会删除源记录。如果源记录具有多个[`CKReferenceActionDeleteSelf`](https://developer.apple.com/documentation/cloudkit/ckrecord_reference_action/deleteself)引用，则即使存在某些其他目标记录，也可能将其删除，从而导致数据模型不一致。尝试设计您的架构，以便每个记录只有一个[`CKReferenceActionDeleteSelf`](https://developer.apple.com/documentation/cloudkit/ckrecord_reference_action/deleteself)引用。![../Art/first\_delete\_wins\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/first_delete_wins_2x.png)

