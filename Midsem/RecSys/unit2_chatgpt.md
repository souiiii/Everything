# Chat-GPT Rec Sys UNIT_2

# Unit 2 — Content-Based Recommendation

## 1. What is Content-Based Recommendation?

A **content-based recommender system** recommends items by comparing the features of items with the interests of a particular user.

The central idea is:

> If a user liked certain kinds of items in the past, the user is likely to prefer new items with similar characteristics.
> 

For example, if a student frequently reads articles about machine learning and recommender systems, the system may recommend other articles with similar topics, keywords, titles, or abstracts.

A content-based system mainly depends on two things:

- a representation of each **item**
- a representation of the **user’s interests**

The basic flow is:

$$
\text{User preferences} + \text{Item features}
\rightarrow
\text{Similarity score}
\rightarrow
\text{Recommendation}
$$

Unlike collaborative filtering, content-based recommendation does not primarily depend on finding other users with similar behaviour.

### Formal Representation

Represent item $i$ using a feature vector:

$$
x_i = [x_{i1},x_{i2},\ldots,x_{im}]
$$

Represent user $u$ using a profile over the same features:

$$
P_u = [p_{u1},p_{u2},\ldots,p_{um}]
$$

The recommendation score can be written as:

$$
Score(u,i)=sim(P_u,x_i)
$$

Items with larger scores are ranked higher, and the best $k$ items are recommended.

---

## 2. Architecture and Working of a Content-Based Recommender

A content-based recommender usually performs two parallel tasks:

1. It converts raw item information into **item profiles**.
2. It learns a **user profile** from the user’s previous interactions.

The two representations are then compared.

A simplified architecture is:

$$
\text{Item Database}
\rightarrow
\text{Content Analysis}
\rightarrow
\text{Item Profiles}
$$

and

$$
\text{User Interactions}
\rightarrow
\text{User-Profile Construction}
$$

Finally:

$$
\text{User Profile}
+
\text{Item Profiles}
\rightarrow
\text{Matching}
\rightarrow
\text{Ranking}
\rightarrow
\text{Top-}k
$$

### Main Components

| Component | Role |
| --- | --- |
| **Item database** | Stores items and their available content or attributes |
| **Content analyser** | Cleans raw content and extracts useful features |
| **Item-profile generator** | Converts each item into a feature representation |
| **User-profile learner** | Learns the user’s preferences from previous interactions |
| **Matching function** | Compares the user profile with candidate-item profiles |
| **Ranking engine** | Orders items according to their scores |
| **Feedback updater** | Updates the user profile when new behaviour is observed |

### Complete Working

A typical content-based recommender works as follows:

1. Collect item descriptions, tags, metadata, or other attributes.
2. Preprocess the raw content.
3. Extract useful features from every item.
4. Represent each item in a common feature space.
5. Build the user’s profile from liked, rated, or consumed items.
6. Compare the user profile with unseen item profiles.
7. Rank the items according to similarity or predicted relevance.
8. Recommend the top-$k$ items.
9. Update the user profile when new feedback becomes available.

A very important point is that the **user profile and item profiles must use the same feature space**. If their feature meanings or order differ, their similarity score is meaningless.

---

## 3. Item Profiles and Document Features

An **item profile** is a structured representation of an item’s characteristics.

It answers the question:

> What features describe this item?
> 

We can write:

$$
IP_i = f(Content_i)
$$

where $f$ represents the feature-extraction process.

### Common Types of Features

| Feature type | Meaning | Example |
| --- | --- | --- |
| **Categorical** | Item belongs to a category | movie genre, product brand |
| **Numerical** | Measurable quantity | price, duration, year |
| **Textual** | Words or phrases in the content | title, abstract, description |
| **Binary** | Presence or absence of a property | subtitles present or not |
| **Metadata** | Information about the item | author, language, publication date |
| **Structural** | Organization or shape of content | document length, heading count |
| **Tags** | Short descriptive labels | sci-fi, action, machine learning |

For text documents, the most useful exam-level features are usually words and term frequencies, n-grams, metadata, structural information, and tags.

Semantic or embedding-based features may be remembered as dense representations that capture meaning, but their training details are not necessary for this unit.

---

## 4. Text Preprocessing

Raw text cannot usually be used directly. It may contain inconsistent capitalization, punctuation, unnecessary words, or different forms of the same word.

**Text preprocessing** converts this raw content into a cleaner and more useful representation before feature extraction.

### Typical Preprocessing Pipeline

1. **Cleaning:** remove unnecessary symbols, markup, or obvious noise.
2. **Tokenization:** split text into words or tokens.
3. **Case normalization:** convert terms to a common case.
4. **Stop-word handling:** remove very common words when they carry little useful meaning.
5. **Stemming or lemmatization:** reduce related word forms.
6. **Number handling:** keep or remove numbers depending on whether they carry meaning.

Numbers should not be removed automatically. For example, the number in **“64 GB RAM”** is meaningful for product recommendation.

### Stemming vs Lemmatization

| Stemming | Lemmatization |
| --- | --- |
| Uses simple rules to remove endings | Uses linguistic analysis |
| Usually faster | Usually slower |
| May produce an invalid root such as `studi` | Produces a valid base word such as `study` |
| Less linguistically precise | More linguistically precise |

The purpose of both is to reduce unnecessary variation between related word forms.

---

## 5. Feature Extraction and Representation

**Feature extraction** creates useful features from raw content.

**Feature selection** is different: it starts with an existing set of features and keeps only the most useful ones.

This distinction is important in exams.

### 5.1 Bag of Words

The **Bag-of-Words (BoW)** model creates a vocabulary and represents each document using word counts.

Suppose the vocabulary is:

$$
[\text{AI},\text{recommender},\text{system}]
$$

A document containing:

> AI recommender recommender
> 

has the count vector:

$$
[1,2,0]
$$

A binary BoW representation records only whether a word is present:

$$
[1,1,0]
$$

BoW is easy to understand and interpret, but it ignores word order and meaning. Its vectors may also become very large and sparse.

### 5.2 TF-IDF

TF-IDF gives a high weight to a term when it appears frequently in a particular document but is relatively uncommon across the full collection.

The basic formula is:

$$
TFIDF(t,d)=TF(t,d)\times IDF(t)
$$

with:

$$
IDF(t)=\log\left(\frac{N}{df(t)}\right)
$$

where:

- $N$ = total number of documents
- $df(t)$ = number of documents containing term $t$

### Worked Numerical

Suppose:

$$
N=100,\qquad df(t)=10,\qquad TF(t,d)=0.2
$$

Using the natural logarithm:

$$
IDF(t)=\ln(100/10)=\ln(10)\approx2.303
$$

Therefore:

$$
TFIDF(t,d)=0.2(2.303)\approx0.461
$$

**Exam point:** use the logarithm convention given in the question and use it consistently.

### 5.3 N-grams

An **n-gram** is a sequence of $n$ consecutive tokens.

- **Unigram:** one word
- **Bigram:** two consecutive words
- **Trigram:** three consecutive words

For example, the bigram **“machine learning”** preserves more meaning than treating “machine” and “learning” as unrelated words.

N-grams preserve some local word order, but they increase the size and sparsity of the feature space.

### Feature Selection

Very rare, extremely common, noisy, or unhelpful features may be removed.

The source material mentions methods such as document frequency, chi-square, mutual information, and information gain.

For this unit, understand the purpose of feature selection rather than learning detailed statistical calculations for these methods.

---

## 6. Obtaining Item Features from Tags

A **tag** is a short descriptive label attached to an item.

Tags may be provided by users, content providers, or automatic content-analysis systems.

Suppose the tag vocabulary is:

$$
[\text{action},\text{comedy},\text{romance},\text{sci-fi},\text{space}]
$$

An item tagged **action**, **sci-fi**, and **space** may be represented as:

$$
[1,0,0,1,1]
$$

### Tag Representations

Tags can be represented in several ways.

**Binary tags** use:

$$
1=\text{tag present},\qquad 0=\text{tag absent}
$$

**Weighted tags** assign different importance to different tags.

**Tag frequency** uses how often a tag is associated with an item.

**Tag TF-IDF** reduces the weight of very common tags:

$$
TFIDF(\text{tag},i)
=
TF(\text{tag},i)\times IDF(\text{tag})
$$

### Building a User Profile from Tags

Suppose a user interacts with items tagged:

$$
\{\text{sci-fi, space}\}
$$

$$
\{\text{sci-fi, robot}\}
$$

$$
\{\text{action, sci-fi}\}
$$

The tag **sci-fi** appears three times, while the other tags appear once.

This frequency information can be turned into a user-tag profile and compared with candidate-item tag vectors.

The general process is:

$$
\text{User Interactions}
\rightarrow
\text{User Tag Profile}
\rightarrow
\text{Similarity}
\rightarrow
\text{Recommendation}
$$

### Advantages and Limitations of Tags

Tags are useful because they are easy to interpret and can describe a new item before any user has interacted with it.

However, tags may be missing, inconsistent, subjective, noisy, too general, or written using different spellings and synonyms.

---

## 7. Learning User Profiles

A **user profile** represents how strongly a user prefers different item features.

It can be learned from ratings, likes and dislikes, clicks, purchases, reading history, or other interactions.

### 7.1 Simple Average Profile

Suppose $L_u$ is the set of items liked by user $u$.

A simple user profile can be created by averaging the feature vectors of those items:

$$
P_u=
\frac{1}{|L_u|}
\sum_{i\in L_u}x_i
$$

### Worked Numerical

Let:

$$
x_1=[1,0.8,0.2]
$$

$$
x_2=[0.9,0.7,0.3]
$$

$$
x_3=[0.8,0.9,0.1]
$$

Then:

$$
P_u=
\frac{x_1+x_2+x_3}{3}
=
\frac{[2.7,2.4,0.6]}{3}
=
[0.9,0.8,0.2]
$$

This method is easy to compute, but every liked item contributes equally.

### 7.2 Weighted User Profile

If some items should influence the profile more than others, use a preference weight $w_{ui}$:

$$
P_u=
\frac{\sum_i w_{ui}x_i}
{\sum_i |w_{ui}|}
$$

A strongly preferred item therefore contributes more to the final profile.

A centred rating weight can be written as:

$$
w_{ui}=r_{ui}-\bar{r}_u
$$

where $r_{ui}$ is the rating given by user $u$ to item $i$, and $\bar{r}_u$ is that user’s mean rating.

This reduces the effect of users who generally rate everything unusually high or low.

An important idea is that **lack of interaction does not automatically mean dislike**. The user may simply never have seen the item.

### 7.3 Dynamic Profile Update

A user profile may change as new behaviour is observed.

One simple update rule is:

$$
P_u^{(t+1)}
=
(1-\alpha)P_u^{(t)}
+
\alpha x_i
$$

where:

$$
0\leq\alpha\leq1
$$

A larger $\alpha$ gives more influence to the newest interaction.

### Worked Numerical

Suppose:

$$
P_u^{(t)}=[0.8,0.6,0.2]
$$

$$
x_i=[0.2,0.8,0.9]
$$

and:

$$
\alpha=0.3
$$

Then:

$$
P_u^{(t+1)}
=
0.7[0.8,0.6,0.2]
+
0.3[0.2,0.8,0.9]
$$

$$
P_u^{(t+1)}
=
[0.62,0.66,0.41]
$$

The third feature becomes more important because it is strong in the new interaction.

### Short-Term and Long-Term Profiles

A **long-term profile** represents stable interests.

A **short-term profile** represents the user’s current or recent interests.

A system may combine both:

$$
P_u
=
\lambda P_u^{long}
+
(1-\lambda)P_u^{short}
$$

---

## 8. Similarity-Based Retrieval

Similarity-based retrieval treats the user’s profile as a personalized query.

Each candidate item is compared with the user profile, and items with better similarity scores are ranked higher.

### 8.1 Cosine Similarity

Cosine similarity is especially useful for text and non-negative feature vectors.

$$
cosine(A,B)
=
\frac{A\cdot B}
{\lVert A\rVert\lVert B\rVert}
$$

A value closer to $1$ means the vectors point in similar directions.

### Worked Numerical

Let:

$$
P_u=[0.8,0.6,0.2]
$$

and:

$$
x_i=[0.9,0.7,0.1]
$$

Dot product:

$$
P_u\cdot x_i
=
0.8(0.9)+0.6(0.7)+0.2(0.1)
=
1.16
$$

Magnitudes:

$$
\lVert P_u\rVert=\sqrt{1.04}
$$

$$
\lVert x_i\rVert=\sqrt{1.31}
$$

Therefore:

$$
cosine(P_u,x_i)
=
\frac{1.16}
{\sqrt{1.04}\sqrt{1.31}}
\approx0.994
$$

The item is therefore highly similar to the user’s profile.

### 8.2 Euclidean Distance

Euclidean distance measures the straight-line distance between two vectors:

$$
d(A,B)
=
\sqrt{
\sum_{j=1}^{m}(a_j-b_j)^2
}
$$

A **smaller** value means the vectors are closer.

For:

$$
A=[1,2],\qquad B=[2,3]
$$

we get:

$$
d(A,B)
=
\sqrt{(1-2)^2+(2-3)^2}
=
\sqrt{2}
\approx1.414
$$

### 8.3 Jaccard Similarity

Jaccard similarity is useful for binary features or tag sets.

$$
J(A,B)
=
\frac{|A\cap B|}
{|A\cup B|}
$$

Suppose:

$$
A=\{\text{AI, ML, Healthcare}\}
$$

and:

$$
B=\{\text{AI, ML, Robotics}\}
$$

The intersection contains 2 elements and the union contains 4.

Therefore:

$$
J(A,B)=\frac{2}{4}=0.5
$$

### 8.4 Pearson Similarity

Pearson similarity compares centred numerical patterns:

$$
r_{XY}
=
\frac{
\sum_i(x_i-\bar{x})(y_i-\bar{y})
}{
\sqrt{\sum_i(x_i-\bar{x})^2}
\sqrt{\sum_i(y_i-\bar{y})^2}
}
$$

It is useful when the relative pattern matters more than the absolute values.

For this unit, understand what Pearson measures and when it is useful. The supplied material does not provide a complete Pearson numerical.

### Choosing a Similarity Measure

| Data type | Suitable measure |
| --- | --- |
| TF-IDF or non-negative feature vectors | **Cosine similarity** |
| Numerical coordinates where absolute distance matters | **Euclidean distance** |
| Binary tags or sets | **Jaccard similarity** |
| Centred numerical patterns | **Pearson correlation** |

---

## 9. Classification-Based Recommendation

Content-based recommendation can also be treated as a **classification problem**.

The system tries to predict whether an item is relevant to a user:

$$
y_{ui}
=
\begin{cases}
1, & \text{item } i \text{ is relevant to user } u \\
0, & \text{item } i \text{ is not relevant to user } u
\end{cases}
$$

A typical pipeline is:

$$
\text{Interactions}
\rightarrow
\text{Labelled Examples}
\rightarrow
\text{Features}
\rightarrow
\text{Classifier}
\rightarrow
\text{Relevance Prediction}
$$

Positive examples may come from strong likes or interactions.

Negative examples require care because an unseen item is not necessarily disliked.

### 9.1 Naive Bayes

Naive Bayes uses Bayes’ theorem:

$$
P(C\mid X)
=
\frac{P(X\mid C)P(C)}
{P(X)}
$$

It assumes that features are conditionally independent given the class:

$$
P(X\mid C)
=
\prod_{j=1}^{m}P(x_j\mid C)
$$

For recommendation, the system compares the probability of classes such as **relevant** and **not relevant**.

The method is simple and useful for text data, although the independence assumption is often only approximate.

### 9.2 K-Nearest Neighbours

KNN classifies an item according to nearby labelled examples.

The basic steps are:

1. Compute distance or similarity between the candidate and training items.
2. Select the $k$ nearest examples.
3. Use the majority class or a weighted vote.

For example, if 3 of the 5 nearest items are relevant and 2 are irrelevant, KNN predicts the item as **relevant**.

KNN is easy to understand, but its quality depends strongly on the feature representation and the choice of $k$.

### 9.3 Logistic Regression

Logistic regression predicts the probability that an item belongs to the relevant class.

$$
P(y=1\mid X)
=
\frac{1}
{1+e^{-(w^TX+b)}}
$$

### Worked Numerical

If:

$$
w^TX+b=2
$$

then:

$$
P(y=1\mid X)
=
\frac{1}{1+e^{-2}}
\approx0.881
$$

Since the probability is greater than $0.5$, the item would be classified as relevant when using a threshold of $0.5$.

### 9.4 Support Vector Machine

An SVM learns a separating hyperplane between relevant and irrelevant examples.

Its decision function can be written as:

$$
f(X)=w^TX+b
$$

The sign of $f(X)$ determines the predicted class.

For this unit, understand the idea of finding a separating boundary. Detailed optimization is not required.

### 9.5 Decision Tree

A decision tree makes a prediction by repeatedly testing feature values.

For example, a tree may first test whether an item contains the feature **healthcare**, and then whether it contains **deep learning**.

Decision trees are easy to interpret, although very deep trees may overfit the training data.

Detailed split calculations are not required by the supplied Unit 2 material.

---

## 10. Similarity-Based Retrieval vs Classification

Both approaches can be used for content-based recommendation, but they solve the problem differently.

| Similarity-Based Retrieval | Classification-Based Recommendation |
| --- | --- |
| Compares item profile directly with user profile | Learns a model that predicts relevance |
| Produces a similarity or distance score | Produces a class label or probability |
| Usually requires little explicit training | Requires labelled examples |
| Common methods: cosine, Euclidean, Jaccard, Pearson | Common methods: Naive Bayes, KNN, Logistic Regression, SVM, Decision Tree |
| Often easier to explain | Explainability depends on classifier |

In practice, the two approaches can also be combined, for example by using similarity to retrieve candidates and a classifier to rank them.

---

## 11. Advantages and Drawbacks of Content-Based Recommendation

### Advantages

**User independence:** recommendations can be produced without needing similar users.

**New-item support:** a new item can be recommended as soon as its features are available.

**Explainability:** recommendations can often be explained using matching item features.

**Personalization:** every user can have an independent profile.

**Direct use of content:** the method can work even when collaborative rating data is limited.

### Drawbacks

**Overspecialization:** the system may repeatedly recommend items too similar to the user’s past choices.

**Limited serendipity:** useful but different items may be ignored.

**Feature dependence:** poor or missing item features lead to poor recommendations.

**New-user cold start:** a new user has no reliable profile until some preferences are observed.

**Limited semantic understanding:** simple word-based methods may miss synonyms or deeper meaning.

**No collective taste information:** useful patterns from other users may be ignored.

**Feedback ambiguity:** a click does not always mean that the user truly liked the item.

Content-based systems are therefore strong when item features are informative, but they naturally tend to stay close to what the user has already shown interest in.

---

## 12. High-Value Midsem Comparisons

| Comparison | Main difference |
| --- | --- |
| **Content-based vs collaborative** | Item-feature matching vs patterns across many users |
| **Item profile vs user profile** | Description of one item vs learned interests of one user |
| **Feature extraction vs feature selection** | Create features from raw data vs keep useful existing features |
| **BoW vs TF-IDF** | Raw/binary term representation vs importance weighted by document frequency |
| **Stemming vs lemmatization** | Rule-based root reduction vs linguistically valid base form |
| **Cosine vs Euclidean** | Directional similarity vs geometric distance |
| **Jaccard vs Pearson** | Set overlap vs correlation of centred numerical patterns |
| **Similarity retrieval vs classification** | Direct matching vs learned relevance prediction |
| **New item vs new user** | Item can be described using features, but a new user lacks a profile |

## Unit 2 Midsem Priority

If revising quickly, prioritize these topics:

1. **Core idea and architecture of content-based recommendation**
2. **Item profile and user profile**
3. **Text preprocessing**
4. **Feature extraction, BoW and TF-IDF**
5. **Tags and tag-based item representation**
6. **Learning user profiles**
7. **Cosine similarity and similarity-based retrieval**
8. **Euclidean distance, Jaccard and Pearson**
9. **Classification-based recommendation**
10. **Naive Bayes, KNN, Logistic Regression, SVM and Decision Tree**
11. **Advantages and drawbacks**
12. **High-value comparison questions**