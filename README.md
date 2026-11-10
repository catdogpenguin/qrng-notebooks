English | [日本語](README.ja.md)

# Quantum random number generation (QRNG) textbook notebooks

A collection of notebooks that simulate a quantum random number generator (QRNG) with Qiskit and go on to assess the entropy of the resulting random numbers, extract uniform randomness, and run statistical tests.

## Notebooks

The notebooks are meant to be read in the order below. The text of each notebook is in English, and the original Japanese text is kept in HTML comments (`<!-- ... -->`) in each Markdown cell. To read the Japanese, put the cell in edit mode.

| Order | Notebook | Contents | Official tool required |
|---|---|---|---|
| 1 | `C1_circuit_textbook.ipynb` | Generate random numbers with the 1-qubit C1 circuit and examine their bias and entropy | None |
| 2 | `C2_circuit_textbook.ipynb` | The C2 circuit, which applies the C1 circuit in parallel to 5 qubits | None |
| 3 | `C3_circuit_textbook.ipynb` | The C3 circuit, which reduces a 5-qubit GHZ state to 1 bit by majority vote | None |
| 4 | `SP800_90B_textbook.ipynb` | Estimate the min-entropy of raw data with NIST SP 800-90B | `ea_non_iid` |
| 5 | `Toeplitz_extractor_textbook.ipynb` | Extract uniform random numbers with the Toeplitz-hashing extractor | None |
| 6 | `Trevisan_extractor_textbook.ipynb` | Extract uniform random numbers with Trevisan's extractor | None |
| 7 | `SP800_22_textbook.ipynb` | Run the 15 statistical tests of NIST SP 800-22 | STS 2.1.2 |
| 8 | `QRNG_pipeline.ipynb` | Define the circuit and noise, then run generation (simulation or a raw-data file) → SP 800-90B → Toeplitz / Trevisan → SP 800-22 end to end | `ea_non_iid`, STS |

`QRNG_pipeline.ipynb` loads only the function definitions from notebooks 4–7 (SP800_90B, Toeplitz, Trevisan, SP800_22), so keep it in the same folder as them.

The official tools are distributed by NIST and are not included in this repository. Download and build them by following the setup steps below. If a tool is missing, the SP800_90B and SP800_22 notebooks skip the cells that use it and print a message saying so.

## Setup

Run all of the commands below in an Ubuntu terminal (including Ubuntu on WSL).

> **Where to put the repository**: STS can only read file paths of up to 199 characters, and cannot read paths that contain whitespace. Put the repository at a short path without whitespace (e.g., `~/qrng-notebooks`). On WSL, file access is faster in your Linux home folder than under `/mnt/c/...` (the Windows side).

### 1. Install system packages

```bash
sudo apt update
sudo apt install -y git curl unzip build-essential python3.12-venv \
  libbz2-dev libdivsufsort-dev libjsoncpp-dev libssl-dev libgmp-dev libmpfr-dev
```

`build-essential` is used to build both official tools, and the `lib...-dev` packages are used to build `ea_non_iid`.

### 2. Get the repository

```bash
git clone https://github.com/catdogpenguin/qrng-notebooks.git
cd qrng-notebooks
```

Run the remaining commands in this folder (the top of the repository).

### 3. Create the Python virtual environment

```bash
python3.12 -m venv qiskit-env
source qiskit-env/bin/activate
pip install -r requirements.txt
python -c "import qiskit, qiskit_aer; print(qiskit.__version__, qiskit_aer.__version__)"   # success if it prints 2.5.2 0.17.2
```

`requirements.txt` was produced with `pip freeze` in a Python 3.12 environment on Linux, and pins the version of every package. On other operating systems or Python versions, some packages may fail to install.

### 4. Build STS (the official tool for SP 800-22)

Distribution page: <https://csrc.nist.gov/projects/random-bit-generation/documentation-and-software> (`sts-2_1_2.zip` under "NIST SP 800-22 Rev. 1a")

```bash
curl -L -o sts-2_1_2.zip https://csrc.nist.gov/CSRC/media/Projects/Random-Bit-Generation/documents/sts-2_1_2.zip
echo "0238d2f1d26e120e3cc748ed2d4c674cdc636de37fc4027c76cc2a394fff9157  sts-2_1_2.zip" | sha256sum -c -   # OK means it is the version the notebook expects
unzip -q sts-2_1_2.zip -d .
cd sts-2.1.2/sts-2.1.2      # the zip contains a doubly nested folder
make                        # builds assess (a few warnings are printed; they are harmless)
./assess                    # success if it prints "Usage: ./assess <stream length>"
cd ../..
```

If `curl` cannot download the file, download it from the distribution page with a browser. On WSL, a zip in the Windows "Downloads" folder is at `/mnt/c/Users/<user name>/Downloads/sts-2_1_2.zip`. If `sha256sum -c` prints `FAILED`, the file is not the version the notebook expects.

The SP800_22 notebook looks for STS at the path `sts-2.1.2/sts-2.1.2`, relative to the top of the repository (variable `STS_DIR`).

### 5. Build `ea_non_iid` (the official tool for SP 800-90B)

Repository: <https://github.com/usnistgov/SP800-90B_EntropyAssessment>

```bash
git clone https://github.com/usnistgov/SP800-90B_EntropyAssessment.git
cd SP800-90B_EntropyAssessment
git checkout 87c104d0ed4cbc96103e7b8b38d6f2c7e0a6b289   # the version the notebook expects; omit this line to use the latest version
cd cpp
make non_iid                # builds ea_non_iid
./ea_non_iid                # success if it prints its usage
cd ../..
```

If the build fails because a header file (`json/json.h`, `divsufsort.h`, `bzlib.h`, `gmp.h`, `mpfr.h`, `openssl/...`) cannot be found, check that the corresponding package from the `apt install` in step 1 is installed.

The SP800_90B notebook looks for the tool at the path `SP800-90B_EntropyAssessment/cpp/ea_non_iid`, relative to the top of the repository (variable `EA_NON_IID`).

### 6. Open the notebooks

**With VS Code**: Install the WSL and Jupyter extensions and open this folder inside WSL (run `code .` in the terminal). After opening a notebook, choose `qiskit-env/bin/python` with "Select Kernel" at the top right.

**With JupyterLab**: JupyterLab itself is not included in `requirements.txt`, so install it in the virtual environment.

```bash
source qiskit-env/bin/activate
pip install jupyterlab
jupyter lab
```

In either case, run the notebooks **with the folder that contains them (the top of the repository) as the working directory**. The locations of the official tools and of the output files are given as paths relative to it.

## Folder layout

After setup, running the notebooks gives the following layout. `qiskit-env/`, the official tools, and `results/` are excluded from git by `.gitignore`.

```
.
├── README.md                        README (English)
├── README.ja.md                     README (Japanese)
├── requirements.txt
├── *.ipynb                          notebooks
├── qiskit-env/                      Python virtual environment (3.)
├── sts-2.1.2/sts-2.1.2/             STS (4.)
├── SP800-90B_EntropyAssessment/     ea_non_iid (5.)
└── results/                         output of the notebooks (only when SAVE_RESULTS = True)
    ├── c1/2026-11-10_153012/        C1 raw data and results (folder named after the time the setup cell was run)
    ├── c2/<date_time>/              C2
    ├── c3/<date_time>/              C3
    ├── sp800_90b/<date_time>/       SP800_90B
    ├── toeplitz/<date_time>/        Toeplitz
    ├── trevisan/<date_time>/        Trevisan
    ├── pipeline/<date_time>/        QRNG_pipeline
    └── sp800_22/<date_time>/        SP800_22
```

Every notebook saves its files when `SAVE_RESULTS = True` in its setup cell (1. for C1–C3, Toeplitz, and Trevisan; 2. for SP800_90B and SP800_22). Section 5.9 of C1–C3 (the experiment that changes the noise strength step by step) produces a lot of data, so it is saved only when a separate switch, `SAVE_SWEEP = True`, is set. Both default to `False`, which saves nothing. When saving, a new folder named after the date and time the setup cell was run is created, so earlier results are never overwritten. The contents of each date-and-time folder (per-experiment folders, file naming, CSV columns, and so on) are described in "Saved files" under "How to use this notebook" in each notebook.
