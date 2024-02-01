
# Aivis

💠 **Aivis:** **AI** **V**oice **I**mitation **S**ystem

<img width="100%" alt="image" src="https://github.com/tsukumijima/Aivis/assets/39271166/c5b2a9cd-74ec-4b4b-8981-4ed81d0f4345">

## Overview

**Aivis は、高音質で感情豊かな音声を生成できる [Bert-VITS2](https://github.com/fishaudio/Bert-VITS2) 用のデータセットの作成・学習・推論を、オールインワンで行えるツールです。**

通常、専用に作成された音声コーパス以外の音源から学習用データセットを作成するには、膨大な手間と時間がかかります。  
Aivis では、**一般的な音源からデータセットを作成するための工程を AI で可能な限り自動化し、さらに最終的な人力でのアノテーション作業を簡単に行える Web UI を通して、データセット作成の手間と時間を大幅に削減します。**  
さらに Bert-VITS2 でのモデルの学習や推論 (Web UI の起動) も、簡単なコマンドひとつで実行できます。

https://github.com/tsukumijima/Aivis/assets/39271166/dfae6b5d-da73-477b-b316-dc077b06e6ef

大元の音源の量・質・話し方にもよりますが、上のサンプル音声の通り、専用に作成された音声コーパスを使い学習させたモデルと比べても遜色ないクオリティの音声を生成できます。  
Bert-VITS2 の事前学習モデル自体の性能が極めて高いようで、私の環境では Aivis で作成したわずか7分弱のデータセットを学習させたモデルでも、かなり近い声質の明瞭かつ感情豊かな音声を生成できています。

> [!NOTE]  
> Aivis では、実用途に合わせて細部を調整した [フォーク版の Bert-VITS2](https://github.com/tsukumijima/Bert-VITS2) を利用しています。  
> 今のところ学習/推論アルゴリズムは変更していません。Web UI を日本語化したことや学習時に必要なモデルを自動でダウンロードできることなど以外は、オリジナルの Bert-VITS2 ([Japanese-Extra ブランチ](https://github.com/fishaudio/Bert-VITS2/tree/Japanese-Extra)) と同等です。

## Installation

Linux (Ubuntu 20.04 LTS) x64 でのみ検証しています。  
CUDA / cuDNN 関連ライブラリ (.so) は基本 `poetry install` 時に pip wheels としてインストールされるため、別途 CUDA / cuDNN をインストールする必要はないと思われます。

Windows でもある程度動くように調整しているつもりですが、動作確認は取れていないためお勧めしません。Windows では WSL2 を使ってください。

> [!NOTE]  
> 手元に環境がないため WSL2 での動作検証はできていませんが、動作報告はいただいています。  
> WSL2 で動かす場合、NVIDIA GPU と CUDA のサポートが追加されている Windows 11 または Windows 10 (21H2 以降) が必要です。  
> なお、NVIDIA GPU ドライバは Windows 側にのみインストールする必要があります。WSL2 側にはインストールしないでください。

当然ですが、Aivis の実行には NVIDIA GPU が必要です。  
Geforce GTX 1080 (VRAM 8GB) での動作を確認しています。VRAM はおそらく最低 8GB は必要です (VRAM 12GB のグラボが欲しい…) 。

### Non-Docker

Docker を使わない場合、事前に Git・Python 3.11・Poetry・FFmpeg がインストールされている必要があります。

```bash
# サブモジュールが含まれているため --recurse を付ける
git clone --recurse https://github.com/tsukumijima/Aivis.git

# 依存関係のインストール
cd Aivis
poetry env use 3.11
poetry install --no-root

# ヘルプを表示
./Aivis.sh --help
```

以前インストールした環境を最新版に更新する場合は、以下のコマンドを実行してください。

```bash
git pull
git submodule update --init --recursive
poetry install --no-root
```

### Docker

Docker を使う場合、事前に Git・Docker がインストールされている必要があります。  
Docker を使わない場合と比べてあまり検証できていないため、うまく動かないことがあるかもしれません。


```bash
# サブモジュールが含まれているため --recurse を付ける
git clone --recurse https://github.com/tsukumijima/Aivis.git

# 依存関係のインストール
cd Aivis
./Aivis-Docker.sh build

# ヘルプを表示
./Aivis-Docker.sh --help
```

以前インストールした環境を最新版に更新する場合は、以下のコマンドを実行してください。

```bash
git pull
git submodule update --init --recursive
./Aivis-Docker.sh build
```

## Dataset Directory Structure

Aivis のデータセットディレクトリは、5段階に分けて構成されています。

- **01-Sources:** データセットにする音声をそのまま入れるディレクトリ
  - データセットの素材にする音声ファイルをそのまま入れてください。
    - 基本どの音声フォーマットでも大丈夫です。`create-segments` での下処理にて、自動的に wav に変換されます。
    - 背景 BGM の除去などの下処理を行う必要はありません。`create-segments` での下処理にて、自動的に BGM や雑音の除去が行われます。
    - 数十分〜数時間ある音声ファイルの場合は `create-segments` での書き起こしの精度が悪くなることがあるため、事前に10分前後に分割することをおすすめします。
  - `create-segments` サブコマンドを実行すると、BGM や雑音の除去・書き起こし・一文ごとのセグメントへの分割・セグメント化した音声の音量/フォーマット調整が、すべて自動的に行われます。
- **02-PreparedSources:** `create-segments` サブコマンドで下処理が行われた音声ファイルと、その書き起こしテキストが入るディレクトリ
  - `create-segments` サブコマンドを実行すると、`01-Sources/` にある音声ファイルの BGM や雑音が除去され、このディレクトリに書き起こしテキストとともに保存されます。
  - `create-segments` の実行時、このディレクトリに当該音声の下処理済みの音声ファイルや書き起こしテキストが存在する場合は、そのファイルが再利用されます。
  - 下処理済みの音声ファイル名は `02-PreparedSources/(01-Sourceでのファイル名).wav` となります。
  - 書き起こしテキストのファイル名は `02-PreparedSources/(01-Sourceでのファイル名).json` となります。
    - 書き起こしの精度がよくない (Whisper の音声認識ガチャで外れを引いた) 場合は、書き起こしテキストの JSON ファイルを削除してから `create-segments` を実行すると、再度書き起こし処理が行われます。
- **03-Segments:** `create-segments` サブコマンドでセグメント化された音声ファイルが入るディレクトリ
  - `create-segments` サブコマンドを実行すると、`02-PreparedSources/` 以下にある音声ファイルが書き起こし文や無音区間などをもとに一文ごとにセグメント化され、このディレクトリに保存されます。
  - セグメントデータのファイル名は `03-Segments/(01-Sourceでのファイル名)/(4桁の連番)_(書き起こし文).wav` となります。
    - 基本発生しませんが、万が一書き起こし文がファイル名の最大長を超える場合は、ファイル名が切り詰められ、代わりにフルの書き起こし文が `03-Segments/(01-Sourceでのファイル名)/(4桁の連番)_(書き起こし文).txt` に保存されます。
  - なんらかの理由でもう一度セグメント化を行いたい場合は、`03-Segments/(01-Sourceでのファイル名)/` を削除してから `create-segments` を実行すると、再度セグメント化が行われます。
- **04-Datasets:** `create-datasets` サブコマンドで手動で作成されたデータセットが入るディレクトリ
  - `create-datasets` サブコマンドを実行すると Gradio の Web UI が起動し、`03-Segments/` 以下にある一文ごとにセグメント化された音声と書き起こし文をもとにアノテーションを行い、手動でデータセットを作成できます。
  - `03-Segments/` までの処理は AI 技術を使い完全に自動化されています。
    - 調整を重ねそれなりに高い精度で自動生成できるようになった一方で、他の人と声が被っていたり発音がはっきりしないなど、データセットにするにはふさわしくない音声が含まれていることもあります。
    - また、書き起こし文が微妙に誤っていたり、句読点がなかったりすることもあります。
    - さらに元の音声に複数の話者の声が含まれている場合、必要な話者の音声だけを抽出する必要もあります。
  - `create-datasets` サブコマンドで起動する Web UI は、どうしても最後は人力で行う必要があるアノテーション作業を、簡単に手早く行えるようにするためのものです。
    - 話者の選別 (データセットから除外することも可能)・音声の再生・音声のトリミング (切り出し)・書き起こし文の修正を一つの画面で行えます。
    - 確定ボタンを押すと、そのセグメントが指定された話者のデータセットに追加されます ([このデータを除外] ボタンが押された場合はデータセットへの追加がスキップされる) 。
    - `create-datasets` サブコマンドによって、`03-Segments/` 以下のセグメント化された音声ファイルが変更されることはありません。
  - データセットは音声ファイルが `04-Datasets/(話者名)/audio/wavs/(連番).wav` に、書き起こし文が `04-Datasets/(話者名)/filelists/speaker.list` にそれぞれ保存されます。
    - このディレクトリ構造は Bert-VITS2 のデータセット構造に概ね準拠したものですが、`config.json` など一部のファイルやディレクトリは存在しません。
    - Bert-VITS2 の学習処理によって、`04-Datasets/` 以下のデータセットが変更されることはありません。
- **05-Models:** `train` サブコマンドで生成された、Bert-VITS2 の学習済みモデルが入るディレクトリ
  - 実体は `Bert-VITS2/Data/` へのシンボリックリンクです。
  - `train` サブコマンドを実行すると、`04-Datasets/` 以下の指定された話者のデータセットディレクトリが Bert-VITS2 側にコピーされ、Bert-VITS2 の学習処理が開始されます。
    - 生成された学習済みモデルは、`05-Models/(話者名)/models/` 以下に保存されます。
    - 再度学習を行う場合は、`05-Models/(話者名)/` ディレクトリを削除してから再度 `train` サブコマンドを実行してください。
  - `infer` サブコマンドを実行すると、Bert-VITS2 の推論用 Web UI が起動されます。
    - Bert-VITS2 の推論用 Web UI によって、`05-Models/` 以下の学習済みモデルが変更されることはありません。

## Usage

概ね前述した通りですが、念のためにここでも説明します。  
ここでは、学習するモデルの名前を「MySpeaker1」「MySpeaker2」とします。

### 1. データセットの準備

`01-Sources/` 以下に、データセットにする音声ファイルをそのまま入れます。

基本どの音声フォーマットでも大丈夫です。`create-segments` での下処理にて、自動的に wav に変換されます。  
また、背景 BGM の除去などの下処理を行う必要はありません。`create-segments` での下処理にて、自動的に BGM や雑音の除去が行われます。

なお、数十分〜数時間ある音声ファイルの場合は `create-segments` での書き起こしの精度が悪くなることがあるため、事前に10分前後に分割することをおすすめします。

### 2. データセットの下処理とセグメント化

```bash
# Non-Docker
./Aivis.sh create-segments

# Docker
./Aivis-Docker.sh create-segments
```

実行すると、音源抽出 AI の [Demucs (htdemucs_ft)](https://github.com/facebookresearch/demucs) により `01-Sources/` 以下にある音声ファイルの BGM や雑音が除去され、書き起こしテキストとともに `02-PreparedSources/` 以下に保存されます。  
書き起こしテキストは音声認識 AI の [faster-whisper (large-v3)](https://github.com/SYSTRAN/faster-whisper) によって生成され、[stable-ts](https://github.com/jianfch/stable-ts) によってアライメントされます。  
すでに `02-PreparedSources/` 以下に当該音声の下処理済みの音声ファイルや書き起こしテキストが存在する場合は、そのファイルが再利用されます。

上記の処理が完了すると、`02-PreparedSources/` 以下にある音声ファイルが書き起こし文や無音区間などをもとに一文ごとにセグメント化され、`03-Segments/` 以下に保存されます。  
すでに `03-Segments/` 以下に当該音声のセグメント化された音声ファイルが存在する場合は、当該音声のセグメント化はスキップされます。

> [!NOTE]  
> `create-segments` サブコマンドにはいくつかオプションがあります。  
>   
> `--force-transcribe` オプションを指定すると、既に書き起こしテキストが存在する音声ファイルでも、再度書き起こし処理が行われます。  
> Whisper の書き起こし結果にはガチャ (ランダム性) があり、稀にハルシネーションと無限ループだらけの使い物にならない書き起こし結果が出てくることもあります。  
> 随時ログに出力される書き起こし結果が微妙だと感じた際は、このオプションを指定して再度書き起こし処理を行うことをおすすめします。  
>   
> `--whisper-model` オプションを指定すると、Whisper の書き起こしに使うモデルを指定できます。  
> デフォルトは `large-v3` です。モデルの変更は GPU 性能の関係で `large-v3` が動かない場合のみ行ってください。  
>   
> `--no-use-demucs` オプションを指定すると、Demucs による BGM や雑音の除去を行わず、そのままの音声ファイルを使って書き起こし処理を行います。  
> デフォルトでは、すべての音声ファイルに対して Demucs による BGM や雑音の除去が行われます。
>   
> `--no-trim-silence` オプションを指定すると、先頭と末尾の無音区間のトリミングを行わずにセグメント化された音声ファイルを保存します。  
> デフォルトでは、セグメント化された音声ファイルを保存する際に、先頭と末尾の無音区間のトリミングが行われます。

### 3. データセットの作成 (アノテーション)

<img width="100%" alt="image" src="https://github.com/tsukumijima/Aivis/assets/39271166/d2c2009d-5195-49b2-8771-980a3d317fa6"><br>

```bash
# Non-Docker
./Aivis.sh create-datasets '*' 'MySpeaker1,MySpeaker2'

# Docker
./Aivis-Docker.sh create-datasets '*' 'MySpeaker1,MySpeaker2'
```

Aivis でのデータセット作成工程は、大半が `create-segments` サブコマンドで自動化されています。  
しかし、話者やセグメント自体の選別・書き起こし文の修正・うまく切り出せていない音声のトリミングなどの仕上げのアノテーション作業は、どうしても人力で行う必要があります。

`create-datasets` サブコマンドを実行すると、Gradio の Web UI が起動します。  
この Web UI から `03-Segments/` 以下の一文ごとにセグメント化された音声と書き起こし文をもとにアノテーションを行い、人力でアノテーションを行った最終的なデータセットを作成できます。

`create-datasets` サブコマンドの第一引数には、`03-Segments/` 以下に自動生成されている、セグメント化された音声ファイルのディレクトリ名を指定します。通常は `01-Sources/` 以下の音声ファイル名と同じです。  
内部では Glob が使われているため、ワイルドカード (`*`) を活用し、複数のディレクトリのアノテーション作業を一括で実行できます。

`create-datasets` サブコマンドの第二引数には、データセットを作成する話者の名前をカンマ区切りで指定します。  
ここで指定した話者のデータセットが、`04-Datasets/` 以下に作成されます。  
Web UI 上では、セグメント化された音声ファイルごとにどの話者に割り当てるか、あるいはデータセットから除外するかを選択できます。

Web UI 上で確定ボタンを押すと、次のセグメントのアノテーション作業に移ります。  
実装上、一度確定したセグメントのアノテーションをやり直すことはできません。間違いがないかよく確認してください。  
作成中のデータセットの進捗ログは、`create-datasets` サブコマンドの標準出力に表示されます。

> [!TIP]
> `--accept-all` オプションを指定すると、UI を表示せずにすべての音声ファイルを一括処理できます。  
> あらかじめ第一引数で指定したディレクトリパターン内の音声が第二引数で指定した単一話者だけなことが分かっていて、さらに書き起こし文を調整する必要がないときは、このオプションを使うと大幅に作業時間を短縮できます。  
> セグメントがどの話者に対応するかは自動判定できないため、`--accept-all` 指定時に複数の話者を指定することはできません。

> [!NOTE]  
> すでにデータセットが途中まで作成されている状態で再度 `create-datasets` サブコマンドを実行すると、途中まで作成されているデータセットの次の連番から、データセット作成が再開されます。  
> 最初からアノテーション作業をやり直したい場合は、`04-Datasets/` 以下の話者ごとのデータセットディレクトリを削除してから、再度 `create-datasets` サブコマンドを実行してください。

```bash
# Non-Docker
./Aivis.sh check-dataset 'MySpeaker1'

# Docker
./Aivis-Docker.sh check-dataset 'MySpeaker1'
```

`check-dataset` サブコマンドを実行すると、指定された話者のデータセットディレクトリにある音声ファイルと書き起こし文、音声ファイルの総時間 (秒) を確認できます。  

`check-dataset` サブコマンドの第一引数には、データセットを確認したい話者の名前 (`04-Datasets/` 以下のディレクトリ名と一致する) を指定します。ワイルドカードは使えないため注意してください。

### 4. 学習の実行

<img width="100%" alt="image" src="https://github.com/tsukumijima/Aivis/assets/39271166/6d67a57b-d53e-465c-a454-d94d981278ad"><br>

```bash
# Non-Docker
./Aivis.sh train 'MySpeaker1' --steps 8000 --batch-size 4

# Docker
./Aivis-Docker.sh train 'MySpeaker1' --steps 8000 --batch-size 4
```

`train` サブコマンドを実行すると、指定された話者のデータセットディレクトリのコピー、`config.json` などの学習時に必要なファイルの生成、引数に応じた `Bert-VITS2/config.yml` の自動書き換えといった下処理の後、Bert-VITS2 の学習処理が開始されます。

[Bert-VITS2 の事前学習モデル](https://huggingface.co/Stardust-minus/Bert-VITS2-Japanese-Extra/tree/main) がまだダウンロードされていない場合は、実行時に `.cache/` 以下に自動的にダウンロードされます。

学習時には、`--epochs` (エポック数) と `--steps` (ステップ数) のどちらかを指定する必要があります。  
`--batch-size` はバッチサイズを指定するオプションで、指定しなかった場合は `4` に設定されます。

NVIDIA GPU のスペックにもよりますが、学習の完了には相応の時間がかかります。  
Geforce GTX 1080 (バッチサイズ: 2) で 8000 ステップ学習させたときは3時間程度かかりました。

一般的に、2000 ステップ 〜 4000 ステップで十分似た声質になるようですが、データセットの数や質にも依存します。  
最大でも 8000 ステップも学習させれば、かなり自然な音声が生成できるようになります。

> [!IMPORTANT]  
> ステップ数は、`(データセットの総数 ÷ バッチサイズ) × エポック数` で求められます。  
> 逆に必要なエポック数は、`ステップ数 ÷ (データセットの総数 ÷ バッチサイズ)` で求められます。  
> データセットの総数が少ない場合は、エポック数を増やして目標ステップ数を超えるように調整してください。
> `--epochs` の代わりに `--steps` オプションを使うと、バッチサイズ・データセットの総数から自動的にエポック数を計算して学習を行います。

> [!TIP]  
> VRAM 不足で実行途中に CUDA Out Of Memory エラーが出る場合は、`--batch-size` で学習時のバッチサイズを小さくしてください。  
> Geforce GTX 1080 ではバッチサイズ 2 〜 3 でギリギリな印象です。

学習中は、標準出力に学習の進捗ログが表示されます。

学習したモデルは `05-Models/(話者名)/models/` 以下に保存されます。  
`05-Models/(話者名)/` ディレクトリを別 PC の Aivis の `05-Models/` 以下にコピーすることで、学習済みモデルを別の環境に移行できます。

> [!NOTE]  
> 学習済みモデルは 1000 ステップごとに異なるファイル名で保存されます。  
> もしモデルディレクトリに `G_7000.pth` が存在するなら、7000 ステップまで学習させたモデルです。

### 5. 学習済みモデルの推論

<img width="100%" alt="image" src="https://github.com/tsukumijima/Aivis/assets/39271166/b38ce1e9-abae-4e0f-b9d4-33f896ba4408"><br>

```bash
# Non-Docker
./Aivis.sh infer 'MySpeaker1' --model-step 5000

# Docker
./Aivis-Docker.sh infer 'MySpeaker1' --model-step 5000
```

`infer` サブコマンドを実行すると、引数で指定された話者の学習済みモデルを使って音声を生成できる、推論用 Web UI を起動できます。

> [!TIP]
> `--model-step` はオプションで、指定しなかった場合は一番最後に保存されたステップ数のモデルが使われます。  
> もし一番最後に保存されたステップ数のモデルでの性能が芳しくない場合は、過学習気味かもしれません。  
> より前の低いステップ数のモデルの方が発声やイントネーションが安定した音声を生成できる場合もあります。

この Web UI はオリジナルの Bert-VITS2 の Web UI を日本語化し、より使いやすくなるよう画面構成を整理し説明文を変更したものです。  
コマンド実行時に、引数に合わせて自動的に `Bert-VITS2/config.yml` が書き換えられます。

Web UI にはいくつかボタンがありますが、基本は喋らせたい音声を入力して、「音声を行ごとに分割して生成 (おすすめ)」をクリックするだけです。

> [!NOTE]
> Bert-VITS2 は、読み上げテキストが示す感情に合わせて、自動的に抑揚や感情表現を調整します。  
> 例えば「ありがとうございます！」なら前向きに明るい声で、「とても残念です…。」なら残念そうな声で読み上げできます。  
> 句読点の有無や、！？… などの文末記号の使い方次第で、抑揚や感情表現が大きく変わります。  
> 意図した表現にならないときは、読み上げテキストを工夫してみてください。

> [!TIP]
> Bert-VITS2 は行の最初の方の文から読み取れる感情表現を後まで引き継ぐ傾向があるため、まとまった文ごとに改行で区切り、[音声を行ごとに分割して生成] ボタンを押すとより自然な音声を生成できます。  
> ただし、「とても嬉しいです。しかし残念です。」のように真逆の感情を含む文では、同じ行に含めた方がより自然な繋がりになることもあります。

> [!TIP]
> 音声合成時に抑揚の強さ (SDP Ratio) を指定できます。基本 0.2 〜 0.6 の範囲がおすすめです。  
> 0.0 に近いほど抑揚が弱く読み上げが遅くなり、1.0 に近いほど抑揚が強く読み上げが早くなります。  
> 0.0 では棒読みに、0.6 ではより抑揚の強い感情のこもった音声になります。  
> 0.2 では抑揚が比較的少ない抑制的なトーンで読み上げます。  
> 0.6 以上にしても抑揚はあまり変わらない印象です。むしろ発声が不安定になることもあります。  

## License

[MIT License](License.txt)
