# OctantVectorPlugins

> **To use BeatOctant** (install, known issues): [Getting Started](https://github.com/BeatOctant/OctantSpecs/blob/main/GettingStarted_en.md) · [はじめかた（日本語）](https://github.com/BeatOctant/OctantSpecs/blob/main/GettingStarted_ja.md)
>
> BeatOctant is written mostly with AI coding agents (Anthropic's Claude). The maintainer decides what to build and reviews every change. It does not generate maps; it recommends existing maps from BeatSaver. ([作り方（AI の使い方）](https://github.com/BeatOctant/OctantSpecs/blob/main/README.md#作り方ai-の使い方))
>
> OctantVectorPlugins adds audio and image model views to OctantVector, the engine of BeatOctant, a Beat Saber map recommender (the OctantPath mod for Beat Saber PC and OctantPath for Quest).
>
> BeatOctant is an unofficial fan project for Beat Saber. It is not affiliated with, endorsed by, or sponsored by Beat Games or Meta. Beat Saber and Meta Quest are trademarks of their respective owners. Beat Games・Meta とは関係のない非公式の MOD・ツールです。
>
> 報告と案は Discord の BeatOctant サーバー（バグは `#bug-reports`、案は `#feedback-ideas`）で受けます。外からの pull request は受けていません。

[OctantVector](https://github.com/BeatOctant/OctantVector) に音や画像のモデルを使うビューを足すプラグインです。系統ごとに一つのフォルダで、それぞれが独立したパッケージ（自分の LICENSE 付き）です。BeatOctant 全体の中での位置は [OctantSpecs の概要](https://github.com/BeatOctant/OctantSpecs#概要)にあります。

| フォルダ | パッケージ | ビュー | ライセンス |
|---|---|---|---|
| `clap/` | `octantvector-clap` | `clap_htsat_unfused`、`vocal_clap_htsat_unfused`、`clap_concepts` | MIT（モデルは Apache-2.0） |
| `mert/` | `octantvector-mert` | `mert_v1_95m`、`vocal_mert_v1_95m` | CC BY-NC 4.0（モデルのコードの写しを含むため。`mert/NOTICE`） |
| `wavlm/` | `octantvector-wavlm` | `vocal_wavlm_sv` | MIT（モデルは CC BY-SA 3.0） |
| `whisper/` | `octantvector-whisper` | `whisper_language`（歌の言語。歌のある箇所の言語の割合） | MIT（モデルはモデルカードでは Apache-2.0） |
| `siglip/` | `octantvector-siglip` | `siglip_cover`（カバー画像） | MIT（モデルはモデルカードでは Apache-2.0） |

`vocal_` のビューは Demucs で分けた歌声を使うので、`[vocal]` 付きで入れます（`octantvector[vocal]` が入ります）。

## プラグイン API は不安定です

プラグインは `octantvector.plugin` だけを使います。この API はまだ固まっておらず、変わるたびに `PLUGIN_API` の番号が上がります。各パッケージは書かれた番号を `OCTANTVECTOR_PLUGIN_API` に持ち、OctantVector の番号と違うと読み込まれません（そのビューを頼んだときに理由が出ます）。OctantVector とプラグインは、どちらも最新の版どうしで使ってください。

各ビューは、モデルの出所と条件を `credits` に持ちます。OctantVector が DB を書くとき、マニフェストのそのビューの `credits` に入ります（ボーカルのビューには Demucs の分も足されます）。

## 入れ方

OctantVector と同じ環境に、使うものだけ入れます。

```bash
pip install "octantvector-clap[vocal] @ git+https://github.com/BeatOctant/OctantVectorPlugins#subdirectory=clap"
pip install "octantvector-mert[vocal] @ git+https://github.com/BeatOctant/OctantVectorPlugins#subdirectory=mert"
pip install "octantvector-wavlm[vocal] @ git+https://github.com/BeatOctant/OctantVectorPlugins#subdirectory=wavlm"
pip install "octantvector-whisper @ git+https://github.com/BeatOctant/OctantVectorPlugins#subdirectory=whisper"
pip install "octantvector-siglip @ git+https://github.com/BeatOctant/OctantVectorPlugins#subdirectory=siglip"
```

main から入れます。入れるとエントリポイント `octantvector.encoders` で OctantVector に登録されます。

## モデルのファイル

プラグインも OctantVector も、モデルを取りに行きません。各ビューは使うモデル（Hugging Face のリポジトリと固定した revision、ファイル名）を宣言し、`models_dir` の下の `hf/<repo>/<revision>/` からだけ読みます。ファイルは OctantHarvest の `octantharvest models` が置きます（契約1 の `models(views)` で何が要るかを聞きます）。

## 開発とテスト

```bash
python -m venv .venv
.venv/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu
.venv/bin/pip install -e "/path/to/OctantVector[embed]" -e "./clap[dev]" -e "./mert[dev]" -e "./wavlm[dev]" -e "./whisper[dev]" -e "./siglip[dev]"
.venv/bin/python -m pytest -q clap/tests mert/tests wavlm/tests whisper/tests siglip/tests
```

テストはネットワークにも本物の重みにも触れません。小さな設定でランダムに初期化したモデルを一時的な `models_dir` に置き、決まった計算と同じベクトルになるか、形と決まり（切り出す位置、歌声の判定、正規化など）が合うかを確かめます。本物の重みでの値は、開発用の PC で確かめます。

### CI

`.github/workflows/test.yml` がプルリクエストでテストを動かします。

## ライセンス

パッケージ（フォルダ）ごとに違い、リポジトリ全体のライセンスはありません。各フォルダの LICENSE と、上の表を見てください（`mert/` は非営利の CC BY-NC 4.0 で、[NOTICE](mert/NOTICE) があります）。モデルの重みの条件はモデルカードを見てください。
