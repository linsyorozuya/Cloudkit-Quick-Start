# 使用Asset和Location字段

CloudKit提供专门用于存储大型数据文件和按位置获取记录的字段类型。使用这些数据类型可以利用CloudKit为此类数据提供的性能改进。您还可以按位置获取记录。例如，在用户定义的区域中的地图上显示记录。

### 在CloudKit中存储大文件

您可以使用`Asset`字段类型在CloudKit中存储大型数据文件。`Asset`由关联记录拥有，CloudKit为您处理垃圾回收。CloudKit还可以有效地上传和下载`Asset`。

在代码中，`Asset`字段类型由[`CKAsset`](https://developer.apple.com/documentation/cloudkit/ckasset)对象表示。此代码片段将记录中的`Asset`字段设置`Artwork`为资源文件。

```swift
   // Create a URL to the local file
   NSURL *resourceURL = [NSURL fileURLWithPath:@"…"];
   if (resourceURL){
      CKAsset *asset = [[CKAsset alloc] initWithFileURL:resourceURL];
      artworkRecord[@"image"] = asset;
   }
```

保存记录后，文件将上载到iCloud。

添加类似的代码，将带有`Asset`字段的记录类型保存到您的应用程序并运行它。要保存记录，请阅读[维护CloudKit记录的本地缓存](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW1)。

#### 验证您的步骤

要验证您对架构和记录的更改是否已保存到iCloud，请阅读[“常规工作流程”](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW2) 和“ [获取更改”](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW7)。当您使用`Asset`对象查看记录时，CloudKit仪表板会显示二进制数据的大小。![../Art/ViewAsset.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/ViewAsset_2x.png)

### 添加位置字段

如果您的记录具有地址或其他位置数据，则可以将其保存为[`CLLocation`](https://developer.apple.com/documentation/corelocation/cllocation)记录中的对象，然后按位置获取记录。例如，您的应用可能会显示代表地图上记录的图钉。

此代码片段使用[`CLGeocoder`](https://developer.apple.com/documentation/corelocation/clgeocoder)该类将字符串地址转换为位置对象并将其存储在记录中。

```swift
CLGeocoder *geocoder = [CLGeocoder new];
[geocoder geocodeAddressString:artwork[kArtworkAddressKey] completionHandler:^(NSArray *placemark, NSError *error){
   if (!error) {
      if (placemark.count > 0){
         CLPlacemark *placement = placemark[0];
         artworkRecord[kArtworkLocationKey] = placement.location;
      }
   }
   else {
      // insert error handling here
   }
   // Save the record to the database
}];
```

添加类似的代码，将带有`Location`字段的记录类型保存到您的应用程序并运行它。要保存记录，请阅读[维护CloudKit记录的本地缓存](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW1)。

#### 验证您的步骤

在CloudKit仪表板中查看记录时，它会显示位置字段的经度和纬度。

![../Art/ViewLocation.shot/Resources/shot\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/ViewLocation_2x.png)

### 按位置获取记录

在数据库中有位置数据后，可以使用包含记录类型，谓词和排序描述符的查询按位置获取记录。`Location`必须为谓词中指定的字段建立索引才能使提取成功。

```swift
// Get the public database object
CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
 
// Create a predicate to retrieve records within a radius of the user's location
CLLocation *fixedLocation = [[CLLocation alloc] initWithLatitude:37.7749300 longitude:-122.4194200];
CGFloat radius = 100000; // meters
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"distanceToLocation:fromLocation:(location, %@) < %f", fixedLocation, radius];
 
// Create a query using the predicate
CKQuery *query = [[CKQuery alloc] initWithRecordType:@"Artwork" predicate:predicate];
 
// Execute the query
[publicDatabase performQuery:query inZoneWithID:nil completionHandler:^(NSArray *results, NSError *error) {
   if (error) {
      // Error handling for failed fetch from public database
   }
   else {
      // Display the fetched records
   }
}];
```

此代码片段提取位于旧金山100,000米范围内的所有记录。

在此iOS应用程序中，将获取固定位置指定半径内的图稿。

![../Art/6\_fetching\_by\_location\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/6_fetching_by_location_2x.png)

要了解有关位置和地图的更多信息，请阅读[_位置和地图编程指南_](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/LocationAwarenessPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009497)。

### 概括

在本章中，您学习了如何：

* 添加`Asset`添加一个类型的记录类型`CKAsset`字段，记录和保存记录
* `Location`使用`CLLocation`对象将类型添加到记录类型
* 按位置获取对象

