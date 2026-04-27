# Wine Reviews Text Mining & Market Analysis (RapidMiner)

A text mining and machine learning project applying three parallel analytical pipelines to 130,000+ wine reviews - correlation analysis, association rule mining with FP-Growth, and K-Means clustering - to answer three specific business questions about pricing, regional quality, and grape variety patterns.

<p align="center">
  <a href="Wine Reviews.pdf">View Full Project Presentation</a> · <a href="https://tejashwinisaravanan.github.io/wine-reviews-text-mining-rapidminer/">View GitHub Pages</a>
</p>

<p align="center">
  <img src="visualizations/process .png" width="680" alt="RapidMiner three-pipeline workflow diagram"/>
</p>

<p align="center"><em>The full RapidMiner pipeline - three parallel sub-processes running correlation analysis, FP-Growth association rule mining, and K-Means clustering simultaneously on the same dataset.</em></p>

---

## Overview

Wine review data is rich, messy, and largely unstructured. A dataset of 130,000+ reviews with free-text descriptions, numeric quality scores, prices, taster identities, and geographic metadata contains multiple layers of insight - but only if the right analytical tools are applied to each layer separately.

This project treats the dataset as three distinct analytical problems running in parallel. The numeric layer - points and price - is interrogated with correlation analysis. The text layer - free-form review descriptions - is processed through a document pipeline and mined for frequent itemsets using FP-Growth. And the combined numeric and text features are used to segment wines into natural clusters using K-Means.

The same multi-pipeline approach used here maps directly to clinical text mining in healthcare. Electronic health records contain exactly this structure: numeric lab values, free-text clinical notes, categorical diagnosis codes, and geographic patient data. The FP-Growth association rule technique used to discover grape variety patterns is the same technique applied to medical co-morbidity mining - discovering which diagnoses frequently appear together in patient populations. This project demonstrates that capability on a large, publicly available dataset.

---

## The Data

| | |
|---|---|
| Source | [Kaggle - Wine Reviews](https://www.kaggle.com/datasets/zynicide/wine-reviews) |
| Original Records | 130,000+ reviews |
| Filtered Working Set | ~9,000 records after scoping to US wines |
| Attributes | 14 total - variety, location, winery, price, points, description, taster handle |
| Numeric Fields | Points (80-100 scale), Price (USD) |
| Text Field | Description (free-text taster reviews) |
| Tool | RapidMiner Studio |

**Key statistics across the working dataset:**

Points range from 80 to 100 with an average of 89.15 and standard deviation of 2.83. Price ranges from $4 to $2,013 with an average of $37.58 and standard deviation of $27.25. Missing values were handled using RapidMiner's Replace Missing Values operator with mean imputation for numeric fields.

<p align="center">
  <img src="visualizations/statistics .png" width="600" alt="Dataset statistics overview showing attribute distributions"/>
</p>

<p align="center"><em>Attribute-level statistics confirming data quality and distribution before pipeline execution - a necessary validation step before any text mining can be trusted.</em></p>

---

## The Three Research Questions

Before building any pipeline, three specific business questions were defined to keep the analysis focused and actionable:

1. Does a wine's price depend on reviewer ratings - is expensive wine actually better?
2. Which US province produces the highest-rated wines?
3. What are the most commonly occurring grape variety combinations in wine descriptions?

Each question required a different analytical technique, which is why the pipeline architecture runs three sub-processes in parallel rather than sequentially.

---

## Pipeline Architecture

### Pipeline 1 - Correlation Analysis (Price vs. Quality)

```
Retrieve → Select Attributes (points, price) → Filter Examples → Correlation Matrix
```

A Pearson Correlation Matrix was computed between points and price to quantify the statistical relationship between rating and cost.

<p align="center">
  <img src="visualizations/correlation Matrix .png" width="560" alt="Pearson correlation matrix between points and price"/>
</p>

<p align="center"><em>Pearson correlation matrix - points and price show r = 0.43, a weak positive relationship that answers the first research question directly.</em></p>

| | Points | Price |
|---|---|---|
| Points | 1.000 | 0.430 |
| Price | 0.430 | 1.000 |

**Finding:** Points and price are weakly positively correlated (r = 0.43). Higher-rated wines tend to cost more, but the relationship is not strong enough for price to serve as a reliable quality proxy. A wine priced at $200 is not reliably better than one priced at $40 - the rating data does not support that assumption.

<p align="center">
  <img src="visualizations/bell curve - plot .png" width="540" alt="Price distribution bell curve showing concentration around $30-40"/>
</p>

<p align="center"><em>Price distribution for US wines - near-normal, centered around $30-40. The long right tail confirms that premium-priced wines are the exception, not the norm, in this dataset.</em></p>

---

### Pipeline 2 - Association Rule Mining (Grape Variety Patterns)

```
Retrieve → Replace Missing Values → Select Attributes → Process Documents from Data →
Numerical to Binominal → FP-Growth → Create Association Rules
```

The text descriptions were processed through RapidMiner's Process Documents operator, which tokenizes the free text, removes stop words, and converts terms to a binary presence/absence representation. FP-Growth was then applied to find frequent itemsets - combinations of words that appear together more often than chance would predict.

Association rules were generated from the frequent itemsets with a minimum confidence threshold. Confidence measures the conditional probability that the consequent appears given the antecedent - a confidence of 1.000 means the rule held without exception across the entire dataset.

| Rule | Confidence | Interpretation |
|---|---|---|
| [bordeaux] → [blend] | 1.000 | Every Bordeaux mention includes "blend" - 100% of the time |
| [style, bordeaux] → [blend] | 1.000 | Bordeaux style references always co-occur with blend terminology |
| [noir] → [pinot] | 0.997 | "Noir" appears with "pinot" in 99.7% of cases |
| [sparkling] → [blend] | 0.955 | Sparkling wine descriptions almost always reference blending |
| [blanc] → [sauvignon] | 0.784 | "Blanc" co-occurs with "sauvignon" in 78.4% of descriptions |

**Finding:** Bordeaux Blend is the dominant variety pattern in the dataset, appearing with 100% confidence in multiple rules. Pinot Noir is the most tightly associated two-word pair (0.997 confidence), confirming that tasters use these terms inseparably in review language. These rules could be used to automatically classify review descriptions by variety without requiring a human tagger.

---

### Pipeline 3 - K-Means Clustering (Wine Segmentation)

```
Retrieve → Replace Missing Values → Select Attributes → Process Documents from Data →
Clustering (K-Means, k=5)
```

K-Means clustering was applied to segment wines into natural groupings based on their text description features. k=5 was selected as the target number of clusters.

<p align="center">
  <img src="visualizations/Cluster Model .png" width="560" alt="K-Means cluster model output showing five wine segments"/>
</p>

<p align="center"><em>K-Means cluster model - five segments identified based on common flavor and style terminology in review descriptions.</em></p>

| Cluster | Size | Dominant Terms |
|---|---|---|
| Cluster 0 | 32,235 | General mixed - the majority cluster |
| Cluster 1 | 21 | Citrus-forward notes |
| Cluster 2 | 15 | Fruity profiles |
| Cluster 3 | 61 | Berry and spice combinations |
| Cluster 4 | 81 | Brand and producer-focused language |

**Performance metrics:** Average within-centroid distance of -1.009 and Davies-Bouldin Index of -12.444.

**An honest assessment of the clustering result:** The severe size imbalance - Cluster 0 containing 32,235 records while Clusters 1-4 contain between 15 and 81 - indicates that the k=5 selection did not produce well-separated clusters. The majority of reviews share generic tasting language that K-Means cannot distinguish, while only a small subset use highly specific flavor terminology that creates tight minority clusters. This is a real limitation of applying K-Means to sparse, high-dimensional text features without dimensionality reduction. A more robust approach would apply TF-IDF weighting to reduce the dominance of common terms, followed by PCA or SVD to reduce dimensionality before clustering. This is documented as the primary methodological improvement for future work.

<p align="center">
  <img src="visualizations/Data From Process .png" width="560" alt="Clustered output data showing cluster assignments per record"/>
</p>

<p align="center"><em>Cluster assignment output - each wine review is tagged with its cluster label, enabling downstream filtering and targeted analysis by flavor profile.</em></p>

---

## Regional Quality Analysis

<p align="center">
  <img src="visualizations/box plot .png" width="560" alt="Box plot of wine points by US province"/>
</p>

<p align="center"><em>Points distribution by province - California leads in median score, with Oregon and Washington showing consistently strong performance. New York's wider variance reflects a more uneven quality distribution.</em></p>

| Province | Median Points | Quality Profile |
|---|---|---|
| California | ~90 | Highest median, consistent performer |
| Oregon | ~89 | Strong median, tighter distribution |
| Washington | ~89 | Comparable to Oregon |
| New York | ~84 | Lower median, higher variance |

**Finding:** California produces the highest-rated wines on average among US provinces, but Oregon and Washington close the gap significantly. New York's higher variance suggests a more bifurcated market - some excellent producers alongside a larger volume of lower-scoring wines.

---

## Taster and Winery Distribution

<p align="center">
  <img src="visualizations/Histogram .png" width="540" alt="Histogram of review points frequency by taster"/>
</p>

<p align="center"><em>Points frequency by taster - most reviews fall between 87 and 92 points. The distribution is left-skewed, confirming that wine reviewers rarely assign scores below 85 regardless of actual quality - a known bias in professional wine scoring.</em></p>

The concentration of scores in the 87-92 range is itself an analytical finding. Professional tasters rarely use the full 80-100 scale, which compresses the effective range and reduces the discriminating power of numeric scores. This compression is one reason the price-points correlation of 0.43 is relatively low - the limited score variance makes it harder to distinguish quality tiers statistically.

<p align="center">
  <img src="visualizations/scatter plot .png" width="540" alt="Scatter plot of winery distribution by region"/>
</p>

<p align="center"><em>Winery geographic distribution - Willamette Valley and California show the highest producer density, consistent with their dominance in the quality rankings.</em></p>

---

## Key Findings

**Price is not a reliable quality proxy.** A Pearson correlation of r = 0.43 between price and points is statistically significant but practically weak. Consumers and buyers who use price as a quality signal are making decisions with a 57% unexplained variance in quality outcomes. Blind tasting and score-based selection outperform price-based selection on this dataset.

**Bordeaux Blend dominates the variety landscape.** With 100% confidence across multiple association rules, Bordeaux Blend is the most systematically referenced grape combination in the review corpus. Any producer or retailer building a portfolio around consumer vocabulary in reviews should center Bordeaux Blend terminology.

**California leads but the Pacific Northwest is competitive.** Oregon and Washington trail California's median by approximately one point - a gap small enough that sourcing strategies should treat all three states as premium-tier regions rather than treating California as uniquely superior.

**Professional scoring uses a compressed scale.** The concentration of scores between 87 and 92 points means that numeric differentiation between wines is limited. Category-based segmentation - using association rules and clustering - provides additional discriminating power that the numeric score alone cannot.

---

## Business and Industry Applications

**For wine retailers and sommeliers:** The association rules provide a vocabulary map of how professional tasters describe wines. Retailers can use this map to align shelf descriptions and marketing copy with the language consumers will encounter in professional reviews - improving search and recommendation accuracy.

**For producers and vineyard managers:** The regional quality analysis quantifies California's premium positioning and Oregon/Washington's competitive standing. Producers in emerging regions can use this benchmark to set realistic pricing and positioning strategies relative to established appellations.

**For data teams in healthcare:** The FP-Growth pipeline architecture used here is directly applicable to clinical co-morbidity mining. Replacing wine descriptions with clinical notes and grape variety terms with diagnosis codes produces an association rule engine that discovers which medical conditions frequently co-occur - a standard technique in population health analytics and clinical decision support systems.

---

## Limitations and What I Would Do Next

The clustering result is the most important area for improvement. Applying TF-IDF weighting to the document-term matrix before K-Means would reduce the influence of generic high-frequency terms that pull most records into a single large cluster. Dimensionality reduction via SVD or PCA before clustering would further improve cluster separation. These two changes would likely produce meaningfully distinct flavor profile segments rather than one dominant cluster and four micro-clusters.

Sentiment analysis on the review text is the next highest-value analytical addition. Classifying descriptions as positive, negative, or mixed sentiment and correlating sentiment polarity with price and rating would add a layer of insight not available from frequency-based text mining alone.

A predictive model - using review text to predict variety, region, or price tier - would extend the project from descriptive to predictive analytics. A Naive Bayes or SVM text classifier trained on the full 130K dataset would be a natural next step, and it would demonstrate that the association rules extracted here generalize to classification tasks.

---

## Repository Structure

```
wine-reviews-text-mining-rapidminer/
│
├── visualizations/
│   ├── process .png                    # Full RapidMiner pipeline diagram
│   ├── statistics .png                 # Dataset attribute statistics
│   ├── correlation Matrix .png         # Pearson correlation - points vs price
│   ├── bell curve - plot .png          # Price distribution
│   ├── box plot .png                   # Points by US province
│   ├── Histogram .png                  # Points frequency by taster
│   ├── scatter plot .png               # Winery distribution by region
│   ├── Cluster Model .png              # K-Means clustering output
│   └── Data From Process .png          # Cluster-tagged output data
│
├── Wine Review .rmp                    # RapidMiner process file (fully reproducible)
├── Wine Reviews.xlsx                   # Working dataset (~9,000 filtered records)
├── Wine Reviews.pdf                    # Full project presentation slides
├── LICENSE                             # MIT License
└── README.md
```

---

## Limitations and What I Would Do Next

The clustering result is the most important area for improvement. Applying TF-IDF weighting before K-Means, combined with SVD dimensionality reduction, would produce more balanced and interpretable clusters. Sentiment analysis on review text and a full predictive classification model are the two highest-value next steps.

---

## Tools and Technologies

RapidMiner Studio · FP-Growth · K-Means Clustering · Pearson Correlation · Text Mining · Association Rule Mining · Microsoft Excel

---

## Related Projects

- **[Clinical Trial Patient Selection](https://github.com/TejashwiniSaravanan/Clinical-Trial-Patient-Selection-Optimization)** - The same data mining techniques applied to patient classification in a healthcare context
- **[Global Affordability Dashboard](https://github.com/TejashwiniSaravanan/global-affordability-dashboard-tableau-ml)** - K-Means clustering applied to economic segmentation across 3,700+ cities

---

## About Me

**Tejashwini Saravanan** - Master's student in Data Analytics at Seattle Pacific University, focused on healthcare data engineering, text mining, and scalable ML pipelines.

[LinkedIn](https://www.linkedin.com/in/tejashwinisaravanan/) · [GitHub](https://github.com/TejashwiniSaravanan)

---

*Dataset: Kaggle Wine Reviews · Tool: RapidMiner Studio · License: MIT · Seattle Pacific University*
