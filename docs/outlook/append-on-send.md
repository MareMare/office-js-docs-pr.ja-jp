---
title: Outlook アドインに Append-on-send を実装する
description: Outlook アドインで追加オン送信機能を実装する方法について説明します。
ms.topic: article
ms.date: 10/13/2022
ms.localizationpriority: medium
ms.openlocfilehash: 18d3e8300a53d08cf484f14cd4fd05adf6382fe3
ms.sourcegitcommit: a2df9538b3deb32ae3060ecb09da15f5a3d6cb8d
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/12/2022
ms.locfileid: "68541144"
---
# <a name="implement-append-on-send-in-your-outlook-add-in"></a>Outlook アドインに Append-on-send を実装する

このチュートリアルの終わりまでに、メッセージの送信時に免責事項を挿入できる Outlook アドインが作成されます。

> [!NOTE]
> この機能のサポートは、要件セット 1.9 で導入されました。 この要件セットをサポートする [クライアントおよびプラットフォーム](/javascript/api/requirement-sets/outlook/outlook-api-requirement-sets#requirement-sets-supported-by-exchange-servers-and-outlook-clients) を参照してください。

## <a name="set-up-your-environment"></a>環境を設定する

Office アドイン用 Yeoman ジェネレーターを使用してアドイン プロジェクトを作成する [Outlook クイック スタート](../quickstarts/outlook-quickstart.md?tabs=yeomangenerator) を完了します。

> [!NOTE]
> [Office アドインの Teams マニフェスト (プレビュー)](../develop/json-manifest-overview.md) を使用する場合は、[Outlook クイック スタートで Teams マニフェスト (プレビュー)](../quickstarts/outlook-quickstart-json-manifest.md) を使用して代替クイック スタートを完了しますが、[**試してみる**] セクションの後のすべてのセクションをスキップします。

## <a name="configure-the-manifest"></a>マニフェストを構成する

マニフェストを構成するには、使用しているマニフェストの種類のタブを開きます。

# <a name="xml-manifest"></a>[XML マニフェスト](#tab/xmlmanifest)

アドインで追加の送信機能を有効にするには、[ExtendedPermissions](/javascript/api/manifest/extendedpermissions) のコレクションにアクセス許可を含める`AppendOnSend`必要があります。

このシナリオでは、[アクションの実行] ボタンを`action`選択して関数を実行する代わりに、関数を`appendOnSend`実行します。

1. コード エディターで、クイック スタート プロジェクトを開きます。

1. プロジェクトのルートにある **manifest.xml** ファイルを開きます。

1. ノード全体 **\<VersionOverrides\>** (開いているタグと閉じるタグを含む) を選択し、次の XML に置き換えます。

    ```XML
    <VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides" xsi:type="VersionOverridesV1_0">
      <VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides/1.1" xsi:type="VersionOverridesV1_1">
        <Requirements>
          <bt:Sets DefaultMinVersion="1.9">
            <bt:Set Name="Mailbox" />
          </bt:Sets>
        </Requirements>
        <Hosts>
          <Host xsi:type="MailHost">
            <DesktopFormFactor>
              <FunctionFile resid="Commands.Url" />
              <ExtensionPoint xsi:type="MessageComposeCommandSurface">
                <OfficeTab id="TabDefault">
                  <Group id="msgComposeGroup">
                    <Label resid="GroupLabel" />
                    <Control xsi:type="Button" id="msgComposeOpenPaneButton">
                      <Label resid="TaskpaneButton.Label" />
                      <Supertip>
                        <Title resid="TaskpaneButton.Label" />
                        <Description resid="TaskpaneButton.Tooltip" />
                      </Supertip>
                      <Icon>
                        <bt:Image size="16" resid="Icon.16x16" />
                        <bt:Image size="32" resid="Icon.32x32" />
                        <bt:Image size="80" resid="Icon.80x80" />
                      </Icon>
                      <Action xsi:type="ShowTaskpane">
                        <SourceLocation resid="Taskpane.Url" />
                      </Action>
                    </Control>
                    <Control xsi:type="Button" id="ActionButton">
                      <Label resid="ActionButton.Label"/>
                      <Supertip>
                        <Title resid="ActionButton.Label"/>
                        <Description resid="ActionButton.Tooltip"/>
                      </Supertip>
                      <Icon>
                        <bt:Image size="16" resid="Icon.16x16"/>
                        <bt:Image size="32" resid="Icon.32x32"/>
                        <bt:Image size="80" resid="Icon.80x80"/>
                      </Icon>
                      <Action xsi:type="ExecuteFunction">
                        <FunctionName>appendDisclaimerOnSend</FunctionName>
                      </Action>
                    </Control>
                  </Group>
                </OfficeTab>
              </ExtensionPoint>

              <!-- Configure AppointmentOrganizerCommandSurface extension point to support
              append on sending a new appointment. -->

            </DesktopFormFactor>
          </Host>
        </Hosts>
        <Resources>
          <bt:Images>
            <bt:Image id="Icon.16x16" DefaultValue="https://localhost:3000/assets/icon-16.png"/>
            <bt:Image id="Icon.32x32" DefaultValue="https://localhost:3000/assets/icon-32.png"/>
            <bt:Image id="Icon.80x80" DefaultValue="https://localhost:3000/assets/icon-80.png"/>
          </bt:Images>
          <bt:Urls>
            <bt:Url id="Commands.Url" DefaultValue="https://localhost:3000/commands.html" />
            <bt:Url id="Taskpane.Url" DefaultValue="https://localhost:3000/taskpane.html" />
            <bt:Url id="WebViewRuntime.Url" DefaultValue="https://localhost:3000/commands.html" />
            <bt:Url id="JSRuntime.Url" DefaultValue="https://localhost:3000/runtime.js" />
          </bt:Urls>
          <bt:ShortStrings>
            <bt:String id="GroupLabel" DefaultValue="Contoso Add-in"/>
            <bt:String id="TaskpaneButton.Label" DefaultValue="Show Taskpane"/>
            <bt:String id="ActionButton.Label" DefaultValue="Perform an action"/>
          </bt:ShortStrings>
          <bt:LongStrings>
            <bt:String id="TaskpaneButton.Tooltip" DefaultValue="Opens a pane displaying all available properties."/>
            <bt:String id="ActionButton.Tooltip" DefaultValue="Perform an action when clicked."/>
          </bt:LongStrings>
        </Resources>
        <ExtendedPermissions>
          <ExtendedPermission>AppendOnSend</ExtendedPermission>
        </ExtendedPermissions>
      </VersionOverrides>
    </VersionOverrides>
    ```

# <a name="teams-manifest-developer-preview"></a>[Teams マニフェスト (開発者プレビュー)](#tab/jsonmanifest)

1. manifest.json ファイルを開きます。

1. 次のオブジェクトを "extensions.runtimes" 配列に追加します。 このコードについては、次の点に注意してください。

   - メールボックス要件セットの "minVersion" は "1.9" に設定されているため、この機能がサポートされていないプラットフォームおよび Office バージョンにはアドインをインストールできません。 
   - ランタイムの "id" は、わかりやすい名前 "function_command_runtime" に設定されます。
   - "code.page" プロパティは、関数コマンドを読み込む UI レス HTML ファイルの URL に設定されます。
   - "有効期間" プロパティは "short" に設定されています。これは、関数コマンド ボタンが選択されたときにランタイムが起動し、関数が完了したときにシャットダウンすることを意味します。 (まれに、ハンドラーが完了する前にランタイムがシャットダウンされる場合があります。 [「Office アドインのランタイム](../testing/runtimes.md)」を参照してください)。
   - "appendDisclaimerOnSend" という名前の関数を実行するアクションがあります。 この関数は、後の手順で作成します。

    ```json
    {
        "requirements": {
            "capabilities": [
                {
                    "name": "Mailbox",
                    "minVersion": "1.9"
                }
            ],
            "formFactors": [
                "desktop"
            ]
        },
        "id": "function_command_runtime",
        "type": "general",
        "code": {
            "page": "https://localhost:3000/commands.html"
        },
        "lifetime": "short",
        "actions": [
            {
                "id": "appendDisclaimerOnSend",
                "type": "executeFunction",
                "displayName": "appendDisclaimerOnSend"
            }
        ]
    }
    ```

1. "authorization.permissions.resourceSpecific" 配列に、次のオブジェクトを追加します。 配列内の他のオブジェクトとコンマで区切っていることを確認します。

    ```json
    {
      "name": "Mailbox.AppendOnSend.User",
      "type": "Delegated"
    }
    ```

---

> [!TIP]
> Outlook アドインのマニフェストの詳細については、「 [Outlook アドイン マニフェスト](manifests.md)」を参照してください。

## <a name="implement-append-on-send-handling"></a>append-on-send 処理を実装する

次に、送信イベントに追加を実装します。

> [!IMPORTANT]
> アドインで [On-send イベント処理も実装 `ItemSend`](outlook-on-send-addins.md)されている場合は、送信ハンドラーで呼び出すと `AppendOnSendAsync` 、このシナリオはサポートされていないため、エラーが返されます。

このシナリオでは、ユーザーが送信したときにアイテムに免責事項を追加することを実装します。

1. 同じクイック スタート プロジェクトから、コード エディターで **ファイル ./src/commands/commands.js** を開きます。

1. 関数の後に `action` 、次の JavaScript 関数を挿入します。

    ```js
    function appendDisclaimerOnSend(event) {
      const appendText =
        '<p style = "color:blue"> <i>This and subsequent emails on the same topic are for discussion and information purposes only. Only those matters set out in a fully executed agreement are legally binding. This email may contain confidential information and should not be shared with any third party without the prior written agreement of Contoso. If you are not the intended recipient, take no action and contact the sender immediately.<br><br>Contoso Limited (company number 01624297) is a company registered in England and Wales whose registered office is at Contoso Campus, Thames Valley Park, Reading RG6 1WG</i></p>';  
      /**
        *************************************************************
         Ideal Usage - Call the getBodyType API. Use the coercionType
         it returns as the parameter value below.
        *************************************************************
      */
      Office.context.mailbox.item.body.appendOnSendAsync(
        appendText,
        {
          coercionType: Office.CoercionType.Html
        },
        function(asyncResult) {
          console.log(asyncResult);
        }
      );

      event.completed();
    }
    ```

1. 関数のすぐ下に次の行を追加して関数を登録します。

    ```js
    Office.actions.associate("appendDisclaimerOnSend", appendDisclaimerOnSend);
    ```

## <a name="try-it-out"></a>試してみる

1. プロジェクトのルート ディレクトリから次のコマンドを実行します。 このコマンドを実行すると、ローカル Web サーバーがまだ実行されておらず、アドインがサイドロードされると、ローカル Web サーバーが起動します。

    ```command&nbsp;line
    npm start
    ```

1. 新しいメッセージを作成し、 **To 行に** 自分を追加します。

1. リボンまたはオーバーフロー メニューから、[ **アクションの実行**] を選択します。

1. メッセージを送信し、 **受信トレイ** または **送信済みアイテム** フォルダーからメッセージを開き、追加された免責事項を表示します。

    ![Outlook on the webでの送信時に免責事項が追加されたサンプル メッセージ。](../images/outlook-web-append-disclaimer.png)

## <a name="see-also"></a>関連項目

[Outlook アドインのマニフェスト](manifests.md)
