# Clustering Flight Delay Patterns for Operational Optimization

Which groups of U.S. airline–airport operations share the same delay behaviour, and which delay cause dominates each group? This project clusters monthly delay records with PCA, feature selection, k-Means++ and OPTICS to find patterns that airlines could act on.

## Highlights

- **Data**: 89,077 monthly records (carrier x airport x month, 2015–2019) with 21 features: flight counts, delay counts and delay minutes by cause (carrier, weather, NAS, security, late aircraft)
- **Preprocessing**: missing values filled with 0, 99 rows with zero flights removed (**88,978 rows**), two engineered features (`delay_rate`, `total_delay_hours`)
- **EDA**: carrier and late-aircraft delays move together and dominate total delay minutes. A combined ranking puts C5, B6, AX, F9 and NK as the five worst-performing carriers.
- **Three dataset versions**: full scaled data, **PCA** (3 components, about 74% of variance; PC1 alone 59%) and **VarianceThreshold** (2 low-variance features removed)
- **Clustering**: k-Means++ silhouette scores as a reference, then **OPTICS** with distance cuts chosen from the reachability plots for each version
- **Main finding**: in the selected PCA clustering, **late-aircraft delay is the dominant cause in 9 of 13 clusters**, NAS delay in 3 and carrier delay in 1

## Results

k-Means++ (best k per dataset version, silhouette score):

| Dataset | Best k | Silhouette |
|---|---|---|
| Full data | 2 | 0.7960 |
| PCA (3 components) | 2 | 0.7681 |
| Variance threshold | 2 | 0.8053 |

OPTICS (`min_samples=10`, Euclidean distance), one distance cut per version:

| Dataset | eps | Clusters | Noise points |
|---|---|---|---|
| Full data | 0.50 | 10 | 24,713 |
| **PCA** | **0.30** | **13** | **2,706** |
| Variance threshold | 0.50 | 13 | 19,955 |

Agreement between the clusterings:

| Comparison | ARI | AMI |
|---|---|---|
| Full vs. PCA | -0.0094 | 0.0457 |
| Full vs. VT | 0.8615 | 0.8778 |
| PCA vs. VT | -0.0015 | 0.0519 |

The PCA clustering was selected as the final result because it has by far the least noise. However, 86,099 of the 88,978 rows fall into a single large cluster, so the 12 small clusters describe niche patterns only. The full and variance-threshold clusterings are almost identical (ARI 0.86), while PCA produces a very different partition.

Recommended actions derived from the clusters: reduce late-aircraft propagation (turnaround, gate coordination, schedule buffers), handle the NAS-heavy clusters with ATC and routing measures, and treat OPTICS noise points as outliers worth investigating separately.

## Project structure

| File | Description |
|---|---|
| `final.ipynb` | Full analysis: data description, IDA, preprocessing, EDA, PCA, feature selection, k-Means++, OPTICS, comparison, conclusions |
| `Airline_Delay_Cause.csv` | Raw dataset (about 14 MB) |
| `requirements.txt` | Python dependencies |

## Data source

U.S. Department of Transportation, **Bureau of Transportation Statistics (BTS)**: [Airline On-Time Statistics and Delay Causes](https://www.transtats.bts.gov/ot_delay/ot_delaycause1.asp). The data is public U.S. government data, reported by U.S. air carriers, January 2015 to December 2019.

## How to run

```bash
pip install -r requirements.txt
jupyter notebook final.ipynb
```

OPTICS on about 89,000 rows is slow; expect several minutes per fit.

## Limitations and next steps

- The data is aggregated per carrier, airport and month, not per flight, so large airports and carriers dominate.
- OPTICS results depend strongly on scaling, `min_samples` and the eps cut. Only `min_samples=10` was tested.
- The PCA clustering is very unbalanced (one cluster holds 97% of the rows), and 3 components keep only about 74% of the variance.
- Noise points may be the most important operational events (storms, disruptions) and deserve their own analysis.
- A dominant delay cause in a cluster does not prove causality.
- Next steps: HDBSCAN on the PCA data, a wider `min_samples` search, seasonal/yearly clustering, and additional features such as aircraft type or airport congestion.

**Author:** Mehdi Gurbanzade · M.Sc. Data Science
