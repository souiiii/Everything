# Chat-GPT Rec Sys UNIT_3

# Unit 3 — Collaborative Filtering and Matrix Factorization

## 1. Collaborative Filtering: Core Idea

**Collaborative Filtering (CF)** recommends items by learning from the behaviour of many users.

Unlike content-based recommendation, it does not primarily depend on item descriptions. Instead, it looks at patterns in the user-item interaction matrix.

The basic idea is:

> If two users behaved similarly in the past, they may prefer similar items in the future. Likewise, if two items receive similar responses from users, they may be related.
> 

Typical interactions include ratings, purchases, clicks, likes, watch history, and song plays.

A CF system therefore follows this broad flow:

$$
\text{Past interactions}
\rightarrow
\text{Find similarities or patterns}
\rightarrow
\text{Predict unknown preferences}
\rightarrow
\text{Recommend items}
$$

### User-Item Interaction Matrix

Collaborative filtering data are usually stored in a matrix:

$$
R=[r_{ui}]
$$

where:

- each row represents a user
- each column represents an item
- $r_{ui}$ represents user $u$’s interaction with item $i$

Example:

| User | Movie A | Movie B | Movie C | Movie D |
| --- | --- | --- | --- | --- |
| $U_1$ | 5 | 4 | 5 | ? |
| $U_2$ | 5 | 4 | 4 | 5 |
| $U_3$ | 1 | 2 | 1 | 4 |

The unknown value `?` is something the recommender may try to predict.

A missing value should not automatically be treated as a rating of zero.

### Explicit and Implicit Feedback

**Explicit feedback** is directly given by the user, such as a rating or like/dislike.

**Implicit feedback** is inferred from behaviour, such as a click, purchase, watch, or play.

In implicit data, a zero generally means **no interaction was observed**, not necessarily that the user disliked the item.

---

## 2. Main Types of Collaborative Filtering

At the broadest level, CF can be divided into:

1. **Memory-based CF**, which directly uses similarities between users or items.
2. **Model-based CF**, which learns a predictive model from the interaction data.

For the 2025 midsem paper, the especially important practical distinction is between the two memory-based methods:

- **User-based collaborative filtering**
- **Item-based collaborative filtering**

These two must be understood clearly because the previous paper directly asked about them.

---

## 3. Similarity Measures Used in Collaborative Filtering

Similarity determines which users or items should influence a recommendation.

For explicit ratings, compare only the entries that are valid for both users or both items.

### 3.1 Cosine Similarity

Cosine similarity compares the direction of two rating vectors.

$$
sim(u,v)
=
\frac{
\sum_{i\in I_{uv}} r_{ui}r_{vi}
}{
\sqrt{\sum_{i\in I_{uv}} r_{ui}^{2}}
\sqrt{\sum_{i\in I_{uv}} r_{vi}^{2}}
}
$$

where $I_{uv}$ is the set of items rated by both users.

A value closer to $1$ means the two rating patterns are more similar.

For example, for:

$$
A=[5,4,5,1]
$$

and

$$
B=[4,5,4,2]
$$

the similarity is approximately:

$$
sim(A,B)\approx0.970
$$

so the two vectors are highly similar.

### 3.2 Pearson Correlation

Pearson is similar in spirit to cosine similarity, but it first subtracts each user’s mean rating.

This is useful when one user usually gives high ratings and another usually gives low ratings.

The formula is:

$$
sim(u,v)=
\frac{
\sum_{i\in I_{uv}}
(r_{ui}-\bar r_u)(r_{vi}-\bar r_v)
}{
\sqrt{
\sum_{i\in I_{uv}}(r_{ui}-\bar r_u)^2
}
\sqrt{
\sum_{i\in I_{uv}}(r_{vi}-\bar r_v)^2
}
}
$$

For the midsem, the important idea is:

> **Pearson compares rating patterns after removing each user’s average rating.**
> 

You do not need to prioritize a full Pearson numerical unless specifically asked.

### 3.3 Jaccard Similarity

Jaccard similarity is useful for binary or implicit interactions.

$$
J(u,v)
=
\frac{|I_u\cap I_v|}{|I_u\cup I_v|}
$$

If:

$$
I_u=\{A,B,C\}
$$

and:

$$
I_v=\{B,C,D,E\}
$$

then:

$$
J(u,v)=\frac{2}{5}=0.4
$$

Jaccard only checks whether interactions occurred. It does not use the numerical rating values.

---

## 4. User-Based Collaborative Filtering

User-based CF asks:

> Which users are most similar to the target user, and what did those users like?
> 

The system finds users with similar rating patterns and uses their preferences to recommend unseen items.

### Working Procedure

1. Build the user-item matrix.
2. Select the target user.
3. Compute similarity between the target user and other users.
4. Select the nearest neighbours.
5. Find items those neighbours liked but the target user has not rated.
6. Predict a rating or score for those items.
7. Rank the candidates and recommend the best ones.

### Weighted Prediction

A common prediction formula is:

$$
\hat r_{ui}
=
\frac{
\sum_{v\in N_i(u)}
sim(u,v)\,r_{vi}
}{
\sum_{v\in N_i(u)}
|sim(u,v)|
}
$$

where:

- $u$ = target user
- $i$ = target item
- $N_i(u)$ = selected neighbours who rated item $i$
- $r_{vi}$ = neighbour $v$’s rating of item $i$

The idea is simple: **similar users should have more influence on the prediction**.

### Mean-Centred Prediction

If users use the rating scale differently, the ratings can be centred around their averages:

$$
\hat r_{ui}
=
\bar r_u
+
\frac{
\sum_{v\in N_i(u)}
sim(u,v)(r_{vi}-\bar r_v)
}{
\sum_{v\in N_i(u)}
|sim(u,v)|
}
$$

For the exam, understand why this is used: it reduces the effect of generous or strict raters.

### Advantages

User-based CF is intuitive and can capture community taste well.

It can also explain a recommendation naturally:

> Similar users liked this item.
> 

### Limitations

It becomes expensive when there are very many users and can suffer badly from sparse data and new-user cold start.

User neighbourhoods may also change frequently as preferences evolve.

---

## 5. Item-Based Collaborative Filtering

Item-based CF asks:

> Which items are similar to the items the target user already likes?
> 

Here, each item is represented by the pattern of user ratings it receives.

Two items are considered similar when users tend to respond to them in similar ways.

### Working Procedure

1. Build the user-item matrix.
2. Represent each item using ratings across users.
3. Compute item-item similarities.
4. Identify items already liked by the target user.
5. Find similar unseen items.
6. Predict a score for each candidate.
7. Rank and recommend the best items.

### Item-Based Prediction

A typical prediction is:

$$
\hat r_{ui}
=
\frac{
\sum_{j\in I(u)}
sim(i,j)\,r_{uj}
}{
\sum_{j\in I(u)}
|sim(i,j)|
}
$$

where $I(u)$ is the set of relevant items already rated by user $u$.

### Worked Example

Suppose the user rated:

$$
A=5,\qquad B=4
$$

and for candidate item $C$:

$$
sim(C,A)=0.8,\qquad sim(C,B)=0.6
$$

Then:

$$
\hat r_{uC}
=
\frac{0.8(5)+0.6(4)}{0.8+0.6}
$$

$$
\hat r_{uC}
=
\frac{6.4}{1.4}
\approx4.57
$$

So item $C$ is a strong recommendation candidate.

### Advantages

Item-item relationships are often more stable than user-user relationships.

Item similarities can also be precomputed, which makes recommendation faster when the user population is very large.

### Limitations

New items are difficult to recommend because they do not yet have enough interaction history.

Item-based CF can also become too narrow because it tends to recommend items similar to things the user already consumed.

---

## 6. User-Based vs Item-Based Collaborative Filtering

This comparison is especially important because the 2025 midsem directly tested it.

| Basis | User-Based CF | Item-Based CF |
| --- | --- | --- |
| Main question | Which users are similar? | Which items are similar? |
| Similarity | User-user | Item-item |
| Recommendation source | Items liked by similar users | Items similar to already liked items |
| Stability | Usually lower | Usually higher |
| Serendipity | Often higher | Often lower |
| Scalability | Harder with huge user populations | Often better when users greatly outnumber items |
| Precomputation | Harder because user neighbourhoods change | Easier because item similarities are more stable |
| Main cold-start weakness | New user | New item |

### Stability

**Stability** means how much the learned neighbourhoods change over time.

User-based CF is usually less stable because user tastes and behaviour change more often.

Item-based CF is usually more stable because relationships between items change more slowly.

### Serendipity

**Serendipity** means recommending something useful that the user did not obviously expect.

User-based CF often has higher serendipity because similar users may introduce items outside the target user’s direct history.

Item-based CF often has lower serendipity because it remains closer to items the user already consumed.

### When is Item-Based CF Preferred?

Item-based CF is often preferred when:

- there are far more users than items
- the catalogue is reasonably stable
- item similarities can be precomputed
- fast online recommendation is important

This is why item-based CF can be practical for large e-commerce platforms.

---

## 7. High-Priority Missing-Rating Numerical

The 2025 midsem directly asked for a missing-rating prediction.

Given:

| Name | Item1 | Item2 | Item3 | Item4 | Item5 |
| --- | --- | --- | --- | --- | --- |
| Alice | 5 | 4 | 1 | 4 | ? |
| U1 | 3 | 1 | 2 | 3 | 3 |
| U2 | 4 | 3 | 4 | 3 | 5 |
| U3 | 3 | 3 | 1 | 4 | 5 |

Assume:

- user-based CF
- cosine similarity
- Items 1-4 for similarity
- all three users as neighbours

### Step 1: Represent Alice and the neighbours

$$
A=[5,4,1,4]
$$

$$
U_1=[3,1,2,3]
$$

$$
U_2=[4,3,4,3]
$$

$$
U_3=[3,3,1,4]
$$

### Step 2: Compute cosine similarities

$$
sim(A,U_1)\approx0.904
$$

$$
sim(A,U_2)\approx0.891
$$

$$
sim(A,U_3)\approx0.977
$$

### Step 3: Predict Alice’s rating for Item5

$$
\hat r_{A,5}
=
\frac{
0.904(3)+0.891(5)+0.977(5)
}{
0.904+0.891+0.977
}
$$

$$
\hat r_{A,5}
\approx4.35
$$

So Alice is predicted to rate Item5 highly.

### Safe Exam Method

Whenever a question asks for a missing rating:

1. identify the target user and missing item
2. choose or state the similarity method
3. compare only valid common ratings
4. calculate similarities
5. use neighbours who rated the target item
6. apply the prediction formula
7. round the final answer sensibly

If the question does not specify the method or $k$, clearly state your assumption.

---

## 8. Major Problems in Collaborative Filtering

### Data Sparsity

Most users interact with only a small fraction of available items.

This creates a sparse matrix, which makes user-user or item-item similarity less reliable.

### Cold Start

Cold start is especially serious in CF because collaborative filtering depends directly on interaction history.

**New-user cold start:** there is too little data to identify similar users or personal preferences.

**New-item cold start:** too few users have interacted with the item to estimate reliable similarity.

**New-system cold start:** the entire system has too little history to discover collaborative patterns.

This is why cold start hurts pure CF more than content-based recommendation, where item features may still be available.

### Scalability

Computing similarities among very large numbers of users or items can become expensive.

Item-based CF often helps because item similarities can be precomputed and reused.

### Popularity Bias

Popular items receive more interactions, which makes them more likely to be recommended again.

This creates a feedback loop:

$$
\text{Popular item}
\rightarrow
\text{More interactions}
\rightarrow
\text{More recommendations}
\rightarrow
\text{Even more popularity}
$$

This can reduce exposure of new or less-popular items.

---

## 9. Model-Based Collaborative Filtering

Memory-based CF directly compares users or items.

**Model-based CF** instead learns a predictive model from the interaction data.

We can write:

$$
\hat r_{ui}=f(u,i;\Theta)
$$

where $\Theta$ represents learned model parameters.

The main model-based method in this unit is **matrix factorization**.

### Memory-Based vs Model-Based CF

| Memory-Based CF | Model-Based CF |
| --- | --- |
| Finds similar users or items | Learns a predictive model |
| Uses the original interaction matrix directly | Uses learned parameters |
| Little or no training | Requires training |
| Prediction may be expensive | Prediction is often faster after training |
| Easier to interpret | May be less interpretable |
| Example: user-based or item-based CF | Example: matrix factorization |

Other model families such as clustering, classification, regression, probabilistic methods, and neural models only need to be remembered at a high level for this unit.

---

## 10. Matrix Factorization

Matrix factorization represents a large sparse rating matrix using smaller user and item factor matrices.

The idea is:

$$
R_{m\times n}
\approx
P_{m\times k}Q_{n\times k}^{T}
$$

where:

- $m$ = number of users
- $n$ = number of items
- $k$ = number of latent factors
- $P$ = user-factor matrix
- $Q$ = item-factor matrix

The latent factors represent hidden preference dimensions learned from data.

For a user vector $p_u$ and item vector $q_i$:

$$
\hat r_{ui}=p_u^Tq_i
$$

The dot product becomes high when the user’s hidden preferences match the item’s hidden characteristics.

### Simple Numerical

Suppose:

$$
p_u=[0.9,0.8]
$$

and:

$$
q_i=[0.3,0.9]
$$

Then:

$$
\hat r_{ui}
=
0.9(0.3)+0.8(0.9)
$$

$$
\hat r_{ui}
=
0.27+0.72
=
0.99
$$

This is a latent preference score.

### Learning the Factors

The model learns $P$ and $Q$ by trying to reduce the error between actual and predicted ratings for observed entries.

A simplified objective is:

$$
\min_{P,Q}
\sum_{(u,i)\in\Omega}
(r_{ui}-p_u^Tq_i)^2
$$

where $\Omega$ is the set of observed ratings.

Missing ratings are not treated as zero. They are the values we ultimately want to predict.

### Regularization

Regularization prevents the model from fitting the observed ratings too closely.

Conceptually:

- too little regularization can cause **overfitting**
- too much regularization can cause **underfitting**

For the midsem, understanding this idea is more important than memorizing the full regularized loss.

### SGD, ALS and SVD

At this depth:

- **SGD** repeatedly adjusts user and item factors to reduce prediction error.
- **ALS** alternates between updating user factors and item factors.
- **SVD/Funk-SVD** are related factorization ideas used to learn compact representations.

Detailed derivations and update equations are not necessary unless specifically asked.

### Advantages

Matrix factorization creates compact representations and can discover hidden preference patterns.

It often works well on sparse recommendation data and supports fast prediction after training.

### Limitations

It still suffers from cold start, requires training and parameter tuning, and its latent factors may be difficult to interpret.

---

## 11. Attacks on Collaborative Recommender Systems

A **shilling attack**, also called a **profile-injection attack**, occurs when attackers create fake users or ratings to manipulate recommendations.

The basic process is:

$$
\text{Fake ratings}
\rightarrow
\text{Artificial similarity}
\rightarrow
\text{Manipulated prediction}
\rightarrow
\text{Manipulated recommendation}
$$

### Promotion Attack

The attacker gives very high ratings to push a target item upward.

### Demotion Attack

The attacker gives very low ratings to reduce the ranking of a target item.

### Effects

Such attacks can distort similarity calculations, prediction scores, and item exposure.

They can also reduce recommendation quality and user trust.

### Basic Countermeasures

At midsem depth, know these approaches:

- anomaly detection
- trust-based weighting
- robust aggregation
- user-behaviour analysis

Detailed attack taxonomies or detection algorithms are not necessary.

---

## 12. Movie Recommendation System Using Collaborative Filtering

A movie recommender using CF can be explained in the following steps:

1. Collect ratings or implicit signals such as views and likes.
2. Construct the user-movie interaction matrix.
3. Choose user-based, item-based, or model-based CF.
4. Compute similarities or learn latent factors.
5. Generate candidate movies the user has not watched.
6. Predict a score for each candidate.
7. Rank the candidates.
8. Recommend the Top-$N$ movies.
9. Update the system when new feedback arrives.

In **user-based CF**, candidate movies come from similar viewers.

In **item-based CF**, candidate movies are similar to movies the user already liked.

In **matrix factorization**, candidates receive high scores when user and movie latent factors match well.

---

## High-Value Midsem Revision

Prioritize these topics:

1. **Meaning of collaborative filtering**
2. **User-based vs item-based CF**
3. **Stability and serendipity**
4. **When item-based CF is preferred**
5. **Cosine similarity**
6. **Missing-rating prediction**
7. **Cold start in collaborative filtering**
8. **Memory-based vs model-based CF**
9. **Matrix factorization core idea**
10. **Shilling attacks**
11. **Movie recommendation workflow**

These topics align closely with the 2025 midsem paper and the Unit 3 syllabus.