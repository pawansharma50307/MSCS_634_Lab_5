# MSCS 634 – Lab 5: Clustering Techniques Using DBSCAN and Hierarchical Clustering

**Author:** Pawan Sharma
**Course:** MSCS 634 – Advanced Big Data and Data Mining

## Purpose

In this lab, I have performed the comparison of two clustering methods: Agglomerative Hierarchical Clustering and DBSCAN on the scikit-learn dataset “Wine”. My goal was to check the ability of these algorithms to group data, how sensitive is each of them to the parameters’ changes and to what extent the obtained results are close to the actual labels. As a quality measure, I applied Silhouette, Homogeneity and Completeness. The ground truth labels were used only for scoring.

## Files

| File | Description |
|---|---|
| `Lab5_Clustering_DBSCAN_Hierarchical.ipynb` | Full notebook: data exploration, standardization, hierarchical clustering + dendrogram, DBSCAN parameter search, evaluation, and analysis |
| `README.md` | This summary |

## How to Run

```bash
pip install numpy pandas matplotlib seaborn scikit-learn scipy jupyter
jupyter notebook Lab5_Clustering_DBSCAN_Hierarchical.ipynb
```

## Key Results

| Model | Clusters | Noise | Silhouette | Homogeneity | Completeness |
|---|---|---|---|---|---|
| Agglomerative (Ward, k=3) | 3 | 0 | 0.277 | **0.790** | **0.783** |
| DBSCAN (eps=2.5, min_samples=10) | 2 | 32 | 0.323* | 0.504 | 0.560 |

\*DBSCAN silhouette is computed on non-noise points only.

## Key Insights

- **Hierarchical clustering worked best.** Ward linkage with k = 3 recovered the three cultivars well (homogeneity and completeness both about 0.79). k = 3 also had the highest silhouette of the values I tested (k = 2 to 6).
- **The dendrogram agreed.** The last two merges happen at heights of about 27.7 and 35.4, compared with about 12.6 for the merge before them. That large gap is a strong sign that the data has three natural groups.
- **Linkage mattered a lot.** Ward and complete linkage gave balanced clusters. Average and single linkage put 174 of the 178 wines into one cluster (the chaining effect).
- **DBSCAN was very sensitive to its parameters.** With `eps` ≤ 1.5, almost every point was noise. With `eps` ≥ 2.6 and a small `min_samples`, everything merged into one cluster. Only a narrow range (about 2.2 to 2.5) gave useful results, and even there DBSCAN usually merged `class_0` and `class_1` into one cluster while separating `class_2` cleanly.
- **DBSCAN's higher silhouette is misleading here.** It gets that score by dropping the hardest border points as noise, so the clusters that remain look tighter than they really are.
- **The number of dimensions hurts DBSCAN.** In a side experiment, DBSCAN on the 2-D PCA projection found 3 clusters and scored much better (homogeneity 0.754). This suggests the 13-D space was a big part of why it struggled.

## Challenges and Decisions

- **Scaling:** The features are on very different scales (`proline` goes past 1,000, while `hue` stays below 2), so I standardized every feature before clustering. Both algorithms rely on Euclidean distance.
- **Visualizing 13-D clusters:** I used PCA (2 components) only for the scatter plots. All models were fit on the full standardized feature set.
- **Choosing `eps`:** I used a k-distance plot to find a starting range, then ran a grid search over `eps` (1.5 to 3.0) and `min_samples` (3 to 10).
- **Picking the "best" DBSCAN run:** Choosing by silhouette alone favored settings that labeled most points as noise. Instead, I kept only runs with at least 2 clusters and no more than 30% noise, then picked the one with the highest V-measure.
- **Silhouette with noise:** Noise points (`-1`) aren't a real cluster, so I left them out of the silhouette calculation and noted this in the results.