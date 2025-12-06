# Hybrid Recommender System — Spotify

Professional, production-oriented repository for a weighted hybrid recommender system built for Spotify-style music recommendations.

---

**Table of contents**
- [Project overview](#project-overview)
- [Project structure](#project-structure)
- [Pipeline diagram](#pipeline-diagram)
- [Installation](#installation)
- [Configuration](#configuration)
- [Run locally](#run-locally)
- [Docker](#docker)
- [DVC & Data management](#dvc--data-management)
- [Model information](#model-information)
- [Evaluation](#evaluation)
- [Development & Contributing](#development--contributing)
- [License](#license)

---

## Project overview

This repository implements a hybrid recommender system that combines content-based and collaborative filtering approaches to generate personalized track recommendations. The codebase includes data preprocessing, feature extraction, model training, evaluation, and a Streamlit demo app to interact with the system.

Key goals:
- Demonstrate a reproducible data pipeline using DVC
- Keep large binaries in Git LFS
- Provide an easy-to-run demo (Streamlit + Docker)

## Project structure

Top-level layout (important files/folders):

- `data/` — raw, intermediate and processed data artifacts (DVC-managed). Key files: `Music Info.csv`, `User Listening History.csv`.
- `docs/` — project documentation and Sphinx assets.
- `notebooks/` — EDA and experimentation notebooks.
- `src/` — reusable code, feature builders, model training and prediction helpers.
- `models/` — saved model artifacts (joblib, pickle) and reports.
- `app.py` — Streamlit demo that exposes recommendation endpoints.
- `dvc.yaml` & `dvc.lock` — DVC pipeline describing reproducible stages.
- `Dockerfile` — image definition for running the demo in a container.
- `requirements.txt` — pinned Python dependencies.

Example structure (short):

```
.
 app.py
 data/
   raw/
     Music Info.csv
     User Listening History.csv
   cleaned_data.csv        # produced by `data_cleaning.py` (DVC output)
   collab_filtered_data.csv
   transformed_*.npz
 Dockerfile
 dvc.yaml
 notebooks/
 src/
   data/
   features/
   models/
 requirements.txt
```

## Pipeline diagram

High-level pipeline (DVC stages):

```
raw data (data/raw/*.csv)
    |
    v
data_cleaning (python data_cleaning.py)
    -> data/cleaned_data.csv
    |
    v
content_based (python content_based_filtering.py)
    -> data/transformed_data.npz, transformer.joblib
    |
    v
collaborative (python collaborative_filtering.py)
    -> data/collab_filtered_data.csv, data/track_ids.npy, data/interaction_matrix.npz
    |
    v
transform_filtered_data (python transform_filtered_data.py)
    -> data/transformed_hybrid_data.npz
    |
    v
app (Streamlit) / model training & evaluation
```

You can visualize the pipeline with `dvc dag` when DVC is installed.

## Installation

Prerequisites
- Python 3.10+ (3.12 recommended)
- Docker (optional, for containerized demo)
- Git LFS (for large data files)
- DVC (for pipeline reproducibility)

1. Clone the repository and enter it:

```bash
git clone https://github.com/AmitZala/hybrid-recommender-system-spotify.git
cd hybrid-recommender-system-spotify
```

2. Install system tools (Windows example):

```powershell
# Install Chocolatey, then:
choco install git-lfs -y
choco install dvc -y
# or follow OS-specific instructions from https://dvc.org and https://git-lfs.github.com
```

3. Set up Python environment and install dependencies:

```bash
python -m venv env
# Windows PowerShell
env\\Scripts\\Activate.ps1
pip install -r requirements.txt
```

4. Fetch large files (Git LFS) and DVC data

```bash
git lfs install
git lfs pull
# If the project uses DVC remote storage (optional):
dvc pull
```

## Configuration

Key configuration variables are set via environment variables or by editing the script constants:

- `DATA_PATH` — path to `Music Info.csv` (default `data/Music Info.csv`)
- `STREAMLIT_SERVER_PORT` — port for the demo (default 8501)

For production or advanced runs, provide environment variables when launching Docker or the app.

## Running locally (development)

1. Make sure dependencies are installed and data is available (see Installation).
2. Reproduce pipeline and generate artifacts:

```bash
# run the DVC pipeline (creates cleaned data & transformer artifacts)
dvc repro
```

3. Start the Streamlit demo locally:

```bash
# from repo root, with virtualenv activated
streamlit run app.py --server.port=8501 --server.address=0.0.0.0 --server.headless=true
```

Open `http://localhost:8501` in your browser.

## Docker (recommended for demo)

Build the image (tested on Windows with Docker Desktop):

```powershell
# from repo root
docker build -t amitzala93/hybrid-recommender-system-spotify:latest .
```

Run the container exposing the demo port:

```powershell
docker run -d -p 8501:8501 --name hybrid-run \\
  -e STREAMLIT_BROWSER_GATHER_USAGE_STATS=false \\
  amitzala93/hybrid-recommender-system-spotify:latest \\
  streamlit run app.py --server.port=8501 --server.address=0.0.0.0 --server.headless=true
```

Then open `http://localhost:8501`.

Notes about `0.0.0.0`:
- `0.0.0.0` is a listening address used inside the container. In your browser use `http://localhost:8501` or `http://127.0.0.1:8501`.

## DVC & Data management

- `dvc.yaml` defines stages and outputs. Use `dvc repro` to run stages.
- Large binary data is stored with Git LFS. Use `git lfs pull` to download tracked files.
- Use `dvc push` to send DVC-tracked outputs to remote storage (S3, GDrive, etc.) if configured.

## Model information

This project contains a hybrid approach combining content-based and collaborative filtering:

- Content-based:
  - Text + audio feature extraction from `Music Info.csv` (tags, metadata, audio features)
  - Vectorization and transformation stored in `transformer.joblib` and `data/transformed_data.npz`

- Collaborative filtering:
  - Interaction matrix computed from `User Listening History.csv`
  - Matrix factorization / neighborhood or implicit feedback technique used to compute latent factors
  - Outputs: `data/interaction_matrix.npz`, `data/track_ids.npy`, `data/collab_filtered_data.csv`

- Hybrid recommendation:
  - Combine scores from content model + collaborative model using a weighted scheme
  - Final transformed hybrid features saved in `data/transformed_hybrid_data.npz`

Model training and inference scripts:
- `collaborative_filtering.py` — builds the interaction matrix and trains the collaborative model
- `content_based_filtering.py` — builds content features and transformer
- `hybrid_recommendations.py` — glue logic that merges both models' outputs into final recommendations

Saved artifacts (examples):
- `transformer.joblib` — feature transformer for content model

Hyperparameters and evaluation details can be found in the notebooks under `notebooks/`.

## Evaluation

Typical metrics and charts to evaluate:
- Precision@K, Recall@K
- MAP (mean average precision)
- NDCG@K
- Offline A/B tests and user engagement tracking for production evaluation

Check `notebooks/` for detailed EDA and evaluation notebooks.

## Development & contributing

- Use `make` or the tasks in `Makefile` to run common operations (see `Makefile`).
- Follow the repository coding style. Open a PR for changes and include unit tests where applicable.

## Troubleshooting

- If the app prints `URL: http://0.0.0.0:8501` in logs, do not open `0.0.0.0` in browser — use `http://localhost:8501` or `http://<host-ip>:8501`.
- If you see large files in Git, ensure you ran `git lfs pull` and `dvc pull` if a DVC remote is configured.

## License

This project uses the `LICENSE` file at the repository root.

---
