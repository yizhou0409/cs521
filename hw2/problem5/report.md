# Problem 5(a): A dual-lower-plane certifier

I extend the starter DeepPoly shape with a second symbolic lower bound, $L_a$.
The implementation is [dual_lower.cf](https://github.com/yizhou0409/cs521/blob/main/hw2/problem5/dual_lower.cf).
The repository is private; grader access is needed before submission.

## Shape declaration

```text
def Shape as
  (Float l, Float u, PolyExp L, PolyExp U, PolyExp La){
  [(curr[l] <= curr), (curr[u] >= curr),
   (curr[L] <= curr), (curr[U] >= curr),
   (curr[La] <= curr)]
};
```

The shape represents $l\le n\le u$, $L\le n\le U$, and $L_a\le n$.
At an unstable ReLU, DeepPoly retains either $0$ or $x$ as its symbolic lower
bound. My shape retains both, so its concretization is the intersection with
an additional valid half-space. This can tighten subsequent affine bounds.
For example, on $[-2,1]$, DeepPoly's $y\ge0$ admits $(x,y)=(1,0)$;
adding $y\ge x$ excludes this spurious pair.
At the input, the starter's `--init La=L` initializes the extra lower bound.

## Affine transformer

Let $e=\sum_i w_i n_i+b$. Write $B_L(e),B_U(e)$ for the starter's complete
backsubstitution to the input box, and $B_L^a(e),B_U^a(e)$ for the same
traversal using $L_a$ wherever the original traversal uses $L$.
The Affine transformer returns, in shape-member order,

$$
\left(\max\{B_L(e),B_L^a(e)\},\;
\min\{B_U(e),B_U^a(e)\},\;e,\;e,\;e\right).
$$

For a lower-bound traversal, a nonnegative coefficient substitutes the selected
lower plane and a negative coefficient substitutes $U$. For an upper-bound
traversal, a nonnegative coefficient substitutes $U$ and a negative coefficient
substitutes the selected lower plane. At the input, expressions are evaluated
with the concrete box endpoints according to coefficient sign.

Both traversals are sound; their maximum lower bound and minimum upper bound
remain sound. This tries two complete policies, rather than all combinations
of per-neuron choices. The affine symbolic expressions remain exact.

---

# Problem 5(a)–(b): ReLU and soundness

## ReLU transformer

For a predecessor $x$ with concrete bounds $[l,u]$, keep the starter's
$L(x)=x$ when $l+u\ge0$, and $L(x)=0$ otherwise. The added lower plane is

$$
L_a(x)=\begin{cases}
x,&l\ge0,\\
0,&l<0\text{ and }u\le0,\\
0,&l<0<u\text{ and }l+u\ge0,\\
x,&l<0<u\text{ and }l+u<0.
\end{cases}
$$

The upper plane is unchanged:

$$
U(x)=\begin{cases}
x,&l\ge0,\\
0,&l<0\text{ and }u\le0,\\
\dfrac{u}{u-l}x-\dfrac{ul}{u-l},&l<0<u.
\end{cases}
$$

The implemented transformer is

```text
Relu -> (relu(prev[l]), relu(prev[u]),
         relu_lower(prev), relu_upper(prev),
         relu_lower_alt(prev));
```

Both $0$ and $x$ are lower bounds of $\max(0,x)$. For unstable ReLUs, the two
stored planes are complementary; stable ReLUs retain the exact relation.
The forward flow, traversal priorities, and stopping conditions are unchanged.

## (b) Actual ConstraintFlow output

Using ConstraintFlow commit `881e2703aa0a1fcf8022c8664632a16f9c0bfbd5`:

```text
$ constraintflow provesound dual_lower.cf
Proved Affine
Proved Relu
Provesound result: None
```

The command exited successfully. `Proved Affine` and `Proved Relu` confirm
both transformer checks; the CLI also prints its return value `None`.
[Soundness log](https://github.com/yizhou0409/cs521/blob/main/hw2/problem5/results/provesound_dual_lower.log).

---

# Problem 5(c): Measured comparison

I used the supplied **784–100–100–100–10** ONNX network without changing its
architecture or weights, and the **first 100 MNIST test images**. Clean accuracy
on this subset was **99%** for both runs. Both certifiers use the starter's
normalization, input-region construction, and nonnegative logit-margin test.
Verified accuracy is the verified count divided by 100.

| $\epsilon$ | DeepPoly verified | DeepPoly s/image | Dual lower verified | Dual lower s/image |
| --- | ---: | ---: | ---: | ---: |
| 0.005 | 99% | 0.1405 | 99% | 0.2463 |
| 0.01 | 94% | 0.1382 | 94% | 0.2471 |
| 0.02 | 83% | 0.1370 | 83% | 0.2492 |
| 0.03 | 49% | 0.1369 | 49% | 0.2503 |
| 0.05 | 2% | 0.1413 | 2% | 0.2562 |

Times are seconds per image on an arm64 macOS CPU, Python 3.10.9 and
PyTorch 2.1.0, with one intra-op thread (`OMP_NUM_THREADS=1`, `MKL_NUM_THREADS=1`).
The two runs were sequential, with `CF_NO_PARALLEL=1` to avoid an invalid
variable scope emitted by the pinned compiler's parallelization pass. Times include the starter's per-image loading
and certification work, and exclude compilation and initial dataset loading.

The verified percentages match at all five radii. Retaining both lower planes
can strengthen the local shape, but does not guarantee additional certified
images: certification still requires all output-margin lower bounds to cross
zero. This implementation tries only two global lower-plane policies, and
keeps the same ReLU upper relaxation; it does not solve for the best mixed
choice at every neuron. Equal verified counts do not imply identical bounds.
The extra traversals cost time: about 0.25 s/image, versus 0.14 s/image for
DeepPoly in these runs, with no verified-accuracy gain on this sample.

---

# Problem 5: Changes from the starter

I added `La` and its lower-bound constraint, two coefficient-sensitive
replacement helpers, two alternative backsubstitution helpers, and the
complementary ReLU lower-plane helper. Affine now combines the two traversals;
ReLU returns the extra bound. I renamed the transformer and its flow reference.
The original four ReLU outputs and all original helper functions are unchanged.
Input initialization uses the existing `--init La=L` option.

The runner has one added line, `sys.modules.pop("transformers", None)`, before
importing generated `main.run`, to resolve a local package-name collision.
The dataset selection, network, verification condition, and timing loop are
unchanged. ConstraintFlow itself was not modified.

[Complete change report](https://github.com/yizhou0409/cs521/blob/main/hw2/problem5/CHANGES.md)
and [benchmark evidence](https://github.com/yizhou0409/cs521/tree/main/hw2/problem5/results).
