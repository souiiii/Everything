# Rec Sys - FINAL_REVISION

# Recommender Systems — Final Midsem Revision

These notes are designed for answer-writing practice. Read each question, try to recall its structure, and then revise the model answer.

---

# Unit 1 — Foundations, Information Retrieval and Evaluation

### Q1. Define the two primary approaches in recommender systems and briefly explain how they differ. — 2 marks [Asked in 2025 Midsem]

The two primary approaches are **content-based filtering** and **collaborative filtering**. Content-based filtering recommends items whose features resemble the items previously liked by the user. Collaborative filtering recommends from the interaction patterns of many users, without requiring detailed item descriptions. Thus, the former mainly depends on item content and a user profile, whereas the latter mainly depends on user-item interaction data.

### Q2. What is a recommender system? Why is it needed? — 5 marks

A **recommender system** is a software system that studies users, items and their interactions to predict and rank items that may be useful to a particular user. It answers the practical question, “What should this user choose?”

It is needed because online platforms contain far more choices than a person can examine. **Information overload** occurs when the available information exceeds the user’s ability to process it, while **choice overload** makes comparison and decision-making difficult. A recommender reduces this search space while preserving relevance through:

1. Preference prediction;
2. Candidate filtering;
3. Personalized ranking;
4. Useful-item discovery; and
5. Decision support.

Formally, a model estimates user $u$’s preference for item $i$:

$$
\hat{r}_{ui}=f(u,i,D)
$$

where $D$ is the available user, item and interaction data. The system sorts candidates by $\hat{r}_{ui}$ and returns the highest-scoring items. Because these scores depend on the individual user, two people can receive different rankings from the same catalogue. Examples include movie, product, music, news, course and research-paper recommendation.

### Q3. Explain the information and feedback used by a recommender-system pipeline. — 5 marks

A recommender may use **user information** such as interests and history, **item information** such as category and description, and **context information** such as time, location or device. These signals have different roles: user data indicates who is being served, item data describes what can be recommended, and context explains the situation in which the choice is made.

**Explicit feedback** is directly supplied by the user, for example a rating or like. It has clearer meaning but requires effort. **Implicit feedback** is inferred from clicks, purchases, searches or watch time. It is abundant but ambiguous because an interaction does not always mean that the item was liked.

A standard pipeline is:

1. Collect user, item, interaction and context data;
2. Construct user and item representations;
3. Generate candidate items;
4. Predict preference scores;
5. Rank and recommend the Top-N items; and
6. Use new feedback to update the profile or model.

The final step forms a feedback loop. For example, a movie watched after a recommendation becomes new evidence for later predictions. A sound system therefore updates its understanding of the user instead of treating preferences as permanently fixed.

### Q4. Differentiate Information Retrieval and recommender systems. Briefly describe the main retrieval models. — 5 marks

**Information Retrieval (IR)** finds relevant information in response to an explicit query, whereas a recommender predicts items from a user’s preferences and behaviour. An information need is what the user actually wants, while the query is the expression submitted to the system; the two are not always identical. IR is normally query-driven, whereas recommendation is preference-driven and may work proactively. A real platform can combine both by retrieving candidates using a query and then ranking them personally.

| Retrieval model | Main idea | Main strength/limitation |
| --- | --- | --- |
| Boolean model | Uses AND, OR and NOT for exact set matching. | Gives precise logical control but normally no natural relevance ranking. |
| Vector Space Model | Represents queries and documents as weighted vectors and supports ranked partial matching. | Supports cosine ranking, but quality depends on feature weights. |
| Probabilistic model | Ranks a document by its estimated probability of relevance to the query. | Gives a clear ranking interpretation, but probabilities can be difficult to estimate. |
| BM25 | Uses term importance, term-frequency saturation and document-length normalization. | Produces strong classical rankings but requires parameters and more computation than basic TF-IDF. |

On the document side, text is preprocessed and stored in an **inverted index**, which lists the documents containing each term. On the query side, the query is processed, matching candidates are retrieved, and the chosen model ranks them. The index avoids scanning the complete collection for every query.

### Q5. Calculate TF-IDF for a term and interpret the result. — 5 marks

**Question:** A collection has $N=1000$ documents. A term occurs in $df_t=10$ documents and has normalized term frequency $TF(t,d)=0.4$ in document $d$. Find its TF-IDF using base-10 logarithm.

**Formula:**

$$
IDF(t)=\log_{10}\left(\frac{N}{df_t}\right)
$$

$$
TFIDF(t,d)=TF(t,d)\times IDF(t)
$$

Here, $N$ is the number of documents and $df_t$ is the number containing the term.

**Working:**

$$
IDF(t)=\log_{10}\left(\frac{1000}{10}\right)=\log_{10}(100)=2
$$

$$
TFIDF(t,d)=0.4\times2=\boxed{0.8}
$$

A high TF-IDF means that the term is frequent in this document but relatively uncommon in the collection.

**Exam tip:** State the logarithm base and use it consistently.

### Q6. Explain the difference between search, filtering and ranking. Define precision and recall. — 5 marks

**Search or retrieval** finds candidates that match a query. **Filtering** removes candidates that violate a requirement or are unsuitable. **Ranking** orders the remaining candidates by estimated usefulness. A typical sequence is:

$$
\text{Query}\rightarrow\text{Retrieval}\rightarrow\text{Filtering}
\rightarrow\text{Personalized ranking}\rightarrow\text{Top-N results}
$$

If $TP$ is the number of relevant retrieved items, $FP$ is the number of irrelevant retrieved items, and $FN$ is the number of relevant items missed:

$$
Precision=\frac{TP}{TP+FP}
$$

$$
Recall=\frac{TP}{TP+FN}
$$

Precision measures the correctness of retrieved results, while recall measures how completely the relevant set was found.

Their harmonic mean is:

$$
F_1=\frac{2(Precision)(Recall)}{Precision+Recall}
$$

**Worked example:** Suppose 50 documents are retrieved, 40 of them are relevant, and 100 relevant documents exist in total.

$$
Precision=\frac{40}{50}=0.8,
\qquad
Recall=\frac{40}{100}=0.4
$$

$$
F_1=\frac{2(0.8)(0.4)}{0.8+0.4}
=\boxed{0.533}
$$

The result has high precision but modest recall: most retrieved documents are useful, yet many relevant documents remain unfound.

### Q7. What is relevance feedback? Explain the Rocchio model with a numerical. — 6 marks

**Relevance feedback** uses a user’s judgement or behaviour after seeing results to improve later retrieval or recommendation. Feedback may be explicit, such as relevant/non-relevant labels, or implicit, such as clicks and dwell time.

Rocchio moves the query vector towards relevant documents and away from non-relevant documents:

$$
q_{\text{new}}=\alpha q_0+
\frac{\beta}{|D_r|}\sum_{d\in D_r}d-
\frac{\gamma}{|D_{nr}|}\sum_{d\in D_{nr}}d
$$

Here, $q_0$ is the original query, while $D_r$ and $D_{nr}$ are the relevant and non-relevant document sets.

**Numerical:** Let

$$
q_0=(2,1),\quad D_r=\{(4,3),(6,2)\},\quad D_{nr}=\{(1,5)\}
$$

with $\alpha=1$, $\beta=0.75$ and $\gamma=0.25$.

The average relevant vector is:

$$
\frac{(4,3)+(6,2)}{2}=(5,2.5)
$$

Therefore:

$$
q_{\text{new}}=(2,1)+0.75(5,2.5)-0.25(1,5)
$$

$$
q_{\text{new}}=\boxed{(5.5,1.625)}
$$

The updated query gives more weight to features found in relevant documents.

### Q8. What is a user profile? How can it be updated when preferences change? — 5 marks

A **user profile** is a stored representation of a user’s interests or behaviour. It may contain demographic data, browsing behaviour, explicit preferences, or a numerical vector over features. Profiles can be static or dynamic and may represent long-term as well as short-term interests.

The profile may be constructed from registration details, ratings, likes, purchases, searches and features of consumed items. A **static profile** changes rarely, whereas a **dynamic profile** is revised when new behaviour arrives. A **long-term profile** represents stable interests; a **short-term profile** captures the current session or recent need. Practical recommenders may combine both so that new interests matter without erasing established preferences.

A simple incremental update is:

$$
P_{\text{updated}}=(1-\lambda)P_{\text{old}}+\lambda P_{\text{new}}
$$

where $\lambda$ controls the influence of new evidence.

Here, a small $\lambda$ preserves more of the old profile, while a large $\lambda$ makes the system respond more strongly to recent behaviour. This gradual update lets the recommender adapt without immediately discarding stable earlier interests.

### Q9. Explain ratings and the user-item matrix. Why must missing values be handled carefully? — 4 marks

Ratings may be **explicit**, such as 1–5 stars, or **implicit**, such as a click or purchase. They may also be binary, ordinal or continuous. Explicit ratings are clearer but require effort; implicit signals are easier to collect but may not express true liking.

The observations are commonly placed in a matrix $R=[r_{ui}]$, where rows represent users, columns represent items, and $r_{ui}$ is user $u$’s interaction with item $i$. For example:

$$
R=
\begin{bmatrix}
5&4&?\\
4&?&3\\
1&2&5
\end{bmatrix}
$$

The matrix supports user comparison, item comparison and missing-rating prediction. In practice it is usually sparse because each user interacts with only a small fraction of the catalogue.

A blank entry means that the interaction is unknown. A displayed zero is often only a placeholder for a missing value and should not automatically be treated as a genuine zero preference. Incorrect zero-filling can distort similarities, covariance values and predicted ratings.

### Q10. Distinguish covariance and correlation, and calculate a sample covariance. — 6 marks

Covariance measures whether two variables tend to vary together:

$$
\operatorname{Cov}(X,Y)=\frac{1}{n-1}\sum_{i=1}^{n}(x_i-\bar{x})(y_i-\bar{y})
$$

A positive value indicates movement in the same direction, while a negative value indicates opposite movement. Correlation is normalized covariance:

$$
\rho_{XY}=\frac{\operatorname{Cov}(X,Y)}{\sigma_X\sigma_Y}
$$

Covariance depends on measurement scale and is unbounded. Correlation is scale-independent and lies between $-1$ and $1$.

**Worked covariance:** Let $X=[2,3,4,5]$ and $Y=[1,2,4,5]$. Their means are $\bar{X}=3.5$ and $\bar{Y}=3$.

$$
X-\bar{X}=[-1.5,-0.5,0.5,1.5]
$$

$$
Y-\bar{Y}=[-2,-1,1,2]
$$

The paired deviation products are $[3,0.5,0.5,3]$, whose sum is $7$. Therefore, the sample covariance is:

$$
\operatorname{Cov}(X,Y)=\frac{7}{4-1}=\boxed{2.333}
$$

The positive result means that the two variables generally rise together.

**Exam tip:** Check whether the question asks for sample covariance $(n-1)$ or population covariance $(n)$, and use the same convention throughout.

### Q11. Multiply two matrices and explain the dimension rule. — 4 marks

Two matrices can be multiplied only when the number of columns in the first equals the number of rows in the second. If $A$ is $m\times n$ and $B$ is $n\times p$, then $AB$ is $m\times p$. Each result entry is the dot product of one row of $A$ and one column of $B$.

**Numerical:** Let

$$
A=\begin{bmatrix}1&2\\3&4\end{bmatrix},
\qquad
B=\begin{bmatrix}5&6\\7&8\end{bmatrix}
$$

Then:

$$
AB=
\begin{bmatrix}
1(5)+2(7) & 1(6)+2(8)\\
3(5)+4(7) & 3(6)+4(8)
\end{bmatrix}
$$

$$
AB=\boxed{
\begin{bmatrix}
19&22\\43&50
\end{bmatrix}}
$$

Matrix multiplication is useful in recommendation because user-factor and item-factor matrices can be multiplied to generate predicted scores.

**Exam tip:** Check the inner dimensions before multiplying and calculate every result entry using a row-column dot product.

### Q12. Construct a covariance matrix from two variables. — 6 marks

A covariance matrix contains all pairwise covariance values. Its diagonal entries are variances, its off-diagonal entries are covariances, and it is symmetric.

**Numerical:** Let $X=[1,2,3]$ and $Y=[2,4,6]$. The means are $\bar{X}=2$ and $\bar{Y}=4$, so:

$$
X-\bar{X}=[-1,0,1],
\qquad
Y-\bar{Y}=[-2,0,2]
$$

Using the sample denominator $n-1=2$:

$$
\operatorname{Var}(X)=\frac{1^2+0^2+1^2}{2}=1
$$

$$
\operatorname{Var}(Y)=\frac{2^2+0^2+2^2}{2}=4
$$

$$
\operatorname{Cov}(X,Y)=\frac{(-1)(-2)+(0)(0)+(1)(2)}{2}=2
$$

Therefore:

$$
\Sigma=
\begin{bmatrix}
\operatorname{Var}(X)&\operatorname{Cov}(X,Y)\\
\operatorname{Cov}(Y,X)&\operatorname{Var}(Y)
\end{bmatrix}
=\boxed{
\begin{bmatrix}
1&2\\
2&4
\end{bmatrix}}
$$

The equal off-diagonal entries confirm symmetry. Since $Y=2X$, the two variables have a perfect positive linear relationship.

**Exam tip:** Use the same sample or population denominator for every variance and covariance entry.

### Q13. Explain the major issues faced by recommender systems. — 6 marks

Recommendation quality is affected not only by the chosen algorithm but also by the amount, reliability and social effect of the available data.

| Issue | Explanation and consequence |
| --- | --- |
| Sparsity | Only a small fraction of possible user-item interactions is observed. Users and items then have little overlap, so similarity and preference estimates become unreliable. |
| Cold start | A new user, new item or new system has insufficient history. The system cannot yet construct a dependable profile or collaborative pattern. |
| Scalability | Storage, similarity calculation and prediction costs rise rapidly as users and items increase. A method suitable for a classroom matrix may become too slow on a large platform. |
| Popularity bias and long tail | Popular items receive more exposure and hence even more interactions. Useful niche items in the long tail may remain almost invisible. |
| Changing interests | User preferences evolve over time. A profile based only on old behaviour may recommend items that are no longer relevant. |
| Manipulation | Attackers may inject fake ratings to promote their own items or demote competitors, distorting learned patterns. |
| Privacy and fairness | Histories can reveal sensitive information, while biased data may unfairly favour particular users, providers or item groups. |
| Filter bubbles and weak diversity | Recommending only items similar to earlier choices narrows exposure and reduces novelty and serendipity. |

One way to quantify sparsity is:

$$
Sparsity=1-\frac{\text{observed interactions}}
{\text{number of users}\times\text{number of items}}
$$

These problems show that prediction accuracy alone is not sufficient; diversity, novelty, coverage, fairness and explainability also matter.

### Q14. Name two common offline evaluation metrics used for recommender systems and briefly define each. — 2 marks [Asked in 2025 Midsem]

Two common offline metrics for rating prediction are **Mean Absolute Error (MAE)** and **Root Mean Squared Error (RMSE)**. MAE is the average absolute difference between actual and predicted ratings. RMSE is the square root of the average squared error and therefore penalizes large errors more strongly. Lower values of both metrics indicate more accurate rating predictions.

$$
MAE=\frac{1}{n}\sum_{j=1}^{n}|r_j-\hat{r}_j|
$$

$$
RMSE=\sqrt{\frac{1}{n}\sum_{j=1}^{n}(r_j-\hat{r}_j)^2}
$$

Here, $n$ is the number of evaluated ratings, $r_j$ is an actual rating and $\hat{r}_j$ is its prediction.

**Worked variation:** For actual ratings $[4,3,5]$ and predictions $[3.5,2.5,4]$:

$$
MAE=\frac{0.5+0.5+1}{3}=\boxed{0.667}
$$

$$
RMSE=\sqrt{\frac{0.25+0.25+1}{3}}
=\sqrt{0.5}=\boxed{0.707}
$$

For Top-K recommendation, two brief concepts are:

$$
Precision@K=\frac{\text{relevant items in Top-K}}{K}
$$

$$
Recall@K=\frac{\text{relevant items in Top-K}}
{\text{total relevant items for the user}}
$$

[Precision@K](mailto:Precision@K) measures correctness within the displayed list, while [Recall@K](mailto:Recall@K) measures how much of the user’s relevant set was recovered. These ranking concepts are included only at the depth required for the paper.

---

# Unit 2 — Content-Based Recommendation

### Q1. Explain how content-based filtering generates recommendations. What is one advantage over collaborative filtering? — 2 marks [Asked in 2025 Midsem]

Content-based filtering represents each item using features such as genre, tags, keywords or metadata. It builds a user profile from the features of items the user previously liked and recommends unseen items with similar profiles. A matching function such as cosine similarity is used to score and rank candidates. One advantage over collaborative filtering is that a new item can be recommended immediately if its content features are available.

### Q2. Explain the architecture and working of a content-based recommender. — 6 marks

A content-based system recommends by matching a user’s learned feature preferences with candidate-item features.

If item $i$ has feature vector $x_i$ and user $u$ has profile $P_u$, a common score is:

$$
\operatorname{Score}(u,i)=\operatorname{sim}(P_u,x_i)
$$

The two vectors must use the same feature order and meaning.

1. Collect item descriptions, attributes, documents or tags.
2. Clean the content and extract useful features.
3. Represent every item as a profile in a common feature space.
4. Build the user profile from liked, rated or consumed items.
5. Compare the user profile with unseen item profiles.
6. Rank candidates by similarity or predicted relevance.
7. Recommend the Top-N items and update the profile using feedback.

Its main components are the item database, content analyser, item-profile generator, user-profile learner, matching function, ranking engine and feedback updater. For example, if a viewer repeatedly likes action and science-fiction movies, those feature weights become strong in the user profile, so unseen movies with similar features receive high scores. New feedback then modifies the profile and improves later rankings.

### Q3. What is an item profile? Explain important document and item features. — 5 marks

An **item profile** is a structured representation of the characteristics of one item. It is produced by extracting usable features from the item’s content or metadata. Textual items may use words, TF-IDF weights or n-grams. Other useful features include category, price, genre, author, language, tags, publication date and document structure.

| Feature group | Example |
| --- | --- |
| Lexical | Words, term frequencies and n-grams |
| Metadata | Author, genre, language or year |
| Structural | Title, headings and document length |
| Tags | Action, sci-fi or machine learning |
| Numerical/categorical | Price, duration, brand or category |

Categorical properties may be one-hot encoded, numerical values may be normalized, and text may be converted into BoW or TF-IDF vectors. Every item must use the same vector coordinates. For example, if the order is $[\text{action},\text{comedy},\text{romance}]$, the vector $[1,0,1]$ represents an action-romance movie. An item profile answers “What describes this item?” while a user profile answers “Which of these features does this user prefer?”

### Q4. Explain the text-preprocessing pipeline. Differentiate stemming and lemmatization. — 5 marks

Text preprocessing converts raw content into a clean and consistent representation:

1. Remove unwanted symbols or markup;
2. Tokenize the text;
3. Normalize letter case;
4. Handle stop words;
5. Apply stemming or lemmatization; and
6. Retain or remove numbers according to their meaning.

| Stemming | Lemmatization |
| --- | --- |
| Removes endings using simple rules. | Uses linguistic analysis. |
| Faster but less precise. | Slower but more precise. |
| May produce an invalid root such as “studi.” | Produces a valid base word such as “study.” |

Numbers should not be removed blindly because expressions such as “64 GB” may carry important meaning.

### Q5. Compare Bag of Words, TF-IDF and n-grams. — 5 marks

These methods convert text into numerical features that can be compared with a user profile.

| Method | Representation | Main strength | Main limitation |
| --- | --- | --- | --- |
| Bag of Words | Word counts or binary word presence | Simple and interpretable | Ignores word order and meaning; vectors are often sparse |
| TF-IDF | Term frequency weighted by rarity across documents | Emphasizes discriminating terms | Still has limited semantic understanding |
| n-grams | Counts consecutive sequences of $n$ tokens | Preserves some local word order | Creates a larger and sparser feature space |

If the vocabulary is $[\text{AI},\text{recommender},\text{system}]$, the text “AI recommender recommender” has BoW vector $[1,2,0]$. A binary BoW records only presence and gives $[1,1,0]$.

TF-IDF improves ordinary counts by reducing the weight of terms that occur in many documents:

$$
TFIDF(t,d)=TF(t,d)\times\log\left(\frac{N}{df(t)}\right)
$$

An n-gram is a sequence of consecutive tokens. For example, the bigram “machine learning” preserves a useful phrase that separate unigram counts would lose. The appropriate representation depends on whether simple occurrence, term importance or limited word order matters most.

### Q6. How are tags used to obtain item features and recommendations? — 5 marks

Tags are short descriptive labels supplied by users, content providers or automatic analysis. They can describe concepts such as “action,” “space” or “machine learning” even when a long textual description is unavailable.

The recommendation process is:

1. Create a fixed tag vocabulary.
2. Represent each item using binary tags, tag counts, weighted tags or tag TF-IDF.
3. Aggregate tags from the user’s preferred items into a user tag profile.
4. Compare that profile with unseen item-tag vectors.
5. Rank and recommend the strongest matches.

For vocabulary $[\text{action},\text{comedy},\text{sci-fi},\text{space}]$, an action, sci-fi and space item becomes $[1,0,1,1]$. Tags are interpretable, support explanations and can describe a new item before it receives ratings. However, they may be missing, noisy, subjective, inconsistent or too general; very common tags also have little distinguishing power.

### Q7. Explain how a content-based user profile is learned. — 5 marks

A user profile gives a preference weight to each item feature. It is learned from explicit signals such as ratings and likes, or implicit signals such as clicks, purchases and reading history. The features of preferred items must first be represented in one common coordinate system.

For equally liked items:

$$
P_u=\frac{1}{|L_u|}\sum_{i\in L_u}x_i
$$

For weighted evidence:

$$
P_u=\frac{\sum_i w_{ui}x_i}{\sum_i|w_{ui}|}
$$

Here, $x_i$ is item $i$’s feature vector and $w_{ui}$ is the strength of user $u$’s preference for it. Positive weights increase a feature’s influence; negative weights can represent disliked content. A rating-centred weight $w_{ui}=r_{ui}-\bar{r}_u$ reduces the effect of unusually strict or generous rating habits.

A dynamic profile can give recent behaviour greater influence:

$$
P_u^{(t+1)}=(1-\alpha)P_u^{(t)}+\alpha x_i
$$

Short-term and long-term profiles may also be combined to balance a current need with stable interests. An unseen item must not automatically be treated as disliked because the user may simply not have discovered it.

### Q8. Construct average, weighted and dynamically updated user profiles. — 6 marks

Suppose three liked items have feature vectors:

$$
x_1=[1,0.8,0.2],\quad
x_2=[0.9,0.7,0.3],\quad
x_3=[0.8,0.9,0.1]
$$

Corresponding entries represent the same feature in every vector.

**(a) Average profile**

The average gives equal importance to every liked item:

$$
P_u=\frac{x_1+x_2+x_3}{3}
$$

$$
P_u=\frac{[2.7,2.4,0.6]}{3}
=\boxed{[0.9,0.8,0.2]}
$$

Thus, the first feature has the greatest average importance.

**(b) Weighted profile**

Suppose the three preference weights are $2$, $1$ and $1$. A weighted profile gives stronger evidence more influence:

$$
P_u=\frac{\sum_iw_{ui}x_i}{\sum_i|w_{ui}|}
$$

$$
P_u=\frac{2x_1+x_2+x_3}{2+1+1}
$$

$$
P_u=
\frac{[2,1.6,0.4]+[0.9,0.7,0.3]+[0.8,0.9,0.1]}{4}
$$

$$
P_u=\frac{[3.7,3.2,0.8]}{4}
=\boxed{[0.925,0.8,0.2]}
$$

The first liked item now contributes twice as much as either remaining item.

**(c) Dynamic-profile update**

Let the current profile be $P_u^{(t)}=[0.8,0.6,0.2]$, a newly preferred item be $x=[0.2,0.8,0.9]$, and $\alpha=0.3$:

$$
P_u^{(t+1)}=(1-\alpha)P_u^{(t)}+\alpha x
$$

$$
P_u^{(t+1)}
=0.7[0.8,0.6,0.2]+0.3[0.2,0.8,0.9]
$$

$$
P_u^{(t+1)}
=[0.56,0.42,0.14]+[0.06,0.24,0.27]
=\boxed{[0.62,0.66,0.41]}
$$

The third feature becomes more important because it is strong in the recent item.

**Exam tip:** Add and multiply vectors component-wise, and preserve the same feature order throughout.

### Q9. Which similarity measures are used for content-based retrieval? — 5 marks

| Measure | Formula/idea | Best suited to |
| --- | --- | --- |
| Cosine | $\dfrac{A\cdot B}{\|A\|\|B\|}$ | TF-IDF and non-negative feature vectors |
| Euclidean distance | $\sqrt{\sum_j(a_j-b_j)^2}$ | Numerical coordinates where absolute distance matters |
| Jaccard | $\dfrac{|A\cap B|}{|A\cup B|}$ | Binary tags and feature sets |
| Pearson | Correlation of mean-centred values | Numerical patterns measured on different baselines |

Cosine measures vector direction and is usually the first choice for TF-IDF profiles. Euclidean distance measures absolute geometric separation, so feature scale can strongly affect it. Jaccard ignores numerical weights and measures overlap between binary sets. Pearson first centres values around their means, making it useful when the relative pattern matters more than the absolute level.

Cosine, Jaccard and Pearson become larger as similarity increases, whereas a smaller Euclidean distance indicates greater similarity. The correct measure must match the data representation, and zero-magnitude vectors make cosine undefined.

### Q10. Measure the similarity of the two documents using Jaccard distance and cosine distance measures. — 4 marks [Asked in 2025 Midsem]

**Documents:**

- Doc 1: “Machine Learning is a subset of Artificial Intelligence”
- Doc 2: “Most Artificial Intelligence work now involves Machine Learning”

Assume lowercase binary word vectors, punctuation removal and no stop-word removal.

The term sets are:

$$
D_1=\{\text{machine, learning, is, a, subset, of, artificial, intelligence}\}
$$

$$
D_2=\{\text{most, artificial, intelligence, work, now, involves, machine, learning}\}
$$

There are 4 common terms and 12 terms in the union.

**Jaccard similarity and distance:**

$$
J(D_1,D_2)=\frac{|D_1\cap D_2|}{|D_1\cup D_2|}
=\frac{4}{12}=\frac{1}{3}
$$

$$
d_J=1-J=\boxed{\frac{2}{3}\approx0.667}
$$

For binary vectors, the dot product equals the 4 common terms and each vector has 8 non-zero entries.

**Cosine similarity and distance:**

$$
\operatorname{cosine}(D_1,D_2)
=\frac{4}{\sqrt{8}\sqrt{8}}=\frac{4}{8}=0.5
$$

$$
d_{\text{cosine}}=1-0.5=\boxed{0.5}
$$

**Exam tip:** State tokenization and stop-word assumptions because they affect the answer.

### Q11. Explain classification-based recommendation and compare it with similarity-based retrieval. — 6 marks

Classification-based recommendation treats recommendation as a supervised-learning problem. Each training item is represented by content features and assigned a label such as **Like/Dislike** or **Relevant/Not Relevant**. Positive labels may come from likes, high ratings or strong interactions. Negative examples must be chosen carefully because an unseen item is not necessarily disliked.

1. Obtain positive and carefully chosen negative examples.
2. Represent each pair using item and user features.
3. Train a classifier.
4. Predict relevance for candidate items.
5. Rank candidates by the predicted score.

**K-Nearest Neighbours (KNN):** KNN stores labelled examples rather than learning an explicit prediction equation. For a candidate item, it:

1. calculates similarity or distance to the labelled training items;
2. selects the $k$ nearest examples;
3. uses a majority or similarity-weighted vote; and
4. assigns the winning class or score.

If three of the five nearest items are labelled Like and two are labelled Dislike, KNN predicts Like. A weighted vote can give closer neighbours more influence. KNN is intuitive, but its output depends on feature scaling and the choice of $k$, and prediction can be slow when many examples are stored.

**Naive Bayes:** This method compares the posterior probabilities of the possible classes. For the Like class:

$$
P(\text{Like}\mid X)
\propto P(\text{Like})\prod_{j=1}^{m}P(x_j\mid\text{Like})
$$

Here, $X$ is the candidate’s feature vector, $P(\text{Like})$ is the prior probability of the class, and $P(x_j\mid\text{Like})$ is the likelihood of feature $x_j$ among liked items. The system similarly calculates the score for Dislike and chooses the larger posterior. Naive Bayes is efficient for high-dimensional text, but it assumes that features are conditionally independent once the class is known; this assumption is often only approximate.

**Other classifiers, briefly:**

- **Logistic regression** directly estimates a relevance probability.
- **SVM** learns a separating boundary between relevant and irrelevant examples.
- **Decision tree** learns interpretable feature-based decision rules but may overfit if it becomes too deep.

| Similarity-based retrieval | Classification-based recommendation |
| --- | --- |
| Directly compares a user profile with item profiles. | Learns a decision function from labelled examples. |
| Produces a similarity or distance score. | Produces a class or probability. |
| Usually needs little explicit training. | Requires training and labelled data. |
| Is often easy to explain through matching features. | Explainability depends on the classifier. |

The two can be combined by retrieving candidates through similarity and re-ranking them with a classifier.

### Q12. State the advantages and drawbacks of content-based recommendation. — 5 marks

**Advantages:**

- It does not require similar users.
- It can recommend a new item as soon as its features are available.
- Feature matches provide natural explanations.
- Each user can have an independent profile.
- It works in any domain with meaningful item features.

**Drawbacks:**

- It may overspecialize around the user’s past choices.
- Serendipity can be limited.
- Poor features produce poor recommendations.
- A new user has no reliable preference profile.
- Simple keywords may miss deeper meaning.
- It ignores useful collective-taste patterns.

Thus, content-based recommendation is particularly useful when descriptive item features are available or new items arrive frequently. Its central trade-off is independence from other users at the cost of overspecialization and weaker discovery beyond the user’s established feature profile.

---

# Unit 3 — Collaborative Filtering and Matrix Factorization

### Q1. What is collaborative filtering and what are its two main categories? — 2 marks [Asked in 2025 Midsem]

**Collaborative filtering (CF)** recommends items by learning from the preferences or interaction behaviour of many users. Its central assumption is that similar past behaviour can reveal similar future preferences. At the broadest level, CF is divided into **memory-based CF** and **model-based CF**. Memory-based CF directly uses the interaction matrix and has two major neighbourhood forms—**user-based CF** and **item-based CF**—whereas model-based CF first learns a predictive model. Thus, the complete hierarchy is CF → memory-based/model-based, with memory-based CF → user-based/item-based.

### Q2. Explain user-based collaborative filtering with its prediction formula. — 6 marks

User-based CF recommends items liked by users whose preference patterns resemble the target user’s pattern.

The method compares users only through valid co-rated items; an unknown rating should not automatically be entered as zero. Cosine similarity compares rating-vector direction, while Pearson similarity centres ratings and is useful when users employ the rating scale differently.

1. Construct the user-item matrix.
2. Select the target user.
3. Calculate similarity with other users using cosine or Pearson similarity.
4. Select the $k$ nearest reliable neighbours.
5. Collect items rated by neighbours but unseen by the target user.
6. Predict ratings, rank candidates and return the Top-N items.

The raw weighted prediction is:

$$
\hat{r}_{ui}=
\frac{\sum_{v\in N_i(u)}\operatorname{sim}(u,v)r_{vi}}
{\sum_{v\in N_i(u)}|\operatorname{sim}(u,v)|}
$$

If users have different rating habits, mean-centred prediction is safer:

$$
\hat{r}_{ui}=\bar{r}_u+
\frac{\sum_{v\in N_i(u)}\operatorname{sim}(u,v)(r_{vi}-\bar{r}_v)}
{\sum_{v\in N_i(u)}|\operatorname{sim}(u,v)|}
$$

Here, $\hat{r}_{ui}$ is the predicted rating, $N_i(u)$ is the set of selected neighbours who rated item $i$, $\operatorname{sim}(u,v)$ is user similarity, and $r_{vi}$ is neighbour $v$’s rating. The denominator normalizes the weighted total, while mean-centring corrects for generous or strict raters. User-based CF is intuitive, captures community taste and can explain “similar users liked this.” Its limitations are sparse overlap, new-user cold start, changing neighbourhoods and expensive user comparisons at large scale.

### Q3. Explain item-based collaborative filtering with a numerical example. — 6 marks

Item-based CF recommends candidate items that have interaction patterns similar to items already liked by the target user.

Each item is represented by its ratings or interactions across users. Two items are considered similar when the same users tend to respond to them in similar ways.

1. Represent each item by its ratings across users.
2. Calculate item-item similarity.
3. Identify items already preferred by the target user.
4. Find similar unseen candidates.
5. Predict scores, rank candidates and recommend the Top-N items.

The prediction formula is:

$$
\hat{r}_{ui}=
\frac{\sum_{j\in I(u)}\operatorname{sim}(i,j)r_{uj}}
{\sum_{j\in I(u)}|\operatorname{sim}(i,j)|}
$$

Here, $I(u)$ is the set of relevant items already rated by user $u$, $\operatorname{sim}(i,j)$ is item-item similarity, and $r_{uj}$ is the user’s known rating for item $j$. The denominator normalizes the similarity weights.

Suppose a user rated A as 5 and B as 4, while $\operatorname{sim}(C,A)=0.8$ and $\operatorname{sim}(C,B)=0.6$:

$$
\hat{r}_{uC}
=\frac{0.8(5)+0.6(4)}{0.8+0.6}
=\frac{6.4}{1.4}
=\boxed{4.57}
$$

Thus, item C should be ranked highly.

Item relationships are often stable and can be precomputed, making this approach efficient when users greatly outnumber items. Its main limitations are new-item cold start and weak similarity evidence for rarely rated items.

### Q4. Compare user-based and item-based collaborative filtering in terms of stability and serendipity. — 2 marks [Asked in 2025 Midsem]

User-based CF is usually less stable because users’ tastes and nearest neighbours can change frequently. However, it can provide greater serendipity because similar users may introduce relevant items outside the target user’s obvious history. Item-based CF is usually more stable because item relationships change more slowly and can be precomputed. It may offer lower serendipity because recommendations often remain close to items already consumed.

### Q5. In what scenario might item-based collaborative filtering be preferred over user-based filtering? Give one practical reason. — 2 marks [Asked in 2025 Midsem]

Item-based CF is preferred when a platform has far more users than items and its item catalogue is reasonably stable. Item-item similarities can then be computed offline and reused for many users. This reduces repeated online computation and produces more stable recommendations. For example, a large store can reuse “customers who bought A also bought B” relationships across millions of customers.

### Q6. What is the “cold start” problem in recommender systems, and why is it particularly problematic for collaborative filtering methods? — 2 marks [Asked in 2025 Midsem]

Cold start means that there is too little historical information to make reliable recommendations. A new user lacks enough interactions to reveal preferences, while a new item lacks enough interactions to establish similarity or popularity. A new system may lack both kinds of history. The problem is especially serious for collaborative filtering because CF depends directly on shared user-item interaction patterns rather than item descriptions.

### Q7. Calculate the predicted rating of Alice for Item5, which is missing, using the given data matrix. — 6 marks [Asked in 2025 Midsem]

| Name | Item1 | Item2 | Item3 | Item4 | Item5 |
| --- | --- | --- | --- | --- | --- |
| Alice | 5 | 4 | 1 | 4 | ? |
| U1 | 3 | 1 | 2 | 3 | 3 |
| U2 | 4 | 3 | 4 | 3 | 5 |
| U3 | 3 | 3 | 1 | 4 | 5 |

The paper does not state the method or $k$, so clearly state the assumption. Use user-based CF, cosine similarity over Items 1–4, all three users as neighbours, and raw weighted prediction.

**Step 1: Rating vectors**

$$
A=[5,4,1,4]
$$

$$
U_1=[3,1,2,3],\quad
U_2=[4,3,4,3],\quad
U_3=[3,3,1,4]
$$

**Step 2: Cosine similarities**

$$
\operatorname{sim}(A,U_1)
=\frac{33}{\sqrt{58}\sqrt{23}}
\approx0.904
$$

$$
\operatorname{sim}(A,U_2)
=\frac{48}{\sqrt{58}\sqrt{50}}
\approx0.891
$$

$$
\operatorname{sim}(A,U_3)
=\frac{44}{\sqrt{58}\sqrt{35}}
\approx0.977
$$

**Step 3: Weighted prediction**

$$
\hat{r}_{A,5}
=\frac{0.904(3)+0.891(5)+0.977(5)}
{0.904+0.891+0.977}
$$

$$
\hat{r}_{A,5}
\approx\frac{12.052}{2.772}
\approx\boxed{4.35}
$$

Therefore, Alice’s predicted rating for Item5 is approximately **4.35**, so it is a strong recommendation.

**Exam tip:** Use only co-rated entries for similarity and state the chosen method when the question is ambiguous.

### Q8. Compare memory-based and model-based collaborative filtering. — 5 marks

Both methods learn from interaction data, but they differ in when and how the main computation is performed. Memory-based methods retain neighbourhood relationships, whereas model-based methods compress interaction patterns into learned parameters.

| Memory-based CF | Model-based CF |
| --- | --- |
| Directly uses the interaction matrix and neighbourhoods. | Learns parameters or compact representations from interactions. |
| Needs little or no training. | Requires a training stage. |
| Predicts by aggregating neighbour ratings. | Predicts by applying the learned model. |
| May be expensive at prediction time. | Usually performs more work during training. |
| Is generally easier to explain. | Interpretability depends on the model. |
| Examples: user-based and item-based CF. | Main example: matrix factorization. |

Memory-based methods are attractive for small, explainable systems, while model-based methods are often preferred when repeated prediction must be fast after training. Other model-based families include clustering, classification, regression and probabilistic models, but only their broad purpose is required at this depth.

### Q9. Explain matrix factorization at normal midsem depth. — 5 marks

Matrix factorization is a model-based method that represents users and items with a small number of learned **latent factors**. It approximates the sparse rating matrix as:

$$
R\approx PQ^{T}
$$

If $R$ has $m$ users and $n$ items, then $P$ stores user-factor vectors and $Q$ stores item-factor vectors. The factors are hidden preference dimensions learned from observed interactions; they may roughly represent interests such as action or comedy, but need not have fixed human labels.

For user vector $p_u$ and item vector $q_i$, the predicted score is:

$$
\hat{r}_{ui}=p_u^{T}q_i
$$

For example, if $p_u=[0.9,0.8]$ and $q_i=[0.3,0.9]$:

$$
\hat{r}_{ui}=0.9(0.3)+0.8(0.9)=\boxed{0.99}
$$

During training, the system repeatedly adjusts user and item factors so that predicted scores become close to the observed ratings. After learning, dot products are used to estimate missing preferences and rank items.

### Q10. What are shilling or profile-injection attacks on collaborative recommenders? — 5 marks

A **shilling attack**, also called a **profile-injection attack**, occurs when an attacker creates fake profiles and submits artificial ratings to manipulate recommendations.

The mechanism is:

$$
\text{Fake accounts and ratings}
\rightarrow\text{Artificial similarity or patterns}
\rightarrow\text{Changed predictions and rankings}
$$

An attacker can assign ratings to ordinary **filler items** so that a fake profile resembles genuine users, and then assign an extreme rating to the **target item**. A **promotion** attack gives high target ratings to push an item upward; a **demotion** attack gives low ratings to reduce a competitor’s rank. The injected profiles can distort user neighbourhoods, item relationships or a learned model, thereby reducing accuracy, fairness and user trust.

High-level countermeasures are:

- **Anomaly detection:** Identify unusual rating distributions or coordinated activity.
- **Trust models:** Reduce the influence of new or suspicious accounts.
- **Robust aggregation:** Prevent a few extreme profiles from dominating predictions.
- **Behaviour analysis:** Examine timing and activity patterns in addition to rating values.

The goal of defence is to reduce malicious influence without incorrectly rejecting genuine users with unusual tastes.

### Q11. Explain with steps the working mechanism of Movie Recommendation System. — 6 marks [Asked in 2025 Midsem]

A movie recommender converts viewer information and movie information into ranked personalized suggestions.

1. **Collect data:** Obtain ratings, likes, watch history, searches, movie genres, tags and metadata.
2. **Preprocess data:** Clean interactions and construct user, movie or user-item representations.
3. **Choose an approach:** Use content-based filtering, user-based CF, item-based CF or a suitable model-based method.
4. **Generate candidates:** Remove already watched or unsuitable movies and find promising unseen movies.
5. **Predict scores:** Match user and movie features, aggregate neighbour ratings, or use a learned user-item score.
6. **Rank movies:** Sort candidates by predicted relevance.
7. **Recommend Top-N:** Display the best unseen movies with suitable explanations.
8. **Update from feedback:** Record new watches, ratings or skips and periodically update profiles, similarities or the model.

In user-based CF, candidates come from similar viewers. In item-based CF, they resemble movies already liked. In content-based recommendation, they match the user’s preferred genres, tags or other features.

---

# Last-Minute High-Priority Questions

## VERY HIGH

1. Define content-based and collaborative filtering and explain how they differ. **[2025]**
2. Define collaborative filtering and name its two main categories. **[2025]**
3. Explain cold start and why it is especially serious for CF. **[2025]**
4. Compare user-based and item-based CF in stability and serendipity. **[2025]**
5. Explain content-based recommendation and one advantage over CF. **[2025]**
6. Define MAE and RMSE; briefly distinguish [Precision@K](mailto:Precision@K) and [Recall@K](mailto:Recall@K). **[2025 theme]**
7. State when item-based CF is preferred and give a practical reason. **[2025]**
8. Solve the Alice–Item5 missing-rating numerical. **[2025]**
9. Solve the two-document Jaccard-distance and cosine-distance numerical. **[2025]**
10. Explain the complete movie-recommendation workflow. **[2025]**
11. Explain user-based CF with similarity and prediction.
12. Explain item-based CF with weighted prediction.

## HIGH

1. Define a recommender system and explain its need and pipeline.
2. Explain TF, IDF and TF-IDF with a numerical.
3. Explain content-based architecture, item profiles and user profiles.
4. Explain preprocessing, BoW, TF-IDF and n-grams.
5. Compare cosine, Euclidean, Jaccard and Pearson measures.
6. Solve average, weighted and dynamic user-profile numericals.
7. Explain classification-based recommendation, especially KNN and Naive Bayes.
8. Compare memory-based and model-based CF.
9. Explain matrix factorization using $R\approx PQ^{T}$ and dot-product prediction.
10. Explain shilling/profile-injection attacks and countermeasures.

## MEDIUM

1. Solve matrix-multiplication, covariance and covariance-matrix numericals.
2. Explain relevance feedback and solve a Rocchio update.
3. Discuss sparsity, scalability, popularity bias and other recommender issues.

# Last-Minute Numerical Checklist

| Numerical type | Key formula | One-line solving reminder |
| --- | --- | --- |
| TF-IDF | $TFIDF=TF\times\log(N/df)$ | State the log base, calculate IDF, then multiply by TF. |
| Matrix multiplication | $c_{ij}=\sum_{k=1}^{n}a_{ik}b_{kj}$ | Check inner dimensions, then use a row-column dot product for each entry. |
| Sample covariance | $\operatorname{Cov}(X,Y)=\dfrac{1}{n-1}\sum_i(x_i-\bar{x})(y_i-\bar{y})$ | Find means, deviations and paired products before dividing by $n-1$. |
| Covariance matrix | $\Sigma_{ij}=\operatorname{Cov}(X_i,X_j)$ | Put variances on the diagonal and symmetric covariances off the diagonal. |
| Cosine similarity/distance | $\cos(A,B)=\dfrac{A\cdot B}{\|A\|\|B\|}$; $d=1-\cos$ | Use the same feature order and calculate dot product and both magnitudes. |
| Euclidean distance | $d(A,B)=\sqrt{\sum_j(a_j-b_j)^2}$ | Subtract corresponding coordinates, square, add and take the square root. |
| Jaccard similarity/distance | $J=\dfrac{|A\cap B|}{|A\cup B|}$; $d_J=1-J$ | Count intersection and union after stating tokenization assumptions. |
| Pearson similarity | $\operatorname{sim}(u,v)=\dfrac{\sum_i(r_{ui}-\bar{r}_u)(r_{vi}-\bar{r}_v)}{\sqrt{\sum_i(r_{ui}-\bar{r}_u)^2}\sqrt{\sum_i(r_{vi}-\bar{r}_v)^2}}$ | Use common ratings, centre both users and normalize the deviation product. |
| Precision, Recall and F1 | $P=\dfrac{TP}{TP+FP}$, $R=\dfrac{TP}{TP+FN}$, $F_1=\dfrac{2PR}{P+R}$ | Separate relevant retrieved items from all retrieved and all relevant items. |
| [Precision@K](mailto:Precision@K) and [Recall@K](mailto:Recall@K) | $P@K=\dfrac{rel_K}{K}$, $R@K=\dfrac{rel_K}{R_{\text{total}}}$ | Count relevant items in the first $K$ positions before forming each denominator. |
| MAE and RMSE | $MAE=\dfrac{1}{n}\sum|r-\hat{r}|$, $RMSE=\sqrt{\dfrac{1}{n}\sum(r-\hat{r})^2}$ | Compute each error; RMSE squares before averaging and rooting. |
| Rocchio update | $q_{\text{new}}=\alpha q_0+\beta\bar D_r-\gamma\bar D_{nr}$ | Average relevant and non-relevant vectors before applying weights. |
| Sparsity | $1-\dfrac{\text{observed entries}}{\text{users}\times\text{items}}$ | Divide observed by possible interactions and subtract the density from 1. |
| Average user profile | $P_u=\dfrac{1}{|L_u|}\sum_{i\in L_u}x_i$ | Add corresponding feature entries and divide by the number of liked items. |
| Weighted user profile | $P_u=\dfrac{\sum_iw_{ui}x_i}{\sum_i|w_{ui}|}$ | Weight each item vector, add component-wise and normalize by total weight. |
| Dynamic profile update | $P_u^{(t+1)}=(1-\alpha)P_u^{(t)}+\alpha x_i$ | Retain the old-profile share and add the recent item’s weighted contribution. |
| Logistic probability | $P(y=1\mid X)=\dfrac{1}{1+e^{-z}}$ | Calculate the linear score $z$, apply the sigmoid, then compare with the threshold. |
| Alice–Item5 user-based rating [2025] | $\hat{r}_{ui}=\dfrac{\sum_v\operatorname{sim}(u,v)r_{vi}}{\sum_v|\operatorname{sim}(u,v)|}$ | Use co-rated Items 1–4, calculate all stated similarities, then weight Item5 ratings. |
| Mean-centred user rating | $\hat{r}_{ui}=\bar{r}_u+\dfrac{\sum_v\operatorname{sim}(u,v)(r_{vi}-\bar{r}_v)}{\sum_v|\operatorname{sim}(u,v)|}$ | Centre each neighbour’s rating, form the weighted deviation, then add the target user’s mean. |
| Item-based rating | $\hat{r}_{ui}=\dfrac{\sum_j\operatorname{sim}(i,j)r_{uj}}{\sum_j|\operatorname{sim}(i,j)|}$ | Weight the user’s known ratings by similarity to the target item. |
| Matrix-factor score | $\hat{r}_{ui}=p_u^{T}q_i$ | Multiply matching latent-factor entries and add. |