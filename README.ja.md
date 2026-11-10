[English](README.md) | 日本語

# 量子乱数生成（QRNG）の教材ノートブック

量子乱数生成器（QRNG）を Qiskit でシミュレーションし、得られた乱数のエントロピーの評価、乱数の抽出、統計的検定までを行うノートブック集です。

## ノートブック

上から順に読むことを想定しています。ノートブックの本文は英語で、日本語の原文を各 Markdown セルの HTML コメント（`<!-- ... -->`）に残しています。日本語で読む場合は、セルを編集モードにしてください。

| 順 | ノートブック | 内容 | 必要な公式ツール |
|---|---|---|---|
| 1 | `C1_circuit_textbook.ipynb` | 1 量子ビットの C1 回路で乱数を作り、偏りとエントロピーを調べる | なし |
| 2 | `C2_circuit_textbook.ipynb` | 5 量子ビットに C1 回路を並べた C2 回路 | なし |
| 3 | `C3_circuit_textbook.ipynb` | 5 量子ビットの GHZ 状態を多数決で 1 ビットにする C3 回路 | なし |
| 4 | `SP800_90B_textbook.ipynb` | NIST SP 800-90B で raw data の min-entropy を推定する | `ea_non_iid` |
| 5 | `Toeplitz_extractor_textbook.ipynb` | Toeplitz-hashing extractor で一様乱数を取り出す | なし |
| 6 | `Trevisan_extractor_textbook.ipynb` | Trevisan's extractor で一様乱数を取り出す | なし |
| 7 | `SP800_22_textbook.ipynb` | NIST SP 800-22 の 15 種類の統計的検定を行う | STS 2.1.2 |
| 8 | `QRNG_pipeline.ipynb` | 回路とノイズを定義し、生成（シミュレーションまたは raw data のファイル）→ SP 800-90B → Toeplitz / Trevisan → SP 800-22 を通して実行する | `ea_non_iid`, STS |

`QRNG_pipeline.ipynb` は、4〜7 のノートブック（SP800_90B・Toeplitz・Trevisan・SP800_22）から関数の定義だけを読み込んで使うので、同じフォルダに置いたまま使ってください。

公式ツールは NIST が配布しているもので、このリポジトリには含めていません。下のセットアップの手順でダウンロードしてビルドしてください。ツールがない場合、SP800_90B と SP800_22 のノートブックはツールを使うセルを実行せず、そのことを表示します。

## セットアップ

以下のコマンドは、すべて Ubuntu（WSL の Ubuntu を含む）のターミナルで実行します。

> **リポジトリを置く場所**：STS はファイルのパスを 199 文字までしか読めず、空白を含むパスも読めません。空白を含まない短いパス（例：`~/qrng-notebooks`）に置いてください。WSL では、`/mnt/c/...`（Windows 側のフォルダ）よりも Linux 側のホームフォルダに置くほうがファイルの読み書きが速くなります。

### 1. システムのパッケージを入れる

```bash
sudo apt update
sudo apt install -y git curl unzip build-essential python3.12-venv \
  libbz2-dev libdivsufsort-dev libjsoncpp-dev libssl-dev libgmp-dev libmpfr-dev
```

`build-essential` は 2 つの公式ツールのビルドに、`lib...-dev` は `ea_non_iid` のビルドに使います。

### 2. リポジトリを取得する

```bash
git clone https://github.com/catdogpenguin/qrng-notebooks.git
cd qrng-notebooks
```

以降のコマンドは、このフォルダ（リポジトリのトップ）で実行します。

### 3. Python の仮想環境を作る

```bash
python3.12 -m venv qiskit-env
source qiskit-env/bin/activate
pip install -r requirements.txt
python -c "import qiskit, qiskit_aer; print(qiskit.__version__, qiskit_aer.__version__)"   # 2.5.2 0.17.2 と表示されれば成功
```

`requirements.txt` は、Linux の Python 3.12 の環境で `pip freeze` したもので、すべてのパッケージの版を固定しています。ほかの OS や Python の版では、インストールできないパッケージがあるかもしれません。

### 4. STS（SP 800-22 の公式ツール）をビルドする

配布ページ：<https://csrc.nist.gov/projects/random-bit-generation/documentation-and-software>（"NIST SP 800-22 Rev. 1a" の項目の `sts-2_1_2.zip`）

```bash
curl -L -o sts-2_1_2.zip https://csrc.nist.gov/CSRC/media/Projects/Random-Bit-Generation/documents/sts-2_1_2.zip
echo "0238d2f1d26e120e3cc748ed2d4c674cdc636de37fc4027c76cc2a394fff9157  sts-2_1_2.zip" | sha256sum -c -   # OK と表示されれば、ノートブックが想定している版と同じ
unzip -q sts-2_1_2.zip -d .
cd sts-2.1.2/sts-2.1.2      # zip の中でフォルダが 2 重になっている
make                        # assess ができる（警告がいくつか出るが問題ない）
./assess                    # "Usage: ./assess <stream length>" と表示されれば成功
cd ../..
```

`curl` でダウンロードできない場合は、配布ページからブラウザでダウンロードしてください。Windows の「ダウンロード」フォルダの zip は、WSL からは `/mnt/c/Users/<ユーザー名>/Downloads/sts-2_1_2.zip` です。`sha256sum -c` で `FAILED` と表示された場合は、ノートブックが想定している版と違うファイルです。

SP800_22 のノートブックは、リポジトリのトップからの相対パス `sts-2.1.2/sts-2.1.2` で STS を探します（変数 `STS_DIR`）。

### 5. `ea_non_iid`（SP 800-90B の公式ツール）をビルドする

リポジトリ：<https://github.com/usnistgov/SP800-90B_EntropyAssessment>

```bash
git clone https://github.com/usnistgov/SP800-90B_EntropyAssessment.git
cd SP800-90B_EntropyAssessment
git checkout 87c104d0ed4cbc96103e7b8b38d6f2c7e0a6b289   # ノートブックが想定している版。最新版を使う場合はこの行を省略
cd cpp
make non_iid                # ea_non_iid ができる
./ea_non_iid                # 使い方（Usage）が表示されれば成功
cd ../..
```

ビルドでヘッダファイル（`json/json.h`、`divsufsort.h`、`bzlib.h`、`gmp.h`、`mpfr.h`、`openssl/...`）が見つからないというエラーが出た場合は、1. の `apt install` のうち対応するパッケージが入っているか確かめてください。

SP800_90B のノートブックは、リポジトリのトップからの相対パス `SP800-90B_EntropyAssessment/cpp/ea_non_iid` でツールを探します（変数 `EA_NON_IID`）。

### 6. ノートブックを開く

**VS Code を使う場合**：拡張機能の WSL と Jupyter を入れ、WSL の中でこのフォルダを開きます（ターミナルで `code .`）。ノートブックを開いたら、右上の「カーネルの選択」で `qiskit-env/bin/python` を選んでください。

**JupyterLab を使う場合**：`requirements.txt` には JupyterLab 本体を含めていないので、仮想環境に追加で入れます。

```bash
source qiskit-env/bin/activate
pip install jupyterlab
jupyter lab
```

どちらの場合も、ノートブックは**ノートブックがあるフォルダ（リポジトリのトップ）を作業フォルダとして**実行してください。公式ツールの場所と結果の保存先は、そこからの相対パスで指定しています。

## フォルダの構成

セットアップ後、ノートブックを実行すると次のようになります。`qiskit-env/`、公式ツール、`results/` は `.gitignore` で git の管理から外しています。

```
.
├── README.md                        README（英語）
├── README.ja.md                     README（日本語）
├── requirements.txt
├── *.ipynb                          ノートブック
├── qiskit-env/                      Python の仮想環境（3.）
├── sts-2.1.2/sts-2.1.2/             STS（4.）
├── SP800-90B_EntropyAssessment/     ea_non_iid（5.）
└── results/                         ノートブックの出力（SAVE_RESULTS = True のときだけ）
    ├── c1/2026-11-10_153012/        C1 の raw data と結果（準備のセルを実行した日時のフォルダ）
    ├── c2/<日時>/                   C2
    ├── c3/<日時>/                   C3
    ├── sp800_90b/<日時>/            SP800_90B
    ├── toeplitz/<日時>/             Toeplitz
    ├── trevisan/<日時>/             Trevisan
    ├── pipeline/<日時>/             QRNG_pipeline
    └── sp800_22/<日時>/             SP800_22
```

どのノートブックも、準備のセル（C1〜C3・Toeplitz・Trevisan は 1.、SP800_90B・SP800_22 は 2.）の `SAVE_RESULTS = True` で保存します。C1〜C3 の 5.9（ノイズの強さを段階的に変える実験）は保存量が多いので、別のスイッチ `SAVE_SWEEP = True` のときだけ保存します。既定はどちらも `False` で、何も保存しません。保存するときは、準備のセルを実行した日時の名前のフォルダを新しく作るので、前の結果を上書きしません。日時のフォルダの中身（実験ごとのフォルダ、ファイル名の付け方、CSV の列など）は、各ノートブックの「使い方（How to use this notebook）」の「保存されるファイル（Saved files）」に書いてあります。

## 参考文献

ノートブックは、次の論文と規格にもとづいています。それぞれのノートブックにも、使った出典を書いています。

- A. Gnatowski *et al.*, "True Random Number Generators on IQM Spark," arXiv:2512.09862 (2025). — C1〜C3 の回路とノイズのパラメータ（C1〜C3 のノートブック）
- M. S. Turan, E. Barker, J. Kelsey, K. A. McKay, M. L. Baish, and M. Boyle, "Recommendation for the Entropy Sources Used for Random Bit Generation," NIST Special Publication 800-90B (2018). — min-entropy の評価（SP800_90B のノートブック）
- L. E. Bassham III *et al.*, "A Statistical Test Suite for Random and Pseudorandom Number Generators for Cryptographic Applications," NIST Special Publication 800-22 Revision 1a (2010). — 統計的検定（SP800_22 のノートブック）
- X. Ma, F. Xu, H. Xu, X. Tan, B. Qi, and H.-K. Lo, "Postprocessing for quantum random-number generators: Entropy evaluation and randomness extraction," *Phys. Rev. A* **87**, 062327 (2013); arXiv:1207.1473. — Toeplitz-hashing extractor（Toeplitz のノートブック）
- W. Mauerer, C. Portmann, and V. B. Scholz, "A modular framework for randomness extraction based on Trevisan's construction," arXiv:1212.0520 (2012). — Trevisan's extractor（Trevisan のノートブック）
- M. Tomamichel, C. Schaffner, A. Smith, and R. Renner, "Leftover Hashing Against Quantum Side Information," *IEEE Trans. Inf. Theory* **57**, 5524 (2011). — 量子的な side information がある場合の抽出（Toeplitz のノートブック）

公式ツール（このリポジトリには含めていません）：

- NIST Statistical Test Suite（STS）2.1.2：<https://csrc.nist.gov/projects/random-bit-generation/documentation-and-software>
- SP800-90B_EntropyAssessment（`ea_non_iid`）：<https://github.com/usnistgov/SP800-90B_EntropyAssessment>

## 免責事項

- このリポジトリは、独立して作成した教材のノートブック集です。上の論文の著者、NIST、IQM とは関係がなく、それらの承認や確認を受けたものではありません。
- 回路は論文の記述をもとに実装し直したもので、結果はすべて Qiskit Aer のシミュレータで得たものです（実機ではありません）。ノイズモデルはこのノートブックで選んだもので、論文の著者のモデルではないので、結果は論文の実験結果を再現するものではありません。
- シミュレータの出力は、古典コンピュータの擬似乱数から作られます。このノートブックで得たビット列は真の乱数ではなく、暗号やその他のセキュリティの用途に使ってはいけません。
- NIST の公式ツールは、このリポジトリに含めていません。上の配布元から取得してください。ツールにはそれぞれの利用条件があります。
- このリポジトリのコードと文章は MIT License（`LICENSE` を参照）で提供し、いかなる保証もしません。
