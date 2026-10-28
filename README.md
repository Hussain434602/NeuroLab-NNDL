# NeuroLab

A small neural-network library written in **pure NumPy** (no autograd), a numerical gradient check that proves the
backward passes are correct, a PyTorch rebuild for comparison, and a web **playground** where you can watch different
activations and optimizers learn in real time.

Course project: Neural Networks and Deep Learning. Dataset: UCI Energy Efficiency (id 242).

## What is inside

| Requirement | Where |
|---|---|
| Dense layer, activations (sigmoid, tanh, ReLU, Leaky ReLU, softmax), losses (MSE, cross-entropy) with hand-written forward/backward | `neurolab/layers.py`, `activations.py`, `losses.py` |
| SGD, SGD + momentum, RMSprop, Adam by hand | `neurolab/optimizers.py` |
| Numerical gradient check, relative error < 1e-5 for every layer | `neurolab/gradcheck.py`, `experiments/gradient_check.py`, `tests/test_gradients.py` |
| Multi-output regressor on Energy Efficiency + PyTorch rebuild | `neurolab/energy.py`, `neurolab/torch_baseline.py`, `experiments/train_energy.py` |
| Optimizer paths on Rosenbrock and Beale | `experiments/optimizer_surfaces.py` |
| Written convergence comparison of the four optimizers | `experiments/compare_optimizers.py` -> `results/convergence_comparison.md` |
| Web app: dataset / layers / neurons / activation / optimizer / lr, live loss, decision boundary or predicted-vs-actual, two optimizers side by side from the same initial weights | `app/` (FastAPI + plain JS) |
| Page on biological vs artificial neurons with our own diagram | `app/static/neurons.html` (route `/neurons`) |
| Classification demos (decision boundaries, four optimizers) and own neuron diagrams | `experiments/classification_demos.py`, `experiments/neuron_figures.py` |
| Project report (Word) | `docs/NeuroLab_Report.docx` (yellow fields = fill with your real-data results) |
| Prediction logging | `logs/predictions.jsonl` (written by `/api/predict`) |

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 1. Get the real dataset (important)

The code looks for the data in this order:

1. `data/ENB2012_data.xlsx` (or `.csv`), downloaded from <https://archive.ics.uci.edu/dataset/242/energy+efficiency>
2. `ucimlrepo` (`fetch_ucirepo(id=242)`), needs internet
3. A **synthetic stand-in**, used only if both fail so the code can be tested offline. It prints a loud warning, and every
   result file records `dataset_source`. **Never report numbers produced on the stand-in.**

### 2. Run everything

```bash
python -m experiments.gradient_check        # gradient check table -> results/gradient_check.json
python -m experiments.train_energy          # NumPy vs PyTorch, plots, saves models/energy_numpy.npz
python -m experiments.optimizer_surfaces    # Rosenbrock + Beale paths
python -m experiments.compare_optimizers    # convergence table -> results/convergence_comparison.md
python -m experiments.classification_demos  # decision boundaries + optimizer curves on moons/circles/spiral
python -m experiments.neuron_figures        # biological / artificial neuron diagrams
pytest                                      # unit tests
```

Check that `results/energy_results.json` shows `"dataset_source": "ucimlrepo"` or `"local-file"` and that the heating-load
test R2 is at least 0.95 (the script prints PASS or FAIL).

### 3. Start the web app

```bash
uvicorn app.main:app --port 8000
# open http://localhost:8000
```

Tabs: **Playground** (train live, optionally compare Run A vs Run B), **Energy predictor** (uses the saved NumPy model),
**Gradient check** (runs the check in the browser), and a separate **Biological vs artificial neuron** page.
If `models/energy_numpy.npz` does not exist yet, the server trains it on the first prediction request.

## API

| Method and path | Purpose |
|---|---|
| `POST /api/train/stream` | Streams newline-delimited JSON: one `init`, many `epoch`, then `done` |
| `GET /api/energy/info` | Model metadata and test metrics |
| `POST /api/predict` | `{"features": [8 numbers]}` -> heating and cooling load (logged) |
| `GET /api/gradcheck` | Runs the numerical gradient check |
| `GET /api/health` | Liveness check |

## Deployment

Hugging Face Spaces: create a Space with the **Docker** SDK, push this repo (the `Dockerfile` listens on port 7860).
Put `data/ENB2012_data.xlsx` and `models/energy_numpy.npz` in the repo first so the Space does not need internet.
Render: a Web Service using the Dockerfile works the same way. If you cannot deploy, run locally and record a demo video.

## Design decisions worth knowing for the viva

- **Split**: 70/15/15 train/validation/test with seed 42. Scalers are fitted on the training part only. The best epoch is chosen on
  validation, and the test set is evaluated once at the end of each model.
- **Fair optimizer comparison**: every optimizer starts from cloned initial weights and sees the same mini-batch order.
- **PyTorch check**: with the NumPy initial weights copied over, the same batch order and float64, the PyTorch curve should track the
  NumPy one closely. A second PyTorch run with default initialisation shows the result does not depend on that.
- **Loss scale**: losses are MSE on standardised targets. R2 and RMSE are reported in original kWh/m2 units.

## Team roles

- **Member A (model lead)**: `neurolab/`, `experiments/`, ablations, results tables.
- **Member B (application lead)**: `app/`, deployment, prediction logging, demo video.
- Both: report (`docs/report_outline.md`) and viva (`docs/viva_notes.md`). Both members should commit to this repository.

## Layout

```
neurolab/       library: activations, layers, losses, optimizers, network, gradcheck, data, energy, playground, torch_baseline
experiments/    scripts that produce every table and figure
app/            FastAPI server + static front end
tests/          pytest suite
results/        generated figures and tables
docs/           report outline, viva notes
```
