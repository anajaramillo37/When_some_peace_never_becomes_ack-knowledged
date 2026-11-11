# When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies.

This library presentes a detailed methodology and empirical results of the paper [**When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies**](link), published in xxxxxx.


## 👥 Who is this for?
- **Researchers from the social sciences** that would like to perform bibliometrics or text analyses.
- **Social computer sciences** that would like to replicate this study in other domains than peace and conflict.
- **Policymakers and analysts** who would like to get involved in research policies to increase the participation of non-meanstream perspectives in the studies of peace and conflict.

## ✨ About

Through an analysis of 36,975 peace and conflict studies articles (1958-2024), this paper develops a novel framework for examining the epistemic conditions that prevent certain understandings of peace from becoming objects of knowledge. We argue that this stems not only from racialized, gendered, and Western-centric hierarchies, but from a coloniality of non-knowledge that selectively and structurally ignores particular actors, contexts, and histories. This shapes who is recognized as a legitimate knowledge producer and whose visions of peace matter. By foregrounding non-knowledge, we reveal how epistemic exclusions can constrain opportunities for peace for all.

## 🚀 Highlights
- 📈 Open Alex datasets about multilingual research on peace & conflict available for reproducibility 
- ⚖️ Full code to reproduce BERT based text labelling on concepts of peace and conflict.

## Table of Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Quick Start](#start)
4. [How to use the library](#how-to)
5. [Comparison with FA*IR](#fair)
6. [Bugs and Feedback](#bugs)


You can directly install the package using pip:

```sh
pip install hyperfair
```

Or you can clone the repository and install the dependencies:

```sh
git clone https://github.com/CSHVienna/hyper_fair.git
cd hyper_fair
pip install -r requirements.txt
```

## Quick Start<a name="start">

Get started with a single line: compute the p-value for fairness in your ranking!

Suppose you have a ranking where `1` indicates a protected candidate and `0` an unprotected candidate. To test whether the protected group is under-represented in the top-$k$ positions, simply run:

```python
from hyperfair.hyperfair import measure_fairness_multiple_points

pvalue, _ = measure_fairness_multiple_points(
    x_seq=[0, 0, 0, 0, 1, 0, 1, 1, 1, 1],  # 1=protected, 0=unprotected
    k=10,  # Test the top 10 positions
    test_side='lower',  # Test for under-representation
    n_exp=100_000  # Number of Monte Carlo simulations
)
print(f"P-value: {pvalue:.3f}")
# Output: P-value: 0.023
```

This tells you how likely it is to observe as few protected candidates in the top $k$ as you did, under random selection. A small p-value (e.g., < 0.05) means the ranking is likely unfair to the protected group.

## How to use the library<a name="how-to">

### Loading data from a Pandas DataFrame

To analyze fairness in your rankings, you first need to load your data. The most stratight forward way is to load it from a Pandas DataFrame, and the package provides the [`load_data_from_pandas_df`](code/data_loader.py) function for this purpose. This function extracts the relevant ranking and protected attribute information from your DataFrame and prepares it for fairness analysis.

Suppose your DataFrame has the following structure:

|   ID | SES      |   Score |
|-----:|:---------|--------:|
|    0 | Low SES  |    4.82 |
|    1 | High SES |    6.87 |
|    2 | High SES |    7.84 |
|    3 | Low SES |    4.17 |
|    4 | High SES |    4.71 |
|  ... |   ...    |   ...   |

Here, `SES` (socioeconomic status) is the protected attribute, and `Score` is the ranking criterion.

To load and process this data, use:

```python
from hyperfair.data_loader import load_data_from_pandas_df
import pandas as pd

df = pd.read_csv(CSV_PATH)
ranking, ids = load_data_from_pandas_df(
    df,
    protected_attribute='SES',
    binary_dict={'Low SES': 1, 'High SES': 0},
    id_attribute='ID',
    order_by='Score',
    ascending=False  # Set to True if higher scores are better
)
```

**Inputs:**
- `df`: A pandas DataFrame containing your ranking data.
- `protected_attribute`: The column name in `df` that indicates group membership.
- `binary_dict`: A dictionary mapping the values in the protected attribute column to 1 (protected group) and 0 (unprotected group).
- `id_attribute`: The column name in `df` that uniquely identifies each candidate. If None, it selects the index of the DataFrame.
- `order_by`: The column name in `df` used to rank candidates.

**Outputs:**
- `ranking`: A NumPy array of 0s and 1s, ordered by rank (after sorting), where 1 indicates a protected candidate and 0 an unprotected candidate. This is the main input for all fairness analysis functions.
- `ids`: A NumPy array of candidate IDs, ordered in the same way as `ranking`.

These outputs allow you to analyze the representation of protected and unprotected groups at every position in the ranking, and are required for all subsequent fairness tests and visualizations in the package.

### Sequential tests for fairness

A key feature of **hyperFA*IR** is the ability to test rankings for fairness at multiple cutoffs. For example, you may want to check if the protected group is under-represented in the top $k$ positions of your ranking, or across all prefixes [1:j] for $j=1,\ldots,k$.

To do this, use the [`measure_fairness_multiple_points`](code/hyperfair.py) function. This function performs sequential statistical tests (using Monte Carlo simulations) to determine whether the observed representation of the protected group is consistent with random selection.

Example usage:

```python
from hyperfair.hyperfair import measure_fairness_multiple_points

pvalue, generatedData = measure_fairness_multiple_points(
    x_seq=ranking,      # binary array: 1 if protected, 0 otherwise, ordered by rank
    k=30,               # number of top positions to test
    test_side='lower',  # test for under-representation
    n_exp=1000000       # number of Monte Carlo simulations
)
```
- `x_seq`: Binary array indicating protected group membership, sorted by ranking.
- `k`: Number of top positions (prefixes) to test.
- `test_side`: Use `'lower'` to test for under-representation, `'upper'` for over-representation, or `'two-sided'` for both.
- `n_exp`: Number of Monte Carlo simulations for estimating p-values.

The function returns the p-value for the fairness test and a `generatedData` object that can be reused for further analysis or re-ranking.

For a more detailed guide and practical examples, see [example.ipynb](example.ipynb).

## Comparison with FA*IR<a name="fair">

Finally, in the paper we mention that our method to compute the adjusted parameter for multiple tests is more efficient than the one implemented in [FA*IR](https://arxiv.org/abs/1706.06368) because it doesn't rely on the linear search of the optimal parameter.

To support our claim, we show the differences in speed (right plot below) between our method (in red) and the method implemented in the [companion repository](https://github.com/fair-search/fairsearch-fair-python) of the paper by Zehlike et al.. In particular, we show the elapsed time (in seconds) using a log-log plot, as a function of the length of the ranking.
For our Monte Carlo algorithm, we fixed the number of experiments to $100000$.

![Logo](comparison.png)

For small `n`, our method is slightly slower. However, this is also the scenario in which our method is significantly more accurate. When `n` increases, it is clear that our method is much more efficient (remember that the time is in log-scale!). 

For `n=2000` (last point), the difference is substantial: the elapsed time for Zehlike's algorithm is $2816$ seconds, while our algorithm only takes around $37$ seconds to run.

## Bugs and Feedback<a name="bugs">

Please report any bugs that you find or any other feedback directly to [cartiervandissel@csh.ac.at](mailto:cartiervandissel@csh.ac.at). We welcome all changes, big or small. Thank you!

## 📜 Attributions
```bibtex
@inproceedings{cartier2025hyperfa,
  title={hyperFA* IR: A hypergeometric approach to fair rankings with finite candidate pool},
  author={Cartier van Dissel, Mauritz N and Martin-Gutierrez, Samuel and Esp{\'\i}n-Noboa, Lisette and Jaramillo, Ana Mar{\'\i}a and Karimi, Fariba},
  booktitle={Proceedings of the 2025 ACM Conference on Fairness, Accountability, and Transparency},
  pages={2112--2126},
  year={2025}
  }
```
- **Project:** [MAMMOth](https://mammoth-ai.eu/)
- **Maintainer:** Mauritz Cartier van Dissel (cartiervandissel@csh.ac.at)
- **License:** Apache 2.0
- **Contributors:** Ana Maria Jaramillo
