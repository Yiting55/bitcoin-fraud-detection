Place the extracted, authorized competition files in this `data/` directory,
relative to the repository root:

```text
data/train.csv
data/test.csv
data/txs_edgelist.csv
data/sample_submission.csv
```

Run commands and launch the notebook from the repository root. The notebook and
main Python scripts use this directory by default; no absolute dataset path is
required. Raw data is excluded from Git by the root `.gitignore`.

Download from the Kaggle competition `bt-4012-competition-2026`, after accepting
its rules. `download_data.py` uses kagglehub and prints the cache location;
copy the four files from that location into this directory. Install kagglehub
separately if using that optional helper. Do not commit Kaggle credentials.
