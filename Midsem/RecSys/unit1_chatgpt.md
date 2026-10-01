# Chat-GPT Rec Sys UNIT_1 prep

# Unit 1 — Introduction to Recommender Systems

## 1. What is a Recommender System?

A **Recommender System (RS)** is a system that studies users, items and their interactions to predict which items a particular user is likely to prefer. It then ranks those items and recommends the most suitable ones.

The basic problem it solves is simple: online platforms usually contain far more options than a user can reasonably examine.

Two related problems arise:

- **Information overload:** there is more information than a person can effectively process.
- **Choice overload:** too many alternatives make comparison and decision-making difficult.

A recommender system therefore tries to **reduce the search space while still showing relevant items**.

For a user $u$ and item $i$, we can represent the predicted preference as:

$$
\hat{r}_{ui} = f(u,i,D)
$$

where:

- $u$ = user
- $i$ = item
- $D$ = available user/item/interaction data
- $\hat{r}_{ui}$ = predicted preference score

The system calculates such scores for candidate items, ranks them and returns the best ones.

$$
\text{Prediction} \rightarrow \text{Ranking} \rightarrow \text{Recommendation}
$$

### Personalization

**Personalization** means adapting recommendations to a particular user’s interests, behaviour or context.

For example, the same online bookstore may recommend machine-learning books to one user and cybersecurity books to another because their past behaviour differs.

---

## 2. Information Used by a Recommender System

A recommendation can be based on several types of information.

| Information | Meaning | Examples |
| --- | --- | --- |
| **User information** | Information about the user’s interests or history | Ratings, purchases, searches, watch history |
| **Item information** | Properties describing an item | Genre, tags, description, category, price |
| **Context information** | Situation in which the recommendation is made | Time, location, device, season |

### Explicit vs Implicit Feedback

**Explicit feedback** is directly provided by the user.

Examples include:

- star ratings
- likes/dislikes
- reviews
- favourites

It is usually clear and reliable, but users may not provide enough of it.

**Implicit feedback** is inferred from user behaviour.

Examples include:

- clicks
- purchases
- searches
- watch time
- browsing history

Implicit feedback is easier to collect in large quantities, but it can be ambiguous. A user clicking an item does not necessarily mean that they liked it.

### Basic Recommendation Pipeline

A typical recommender follows this flow:

1. Collect user, item and interaction data.
2. Represent the user’s preferences.
3. Generate candidate items.
4. Estimate scores for those candidates.
5. Rank them.
6. Return the top-$k$ recommendations.
7. Observe new user behaviour and update the system.

This creates a feedback loop:

$$
\text{Interaction} \rightarrow \text{Recommendation} \rightarrow \text{Response} \rightarrow \text{Learning}
$$

At a high level, the main recommendation approaches are:

- **Content-based filtering:** recommends items similar to what the user liked earlier.
- **Collaborative filtering:** uses behaviour of multiple users to discover useful recommendations.
- **Hybrid systems:** combine multiple recommendation approaches.

The algorithms themselves belong to later units.

---

## 3. Functions and Applications of Recommender Systems

A recommender system does more than simply predict ratings.

Its main functions include:

- **Prediction:** estimate the user’s preference for an item.
- **Filtering:** remove unsuitable candidates.
- **Ranking:** arrange candidates according to usefulness.
- **Personalization:** adapt results to an individual.
- **Discovery:** expose users to useful items they may not have found themselves.
- **Decision support:** reduce the effort required to choose between many alternatives.

### Applications

Recommender systems are widely used in:

- **E-commerce:** products and accessories
- **Entertainment:** movies, music and videos
- **News:** articles and topics
- **Education:** courses and books
- **Research:** papers and researchers
- **Social media:** posts, communities and users
- **Travel and food:** hotels, destinations and restaurants

### Qualities of a Good Recommender

A useful recommendation system should consider more than raw prediction accuracy.

**Relevance** means the recommendations should match the user’s needs.

**Diversity** means the list should contain sufficiently different items rather than many nearly identical choices.

**Novelty** means recommending useful items the user may not already know.

**Serendipity** means producing useful recommendations that are also pleasantly unexpected.

**Coverage** describes how much of the available item catalogue the system can meaningfully recommend.

**Explainability** means the system can provide understandable reasons for its recommendations.

These properties can conflict. A highly accurate system may still produce repetitive recommendations with poor diversity.

---

## 4. Information Retrieval and Recommender Systems

### What is Information Retrieval?

**Information Retrieval (IR)** is the process of finding relevant information from a large collection according to a user’s information need.

An **information need** is what the user actually wants.

A **query** is the expression submitted to the system to represent that need.

These are not always identical. For example, the query:

> `java`
> 

could represent a need for information about the Java programming language, Java island, or something else.

### Information Retrieval vs Recommender System

| Information Retrieval | Recommender System |
| --- | --- |
| Usually starts with a query | Usually starts from preferences/history |
| User actively searches | System may recommend proactively |
| Query is central | User and interaction data are central |
| Retrieves documents relevant to query | Predicts items useful to a user |

Real systems can combine both.

For example:

$$
\text{Search query} \rightarrow \text{candidate products} \rightarrow \text{personalized ranking}
$$

### Basic IR Architecture

On the document side:

$$
\text{Documents} \rightarrow \text{Preprocessing} \rightarrow \text{Index}
$$

On the query side:

$$
\text{Query} \rightarrow \text{Query Processing} \rightarrow \text{Retrieval} \rightarrow \text{Ranking}
$$

An **inverted index** stores, for every term, the documents in which that term occurs. This avoids scanning the entire document collection for every search.

---

## 5. Retrieval Models

A **retrieval model** defines how queries and documents are represented and how relevance between them is determined.

The main models in this unit are:

| Model | Main idea |
| --- | --- |
| **Boolean Model** | Exact matching using AND, OR and NOT |
| **Vector Space Model** | Query and documents represented as vectors and ranked using similarity |
| **Probabilistic Model** | Ranks documents according to estimated probability of relevance |
| **BM25** | Ranked retrieval using term importance, term frequency and document length |

### 5.1 Boolean Retrieval

Suppose:

- $A$ = documents containing “recommender”
- $B$ = documents containing “system”

Then:

$$
A \text{ AND } B = A \cap B
$$

returns documents containing both terms.

$$
A \text{ OR } B = A \cup B
$$

returns documents containing either term.

$$
A \text{ NOT } B = A - B
$$

returns documents containing $A$ but not $B$.

The main weakness is that basic Boolean retrieval performs **exact matching rather than natural relevance ranking**.

---

### 5.2 TF-IDF

TF-IDF is used to measure how important a term is to a document.

#### Term Frequency

$$
TF(t,d)=\frac{\text{count of }t\text{ in document }d}{\text{total terms in }d}
$$

A larger TF means the term appears frequently in that document.

#### Inverse Document Frequency

$$
IDF(t)=\log_{10}\left(\frac{N}{df_t}\right)
$$

where:

- $N$ = total number of documents
- $df_t$ = number of documents containing term $t$

A term appearing in almost every document has low IDF because it does not help distinguish documents.

Finally:

$$
TFIDF(t,d)=TF(t,d)\times IDF(t)
$$

#### Numerical

Suppose:

$$
N=1000,\qquad df_t=10,\qquad TF=0.4
$$

Then:

$$
IDF=\log_{10}(1000/10)=\log_{10}(100)=2
$$

Therefore:

$$
TFIDF=0.4\times2=0.8
$$

**Exam point:** if a question specifies a logarithm convention, use it consistently.

---

### 5.3 Cosine Similarity

Cosine similarity measures the similarity in direction between two vectors.

$$
\operatorname{cosine}(Q,D)=\frac{Q\cdot D}{\lVert Q\rVert\lVert D\rVert}
$$

For:

$$
Q=[1,1,0]
$$

and

$$
D=[1,2,1]
$$

the dot product is:

$$
Q\cdot D=1(1)+1(2)+0(1)=3
$$

Magnitudes:

$$
\lVert Q\rVert=\sqrt{2}
$$

$$
\lVert D\rVert=\sqrt{6}
$$

Therefore:

$$
\operatorname{cosine}(Q,D)=\frac{3}{\sqrt{2}\sqrt{6}}\approx0.866
$$

A value closer to $1$ means greater similarity.

---

### 5.4 Probabilistic Retrieval

The probabilistic model ranks a document using the probability that it is relevant to the query:

$$
P(R=1\mid d,q)
$$

Documents with greater estimated probability of relevance are ranked higher.

For the midsem, understand the **idea of probability-based ranking** rather than going deeply into probability estimation.

---

### 5.5 BM25

BM25 is a classical ranked-retrieval model.

It considers:

- importance of a query term
- frequency of that term in the document
- document length
- saturation of repeated term frequency

The score is:

$$
Score(D,Q)=\sum_{t\in Q} IDF(t)
\frac{f(t,D)(k_1+1)}
{f(t,D)+k_1\left(1-b+b\frac{|D|}{avgdl}\right)}
$$

where:

- $f(t,D)$ = frequency of term $t$ in document $D$
- $|D|$ = document length
- $avgdl$ = average document length
- $k_1$ = controls term-frequency saturation
- $b$ = controls document-length normalization

For this unit, focus primarily on **what BM25 does and what the terms mean**. The supplied material does not contain a complete BM25 numerical.

---

## 6. Search, Filtering and Ranking

These three operations should not be confused.

### Search or Retrieval

Finds items that match the user’s query.

### Filtering

Removes items that violate requirements or are unsuitable.

### Ranking

Orders the remaining candidates according to expected usefulness.

A typical pipeline is:

$$
\text{Query} \rightarrow \text{Candidate Retrieval} \rightarrow \text{Filtering} \rightarrow \text{Ranking} \rightarrow \text{Top-}k
$$

Common search techniques include:

- **Keyword search:** matches supplied terms.
- **Boolean search:** combines conditions using AND, OR and NOT.
- **Phrase search:** searches for an exact word sequence.
- **Wildcard search:** matches word patterns.
- **Fuzzy search:** allows approximate matches.
- **Field search:** searches only a selected field such as title.
- **Range search:** restricts numerical/date values to a range.
- **Faceted search:** lets users progressively narrow results using properties such as category or price.

Here, **filtering** means reducing candidate results. It should not be confused with **content-based filtering** or **collaborative filtering**, which are recommendation approaches.

---

## 7. Precision, Recall and F1-Score

These metrics are useful for understanding retrieval quality.

Let:

- $A$ = retrieved documents
- $R$ = relevant documents

Then:

$$
Precision=\frac{|A\cap R|}{|A|}
$$

Precision answers:

> Of everything retrieved, how much was actually relevant?
> 

$$
Recall=\frac{|A\cap R|}{|R|}
$$

Recall answers:

> Of everything relevant, how much did we successfully retrieve?
> 

F1-score balances the two:

$$
F_1=\frac{2PR}{P+R}
$$

### Numerical

Suppose:

- 50 documents were retrieved
- 40 retrieved documents were relevant
- 100 relevant documents existed in total

Then:

$$
Precision=\frac{40}{50}=0.8
$$

$$
Recall=\frac{40}{100}=0.4
$$

$$
F_1=\frac{2(0.8)(0.4)}{0.8+0.4}\approx0.533
$$

High precision means fewer irrelevant results.

High recall means fewer relevant results were missed.

---

## 8. Relevance Feedback

**Relevance feedback** improves future results using information about how the user reacted to previous results.

$$
\text{Result} \rightarrow \text{Feedback} \rightarrow \text{Updated Representation} \rightarrow \text{Improved Result}
$$

Feedback may be:

- **Explicit:** ratings, likes, dislikes or relevance judgements.
- **Implicit:** clicks, purchases, watch time or dwell time.
- **Positive:** evidence of preference.
- **Negative:** evidence of rejection.
- **Hybrid:** combination of explicit and implicit signals.

A weighted combination may be written as:

$$
F=\alpha E+(1-\alpha)I
$$

where $E$ is explicit feedback and $I$ is implicit feedback.

The two should be represented on compatible scales before combining them.

### Rocchio Relevance Feedback

The Rocchio method updates a query or preference vector by moving it:

- towards relevant items
- away from non-relevant items

$$
q_{new}
=
\alpha q_0
+
\frac{\beta}{|D_r|}\sum_{d\in D_r}d
-
\frac{\gamma}{|D_{nr}|}\sum_{d\in D_{nr}}d
$$

where:

- $q_0$ = original query/profile vector
- $D_r$ = relevant items
- $D_{nr}$ = non-relevant items
- $\alpha$ = importance of original vector
- $\beta$ = influence of relevant items
- $\gamma$ = influence of non-relevant items

### Numerical

Given:

$$
q_0=(2,1)
$$

$$
D_r=\{(4,3),(6,2)\}
$$

$$
D_{nr}=\{(1,5)\}
$$

and:

$$
\alpha=1,\qquad \beta=0.75,\qquad \gamma=0.25
$$

Average relevant vector:

$$
\frac{(4,3)+(6,2)}{2}=(5,2.5)
$$

Therefore:

$$
q_{new}=(2,1)+0.75(5,2.5)-0.25(1,5)
$$

$$
q_{new}=(5.5,1.625)
$$

The updated representation is now closer to the characteristics of relevant documents/items and farther from non-relevant ones.

---

## 9. User Profiles

A **user profile** is a representation of information used to describe a user’s interests or behaviour.

It allows the recommender to convert previous activity into information that can be used for future recommendations.

### Common Types

**Demographic profile:** age, location, language, occupation, etc.

**Behavioural profile:** clicks, searches, purchases and watched items.

**Preference profile:** ratings, likes, dislikes and explicitly stated interests.

**Vector profile:** represents interest numerically.

For example:

$$
P_u=(0.8,0.2,0.6)
$$

could describe the user’s strengths of preference for three predefined features.

### Updating a Profile

Because preferences change, profiles can be updated over time:

$$
P_{updated}=(1-\lambda)P_{old}+\lambda P_{new}
$$

A larger $\lambda$ gives more importance to recent behaviour.

### Numerical

Suppose:

$$
P_{old}=(0.7,0.3,0.5)
$$

$$
P_{new}=(0.9,0.8,0.2)
$$

and:

$$
\lambda=0.2
$$

Then:

$$
P_{updated}=0.8(0.7,0.3,0.5)+0.2(0.9,0.8,0.2)
$$

$$
P_{updated}=(0.74,0.40,0.44)
$$

### Static vs Dynamic Profile

A **static profile** changes rarely.

A **dynamic profile** changes as new user behaviour becomes available.

### Long-Term vs Short-Term Profile

A **long-term profile** represents stable interests.

A **short-term profile** represents recent or session-specific interests.

A system can combine both.

---

## 10. Ratings and the User-Item Matrix

Ratings represent the relationship between users and items.

For user $u$ and item $i$:

$$
r_{ui}
$$

represents an observed rating or interaction.

### Types of Ratings or Signals

| Type | Meaning | Example |
| --- | --- | --- |
| Explicit | Direct preference | 4/5 stars |
| Implicit | Behavioural evidence | Purchase or watch time |
| Binary | Two possible states | clicked/not clicked |
| Ordinal | Ordered categories | poor, fair, good, excellent |
| Continuous | Real-valued behavioural measure | minutes watched |

### User-Item Matrix

A rating matrix places:

- users in rows
- items in columns

Example:

$$
R=
\begin{bmatrix}
5 & 4 & 0 & 2 & 0 \\
4 & 5 & 3 & 0 & 1 \\
1 & 2 & 0 & 5 & 4 \\
0 & 1 & 4 & 5 & 5
\end{bmatrix}
$$

For example:

$$
R_{12}=4
$$

means user $U_1$ rated item $I_2$ as 4.

### Important Exam Point

In such examples, a `0` may be used simply to represent a **missing or unobserved rating**.

It must not automatically be interpreted as an actual rating of zero.

---

## 11. Matrix Operations Used in Recommendation

Matrix operations are important because recommendation data is often represented as vectors and matrices.

### Addition and Subtraction

Matrices must have the same dimensions, and corresponding elements are added or subtracted.

### Scalar Multiplication

Every matrix element is multiplied by the scalar.

### Transpose

Rows become columns.

If:

$$
A\in\mathbb{R}^{m\times n}
$$

then:

$$
A^T\in\mathbb{R}^{n\times m}
$$

### Dot Product

For vectors $x$ and $y$:

$$
x\cdot y=\sum_i x_i y_i
$$

Dot products are widely used in similarity calculations and preference scoring.

### Matrix Multiplication

If:

$$
A_{m\times n}B_{n\times p}
$$

then the result has dimension:

$$
C_{m\times p}
$$

The **inner dimensions must match**.

### Numerical

$$
A=
\begin{bmatrix}
1 & 2 \\
3 & 4
\end{bmatrix},
\qquad
B=
\begin{bmatrix}
5 & 6 \\
7 & 8
\end{bmatrix}
$$

Then:

$$
AB=
\begin{bmatrix}
1(5)+2(7) & 1(6)+2(8) \\
3(5)+4(7) & 3(6)+4(8)
\end{bmatrix}
$$

$$
AB=
\begin{bmatrix}
19 & 22 \\
43 & 50
\end{bmatrix}
$$

---

## 12. Covariance and Covariance Matrix

### Covariance

Covariance measures whether two variables tend to change together.

For sample data:

$$
\operatorname{Cov}(X,Y)
=
\frac{1}{n-1}
\sum_{i=1}^{n}
(x_i-\bar{x})(y_i-\bar{y})
$$

Interpretation:

- $\operatorname{Cov}(X,Y)>0$: variables tend to increase together.
- $\operatorname{Cov}(X,Y)<0$: when one increases, the other tends to decrease.
- $\operatorname{Cov}(X,Y)\approx0$: little linear co-variation.

### Numerical

Take:

$$
X=[2,3,4,5]
$$

$$
Y=[1,2,4,5]
$$

Means:

$$
\bar{X}=3.5,\qquad \bar{Y}=3
$$

Deviations:

$$
X-\bar{X}=[-1.5,-0.5,0.5,1.5]
$$

$$
Y-\bar{Y}=[-2,-1,1,2]
$$

Products:

$$
[3,0.5,0.5,3]
$$

Their sum is:

$$
7
$$

Therefore:

$$
\operatorname{Cov}(X,Y)=\frac{7}{4-1}=2.333
$$

The positive covariance means the two variables tend to move in the same direction.

### Covariance Matrix

For variables $X_1,X_2,\ldots,X_p$, all pairwise covariances can be stored in a matrix:

$$
\Sigma=
\begin{bmatrix}
\operatorname{Var}(X_1) & \operatorname{Cov}(X_1,X_2) & \cdots \\
\operatorname{Cov}(X_2,X_1) & \operatorname{Var}(X_2) & \cdots \\
\vdots & \vdots & \ddots
\end{bmatrix}
$$

Important properties:

- diagonal entries are **variances**
- off-diagonal entries are **covariances**
- the matrix is symmetric:

$$
\Sigma=\Sigma^T
$$

In recommendation problems, covariance can describe how different ratings or features vary together.

The supplied material mentions PCA as an application of covariance matrices, but **detailed PCA is not necessary for this Unit 1 midsem scope**.

---

## 13. Correlation and Pearson Similarity

Covariance tells us the direction of joint variation, but its magnitude depends on the measurement scale.

Correlation normalizes covariance:

$$
\rho_{XY}
=
\frac{\operatorname{Cov}(X,Y)}
{\sigma_X\sigma_Y}
$$

with:

- 1\leq\rho_{XY}\leq1

| Covariance | Correlation |
| --- | --- |
| Depends on scale | Scale-independent |
| Unbounded | Lies between -1 and 1 |
| Shows joint direction | Shows standardized direction and strength |

### Pearson Similarity

In recommender systems, Pearson similarity can compare two users using items they both rated.

$$
sim(u,v)=
\frac{
\sum_{i\in I_{uv}}
(r_{ui}-\bar{r}_u)(r_{vi}-\bar{r}_v)
}{
\sqrt{\sum_{i\in I_{uv}}(r_{ui}-\bar{r}_u)^2}
\sqrt{\sum_{i\in I_{uv}}(r_{vi}-\bar{r}_v)^2}
}
$$

where $I_{uv}$ is the set of items rated by both users.

Pearson is useful when users use rating scales differently because it compares ratings relative to each user’s mean.

For numericals, be careful to use the **commonly rated items** rather than treating missing entries as actual zeros.

---

## 14. Major Issues in Recommender Systems

### 14.1 Data Sparsity

In real systems, most users interact with only a small fraction of available items.

Therefore, the user-item matrix contains many missing values.

$$
Sparsity
=
1-
\frac{\text{observed interactions}}
{\text{users}\times\text{items}}
$$

### Example

For:

- 1,000 users
- 10,000 items
- 100,000 observed interactions

total possible interactions:

$$
1000\times10000=10,000,000
$$

Therefore:

$$
Sparsity
=
1-\frac{100000}{10000000}
=
0.99
=
99\%
$$

High sparsity makes similarity and prediction less reliable.

### 14.2 Cold Start

**Cold start** occurs when insufficient information exists for useful recommendations.

#### New-user cold start

There is little information about the user’s interests.

#### New-item cold start

Very few users have interacted with the new item.

#### New-system cold start

The entire system initially contains little interaction data.

Initial preference questions, item features and gradually collected implicit feedback can reduce the problem.

### 14.3 Scalability

As the number of users, items and interactions increases, storing and processing recommendation data becomes more expensive.

For the midsem, understand **the scalability problem itself** rather than implementation details.

### 14.4 Bias and Related Problems

#### Rating Bias

Some users consistently give higher or lower ratings than others.

#### Popularity Bias

Popular items receive more exposure, creating more interactions and making them even more likely to be recommended.

#### Long-Tail Problem

Less-popular items may receive little exposure even though they may be useful to some users.

#### Filter Bubble

Repeated personalization can keep showing users similar content and narrow their exposure to different ideas or items.

### 14.5 Manipulation Attacks

In a **shilling or profile-injection attack**, fake users or ratings are introduced to artificially promote or demote an item.

A simple chain is:

$$
\text{Fake ratings} \rightarrow \text{Artificial similarity} \rightarrow \text{Manipulated recommendation}
$$

Detailed attack methods belong mainly to Unit 3.

### 14.6 Privacy, Fairness and Explainability

**Privacy** concerns the handling of users’ personal behaviour and preference data.

**Fairness** concerns whether users, creators or item groups are treated unjustifiably differently.

**Explainability** means users can understand why an item was recommended, for example:

> Recommended because you liked similar movies.
> 

These problems show why a recommender cannot be judged only by prediction accuracy.

---

## 15. High-Value Midsem Comparisons

These are worth being able to explain directly in an exam.

| Comparison | Main difference |
| --- | --- |
| **IR vs RS** | Query-driven retrieval vs personalized recommendation |
| **Explicit vs implicit feedback** | Direct opinion vs inferred behaviour |
| **Search vs filtering vs ranking** | Find candidates vs remove unsuitable items vs order them |
| **Precision vs recall** | Correctness of retrieved set vs coverage of relevant set |
| **Static vs dynamic profile** | Rarely changes vs continuously updated |
| **Long-term vs short-term profile** | Stable preferences vs current/recent interests |
| **Covariance vs correlation** | Scale-dependent variation vs normalized relationship |
| **Sparsity vs cold start** | Missing interactions throughout the matrix vs insufficient data for something new |

## Unit 1 Midsem Priority

If you are revising quickly, prioritize these in this order:

1. **What recommender systems are, their functions and feedback**
2. **Information Retrieval vs Recommender Systems**
3. **Retrieval models**
4. **TF-IDF and cosine similarity numericals**
5. **Search vs filtering vs ranking**
6. **Relevance feedback and Rocchio numerical**
7. **User profiles and profile updating**
8. **Ratings and the user-item matrix**
9. **Matrix operations**
10. **Covariance, covariance matrix and Pearson correlation**
11. **Sparsity, cold start, bias and other major RS issues**
12. **Precision, recall and F1**
13. **BM25 interpretation**