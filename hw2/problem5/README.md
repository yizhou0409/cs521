# Problem 5(a)–(c): dual-lower-plane certifier

`dual_lower.cf` extends the [starter DeepPoly certifier](https://github.com/nalinwadhwa02/CCS521FA26HW/tree/main/hw2/q5) with a second polyhedral lower bound. An unstable ReLU retains both `0` and `x`. Affine backsubstitution tries the original and complementary policies, taking the tighter concrete bounds. `CHANGES.md` and `changes.patch` list every code change.

The supplied network and its weights are unchanged. Part (d) is not included.

## Setup

Follow the starter's pinned source installation (Python 3.10 or newer):

```bash
python -m venv cfenv
source cfenv/bin/activate
git clone https://github.com/uiuc-focal-lab/constraintflow.git
cd constraintflow
git checkout 881e2703aa0a1fcf8022c8664632a16f9c0bfbd5
pip install 'setuptools<81' -e .
cd ..
```

Do not install the published PyPI package instead of the editable source checkout. The exact environment used for the recorded results is in `results/environment.json`. Locally, an isolated venv inherited the existing PyTorch 2.1.0 / torchvision 0.15.2a0 / NumPy 1.23.5 installation. Missing graphviz 0.20.1, networkx 3.1, onnx 1.16.2, z3-solver 4.12.2.0, antlr4-python3-runtime 4.9.2, and typer 0.12.5 were installed with `--no-deps`; the pinned ConstraintFlow checkout was also installed editable with `--no-deps`. This preserved the working PyTorch/NumPy stack.

## Soundness and benchmark

From this directory, with the environment active:

```bash
constraintflow provesound starter/deeppoly.cf
constraintflow provesound dual_lower.cf

CF_NO_PARALLEL=1 OMP_NUM_THREADS=1 MKL_NUM_THREADS=1 python -u run_cf.py \
  starter/deeppoly.cf --network starter/mnist_relu_3_100.onnx \
  --num-images 100 --eps 0.005 0.01 0.02 0.03 0.05

CF_NO_PARALLEL=1 OMP_NUM_THREADS=1 MKL_NUM_THREADS=1 python -u run_cf.py \
  dual_lower.cf --init La=L --network starter/mnist_relu_3_100.onnx \
  --num-images 100 --eps 0.005 0.01 0.02 0.03 0.05
```

Run the two commands sequentially in separate processes. `CF_NO_PARALLEL=1` disables the pinned compiler's loop-parallelization pass, which emitted invalid variable scopes for the dual-traversal certifier. This uses a built-in compiler option; no compiler source was changed. MNIST downloads automatically to `data/`; `--data PATH` can reuse an existing download. Both benchmarks use CPU, one PyTorch intra-op thread, the first 100 test images, and the starter's nonnegative lower-margin criterion. Runtime is total certification-loop wall time divided by 100, excluding compilation and the initial dataset load. No extra clean-correct filter was added.

`run_cf.py` is the starter runner with one import-cache fix; see `CHANGES.md`. The untouched runner is retained under `starter/`. Compiled Python output is generated under `build_deeppoly/` and `build_dual_lower/` and is not needed in the repository.

## Evidence

- `results/provesound_dual_lower.log`: actual soundness output.
- `results/deeppoly.log`, `results/dual_lower.log`: actual benchmark output.
- `results/comparison.csv`, `results/comparison.json`: table parsed from those outputs.
- `report.md`: high-level explanation and measured comparison, also included in the homework Markdown.

Repository: https://github.com/yizhou0409/cs521 (private, as requested). The assignment asks for public code, so grader access must be arranged before submission.
