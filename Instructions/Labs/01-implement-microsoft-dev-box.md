---
lab:
  title: Microsoft Dev Box を実装する
  module: Implement Developer Self-Service
---

# ラボ 01: Microsoft Dev Box を実装する

## 推定時間:60 分

## 前提条件

- Microsoft Dev Box のデプロイに関係する 3 つの異なるロールを表す、事前に作成された 3 つのユーザー アカウント (および必要に応じて事前に作成された 3 つの Microsoft Entra グループ) を含む Microsoft Entra テナント。 わかりやすくするため、ラボの手順のユーザー名とグループ名は、次の表の情報と一致させています。

| User              | グループ                        | 役割                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | プラットフォーム エンジニア     |
| devlead01         | DevCenter_Dev_Leads          | 開発チーム リーダー |
| devuser01         | DevCenter_Dev_Users          | 開発者             |

- ユーザーとグループのアカウントをホストする Microsoft Entra テナントに関連付けられている Microsoft Dev Box リソースをホストする Azure サブスクリプション
- Azure サブスクリプションと同じ Microsoft Entra テナントに関連付けられている Microsoft Intune サブスクリプション
- 事前に作成された 3 つの Microsoft Entra ユーザー アカウントに割り当てられた Microsoft Intune ライセンス
- GitHub アカウントとリポジトリが必要になります。
  - GitHub アカウントを作成するには、「[GitHub でのアカウントの作成](https://docs.github.com/get-started/quickstart/creating-an-account-on-github)」の記事の手順に従います。
  - GitHub リポジトリを作成するには、「[新しいリポジトリの作成](https://docs.github.com/repositories/creating-and-managing-repositories/creating-a-new-repository)」の記事の手順に従います。
- https://github.com/microsoft/devcenter-catalog のフォークとして作成された GitHub リポジトリ
- https://github.com/MicrosoftLearning/contoso-co-eShop のフォークとして作成された GitHub リポジトリ

## 目標

このトピックを完了すると、以下ができるようになります。

- Microsoft Dev Box 環境を実装する
- Microsoft Dev Box 環境をカスタマイズする

## 演習 1: Microsoft Dev Box 環境を実装する

この演習では、Microsoft が提供する一連の機能を利用して、Microsoft Dev Box 環境を実装します。 このアプローチでは、機能的な開発者セルフサービス ソリューションの構築に伴う作業を最小限に抑えることに重点を置いています。

演習は次のタスクで構成されます。

- タスク 1: デベロッパー センターを作成する
- タスク 2: デベロッパー センターの設定を確認する
- タスク 3: 開発ボックス定義を作成する
- タスク 4: プロジェクトを作成する
- タスク 5: 開発ボックス プールを作成する
- タスク 6: アクセス許可を構成する
- タスク 7: 開発ボックスを評価する

### タスク 1: デベロッパー センターを作成する

このタスクでは、このラボの最初の演習全体で使用する Azure デベロッパー センターを作成します。 デベロッパー センターは、事前構成されたスケーラブルな開発環境およびデプロイ環境の作成と管理を一元化し、ソフトウェア開発チームのコラボレーションとリソース活用を最適化する、プラットフォーム エンジニアリング サービスです。

1. Web ブラウザーを開始し、Azure portal (`https://portal.azure.com`) にアクセスします。
1. 認証を求められたら、Microsoft Entra ユーザー アカウントを使用してサインインします。
1. Azure portal の **[検索]** テキスト ボックスで、「**`Dev centers`**」を検索して選択します。
1. **[デベロッパー センター]** ページで、**[+ 作成]** を選択します。
1. **[デベロッパー センターの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ: 設定]** を選択します。

   | 設定                                                                 | 値                                                        |
   | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
   | サブスクリプション                                                            | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ                                                          | **新しい**リソース グループの名前 **rg-devcenter-01**     |
   | Name                                                                    | **devcenter-01**                                             |
   | 場所                                                                | **(米国) 米国東部**                                             |
   | クイック スタート カタログを添付する: Azure デプロイ環境の定義 | Enabled                                                      |
   | クイック スタート カタログを添付する: 開発ボックスのカスタマイズ タスク              | 無効                                                     |

   > **注:** このラボの 2 つ目の演習では、カスタマイズ タスクを有効にしてカタログを構成します。

1. **[デベロッパー センターの作成]** ページの **[設定]** タブで、以下の設定を指定し、**[レビューと作成]** を選択します。

   | 設定                                    | Value   |
   | ------------------------------------------ | ------- |
   | プロジェクトごとにカタログを有効にする                | Enabled |
   | プロジェクトで Microsoft がホストするネットワークを許可する | Enabled |
   | Azure Monitor エージェントのインストールを有効にする    | Enabled |

   > **注:** 設計上、デベロッパー センターにアタッチされているカタログのリソースは、その中のすべてのプロジェクトで使用できます。 **[プロジェクトごとにカタログを有効にする]** 設定をオンにすると、任意に選択したプロジェクトに追加のカタログをアタッチすることもできます。

   > **注:** 開発ボックスは、環境内のリソースと通信する必要があるかどうかに応じて、独自の Azure サブスクリプション内の仮想ネットワークまたは Microsoft がホストする仮想ネットワークに接続できます。 このような必要がない場合は、**[プロジェクトで Microsoft がホストするネットワークを許可する]** 設定を有効にすると、開発ボックスを Microsoft がホストするネットワークに接続するオプションが導入され、管理と構成のオーバーヘッドが効果的に最小限に抑えられます。

   > **注:** **[Azure Monitor エージェントのインストールを有効にする]** 設定では、デベロッパー センターのすべての開発ボックスで Azure Monitor エージェントのインストールが自動的にトリガーされます。

1. **[レビューと作成]** タブで、検証が完了するのを待ち、**[作成]** を選択します。

   > **注:** プロジェクトがプロビジョニングされるまで待ってください。 プロジェクトの作成には約 1 分かかります。

1. **[デプロイが完了しました]** ページで、 **[リソースに移動]** を選択します。

### タスク 2: デベロッパー センターの設定を確認する

このタスクでは、前のタスクで作成したデベロッパー センターの基本的な構成設定を確認します。

1. Azure portal が表示されている Web ブラウザーの **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[環境の構成]** セクションを展開し、**[カタログ]** を選択します。
1. **[devcenter-01 \| カタログ]** ページで、デベロッパー センターが GitHub リポジトリ `https://github.com/microsoft/devcenter-catalog.git` をポイントする **quickstart-environment-definitions** カタログを含んで構成されていることを確認します。
1. **[ステータス]** 列に **[同期成功]** エントリがあることを確認します。 そうでない場合は、次の一連の手順を使用してカタログを再作成します。

   1. 自動生成されたカタログ エントリ **quickstart-environment-definitions** の横にあるチェック ボックスをオンにし、ツール バーの **[削除]** を選択します。
   1. **[devcenter-01 \| カタログ]** ページで、**[+ 追加]** を選択します。
   1. **[カタログの追加]** ペインで、**[名前]** テキスト ボックスに「**`quickstart-environment-definitions-fix`**」と入力し、**[カタログの場所]** セクションで **[GitHub]** を選択し、**[認証の種類]** で **[GitHub アプリ]** を選択し、**[このカタログを自動的に同期する]** チェックボックスをオンのままにして **[GitHub でサインイン]** を選択します。
   1. **[GitHub でサインイン]** ウィンドウで、GitHub 資格情報を入力し、**[サインイン]** を選択します。

      > **注:** この GitHub 資格情報を使用すると、<https://github.com/microsoft/devcenter-catalog> のフォークとして作成された GitHub リポジトリにアクセスできます。

   1. メッセージが表示されたら、**[Microsoft デベローッパー センターを承認する]** ウィンドウで **[Microsoft デベローッパー センターを承認する]** を選択します。
   1. **[カタログの追加]** ペインに戻って、**[リポジトリ]** ドロップダウン リストで **devcenter-catalog** を選択し、**[ブランチ]** ドロップダウン リストの **[既定のブランチ]** エントリを受け入れ、**[フォルダー パス]** に「**`Environment-Definitions`**」と入力し、**[追加]** を選択します。
   1. **[devcenter-01 \| カタログ]** ページに戻り、**[ステータス]** 列のエントリを監視して同期が正常に完了したことを確認します。

1. **[devcenter-01 \| カタログ]** ページで、**[quickstart-environment-definitions-fix]** エントリを選択します。
1. **[quickstart-environment-definitions-fix]** ページで、事前定義された環境定義の一覧を確認します。

   > **注:** 各エントリは、GitHub リポジトリ `https://github.com/microsoft/devcenter-catalog.git` の **Environment-Definitions** フォルダーのそれぞれのサブフォルダーで定義された Azure デプロイ環境の定義を表します。

   > **注:** デプロイ環境は、環境定義と呼ばれるテンプレートで定義された Azure リソースのコレクションです。 開発者はこの定義を使用して、ソリューションをホストするインフラストラクチャをデプロイできます。 Azure デプロイ環境の詳細については、Microsoft Learn の記事「[Azure Deployment Environments とは](https://learn.microsoft.com/azure/deployment-environments/overview-what-is-azure-deployment-environments)」をご覧ください。

### タスク 3: 開発ボックス定義を作成する

このタスクでは、開発ボックス定義を作成します。 その目的は、一貫性のあるカスタマイズされた開発環境 (開発ボックスという) を作成するためのブループリントとして機能するオペレーティング システム、ツール、設定、およびリソースを定義することです。

1. Azure portal が表示されている Web ブラウザーの **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[Dev Box 構成]** セクションを展開し、**[Dev Box 定義]** を選択します。
1. **[devcenter-01 \| Dev Box 定義]** ページで、**[+ 作成]** を選択します。
1. **[開発ボックス定義の作成]** ページで、以下の設定を指定し、**[作成]** を選択します。

   | 設定            | 値                                                                                |
   | ------------------ | ------------------------------------------------------------------------------------ | ----------------------- |
   | 名前               | **devbox-definition-01**                                                             |
   | Image              | \*\*Visual Studio 2022 Enterprise on Windows 11 Enterprise + Microsoft 365 Apps 24H2 | 休止状態サポート\*\* |
   | イメージ バージョン      | **最新**                                                                           |
   | Compute            | **8 vCPU、32 GB RAM**                                                                |
   | Storage            | **256 GB SSD**                                                                       |
   | 休止状態を有効にする | Enabled                                                                              |

   > **注:** 開発ボックス定義が作成されるまで待ってください。 これにかかる時間は 1 分未満です。

### タスク 4: デベロッパー センター プロジェクトを作成する

このタスクでは、デベロッパー センター プロジェクトを作成します。 通常、デベロッパー センター プロジェクトは、組織内の開発プロジェクトに対応しています。 たとえば、基幹業務アプリケーションの開発用のプロジェクトと、会社の Web サイトの開発用の別のプロジェクトを作成できます。 デベロッパー センター内のすべてのプロジェクトが、同じ開発ボックス定義、ネットワーク接続、カタログ、コンピューティング ギャラリーを共有します。 プロジェクト管理者とアクセス許可の要件が異なる複数の開発プロジェクトがある場合は、複数のデベロッパー センター プロジェクトを作成することを検討してください。

1. Azure portal が表示されている Web ブラウザーの **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[管理]** セクションを展開し、**[プロジェクト]** を選択します。
1. **[devcenter-01 \| プロジェクト]** ページで、**[+ 作成]** を選択します。
1. **[プロジェクトを作成]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ: 開発ボックスの管理]** を選択します。

   | 設定        | 値                                                        |
   | -------------- | ------------------------------------------------------------ |
   | サブスクリプション   | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **rg-devcenter-01**                                          |
   | デベロッパー センター     | **devcenter-01**                                             |
   | Name           | **devcenter-project-01**                                     |
   | 説明    | **devcenter-project-01**                                     |

1. **[プロジェクトを作成]** ページの **[開発ボックスの管理]** タブで、以下の設定を指定し、**[次へ: カタログ]** を選択します。

   | 設定                 | Value |
   | ----------------------- | ----- |
   | Dev Box の制限を有効にする   | はい   |
   | 開発者あたりの Dev Box 数 | **2** |

1. **[プロジェクトを作成]** ページの **[カタログ]** タブで、以下の設定を指定し、**[レビューと作成]** を選択します。

   | 設定                            | Value   |
   | ---------------------------------- | ------- |
   | デプロイ環境の定義 | Enabled |
   | イメージ定義                  | Enabled |

   > **注:** デベロッパー センターの作成時にプロジェクト レベルのカタログを有効にしたので、開発プロジェクトにアタッチされているカタログでは、これらの設定によって、同期時にプロジェクトにインポートされるカタログ項目が決まります。

1. **[デベロッパー センターの作成]** ページの **[レビューと作成]** タブで、**[作成]** を選択します。

   > **注:** プロジェクトが作成されるまで待ってください。 これにかかる時間は 1 分未満です。

1. **[デプロイが完了しました]** ページで、 **[リソースに移動]** を選択します。

### タスク 5: 開発ボックス プールを作成する

このタスクでは、前のタスクで作成したデベロッパー センター プロジェクトに開発ボックス プールを作成します。 開発ボックス プールは、開発ボックス ユーザーが開発ボックスを作成するのに使用します。 開発ボックス プールは、開発ボックス定義とネットワーク接続をリンクします。 通常、Microsoft ホステッド 接続または独自の Azure ネットワーク接続から選択できます。 このネットワーク接続によって、開発ボックスがホストされる場所と、そこから他のクラウドおよびオンプレミスのリソースへのアクセスが決定します。 開発ボックスのユーザーに最も近いネットワーク接続を持つ開発ボックス プールを作成することを検討してください。 さらに、開発ボックスの実行コストを削減するために、事前に定義された時刻に毎日シャットダウンするように開発ボックス プールを構成できます。

1. Azure portal が表示されている Web ブラウザーの **[devcenter-project-01]** ページの左側にある縦ナビゲーション メニューで、**[管理]** セクションを展開し、**[開発ボックス プール]** を選択します。
1. **[devcenter-project-01 \| 開発ボックス プール]** ページで、**[+ 作成]** を選択します。
1. **[開発ボックス プールの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[作成]** を選択します。

   | 設定                                                                                                           | 値                                    |
   | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
   | 名前                                                                                                              | **devcenter-project-01-devbox-pool-01**  |
   | Definition                                                                                                        | **devbox-definition-01**                 |
   | ネットワーク接続                                                                                                | **Microsoft ホステッド ネットワークにデプロイする** |
   | リージョン                                                                                                            | **(米国) 米国東部**                         |
   | シングル サインオンの有効化                                                                                             | Enabled                                  |
   | Dev box Creator Privileges (開発ボックス作成者の特権)                                                                                        | **ローカル管理者**                  |
   | 自動停止を有効にする                                                                                      | Enabled                                  |
   | 停止時刻                                                                                                         | **07:00 PM**                             |
   | タイム ゾーン                                                                                                         | 現在使用しているタイム ゾーン                   |
   | 切断時の休止状態を有効にする                                                                                    | Enabled                                  |
   | 猶予時間 (分)                                                                                           | **60**                                   |
   | この組織に Azure ハイブリッド特典 ライセンスがあり、このプール内のすべての開発ボックスに適用されることを確認しました | Enabled                                  |

   > **注:** 開発ボックス プールが作成されるまで待ってください。 これには 2 分ほどかかる場合があります。

### タスク 6: アクセス許可を構成する

このタスクでは、ラボ環境でプロビジョニングされている 3 つの Microsoft Entra セキュリティ プリンシパルに、Microsoft 開発ボックス関連の適切なアクセス許可を割り当てます。 このセキュリティ プリンシパルは、プラットフォーム エンジニアリング シナリオにおける一般的な役割に対応しています。

| User              | グループ                        | 役割                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | プラットフォーム エンジニア     |
| devlead01         | DevCenter_Dev_Leads          | 開発チーム リーダー |
| devuser01         | DevCenter_Dev_Users          | 開発者             |

Microsoft Dev Box では、プロジェクト レベルの機能へのアクセスを管理するのに、Azure ロールベースのアクセス制御 (Azure RBAC) に依存しています。 プラットフォーム エンジニアは、デベロッパー センター、カタログ、プロジェクトの作成と管理を完全に制御できる必要があります。 これには、他のユーザーにアクセス許可を委任する機能も必要かどうかに応じて、所有者ロールまたは共同作成者ロールが実質的に必要です。 開発チームのリーダーには、Microsoft Dev Box プロジェクトで管理タスクを実行できるようになる、デベロッパー センターのプロジェクト管理者ロールを割り当てる必要があります。 Dev Box ユーザーは、Dev Box ユーザー ロールに関連付けられている独自の開発ボックスを作成および管理できる必要があります。

> **注:** まず、プラットフォーム エンジニアのユーザー アカウントを入れるための Microsoft Entra グループにアクセス許可を割り当てます。

1. Azure portal が表示されている Web ブラウザーで、 **[devcenter-01]** ページに移動し、左側の縦ナビゲーション メニューで **[アクセス制御 (IAM)]** を選択します。
1. **[devcenter-01 \| アクセス制御 (IAM)]** ページで、**[+ 追加]** を選択し、ドロップダウン リストで **[ロールの割り当ての追加]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[ロール]** タブで、**[特権管理者ロール]** タブを選択し、ロールの一覧で **[所有者]** を選択して、最後に **[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブで、**[ユーザー、グループ、またはサービス プリンシパル]** オプションが選択されていることを確認し、**[+ メンバーを選択する]** をクリックします。
1. **[メンバーを選択する]** ペインで、「**`DevCenter_Platform_Engineers`**」を検索して選択し、**[選択]** をクリックします。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブに戻り、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[条件]** タブの **[ユーザーが実行できる操作]** セクションで、オプション **[ユーザーがすべてのロールを割り当てることを許可する (高い特権)]** を選択し、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[確認と割り当て]** タブで、**[確認と割り当て]** を選択します。

   > **注:** 次に、開発チーム リーダーのユーザー アカウントを入れるための Microsoft Entra グループにアクセス許可を割り当てます。

1. **[devcenter-01 \| アクセス制御 (IAM)]** ページに戻って、左側の縦ナビゲーション メニューで **[管理]** セクションを展開し、**[プロジェクト]** を選択し、プロジェクトの一覧で **devcenter-project-01** を選択します。
1. **[devcenter-project-01]** ページの左側の縦ナビゲーション メニューで、**[アクセス制御 (IAM)]** を選択します。
1. **[devcenter-project-01 \| アクセス制御 (IAM)]** ページで、**[+ 追加]** を選択し、ドロップダウン リストで **[ロールの割り当ての追加]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[ロール]** タブで、**[職務ロール]** タブが選択されていることを確認し、ロールの一覧で **[DevCenter Project Admin]** を選択し、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブで、**[ユーザー、グループ、またはサービス プリンシパル]** オプションが選択されていることを確認し、**[+ メンバーを選択する]** をクリックします。
1. **[メンバーを選択する]** ペインで、「**`DevCenter_Dev_Leads`**」を検索して選択し、**[選択]** をクリックします。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブに戻り、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[確認と割り当て]** タブで、**[確認と割り当て]** を選択します。

   > **注:** 最後に、開発者ユーザー アカウントを入れるための Microsoft Entra グループにアクセス許可を割り当てます。

1. **[devcenter-project-01 \| アクセス制御 (IAM)]** ページに戻って、**[+ 追加]** を選択し、ドロップダウン リストで **[ロールの割り当ての追加]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[ロール]** タブで、**[職務ロール]** タブが選択されていることを確認し、ロールの一覧で **[DevCenter Dev Box Users]** を選択し、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブで、**[ユーザー、グループ、またはサービス プリンシパル]** オプションが選択されていることを確認し、**[+ メンバーを選択する]** をクリックします。
1. **[メンバーを選択する]** ペインで、「**`DevCenter_Dev_Users`**」を検索して選択し、**[選択]** をクリックします。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブに戻り、**[次へ]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[確認と割り当て]** タブで、**[確認と割り当て]** を選択します。

### タスク 7: 開発ボックスを評価する

このタスクでは、Microsoft Entra 開発者ユーザー アカウントを使用して開発ボックス機能を評価します。

1. Web ブラウザーのシークレット/プライベート モードを開始し、Microsoft Dev Box 開発者ポータル (`https://aka.ms/devbox-portal`) にアクセスします。
1. サインインを求められたら、**devuser01** ユーザー アカウントの資格情報を指定します。
1. Microsoft Dev Box 開発者ポータルの **[ようこそ、devuser01 さん]** ページで、**[+ 新しい開発ボックス]** を選択します。
1. **[開発ボックスを追加する]** ペインの **[名前]** ボックスに「**`devuser01box01`**」と入力します。
1. プロジェクト名、開発ボックス プールの仕様、休止状態サポートの状態、スケジュールされたシャットダウンのタイミングなど、**[開発ボックスを追加する]** ペインに表示されるその他の情報を確認します。 さらに、カスタマイズを適用するオプションと、開発ボックスの作成に最大 65 分かかる可能性があることを示す通知に注意してください。

   > **注:** 開発ボックスの名前は、プロジェクト内で一意である必要があります。

1. **[開発ボックスを追加する]** ペインで、**[作成]** を選択します。

   > **注:** 開発ボックスが作成されるまで待つ必要はありません。 代わりに、このラボの次の演習に直接進んでください。 ただし、次の演習を完了したらこの演習に戻ってこのタスクの残りを実行することを検討してください。

1. 開発ボックスが完全にプロビジョニングされて実行されたら、**アプリ経由で接続**するオプションを選択して接続します。

   > **注:** 開発ボックスへの接続は、リモート デスクトップ Windows アプリ、リモート デスクトップ クライアント (mstsc.exe)、または直接 Web ブラウザー ウィンドウ内で確立できます。

1. **[このサイトが Microsoft リモート接続センターを開こうとしています]** というタイトルのプロンプトが表示されたら、**[開く]** を選択します。 すると、開発ボックスへのリモート デスクトップ セッションが自動的に開始されます。
1. 資格情報の入力を求められたら、**devuser01** アカウントのユーザー名とパスワードを指定して認証を行います。
1. 開発ボックスへのリモート デスクトップ セッション内で、その構成に Visual Studio 2022 および Microsoft 365 アプリのインストールが含まれていることを確認します。

   > **注:** 最初に **[お使いの Dev Box]** インターフェイスにある省略記号を選択し、次にカスケード メニューから **[シャットダウン]** を選択することで、Microsoft Dev Box 開発者ポータルから開発ユーザーとして直接開発ボックスをシャットダウンできます。 または、プラットフォーム エンジニアまたは開発チームのリーダーとして、対応するデベロッパー センター プロジェクトの **[開発ボックス プール]** セクションから開発ボックスのライフサイクルを制御できます。

## 演習 2:Microsoft Dev Box 環境をカスタマイズする

この演習では、Microsoft Dev Box 環境の機能をカスタマイズします。 このアプローチでは、カスタムの開発者セルフサービス ソリューションを実装するときに適用できる変更の範囲に重点を置いています。

演習は次のタスクで構成されます。

- タスク 1: Azure コンピューティング ギャラリーを作成してデベロッパー センターにアタッチする
- タスク 2: Azure Image Builder の認証と承認を構成する
- タスク 3: Azure Image Builder を使用してカスタム イメージを作成する
- タスク 4: Azure デベロッパー センター ネットワーク接続を作成する
- タスク 5: Azure デベロッパー センター プロジェクトにイメージ定義を追加する
- タスク 6: カスタマイズした開発ボックス プールを作成する
- タスク 7: カスタマイズした開発ボックスを評価する

### タスク 1: Azure コンピューティング ギャラリーを作成してデベロッパー センターにアタッチする

このタスクでは、Azure コンピューティング ギャラリーを作成し、このラボの前の演習で作成したデベロッパー センターにアタッチします。 ギャラリーは Azure サブスクリプションに備わっているリポジトリで、カスタム イメージに関連する構造と体系を構築するのに役立ちます。 コンピューティング ギャラリーをデベロッパー センターにアタッチしてイメージで事前設定すると、そのコンピューティング ギャラリーに格納されているイメージに基づいて開発ボックスの定義を作成できます。

1. Web ブラウザーを開始し、Azure portal (`https://portal.azure.com`) にアクセスします。
1. 認証を求められたら、Microsoft アカウントを使用してサインインします。
1. Azure portal の **[検索]** テキスト ボックスで、「**`Azure compute galleries`**」を検索して選択します。
1. **[Azure コンピューティング ギャラリー]** ページで、**[+ 作成]** を選択します。
1. **[Azure コンピューティング ギャラリーの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ: 共有方法]** を選択します。

   | 設定        | 値                                                        |
   | -------------- | ------------------------------------------------------------ |
   | サブスクリプション   | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **rg-devcenter-01**                                          |
   | Name           | **compute_gallery_01**                                       |
   | リージョン         | **(米国) 米国東部**                                             |

1. **[Azure コンピューティング ギャラリーの作成]** ページの **[共有方法]** タブで、**[ロールベースのアクセス制御 (RBAC)]** オプションが選択されていることを確認し、**[確認および作成]** を選択します。
1. **[レビューと作成]** タブで、検証が完了するのを待ち、**[作成]** を選択します。

   > **注:** プロジェクトがプロビジョニングされるまで待ってください。 Azure コンピューティング ギャラリーの作成にかかる時間は 1 分未満のはずです。

1. Azure portal で、「**`Dev centers`**」を検索して選択し、**[デベロッパー センター]** ページで **devcenter-01** を選択します。
1. **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[Dev Box 構成]** セクションを展開し、**[Azure Compute Gallery]** を選択します。
1. **[devcenter-01 \| Azure コンピューティング ギャラリー]** ページで、**[+ 追加]** を選択します。
1. **[Azure コンピューティング ギャラリーの追加]** ペインの **[ギャラリー]** ドロップダウン リストで、**compute_gallery_01**を選択して、**[追加]** を選択します。

   > **注:** _"このデベロッパー センターには、システム割り当て ID またはユーザー割り当て ID がありません。ID が割り当てられるまでギャラリーを追加できません。"_ というエラー メッセージが表示された場合は、 デベロッパー センターにシステム割り当て ID を割り当てる必要があります。
   > そうするには、Azure portal の **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、[設定] の下にある **[ID]** を選択し、**[システム割り当て]** タブで **[状態]** スイッチを **[オン]** に設定し、**[保存]** を選択します。

### タスク 2: 認証と承認を構成する

このタスクでは、前のタスクで作成した Azure コンピューティング ギャラリーにイメージを追加するために Azure Image Builder によって使用されるユーザー割り当てマネージド ID を作成します。 また、カスタムのロールベースのアクセス制御 (RBAC) ロールを作成し、それをマネージド ID に割り当てることで、必要なアクセス許可を構成します。 こうすることで、次のタスクで Azure Image Builder を使用してカスタム イメージをビルドできます。

1. Azure portal で、**Cloud Shell** ツール バー アイコンを選択して Cloud Shell ウィンドウを開き、必要に応じて **[PowerShell に切り替える]** を選択して PowerShell セッションを開始し、**[Cloud Shell の PowerShell に切り替える]** ダイアログ ボックスで **[確認]** を選択します。

   > **注:** Cloud Shell を初めて開く場合は、**[Azure Cloud Shell へようこそ]** ダイアログ ボックスで **[PowerShell]** を選択し、**[作業の開始]** ペインでオプション **[ストレージ アカウントは不要です]** を選択し、**[サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションの名前を選択します。

1. Cloud Shell ウィンドウの PowerShell セッションで、次のコマンドを実行して、必要なすべてのリソース プロバイダーが登録されていることを確認します。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
   Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
   Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
   Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
   Register-AzResourceProvider -ProviderNamespace Microsoft.Network
   ```

1. 次のコマンドを実行して、必要な PowerShell モジュールをインストールします (プロンプトが表示されたら、「**A**」を入力して **Enter** キーを押します)。

   ```powershell
   'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
   ```

1. 次のコマンドを実行して、イメージ ビルド プロセス全体で参照される変数を設定します。

   ```powershell
   $currentAzContext = Get-AzContext
   # the target Azure subscription ID
   $subscriptionID=$currentAzContext.Subscription.Id
   # the target Azure resource group name
   $imageResourceGroup='rg-devcenter-01'
   # the target Azure region
   $location='eastus'
   # the reference name assigned to the image created by using the Azure Image Builder service
   $runOutputName="aibWinImg01"
   # image template name
   $imageTemplateName="templateWinVSCode01"
   # the Azure compute gallery name
   $computeGallery = 'compute_gallery_01'
   ```

1. 次のコマンドを実行して、ユーザー割り当てマネージド ID を作成します (VM Image Builder では、指定されたユーザー ID を使用して、対象の Azure コンピューティング ギャラリーにイメージを格納します)。

   ```powershell
   # Install the Azure PowerShell module to support AzUserAssignedIdentity
   Install-Module -Name Az.ManagedServiceIdentity
   # Generate a pseudo-random integer to be used for resource names
   $timeInt=$(get-date -UFormat "%s")

   # Create an identity
   $identityName='identityAIB' + $timeInt
   New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName -Location $location
   $identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
   $identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
   ```

1. 次のコマンドを実行して、新しく作成されたユーザー割り当てマネージド ID に、**rg-devcenter-01** リソース グループにイメージを格納するのに必要なアクセス許可を付与します。

   ```powershell
   # Set variables
   $imageRoleDefName = 'Custom Azure Image Builder Image Def' + $timeInt
   $aibRoleImageCreationUrl = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
   $aibRoleImageCreationPath = 'aibRoleImageCreation.json'

   # Customize the role definition file
   Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<subscriptionID>', $subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

   # Create a role definition
   New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

   # Assign the role to the VM Image Builder user-assigned managed identity within the scope of the **rg-devcenter-01** resource group
   New-AzRoleAssignment -ObjectId $identityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
   ```

### タスク 3: Azure Image Builder を使用してカスタム イメージを作成する

このタスクでは、Azure Image Builder を使用して、Choco と Visual Studio Code が自動的にインストールされる Windows 11 Enterprise イメージを定義する既存の Azure Resource Manager (ARM) テンプレートに基づいてカスタム イメージを作成します。 Azure VM Image Builder を使用すると、VM イメージの定義とプロビジョニングのプロセスが大幅に簡略化されます。 これは、自動イメージング パイプラインを構成するために指定したイメージ構成に依存します。 その後に、開発者はこのようなイメージを使用して開発ボックスをプロビジョニングできるようになります。

1. Cloud Shell ウィンドウの PowerShell セッションで、次のコマンドを実行して、この演習の最初のタスクで作成した Azure コンピューティング ギャラリーに追加するイメージ定義を作成します。

   ```powershell
   # ensure that the image definition security type property is set to 'TrustedLaunch'
   $securityType = @{Name='SecurityType';Value='TrustedLaunch'}
   $features = @($securityType)
   # Image definition name
   $imageDefName = 'imageDefDevBoxVSCode01'

   # Create the image definition
   New-AzGalleryImageDefinition -GalleryName $computeGallery -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'Contoso' -Offer 'vscodedevbox' -Sku '1-0-0' -Feature $features -HyperVGeneration 'V2'
   ```

   > **注:** 開発ボックス イメージは、第 2 世代、Hyper-V v2、Windows 10 または 11 Enterprise バージョン 20H2 以降の使用を含む多数の要件を満たす必要があります。 完全なリストについては、Microsoft Learn の記事「[Microsoft Dev Box 用に Azure Compute Gallery を構成する](https://learn.microsoft.com/en-us/azure/dev-box/how-to-configure-azure-compute-gallery)」をご覧ください。

1. 次のコマンドを実行して、template.json という名前の空のファイルを作成します。このファイルには、Choco と Visual Studio Code が自動的にインストールされる Windows 11 Enterprise イメージを定義する ARM テンプレートが格納されます。

   ```powershell
   Set-Location -Path ~
   $templateFile = 'template.json'
   Set-Content -Path $templateFile -Value ''
   ```

1. Cloud Shell の PowerShell セッションで、nano テキスト エディターを使用して、新しく作成されたファイルに次のコンテンツを追加します。

   > **注:** nano テキスト エディターを開くには、コマンド `nano ./template.json` を実行します。 変更を保存して nano テキスト エディターを終了するには、**Ctrl + X** キー、**Y** キー、**Enter** キーの順に押します。

   ```json
   {
     "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "imageTemplateName": {
         "type": "string"
       },
       "api-version": {
         "type": "string"
       },
       "svclocation": {
         "type": "string"
       }
     },
     "variables": {},
     "resources": [
       {
         "name": "[parameters('imageTemplateName')]",
         "type": "Microsoft.VirtualMachineImages/imageTemplates",
         "apiVersion": "[parameters('api-version')]",
         "location": "[parameters('svclocation')]",
         "dependsOn": [],
         "tags": {
           "imagebuilderTemplate": "win11multi",
           "userIdentity": "enabled"
         },
         "identity": {
           "type": "UserAssigned",
           "userAssignedIdentities": {
             "<imgBuilderId>": {}
           }
         },
         "properties": {
           "buildTimeoutInMinutes": 100,
           "vmProfile": {
             "vmSize": "Standard_DS2_v2",
             "osDiskSizeGB": 127
           },
           "source": {
             "type": "PlatformImage",
             "publisher": "MicrosoftWindowsDesktop",
             "offer": "Windows-11",
             "sku": "win11-21h2-ent",
             "version": "latest"
           },
           "customize": [
             {
               "type": "PowerShell",
               "name": "Install Choco and Vscode",
               "inline": [
                 "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))",
                 "choco install -y vscode"
               ]
             }
           ],
           "distribute": [
             {
               "type": "SharedImage",
               "galleryImageId": "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>",
               "runOutputName": "<runOutputName>",
               "artifactTags": {
                 "source": "azureVmImageBuilder",
                 "baseosimg": "win11multi"
               },
               "replicationRegions": ["<region1>", "<region2>"]
             }
           ]
         }
       }
     ]
   }
   ```

1. 次のコマンドを実行して、template.json のプレースホルダーを実際の Azure 環境に固有の値に置き換えます。

   ```powershell
   $replRegion2 = 'eastus2'
   $templateFilePath = '.\template.json'
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<subscriptionID>', $subscriptionID | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<rgName>', $imageResourceGroup | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<runOutputName>', $runOutputName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<imageDefName>', $imageDefName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<sharedImageGalName>', $computeGallery | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region1>', $location | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region2>', $replRegion2 | Set-Content -Path $templateFilePath
   ((Get-Content -Path $templateFilePath -Raw) -Replace '<imgBuilderId>', $identityNameResourceId) | Set-Content -Path $templateFilePath
   ```

1. 次のコマンドを実行して、Azure Image Builder サービスにテンプレートを送信します (このサービスは、カスタム仮想マシン イメージをビルドするために、スクリプトなどの依存アーティファクトをダウンロードし、**IT\_** プレフィックスを含むステージング リソース グループに格納することで、送信されたテンプレートを処理します)。

   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -Api-Version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location
   ```

1. 次のコマンドを実行して、イメージ ビルド プロセスを起動します。

   ```powershell
   Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
   ```

1. 次のコマンドを実行して、イメージのプロビジョニング状態を判定します。

   ```powershell
   Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup | Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
   ```

   > **注:** 次の出力は、ビルド プロセスが正常に完了したことを示します。

   ```powershell
   Name                LastRunStatusRunState LastRunStatusMessage ProvisioningState
   ----                --------------------- -------------------- -----------------
   templateWinVSCode01 Succeeded                                  Succeeded
   ```

1. または、ビルドの進行状況を監視するには、次の手順を使用します。

   1. Azure portal で、**`Image templates`** を検索して選択します。
   1. **[イメージ テンプレート]** ページで、**templateWinVSCode01** を選択します。
   1. **[templateWinVSCode01]** ページの **[要点]** セクションで、**[ビルドの実行状態]** エントリの値を確認します。

   > **注:** ビルド プロセスは約 30 分かかる場合があります。 完了を待たずに、この演習の次のタスクに進んでください。

1. ビルドが完了したら、Azure portal で「**`Azure compute galleries`**」を検索して選択します。
1. **[Azure コンピューティング ギャラリー]** ページで、**compute_gallery_01** を選択します。
1. **compute_gallery_01** ページで、**[定義]** タブが選択されていることを確認し、定義の一覧で **imageDefDevBoxVSCode** を選択します。
1. **[imageDefDevBoxVSCode]** ページで、**[バージョン]** タブを選択し、**1.0.0 (最新バージョン)** エントリが、**[プロビジョニングの状態]** が **[Succeeded]** になって一覧に表示されていることを確認します。
1. **[1.0.0 (最新バージョン)]** エントリを選択します。
1. **[1.0.0 (compute_gallery_01/imageDefDevBoxVSCode/1.0.0)]** ページで、VM イメージのバージョン設定を確認します。

### タスク 4: Azure デベロッパー センター ネットワーク接続を作成する

このタスクでは、Azure 仮想ネットワーク内でホストされているリソースへのプライベート接続を必要とするシナリオで使用するための、Azure デベロッパー センター ネットワークを構成します。 このラボの最初の演習で利用した Microsoft がホストするネットワークとは異なり、仮想ネットワーク接続では、ハイブリッド シナリオ (オンプレミス リソースへの接続を提供) と Azure 開発ボックスの Microsoft Entra ハイブリッド参加 (Microsoft Entra 参加のサポートに加えて) もサポートされます。

1. Azure portal が表示されている Web ブラウザーの **[検索]** テキスト ボックスで、「**`Virtual networks`**」を検索して選択します。
1. **[仮想ネットワーク]** ページで、**[+ 作成]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ]** を選択します。

   | 設定        | 値                                                        |
   | -------------- | ------------------------------------------------------------ |
   | サブスクリプション   | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **rg-devcenter-01**                                          |
   | Name           | **vnet-01**                                                  |
   | 場所       | **(米国) 米国東部**                                             |

1. **[仮想ネットワークの作成]** ページの **[セキュリティ]** タブで、既定値を変更せずに既存の設定を確認し、**[次へ]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[IP アドレス]** タブで、既定値を変更せずに既存の設定を確認し、**[確認 + 作成]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注:** 仮想ネットワークが作成されるまで待ってください。 これにかかる時間は 1 分未満です。

1. Azure portal の **[検索]** テキスト ボックスで、「**`Network connections`**」を検索して選択します。
1. **[ネットワーク接続]** ページで、**[+ 作成]** を選択します。
1. **[ネットワーク接続の作成]** ページの **[基本]** タブで、以下の設定を指定し、**[レビューと作成]** を選択します。

   | 設定         | 値                                                        |
   | --------------- | ------------------------------------------------------------ |
   | サブスクリプション    | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ  | **rg-devcenter-01**                                          |
   | Name            | **network-connection-vnet-01**                               |
   | 仮想ネットワーク | **vnet-01**                                                  |
   | Subnet          | **default**                                                  |

1. **[仮想ネットワークの作成]** ページの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注:** ネットワーク接続が作成されるまで待ってください。 これには 1 分ほどかかる場合があります。

1. Azure portal で、「**`Dev centers`**」を検索して選択し、**[デベロッパー センター]** ページで **devcenter-01** を選択します。
1. **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[Dev Box 構成]** セクションを展開し、**[ネットワーク]** を選択します。
1. **[devcenter-01 \| ネットワーク]** ページで、**[+ 追加]** を選択します。
1. **[ネットワーク接続を追加]** ペインの **[ネットワーク接続]** ドロップダウン リストで、**network-connection-vnet-01** を選択し、**[追加]** を選択します。

   > **注:** ネットワーク接続が作成されるまで待たずに、次のタスクに進んでください。 ネットワーク接続の追加には約 1 分かかります。

### タスク 5: Azure デベロッパー センター プロジェクトにイメージ定義を追加する

このタスクでは、このラボの最初の演習で作成した Azure デベロッパー センター プロジェクトにイメージ定義を追加します。 イメージ定義では、Azure Marketplace またはカスタム イメージと、基になるイメージに適用する追加の変更を定義する構成可能なタスクが組み合わされます。 イメージ定義を使用して、新しいイメージ (タスクによって適用されたものを含むすべての変更を含む) をビルドしたり、開発ボックス プールを直接作成したりできます。 再利用可能なイメージを作成すると、開発ボックスのプロビジョニングに必要な時間が最小限に抑えられます。

Microsoft Dev Box チームのカスタマイズ用にイメージングを構成するには、プロジェクト レベルのカタログ (このラボの最初の演習で既に完了) を有効にする必要があります。 このタスクでは、プロジェクトのカタログ同期設定を構成します。 この作業には、イメージ定義ファイルを含むカタログのアタッチが含まれます。

1. Azure portal が表示されている Web ブラウザーの **[検索]** テキスト ボックスで、「**`Dev centers`**」を検索して選択します。
1. **[デベロッパー センター]** ページで、**devcenter-01** を選択します。
1. **[devcenter-01]** ページの左側にある縦ナビゲーション メニューで、**[管理]** セクションを展開し、**[プロジェクト]** を選択します。
1. **[devcenter-01 \| プロジェクト]** ページのプロジェクトの一覧で、**devcenter-project-01** を選択します。
1. **[devcenter-project-01]** ページの左側にある縦ナビゲーション メニューで、**[設定]** セクションを展開し、**[カタログ]** を選択します。
1. **[devcenter-project-01 \| カタログ]** ページで、**[+ 追加]** を選択します。
1. **[カタログの追加]** ペインで、**[名前]** テキスト ボックスに「**`image-definitions-01`**」と入力し、**[カタログ ソース]** セクションで **[GitHub]** を選択し、**[認証の種類]** で **[GitHub アプリ]** を選択し、**[このカタログを自動的に同期する]** チェックボックスをオンのままにして **[GitHub でサインイン]** を選択します。
1. メッセージが表示されたら、**[GitHub でサインイン]** ウィンドウで GitHub 資格情報を入力し、**[サインイン]** を選択します。

   > **注:** この手順を完了するには、https://github.com/MicrosoftLearning/contoso-co-eShop リポジトリを GitHub アカウントにフォークする必要があります。

1. メッセージが表示されたら、**[Microsoft デベローッパー センターを承認する]** ウィンドウで、**[Microsoft デベローッパー センターを承認する]** を選択します。
1. **[カタログの追加]** ペインに戻って、**[リポジトリ]** ドロップダウン リストで **contoso-co-eShop** を選択し、**[ブランチ]** ドロップダウン リストで **[既定のブランチ]** エントリを受け入れ、**[フォルダー パス]** に「**`.devcenter/catalog/image-definitions`**」と入力し、**[追加]** を選択します。
1. **[devcenter-project-01 \| カタログ]** ページに戻り、**[ステータス]** 列のエントリを監視して同期が正常に完了したことを確認します。
1. **[ステータス]** 列の **[同期に成功しました]** リンクを選択し、結果の通知ウィンドウを確認し、3 つの項目がカタログに追加されたことを確認し、右上隅の **[x]** 記号を選択してウィンドウを閉じます。
1. **[devcenter-project-01 \| カタログ]** ページに戻って、**image-definitions-01** を選択し、そこに **contosoBaseImageDefinition**、**backend-eng**、および **frontend-eng** という名前の 3 つのエントリがあることを確認します。

   > **注:** このラボの目的として、次の内容を含む **frontend-eng** イメージ定義の機能をテストします。

   ```yaml
   $schema: "1.0"
   name: "frontend-eng"
   # Using the "Windows 11 Enterprise 24H2" image as the base
   image: microsoftwindowsdesktop_windows-ent-cpc_win11-24H2-ent-cpc
   description: "This definition is for the eShop frontend engineering environment"

   tasks:
     - name: ~/winget
       description: Install Visual Studio Code
       parameters:
         package: Microsoft.VisualStudioCode
         runAsUser: true
   ```

1. Azure portal で、**[devcenter-project-01]** ページに戻って、左側の縦ナビゲーション メニューにある **[管理]** セクションを展開し、**[イメージ定義]** を選択して、このタスクで前に特定したものと同じ 3 つのイメージ定義がページに表示されることを確認します。

### タスク 6: カスタマイズした開発ボックス プールを作成する

このタスクでは、新しくプロビジョニングされたイメージ定義を使用して開発ボックス プールを作成します。 このプールでは、この演習で前に設定したネットワーク接続も利用します。

1. **[devcenter-project-01 \| イメージ定義]** ページが表示されている Azure portal で、左側の縦ナビゲーション メニューの **[管理]** セクションにある **[開発ボックス プール]** を選択します。
1. **[devcenter-project-01 \| 開発ボックス プール]** ページで、**[+ 作成]** を選択します。
1. **[開発ボックス プールの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[作成]** を選択します。

   | 設定                                                                                                           | 値                                                 |
   | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
   | 名前                                                                                                              | **devcenter-project-01-devbox-pool-02**               |
   | Definition                                                                                                        | **frontend-eng**                                      |
   | Compute                                                                                                           | **8 vCPU、32 GB RAM**                                 |
   | Storage                                                                                                           | **256 GB SSD**                                        |
   | ネットワーク接続                                                                                                | **組織内のネットワーク接続に展開する** |
   | ネットワーク接続名                                                                                           | **network-connection-vnet-01**                        |
   | シングル サインオンの有効化                                                                                             | Enabled                                               |
   | Dev box Creator Privileges (開発ボックス作成者の特権)                                                                                        | **ローカル管理者**                               |
   | 自動停止を有効にする                                                                                      | Enabled                                               |
   | 停止時刻                                                                                                         | **07:00 PM**                                          |
   | タイム ゾーン                                                                                                         | 現在使用しているタイム ゾーン                                |
   | 切断時の休止状態を有効にする                                                                                    | Enabled                                               |
   | 猶予時間 (分)                                                                                           | **60**                                                |
   | この組織に Azure ハイブリッド特典 ライセンスがあり、このプール内のすべての開発ボックスに適用されることを確認しました | Enabled                                               |

   > **注:** 開発ボックス プールが作成されるまで待ってください。 これには 2 分ほどかかる場合があります。

### タスク 7: カスタマイズした開発ボックスを評価する

このタスクでは、Microsoft Entra 開発者ユーザー アカウントを使用して、カスタマイズされた開発ボックスの機能を評価します。

> **注:** このタスクを完了するために必要な追加の時間を考慮して、その完了を省略することができます。

1. Web ブラウザーのシークレット/プライベート モードを開始し、Microsoft Dev Box 開発者ポータル (`https://aka.ms/devbox-portal`) にアクセスします。
1. サインインを求められたら、**devuser01** ユーザー アカウントの資格情報を指定します。
1. Microsoft Dev Box 開発者ポータルの **[ようこそ、devuser01 さん]** ページで、**[+ 新しい開発ボックス]** を選択します。
1. **[開発ボックスを追加する]** ペインの **[名前]** テキスト ボックスに「**`devuser01box02`**」と入力します。
1. **[開発ボックス プール]** ドロップダウン リストで、**devcenter-project-01-devbox-pool-02** を選択します。
1. プロジェクト名、開発ボックス プールの仕様、休止状態サポートの状態、スケジュールされたシャットダウンのタイミングなど、**[開発ボックスを追加する]** ペインに表示されるその他の情報を確認します。 さらに、カスタマイズを適用するオプションと、開発ボックスの作成に最大 65 分かかる可能性があることを示す通知に注意してください。
1. **[開発ボックスを追加する]** ペインで、**[作成]** を選択します。
1. 開発ボックスが完全にプロビジョニングされて実行されたら、**アプリ経由で接続**するオプションを選択して接続します。
1. **[このサイトが Microsoft リモート接続センターを開こうとしています]** というタイトルのプロンプトが表示されたら、**[開く]** を選択します。 すると、開発ボックスへのリモート デスクトップ セッションが自動的に開始されます。
1. 資格情報の入力を求められたら、**devuser01** アカウントのユーザー名とパスワードを指定して認証を行います。
1. 開発ボックスへのリモート デスクトップ セッション内で、その構成に Visual Studio Code のインストールが含まれていることを確認します。

   > **注:** **開発ボックス**のインターフェイスで省略記号を選択して、カスケード メニューから **[カスタマイズ]** を選択することで、カスタマイズの結果を検証することもできます。
