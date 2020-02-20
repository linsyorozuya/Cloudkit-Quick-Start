# 部署架构

在完成架构并在开发环境中测试应用程序之后，您就可以将架构部署到生产环境中了。部署将模式提升到生产环境，但不会将开发环境中的记录复制到生产环境。因此，在部署之后，根据需要使用记录填充生产环境。然后在生产环境中测试您的应用。您可以继续在开发环境中更改架构，但在部署架构后，您只能创建记录类型和添加字段。索引是架构的一部分，因此需要部署类似于记录类型更改。下次部署开发模式时，更改将与生产模式合并。

您必须具有编辑生产环境的权限才能执行本章中的任务。如果您是个人，则您是团队管理员并拥有这些权限。否则，请让您的团队管理员为您执行这些步骤，或授予您“编辑生产”权限，如[将权限分配给其他团队成员中所述](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/DeployingYourCloudKitApp/DeployingYourCloudKitApp.html#//apple_ref/doc/uid/TP40014987-CH10-SW1)。

部署架构后，应用程序上传到iTunes连接，如在上传您的应用程序iTunes Connect中的_应用发布指南_，并为iOS和tvOS应用，发布应用程式使用TestFlight测试，如在分发您的应用程序中使用TestFlight（ iOS，tvOS，watchOS）。当您准备好将应用程序提交到商店时，请阅读提交您的应用程序。

### 将开发模式部署到生产中

第一次部署应用程序时，CloudKit会将容器架构复制到生产环境中。这包括记录类型，安全角色和订阅类型，但不包括您在开发环境中创建的记录。将架构部署到生产环境后，无法删除在开发环境中部署的记录类型和字段。

**警告：**  要在生产中查看记录，请在执行[高级本地缓存中](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW9)描述的这些步骤之前为相关记录类型启用ID元数据索引。您无法在生产环境中更改元数据索引。

**将架构部署到生产**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，单击“部署到生产...”按钮。
2. 查看要部署的更改。
3. 单击“部署更改”。

#### 验证您的步骤

验证架构是否已复制到生产环境。

**查看生产架构和数据**

1. 单击生产环境中的数据。
2. 单击选项卡栏中的“记录类型”。
3. 选择要查看的记录类型。

### 将权限分配给其他团队成员

对于组织，您可以通过更改团队成员权限来委派部署CloudKit应用程序的一些职责，如下所示。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7279;&#x6743;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x7BA1;&#x7406;&#x56E2;&#x961F;</td>
      <td style="text-align:left">&#x53EF;&#x4EE5;&#x66F4;&#x6539;&#x9664;&#x56E2;&#x961F;&#x4EE3;&#x7406;&#x4E4B;&#x5916;&#x7684;&#x5176;&#x4ED6;&#x56E2;&#x961F;&#x6210;&#x5458;&#x7684;&#x6743;&#x9650;&#x3002;&#x56E2;&#x961F;&#x4EE3;&#x7406;&#x603B;&#x662F;&#x62E5;&#x6709;&#x6240;&#x6709;&#x6743;&#x9650;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7F16;&#x8F91;&#x5F00;&#x53D1;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x53EF;&#x4EE5;&#x4F7F;&#x7528;CloudKit&#x4EEA;&#x8868;&#x677F;&#x7F16;&#x8F91;&#x5F00;&#x53D1;&#x6A21;&#x5F0F;&#x3002;</li>
          <li>&#x53EF;&#x4EE5;&#x5728;&#x5F00;&#x53D1;&#x4E2D;&#x67E5;&#x770B;&#x8BB0;&#x5F55;&#x3002;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7F16;&#x8F91;&#x5236;&#x4F5C;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x53EF;&#x4EE5;&#x5C06;&#x5F00;&#x53D1;&#x6A21;&#x5F0F;&#x90E8;&#x7F72;&#x5230;&#x751F;&#x4EA7;&#x4E2D;&#x3002;</li>
          <li>&#x53EF;&#x4EE5;&#x67E5;&#x770B;&#x751F;&#x4EA7;&#x6A21;&#x5F0F;&#x3002;</li>
          <li>&#x53EF;&#x4EE5;&#x5728;&#x751F;&#x4EA7;&#x4E2D;&#x67E5;&#x770B;&#x548C;&#x7F16;&#x8F91;&#x8BB0;&#x5F55;&#x3002;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>您可以为每个容器单独设置团队成员权限。权限不适用于属于团队的所有容器。

**授予团队成员特权**

1. 在[CloudKit仪表板中](https://icloud.developer.apple.com/dashboard)，在适当的容器中，单击右上角的“容器权限...”。
2. 在团队成员和权限列的行中，选择要授予团队成员的权限。

   如果无法更改权限或您无权更改权限，则会禁用该复选框。

### 概括

在本章中，您学习了如何将开发模式部署到生产环境，以及如何在继续开发应用程序时使其保持最新。

