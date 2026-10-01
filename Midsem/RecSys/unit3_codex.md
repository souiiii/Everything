# Codex Rec Sys UNIT_3

# Unit 3: Collaborative Filtering and Matrix Factorization

## Exam priority from the 2025 midsem

The 2025 paper directly tested:

- the meaning and two main categories of collaborative filtering;
- cold start and why it is especially serious for collaborative filtering;
- user-based versus item-based CF in terms of stability and serendipity;
- when item-based CF is preferred;
- prediction of a missing rating from a user-item matrix.

These are the highest-priority topics in this unit. Matrix factorization, model-based methods, and attacks are also syllabus topics and should be prepared at normal 5/10-mark depth.

---

## 1. Collaborative Filtering Basics

### 1.1 Meaning

**Collaborative filtering (CF)** recommends items by using the preferences or interaction behaviour of many users. It does not require detailed descriptions of the items.

Its basic assumption is:

> Users who behaved similarly in the past are likely to behave similarly in the future, and items that received similar responses are likely to be related.
> 

Examples of interactions are movie ratings, product purchases, song plays, clicks, likes and video views.

The general process is:

`Past interactions -> discover similarities/patterns -> predict unknown preferences -> rank items -> recommend Top-N items`

### 1.2 User-item interaction matrix

CF data are commonly stored in a matrix $R=[r_{ui}]$:

- Each row represents a user $u$.
- Each column represents an item $i$.
- $r_{ui}$ is the interaction of user $u$ with item $i$.
- A blank or `?` represents an unknown interaction to be predicted. It is not an actual rating of zero.

| User | Movie A | Movie B | Movie C | Movie D |
| --- | --- | --- | --- | --- |
| $U_1$ | 5 | 4 | 5 | ? |
| $U_2$ | 5 | 4 | 4 | 5 |
| $U_3$ | 1 | 2 | 1 | 4 |

Since $U_1$ and $U_2$ have similar known ratings and $U_2$ likes Movie D, the system may recommend D to $U_1$.

### 1.3 Explicit and implicit feedback

| Feedback | Meaning | Examples |
| --- | --- | --- |
| Explicit | The user deliberately states a preference. | A 1–5 rating or like/dislike. |
| Implicit | Preference is inferred from behaviour. | A click, purchase, watch or play. |

In implicit data, `0` usually means **no interaction was observed**, not that the user dislikes the item.

### 1.4 Main categories of CF

1. **Memory-based or neighbourhood-based CF** directly uses the interaction matrix and similarities. Its two forms are user-based CF and item-based CF.
2. **Model-based CF** first learns a predictive model from the interaction data. Matrix factorization is its most important example in this unit.

---

## 2. Similarity Measures Used in CF

Similarity decides which users or items should influence a prediction.

### Important missing-value convention

For explicit ratings, calculate similarity using items rated by **both** users, or users who rated **both** items. Do not automatically replace `?` by zero. In a numerical, clearly state the convention being used.

Let $I_{uv}$ be the set of items co-rated by users $u$ and $v$.

### 2.1 Cosine similarity

Cosine similarity measures the angle between two rating vectors:

$$
\operatorname{sim}(u,v)=
\frac{\sum_{i\in I_{uv}}r_{ui}r_{vi}}
{\sqrt{\sum_{i\in I_{uv}}r_{ui}^{2}}\sqrt{\sum_{i\in I_{uv}}r_{vi}^{2}}}
$$

- A value near $1$ indicates similar rating directions.
- It is simple and commonly used for both user-user and item-item similarity.
- It does not directly remove differences in users’ rating scales.

**Small example:** For $A=[5,4,5,1]$ and $B=[4,5,4,2]$,

$$
A\cdot B=62,\quad \lVert A\rVert=\sqrt{67},\quad \lVert B\rVert=\sqrt{61}
$$

$$
\operatorname{sim}(A,B)=\frac{62}{\sqrt{67}\sqrt{61}}\approx0.970
$$

Thus, the two users or items are highly similar.

### 2.2 Pearson correlation

Pearson correlation first centres ratings around each user’s mean. It is useful when users use the rating scale differently.

$$
\operatorname{sim}(u,v)=
\frac{\sum_{i\in I_{uv}}(r_{ui}-\bar{r}_u)(r_{vi}-\bar{r}_v)}
{\sqrt{\sum_{i\in I_{uv}}(r_{ui}-\bar{r}_u)^{2}}
\sqrt{\sum_{i\in I_{uv}}(r_{vi}-\bar{r}_v)^{2}}}
$$

Here, $\bar{r}_u$ and $\bar{r}_v$ are the relevant mean ratings of users $u$ and $v$. State whether the means are over all available ratings or only the co-rated set if the question is ambiguous.

Its range is $[-1,1]$: $+1$ means perfect positive relation, $0$ means no linear relation, and $-1$ means opposite rating patterns.

### 2.3 Jaccard similarity

Jaccard similarity is mainly useful for binary or implicit interaction sets:

$$
J(u,v)=\frac{|I_u\cap I_v|}{|I_u\cup I_v|}
$$

For example, if $I_u=\{A,B,C\}$ and $I_v=\{B,C,D,E\}$, then

$$
J(u,v)=\frac{2}{5}=0.4
$$

Jaccard considers whether an interaction occurred, not the numerical rating value.

---

## 3. User-Based Collaborative Filtering

### 3.1 Idea

User-based CF asks:

> Which users are similar to the target user, and what did those users like?
> 

It recommends items liked by the target user’s nearest neighbours.

### 3.2 Algorithm

1. Construct the user-item matrix.
2. Select the target user.
3. Calculate the target user’s similarity with other users.
4. Select the $k$ nearest reliable neighbours.
5. Collect candidate items rated or used by those neighbours but not by the target user.
6. Predict a score for every candidate item.
7. Rank the candidates and return the Top-N items.

### 3.3 Raw weighted prediction

For target user $u$ and item $i$:

$$
\hat{r}_{ui}=
\frac{\sum_{v\in N_i(u)}\operatorname{sim}(u,v)r_{vi}}
{\sum_{v\in N_i(u)}|\operatorname{sim}(u,v)|}
$$

where:

- $N_i(u)$ is the selected neighbourhood of users who rated item $i$;
- $r_{vi}$ is neighbour $v$’s rating for $i$;
- $\hat{r}_{ui}$ is the predicted rating.

If only positive similarities are selected, the absolute signs in the denominator do not change the result.

### 3.4 Mean-centred prediction

Raw prediction can be unfair when one user rates everything generously and another rates strictly. Mean-centred prediction adjusts for this:

$$
\hat{r}_{ui}=\bar{r}_u+
\frac{\sum_{v\in N_i(u)}\operatorname{sim}(u,v)(r_{vi}-\bar{r}_v)}
{\sum_{v\in N_i(u)}|\operatorname{sim}(u,v)|}
$$

**Worked example:** Suppose $\bar{r}_u=3.5$. Two neighbours have:

$$
\operatorname{sim}(u,v_1)=0.9,\quad \bar{r}_{v_1}=3,\quad r_{v_1,i}=5
$$

$$
\operatorname{sim}(u,v_2)=0.7,\quad \bar{r}_{v_2}=4,\quad r_{v_2,i}=4
$$

Then:

$$
\hat{r}_{ui}=3.5+\frac{0.9(5-3)+0.7(4-4)}{0.9+0.7}
=3.5+1.125=4.625
$$

The high predicted rating makes the item a strong recommendation candidate.

### 3.5 Advantages and limitations

**Advantages:** It is intuitive, captures community preferences, needs no item descriptions, and can explain a result as “users like you liked this item.”

**Limitations:** It becomes expensive with many users, suffers from sparse data and new-user cold start, and its neighbourhood can change when user tastes or interactions change.

---

## 4. Item-Based Collaborative Filtering

### 4.1 Idea

Item-based CF asks:

> Which items are similar to the items that the target user already likes?
> 

An item is represented by its ratings or interactions across users. Two items are similar if users tend to respond to them similarly.

### 4.2 Algorithm

1. Construct the user-item matrix.
2. Represent each item by its user-interaction vector.
3. Calculate item-item similarities.
4. Identify items already preferred by the target user.
5. Find unseen candidate items similar to those items.
6. Calculate predicted scores for candidates.
7. Rank the candidates and return the Top-N items.

### 4.3 Item-based prediction

For a target item $i$:

$$
\hat{r}_{ui}=
\frac{\sum_{j\in I(u)}\operatorname{sim}(i,j)r_{uj}}
{\sum_{j\in I(u)}|\operatorname{sim}(i,j)|}
$$

where $I(u)$ is the set of relevant items already rated by user $u$.

**Worked example:** A user rated item A as 5 and item B as 4. For candidate C:

$$
\operatorname{sim}(C,A)=0.8,\qquad \operatorname{sim}(C,B)=0.6
$$

$$
\hat{r}_{u,C}=\frac{0.8(5)+0.6(4)}{0.8+0.6}
=\frac{6.4}{1.4}\approx4.57
$$

Therefore, C should be ranked highly.

### 4.4 Advantages and limitations

**Advantages:** Item relationships are often stable, similarities can be precomputed, and the method scales well when the number of items is much smaller than the number of users. It also gives an explanation such as “because you liked A, you may like C.”

**Limitations:** New items have too few interactions to obtain reliable neighbours. A very large or rapidly changing catalogue also makes similarity maintenance difficult. Recommendations may stay close to previously consumed items.

---

## 5. User-Based versus Item-Based CF

| Basis | User-based CF | Item-based CF |
| --- | --- | --- |
| Main question | Who is similar to the target user? | Which items are similar? |
| Similarity | User-user | Item-item |
| Recommendation source | Items liked by similar users | Items similar to those already liked |
| Basic unit | User | Item |
| Stability | Usually less stable because users and their tastes change. | Usually more stable because item relationships change more slowly. |
| Serendipity | Often higher because neighbours may introduce relevant items unlike the user’s obvious history. | Often lower because it remains near items already consumed. |
| Scalability | Difficult when the user population is extremely large. | Often better when users greatly outnumber items. |
| Precomputation | User neighbourhoods may change frequently. | Item similarities can often be precomputed and reused. |
| Main cold-start weakness | New user | New item |
| Explanation | “Similar users liked X.” | “Because you liked A, try X.” |

**Stability** means how little the learned neighbourhoods change as new interactions arrive. **Serendipity** means recommending something both relevant and pleasantly unexpected. The table gives common tendencies, not absolute guarantees.

Both methods can suffer from both new-user and new-item cold start; the table shows their strongest direct dependence.

### When is item-based CF preferred?

Item-based CF is preferred when:

- the system has far more users than items;
- the item catalogue is reasonably stable;
- item similarities can be computed offline and reused for many users;
- fast online recommendation is important.

For example, a large shopping platform with millions of users and a comparatively stable catalogue can precompute “customers who interacted with A also interacted with B.” This is more stable and usually cheaper than repeatedly finding neighbours among millions of users.

---

## 6. High-Priority Missing-Rating Numerical

The following is the data pattern used in the 2025 midsem paper:

| Name | Item1 | Item2 | Item3 | Item4 | Item5 |
| --- | --- | --- | --- | --- | --- |
| Alice | 5 | 4 | 1 | 4 | ? |
| U1 | 3 | 1 | 2 | 3 | 3 |
| U2 | 4 | 3 | 4 | 3 | 5 |
| U3 | 3 | 3 | 1 | 4 | 5 |

The question does not state a method or $k$. A clear exam solution should state an assumption. Use **user-based CF, cosine similarity over Items 1–4, all three users as neighbours, and raw weighted prediction**.

### Step 1: Represent the co-rated vectors

$$
A=[5,4,1,4]
$$

$$
U_1=[3,1,2,3],\quad U_2=[4,3,4,3],\quad U_3=[3,3,1,4]
$$

### Step 2: Calculate cosine similarities

$$
\operatorname{sim}(A,U_1)=
\frac{33}{\sqrt{58}\sqrt{23}}\approx0.904
$$

$$
\operatorname{sim}(A,U_2)=
\frac{48}{\sqrt{58}\sqrt{50}}\approx0.891
$$

$$
\operatorname{sim}(A,U_3)=
\frac{44}{\sqrt{58}\sqrt{35}}\approx0.977
$$

### Step 3: Predict Alice’s Item5 rating

$$
\hat{r}_{A,5}=
\frac{0.904(3)+0.891(5)+0.977(5)}{0.904+0.891+0.977}
$$

$$
\hat{r}_{A,5}\approx\frac{12.052}{2.772}\approx\boxed{4.35}
$$

Therefore, Item5 is a strong recommendation for Alice. A different stated method, such as Pearson similarity, mean-centred prediction or a different $k$, can produce a different valid result.

### Safe numerical procedure

1. Mark the target user and missing entry.
2. Use only valid common entries for similarity.
3. Show each similarity and select the stated number of neighbours.
4. Use only neighbours who rated the target item.
5. Substitute in the correct prediction formula.
6. Round only the final result and keep it within the allowed rating scale.

---

## 7. Major Problems in Collaborative Filtering

### 7.1 Data sparsity

Real interaction matrices contain very few observed entries compared with all possible user-item pairs. Users or items may then have too little overlap for reliable similarity.

### 7.2 Cold start

CF depends directly on historical interactions, so a lack of history is especially damaging.

| Case | Problem in CF |
| --- | --- |
| New-user cold start | The user has rated almost nothing, so similar users and personal preferences cannot be identified reliably. |
| New-item cold start | Almost nobody has interacted with the item, so its similarity and attractiveness cannot be estimated reliably. |
| New-system cold start | The system has too little user and item history to discover useful collaborative patterns. |

Pure CF cannot infer much until interactions are collected. This is why cold start is more serious for CF than for a method that has usable item descriptions.

### 7.3 Scalability

A simplified all-pairs comparison may require about $O(U^{2})$ user comparisons or $O(I^{2})$ item comparisons. Precomputation, sparse operations and nearest-neighbour search are therefore important at scale.

### 7.4 Popularity bias

Popular items receive more interactions, which makes them more likely to be recommended and creates a feedback loop:

`Popular -> more interactions -> more recommendations -> still more popular`

This can reduce exposure of less popular or new items.

### 7.5 Other limitations

CF can also face changing user interests, privacy concerns, noisy ratings and deliberate manipulation through fake profiles.

---

## 8. Model-Based Collaborative Filtering

### 8.1 Meaning

Model-based CF learns a prediction function from observed interactions:

$$
\hat{r}_{ui}=f(u,i;\Theta)
$$

Here, $\Theta$ represents learned model parameters. Training is performed first; the learned model is then used to predict unknown interactions.

### 8.2 Memory-based versus model-based methods

| Feature | Memory-based CF | Model-based CF |
| --- | --- | --- |
| Basic idea | Find similar users or items. | Learn a predictive model. |
| Representation | Original user-item matrix and neighbourhoods. | Learned parameters or compact representations. |
| Training | Little or none. | Required. |
| Prediction | Aggregate neighbours’ ratings. | Apply the learned model. |
| Computation | Often heavier at prediction time. | Often heavier during training. |
| Scalability | Can be difficult on very large matrices. | Generally better after training. |
| Interpretation | Usually easier. | Depends on the model. |
| Example | User-based or item-based CF. | Matrix factorization. |

### 8.3 Main model families — brief overview

Only an overview is required because the supplied material treats these families briefly.

| Family | Basic prediction idea | Examples |
| --- | --- | --- |
| Matrix factorization | Learn compact latent vectors for users and items. | Regularized matrix factorization. |
| Clustering | Group similar users or items and use their group pattern. | K-means, hierarchical clustering. |
| Classification | Predict whether a user will interact with an item. | Logistic regression, decision tree, Naive Bayes. |
| Regression | Predict a numerical rating or preference score. | Linear or ridge regression. |
| Probabilistic models | Represent preferences and uncertainty probabilistically. | Probabilistic matrix factorization. |
| Neural models | Learn nonlinear user-item interaction patterns. | Neural collaborative filtering, autoencoders. |

Do not present every listed family as if its complete algorithm has been taught. Matrix factorization is the main model-based method for this unit.

---

## 9. Matrix Factorization

### 9.1 Core idea

Matrix factorization approximates the large, sparse rating matrix by two low-dimensional matrices:

$$
R_{m\times n}\approx P_{m\times k}Q_{n\times k}^{T},\qquad k\ll m,n
$$

- $m$: number of users;
- $n$: number of items;
- $k$: number of latent factors;
- $P$: user-factor matrix;
- $Q$: item-factor matrix.

The factors are hidden preference dimensions learned from data. In a movie system they may roughly correspond to action, comedy or romance preference, but they do not have to receive clear human labels.

For user vector $p_u$ and item vector $q_i$:

$$
\hat{r}_{ui}=p_u^{T}q_i=\sum_{f=1}^{k}p_{uf}q_{if}
$$

The dot product is high when the user’s latent preferences match the item’s latent characteristics.

### 9.2 Worked latent-factor prediction

Suppose:

$$
p_u=[0.9,0.8],\qquad q_i=[0.3,0.9]
$$

Then:

$$
\hat{r}_{ui}=0.9(0.3)+0.8(0.9)=0.27+0.72=\boxed{0.99}
$$

This is an illustrative latent score. In an actual rating system, factors are learned so that predictions match the rating scale.

### 9.3 Learning the factors

Only observed entries are used in the training loss. Let $\Omega$ be the set of observed user-item pairs:

$$
\min_{P,Q}\sum_{(u,i)\in\Omega}(r_{ui}-p_u^{T}q_i)^{2}
$$

Missing entries are targets for prediction; they should not be treated as ordinary zero ratings.

### 9.4 Regularization

A model with too much freedom may memorize the observed ratings. L2 regularization discourages excessively large factor values:

$$
\min_{P,Q}\sum_{(u,i)\in\Omega}(r_{ui}-p_u^{T}q_i)^{2}
+\lambda\left(\sum_{u}\lVert p_u\rVert^{2}+\sum_{i}\lVert q_i\rVert^{2}\right)
$$

$\lambda$ controls regularization. Too little can cause overfitting; too much can cause underfitting.

### 9.5 Bias-aware prediction

Users and items can have systematic rating tendencies. A better prediction is:

$$
\hat{r}_{ui}=\mu+b_u+b_i+p_u^{T}q_i
$$

where $\mu$ is the global mean, $b_u$ is the user bias, and $b_i$ is the item bias.

**Example:** If $\mu=3.5$, $b_u=0.5$, $b_i=0.7$, and $p_u^{T}q_i=0.6$, then

$$
\hat{r}_{ui}=3.5+0.5+0.7+0.6=5.3
$$

For a 1–5 scale, the output may be clipped to 5.

### 9.6 Learning with stochastic gradient descent (SGD)

SGD is a common method for learning the factors. For each observed rating, it:

1. predicts the rating using the current user and item factors;
2. computes the prediction error;
3. slightly adjusts the factors and biases to reduce that error while applying regularization;
4. repeats this process for several passes through the observed data.

After training, the learned factors are used to predict missing ratings and rank items. Detailed SGD update equations are not needed at normal midsem depth unless specifically asked.

### 9.7 ALS and SVD — conceptual distinction

**Alternating Least Squares (ALS)** alternately fixes the item factors and learns the user factors, then fixes the user factors and learns the item factors. It is simply an alternative to SGD at this depth.

Classical **Singular Value Decomposition (SVD)** factorizes a complete matrix, whereas recommender matrices contain many missing entries. **Funk-SVD** informally refers to learning user and item factors directly from observed ratings. These terms are supporting concepts; detailed derivations are not required.

### 9.8 Advantages and limitations

**Advantages:** Matrix factorization gives compact representations, captures hidden preference patterns, handles sparse data better than direct neighbourhood matching in many cases, and supports fast prediction after training.

**Limitations:** It still needs interaction data, so cold start remains; training and hyperparameter selection are required; latent factors may be hard to interpret; and poor choices of $k$ or $\lambda$ can underfit or overfit.

---

## 10. Attacks on Collaborative Recommender Systems

### 10.1 Shilling or profile-injection attack

A **shilling attack**, also called a **profile-injection attack**, occurs when an attacker creates fake accounts or profiles and submits artificial ratings to manipulate recommendations.

The basic chain is:

`Fake ratings -> artificial similarity/patterns -> manipulated predicted scores -> manipulated recommendations`

### 10.2 Main goals

- **Promotion:** Give very high ratings, such as repeated 5s, to push a target product upward.
- **Demotion:** Give very low ratings, such as repeated 1s, to reduce the ranking of a competitor’s product.

An attacker may rate several other items so that fake profiles appear similar to genuine users. The false profile can then influence neighbourhoods or the learned model.

### 10.3 Effects

Attacks can distort similarities and predicted ratings, unfairly change item exposure, reduce recommendation quality and damage users’ trust in the system.

### 10.4 Countermeasures

The supplied material gives the following high-level countermeasures:

- **Anomaly detection:** Flag unusual rating patterns or sudden coordinated activity.
- **Trust models:** Give less influence to accounts with weak or suspicious history.
- **Robust aggregation:** Prevent a small set of extreme ratings from dominating a prediction.
- **User-behaviour analysis:** Examine account and interaction behaviour in addition to rating values.

For this midsem, know the definition, promotion/demotion idea, manipulation chain, effects and these countermeasure names. Detailed attack taxonomies and detection algorithms are not required by the supplied material.

---

## 11. Movie Recommendation System Using CF

A concise 6-mark working mechanism is:

1. Collect explicit ratings or implicit actions such as views and likes.
2. Construct the user-movie interaction matrix.
3. Choose user-based, item-based or model-based CF.
4. Learn user-user/item-item similarities or latent factors.
5. Generate candidate movies not already watched by the target user.
6. Predict a score for each candidate.
7. Rank candidates and recommend the Top-N movies.
8. Record new feedback and update similarities or retrain the model periodically.

In user-based CF, the candidates come from similar viewers. In item-based CF, they are movies similar to those the user already liked. In matrix factorization, they receive high scores when user and movie latent factors match.