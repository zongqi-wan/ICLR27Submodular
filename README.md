We compare our practical FTPL variant with existing methods under cardinality and graphic matroid constraints. Each experiment uses **500,000 rounds and 80 paired test seeds**. Parameters are selected on eight separate development seeds.

## Shared setting

Each round uses a weighted coverage function:

$$f_t(S)=\frac{1}{Z}\sum_{e\in\bigcup_{i\in S}C_i}w_{t,e}.$$

Coverage sets are fixed. Feature-weight means move smoothly between targets with different, pairwise disjoint optimal bases. Private-feature means are 1 for target elements and 0.005 for other elements; shared features have smaller, fixed means.

Weights receive independent Gaussian perturbations on **every round**, including periods with fixed means. The perturbation standard deviation is 2% of the mean, clipped at three standard deviations. A fixed normalization keeps function values in [0,1]. Each method receives one bandit observation per round and the same function sequence for each seed.

Curves show cumulative reward divided by the reward of the **best fixed feasible set** for the same prefix, including all exploration rounds. Shading shows pointwise 95% confidence intervals; gray bands mark drift intervals. Ratios can exceed one because the comparator uses a fixed set.

## Cardinality constraint

- **n = 20, k = 5**; any subset of at most five elements is feasible.
- Each element covers a private feature and a shared topic from one of five groups of four elements. These groups define coverage overlap only.
- Three target optima, with cosine transitions during rounds **40,000–80,000** and **140,000–180,000**. Mean weights are fixed between transitions.
- Methods: **ours**, **OGo**, and **ETCG** with paper-prescribed exploration length m = 182.

![Cardinality experiment](figures/cardinality.png)

[PDF](figures/cardinality.pdf)

## Graphic matroid

- **n = 30, k = 6**; feasible sets are forests in a seven-vertex multigraph formed by two K4 blocks sharing one vertex.
- Each block has 15 edges, with parallel-copy counts (3, 3, 3, 2, 2, 2). There are **123,904 feasible forests**.
- Coverage includes private edge features, underlying-edge topics, triangles, and blocks.
- Four target optima, using the two transition intervals above and an additional transition during rounds **250,000–290,000**.
- Methods: **ours**, **ETCG** with feasible greedy insertions and paper-prescribed m = 124, and **EXP3**, treating each feasible forest as an arm with the classic update and tuned exploration rate.

![Graphic matroid experiment](figures/graphic.png)

[PDF](figures/graphic.pdf)

## Results

Mean final cumulative reward ratios:

| Constraint | ours | OGo | ETCG | EXP3 |
|---|---:|---:|---:|---:|
| Cardinality | 0.835 | 0.850 | 0.189 | — |
| Graphic matroid | 0.804 | — | 0.290 | 0.448 |

Our method uses cumulative Gaussian FTPL with a modified bandit estimator, without discounting or resets. The original theorem is not claimed for this tuned implementation. ETCG commits after exploration, so these drifting environments test adaptation beyond its stationary setting.
