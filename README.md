# Supplementary Experiments: Bandit Submodular Maximization

This document describes two synthetic experiments under cardinality and graphic matroid constraints. Both use weighted coverage functions with independently randomized weights whose means drift smoothly between different target optima.

The experiments use 500,000 rounds and 80 paired test seeds. The method labeled **ours** uses cumulative Gaussian Follow-the-Perturbed-Leader (FTPL) to update the fractional exchange policy. The exact practical implementation and parameters are specified below.

## Results and figures

| Constraint | Method | Final cumulative reward ratio | 95% confidence interval |
|---|---|---:|---:|
| Cardinality, n = 20, k = 5 | ours | 0.835282 | [0.825422, 0.845142] |
| Cardinality, n = 20, k = 5 | OGo | 0.850075 | [0.837391, 0.862759] |
| Cardinality, n = 20, k = 5 | ETCG | 0.189231 | [0.189229, 0.189232] |
| Graphic matroid, n = 30, k = 6 | ours | 0.804041 | [0.788267, 0.819816] |
| Graphic matroid, n = 30, k = 6 | ETCG | 0.289758 | [0.289573, 0.289943] |
| Graphic matroid, n = 30, k = 6 | EXP3 | 0.447984 | [0.447904, 0.448064] |

### Cardinality

![Cardinality experiment](figures/cardinality.png)

[Download the vector PDF](figures/cardinality.pdf).

### Graphic matroid

![Graphic matroid experiment](figures/graphic.png)

[Download the vector PDF](figures/graphic.pdf).

At elapsed round t, the plotted quantity is

$$
\rho_t =
\frac{\sum_{s=1}^{t} f_s(S_s)}
{\max_{S\in\mathcal I}\sum_{s=1}^{t} f_s(S)}.
$$

The denominator is the exact reward of the best **fixed** feasible set for that prefix. It is not the sum of per-round optimal rewards. An adaptive policy can therefore achieve a ratio above one. The final table reports the mean of the 80 seed-specific ratios at t = 500,000.

All exploration rounds contribute to the numerator. Shaded curve bands are pointwise 95% Student-t confidence intervals across seeds. Gray vertical bands indicate drift intervals. The horizontal reference lines are 1 and 1 - 1/e. Crossing the latter is an empirical observation, not a finite-horizon guarantee for the tuned implementation.

## Shared objective and random weights

Let C_i be the set of features covered by element i. The objective is

$$
f_t(S)=\frac{1}{Z}\sum_{e\in\bigcup_{i\in S}C_i}w_{t,e}.
$$

Coverage sets remain fixed throughout each experiment. A feature contributes its weight only once, regardless of how many selected elements cover it. All weights are nonnegative, so each objective is monotone submodular.

Each element covers a private feature. At target r, the private-feature means are

$$
\mu_i^{(r)} =
\begin{cases}
1, & i\in B_r,\\
0.005, & i\notin B_r,
\end{cases}
$$

where B_r is a feasible target base. The target bases within each experiment are pairwise disjoint, and each is the unique optimum at its corresponding mean-weight endpoint. Shared-topic means are 0.03; the additional triangle and block features in the graphic experiment have mean 0.01.

During a transition from target r to target r+1, private-feature means interpolate as

$$
\mu_{t,i}=(1-\lambda(u))\mu_i^{(r)}
+\lambda(u)\mu_i^{(r+1)},\qquad
\lambda(u)=\frac{1-\cos(\pi u)}{2},
$$

where u = (t - start)/(end - start). Shared-feature means do not drift.

Every feature on **every round**, including all fixed-mean phases, independently receives a clipped Gaussian perturbation:

$$
w_{t,e}=\mu_{t,e}
\left[1+0.02\,\operatorname{clip}(G_{t,e},-3,3)\right],
\qquad G_{t,e}\sim\mathcal N(0,1).
$$

Thus, a fixed phase means a fixed mean vector, not fixed realized weights. Before clipping, the perturbation standard deviation is 2% of the feature mean; clipping bounds the fluctuation by +/-6%. There is no additional observation noise: a learner observes the exact value of its selected set, and only one function-value query is allowed per round.

The normalization is fixed over time:

$$
Z=1.00001\times1.06\sum_e\mu_{t,e}.
$$

The total feature mean is constant within each experiment, so this definition gives a single constant Z. It ensures f_t(U) < 1 for the entire ground set, not merely for feasible sets. Feedback is not amplified.

Intervals below use zero-based round indices and exclude the right endpoint. Element and feature indices are also zero-based.

## Experiment 1: Cardinality constraint

### Feasible sets and coverage

- Ground set size n = 20 and cardinality limit k = 5.
- Every subset of size at most five is feasible; there are 15,504 full-rank subsets.
- Elements are assigned to five coverage groups of four: group g contains elements 4g through 4g+3.
- Each element covers its private feature and its group's shared topic, giving 25 features.
- These groups define coverage overlap; they impose no additional feasibility quotas.
- The normalization constant is Z = 5.538555385.

The three target bases are:

| Target | Elements |
|---|---|
| B0 | 3, 5, 9, 14, 17 |
| B1 | 0, 6, 8, 15, 19 |
| B2 | 2, 4, 10, 13, 18 |

### Two smooth objective changes

| Rounds | Mean-weight setting |
|---|---|
| [0, 40,000) | Hold target B0 |
| [40,000, 80,000) | Cosine transition B0 to B1 |
| [80,000, 140,000) | Hold target B1 |
| [140,000, 180,000) | Cosine transition B1 to B2 |
| [180,000, 500,000) | Hold target B2 |

### Compared methods

| Figure label | Implementation and selected parameters |
|---|---|
| ours | Cumulative Gaussian FTPL; exploration probability gamma = 0.1, perturbation scale sigma = 1, 14 perturbed bases per update |
| OGo | Ordered greedy exponential-weight experts with prefix exploration; gamma = 0.1 and reward-update coefficient eta = 3 |
| ETCG | Explore-Then-Commit Greedy; paper-prescribed m = 182 consecutive samples per candidate; no restart |

## Experiment 2: Graphic matroid

### Graph, feasible sets, and coverage

The graph has seven vertices and 30 physical edges. Two K4 blocks share a single articulation vertex. The first block uses vertices (0, 1, 2, 3); the second uses (0, 4, 5, 6).

Within each block, the six underlying edges follow the local order (01, 02, 03, 12, 13, 23). Their parallel-copy counts are respectively (3, 3, 3, 2, 2, 2), giving 15 physical edges per block. The offsets of these six edge groups are (0, 3, 6, 9, 11, 13). A physical edge's global index is 15b + offset + copy, where b is the block index and copies start at zero.

Feasible sets are forests, including partial and empty forests. The matroid has rank six. Each block has 352 forests and 243 spanning trees, so the global counts are:

- All feasible forests: 352 squared = **123,904**.
- Full-rank sets: 243 squared = **59,049**.

Cycles impose genuine graphic constraints. The articulation structure also allows computations to factor over the two blocks; this is a structured multigraph instance.

Each physical edge covers:

1. Its own private feature.
2. The shared topic for its underlying edge within its block.
3. The two triangle features corresponding to three-vertex subsets of that K4 containing the underlying edge.
4. Its block's shared feature.

There are 30 private features, 12 underlying-edge topics, eight triangle features, and two block features: 52 features in total. The normalization constant is Z = 6.974869748.

The four target bases are:

| Target | Physical edge indices |
|---|---|
| B0 | 6, 12, 14, 15, 25, 27 |
| B1 | 1, 5, 7, 22, 26, 29 |
| B2 | 3, 9, 13, 17, 19, 23 |
| B3 | 2, 10, 11, 20, 24, 28 |

Each target is a spanning tree in each block, and hence a global base.

### Three smooth objective changes

| Rounds | Mean-weight setting |
|---|---|
| [0, 40,000) | Hold target B0 |
| [40,000, 80,000) | Cosine transition B0 to B1 |
| [80,000, 140,000) | Hold target B1 |
| [140,000, 180,000) | Cosine transition B1 to B2 |
| [180,000, 250,000) | Hold target B2 |
| [250,000, 290,000) | Cosine transition B2 to B3 |
| [290,000, 500,000) | Hold target B3 |

### Compared methods

| Figure label | Implementation and selected parameters |
|---|---|
| ours | Cumulative Gaussian FTPL; gamma = 0.1, sigma = 1000/3, 14 perturbed bases per update |
| ETCG | Greedy exploration restricted to feasible insertions; paper-prescribed m = 124 consecutive samples per candidate; no restart |
| EXP3 | Classic exponential-weight update with gamma = 0.5 and eta = gamma/K, where K = 123,904 |

EXP3 treats every complete feasible forest as a separate arm, including partial and empty forests. Its update for the chosen arm a is

$$
p_{t,a}=\frac{\gamma}{K}
+(1-\gamma)\frac{W_{t,a}}{\sum_bW_{t,b}},
\qquad
W_{t+1,a}=W_{t,a}
\exp\!\left(\frac{\gamma}{K}\frac{f_t(S_t)}{p_{t,a}}\right).
$$

All other arm weights remain unchanged. A sparse implementation keeps implicit unit initial weights for unvisited arms and explicitly stores visited excess weights. Sampling and updates are algebraically equivalent to these formulas. Feedback is not shared between different complete arms.

## Practical FTPL implementation

The policy starts at the reward-independent uniform fractional base x0 = k/n. The Poisson walk starts from the lexicographic greedy base: elements (0, 1, 2, 3, 4) for cardinality and edges (0, 3, 6, 15, 18, 21) for the graphic matroid.

The implementation uses mixed uniform/policy probing with mixing coefficient beta = 0.5, weighted Poisson-event sampling, and a signed-gain estimator. The Poisson starting parameter is epsilon = 0.05, the event-count mean is k log(1/epsilon), the event cap is L = 300, and the contraction parameter is delta = 0.01.

Estimated gains accumulate throughout the entire run. After each round, 14 independent Gaussian-perturbed maximum-weight bases are averaged and mixed with delta times x0 to form the next fractional policy. There is no discounting, model estimator, or reset at objective transitions. Feasible balanced exchange kernels realize the policy; the graphic implementation exploits the graph's block structure.

These experiments evaluate this practical estimator/FTPL variant with tuned parameters. They do not establish that the precise tested parameter choices inherit the original theoretical regret bound unchanged.

## Parameter selection and evaluation

The same eight development seeds and 80 test seeds are used in each experiment:

- Development: 6,130,101 through 6,130,108.
- Test: 6,131,101 through 6,131,180.
- Learner seed: 43 times the environment seed, plus one.

For each environment and seed, every compared method receives the same function sequence. The learner does not receive the target sets, drift schedule, or feature weights. Weights are generated using NumPy 2.4.6 default_rng with float32 Gaussian draws; weight sums and function values use float64 accumulation.

The parameter grids are:

| Method | Development grid |
|---|---|
| ours | gamma in {0.03, 0.1, 0.3, 0.5}; sigma in {1, 10, 100, 300, 1000}; 14 perturbed bases |
| OGo | gamma in {0.03, 0.1, 0.3, 0.5}; eta in {0.003, 0.01, 0.03, 0.1, 0.3, 1} |
| EXP3 | gamma in {0.003, 0.01, 0.03, 0.1, 0.3, 0.5, 1}; eta constrained to gamma/K |

Candidates are screened over the full horizon on the first two development seeds. The best FTPL perturbation scale and OGo update coefficient are additionally refined by factors of 1/3 and 3. The top three candidates per method are then evaluated on all eight development seeds, and the highest mean reward determines the selected parameters. The two constraint settings are tuned separately. No test-seed results are used for parameter selection.

ETCG uses its paper-prescribed exploration length, with no parameter search:

$$
m=\left\lceil
\left(
\frac{T\sqrt{2\log T}}
{n+2nk\sqrt{2\log T}}
\right)^{2/3}
\right\rceil.
$$

The reported figures include only the methods listed for each experiment. Exact offline comparators are used solely for evaluation. All selected actions and observed rewards have been independently checked for feasibility and replayed against the generated functions.

## Interpretation

These are nonstationary synthetic experiments. ETCG estimates a greedy solution and then commits, so its selected set cannot adapt to subsequent objective changes. Its behavior here does not contradict its guarantees for a fixed stochastic reward distribution. The graphic ETCG baseline is a feasible-greedy extension; its original cardinality guarantee is not asserted for this extension.

The results illustrate the performance of a practical exchange-based FTPL implementation on these instances. They do not establish universal superiority over the baselines or a guarantee relative to the changing per-round optimum.
