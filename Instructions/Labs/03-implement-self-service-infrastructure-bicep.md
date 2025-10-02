---
lab:
  title: Bicep を使用したセルフサービス インフラストラクチャの実装
  module: Strategic Platform Road Mapping
---

# ラボ 03: Bicep を使用したセルフサービス インフラストラクチャの実装

## 推定時間:20 分

## 前提条件

- Azure サブスクリプション - Azure サブスクリプションをお持ちでない場合は、[無料アカウント](https://azure.microsoft.com/free/)を作成してください。
- Azure サービスと Azure CLI に関する基本的な知識。
- Bicep 拡張機能を使用してインストールした Visual Studio Code。
- ローカル コンピューターにインストールされ、構成されている Azure CLI。
- コードとしてのインフラストラクチャ (IaC) の概念に関する知識。

## 目標

- 環境に Bicep を設定する。
- Bicep テンプレートを作成して、Azure リソースを定義する。
- SQL Database バックエンドを使用して Azure App Service をデプロイする。
- タグとポリシーを使用してガバナンスを適用する。
- Bicep を使用して自動スケーリングを実装する。

## 演習 1: Bicep テンプレートを作成して Azure リソースをデプロイする

プラットフォーム エンジニアリング環境では、開発者はインフラストラクチャを一貫して効率的にプロビジョニングする方法が必要です。 このラボでは、コードとしてのインフラストラクチャ (IaC) ツールである Bicep を使用して、セルフサービス方式で Azure リソースをデプロイおよび管理する方法について説明します。 リソース グループ、Azure App Service、Azure SQL Database、ストレージ アカウントをプロビジョニングする Bicep テンプレートを作成し、タグ付けとポリシーを使用してガバナンスを適用します。

### タスク 1: Bicep CLI をインストールする

1. ローカル ターミナルを開きます。
1. Azure アカウントにログインします。

   ```bash
   az login
   ```

   > **注:** プロンプトに従って、Azure アカウントで認証を行います。 これにより、認証用に Web ブラウザーが開きます。

1. Bicep がインストールされていることを確認するには、次を実行します。

   ```bash
   az bicep version
   ```

   Bicep がインストールされていない場合は、次を使用してインストールします。

   ```bash
   az bicep install
   ```

1. 次を実行してインストールを確認します。

   ```bash
   az bicep version
   ```

   「Bicep CLI version X.Y.Z.」のような出力が表示されます。

   > **注:** [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) がインストールされたことを確認してください。

### タスク 2: Bicep テンプレートを作成する

1. ローカル ターミナルで、新しいファイルを作成します。

   ```bash
   code main.bicep
   ```

1. 以下の Bicep コードをコピーし、次のファイルに貼り付けます。

   ```bicep
   param location string = 'eastus2'
   param appServicePlanName string = 'bicepAppPlan'
   param webAppName string = 'bicep-webapp'
   param storageAccountName string = 'biceplabstorage'
   param sqlServerName string = 'bicep-sqlserver'
   param sqlDatabaseName string = 'bicepdb'
   param sqlAdminUser string
   @secure()
   param sqlAdminPassword string

   targetScope = 'resourceGroup'

   resource storage 'Microsoft.Storage/storageAccounts@2021-09-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'StorageV2'
   }

   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }

   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
     name: webAppName
     location: location
     properties: {
       serverFarmId: appPlan.id
     }
   }

   resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
     name: sqlServerName
     location: location
     properties: {
       administratorLogin: sqlAdminUser
       administratorLoginPassword: sqlAdminPassword
       version: '12.0'
     }
   }

   resource sqlDb 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     name: sqlDatabaseName
     location: location
     parent: sqlServer
     properties: {
       collation: 'SQL_Latin1_General_CP1_CI_AS'
       maxSizeBytes: 2147483648
     }
   }
   ```

   > **注:** Visual Studio Code に Bicep 拡張機能をインストールすると、Bicep ファイルの構文を強調表示して IntelliSense を取得できます。

   > **重要:** storageAccountName、webAppName、sqlServerName がグローバルに一意であることを確認してください。 名前の競合が原因でデプロイが失敗した場合は、それに応じて名前を変更します。

   > **注:** リージョンに関連するエラーが発生した場合は、`location` パラメーター値を変更して別のリージョンにデプロイしてみてください。

1. ファイルを保存して閉じます。

### タスク 3: Azure CLI を使用してテンプレートをデプロイする

1. 次のコマンドを実行してリソース グループを作成します。

   ```bash
   az group create --name BicepLab-RG --location centralus

   ```

   > **注:** まだ認証されていない場合は、Azure アカウントへのログインが必要になる場合があります。 詳細な手順については、「[Azure CLI を使用した Azure への認証](https://learn.microsoft.com/cli/azure/authenticate-azure-cli)」を参照してください。

2. テンプレートをリソース グループ内にデプロイします。

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

   > **注:** YourSecurePassword123! を 強力なパスワードに置き換え、azureuser を有効なユーザー名に置き換えます。

3. デプロイが完了するまで待ちます。 確認メッセージが表示されます。
4. Azure portal で、リソース グループに移動します。
5. BicepLab-RG を探してクリックします。
6. リソースが正常に作成されたことを確認します。

Bicep テンプレートを使用して、SQL Database バックエンドで Azure App Service を正常にデプロイしました。 コードとしてのインフラストラクチャを管理し、リソースを一貫して効率的にデプロイできるようになりました。 次の手順では、このテンプレートを CI/CD パイプラインに統合して、自動デプロイを行います。

## 演習 2: タグとポリシーを使用してガバナンスを適用する

セルフサービス インフラストラクチャ環境では、コンプライアンスとコスト管理を確保するためにガバナンスを適用することが不可欠です。 タグとポリシーは、これを実現するための 2 つの重要なメカニズムです。

### タスク 1: リソースにタグを追加する

1. 次のコマンドを実行して、リソース グループにタグを追加します。

   ```bash
   az group update --name BicepLab-RG --set tags.Owner='PlatformEngineering'
   ```

1. 次を実行して、タグが追加されたことを確認します。

   ```bash
   az group show --name BicepLab-RG --query tags
   ```

### タスク 2: タグ付けを適用するポリシーを作成する

1. 次に JSON ポリシー ファイルを作成します。

   ```bash
   code tagging-policy.json
   ```

1. 次のポリシー定義をコピーして、ファイルに貼り付けます。

   ```json
   {
     "if": {
       "not": {
         "field": "tags[Owner]",
         "exists": "true"
       }
     },
     "then": {
       "effect": "deny"
     }
   }
   ```

1. 次の JSON ファイルを使用して、ポリシー定義を作成します。

   ```bash
    az policy definition create --name 'tagging-policy' --display-name 'Enforce tagging' --rules @tagging-policy.json --mode All
   ```

1. そのポリシーをリソース グループに割り当てます。

   ```bash
   az policy assignment create --name 'tagging-policy-assignment' --display-name 'Enforce tagging' --policy "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policyDefinitions/tagging-policy" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG"

   ```

   > **重要:** 両方の <subscription-id> を、使用している Azure サブスクリプション ID に置き換えます。

1. 次を実行して、ポリシーが割り当てられたことを確認します。

   ```bash
   az policy assignment list --output table
   ```

### タスク 3: ポリシーの適用をテストする

1. 次のコマンドを実行して、リソース グループからタグを削除します。

   ```bash
   az tag update --resource-id /subscriptions/<subscription-id>/resourceGroups/BicepLab-RG --operation delete --tags Owner
   ```

   > **重要:** <subscription-id> を、使用している Azure サブスクリプション ID に置き換えます。

1. ポリシーの適用により操作が拒否されたことを示すエラー メッセージが表示されます。 タグを削除しないでください。 これにより、ポリシーが予期したとおりに動作したことを確認できます。 エラーを解決するには、ポリシー割り当てを一時的に削除し、コマンドを再度実行します。

   ```bash
   az policy exemption create --name 'tagging-policy-exemption' --policy-assignment "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG/providers/Microsoft.Authorization/policyAssignments/tagging-policy-assignment" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG" --display-name "Temporary exemption to remove tag" --exemption-category Waiver
   ```

   > **重要:** 両方の <subscription-id> を、使用している Azure サブスクリプション ID に置き換えます。

タグとポリシーを使用してガバナンスを正常に適用しました。 開発者は、所有者タグを使用してリソースにタグを付け、適切な所有権と説明責任を確保する必要があります。 リソースの名前付け規則、リソースの種類、リソースの場所など、他のガバナンス要件を適用する追加のポリシーを作成できます。

## 演習 3: Bicep を使用して自動スケーリングを実装する

プラットフォーム エンジニアリング環境では、アプリケーションを効率的にスケーリングできるようにすることが重要な要件です。 自動スケーリングにより、パフォーマンスが向上し、リソース使用率が最適化され、コストが最小限に抑えられます。 この演習では、Bicep を使用して Azure App Service の自動スケールを実装します。

### タスク 1: Bicep で自動スケールのポリシーを定義する

1. main.bicep ファイルを開き、既存の App Service プランのセクションを探します。

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }
   ```

1. S1 SKU (自動スケールをサポート) を使用するように appPlan リソースを変更します。

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'S1'
       tier: 'Standard'
     }
     properties: {
       perSiteScaling: false
       maximumElasticWorkerCount: 10
     }
   }
   ```

1. このリソースの直後に、autoscaleSetting 構成を追加します。

   ```bicep
   resource autoscaleSetting 'Microsoft.Insights/autoscaleSettings@2022-10-01' = {
   name: 'autoscale-rule'
   location: location
   properties: {
    profiles: [
      {
        name: 'defaultProfile'
        capacity: {
          minimum: '1'
          maximum: '5'
          default: '1'
        }
        rules: [
          {
            metricTrigger: {
              metricName: 'CpuPercentage'
              metricResourceUri: resourceId('Microsoft.Web/serverfarms', appServicePlanName)
              operator: 'GreaterThan'
              threshold: 75
              timeAggregation: 'Average'
              statistic: 'Average'
              timeWindow: 'PT5M'
              timeGrain: 'PT1M'
            }
            scaleAction: {
              direction: 'Increase'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT1M'
            }
          }
        ]
      }
    ]
   }
   }
   ```

1. ファイルを保存します。
1. 更新された Bicep ファイルを確認します。

   ```bash
   az bicep build --file main.bicep
   ```

1. 更新したテンプレートをデプロイする:

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

1. 自動スケールのポリシーが App Service プランに適用されたことを確認します。 Azure portal に移動し、App Service プランに移動して [スケールアウト (App Service プラン)] ブレードを選択し、自動スケーリング ルールが構成されていることを確認します。

   > **注:** 自動スケールの動作をテストする場合は、ロード テストを実行するかトラフィックを生成することで、App Service で高い CPU 使用率をシミュレートできます。 App Service は、定義された規則に基づいて自動的にスケールアウトされます。

Bicep を使用して、Azure App Service の自動スケーリングを正常に実装しました。 これにより、アプリケーションでトラフィックと需要の増加に効率的に対処でき、パフォーマンスが向上し、コストが削減されます。
