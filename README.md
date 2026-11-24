# When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies.

This library presentes a detailed methodology and empirical results of the paper [**When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies**](link), published in 31,506.


## 👥 Who is this for?
- **Researchers from the social sciences** that would like to perform bibliometrics or text analyses.
- **Social computer sciences** that would like to replicate this study in other domains than peace and conflict.
- **Policymakers and analysts** who would like to get involved in research policies to increase the participation of non-meanstream perspectives in the studies of peace and conflict.

## ✨ About

Through an analysis of 31,506 peace and conflict studies articles (1958-2024), this paper develops a novel framework for examining the epistemic conditions that prevent certain understandings of peace from becoming objects of knowledge. We argue that this stems not only from racialized, gendered, and Western-centric hierarchies, but from a coloniality of non-knowledge that selectively and structurally ignores particular actors, contexts, and histories. This shapes who is recognized as a legitimate knowledge producer and whose visions of peace matter. By foregrounding non-knowledge, we reveal how epistemic exclusions can constrain opportunities for peace for all.

## 🚀 Highlights
- 📈 Open Alex datasets about multilingual research on peace & conflict available for reproducibility 
- ⚖️ Full code to reproduce BERT based text labelling on concepts of peace and conflict.

## Table of Contents

1. [Methodology overview](#overview)
2. [Data labeling](#labeling)
3. [Human verification](#humans)
4. [Citation analyses](#citations)

## Methodology overview<a name="overview">

Flow chart with the steps to download the most worldwide diverse literature on Open Alex by different languages of publication and regions of the world for peace & conflict literature.

<p align="left">
  <img src="Methodology_flow_diagram.png" alt=“MethodsFlowChart" width=“500" style="vertical-align: middle; margin-right: 10px;"/>
</p>

An exhaustive list of the files from which each step is taking place is as follows: 1) translated terms peace & conflict is in this table [link] for 86+ lenagues; 2) files derived from the manual search of the translated terms in Open Alex after manual verification of the top results and topic displayed by the platform is downloaded and stored in this folder [link]; 3) Filtering those papers with an abstract and detection of language of each abstract is in this file [link]; translation of the abstracts is carried in this file [file] and the corresponding translation files are in this folder [link]. In this file is the final merge of translated abstracts and this file [link] contains the entire final dataset for continuing the analyses.

## Quick Start<a name="labeling">

In this code you can find the labelling process where we: We first searched for the explicit appearance of each of the 30 dictionary terms in each translated abstract and in the ‘primary_topic’ defined by Open Alex. We then applied sentence transformers (Reimers and Gurevych, 2019), a natural language processing methodology that represents sentences as numerical vectors and measures their semantic proximity, to assess whether the conceptual content (its definition provided in the appendix Table A1) of a term was present in the abstract even when the exact word was not. In particular, we used the sentence-transformers model 'all-MiniLM-L6-v2' hosted in hugging-face, which has been previously trained with over a thousand million texts ranging from blogs to scientific academic content. We first removed stop words, numerical content, and then transformed the definitions into a numerical vector. Then, for each translated abstract in English we repeated the text cleaning process and this time we first divided the abstract in sentences strictly delimited by dot (using the sentence tokenization of NTKL), and transformed each sentence to a vector. Then, for the sentences of each abstract, we computed the cosine similarity between i) the numerical vector transformation of the concept definition provided in Table A1 vs ii) the numerical vector transformation of each sentence. The cosine similarity ranges from -1 when there is no similarity to 1 when the two vectors are identical. As an abstract might contain multiple sentences, we selected the sentence with the highest cosine similarity value for our further comparisons. This last decision obeys to the principle that abstracts of scientific papers highlight the main arguments of the entire article, and while other arguments might be explored and shown in the abstract, we filtered by that at least one of the sentences was related to the searched definition. Distributions of the cosine similarity

## How to use the library<a name="humans">

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

## Comparison with FA*IR<a name="citations">

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
