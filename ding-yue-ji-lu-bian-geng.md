# 订阅记录变更

当结果与上一个查询大致相同时，您的应用重复查询效率很低。更高效的方法是订阅记录更改，让服务器在后台运行查询。服务器将通知您的应用程序对用户或应用程序感兴趣的更改。例如，如果您的应用的一个用户对某位艺术家的作品感兴趣，则可以在上传该艺术家的新作品时通知您的应用。

![../Art/subscriptions\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/subscriptions_2x.png)

### 将订阅保存到数据库

在您的代码中，创建一个订阅对象，指定要通知的记录类型、谓词和更改类型。然后将订阅对象保存到数据库。

**创建和保存订阅**

1. 创建谓词对象。例如，订阅艺术家的艺术作品（其中记录类型中的`artist`字段`Artwork`是一种`Reference`类型）。

   ```swift
   CKRecordID *artistRecordID = [[CKRecordID alloc] initWithRecordName:@"Mei Chen"];
   NSPredicate *predicate = [NSPredicate predicateWithFormat:@"artist = %@", artistRecordID];
   ```

   **注：**  在谓语格式字符串参数右边的表达可能的值[`CKRecord`](https://developer.apple.com/documentation/cloudkit/ckrecord)`、`[`CKRecordID`](https://developer.apple.com/documentation/cloudkit/ckrecordid)以及[`CKReference`](https://developer.apple.com/documentation/cloudkit/ckreference)对象。如果您知道记录名称，则可以创建仅包含记录名称的记录ID。

2. 创建指定记录类型、谓词和通知选项的订阅。

   ```swift
   CKSubscription *subscription = [[CKSubscription alloc]
                                       initWithRecordType:@"Artwork"
                                       predicate:predicate
                                       options:CKSubscriptionOptionsFiresOnRecordCreation];
   ```

   对于可能的值`options`参数有：[`CKSubscriptionOptionsFiresOnRecordCreation`](https://developer.apple.com/documentation/cloudkit/cksubscriptionoptions/cksubscriptionoptionsfiresonrecordcreation)，[`CKSubscriptionOptionsFiresOnRecordDeletion`](https://developer.apple.com/documentation/cloudkit/cksubscriptionoptions/cksubscriptionoptionsfiresonrecorddeletion)，[`CKSubscriptionOptionsFiresOnRecordUpdate`](https://developer.apple.com/documentation/cloudkit/cksubscriptionoptions/cksubscriptionoptionsfiresonrecordupdate)，和[`CKSubscriptionOptionsFiresOnce`](https://developer.apple.com/documentation/cloudkit/cksubscriptionoptions/cksubscriptionoptionsfiresonce)。由于`options`参数是位掩码，因此您可以订阅更改类型的任意组合。例如，你可以传递`CKSubscriptionOptionsFiresOnRecordCreation | CKSubscriptionOptionsFiresOnRecordUpdate`的`options:`参数来接收所有新数据的通知。

3. 创建CloudKit通知对象。

   ```swift
   CKNotificationInfo *notificationInfo = [CKNotificationInfo new];
   notificationInfo.alertLocalizationKey = @"New artwork by your favorite artist.";
   notificationInfo.shouldBadge = YES;
   ```

   要向用户显示本地化字符串，请设置通知的`alertLocalizationKey`属性（而不是`alertBody`属性）。

4. 将订阅的通知对象设置为新的CloudKit通知对象。

   ```swift
   subscription.notificationInfo = notificationInfo;。
   ```

5. 将订阅保存到数据库。

   ```swift
   CKDatabase *publicDatabase = [[CKContainer defaultContainer] publicCloudDatabase];
   [publicDatabase saveSubscription:subscription
      completionHandler:^(CKSubscription *subscription, NSError *error) {
         if (error)
            // insert error handling
       }
    ];
   ```

在Xcode中，运行您的应用程序以保存对数据库的订阅。

### 注册推送通知

保存订阅数据库不会自动将应用程序配置为在订阅触发时接收通知。CloudKit使用[Apple推送通知服务（APN）](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Glossary/Glossary.html#//apple_ref/doc/uid/TP40014987-CH11-SW4)向您的应用发送订阅通知，因此您的应用需要注册推送通知以接收它们。

对于iOS和tvOS应用程序，将此代码添加到[`application:didFinishLaunchingWithOptions:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application)协议方法以注册推送通知：

```swift
// Register for push notifications
UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert categories:nil];
[application registerUserNotificationSettings:notificationSettings];
[application registerForRemoteNotifications];
```

对于Mac应用程序，请实施[`applicationDidFinishLaunching:`](https://developer.apple.com/documentation/appkit/nsapplicationdelegate/1428385-applicationdidfinishlaunching)协议方法以注册推送通知。

（可选）实现[`application:didRegisterForRemoteNotificationsWithDeviceToken:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application)和[`application:didFailToRegisterForRemoteNotificationsWithError:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622962-application)方法，以便在应用程序成功或不成功注册推送通知时采取适当的操作。

**注意：**  您无需在开发者帐户中为应用的显式应用ID启用推送通知即可接收订阅通知。启用CloudKit时，Xcode会自动将APN权利添加到您的权利文件中。

### 处理代码中的推送通知

接下来，实现该[`application:didReceiveRemoteNotification:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623117-application)方法在到达时处理订阅通知。对于iOS和tvOS应用程序，实现[`UIApplicationDelegate`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)协议方法，对于Mac应用程序，实现[`NSApplicationDelegate`](https://developer.apple.com/documentation/appkit/nsapplicationdelegate)协议方法。例如，实现此方法可在创建，更新或删除与谓词匹配的记录时更新视图。

1. 将[`application:didReceiveRemoteNotification:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623117-application)协议方法添加到应用程序的委托。

   ```swift
   - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
   }
   ```

2. 在该[`application:didReceiveRemoteNotification:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623117-application)方法中，将`userInfo`参数转换为[`CKNotification`](https://developer.apple.com/documentation/cloudkit/cknotification)对象。

   ```swift
   CKNotification *cloudKitNotification = [CKNotification notificationFromRemoteNotificationDictionary:userInfo];
   ```

3. 获取通知的正文。

   ```swift
   NSString *alertBody = cloudKitNotification.alertBody;
   ```

4. 从[`CKQueryNotification`](https://developer.apple.com/documentation/cloudkit/ckquerynotification)对象获取新记录或修改记录。

   ```swift
   if (cloudKitNotification.notificationType == CKNotificationTypeQuery) {
      CKRecordID *recordID = [(CKQueryNotification *)cloudKitNotification recordID];
   }
   ```

5. 更新视图或根据记录更改通知用户。

### 测试订阅

您最初可以通过Xcode运行应用程序并使用CloudKit Dashboard创建，修改或删除记录来测试订阅，如[添加，修改和删除记录中所述](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/EditingSchemesUsingCloudKitDashboard/EditingSchemesUsingCloudKitDashboard.html#//apple_ref/doc/uid/TP40014987-CH5-SW5)。然后通过在多个设备上运行您的应用程序来完全测试订阅。使用一个设备进行更改，使用另一个设备接收订阅通知。您使用多个设备，因为通知不会发送到发起通知的同一设备。

对于iOS和tvOS，请使用连接到Mac（不是模拟器）的设备来测试订阅通知。如果某个对话框询问用户是否允许您的应用接收通知，则您的应用会成功注册推送通知。

### 概括

在本章中，您学习了如何：

* 使用谓词订阅记录更改
* 处理订阅通知

