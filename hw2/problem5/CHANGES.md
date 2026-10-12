# Changes from the Problem 5 starter

Source: https://github.com/nalinwadhwa02/CCS521FA26HW/tree/main/hw2/q5

## Certifier: `starter/deeppoly.cf` → `dual_lower.cf`

1. Added one `PolyExp` member, `La`, to `Shape`, with the constraint `curr[La] <= curr`. The original four members and their constraints remain.
2. Added `replace_lower_alt`: positive coefficients use `La`; negative coefficients use `U`.
3. Added `replace_upper_alt`: positive coefficients use `U`; negative coefficients use `La`.
4. Added `backsubs_lower_alt` and `backsubs_upper_alt`. These traverse the same graph with the same priorities, stopping conditions, and concrete simplifications as the original helpers, using the alternative replacement functions.
5. Added `relu_lower_alt`. Stable active ReLUs use `x`, stable inactive ReLUs use `0`, and unstable ReLUs use the lower plane discarded by the original area heuristic. Thus unstable ReLUs retain both `0` and `x`.
6. Changed the Affine transformer to take the maximum of the two lower-bound traversals and the minimum of the two upper-bound traversals. All three symbolic affine bounds are the exact affine expression.
7. Extended the ReLU transformer with its fifth result, `relu_lower_alt(prev)`. The original four results are unchanged.
8. Renamed the transformer and its flow reference from `deeppoly` to `dual_lower`. The flow direction, scheduling, and stopping conditions are unchanged.

No original helper was modified. Input initialization uses the starter's existing `--init La=L` option. The implementation tries two complete backsubstitution policies; it does not optimize every combination of per-neuron lower planes.

## Runner: `starter/run_cf.py` → `run_cf.py`

Exactly one line was added immediately before importing the compiled `main.run`:

```python
sys.modules.pop("transformers", None)
```

The local torchvision import had already loaded Hugging Face's `transformers` package, shadowing ConstraintFlow's generated `transformers.py`. This line lets Python import the generated module from the build directory already placed first on `sys.path`. It does not change the benchmark's inputs, certificate condition, denominator, or timing loop.

## Unchanged starter assets

`starter/deeppoly.cf`, `starter/run_cf.py`, `starter/README.md`, and `starter/mnist_relu_3_100.onnx` are unedited downloaded copies. The supplied 784–100–100–100–10 network, weights, ReLU activations, and MNIST normalization are unchanged. No training was performed.

## Other files

- `README.md`: setup and commands for parts (a)–(c).
- `changes.patch`: complete textual diff of the certifier and runner against their starter versions.
- `results/`: actual soundness outputs, benchmark outputs, environment information, starter-file integrity hashes, and a table parsed from the benchmark logs.
- `../output/hw2.md`: added the Problem 5(a)–(c) explanation, soundness output, measured comparison, and a concise change summary.
- `../scripts/integrate_problem4_results.py`: preserve the appended Problem 5 section when regenerating the Problem 4 report.

ConstraintFlow itself was not patched. The local benchmark used its pinned editable source at commit `881e2703aa0a1fcf8022c8664632a16f9c0bfbd5`, an isolated Python 3.10 environment inheriting the existing PyTorch/NumPy stack, and separately installed missing dependencies with `--no-deps` to preserve that stack. The built-in `CF_NO_PARALLEL=1` option disables a compiler parallelization pass that emitted invalid Python variable scopes for the new certifier. Both benchmarks use this same setting and CPU with `OMP_NUM_THREADS=1` and `MKL_NUM_THREADS=1`, in separate sequential processes. Generated `build_*` directories contain compiler output, not handwritten certifier code.

Part (d) is not implemented in this step. Code is hosted in the user-requested private repository https://github.com/yizhou0409/cs521; grader access must be arranged because the assignment requests public code.
