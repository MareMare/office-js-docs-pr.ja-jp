多くの場合、パフォーマンス上の理由から、アドインは Mac Officeにキャッシュされます。 通常、キャッシュはアドインを再読み込みすることでクリアされます。 同じドキュメント内に複数のアドインが存在する場合、再読み込み時にキャッシュを自動的にクリアするプロセスは信頼できない可能性があります。

### <a name="use-the-personality-menu-to-clear-the-cache"></a>パーソナリティ メニューを使用してキャッシュをクリアする

作業ウィンドウ アドインの [パーソナリティ] メニューを使用してキャッシュをクリアすることができます。 ただし、Outlook アドインではパーソナリティ メニューはサポートされていないので、Outlook を使用している場合は、手動でキャッシュをクリアする[](#clear-the-cache-manually)オプションを試Outlook。

- [パーソナリティ] メニューを選択します。 次に、**[Web キャッシュのクリア]** を選択します。
    > [!NOTE]
    > [パーソナリティ] メニューを表示するには、macOS のバージョン 10.13.6 以降を実行する必要があります。

    ![[パーソナリティ] メニューの [Web キャッシュのクリア] オプションのスクリーン ショット。](../images/mac-clear-cache-menu.png)

### <a name="clear-the-cache-manually"></a>キャッシュを手動でクリアする

`~/Library/Containers/com.Microsoft.OsfWebHost/Data/` フォルダーのコンテンツを削除することによってキャッシュを手動でクリアすることもできます。 ターミナル経由でこのフォルダーを探します。

> [!NOTE]
> そのフォルダーが存在しない場合は、ターミナル経由で次のフォルダーを確認し、見つかった場合は、フォルダーの内容を削除します。
>
> - `{host}` が Office アプリケーション (例: `Excel`) である `~/Library/Containers/com.microsoft.{host}/Data/Library/Caches/`
> - `{host}` が Office アプリケーション (例: `Excel`) である `~/Library/Containers/com.microsoft.{host}/Data/Library/Application Support/Microsoft/Office/16.0/Wef/`
> - `~/Library/Containers/com.microsoft.Office365ServiceV2/Data/Caches/com.microsoft.Office365ServiceV2/`
> - `~/Library/Containers/com.microsoft.Office365ServiceV2/Data/Library/Caches/com.microsoft.Office365ServiceV2/`
>
> Finder を使用してこれらのフォルダーを検索するには、非表示のファイルを表示する Finder を設定する必要があります。 Finder は、**コンテナー** ディレクトリ内のフォルダーを、**com.microsoft.Excel** ではなく、Microsoft Excel などの製品名で表示します。