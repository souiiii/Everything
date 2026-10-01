# Midsem Numerical Practice — Units 1–3

Start with **[VERY HIGH]** questions, then **[HIGH]**, then **[MEDIUM]**. Attempt the practice problems before consulting the final-answer-only key.

Use a basic calculator where needed. Keep intermediate values unrounded and normally round final answers to **3 decimal places**. All covariance questions below use the **sample** denominator $n-1$. All vectors in a question use the same feature order. In rating matrices, **? means unknown, not zero**.

# Formula Quick Sheet

| Numerical type | Key formula | What it calculates | Reminder |
|---|---|---|---|
| TF-IDF | $TF=\frac{c_{td}}{L_d}$; $IDF=\log_{10}\frac{N}{df_t}$; $w_{td}=TF\times IDF$ | Importance of a term in a document | Use the stated TF and logarithm conventions. |
| Cosine similarity and distance | $s=\frac{x\cdot y}{\lVert x\rVert\lVert y\rVert}$; $d_{\cos}=1-s$ | Directional similarity and its corresponding distance | Use non-zero vectors in the same coordinate system. |
| Jaccard similarity and distance | $J=\frac{\lvert A\cap B\rvert}{\lvert A\cup B\rvert}$; $d_J=1-J$ | Overlap of two sets | Count each distinct term only once. |
| Precision, recall and F1 | $P=\frac{TP}{TP+FP}$; $R=\frac{TP}{TP+FN}$; $F_1=\frac{2PR}{P+R}$ | Retrieval correctness and completeness | Precision divides by retrieved items; recall divides by relevant items. |
| MAE and RMSE | $MAE=\frac{1}{n}\sum_j\lvert r_j-\hat{r}_j\rvert$; $RMSE=\sqrt{\frac{1}{n}\sum_j(r_j-\hat{r}_j)^2}$ | Rating-prediction errors | Square before averaging for RMSE. |
| Rocchio update | $q_{\text{new}}=\alpha q_0+\beta\bar{d}_r-\gamma\bar{d}_{nr}$ | Query movement towards relevant content and away from irrelevant content | Calculate the two document centroids first. |
| Matrix multiplication | $c_{ij}=\sum_{\ell=1}^{n}a_{i\ell}b_{\ell j}$ | Row-column dot products | $(m\times n)(n\times p)$ produces $m\times p$. |
| Sample covariance | $\operatorname{Cov}(X,Y)=\frac{\sum_j(x_j-\bar{x})(y_j-\bar{y})}{n-1}$ | Joint variation of paired values | Centre each variable around its own mean. |
| Covariance matrix | $\Sigma_{ij}=\operatorname{Cov}(X_i,X_j)$ | Variances and pairwise covariances together | Put variances on the diagonal; the matrix is symmetric. |
| Average user profile | $P_u=\frac{1}{m}\sum_{i=1}^{m}x_i$ | Equal-weight combination of liked items | Divide by the number of items, not features. |
| Weighted user profile | $P_u=\frac{\sum_iw_{ui}x_i}{\sum_i\lvert w_{ui}\rvert}$ | Preference-weighted feature profile | Multiply each whole vector by its weight. |
| Dynamic profile update | $P_{\text{next}}=(1-\alpha)P_{\text{old}}+\alpha x$ | Blend of earlier preferences and recent evidence | $\alpha$ is the new item's share. |
| User-based CF | $\hat{r}_{ui}=\frac{\sum_{v\in N_i(u)}s_{uv}r_{vi}}{\sum_{v\in N_i(u)}\lvert s_{uv}\rvert}$ | Rating from similar users | Use neighbours who rated the target item. |
| Mean-centred user-based CF | $\hat{r}_{ui}=\bar{r}_u+\frac{\sum_vs_{uv}(r_{vi}-\bar{r}_v)}{\sum_v\lvert s_{uv}\rvert}$ | Rating adjusted for generous or strict raters | Weight deviations, then add the target user's mean. |
| Item-based CF | $\hat{r}_{ui}=\frac{\sum_{j\in J_u(i)}s_{ij}r_{uj}}{\sum_{j\in J_u(i)}\lvert s_{ij}\rvert}$ | Rating from similar items already rated by the user | Compare item columns, not user rows. |
| Latent-factor score | $\hat{r}_{ui}=p_u^{T}q_i=\sum_fp_{uf}q_{if}$ | Match between user and item factors | Multiply matching coordinates and add. |

In CF, $s$ denotes similarity, $r$ a known rating, and $\hat{r}$ a prediction. $N_i(u)$ contains selected users who rated target item $i$; $J_u(i)$ contains selected similar items already rated by user $u$. All CF examples below use positive similarity weights. Pearson is needed here only conceptually: it compares rating patterns after removing each user's mean.

# Unit 1 Numericals

### 1. TF-IDF and Document Weights [HIGH]

#### Worked Example

**Question:** A collection contains 1,000 documents. A term appears in 10 documents and occurs 4 times in a document containing 10 terms. Calculate normalized TF, base-10 IDF and TF-IDF.

Let $c_{td}$ be the term count, $L_d$ the document length, $N$ the collection size, and $df_t$ the number of documents containing the term.

1. Normalize the term count so that TF expresses its share of this document:

$$
TF(t,d)=\frac{c_{td}}{L_d}=\frac{4}{10}=0.4
$$

2. Calculate how rare the term is across the collection:

$$
IDF(t)=\log_{10}\left(\frac{N}{df_t}\right)
=\log_{10}\left(\frac{1000}{10}\right)=2
$$

3. Combine local frequency with collection rarity:

$$
TFIDF(t,d)=0.4(2)=\boxed{0.8}
$$

**Exam tip:** Document frequency counts documents, not total occurrences. This method also constructs the TF-IDF document vectors used in Unit 2.

#### Practice Problems

- **TF1:** A term occurs 3 times in a 10-term document. The collection has $N=100$ and $df_t=10$. Find normalized TF, base-10 IDF and TF-IDF.
- **TF2:** In a 10-term document, “machine” occurs twice and “learning” occurs four times. For $N=100$, their document frequencies are 10 and 1 respectively. Construct the normalized, base-10 TF-IDF vector in the order $[\text{machine},\text{learning}]$.

### 2. Cosine Similarity and Distance [VERY HIGH]

#### Worked Example

**Question:** For $Q=[1,1,0]$ and $D=[1,2,1]$, calculate cosine similarity and cosine distance.

Use:

$$
s(Q,D)=\frac{Q\cdot D}{\lVert Q\rVert\lVert D\rVert},
\qquad d_{\cos}=1-s
$$

1. The dot product measures agreement between corresponding coordinates:

$$
Q\cdot D=1(1)+1(2)+0(1)=3
$$

2. Divide by both vector magnitudes to normalize their sizes:

$$
\lVert Q\rVert=\sqrt{1^2+1^2+0^2}=\sqrt{2},
\qquad
\lVert D\rVert=\sqrt{1^2+2^2+1^2}=\sqrt{6}
$$

3. Calculate similarity, then subtract it from 1 to obtain distance:

$$
s=\frac{3}{\sqrt{2}\sqrt{6}}\approx\boxed{0.866},
\qquad
d_{\cos}=1-\frac{3}{\sqrt{12}}\approx\boxed{0.134}
$$

**Exam tip:** High similarity means low distance. A zero-magnitude vector makes cosine undefined.

#### Practice Problems

- **COS1:** Find cosine similarity and distance between $[1,0,1]$ and $[1,1,0]$.
- **COS2:** Find cosine similarity and distance between $[3,4]$ and $[4,3]$.

### 3. Precision, Recall and F1 [HIGH]

#### Worked Example

**Question:** A system retrieves 50 documents, of which 40 are relevant. There are 100 relevant documents in the collection. Calculate precision, recall and F1.

$TP$ counts relevant retrieved documents, $FP$ counts irrelevant retrieved documents, and $FN$ counts relevant documents missed.

1. Identify the counts before choosing denominators:

$$
TP=40,\qquad FP=50-40=10,\qquad FN=100-40=60
$$

2. Precision measures correctness among retrieved results; recall measures coverage of all relevant results:

$$
P=\frac{TP}{TP+FP}=\frac{40}{50}=\boxed{0.8}
$$

$$
R=\frac{TP}{TP+FN}=\frac{40}{100}=\boxed{0.4}
$$

3. Use their harmonic mean for F1:

$$
F_1=\frac{2PR}{P+R}
=\frac{2(0.8)(0.4)}{0.8+0.4}
=\frac{8}{15}\approx\boxed{0.533}
$$

**Exam tip:** Recall does not use the total collection size; it uses the total relevant set.

#### Practice Problems

- **PR1:** Of 20 retrieved documents, 15 are relevant. There are 30 relevant documents overall. Calculate precision, recall and F1.
- **PR2:** A system has $TP=8$, $FP=2$ and $FN=12$. Calculate precision, recall and F1.

### 4. MAE and RMSE [HIGH]

#### Worked Example

The 2025 paper asks for offline metric definitions; the following calculation is a practice variation, not a past-paper numerical.

**Question:** Actual ratings are $[4,3,5]$ and predictions are $[3.5,2.5,4]$. Calculate MAE and RMSE.

For $n$ evaluated ratings, $r_j$ is actual and $\hat{r}_j$ is predicted:

$$
MAE=\frac{1}{n}\sum_{j=1}^{n}\lvert r_j-\hat{r}_j\rvert,
\qquad
RMSE=\sqrt{\frac{1}{n}\sum_{j=1}^{n}(r_j-\hat{r}_j)^2}
$$

1. Calculate each error before taking the appropriate average:

| Actual | Predicted | Absolute error | Squared error |
|---:|---:|---:|---:|
| 4 | 3.5 | 0.5 | 0.25 |
| 3 | 2.5 | 0.5 | 0.25 |
| 5 | 4 | 1 | 1 |

2. Average the absolute errors:

$$
MAE=\frac{0.5+0.5+1}{3}=\frac{2}{3}\approx\boxed{0.667}
$$

3. Average the squared errors and take the square root:

$$
RMSE=\sqrt{\frac{0.25+0.25+1}{3}}
=\sqrt{0.5}\approx\boxed{0.707}
$$

**Exam tip:** Square individual errors before averaging; do not square MAE.

#### Practice Problems

- **ERR1:** Actual ratings are $[5,3,1]$ and predictions are $[4,3,2]$. Calculate MAE and RMSE.
- **ERR2:** Actual ratings are $[4,2,5,3]$ and predictions are $[3,2,4,1]$. Calculate MAE and RMSE.

### 5. Rocchio Relevance-Feedback Update [HIGH]

#### Worked Example

**Question:** Given $q_0=[2,1]$, relevant documents $D_r=\{[4,3],[6,2]\}$ and non-relevant documents $D_{nr}=\{[1,5]\}$, find the updated query for $\alpha=1$, $\beta=0.75$ and $\gamma=0.25$.

Rocchio retains the original query, adds relevant evidence and subtracts non-relevant evidence:

$$
q_{\text{new}}=\alpha q_0+
\frac{\beta}{\lvert D_r\rvert}\sum_{d\in D_r}d
-\frac{\gamma}{\lvert D_{nr}\rvert}\sum_{d\in D_{nr}}d
$$

1. Find the centroid of each group so that document count does not silently change its weight:

$$
\bar{d}_r=\frac{[4,3]+[6,2]}{2}=[5,2.5],
\qquad \bar{d}_{nr}=[1,5]
$$

2. Weight the centroids and combine corresponding coordinates:

$$
q_{\text{new}}=[2,1]+0.75[5,2.5]-0.25[1,5]
$$

$$
q_{\text{new}}=[2,1]+[3.75,1.875]-[0.25,1.25]
=\boxed{[5.5,1.625]}
$$

**Exam tip:** Subtract the non-relevant contribution; do not add it.

#### Practice Problems

- **ROC1:** Use $q_0=[1,1]$, $D_r=\{[2,2],[4,2]\}$, $D_{nr}=\{[0,2]\}$ and $(\alpha,\beta,\gamma)=(1,0.5,0.25)$.
- **ROC2:** Use $q_0=[2,2]$, $D_r=\{[4,2],[2,4]\}$, $D_{nr}=\{[2,0],[0,2]\}$ and $(\alpha,\beta,\gamma)=(1,1,0.5)$.

### 6. Matrix Multiplication [HIGH]

#### Worked Example

**Question:** Calculate $AB$:

$$
A=\begin{bmatrix}1&2\\3&4\end{bmatrix},
\qquad B=\begin{bmatrix}5&6\\7&8\end{bmatrix}
$$

1. Check dimensions: $(2\times2)(2\times2)$ gives a $2\times2$ result.
2. Each entry follows $c_{ij}=\sum_{\ell}a_{i\ell}b_{\ell j}$: multiply one row of $A$ by one column of $B$ and add.

$$
AB=
\begin{bmatrix}
1(5)+2(7)&1(6)+2(8)\\
3(5)+4(7)&3(6)+4(8)
\end{bmatrix}
$$

$$
AB=\boxed{\begin{bmatrix}19&22\\43&50\end{bmatrix}}
$$

**Exam tip:** This is not entry-by-entry multiplication, and generally $AB\ne BA$.

#### Practice Problems

- **MAT1:** Find $AB$ and state its dimensions:

$$
A=\begin{bmatrix}1&0&2\\0&1&1\end{bmatrix},
\qquad
B=\begin{bmatrix}1&2\\3&0\\2&1\end{bmatrix}
$$

- **MAT2:** Find both $AB$ and $BA$:

$$
A=\begin{bmatrix}2&1\\1&3\end{bmatrix},
\qquad
B=\begin{bmatrix}1&2\\0&1\end{bmatrix}
$$

### 7. Sample Covariance [HIGH]

#### Worked Example

**Question:** Find the sample covariance of $X=[2,3,4,5]$ and $Y=[1,2,4,5]$.

For $n$ paired observations:

$$
\operatorname{Cov}(X,Y)=
\frac{1}{n-1}\sum_{j=1}^{n}(x_j-\bar{x})(y_j-\bar{y})
$$

1. Calculate the means to measure variation around each variable's own baseline:

$$
\bar{x}=\frac{14}{4}=3.5,\qquad \bar{y}=\frac{12}{4}=3
$$

2. Multiply paired deviations:

| $x_j-\bar{x}$ | $y_j-\bar{y}$ | Product |
|---:|---:|---:|
| $-1.5$ | $-2$ | 3 |
| $-0.5$ | $-1$ | 0.5 |
| $0.5$ | $1$ | 0.5 |
| $1.5$ | $2$ | 3 |

3. Add and divide by the sample denominator:

$$
\operatorname{Cov}(X,Y)=\frac{3+0.5+0.5+3}{4-1}
=\frac{7}{3}\approx\boxed{2.333}
$$

The positive value indicates that the two variables tend to rise together.

**Exam tip:** Preserve the pairing of observations. Use $n-1$ for a sample and $n$ only when population covariance is requested.

#### Practice Problems

- **COV1:** Calculate sample covariance for $X=[1,2,3]$ and $Y=[3,2,1]$.
- **COV2:** Calculate sample covariance for $X=[1,2,3]$ and $Y=[2,1,2]$.

### 8. Covariance Matrix [HIGH]

#### Worked Example

**Question:** Construct the sample covariance matrix for variables $X=[1,2,3]$ and $Y=[2,4,6]$, in that order.

The diagonal stores variances and the off-diagonal stores covariance:

$$
\Sigma=
\begin{bmatrix}
\operatorname{Var}(X)&\operatorname{Cov}(X,Y)\\
\operatorname{Cov}(Y,X)&\operatorname{Var}(Y)
\end{bmatrix}
$$

1. The means are $2$ and $4$, giving deviations $[-1,0,1]$ and $[-2,0,2]$.
2. Use squared deviations for variances and paired products for covariance:

$$
\operatorname{Var}(X)=\frac{(-1)^2+0^2+1^2}{2}=1
$$

$$
\operatorname{Var}(Y)=\frac{(-2)^2+0^2+2^2}{2}=4
$$

$$
\operatorname{Cov}(X,Y)=\frac{(-1)(-2)+0(0)+1(2)}{2}=2
$$

3. Place these values in the required order:

$$
\Sigma=\boxed{\begin{bmatrix}1&2\\2&4\end{bmatrix}}
$$

**Exam tip:** Every entry must use the same denominator. Off-diagonal entries must agree because covariance is symmetric.

#### Practice Problems

- **CM1:** Construct the sample covariance matrix, in $X,Y$ order, for $X=[1,2,3]$ and $Y=[3,2,1]$.
- **CM2:** Construct the sample covariance matrix, in $X,Y$ order, for $X=[1,2,3]$ and $Y=[1,3,2]$.

# Unit 2 Numericals

TF-IDF and numerical-vector cosine practice are in Unit 1; the same calculations apply to item and user profiles.

### 9. Document Jaccard and Cosine Distances [VERY HIGH]

#### Worked Example

**[Asked in 2025 Midsem] — Question 9, 4 marks**

“Measure the similarity of the two documents using Jaccard distance and cosine distance measures.”

- **Doc 1:** “Machine Learning is a subset of Artificial Intelligence”
- **Doc 2:** “Most Artificial Intelligence work now involves Machine Learning”

**Assumption:** Convert to lowercase and remove punctuation. Retain all words, including stop words; use binary word-presence vectors for cosine. The paper does not specify preprocessing. Report similarities as well as distances because its wording mentions both.

**Step 1: Identify distinct terms.**

| Group | Terms | Count |
|---|---|---:|
| Common to both | machine, learning, artificial, intelligence | 4 |
| Only in Doc 1 | is, a, subset, of | 4 |
| Only in Doc 2 | most, work, now, involves | 4 |

Each document has 8 distinct terms, and the union has $4+4+4=12$.

**Step 2: Calculate Jaccard similarity and distance.**

For term sets $D_1,D_2$:

$$
J=\frac{\lvert D_1\cap D_2\rvert}{\lvert D_1\cup D_2\rvert}
=\frac{4}{12}=\boxed{\frac{1}{3}\approx0.333}
$$

Distance measures dissimilarity, so subtract the overlap proportion from 1:

$$
d_J=1-J=\boxed{\frac{2}{3}\approx0.667}
$$

**Step 3: Calculate cosine similarity and distance.**

Use the union as the shared vocabulary. Each binary vector contains eight 1s, so each squared magnitude is 8. The dot product is 4 because exactly four coordinates are 1 in both vectors.

$$
s=\frac{x\cdot y}{\lVert x\rVert\lVert y\rVert}
=\frac{4}{\sqrt{8}\sqrt{8}}=\boxed{0.500}
$$

$$
d_{\cos}=1-s=1-0.5=\boxed{0.500}
$$

**Exam tip:** State preprocessing before calculating. Removing stop words changes this answer; repeated words affect count-vector cosine but not Jaccard sets.

#### Practice Problems

- **DOC1:** Find Jaccard similarity/distance and cosine similarity/distance for “machine learning recommends books” and “machine learning recommends movies.” Use lowercase binary word vectors and retain every word.
- **DOC2:** Find the same four quantities for “data data science” and “data science science.” Use distinct-term sets for Jaccard but **raw term-count vectors**, in $[\text{data},\text{science}]$ order, for cosine.

### 10. Average User Profile [HIGH]

#### Worked Example

**Question:** Construct the average profile from three liked items:

$$
x_1=[1,0.8,0.2],\quad x_2=[0.9,0.7,0.3],\quad x_3=[0.8,0.9,0.1]
$$

An average profile gives the $m$ liked items equal influence:

$$
P_u=\frac{1}{m}\sum_{i=1}^{m}x_i
$$

1. Add matching features:

$$
x_1+x_2+x_3=[1+0.9+0.8,\;0.8+0.7+0.9,\;0.2+0.3+0.1]
=[2.7,2.4,0.6]
$$

2. Divide each coordinate by the number of liked items:

$$
P_u=\frac{[2.7,2.4,0.6]}{3}=\boxed{[0.9,0.8,0.2]}
$$

The first feature has the highest average preference.

**Exam tip:** Divide by the number of items, even when the number of features happens to be different.

#### Practice Problems

- **AVG1:** Average the two liked-item vectors $[1,0,1]$ and $[0,1,1]$.
- **AVG2:** Average $[1,0.2,0]$, $[0.5,0.4,0.5]$ and $[0,0.6,1]$.

### 11. Weighted User Profile [HIGH]

#### Worked Example

**Question:** Use the three item vectors from the average-profile worked example, but assign weights $2,1,1$ respectively. Calculate the weighted profile.

For preference weights $w_{ui}$:

$$
P_u=\frac{\sum_iw_{ui}x_i}{\sum_i\lvert w_{ui}\rvert}
$$

1. Multiply each item vector by its weight:

$$
2x_1=[2,1.6,0.4],\quad x_2=[0.9,0.7,0.3],\quad x_3=[0.8,0.9,0.1]
$$

2. Add the weighted feature vectors and normalize by total weight:

$$
P_u=\frac{[2,1.6,0.4]+[0.9,0.7,0.3]+[0.8,0.9,0.1]}{2+1+1}
$$

$$
P_u=\frac{[3.7,3.2,0.8]}{4}=\boxed{[0.925,0.8,0.2]}
$$

The first feature gains influence because it is strongest in the item with double weight.

**Exam tip:** Normalize by total weight, not by the number of items. The practice weights here are all positive.

#### Practice Problems

- **WT1:** Vectors $[1,0,1]$ and $[0,1,1]$ have weights 3 and 1 respectively. Find the weighted profile.
- **WT2:** Vectors $[1,0,0]$, $[0,1,0]$ and $[0,0,1]$ have weights 1, 2 and 3 respectively. Find the weighted profile.

### 12. Dynamic User-Profile Update [HIGH]

#### Worked Example

**Question:** Update $P_{\text{old}}=[0.8,0.6,0.2]$ using a newly preferred item $x=[0.2,0.8,0.9]$ with $\alpha=0.3$.

$\alpha$ is the fraction of influence assigned to recent evidence:

$$
P_{\text{next}}=(1-\alpha)P_{\text{old}}+\alpha x
$$

1. Retain 70% of the old profile:

$$
0.7P_{\text{old}}=[0.56,0.42,0.14]
$$

2. Add 30% of the new item:

$$
0.3x=[0.06,0.24,0.27]
$$

$$
P_{\text{next}}=[0.56,0.42,0.14]+[0.06,0.24,0.27]
=\boxed{[0.62,0.66,0.41]}
$$

The third feature increases because it is strongly represented in the recent item.

**Exam tip:** The two coefficients must add to 1; do not assign $\alpha$ to both vectors.

#### Practice Problems

- **UP1:** Update $[0.8,0.2]$ using new item $[0.2,0.8]$ and $\alpha=0.25$.
- **UP2:** Update $[0.4,0.6,0.2]$ using new item $[1,0,0.8]$ and $\alpha=0.5$.

# Unit 3 Numericals

For user similarity, compare co-rated item entries. For item similarity, compare users who rated both items. Use only eligible neighbours in the prediction and follow the stated value of $k$.

### 13. User-User Cosine and Weighted Rating Prediction [VERY HIGH]

#### Worked Example

**[Asked in 2025 Midsem] — Question 8, 6 marks**

“Calculate the predicted rating of Alice for (Item5) which is missing using the given data matrix.”

| Name | Item1 | Item2 | Item3 | Item4 | Item5 |
|---|---:|---:|---:|---:|---:|
| Alice | 5 | 4 | 1 | 4 | ? |
| U1 | 3 | 1 | 2 | 3 | 3 |
| U2 | 4 | 3 | 4 | 3 | 5 |
| U3 | 3 | 3 | 1 | 4 | 5 |

**Assumption:** Use user-based CF, cosine similarity on Items 1–4, all three other users as neighbours, and raw weighted prediction. The paper does not specify a method or $k$.

**Step 1: Select common entries.**

Alice's Item5 rating is unknown, so it cannot enter a similarity vector:

$$
A=[5,4,1,4],\quad U_1=[3,1,2,3],\quad
U_2=[4,3,4,3],\quad U_3=[3,3,1,4]
$$

**Step 2: Calculate similarities.**

$$
s(A,U_v)=\frac{A\cdot U_v}{\lVert A\rVert\lVert U_v\rVert},
\qquad \lVert A\rVert^2=25+16+1+16=58
$$

| Neighbour | Dot product with Alice | Squared magnitude |
|---|---|---|
| U1 | $5(3)+4(1)+1(2)+4(3)=33$ | $9+1+4+9=23$ |
| U2 | $5(4)+4(3)+1(4)+4(3)=48$ | $16+9+16+9=50$ |
| U3 | $5(3)+4(3)+1(1)+4(4)=44$ | $9+9+1+16=35$ |

$$
s_1=\frac{33}{\sqrt{58}\sqrt{23}}\approx0.903516
$$

$$
s_2=\frac{48}{\sqrt{58}\sqrt{50}}\approx0.891338
$$

$$
s_3=\frac{44}{\sqrt{58}\sqrt{35}}\approx0.976573
$$

**Step 3: Select neighbours.**

Their similarity order is U3, U1, U2. All three have known Item5 ratings and are included under the stated assumption.

**Step 4: Normalize the weighted ratings.**

The prediction is:

$$
\hat{r}_{ui}=
\frac{\sum_{v\in N_i(u)}s(u,v)r_{vi}}
{\sum_{v\in N_i(u)}\lvert s(u,v)\rvert}
$$

Here, $N_i(u)$ contains eligible neighbours and $r_{vi}$ is their rating of target item $i$. Dividing by total similarity weight converts the weighted sum into a weighted average.

Using the unrounded similarities:

$$
\hat{r}_{A,5}=\frac{3s_1+5s_2+5s_3}{s_1+s_2+s_3}
\approx\frac{12.0501016}{2.7714269}
\approx\boxed{4.348}
$$

This is approximately **4.35** to two decimal places and is within the 1–5 rating scale.

**Exam tip:** State the method and $k$. Different valid methods can give different results; do not round similarity weights too early.

#### Practice Problems

- **UB1:** Similarities to V1, V2 and V3 are $0.9,0.6,0.3$, and their target-item ratings are $5,3,1$. Select the **two most similar neighbours** and calculate the raw weighted prediction.
- **UB2:** Predict T's rating of Z using cosine similarity and **all other users with a known Z rating**. Report similarities to V1, V2 and V3, identify the eligible neighbours, and give the raw weighted prediction.

| User | X | Y | Z |
|---|---:|---:|---:|
| T | 5 | 1 | ? |
| V1 | 5 | 1 | 4 |
| V2 | 1 | 5 | 2 |
| V3 | 5 | 1 | ? |

### 14. Mean-Centred Rating Prediction [MEDIUM]

#### Worked Example

**Question:** The target user's mean is $3.5$. Two neighbours have similarities $0.9,0.7$, means $3,4$, and target-item ratings $5,4$ respectively. Find the mean-centred prediction.

Use neighbour deviations to adjust for rating habits:

$$
\hat{r}_{ui}=\bar{r}_u+
\frac{\sum_vs_{uv}(r_{vi}-\bar{r}_v)}{\sum_v\lvert s_{uv}\rvert}
$$

1. Centre the neighbours' ratings: $5-3=2$ and $4-4=0$.
2. Calculate the weighted deviation and add the target user's baseline:

$$
\hat{r}_{ui}=3.5+\frac{0.9(2)+0.7(0)}{0.9+0.7}
=3.5+\frac{1.8}{1.6}
=\boxed{4.625}
$$

**Exam tip:** Subtract each neighbour's own mean, then add the target user's mean. Similarities are supplied here; no Pearson calculation is needed.

#### Practice Problems

- **MC1:** Target mean $=3$; neighbour similarities $=[0.8,0.4]$; neighbour means $=[3,2]$; target-item ratings $=[5,3]$. Predict the rating.
- **MC2:** Target mean $=4$; similarities $=[0.6,0.4]$; neighbour means $=[4,3]$; target-item ratings $=[2,4]$. Predict the rating.

### 15. Item-Item Cosine and Item-Based Prediction [VERY HIGH]

#### Worked Example

**Question:** Predict T's rating of C using item-item cosine similarity and both previously rated items A and B.

| User | A | B | C |
|---|---:|---:|---:|
| T | 5 | 3 | ? |
| V1 | 5 | 1 | 5 |
| V2 | 1 | 5 | 1 |

**Step 1: Compare item columns over common users.**

T has no C rating, so use V1 and V2 for both comparisons:

$$
C=[5,1],\quad A=[5,1],\quad B=[1,5]
$$

Each squared magnitude is $26$. Therefore:

$$
s(C,A)=\frac{5(5)+1(1)}{\sqrt{26}\sqrt{26}}
=\frac{26}{26}=1
$$

$$
s(C,B)=\frac{5(1)+1(5)}{\sqrt{26}\sqrt{26}}
=\frac{10}{26}=\frac{5}{13}
$$

**Step 2: Select the target user's known ratings.**

T rated A as 5 and B as 3. These supply the rating values; the similarities above supply the weights.

**Step 3: Calculate the normalized prediction.**

$$
\hat{r}_{T,C}=
\frac{s(C,A)r_{T,A}+s(C,B)r_{T,B}}
{\lvert s(C,A)\rvert+\lvert s(C,B)\rvert}
$$

$$
\hat{r}_{T,C}=
\frac{1(5)+\frac{5}{13}(3)}{1+\frac{5}{13}}
=\frac{80}{18}
\approx\boxed{4.444}
$$

Dividing by total similarity weight keeps this positive-weight prediction between T's two known ratings.

**Exam tip:** Item similarity comes from other users' ratings; the prediction uses the target user's ratings of those similar items.

#### Practice Problems

- **IB1:** T rated A as 5 and B as 4. Given $s(C,A)=0.8$ and $s(C,B)=0.6$, predict T's C rating using both items.
- **IB2:** Compute similarities of C to A and B using common users, then predict T's C rating using both items.

| User | A | B | C |
|---|---:|---:|---:|
| T | 4 | 2 | ? |
| V1 | 3 | 4 | 3 |
| V2 | 4 | 3 | 4 |

### 16. Latent-Factor Dot Product [HIGH]

#### Worked Example

**Question:** A user has latent vector $p_u=[0.9,0.8]$ and an item has $q_i=[0.3,0.9]$. Calculate their preference score.

The factors must have the same length. Their dot product measures the match:

$$
\hat{r}_{ui}=p_u^{T}q_i=\sum_{f=1}^{k}p_{uf}q_{if}
$$

Multiply corresponding factor coordinates, then add:

$$
\hat{r}_{ui}=0.9(0.3)+0.8(0.9)
=0.27+0.72=\boxed{0.99}
$$

**Exam tip:** This is an illustrative latent score, not necessarily a calibrated 1–5 rating. Do not add biases or invent a rescaling.

#### Practice Problems

- **LF1:** Find the score for $p=[0.6,0.8]$ and $q=[0.5,0.5]$.
- **LF2:** For $p=[0.2,0.5,0.3]$, compute scores for $q_A=[1,0.4,0]$ and $q_B=[0,0.8,1]$. Which item ranks first?

# Practice Answer Key

Final answers only; all conventions are those stated in the questions.

| ID | Final answer |
|---|---|
| TF1 | $TF=0.3$, $IDF=1$, $TFIDF=0.3$ |
| TF2 | $[0.2,0.8]$ |
| COS1 | Similarity $=0.500$; distance $=0.500$ |
| COS2 | Similarity $=0.960$; distance $=0.040$ |
| PR1 | $P=0.750$, $R=0.500$, $F_1=0.600$ |
| PR2 | $P=0.800$, $R=0.400$, $F_1\approx0.533$ |
| ERR1 | $MAE\approx0.667$; $RMSE\approx0.816$ |
| ERR2 | $MAE=1$; $RMSE\approx1.225$ |
| ROC1 | $[2.5,1.5]$ |
| ROC2 | $[4.5,4.5]$ |
| MAT1 | $\begin{bmatrix}5&4\\5&1\end{bmatrix}$; $2\times2$ |
| MAT2 | $AB=\begin{bmatrix}2&5\\1&5\end{bmatrix}$; $BA=\begin{bmatrix}4&7\\1&3\end{bmatrix}$ |
| COV1 | $-1$ |
| COV2 | $0$ |
| CM1 | $\begin{bmatrix}1&-1\\-1&1\end{bmatrix}$ |
| CM2 | $\begin{bmatrix}1&0.5\\0.5&1\end{bmatrix}$ |
| DOC1 | $J=0.600$, $d_J=0.400$; cosine $=0.750$, $d_{\cos}=0.250$ |
| DOC2 | $J=1$, $d_J=0$; count-vector cosine $=0.800$, $d_{\cos}=0.200$ |
| AVG1 | $[0.5,0.5,1]$ |
| AVG2 | $[0.5,0.4,0.5]$ |
| WT1 | $[0.75,0.25,1]$ |
| WT2 | $[1/6,1/3,1/2]\approx[0.167,0.333,0.500]$ |
| UP1 | $[0.65,0.35]$ |
| UP2 | $[0.7,0.3,0.5]$ |
| UB1 | V1 and V2; $\hat{r}=4.200$ |
| UB2 | Similarities to V1, V2, V3: $1,5/13,1$; eligible neighbours: V1 and V2; $\hat{r}=31/9\approx3.444$ |
| MC1 | $\hat{r}=14/3\approx4.667$ |
| MC2 | $\hat{r}=3.200$ |
| IB1 | $\hat{r}=32/7\approx4.571$ |
| IB2 | $s(C,A)=1$, $s(C,B)=24/25=0.960$; $\hat{r}=148/49\approx3.020$ |
| LF1 | $0.700$ |
| LF2 | A $=0.400$; B $=0.700$; B ranks first |
