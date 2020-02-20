# 测试您的CloudKit应用程序

通过使用不同的iCloud帐户在多个设备上运行CloudKit应用程序，为多个同步用户提供真实世界的测试。最初，在指定的测试设备上在开发或生产环境中测试您的CloudKit应用程序。

稍后将iOS或tvOS应用程序上传到iTunes Connect并使用生产环境测试您的应用程序。使用iTunes Connect，邀请内部测试人员（您团队的iTunes Connect用户）或邀请外部测试人员（仅指定他们的电子邮件地址的用户）来测试您的应用。测试人员使用TestFlight应用程序下载您的iOS或tvOS应用程序。请注意，通过TestFlight或商店分发的应用程序无法使用开发环境。要了解更多关于发布使用TestFlight您的应用程序，阅读分发您的应用程序中使用TestFlight（iOS版，tvOS，watchOS）在_应用程序分发指南_。

### 使用Ad Hoc Provisioning分发您的应用程序（iOS，tvOS）

临时配置允许您在不需要Xcode的情况下在测试设备上运行您的应用程序。在使用临时配置文件导出应用程序之前，请在开发者帐户中注册要用于测试的所有设备。临时分发使用您注册进行开发的相同设备池，因此您可以注册的设备数量有限。要注册多个测试设备，请阅读使用成员中心注册设备。

导出应用程序进行临时测试时，您可以选择开发或生产环境。如果尚未将架构部署到生产中，如将[开发架构部署到生产中所述](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/DeployingYourCloudKitApp/DeployingYourCloudKitApp.html#//apple_ref/doc/uid/TP40014987-CH10-SW3)，请选择开发环境。

**归档和导出您的应用以进行临时测试**

1. 从“方案”工具栏菜单中选择一个通用设备或连接到Mac的设备，然后单击“运行”。

   您无法创建为模拟器构建的应用程序的存档。

2. 选择产品&gt;存档。

   Archives组织器出现并显示新存档。

3. 在Archives管理器中，选择存档，然后单击“导出”。
4. 选择“Save for Ad Hoc Deployment”，然后单击“下一步”。

   ![../Art/8\_save\_ad\_hoc\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/8_save_ad_hoc_2x.png)

5. 在出现的对话框中，从弹出菜单中选择一个团队，然后单击“选择”。

   如有必要，Xcode会为您创建分发证书和临时配置文件。

6. 在“设备支持”对话框中，选择是否导出通用应用程序或特定设备的变体，然后单击“下一步”。
7. 在下一个对话框中，从弹出菜单中选择容器环境，然后单击“下一步”。

   * 选择生产以访问生产环境中的数据。
   * 选择“开发”以访问开发环境中的数据。

   ![../Art/8\_choose\_environment\_2x.png](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/Art/8_choose_environment_2x.png)

8. 在显示的对话框中，查看应用程序，其权利及其配置文件，然后单击“导出”。

   临时供应配置文件的名称以文本开头`XC Ad Hoc:`。

9. 输入iOS App文件的文件名和位置，然后单击“导出”。

   Finder显示导出的文件。iOS App文件有`.ipa`扩展名。

稍后，将iOS App文件发送给测试人员。指示他们使用iTunes在其设备上安装应用程序，如在测试设备上安装应用程序中所述。

### 使用团队配置文件（Mac）分发您的应用程序

在使用Xcode导出应用程序之前，请在开发者帐户中注册要用于测试的所有Mac计算机。要将多台测试Mac计算机添加到团队配置文件，请参阅使用成员中心注册设备。添加Mac计算机后，通过刷新Xcode中的配置文件重新生成团队配置文件，如在Xcode中刷新配置文件中所述。

**使用团队配置文件导出代码签名的应用程序**

1. 从“方案”工具栏菜单中选择目标，然后单击“运行”。
2. 选择产品&gt;存档。

   Archives组织器出现并显示新存档。

3. 在Archives管理器中，选择存档，然后单击“导出”。
4. 选择“导出开发签名的应用程序”，然后单击“下一步”。

   Xcode将团队配置文件嵌入到包中，并使用您的开发证书对应用程序进行代码签名。

5. 在出现的对话框中，从弹出菜单中选择一个团队，然后单击“选择”。

   如有必要，Xcode会为您创建所需的签名身份和配置文件。

6. 在显示的对话框中，查看应用程序，其权利和配置文件，然后单击“导出”。

   Finder显示导出的文件。

稍后，将应用程序分发给测试人员并在指定的Mac计算机上运行。该应用程序仅在团队配置文件中指定的Mac计算机上启动。

如果您无法在Mac上启动应用程序，因为该应用程序来自身份不明的开发人员，请绕过OS X中的安全设置以启动该应用程序。

**从未知身份的开发人员打开应用程序**

1. 在Finder中，按住Control键并单击应用程序图标。
2. 单击打开。
3. 在出现的Gatekeeper对话框中，单击“打开”。

### 概括

在本章中您学到了：

* 如何使用不同的方法分发您的CloudKit应用程序以进行测试
* 如何限制您的应用在指定设备上运行
* 如何选择开发或生产容器进行测试

