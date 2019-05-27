## 创建一个Service Principal账户
选取两个不同的方法中的其中一个来创建一个Service Principle：
 - Azure Portal
 - Azure CLI
 
## 使用Azure Portal:
 - 在http://portal.azure.com 登录你的Azure账户
 - 进入`Azure Active Directory`下的`User Setting`。
 - 确保用户能够注册应用，只有一个admin用户可以注册应用。
 - 创建一个 `Azure Active目录应用` [(了解更多)](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal?view=azure-cli-latest).
 - 取回 `application ID` 和`authentication key`。
 - 将新创建的应用程序分配给角色“Contributor”，以便访问Azure资源。

 

 ## 使用Azure CLI:
 - 用如下的命令在Azure Active目录下创建一个应用
  `az ad sp create-for-rbac --name minio-mapp --password "XXXXXXXXX"`
 
  ```
  $ az ad sp create-for-rbac --name minio-mapp --password "XXXXXXXXX"
  Retrying role assignment creation: 1/36
  Retrying role assignment creation: 2/36
  Retrying role assignment creation: 3/36
  {
    "appId": "04e4c5c8-4c77-4147-b1d9-00a0226be5d3",
    "displayName": "minio-mapp",
    "name": "http://minio-mapp",
    "password": "XXXXXXXXX",
    "tenant": "XXxxXXxx-XXXx-XXXx-XXXx-xxxXXXXXxXxx"
  }
  ```
 - Azure中新应用中默认的角色选项是"Contributor"。一个"Contributor"角色也可以用下面的命令创建：

 
```
$ az role assignment create --assignee 04e4c5c8-4c77-4147-b1d9-00a0226be5d3 --role Contributor
{
  "id": "/subscriptions/XXxxXXxx-XXXx-XXXx-XXXx-xxxXXXXXxXxx/providers/Microsoft.Authorization/roleAssignments/1faa99e2-1ceb-48b0-92e9-9aba77078344",
  "name": "1faa99e2-1ceb-48b0-92e9-9aba77078344",
  "properties": {
    "additionalProperties": {
      "createdBy": null,
      "createdOn": "2018-01-18T04:18:03.5317841Z",
      "updatedBy": "XXxxXXxx-XXXx-XXXx-XXXx-xxxXXXXXxXxx",
      "updatedOn": "2018-01-18T04:18:03.5317841Z"
    },
    "principalId": "4ef772e9-bbe2-4ad8-8e9a-a97dd0084169",
    "roleDefinitionId": "/subscriptions/XXxxXXxx-XXXx-XXXx-XXXx-xxxXXXXXxXxx/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "scope": "/subscriptions/XXxxXXxx-XXXx-XXXx-XXXx-xxxXXXXXxXxx"
  },
  "type": "Microsoft.Authorization/roleAssignments"
}

```
