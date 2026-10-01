# Codex Rec Sys UNIT_1 prep

# Unit 1 — Recommender-System Foundations

## 1. What is a recommender system?

A **recommender system (RS)** is a software system that studies users, items, and their interactions to predict and rank items that may be useful to a particular user. Its practical purpose is to answer: **“What should this user choose?”**

Recommender systems are needed because online platforms contain far more choices than a person can examine. This creates:

- **Information overload:** the amount or rate of information exceeds the user’s ability to process it.
- **Choice overload:** having too many alternatives makes comparison and decision-making difficult.

The goal is therefore not merely to show fewer items. It is to **reduce the search space while preserving relevance**.

### Formal view

Let $U$ be the set of users and $I$ the set of items. A model estimates user $u$’s preference for item $i$:

$$
\hat r_{ui}=f(u,i,D)
$$

where $D$ is the available data and $\hat r_{ui}$ is the predicted preference score. The system sorts candidate items by this score and returns the best $k$ items:

$$
TopK(u)=\operatorname{arg\,top}_k\{\hat r_{ui}:i\in I\}
$$

Thus, a recommender performs three connected tasks:

$$
\text{Preference prediction} \rightarrow \text{Ranking} \rightarrow \text{Recommendation}
$$

### Personalization

**Personalization** means adapting results to an individual user’s preferences, behaviour, characteristics, or context. Two users can therefore receive different rankings for the same set of items.

For example, a user interested in machine learning may receive data-mining books first, while a user interested in cybersecurity may receive network-security books first.

## 2. Information used by a recommender

| Information | What it describes | Examples |
| --- | --- | --- |
| User information | The user’s interests and history | Ratings, purchases, searches, watch history |
| Item information | Properties of an item | Category, description, tags, price, genre |
| Context information | The situation in which a choice is made | Time, location, device, season, current activity |

### Explicit and implicit feedback

| Feedback | Meaning | Examples | Main limitation |
| --- | --- | --- | --- |
| Explicit | The user directly states an opinion | Rating, like/dislike, review, favourite | Requires user effort, so it may be sparse |
| Implicit | The system infers preference from behaviour | Click, purchase, watch time, search, browsing | Behaviour is ambiguous; a click does not always mean a like |

Both types are useful. Explicit feedback is usually clearer, while implicit feedback is easier to collect in large quantities.

### Basic recommendation pipeline

1. Collect user, item, interaction, and possibly context data.
2. Build a representation or profile of the user.
3. Generate a manageable set of candidate items.
4. Use a recommendation model to predict scores.
5. Rank the candidates and return the top-$k$ items.
6. Observe the user’s response and update the profile or model.

The last step creates a continuous feedback loop:

$$
\text{Interaction}\rightarrow\text{Recommendation}\rightarrow\text{Response}\rightarrow\text{Learning}
$$

### Main recommendation approaches — overview

- **Content-based recommendation** uses properties of items and the user’s past interests.
- **Collaborative filtering** uses interaction patterns of many users. It may compare similar users or similar items.
- **Hybrid recommendation** combines more than one approach.

The algorithms are studied in later units; in Unit 1, understand the basic distinction.

## 3. Functions and applications of recommender systems

### Major functions

- **Prediction:** estimate how much a user may like an item.
- **Filtering:** remove unsuitable or irrelevant candidates.
- **Ranking:** arrange candidates from most to least promising.
- **Personalization:** adapt the result to the individual.
- **Discovery:** expose the user to useful items they might not find alone.
- **Decision support:** reduce comparison effort and cognitive load.
- **Engagement:** help users continue finding useful products or content.

### Application areas

| Domain | Typical recommendations |
| --- | --- |
| E-commerce | Products and accessories |
| Entertainment | Movies, videos, music |
| News | Articles and topics |
| Education | Courses, books, learning resources |
| Research | Papers, researchers, conferences |
| Social media | Posts, users, communities |
| Travel and food | Hotels, destinations, restaurants, dishes |

### Qualities of a good recommender

- **Relevance:** items should match the user’s needs.
- **Accuracy:** predicted preferences should reasonably match actual preferences.
- **Diversity:** the list should not contain only nearly identical items.
- **Novelty:** the system should include useful items the user does not already know.
- **Serendipity:** some recommendations should be both useful and pleasantly unexpected.
- **Coverage:** the system should be able to recommend a sufficiently broad range of items.
- **Adaptability:** results should change when the user’s interests change.
- **Explainability:** the system should provide understandable reasons for recommendations where possible.

Do not confuse these terms. A list can be accurate but lack diversity, or be novel without being relevant.

## 4. Information Retrieval and recommendation

**Information Retrieval (IR)** is the process of finding relevant information from a large collection in response to an information need, usually expressed as a query.

An **information need** is what the user actually wants to know. A **query** is the text or expression submitted to the system. The two may differ because a short query may not fully express the real need.

| Information Retrieval | Recommender System |
| --- | --- |
| Usually query-driven | Usually preference-driven |
| The user explicitly searches | The system can suggest items proactively |
| The query is central | User, item, interaction, and context data are central |
| Retrieves information relevant to the query | Predicts and ranks items useful to the user |

Real platforms can combine both: a search query creates candidates, after which personal preferences help rank them.

### Basic IR architecture

**Document side:**

$$
\text{Document collection}\rightarrow\text{Preprocessing}\rightarrow\text{Index}
$$

**Query side:**

$$
\text{Query}\rightarrow\text{Query processing}\rightarrow\text{Retrieval}\rightarrow\text{Ranking}
$$

Important preprocessing operations include tokenization, conversion to a common case, stop-word removal, stemming, and lemmatization. An **inverted index** stores, for each term, the list of documents containing that term. This lets the system find matching documents without scanning the entire collection every time.

## 5. Retrieval models

A retrieval model defines how a system represents a query and documents, decides whether they match, and possibly calculates a ranking score.

| Model | Main idea | Strength | Limitation |
| --- | --- | --- | --- |
| Boolean model | Uses AND, OR, and NOT for exact set matching | Simple and gives precise logical control | Usually gives no natural relevance ranking |
| Vector Space Model | Represents queries and documents as weighted vectors | Supports partial matching and ranked output | Quality depends on the chosen term weights |
| Probabilistic model | Estimates $P(R=1\mid d,q)$, the probability that document $d$ is relevant to query $q$ | Gives a direct ranking interpretation | Relevance probabilities can be difficult to estimate |
| BM25 | Scores query terms using IDF, term-frequency saturation, and document-length normalization | Strong classical ranked-retrieval method | Has parameters and is more complex than basic TF-IDF |

### 5.1 Boolean retrieval

If $A$ is the set of documents containing “recommender” and $B$ contains “system”:

- `A AND B` gives $A\cap B$: documents containing both terms.
- `A OR B` gives $A\cup B$: documents containing at least one term.
- `A NOT B` gives $A-B$: documents in $A$ but not in $B$.

Boolean retrieval decides exact satisfaction. It does not itself say which matching document is best.

### 5.2 TF-IDF weighting

**Term Frequency (TF)** measures how strongly a term occurs in one document. It may be the raw count or a normalized count:

$$
TF(t,d)=\frac{\text{count of }t\text{ in }d}{\text{total terms in }d}
$$

**Inverse Document Frequency (IDF)** gives more importance to rare, discriminating terms:

$$
IDF(t)=\log_{10}\left(\frac{N}{df_t}\right)
$$

where $N$ is the total number of documents and $df_t$ is the number containing term $t$.

$$
TFIDF(t,d)=TF(t,d)\times IDF(t)
$$

A frequent term in a particular document receives high TF, but a term appearing in almost every document receives low IDF.

#### Worked numerical: TF-IDF

Given $N=1000$, $df_t=10$, and $TF(t,d)=0.4$, using base-10 logarithm:

1. Calculate IDF:
    
    $$
    IDF(t)=\log_{10}(1000/10)=\log_{10}(100)=2
    $$
    
2. Multiply by TF:
    
    $$
    TFIDF(t,d)=0.4\times2=0.8
    $$
    

**Exam caution:** always state the logarithm convention. Different log bases produce different numerical values but the same ranking trend when used consistently.

### 5.3 Cosine similarity

Cosine similarity measures the angle between two non-zero vectors. It is widely used to compare a query vector with a document vector:

$$
\operatorname{cosine}(Q,D)=\frac{Q\cdot D}{\|Q\|\,\|D\|}
$$

where $Q\cdot D=\sum_i Q_iD_i$ and $\|Q\|=\sqrt{\sum_iQ_i^2}$. A value closer to $1$ means the vectors point in more similar directions.

#### Worked numerical: cosine similarity

Let $Q=[1,1,0]$ and $D=[1,2,1]$.

1. Dot product:
    
    $$
    Q\cdot D=(1)(1)+(1)(2)+(0)(1)=3
    $$
    
2. Magnitudes:
    
    $$
    \|Q\|=\sqrt2,\qquad \|D\|=\sqrt6
    $$
    
3. Similarity:
    
    $$
    \operatorname{cosine}(Q,D)=\frac{3}{\sqrt2\sqrt6}
    =\frac{3}{\sqrt{12}}\approx0.866
    $$
    

Cosine similarity is undefined if either vector has zero magnitude.

### 5.4 BM25 — concise exam view

For query $Q$ and document $D$, BM25 uses:

$$
Score(D,Q)=\sum_{t\in Q}IDF(t)
\frac{f(t,D)(k_1+1)}
{f(t,D)+k_1\left(1-b+b\frac{|D|}{avgdl}\right)}
$$

where:

- $f(t,D)$ is the frequency of term $t$ in $D$;
- $|D|$ is the document length;
- $avgdl$ is the average document length;
- $k_1$ controls term-frequency saturation;
- $b$ controls document-length normalization.

BM25 rewards important query terms but prevents repeated occurrences from increasing the score without limit. It also avoids unfairly favouring long documents. Learn the interpretation; the supplied Unit 1 material does not provide a complete BM25 numerical.

## 6. Search, filtering, and ranking

| Technique | Meaning |
| --- | --- |
| Keyword search | Finds documents containing supplied words |
| Boolean search | Combines conditions using AND, OR, and NOT |
| Phrase search | Finds an exact sequence of words |
| Wildcard search | Uses a pattern to match word variations |
| Fuzzy search | Allows approximate matches, such as spelling variations |
| Field search | Restricts matching to a field such as title or author |
| Range search | Restricts a numeric or date value to a range |
| Faceted search | Lets the user narrow results through categories such as brand or price |

These stages have different roles:

- **Search/retrieval** finds items matching the query.
- **Filtering** removes items that violate requirements or are unsuitable.
- **Ranking** orders the remaining items by estimated usefulness.

A combined system commonly follows:

$$
\text{Query}\rightarrow\text{Candidate retrieval}\rightarrow\text{Filtering}
\rightarrow\text{Personalized ranking}\rightarrow Top\text{-}k
$$

Here, “filtering” means candidate reduction. It should not be confused with the names **content-based filtering** and **collaborative filtering**, which are recommendation approaches.

### Precision, recall, and F1-score

Let $A$ be the retrieved set and $R$ the relevant set:

$$
Precision=\frac{|A\cap R|}{|A|}
\qquad
Recall=\frac{|A\cap R|}{|R|}
$$

- **Precision** asks: “Of the retrieved documents, how many were relevant?”
- **Recall** asks: “Of all relevant documents, how many were retrieved?”

The harmonic mean balances both:

$$
F_1=\frac{2(Precision)(Recall)}{Precision+Recall}
$$

#### Worked numerical

Suppose 50 documents are retrieved, 40 of them are relevant, and 100 relevant documents exist in total.

$$
Precision=\frac{40}{50}=0.8
$$

$$
Recall=\frac{40}{100}=0.4
$$

$$
F_1=\frac{2(0.8)(0.4)}{0.8+0.4}=0.533
$$

High precision means little irrelevant material was returned; high recall means little relevant material was missed.

## 7. Relevance feedback

**Relevance feedback** uses the user’s judgement or behaviour after a result to improve later retrieval or recommendation.

$$
\text{Result}\rightarrow\text{User feedback}\rightarrow\text{Updated representation}
\rightarrow\text{Improved result}
$$

### Types of feedback

- **Explicit feedback:** ratings, likes/dislikes, or direct relevance judgements.
- **Implicit feedback:** clicks, purchases, watch time, skips, or dwell time.
- **Hybrid feedback:** a weighted combination of explicit and implicit signals.
- **Positive feedback:** evidence that the user prefers an item.
- **Negative feedback:** evidence that the user rejects or dislikes an item.

If explicit and implicit scores are combined as

$$
F=\alpha E+(1-\alpha)I,
$$

the two scores should first be placed on compatible scales. Otherwise, the larger scale may dominate the result.

### Rocchio relevance-feedback model

Rocchio moves the current query or preference vector **towards relevant items** and **away from non-relevant items**:

$$
q_{new}=\alpha q_0+
\frac{\beta}{|D_r|}\sum_{d\in D_r}d-
\frac{\gamma}{|D_{nr}|}\sum_{d\in D_{nr}}d
$$

where:

- $q_0$ is the original query or preference vector;
- $D_r$ and $D_{nr}$ are the relevant and non-relevant sets;
- $\alpha$ controls the original vector;
- $\beta$ controls relevant-item influence;
- $\gamma$ controls non-relevant-item influence.

#### Worked numerical: Rocchio update

Given:

$$
q_0=(2,1),\quad D_r=\{(4,3),(6,2)\},\quad D_{nr}=\{(1,5)\}
$$

and $\alpha=1,\ \beta=0.75,\ \gamma=0.25$:

1. Average relevant vector:
    
    $$
    \frac{(4,3)+(6,2)}{2}=(5,2.5)
    $$
    
2. Average non-relevant vector: $(1,5)$.
3. Substitute:
    
    $$
    q_{new}=(2,1)+0.75(5,2.5)-0.25(1,5)
    $$
    
4. Therefore:
    
    $$
    q_{new}=(5.5,1.625)
    $$
    

## 8. User profiles

A **user profile** is a stored representation of information believed to describe a user’s interests or behaviour. It connects past evidence to future recommendations.

### Profile contents and forms

| Profile type | What it contains |
| --- | --- |
| Demographic | Age, location, occupation, language, education |
| Behavioural | Clicks, purchases, searches, watched or visited items |
| Preference | Direct interests, ratings, likes and dislikes |
| Vector profile | Numerical weights for features such as genres or topics |

A vector such as

$$
P_u=(0.8,0.2,0.6)
$$

may represent the user’s strength of interest in three fixed features. The feature order must always be known.

### Constructing a profile

A system may use registration details, explicit ratings, implicit behaviour, social information, and the features of items the user consumed. A usable profile usually converts these heterogeneous signals into a common representation, such as a feature vector.

### Updating a profile

Preferences change, so a profile can be updated incrementally:

$$
P_{updated}=(1-\lambda)P_{old}+\lambda P_{new}
$$

where $0\leq\lambda\leq1$. A larger $\lambda$ gives more influence to the new interaction.

#### Worked numerical: profile update

Let

$$
P_{old}=(0.7,0.3,0.5),\quad P_{new}=(0.9,0.8,0.2),\quad \lambda=0.2.
$$

Then:

$$
P_{updated}=0.8(0.7,0.3,0.5)+0.2(0.9,0.8,0.2)
$$

$$
=(0.56,0.24,0.40)+(0.18,0.16,0.04)=(0.74,0.40,0.44)
$$

### Time-based distinctions

- A **static profile** changes rarely; a **dynamic profile** is updated as behaviour changes.
- A **long-term profile** represents stable interests; a **short-term profile** represents the current session or recent need.

A practical system may combine both long-term stability and recent interest.

## 9. Ratings and the user–item matrix

For user $u$ and item $i$, $r_{ui}$ denotes the observed rating or interaction value.

| Rating/signal type | Meaning | Example |
| --- | --- | --- |
| Explicit | Directly supplied preference | 4 out of 5 stars |
| Implicit | Behaviour used as preference evidence | Purchase or watch time |
| Binary | Two possible states | Clicked/not clicked, liked/not liked |
| Ordinal | Ordered categories whose gaps may not be equal | Poor, fair, good, excellent |
| Continuous measurement | A real-valued behavioural quantity | Minutes watched or completion percentage |

### User–item rating matrix

Rows represent users and columns represent items:

$$
R=
\begin{bmatrix}
5&4&0&2&0\\
4&5&3&0&1\\
1&2&0&5&4\\
0&1&4&5&5
\end{bmatrix}
$$

For example, $R_{12}=4$ means that user $U_1$ rated item $I_2$ as 4.

**Critical caution:** in this matrix, $0$ is only a placeholder for a missing or unobserved rating. It is not an actual rating of zero. Unknown entries must not be silently treated as genuine zero preferences in similarity or covariance calculations.

## 10. Matrix operations used in recommendation

| Operation | Rule and purpose |
| --- | --- |
| Addition/subtraction | Matrices must have the same dimensions; combine corresponding entries |
| Scalar multiplication | Multiply every entry by the scalar; useful for weighting |
| Transpose | Convert rows into columns; if $A$ is $m\times n$, $A^T$ is $n\times m$ |
| Dot product | Multiply matching vector entries and add; produces a similarity or score component |
| Matrix multiplication | Multiply rows of the first matrix by columns of the second |

For multiplication:

$$
A_{m\times n}B_{n\times p}=C_{m\times p}
$$

The inner dimensions must match.

### Worked numerical: matrix multiplication

Let

$$
A=\begin{bmatrix}1&2\\3&4\end{bmatrix},\qquad
B=\begin{bmatrix}5&6\\7&8\end{bmatrix}.
$$

Compute every output entry as a row–column dot product:

$$
AB=
\begin{bmatrix}
1(5)+2(7) & 1(6)+2(8)\\
3(5)+4(7) & 3(6)+4(8)
\end{bmatrix}
=
\begin{bmatrix}19&22\\43&50\end{bmatrix}.
$$

## 11. Covariance, correlation, and covariance matrices

### Covariance

Covariance measures whether two variables tend to vary together. The sample covariance is:

$$
Cov(X,Y)=\frac{1}{n-1}\sum_{i=1}^{n}(x_i-\bar x)(y_i-\bar y)
$$

where $n$ is the number of paired observations, and $\bar x,\bar y$ are the means.

- $Cov(X,Y)>0$: the variables tend to increase together.
- $Cov(X,Y)<0$: one tends to increase while the other decreases.
- $Cov(X,Y)\approx0$: little linear co-variation is present.

#### Worked numerical: covariance

Let $X=[2,3,4,5]$ and $Y=[1,2,4,5]$.

1. Means:
    
    $$
    \bar X=3.5,\qquad \bar Y=3
    $$
    
2. Deviations:
    
    $$
    X-\bar X=[-1.5,-0.5,0.5,1.5]
    $$
    
    $$
    Y-\bar Y=[-2,-1,1,2]
    $$
    
3. Products of paired deviations:
    
    $$
    [3,0.5,0.5,3],\qquad \text{sum}=7
    $$
    
4. Sample covariance:
    
    $$
    Cov(X,Y)=\frac{7}{4-1}=2.333
    $$
    

The positive result indicates that the two rating patterns tend to move in the same direction.

### Covariance matrix

For variables $X_1,X_2,\ldots,X_p$, all pairwise covariance values are arranged as:

$$
\Sigma=
\begin{bmatrix}
Var(X_1)&Cov(X_1,X_2)&\cdots\\
Cov(X_2,X_1)&Var(X_2)&\cdots\\
\vdots&\vdots&\ddots
\end{bmatrix}
$$

Important properties:

- Diagonal entries are variances because $Cov(X_i,X_i)=Var(X_i)$.
- Off-diagonal entries are pairwise covariances.
- The matrix is symmetric: $\Sigma=\Sigma^T$.

In recommendation, it can describe how item ratings, user preferences, or features vary together. PCA may use this matrix to find directions of high variance, but the detailed PCA procedure is outside the required Unit 1 depth.

### Correlation and Pearson similarity

Correlation is normalized covariance:

$$
\rho_{XY}=\frac{Cov(X,Y)}{\sigma_X\sigma_Y},\qquad -1\leq\rho_{XY}\leq1
$$

| Covariance | Correlation |
| --- | --- |
| Depends on measurement scale | Independent of measurement scale |
| Unbounded | Always between $-1$ and $1$ |
| Shows the direction of joint variation | Shows direction and standardized strength |

For two users $u$ and $v$, Pearson similarity applies the same idea to their commonly rated items $I_{uv}$:

$$
sim(u,v)=
\frac{\sum_{i\in I_{uv}}(r_{ui}-\bar r_u)(r_{vi}-\bar r_v)}
{\sqrt{\sum_{i\in I_{uv}}(r_{ui}-\bar r_u)^2}
\sqrt{\sum_{i\in I_{uv}}(r_{vi}-\bar r_v)^2}}
$$

Pearson is useful when users employ rating scales differently because it compares deviations from their mean ratings. In a numerical, clearly state which commonly rated items and means are being used.

## 12. Major issues in recommender systems

### 12.1 Data sparsity

Most users interact with only a tiny fraction of the available items. The rating matrix therefore contains many missing entries.

$$
Sparsity=1-\frac{\text{number of observed interactions}}
{\text{number of users}\times\text{number of items}}
$$

If 1,000 users and 10,000 items give $10,000,000$ possible entries but only 100,000 are observed:

$$
Sparsity=1-\frac{100000}{10000000}=0.99=99\%.
$$

High sparsity makes similarities and preference estimates less reliable.

### 12.2 Cold start

Cold start means that the system has insufficient information to recommend effectively.

- **New-user cold start:** little or no preference history exists.
- **New-item cold start:** few users have interacted with the item.
- **New-system cold start:** the entire platform has little interaction data.

Initial preference questions, item information, and implicit feedback can reduce the problem, but no single method removes it completely.

### 12.3 Scalability

The cost of storing interactions and comparing users or items grows rapidly on large platforms. Candidate generation, compact representations, and distributed computation are common high-level responses. For Unit 1, explain the problem rather than implementation details.

### 12.4 Bias-related issues

- **Rating bias:** some users consistently rate more strictly or generously than others.
- **Popularity bias:** already popular items receive more exposure and therefore even more interactions.
- **Long-tail problem:** less-popular but potentially useful items receive too little exposure.
- **Filter bubble:** repeated personalization narrows the content shown and may reinforce the same interests.

These issues show why prediction accuracy alone is not sufficient; diversity, novelty, coverage, and fairness also matter.

### 12.5 Manipulation attacks

In a **shilling** or **profile-injection attack**, malicious users submit fake ratings to promote or demote items. The false ratings can create artificial similarity patterns and manipulate recommendations. Possible high-level protections include anomaly detection, trust information, and robust aggregation. Detailed attack models belong to Unit 3.

### 12.6 Privacy, fairness, and explainability

- **Privacy:** personal histories and preferences must be collected and processed carefully. Data minimization, consent, access control, and secure storage are important.
- **Fairness:** the system should avoid unjustifiably favouring or disadvantaging users, creators, sellers, or item groups.
- **Explainability:** understandable reasons such as “recommended because you liked…” can improve trust and acceptance.

## 13. High-value exam comparisons

| Comparison | Key distinction |
| --- | --- |
| IR vs recommender system | Query-driven retrieval vs preference-driven personalized suggestion |
| Explicit vs implicit feedback | Direct opinion and clearer meaning vs abundant but ambiguous behaviour |
| Search vs filtering vs ranking | Find candidates vs remove unsuitable candidates vs order candidates |
| Precision vs recall | Correctness of retrieved results vs coverage of all relevant results |
| Static vs dynamic profile | Rarely changes vs updates with new behaviour |
| Long-term vs short-term profile | Stable interests vs current/session interest |
| Covariance vs correlation | Scale-dependent joint variation vs normalized strength in $[-1,1]$ |
| Sparsity vs cold start | Too few entries overall vs too little information for a new user/item/system |