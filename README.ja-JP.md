# OpenNiri-Windows

[![CI](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/actions/workflows/ci.yml/badge.svg)](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/actions/workflows/ci.yml)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

OpenNiri-Windows は、Rust で構築された Windows 10/11 向けのスクロール型タイルウィンドウマネージャーです。

DWM を置き換えることなく、Niri スタイルの水平ワークスペースモデルをネイティブ Windows に導入します。

## プロダクトの位置づけ

Windows 向けタイル型ウィンドウマネージャーの多くは、ツリー/BSP をベースにしています。OpenNiri-Windows はスクロールを中心に設計されています。

- ウィンドウは横方向のストリップ上に配置されます。
- モニターはそのストリップを表示するビューポートとして機能します。
- ウィンドウを追加しても、空間的に一貫した操作ができます。
- 分割ツリーを頻繁に作り直すのではなく、ワークスペースのコンテキスト内を移動できます。

## 対象ユーザー

- 毎日、多数のウィンドウをキーボード中心で管理するユーザー。
- シングルまたはマルチモニター環境を使うエンジニア、アナリスト、オペレーター、クリエイター。
- 透明性の高いアーキテクチャと Rust のコードベースを備えた、オープンソースの Windows 向けタイル型ウィンドウマネージャーを求めるチーム。

## 主な機能

現在実装済みの機能：

- マルチモニター対応ワークスペースと、モニターを考慮したフォーカス・移動コマンド
- グローバルホットキーと設定のライブリロード
- フローティング表示と全画面表示の切り替え
- 幅のプリセット（`Alt+1/2/3`）と均等化（`Alt+0`）
- スムーズなスクロールアニメーション、スナップヒント、タッチパッドジェスチャー（実験的機能、デフォルトでは無効）
- オプションのマウス追従フォーカス
- システムトレイ操作（一時停止/リロード/設定を開く/ログを開く/終了）
- ワークスペースの永続化と、より安全なシャットダウン・復旧動作
- トラブルシューティング用のセーフモード（`--safe-mode`）
- 組み込み診断機能（`openniri-cli doctor`）

## プロダクトのステータス

OpenNiri-Windows は **alpha** 段階で、現在も活発に開発されています。

現状は次のとおりです。

- コア動作は実装済みで、CI でテストされています。
- 現在の UX はキーボードと設定ファイルを中心としており、完全な GUI セットアップフローはまだありません。
- Windows 管理下の一部のウィンドウやシステムウィンドウでは、移動やスタイル変更が拒否される場合があります。
- 現在のリリース状況（2026-02-08 時点）：タグ付きの公開リリースはまだないため、主なインストール方法はソースからのビルドです。

## クイックスタート

### オプション A：ダウンロード（公開後に利用可能）

1. [GitHub Releases](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/releases) を開きます
2. タグ付きリリースがある場合は、最新の `.zip` アーカイブをダウンロードします
3. `openniri.exe` と `openniri-cli.exe` をフォルダーに展開します
4. （任意）そのフォルダーを `PATH` に追加します
5. デフォルト設定を生成します：
   ```
   openniri-cli init
   ```
6. デーモンを起動します：
   ```
   openniri-cli run
   ```

### オプション B：ソースからビルド（現在の Alpha では推奨）

前提条件：[Rust](https://rustup.rs) GNU ツールチェーン（`stable-x86_64-pc-windows-gnu`）と MinGW リンカー

```bash
rustup toolchain install stable-x86_64-pc-windows-gnu
git clone https://github.com/AdEx-Partners-DE/OpenNiri-Windows.git
cd OpenNiri-Windows
cargo +stable-x86_64-pc-windows-gnu build --release
cargo +stable-x86_64-pc-windows-gnu run -p openniri-cli -- init
cargo +stable-x86_64-pc-windows-gnu run -p openniri-cli -- run
```

ワークスペースは `.cargo/config.toml` で GNU（`x86_64-pc-windows-gnu`）を使用するよう設定されており、バイナリは `target/release/` に配置されます。

## インストール後の最初の手順

| 手順 | コマンド | 内容 |
|------|---------|--------------|
| 設定を作成 | `openniri-cli init` | コメント付きのデフォルト `config.toml` を作成 |
| デーモンを起動 | `openniri-cli run` | ウィンドウマネージャーをバックグラウンドで起動 |
| 状態を確認 | `openniri-cli status` | バージョン、モニター、ウィンドウ、稼働時間を表示 |
| 一時停止を切り替え | `openniri-cli toggle-pause`（エイリアス：`pause`） | デーモンを停止せずにタイル表示を一時停止/再開 |
| 診断を実行 | `openniri-cli doctor` | バイナリ、設定、デーモン、システムの状態を確認 |
| デーモンを停止 | `openniri-cli stop` | 終了を要求して停止確認を待機し、確認できない場合はローカルで緊急復旧を実行 |
| ローカル緊急復旧 | `openniri-cli emergency-uncloak`（エイリアス：`restore-windows`） | デーモン IPC を使わず、可能な範囲でローカルのウィンドウ表示を復旧 |
| 設定をリロード | `openniri-cli reload` | 再起動せずに設定変更を適用 |

デスクトップが応答しなくなったり、フォーカスが戻らなくなったりした場合は、まず非破壊的な復旧を行ってください。

```
openniri-cli panic-revert
openniri-cli status   # 再実行する前に、このコマンドが失敗すること（タイムアウト、または "Daemon is not running..."）を確認
```

`panic-revert` でデーモンの応答を確認できない場合、自動的にローカルのウィンドウ表示復旧処理も実行するようになりました。  
`openniri-cli stop` も、終了確認が不明確な場合に同じローカル復旧のセーフティネットを使用するようになりました。
`panic-revert` の短縮エイリアスとして `openniri-cli recover` も使用できます。
ターミナルを使用できない場合は、システムトレイで **Emergency: Uncloak All Windows** を選択してから **Exit** を選択してください。

セーフモードを起動するのは、デーモンの停止を確認した後にしてください。

```
openniri-cli run --safe-mode
```

手順の詳細は [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md) を参照してください。  
インシデントからの復旧については [docs/SUPPORT_PLAYBOOK.md](docs/SUPPORT_PLAYBOOK.md) を参照してください。

## デフォルトのホットキー

Windows 11 では `Win+文字` の組み合わせの大半が予約されており（`Win+L` は登録できません）、GlazeWM と同様にデフォルトでは `Alt` を使用します。既存の設定ファイルに含まれるバインドはそのまま維持されます。登録できないホットキーは名前とともにログに記録され、その他のホットキーは引き続き有効です。

| キー | 操作 |
|---|---|
| `Alt+H / Alt+L` | 左 / 右にフォーカス |
| `Alt+J / Alt+K` | 下 / 上にフォーカス |
| `Alt+Shift+H / Alt+Shift+L` | 列を左 / 右に移動 |
| `Alt+Ctrl+H / Alt+Ctrl+L` | 列の幅を縮小 / 拡大 |
| `Win+Ctrl+Escape` | ウィンドウ表示の緊急復旧 + デーモンの panic-revert |
| `Win+Alt+H / Win+Alt+L` | 左 / 右のモニターにフォーカス |
| `Win+Alt+Shift+H / Win+Alt+Shift+L` | ウィンドウを左 / 右のモニターに移動 |
| `Alt+Shift+Q` | フォーカス中のウィンドウを閉じる |
| `Alt+F` | フローティング表示を切り替え |
| `Alt+Shift+F` | 全画面表示を切り替え |
| `Alt+1 / Alt+2 / Alt+3` | 幅を 1/3、1/2、2/3 に設定 |
| `Alt+0` | すべての列幅を均等化 |
| `Alt+Shift+R` | 更新（ウィンドウを再列挙） |

## Windows 起動時に自動起動

```bash
openniri-cli autostart enable
```

ログイン時にデーモンを起動するため、`HKCU\Software\Microsoft\Windows\CurrentVersion\Run` にレジストリエントリを書き込みます。無効にするには：

```bash
openniri-cli autostart disable
```

## 設定とランタイムのパス

設定ファイル：

- `%APPDATA%\openniri\config\config.toml`

状態データ：

- `%APPDATA%\openniri\data\workspace-state.json`

デーモンログ：

- `%TEMP%\openniri-daemon.log`
- `%TEMP%\openniri-daemon.err.log`

## アーキテクチャ

OpenNiri-Windows は Rust ワークスペースです。

| Crate | 役割 |
|---|---|
| `openniri-core-layout` | プラットフォーム非依存のレイアウトエンジン |
| `openniri-platform-win32` | Win32 連携とウィンドウ操作 |
| `openniri-ipc` | 名前付きパイプのコマンド/レスポンスプロトコル |
| `openniri-daemon` | ランタイムイベントループと状態管理 |
| `openniri-cli` | ユーザー向けコマンドラインインターフェース |

技術ドキュメント：

- [はじめに](docs/GETTING_STARTED.md) - ステップごとのガイド
- [設定](docs/CONFIGURATION.md) - 設定項目の完全なリファレンスと例
- [トラブルシューティング](docs/TROUBLESHOOTING.md) - よくある問題とデバッグガイド
- [仕様](docs/SPEC.md) - 設計仕様
- [アーキテクチャ](docs/ARCHITECTURE.md) - アーキテクチャの概要
- [Windows の制約](docs/WINDOWS_CONSTRAINTS.md) - Win32 プラットフォームの制約

## プラットフォームの制約

OpenNiri-Windows は**ウィンドウコントローラー**であり、コンポジターではありません。

- コンポジターは引き続き DWM が担います。
- 管理者権限で実行されているウィンドウや保護されたウィンドウでは、配置やスタイルの変更が拒否される場合があります。
- アプリケーションのフレームワーク（Win32/WPF/Electron/UWP）によって動作が異なる場合があります。

## コントリビューション

`CONTRIBUTING.md` を参照してください。

## ライセンス

GPL-3.0。`LICENSE` を参照してください。
