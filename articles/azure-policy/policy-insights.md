---
title: ポリシーをプログラムで作成して Azure Policy でコンプライアンス データを表示する | Microsoft Docs
description: この記事では、Azure Policy のポリシーをプログラムで作成して管理する方法について説明します。
services: azure-policy
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 03/28/2018
ms.topic: article
ms.service: azure-policy
manager: carmonm
ms.custom: ''
ms.openlocfilehash: 1809f0b7ef386bb9eeaa55982178e4cd5e1dd2e2
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2018
---
# <a name="programmatically-create-policies-and-view-compliance-data"></a>ポリシーをプログラムで作成してコンプライアンス データを表示する

この記事では、ポリシーをプログラムで作成して管理する方法について説明します。 また、リソースのコンプライアンスの状態とポリシーを表示する方法も示します。 ポリシー定義は、リソースに対してさまざまなルールやアクションを適用します。 適用することで、リソースは会社の標準やサービス レベル アグリーメントに準拠した状態で維持されます。

## <a name="prerequisites"></a>前提条件

開始する前に、次の前提条件が満たされていることを確認します。

1. [ARMClient](https://github.com/projectkudu/ARMClient) をまだインストールしていない場合はインストールします。 Azure Resource Manager ベースの API に HTTP 要求を送信するツールです。
2. AzureRM PowerShell モジュールを最新バージョンに更新します。 最新バージョンの詳細については、Azure PowerShell (https://github.com/Azure/azure-powershell/releases) を参照してください。
3. サブスクリプションがリソース プロバイダーで確実に動作するように、Azure PowerShell を使用して Policy Insights リソース プロバイダーを登録します。 リソース プロバイダーを登録するには、リソース プロバイダーのアクションの登録操作を実行するためのアクセス許可が必要です。 この操作は、共同作成者ロールと所有者ロールに含まれます。 リソース プロバイダーを登録する以下のコマンドを実行します。

    ```
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.PolicyInsights
    ```

    リソース プロバイダーの登録と表示の詳細については、「[リソース プロバイダーと種類](../azure-resource-manager/resource-manager-supported-services.md)」を参照してください。

4. Azure CLI をまだインストールしていない場合はインストールします。 最新バージョンは、「[Install Azure CLI 2.0 on Windows](/azure/install-azure-cli-windows?view=azure-cli-latest)」 (Windows に Azure CLI 2.0 をインストールする) で入手できます。

## <a name="create-and-assign-a-policy-definition"></a>ポリシー定義を作成して割り当てる

リソースの可視性を向上させるには、まず、ポリシーを作成してリソースに割り当てます。 次の手順では、ポリシーをプログラムで作成して割り当てる方法について説明します。 このサンプル ポリシーは、すべてのパブリック ネットワークに開かれているストレージ アカウントを、PowerShell、Azure CLI、および HTTP 要求を使用して監査します。

次のコマンドは、Standard レベルのポリシー定義を作成します。 Standard レベルでは、大規模な管理、コンプライアンス評価、修復を実行できます。 価格レベルの詳細については、「[Azure Policy の価格](https://azure.microsoft.com/pricing/details/azure-policy)」を参照してください。

### <a name="create-and-assign-a-policy-definition-with-powershell"></a>PowerShell でポリシー定義を作成して割り当てる

1. 次の JSON スニペットを使用して、AuditStorageAccounts.jsonという名前の JSON ファイルを作成します。

    ```
    {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
          "equals": "Allow"
        }
      ]
    },
    "then": {
      "effect": "audit"
    }
  }

    ```

    ポリシー定義の作成方法の詳細については、「[Azure Policy の定義の構造](policy-definition.md)」を参照してください。

2. AuditStorageAccounts.json ファイルを使用し、次のコマンドを実行してポリシー定義を作成します。

    ```
    PS C:\>New-AzureRmPolicyDefinition -Name "AuditStorageAccounts" -DisplayName "Audit Storage Accounts Open to Public Networks" -Policy C:\AuditStorageAccounts.json
    ```

    このコマンドは、_Audit Storage Accounts Open to Public Networks_ という名前のポリシー定義を作成します。 使用できるその他のパラメーターの詳細については、「[New-AzureRmPolicyDefinition](/powershell/module/azurerm.resources/new-azurermpolicydefinition?view=azurermps-4.4.1)」を参照してください。

3. ポリシー定義を作成したら、次のコマンドを実行してポリシー割り当てを作成できます。

    ```
$rg = Get-AzureRmResourceGroup -Name "ContosoRG"
```

    ```
$Policy = Get-AzureRmPolicyDefinition -Name "AuditStorageAccounts"
    ```

    ```
New-AzureRmPolicyAssignment -Name "AuditStorageAccounts" -PolicyDefinition $Policy -Scope $rg.ResourceId –Sku @{Name='A1';Tier='Standard'}
    ```

    _ContosoRG_ を対象とするリソース グループの名前に置き換えます。

Azure Resource Manager PowerShell モジュールを使用したリソース ポリシーの管理の詳細については、「[AzureRM.Resources](/powershell/module/azurerm.resources/?view=azurermps-4.4.1#policies)」をご覧ください。

### <a name="create-and-assign-a-policy-definition-using-armclient"></a>ARMClient を使用してポリシー定義を作成して割り当てる

次の手順に従って、ポリシー定義を作成します。

1. 次の JSON スニペットをコピーして JSON ファイルを作成します。 このファイルは次の手順で呼び出します。

    ```
    {
    "properties": {
        "displayName": "Audit Storage Accounts Open to Public Networks",
        "policyType": "Custom",
        "mode": "Indexed",
        "description": "This policy ensures that storage accounts with exposure to Public Networks are audited.",
        "parameters": {},
        "policyRule": {
              "if": {
                "allOf": [
                  {
                    "field": "type",
                    "equals": "Microsoft.Storage/storageAccounts"
                  },
                  {
                    "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                    "equals": "Allow"
                  }
                ]
              },
              "then": {
                "effect": "audit"
              }
            }
    }
}
```

2. 次の呼び出しを使用して、ポリシー定義を作成します。

    ```
    armclient PUT "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2016-12-01 @<path to policy definition JSON file>"
    ```

    前の &lt;subscriptionId&gt; を対象のサブスクリプションの ID に置き換えます。

クエリの構造の詳細については、「[Policy Definitions – Create or Update (ポリシー定義 - 作成または更新)](/rest/api/resources/policydefinitions/createorupdate)」をご覧ください。


次の手順を使用してポリシー割り当てを作成し、ポリシー定義をリソース グループ レベルに割り当てます。

1. 次の JSON スニペットをコピーして、ポリシー割り当ての JSON ファイルを作成します。 例の &lt;&gt; 記号内の情報を独自の値に置き換えます。

    ```
    {
  "properties": {
"description": "This policy assignment makes sure that storage accounts with exposure to Public Networks are audited.",
"displayName": "Audit Storage Accounts Open to Public Networks Assignment",
"parameters": {},
"policyDefinitionId":"/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks",
"scope": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
},
"sku": {
    "name": "A1",
    "tier": "Standard"
    }
}
    ```

2. 次の呼び出しを使用して、ポリシー割り当てを作成します。

    ```
    armclient PUT "/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Authorization/policyAssignments/Audit Storage Accounts Open to Public Networks?api-version=2017-06-01-preview" @<path to Assignment JSON file>
    ```

    例の &lt;&gt; 記号内の情報を独自の値に置き換えます。

 REST API への HTTP 呼び出しを作成する方法については、[Azure REST API のリソース](/rest/api/resources/)に関するページを参照してください。

### <a name="create-and-assign-a-policy-definition-with-azure-cli"></a>Azure CLI でポリシー定義を作成して割り当てる

次の手順に従って、ポリシー定義を作成します。

1. 次の JSON スニペットをコピーして、ポリシー割り当ての JSON ファイルを作成します。

    ```
    {
                  "if": {
                    "allOf": [
                      {
                        "field": "type",
                        "equals": "Microsoft.Storage/storageAccounts"
                      },
                      {
                        "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                        "equals": "Allow"
                      }
                    ]
                  },
                  "then": {
                    "effect": "audit"
                  }
    }
    ```

2. 次のコマンドを実行してポリシー定義を作成します。

    ```
az policy definition create --name 'audit-storage-accounts-open-to-public-networks' --display-name 'Audit Storage Accounts Open to Public Networks' --description 'This policy ensures that storage accounts with exposures to public networks are audited.' --rules '<path to json file>' --mode All
    ```

次のコマンドを使用して、ポリシー割り当てを作成します。 例の &lt;&gt; 記号内の情報を独自の値に置き換えます。

```
az policy assignment create --name '<Audit Storage Accounts Open to Public Networks in Contoso RG' --scope '<scope>' --policy '<policy definition ID>' --sku 'standard'
```

ポリシー定義 ID は、PowerShell で次のコマンドを実行して取得できます。

```
az policy definition show --name 'Audit Storage Accounts with Open Public Networks'
```

作成したポリシー定義のポリシー定義 ID は、次の例のようになります。

```
"/subscription/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks"
```

Azure CLI を使用してリソース ポリシーを管理する方法の詳細については、[Azure CLI リソース ポリシー](/cli/azure/policy?view=azure-cli-latest)に関する記事をご覧ください。

## <a name="identify-non-compliant-resources"></a>準拠していないリソースを特定する

割り当てのリソースがポリシーや統括ルールに従っていない場合、そのリソースは非準拠になります。 次の表は、結果のコンプライアンスの状態に対する条件の評価が、さまざまなポリシーのアクションでどのように処理されるかを示しています。

| **リソースの状態** | **アクション** | **ポリシーの評価** | **コンプライアンスの状態** |
| --- | --- | --- | --- |
| Exists | Deny、Audit、Append\*、DeployIfNotExist\*、AuditIfNotExist\* | True | 非準拠 |
| Exists | Deny、Audit、Append\*、DeployIfNotExist\*、AuditIfNotExist\* | False | 対応 |
| 新規 | Audit、AuditIfNotExist\* | True | 非準拠 |
| 新規 | Audit、AuditIfNotExist\* | False | 対応 |

\* Append、DeployIfNotExist、および AuditIfNotExist の各アクションでは、IF ステートメントが TRUE であることが要求されます。 また、非準拠となるには、既存の条件が FALSE であることが要求されます。 TRUE のとき、IF 条件は関連するリソースの既存の条件の評価をトリガーします。

リソースがどのようにして非準拠としてフラグされるかについて理解を深めるために、前に作成したポリシー割り当ての例を見てみましょう。

たとえば、ContosoRG というリソース グループがあり、このリソース グループの一部のストレージ アカウント (赤で強調表示されているアカウント) がパブリック ネットワークに公開されているとします。

![パブリック ネットワークに公開されているストレージ アカウント](./media/policy-insights/resource-group01.png)

この例では、セキュリティ リスクに注意する必要があります。 これで作成されたポリシー割り当てが ContosoRG リソース グループ内のすべてのストレージ アカウントに対して評価されます。 3 つの非準拠のストレージ アカウントを監査し、結果としてそれらの状態を**非準拠**に変更します。

![監査された非準拠のストレージ アカウント](./media/policy-insights/resource-group03.png)

次の手順を使用して、ポリシー割り当てに準拠していないリソース グループ内のリソースを特定します。 この例では、リソースは ContosoRG リソース グループ内のストレージ アカウントです。

1. 次のコマンドを実行して、ポリシー割り当て ID を入手します。

    ```
    $policyAssignment = Get-AzureRmPolicyAssignment | where {$_.properties.displayName -eq "Audit Storage Accounts with Open Public Networks"}
    ```

    ```
    $policyAssignment.PolicyAssignmentId
    ```

    ポリシー割り当て ID の入手方法の詳細については、「[Get-AzureRMPolicyAssignment](https://docs.microsoft.com/en-us/powershell/module/azurerm.resources/Get-AzureRmPolicyAssignment?view=azurermps-4.4.1)」を参照してください。

2. 次のコマンドを実行して、非準拠リソースの ID を JSON ファイルにコピーします。

    ```
    armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2017-12-12-preview&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
    ```

3. 結果は次の例のようになります。

  ```
      {
  "@odata.context":"https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
  "@odata.count": 3,
  "value": [
  {
      "@odata.id": null,
      "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.storage/storageaccounts/<storageaccount1Id>"
      },
      {
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.storage/storageaccounts/<storageaccount2Id>"
             },
  {
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.storage/storageaccounts/<storageaccount3ID>"
             }
  ]
  }
  ```

結果は、普段 [Azure Portal ビュー](assign-policy-definition.md#identify-non-compliant-resources)の **[準拠していないリソース]** に一覧表示される内容と同じです。

現時点では、非準拠のリソースは Azure Portal および HTTP 要求を使用してのみ特定できます。 ポリシーの状態についてクエリを実行する方法の詳細については、[ポリシーの状態](/rest/api/policy-insights/policystates)に関する API リファレンスの記事を参照してください。

## <a name="view-policy-events"></a>ポリシー イベントを表示する

リソースを作成または更新すると、ポリシーの評価結果が生成されます。 これらの結果は "_ポリシー イベント_" と呼ばれます。 次のクエリを実行して、ポリシー割り当てに関連付けられているすべてのポリシー イベントを表示します。

```
armclient POST "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks/providers/Microsoft.PolicyInsights/policyEvents/default/queryResults?api-version=2017-12-12-preview"
```

次のような結果が返されます。

```
{
  "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default",
  "@odata.count": 1,
  "value": [
    {
      "@odata.id": null,
      "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default/$entity",
      "NumAuditEvents": 3
    }
  ]
}

```

ポリシーの状態と同様に、ポリシー イベントも HTTP 要求でのみ表示できます。 ポリシー イベントについてクエリを実行する方法の詳細については、[ポリシー イベント](/rest/api/policy-insights/policyevents)に関する API リファレンスの記事を参照してください。

## <a name="change-a-policy-assignments-pricing-tier"></a>ポリシー割り当ての価格レベルを変更する

*Set-AzureRmPolicyAssignment* PowerShell コマンドレットを使用して、既存のポリシー割り当ての価格レベルを Standard または Free に更新できます。 例: 

```
Set-AzureRmPolicyAssignment -Id /subscriptions/<subscriptionId/resourceGroups/<resourceGroupName>/providers/Microsoft.Authorization/policyAssignments/<policyAssignmentID> -Sku @{Name='A1';Tier='Standard'}
```

コマンドレットの詳細については、「[Set-AzureRmPolicyAssignment](/powershell/module/azurerm.resources/Set-AzureRmPolicyAssignment?view=azurermps-4.4.1)」を参照してください。

## <a name="next-steps"></a>次の手順

この記事のコマンドとクエリの詳細については、次の記事をご覧ください。

- [Azure REST API リソース](/rest/api/resources/)
- [Azure RM PowerShell モジュール](/powershell/module/azurerm.resources/?view=azurermps-4.4.1#policies)
- [Azure CLI Policy コマンド](/cli/azure/policy?view=azure-cli-latest)
- [Policy Insights Resource Provider REST API リファレンス](/rest/api/policy-insights)
