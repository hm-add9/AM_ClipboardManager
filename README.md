# クリップボード履歴マネージャー

WPF (.NET 8) 製のシンプルなクリップボード履歴管理アプリです。

## 機能
- クリップボードにコピーした内容を自動的に履歴として記録
- 履歴をクリックすると再度クリップボードにコピー
- `Ctrl` キーを素早く2回押すとどこからでもウィンドウを呼び出し（グローバルホットキー）
- 📌ボタンでお気に入り項目をピン留め（自動削除の対象外）。ピン留めすると履歴タブから消え、「ピン留め」タブに移動する
- 「ピン留め」タブでピン留め済みの項目だけを確認・解除・削除できる（「全解除」で一括解除）
- 「固定値」タブでよく使う定型文を自由に登録・編集・削除し、クリックでコピー
- タスクトレイに常駐し、バックグラウンドで動作
- 履歴は `%AppData%\ClipboardManager\history.json`、固定値は `%AppData%\ClipboardManager\fixed_items.json` に自動保存され、再起動後も保持

## 動作環境
- Windows 10 / 11
- Visual Studio 2022（.NET デスクトップ開発ワークロード）
- .NET 8 SDK

## 開き方・実行方法
1. `ClipboardManager.sln` を Visual Studio 2022 でダブルクリックして開く
   - .NET 8 SDK が入っていない場合は Visual Studio Installer から「.NET 8.0 ランタイム」を追加
2. スタートアッププロジェクトが `ClipboardManager` になっていることを確認
3. `F5` でデバッグ実行、または `Ctrl+F5` で実行のみ

起動すると画面には何も表示されず、タスクトレイにアイコンが常駐します。
`Ctrl` キーを素早く2回押すか、トレイアイコンをダブルクリックすると履歴ウィンドウが右下に表示されます。
（Ctrl+C や Ctrl+V など他のキーと組み合わせた場合はカウントされません）

## コマンドラインでビルドする場合
```
cd ClipboardManager
dotnet build
dotnet run
```
※ WPF アプリのため Windows 上でのみビルド・実行できます。

## 他のPCへの配布（自己完結型で発行）
通常のビルド（F5やdotnet build）は「フレームワーク依存」で、実行先PCに .NET 8 デスクトップランタイムが必要です。
他のPCに.NETをインストールせずそのまま渡したい場合は、自己完結型（self-contained）の単一exeとして発行します。

**Visual Studioから発行する場合**
1. ソリューションエクスプローラーで `ClipboardManager` プロジェクトを右クリック → 「発行」
2. 発行プロファイル「FolderProfile」を選択（初回は追加が必要な場合は「フォルダー」を選び、既存の
   `Properties/PublishProfiles/FolderProfile.pubxml` を指定）
3. 「発行」を実行
4. `ClipboardManager\bin\Release\net8.0-windows\publish\` に `ClipboardManager.exe` が生成される
   （このexe1つを他のPCにコピーするだけで動作します）

**コマンドラインから発行する場合**
```
cd ClipboardManager
dotnet publish -c Release
```
※ 対象PCが64bit Windows以外（ARM64など）の場合は `Properties/PublishProfiles/FolderProfile.pubxml`
   内の `RuntimeIdentifier` を `win-arm64` 等に変更してください。

自己完結型のexeはファイルサイズが数十MB程度になりますが、配布先のPCに何もインストールする必要はありません。

## カスタマイズ
- ホットキーの間隔調整: `Services/DoubleCtrlHotkeyWatcher.cs` の `DoubleTapThreshold`（既定値400ミリ秒）
- 履歴の保持件数: `MainWindow.xaml.cs` の `MaxHistoryCount`（既定値100件、ピン留めは対象外）
- 見た目の色: `App.xaml` の `SolidColorBrush` リソースを変更

## プロジェクト構成
```
ClipboardManager/
├── ClipboardManager.sln
└── ClipboardManager/
    ├── ClipboardManager.csproj
    ├── Properties/PublishProfiles/FolderProfile.pubxml … 自己完結型発行プロファイル
    ├── App.xaml / App.xaml.cs             … アプリ起動処理（非表示ウィンドウとして常駐）
    ├── MainWindow.xaml / .xaml.cs         … 履歴・固定値タブのUIとメインロジック
    ├── FixedItemEditWindow.xaml / .xaml.cs … 固定値の追加・編集ダイアログ
    ├── Models/
    │   ├── ClipboardItem.cs               … 履歴1件分のデータモデル
    │   └── FixedItem.cs                   … 固定値1件分のデータモデル
    └── Services/
        ├── NativeMethods.cs               … キーボードフック・クリップボード監視のWin32 API
        ├── DoubleCtrlHotkeyWatcher.cs      … Ctrl 2回押し検出（低レベルキーボードフック）
        ├── HistoryStore.cs                … 履歴のJSON保存・読込
        ├── FixedItemStore.cs              … 固定値のJSON保存・読込
        └── TrayIconManager.cs             … タスクトレイアイコン管理
```
