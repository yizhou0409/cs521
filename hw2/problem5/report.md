# Problem 5(a): Certifier

Code: [dual_lower.cf](https://github.com/yizhou0409/cs521/blob/main/hw2/problem5/dual_lower.cf).

## Shape

```text
def Shape as
  (Float l, Float u, PolyExp L, PolyExp U, PolyExp La){
  [(curr[l] <= curr), (curr[u] >= curr),
   (curr[L] <= curr), (curr[U] >= curr),
   (curr[La] <= curr)]
};
```

The new member $L_a$ stores a second symbolic lower bound. For an unstable
ReLU, the certifier retains both $0$ and $x$, while DeepPoly selects only one.
Keeping both preserves more information and can tighten later affine bounds.

## Affine transformer

For $e=\sum_i w_i n_i+b$, backsubstitute once using $L$ and once using $L_a$.
Let the resulting concrete bounds be $(l_1,u_1)$ and $(l_2,u_2)$.
The transformer returns

$$
(\max(l_1,l_2),\;\min(u_1,u_2),\;e,\;e,\;e).
$$

Thus, the concrete bounds use the tighter result, and the symbolic affine
bounds remain exact. Both traversals use the starter's coefficient-sign rules.

## ReLU transformer

The original four outputs are unchanged. The fifth output is $L_a$: for an
unstable ReLU, it is $0$ when the original lower plane is $x$, and $x$ when
the original lower plane is $0$. For stable ReLUs, it is the exact relation
($x$ if active, $0$ if inactive). Both lower planes are sound because
$\max(0,x)\ge0$ and $\max(0,x)\ge x$. The analysis flow is unchanged.

## (b) Soundness check

```text
$ constraintflow provesound dual_lower.cf
Proved Affine
Proved Relu
Provesound result: None
```

Both transformer checks passed.


---

# Problem 5(c): Measured comparison

I used the unchanged supplied network and the first 100 MNIST test images.
Clean accuracy on this subset was **99%**. Verified accuracy is the percentage
of these 100 images passing the starter's nonnegative logit-margin test.

| $\epsilon$ | DeepPoly verified | DeepPoly s/image | Dual lower verified | Dual lower s/image |
| --- | ---: | ---: | ---: | ---: |
| 0.005 | 99% | 0.1405 | 99% | 0.2463 |
| 0.01 | 94% | 0.1382 | 94% | 0.2471 |
| 0.02 | 83% | 0.1370 | 83% | 0.2492 |
| 0.03 | 49% | 0.1369 | 49% | 0.2503 |
| 0.05 | 2% | 0.1413 | 2% | 0.2562 |

Times are seconds per image on an arm64 macOS CPU with one intra-op thread.
Both runs used the same environment and compiler settings. Timing excludes
compilation and initial dataset loading.

Both certifiers have the same verified percentages. A stronger local shape
need not certify more images: all output-margin lower bounds must still be
nonnegative. The new certifier tries only two complete lower-plane policies
and retains DeepPoly's upper relaxation. It took about **0.25 s/image**, versus
**0.14 s/image** for DeepPoly, without improving verified accuracy on this sample.

[Full code changes](https://github.com/yizhou0409/cs521/blob/main/hw2/problem5/CHANGES.md)
and [experimental outputs](https://github.com/yizhou0409/cs521/tree/main/hw2/problem5/results).
