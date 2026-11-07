 [简体中文](README.md) \| [English](README_EN.md) \|[日本語](README_JA.md)

# Minecraft パーティクル画像データパック生成ツール

[![GitHub
Stars](https://img.shields.io/github/stars/TeaCafeStudio/Minecraft-ParticleImage-Generator?style=flat-square)](https://github.com/TeaCafeStudio/Minecraft-ParticleImage-Generator/stargazers)
[![GitHub
Forks](https://img.shields.io/github/forks/TeaCafeStudio/Minecraft-ParticleImage-Generator?style=flat-square)](https://github.com/TeaCafeStudio/Minecraft-ParticleImage-Generator/network/members)
[![GitHub
Issues](https://img.shields.io/github/issues/TeaCafeStudio/Minecraft-ParticleImage-Generator?style=flat-square)](https://github.com/TeaCafeStudio/Minecraft-ParticleImage-Generator/issues)
[![GitHub
Releases](https://img.shields.io/github/v/release/TeaCafeStudio/Minecraft-ParticleImage-Generator?style=flat-square)](https://github.com/TeaCafeStudio/Minecraft-ParticleImage-Generator/releases)

本プログラムは Copilot-GPT-3.5
の協力を得て開発されました。不具合や問題が発生した場合は、Issue
を作成してください。

PNG、GIF、APNG 画像を Minecraft Java Edition のデータパック ZIP
に変換する Windows デスクトップアプリケーションです。UI には PySide6
を使用しています。アニメーション解析、パーティクルシミュレーション、パフォーマンスプリセット、リスク予測、バージョン別コマンド、ZIP
の構築は、それぞれ独立したモジュールに分離されています。

## プロジェクト構成

``` text
ParticleGenerator/
├── main.py                         # QApplication のエントリーポイント
├── app_info.py                     # アプリケーション名とバージョン
├── build.py                        # アイコンを自動生成し、単体 EXE をビルド
├── ui/
│   ├── main_window.py              # 画像の読み込み、パラメーター設定、エクスポート画面
│   └── preview.py                  # 独立した画像プレビューおよびパーティクルシミュレーション
├── core/
│   ├── image_parser.py              # PNG/GIF/APNG の読み込み、拡大縮小、透明ピクセルの除外
│   ├── particle_model.py            # ローカル座標のパーティクルと Pivot のデータモデル
│   ├── transform.py                 # 拡大縮小、回転、実行位置オフセットの処理パイプライン
│   ├── rotation.py                  # Euler 角による回転計算
│   ├── coordinate_system.py         # ワールド/実行者/プレイヤー視点の座標変換
│   ├── renderer.py                  # モデルを Minecraft の function コマンドに変換
│   ├── particle_generator.py        # 旧形式の呼び出しと互換性を保つモデル生成エントリーポイント
│   ├── datapack_builder.py          # pack.mcmeta、function ファイル、ZIP の構築
│   ├── performance.py               # 低/中/高解像度とリソース予算
│   ├── prediction.py                # コマンド数、ファイルサイズ、実行リスクの予測
│   ├── app_config.py                # AppData に保存するユーザー設定
│   ├── app_logging.py               # ローテーション対応エラーログ
│   └── resource_paths.py            # ソース実行時とパッケージ版のリソースパス検証
├── version/
│   ├── base.py                      # バージョンアダプターのインターフェース
│   ├── registry.py                  # 対応バージョンの登録一覧
│   ├── mc1204.py                    # 1.20.4 のコマンド構文とメタデータ
│   ├── mc1205.py                    # 1.20.5/1.20.6 アダプター
│   └── mc121x.py                    # 1.21.x アダプター
├── minecraft/
│   ├── mc1204_renderer.py           # 1.20.4 のパーティクル構文
│   └── mc1205_renderer.py           # 1.20.5 以降のパーティクル構文
├── editor/
│   ├── viewport.py                  # 回転、パン、ズームに対応した 3D モデルビュー
│   ├── camera.py                    # オービットカメラ
│   └── gizmo.py                     # XYZ 座標 Gizmo
├── resources/
│   └── app_icon.ico                 # ビルドスクリプトが生成して埋め込むアプリアイコン
├── build/
│   └── ParticleGenerator.spec       # PyInstaller の単一ファイル設定
└── requirements.txt
```

処理の流れ：

`画像 → ParsedImage → ParticleModel（ローカル座標）→ ビジュアル編集 → Transform → MinecraftRenderer → DatapackBuilder → ZIP`

デフォルトの Pivot は画像の左下隅です。右方向が +X、上方向が +Y
です。モデル自体には Minecraft
コマンドを保存しません。予測またはエクスポート時にのみ、バージョン別レンダラーが
function
のテキストを生成します。新しいバージョンに対応する場合は、`version/`
にメタデータアダプターを登録し、`minecraft/`
に対応するコマンドレンダラーを登録してください。

## ローカルでの実行

一般ユーザーは Python をインストールせずに
`build/dist/ParticleGenerator.exe`
を直接実行できます。開発者はビルド環境として Windows 10/11 と Python
3.10 以降が必要です。

``` powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python main.py
```

## 単体 EXE のビルド

PyInstaller は、他の OS から Windows 用 EXE
をクロスビルドできません。Windows
のビルド環境で依存パッケージをインストールした後、次のコマンドを実行してください。

``` powershell
python build.py
```

スクリプトはアプリケーションアイコンを自動生成し、PyInstaller の spec
ファイルを使用して単一ファイルの GUI
アプリケーションをビルドします。ソフトウェアのバージョンは **1.3.0**
です。生成されたファイルは `build/dist/ParticleGenerator.exe`
に保存されます。PySide6、Pillow、アプリアイコンはプログラムに同梱されるため、実行先の
PC に Python
や追加のランタイム環境をインストールする必要はありません。初回起動時は、同梱されたランタイムファイルの展開に少し時間がかかる場合があります。

ユーザー設定は
`%APPDATA%\Minecraft Particle Image Generator\config.json`
に保存されます。ローテーションログは
`%LOCALAPPDATA%\Minecraft Particle Image Generator\logs\application.log`
に保存されます。

## 現在の対応範囲

-   **データパック：** Minecraft Java Edition
    1.20.4、1.20.5、1.20.6、および 1.21～1.21.11
    の登録済みバージョンに対応しています。対応するすべてのバージョンで
    `function/` ディレクトリを使用します。1.21.9 以降では
    `min_format`/`max_format` メタデータを使用します。
-   **パーティクル：**
    `minecraft:dust`、`minecraft:flame`、`minecraft:cloud`、`minecraft:end_rod`。
-   アルファ値が 0 のピクセルは無視され、0
    以外のアルファ値はすべて表示対象として扱われます。Dust
    以外のパーティクルでは、ピクセルごとの RGB
    カラーを指定できません。Dust のサイズ範囲は 0.01～4 です。
-   対応するすべてのバージョンで、function のパスは
    `data/<namespace>/function/<name>.mcfunction` 形式です。
-   3D
    ビューポートは、左ボタンで回転、右/中ボタンでパン、マウスホイールでズームできます。グリッド、XYZ
    軸、Pivot
    が表示されます。変換、パーティクルサイズ、アルファ値、Pivot、向きのモードは
    UI から編集できます。
-   **向きのモード：** ワールド方向に固定、function
    の実行者の向きを継承、最も近いプレイヤーの視点方向を継承、の 3
    種類です。最初の 2 つのモードでは `with block`
    は不要です。モデルの位置には実行コンテキストの相対座標を使用します。視点モードでは
    `execute rotated as @p` によって向きを上書きします。
-   ローカル座標の各点は、Pivot を基準に XYZ スケーリング、Euler XYZ
    回転、実行位置オフセットの順に処理され、その後 Minecraft の `~`
    または `^` 座標として出力されます。通常の function
    呼び出しでは、実行位置が選択した Pivot
    のアンカーになります。デフォルトは画像の左下隅です。
-   アニメーション再生では、一時的なエンティティタグを使用して、再生開始者の実行位置と向きを保持します。プレイヤーが
    `start`/`stop` function を実行します。Dust
    などのパーティクルコマンドは、パーティクルごとのアルファ値やスプライト回転に対応していません。アルファ値はモデルプレビューと完全に透明なパーティクルの非表示に使用されます。
-   静的モデルの場合、`/function <namespace>:<name>`
    を実行すると、現在の function 実行位置が選択した Pivot
    のアンカーになります。デフォルトの Pivot
    は左下隅です。プレイヤーの実行コンテキストを明示的に指定するには、`/execute as @p at @s run function <namespace>:<name>`
    を実行してください。位置指定に `with block`
    を使用しないでください。これはマクロ function にブロック NBT
    データを渡すためだけに使用します。
-   **パフォーマンスプリセット：** 低パフォーマンスモードでは画像を最大
    128×128
    に縮小し、領域ごとにピクセルを統合します。中パフォーマンスモードは最大
    500,000 ピクセル、高パフォーマンスモードは最大 1,000,000
    ピクセルに対応します。アニメーションは最大 120
    フレームで、メモリ使用量を制御するための総ピクセル予算も設定されています。
-   アニメーションの各フレームは GIF/APNG
    のフレーム時間に従って自動的にループします。生成後、`/function <namespace>:<name>/animation/start`
    を実行すると再生を開始し、`/function <namespace>:<name>/animation/stop`
    を実行すると停止します。
-   実行リスクは、1 フレームあたりのコマンド数と function
    テキストのサイズから推定されます。リスクが高い内容をエクスポートする場合は確認が必要です。パーティクルシミュレーションはレイアウトや色を確認するための近似表示であり、Minecraft
    クライアントのスクリーンショットではありません。
