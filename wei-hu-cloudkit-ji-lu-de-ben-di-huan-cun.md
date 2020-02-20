# 维护CloudKit记录的本地缓存

您可能希望将CloudKit记录的本地缓存添加到您的应用，以支持离线使用您的应用或提高性能。或者您可能已经为您的应用程序提供了数据存储，并且您希望添加对在CloudKit中持久存储该数据的支持。

### 一般工作流程

配置应用程序以维护本地缓存后，以下是您的应用将遵循的常规流程：

1. 当您的应用首次在新设备上启动时，它将订阅用户私有和共享数据库中的更改。
2. 当用户在设备A上本地修改其数据时，您的应用会将这些更改发送到CloudKit。
3. 您的应用将在同一用户的设备B上收到推送通知，通知它已在服务器上进行了更改。
4. 您在设备B上的应用程序将询问服务器自上次与服务器通信后发生的更改，然后使用这些更改更新其本地缓存。

### 初始化容器

应用程序启动时，应用程序的初始化逻辑应该会运行。无论您是否已经创建了区域和订阅，您的应用都应该在本地缓存，这样您就不会在每次启动时发出不必要的请求。

首先，代码定义了整个示例中要使用的项目。

```swift
let container = CKContainer.default()
let privateDB = container.privateCloudDatabase
let sharedDB = container.sharedCloudDatabase
 
// Use a consistent zone ID across the user's devices
// CKCurrentUserDefaultName specifies the current user's ID when creating a zone ID
let zoneID = CKRecordZoneID(zoneName: "Todos", ownerName: CKCurrentUserDefaultName)
 
// Store these to disk so that they persist across launches
var createdCustomZone = false
var subscribedToPrivateChanges = false
var subscribedToSharedChanges = false
 
let privateSubscriptionId = "private-changes"
let sharedSubscriptionId = "shared-changes"
```

#### 创建自定义区域

要使用CloudKit的更改跟踪功能，您需要将应用程序数据存储在用户私有数据库的自定义区域中。您可以通过实例化[CKModifyRecordZonesOperation](https://developer.apple.com/library/prerelease/ios/documentation/CloudKit/Reference/CKModifyRecordZonesOperation_class/index.html#//apple_ref/occ/cl/CKModifyRecordZonesOperation)对象来创建自定义区域，如下所示。

```swift
let createZoneGroup = DispatchGroup()
 
if !self.createdCustomZone {
    createZoneGroup.enter()
    
    let customZone = CKRecordZone(zoneID: zoneID)
    
    let createZoneOperation = CKModifyRecordZonesOperation(recordZonesToSave: [customZone], recordZoneIDsToDelete: [] )
    
    createZoneOperation.modifyRecordZonesCompletionBlock = { (saved, deleted, error) in
        if (error == nil) { self.createdCustomZone = true }
        // else custom error handling
        createZoneGroup.leave()
    }
    createZoneOperation.qualityOfService = .userInitiated
    
    self.privateDB.add(createZoneOperation)
}
```

#### 订阅变更通知

您的应用需要订阅其他设备所做的更改。订阅会告知CloudKit您关注哪些数据，以便在数据发生变化时向您的应用发送推送通知。

您的应用程序需要创建两个数据库更改（[`CKDatabaseSubscription`](https://developer.apple.com/documentation/cloudkit/ckdatabasesubscription)对象）订阅，一个用于私有数据库，另一个用于共享数据库。

```swift
if !self.subscribedToPrivateChanges {
    let createSubscriptionOperation = self.createDatabaseSubscriptionOperation(subscriptionId: privateSubscriptionId)
    createSubscriptionOperation.modifySubscriptionsCompletionBlock = { (subscriptions, deletedIds, error) in
        if error == nil { self.subscribedToPrivateChanges = true }
        // else custom error handling
    }
    self.privateDB.add(createSubscriptionOperation)
}
 
if !self.subscribedToSharedChanges {
    let createSubscriptionOperation = self.createDatabaseSubscriptionOperation(subscriptionId: sharedSubscriptionId)
    createSubscriptionOperation.modifySubscriptionsCompletionBlock = { (subscriptions, deletedIds, error) in
        if error == nil { self.subscribedToSharedChanges = true }
        // else custom error handling
    }
    self.sharedDB.add(createSubscriptionOperation)
}
 
// Fetch any changes from the server that happened while the app wasn't running
createZoneGroup.notify(queue: DispatchQueue.global()) {
    if self.createdCustomZone {
        self.fetchChanges(in: .private) {}
        self.fetchChanges(in: .shared) {}
    }
}
```

这些订阅告诉CloudKit只要在您创建订阅的数据库中添加，修改或删除记录或区域，就可以在此设备上向您的应用发送推送通知。

您可能希望配置订阅以发送静默推送通知。这些通知唤醒您的应用，以便它可以获取更改，但应用程序不会向用户显示警报。

```swift
func createDatabaseSubscriptionOperation(subscriptionId: String) -> CKModifySubscriptionsOperation {
    let subscription = CKDatabaseSubscription.init(subscriptionID: subscriptionId)
    
    let notificationInfo = CKNotificationInfo()
    // send a silent notification
    notificationInfo.shouldSendContentAvailable = true
    subscription.notificationInfo = notificationInfo
    
    let operation = CKModifySubscriptionsOperation(subscriptionsToSave: [subscription], subscriptionIDsToDelete: [])
    operation.qualityOfService = .utility
    
    return operation
}
```

#### 倾听推送通知

在配置应用程序以使用CloudKit时，您需要配置应用程序以侦听远程通知。

`CKNotification(fromRemoteNotificationDictionary: dict)`在`userInfo`字典上使用以确定您的应用程序接收的远程通知是否由a触发`CKSubscription`。

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    application.registerForRemoteNotifications()
    return true
}
 
func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    print("Received notification!")
    
    let viewController = self.window?.rootViewController as? ViewController
    guard let viewController = self.window?.rootViewController as? ViewController else { return }
    
    let dict = userInfo as! [String: NSObject]
    guard let notification:CKDatabaseNotification = CKNotification(fromRemoteNotificationDictionary:dict) as? CKDatabaseNotification else { return }
    
    viewController!.fetchChanges(in: notification.databaseScope) {
        completionHandler(.newData)
    }
}
```

### 获取更改

应用程序启动或者收到一推后，您的应用程序使用[CKFetchDatabaseChangesOperation](https://developer.apple.com/reference/cloudkit/ckfetchdatabasechangesoperation)然后[CKFetchRecordZoneChangesOperation](https://developer.apple.com/reference/cloudkit/ckfetchrecordzonechangesoperation)向服务器只更改自从上次更新它。

这些操作的关键是`previousServerChangeToken`对象，它告诉服务器您的应用程序上次与服务器通话的时间，允许服务器仅返回自那时以来更改的项目。

首先，您的应用将使用a `CKFetchDatabaseChangesOperation`来找出哪些区域已更改，并且：

1. 收集新区域和更新区域的ID。
2. 清除已删除区域中的本地数据。

以下是一些获取数据库更改的示例代码：

```swift
func fetchChanges(in databaseScope: CKDatabaseScope, completion: @escaping () -> Void) {
    switch databaseScope {
    case .private:
        fetchDatabaseChanges(database: self.privateDB, databaseTokenKey: "private", completion: completion)
    case .shared:
        fetchDatabaseChanges(database: self.sharedDB, databaseTokenKey: "shared", completion: completion)
    case .public:
        fatalError()
    }
}
 
func fetchDatabaseChanges(database: CKDatabase, databaseTokenKey: String, completion: @escaping () -> Void) {
    var changedZoneIDs: [CKRecordZoneID] = []
    
    let changeToken = … // Read change token from disk
    let operation = CKFetchDatabaseChangesOperation(previousServerChangeToken: changeToken)
    
    operation.recordZoneWithIDChangedBlock = { (zoneID) in
        changedZoneIDs.append(zoneID)
    }
    
    operation.recordZoneWithIDWasDeletedBlock = { (zoneID) in
        // Write this zone deletion to memory
    }
    
    operation.changeTokenUpdatedBlock = { (token) in
        // Flush zone deletions for this database to disk
        // Write this new database change token to memory
    }
    
    operation.fetchDatabaseChangesCompletionBlock = { (token, moreComing, error) in
        if let error = error {
            print("Error during fetch shared database changes operation", error)
            completion()
            return
        }
        // Flush zone deletions for this database to disk
        // Write this new database change token to memory
        
        self.fetchZoneChanges(database: database, databaseTokenKey: databaseTokenKey, zoneIDs: changedZoneIDs) {
            // Flush in-memory database change token to disk
            completion()
        }
    }
    operation.qualityOfService = .userInitiated
    
    database.add(operation)
}
```

接下来，您的应用会使用包含[`CKFetchRecordZoneChangesOperation`](https://developer.apple.com/documentation/cloudkit/ckfetchrecordzonechangesoperation)您刚刚收集的区域ID集的对象来执行以下操作：

1. 创建和更新任何已更改的记录
2. 删除不再存在的任何记录
3. 更新区域更改令牌

以下是一些获取区域更改的示例代码：

```swift
func fetchZoneChanges(database: CKDatabase, databaseTokenKey: String, zoneIDs: [CKRecordZoneID], completion: @escaping () -> Void) {
    
    // Look up the previous change token for each zone
    var optionsByRecordZoneID = [CKRecordZoneID: CKFetchRecordZoneChangesOptions]()
    for zoneID in zoneIDs {
        let options = CKFetchRecordZoneChangesOptions()
        options.previousServerChangeToken = … // Read change token from disk
            optionsByRecordZoneID[zoneID] = options
    }
    let operation = CKFetchRecordZoneChangesOperation(recordZoneIDs: zoneIDs, optionsByRecordZoneID: optionsByRecordZoneID)
    
    operation.recordChangedBlock = { (record) in
        print("Record changed:", record)
        // Write this record change to memory
    }
    
    operation.recordWithIDWasDeletedBlock = { (recordId) in
        print("Record deleted:", recordId)
        // Write this record deletion to memory
    }
    
    operation.recordZoneChangeTokensUpdatedBlock = { (zoneId, token, data) in
        // Flush record changes and deletions for this zone to disk
        // Write this new zone change token to disk
    }
    
    operation.recordZoneFetchCompletionBlock = { (zoneId, changeToken, _, _, error) in
        if let error = error {
            print("Error fetching zone changes for \(databaseTokenKey) database:", error)
            return
        }
        // Flush record changes and deletions for this zone to disk
        // Write this new zone change token to disk
    }
    
    operation.fetchRecordZoneChangesCompletionBlock = { (error) in
        if let error = error {
            print("Error fetching zone changes for \(databaseTokenKey) database:", error)
        }
        completion()
    }
    
    database.add(operation)
}
```

上面的代码有几条关于将更改写入内存然后将这些更改刷新到磁盘的注释。一般流程如下。

对于数据库：

* 当被告知区域删除时，将其写入内存。
* 当被告知数据库的新变更令牌时，将该令牌写入内存，然后：

  * 将所有内存中区域更改保留到磁盘（删除已删除的区域，并记录需要更改的区域列表），或
  * 将区域删除保留到磁盘，并获取所有已修改记录区域的更改。

  **注意：**   在获取数据库更改时，需要在获取数据库更改令牌之前保留所有收到的每个区域回调。

* 最后，将更新的数据库更改标记刷新到磁盘

同样，对于区域：

* 当被告知区域中的记录更改时，请将其写入内存。
* 当被告知区域的新变更令牌时，将该区域中的所有内存中记录更改以及该区域的更新变更令牌提交到磁盘。

### 存储记录元数据

要将本地数据存储中的记录与服务器上的记录相关联，您可能需要存储记录的元数据（记录名称，区域ID，更改标记，创建日期等）。有一个方便的方法[`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)，[encodeSystemFieldsWithCoder](https://developer.apple.com/library/prerelease/ios/documentation/CloudKit/Reference/CKRecord_class/index.html#//apple_ref/occ/instm/CKRecord/encodeSystemFieldsWithCoder)，可以帮助您进行系统领域做到这一点。您仍然需要单独处理自己的自定义字段。

以下是您的应用如何读取元数据以便在本地存储它的示例：

```swift
// obtain the metadata from the CKRecord
let data = NSMutableData()
let coder = NSKeyedArchiver.init(forWritingWith: data)
coder.requiresSecureCoding = true
record.encodeSystemFields(with: coder)
coder.finishEncoding()
 
// store this metadata on your local object
yourLocalObject.encodedSystemFields = data
```

根据您的本地数据向CloudKit发送更改时，您可以将本地缓存读回CloudKit对象并操作它们以便在CloudKit中存储：

```swift
// set up the CKRecord with its metadata
let coder = NSKeyedUnarchiver(forReadingWith: yourLocalObject.encodedSystemFields!)
coder.requiresSecureCoding = true
let record = CKRecord(coder: coder)
coder.finishDecoding()
// write your custom fields...
```

### 高级本地缓存

用户可以通过iCloud设置 - &gt;管理存储删除CloudKit服务器上的应用程序数据。您的应用程序需要优雅地处理此问题，如果它们不存在，请再次在服务器上重新创建区域和订阅。在这种情况下返回的具体错误是[`userDeletedZone`](https://developer.apple.com/documentation/cloudkit/ckerror/code/userdeletedzone)。

[WWDC2015](https://developer.apple.com/videos/play/wwdc2015/226/)的[高级NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/)演讲中概述的操作依赖系统是管理CloudKit操作的好方法，以便检查帐户和网络状态，并在适当的时间创建区域和订阅。

网络连接可能随时消失，因此请确保正确处理[`networkUnavailable`](https://developer.apple.com/documentation/cloudkit/ckerrorcode/ckerrornetworkunavailable)任何操作中的错误。

监视网络可访问性，并在网络再次可用时重试该操作。

### 也可以看看

在[WWDC 2016 CloudKit最佳实践视频中](https://developer.apple.com/videos/play/wwdc2016/231/)了解有关本地缓存的更多信息。

