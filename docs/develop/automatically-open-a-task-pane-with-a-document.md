---
title: ドキュメントで作業ウィンドウを自動的に開く
description: ドキュメントが開いたときに自動的に開く Office アドインを構成する方法について説明します。
ms.date: 09/01/2022
ms.localizationpriority: medium
ms.openlocfilehash: ea5981fc8469d391ff03c1d3eefd70c57e41d4cb
ms.sourcegitcommit: a32f5613d2bb44a8c812d7d407f106422a530f7a
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/14/2022
ms.locfileid: "67674631"
---
# <a name="automatically-open-a-task-pane-with-a-document"></a>ドキュメントで作業ウィンドウを自動的に開く

Office アドインでアドイン コマンドを使用すると、Office アプリのリボンにボタンを追加して Office UI を拡張できます。 ユーザーがコマンド ボタンをクリックすると、アクション (作業ウィンドウを開くなど) が実行されます。

いくつかのシナリオでは、ドキュメントを開いたときに、ユーザーの明示的な操作なしで、自動的に作業ウィンドウを開くことが必要になります。 [AddInCommands 1.1 要件セット](/javascript/api/requirement-sets/common/add-in-commands-requirement-sets)で導入された自動開き作業ウィンドウ機能を使用すると、シナリオで必要になったときに作業ウィンドウを自動的に開くことができます。

> [!NOTE]
> アドインのインストール時にすぐに開く作業ウィンドウを構成するには、「アドインがインストール [されたときに作業ウィンドウを自動的に開](automatically-open-on-installation.md)く」を参照してください。

## <a name="how-is-the-autoopen-feature-different-from-inserting-a-task-pane"></a>Autoopen 機能と作業ウィンドウの挿入の相違点

ユーザーがアドイン コマンドを使用しないアドイン (Office 2013 で実行するアドインなど) を起動すると、それらはドキュメントに挿入され、そのドキュメントに永続化されます。 その結果として、別のユーザーがドキュメントを開くと、そのユーザーにアドインのインストールを求めるダイアログが表示され、作業ウィンドウが開きます。 このモデルの課題は、多くの場合、ユーザーがアドインをドキュメントに保持したくないということです。 たとえば、Word ドキュメントで辞書アドインを使用する学生は、そのドキュメントを同級生や教師が開いたときに、アドインのインストールを求めるダイアログが表示されることを望まない場合もあります。

Autoopen 機能では、特定のドキュメントに特定の作業ウィンドウ アドインを永続化させるかどうかをユーザーが明示的に定義できます。

## <a name="support-and-availability"></a>サポートと可用性

autoopen 機能は現在、次の製品とプラットフォームでサポートされています。

|製品|プラットフォーム|
|:-----------|:------------|
|<ul><li>Word</li><li>Excel</li><li>PowerPoint</li></ul>|すべての製品でサポートされているプラットフォーム: <ul><li>Windows デスクトップ版 Office。ビルド 16.0.8121.1000+</li><li>Office on Mac。ビルド 15.34.17051500+</li><li>Office on the web</li></ul>|

## <a name="best-practices"></a>ベスト プラクティス

自動開き機能を使用する場合は、次のベスト プラクティスを適用します。

- Autoopen 機能は、アドイン ユーザーの作業効率の向上に役立つ場合に使用します。たとえば、次の場合に使用します。
  - 適切に機能するには、ドキュメントにアドインが必要になる場合。たとえば、アドインで最新の情報に定期的に更新される在庫の値が含まれているスプレッドシート。最新の値が維持されるように、アドインはスプレッドシートが開かれたときに自動的に開かれる必要があります。
  - 特定のドキュメントでユーザーが常にアドインを使用する可能性が高い場合。たとえば、バックエンド システムから情報を取得して、ドキュメントのデータを設定または変更することでユーザーを支援するアドイン。
- Autoopen 機能はユーザーがオン/オフできるようにします。アドインの作業ウィンドウが自動的に起動されないようにするオプションをユーザーの UI に含めます。  
- 要件セットの検出を使用して、自動開き機能を使用できるかどうかを判断し、そうでない場合はフォールバック動作を提供します。
- アドインの使用率を人為的に増やすために、Autoopen 機能を使用しないでください。 アドインが特定のドキュメントで自動的に開くのが意味がない場合、この機能はユーザーを困らせる可能性があります。

    > [!NOTE]
    > Microsoft では、Autoopen 機能の乱用を見つけた場合は、そのアドインを AppSource から排除することがあります。

- この機能は、複数の作業ウィンドウを固定するために使用しないでください。1 つのドキュメントで自動的に開くアドインのウィンドウは 1 つのみ設定できます。  

## <a name="implement-the-autoopen-feature"></a>autoopen 機能を実装する

- 自動的に開く作業ウィンドウを指定します。
- 作業ウィンドウを自動的に開くドキュメントにタグ設定します。

> [!IMPORTANT]
> 自動的に開くように指定したウィンドウは、アドインがユーザーのデバイスに既にインストールされている場合にのみ開きます。ユーザーがドキュメントを開いたときに、アドインがインストールされていない場合、Autoopen 機能は動作せずに、設定は無視されます。また、アドインをドキュメントと共に配布する必要がある場合は、可視性プロパティを 1 に設定する必要があります。これは、OpenXML を使用する場合にのみ実行できます。例については、この記事で後述します。

### <a name="step-1-specify-the-task-pane-to-open"></a>手順 1: 開く作業ウィンドウを指定する

自動的に開く作業ウィンドウを指定するには、[TaskpaneId](/javascript/api/manifest/action#taskpaneid) の値を **Office.AutoShowTaskpaneWithDocument** に設定します。この値は 1 つの作業ウィンドウにのみ設定できます。この値を複数の作業ウィンドウに設定すると、最初に見つかった値が認識され、その他は無視されます。

次の例では、Office.AutoShowTaskpaneWithDocument に設定された TaskPaneId の値を示しています。

```xml
<Action xsi:type="ShowTaskpane">
    <TaskpaneId>Office.AutoShowTaskpaneWithDocument</TaskpaneId>
    <SourceLocation resid="Contoso.Taskpane.Url" />
</Action>
```

### <a name="step-2-tag-the-document-to-automatically-open-the-task-pane"></a>手順 2:作業ウィンドウを自動的に開くよう、ドキュメントにタグを設定する

Autoopen 機能をトリガーするよう、2 つのうちどちらかの方法でドキュメントにタグを設定できます。 シナリオに最も適した方法を選択します。  

#### <a name="tag-the-document-on-the-client-side"></a>クライアント側でドキュメントにタグを設定する

次の例に示すように、**office.AutoShowTaskpaneWithDocument** を設定するには`true`、Office.js [settings.set](/javascript/api/office/office.settings) メソッドを使用します。

```js
Office.context.document.settings.set("Office.AutoShowTaskpaneWithDocument", true);
Office.context.document.settings.saveAsync();
```

このメソッドは、アドインの対話式操作の一環としてドキュメントにタグを設定する必要がある場合に使用します (たとえば、ユーザーがバインディングを作成した直後に、または自動的にウィンドウを開くことを示すオプションを選択した直後に使用します)。

#### <a name="use-open-xml-to-tag-the-document"></a>Open XML を使用してドキュメントにタグを設定する

Open XML を使用すると、Autoopen 機能をトリガーするために、ドキュメントを作成または変更して、適切な Open Office XML マークアップを追加できます。この方法を示すサンプルについては、「[Office-OOXML-EmbedAddin](https://github.com/OfficeDev/Office-OOXML-EmbedAddin)」を参照してください。

ドキュメントに 2 つの Open XML パーツを追加します。

- `webextension` パート
- `taskpane` パート

次の例は、`webextension` パートを追加する方法を示しています。

```xml
<we:webextension xmlns:we="http://schemas.microsoft.com/office/webextensions/webextension/2010/11" id="[ADD-IN ID PER MANIFEST]">
  <we:reference id="[GUID or AppSource asset ID]" version="[your add-in version]" store="[Pointer to store or catalog]" storeType="[Store or catalog type]"/>
  <we:alternateReferences/>
  <we:properties>
   <we:property name="Office.AutoShowTaskpaneWithDocument" value="true"/>
  </we:properties>
  <we:bindings/>
  <we:snapshot xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"/>
</we:webextension>
```

`webextension` パートには、プロパティ バッグと **Office.AutoShowTaskpaneWithDocument** という名前のプロパティが含まれています。このプロパティは、`true` に設定する必要があります。

また、`webextension` パートには、属性が `id`、`storeType`、`store`、および `version` のストアまたはカタログへの参照も含まれています。 Autoopen 機能に関連する `storeType` の値は、4 つのみです。 その他の 3 つの属性の値は、次の表に示すように、`storeType` の値に応じて決まります。

|`storeType` 値|`id` 値|`store` 値|`version` 値|
|:---------------|:---------------|:---------------|:---------------|
|OMEX (AppSource)|アドインの AppSource 資産 ID (注を参照)。|AppSource のロケール (たとえば、"en-us")。|AppSource カタログのバージョン (注を参照)。|
|WOPICatalog (パートナー [WOPI](/microsoft-365/cloud-storage-partner-program/online/) ホスト)| アドインの AppSource 資産 ID (注を参照)。 | "wopicatalog" アプリ ソースで公開され、WOPI ホストにインストールされているアドインには、この値を使用します。 詳細については、「 [Office Online との統合](/microsoft-365/cloud-storage-partner-program/online/overview)」を参照してください。 | アドイン マニフェストでのバージョン。|
|FileSystem (ネットワーク共有)|アドイン マニフェストでのアドインの GUID。|ネットワーク共有のパス。例: "\\\\MyComputer\\MySharedFolder"。|アドイン マニフェストでのバージョン。|
|EXCatalog (Exchange サーバー経由の展開) |アドイン マニフェストでのアドインの GUID。|"EXCatalog"。 EXCatalog 行は、Microsoft 365 管理センターで一元展開を使用するアドインで使用する行です。|アドイン マニフェストでのバージョン。|
|Registry (システム レジストリ)|アドイン マニフェストでのアドインの GUID。|"developer"|アドイン マニフェストでのバージョン。|

> [!NOTE]
> AppSource でのアドインのアセット ID とバージョンを確認するには、そのアドインの AppSource ランディング ページに移動します。アセット ID は、ブラウザーのアドレス バーに表示されます。バージョンは、そのページの **[詳細]** セクションに示されます。

webextension マークアップの詳細については、「[[MS-OWEXML] 2.2.5.WebExtensionReference](/openspecs/office_standards/ms-owexml/d4081e0b-5711-45de-b708-1dfa1b943ad1)」を参照してください。

次の例は、`taskpane` パートを追加する方法を示しています。

```xml
<wetp:taskpane dockstate="right" visibility="0" width="350" row="4" xmlns:wetp="http://schemas.microsoft.com/office/webextensions/taskpanes/2010/11">
  <wetp:webextensionref xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" r:id="rId1" />
</wetp:taskpane>
```

この例では、`visibility` 属性が "0" に設定されている点に注目してください。 これは、webextension パートと `taskpane` パートの追加後に、初めてドキュメントを開いたときに、ユーザーはリボンの **[アドイン]** ボタンからアドインをインストールする必要があることを意味します。 それ以降は、ファイルを開いたときに、アドイン作業ウィンドが自動的に開きます。 また、`visibility` を "0" に設定すると、ユーザーが Autoopen 機能をオン/オフできるようにするために Office.js を使用できるようにもなります。 具体的には、スクリプトでドキュメント設定の **Office.AutoShowTaskpaneWithDocument** を `true` または `false` に設定します  (詳細については、「[クライアント側でドキュメントにタグを設定する](#tag-the-document-on-the-client-side)」を参照してください)。

`visibility` が "1" に設定されていると、初めてドキュメントを開いたときに、自動的に作業ウィンドウが開きます。アドインを信頼するように求めるダイアログがユーザーに表示され、信頼が付与されるとアドインが開きます。それ以降は、ファイルを開いたときに、アドイン作業ウィンドが自動的に開きます。ただし、`visibility` を "1" に設定すると、ユーザーが Autoopen 機能をオン/オフできるようにするために Office.js を使用することができなくなります。

アドインとドキュメントのテンプレートまたはコンテンツが緊密に統合されていて、ユーザーが Autoopen 機能をオフにすることない場合は、`visibility` を "1" に設定することが適切な選択になります。

> [!NOTE]
> ドキュメントとともにアドインを配布する場合は、ユーザーにアドインをインストールするように求めるために、visibility プロパティを 1 に設定する必要があります。これは、Open XML でのみ実行できます。

XML を簡単に記述する方法は、まずアドインを実行し、 [クライアント側でドキュメントにタグを付](#tag-the-document-on-the-client-side) けて値を書き込み、次にドキュメントを保存して生成された XML を調べることです。Office は、適切な属性値を検出して提供します。 [Open XML SDK Productivity Tool](https://www.nuget.org/packages/Open-XML-SDK) を使用して C# コードを生成し、生成した XML に基づいてマークアップをプログラムで追加することもできます。

## <a name="test-and-verify-opening-task-panes"></a>作業ウィンドウ表示のテストと検証

Microsoft 365 管理センターを使用して一元展開を使用して作業ウィンドウを自動的に開くアドインのテスト バージョンを展開できます。 次の例では、EXCatalog のストア版を使用して一元展開カタログからアドインを挿入する方法を示します。

```xml
<we:webextension xmlns:we="http://schemas.microsoft.com/office/webextensions/webextension/2010/11" id="{52811C31-4593-43B8-A697-EB873422D156}">
    <we:reference id="af8fa5ba-4010-4bcc-9e03-a91ddadf6dd3" version="1.0.0.0" store="EXCatalog" storeType="EXCatalog"/>
    <we:alternateReferences/>
    <we:properties/>
    <we:bindings/>
    <we:snapshot xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"/>
</we:webextension>
```

前の例をテストするには、Microsoft 365 サブスクリプションを使用して一元展開を試し、アドインが期待どおりに機能することを確認します。 Microsoft 365 サブスクリプションをまだお持ちでない場合は、Microsoft 365 開発者プログラムに参加することで、90 日間の無料の更新可能な [Microsoft 365](https://developer.microsoft.com/office/dev-program) サブスクリプションを入手できます。

## <a name="see-also"></a>関連項目

- Autoopen 機能の使用方法を示すサンプルについては、「[Office-Add-in-Commands-Samples](https://github.com/OfficeDev/Office-Add-in-Commands-Samples/tree/master/AutoOpenTaskpane)」を参照してください。
- [アドインがインストールされたときに作業ウィンドウを自動的に開く](automatically-open-on-installation.md)
- [Microsoft 365 開発者プログラムに参加します。](/office/developer-program/office-365-developer-program)