# Codex Rec Sys UNIT_2

# Unit 2 — Content-Based Recommendation

## 1. Core idea

A **content-based recommender system** recommends items whose characteristics are similar to the items a user preferred in the past.

Its central assumption is:

> If a user liked an item before, the user is likely to prefer other items with similar features.
> 

For example, if a student frequently reads papers about machine learning and recommender systems, the system can recommend other papers with similar topics, keywords, titles, or abstracts.

Content-based recommendation mainly uses:

$$
\text{User preferences}+\text{Item content}\rightarrow\text{Recommended items}
$$

Unlike collaborative filtering, it does not primarily depend on finding other users with similar behaviour.

### Formal representation

Represent item $i$ by a feature vector:

$$
x_i=[x_{i1},x_{i2},\ldots,x_{im}]
$$

and user $u$ by a profile over the same features:

$$
P_u=[p_{u1},p_{u2},\ldots,p_{um}].
$$

The recommendation score is:

$$
\operatorname{Score}(u,i)=\operatorname{sim}(P_u,x_i).
$$

Items with larger scores are ranked higher, and the best $k$ items are recommended.

## 2. Architecture and working

The high-level architecture is:

$$
\text{Item database}\rightarrow\text{Content analysis}\rightarrow
\text{Item profiles}
$$

$$
\text{User interactions}\rightarrow\text{User-profile construction}
$$

$$
\text{User profile}+\text{Item profiles}\rightarrow
\text{Matching}\rightarrow\text{Ranking}\rightarrow Top\text{-}k
$$

User feedback then updates the user profile.

### Components

| Component | Responsibility |
| --- | --- |
| Item database | Stores items and their available content or attributes |
| Content analyser | Cleans content and extracts useful features |
| Item-profile generator | Converts each item into a feature vector |
| User-profile learner | Learns feature preferences from the user’s interactions |
| Matching function | Compares the user profile with candidate-item profiles |
| Ranking engine | Sorts candidates by score and selects the top-$k$ items |
| Feedback updater | Changes the profile as new behaviour is observed |

### Complete content-based algorithm

1. Collect item descriptions, attributes, or tags.
2. Preprocess the content and extract features.
3. Represent every item in a common feature space.
4. Build the user’s profile from liked, rated, or consumed items.
5. Compare the user profile with unseen item profiles.
6. Rank items by similarity or predicted relevance.
7. Recommend the top-$k$ items and update the profile from feedback.

The user profile and item profiles must use the **same feature order and meaning**; otherwise, their similarity is meaningless.

## 3. Item profiles and document features

An **item profile** is a structured representation of an item’s characteristics. It answers: **“What features describe this item?”**

$$
IP_i=f(\operatorname{Content}_i)
$$

where $f$ is the feature-extraction process.

### Common feature types

| Feature type | Meaning | Example |
| --- | --- | --- |
| Categorical | One of a set of labels | Movie genre, product brand |
| Numerical | A measurable number | Price, duration, year |
| Textual | Words or phrases from content | Title, abstract, description |
| Binary | Presence or absence of a property | Has subtitles: 1 or 0 |
| Metadata | Information about the item | Author, language, publication date |
| Structural | Organization of the content | Heading count, document length |
| Tags | Short descriptive labels | sci-fi, action, machine learning |
| Multimedia | Features obtained from image, audio, or video | Colour or audio descriptors |

For documents, useful feature groups include:

- **Lexical features:** words, term frequencies, and n-grams.
- **Syntactic features:** parts of speech or grammatical patterns.
- **Semantic features:** topics or meanings represented by concepts or vectors.
- **Metadata and structural features:** author, date, title, headings, and length.
- **Tag features:** labels supplied by users, providers, or automatic tools.

The normal BTech focus is on lexical, metadata, structural, and tag representations. Embeddings may be remembered briefly as dense vectors that can capture semantic similarity, but their training details are not required here.

## 4. Text preprocessing

Raw text is noisy and inconsistent. Preprocessing converts it into a cleaner form before feature extraction.

### Typical pipeline

1. **Cleaning:** remove unwanted symbols, markup, or obvious noise.
2. **Tokenization:** split text into words or tokens.
3. **Case normalization:** convert terms to a common case.
4. **Stop-word handling:** remove very common words when they add little meaning.
5. **Stemming or lemmatization:** reduce related word forms.
6. **Number handling:** remove or retain numbers according to their meaning.

Numbers should not be removed blindly. The value in “64 GB RAM” is useful for product recommendation, and “COVID-19” loses meaning if the number is discarded.

### Stemming versus lemmatization

| Stemming | Lemmatization |
| --- | --- |
| Removes endings using simple rules | Uses linguistic analysis |
| Usually faster | Usually slower |
| May produce an invalid root, such as “studi” | Produces a valid base word, such as “study” |
| Less linguistically precise | More linguistically precise |

## 5. Feature extraction and representation

**Feature extraction** creates a representation from raw content. **Feature selection** keeps the most useful features from an existing set. They are related but not identical.

### 5.1 Bag of Words

The **Bag-of-Words (BoW)** model creates a vocabulary and represents each document using word counts. It ignores word order.

If the vocabulary is:

$$
[\text{AI},\text{recommender},\text{system}],
$$

a document containing “AI recommender recommender” has the count vector:

$$
[1,2,0].
$$

A **binary BoW** records only presence or absence:

$$
[1,1,0].
$$

BoW is simple and interpretable, but its vectors can be very large and sparse, and it does not understand meaning or word order.

### 5.2 TF-IDF

TF-IDF gives a large weight to a term that is frequent in one document but relatively rare across the collection:

$$
\operatorname{TFIDF}(t,d)=\operatorname{TF}(t,d)\times
\operatorname{IDF}(t)
$$

$$
\operatorname{IDF}(t)=\log\left(\frac{N}{df(t)}\right),
$$

where $N$ is the number of documents and $df(t)$ is the number containing term $t$.

#### Worked numerical: TF-IDF

Suppose $N=100$, $df(t)=10$, and $\operatorname{TF}(t,d)=0.2$. Using the natural logarithm:

$$
\operatorname{IDF}(t)=\ln(100/10)=\ln(10)\approx2.303
$$

$$
\operatorname{TFIDF}(t,d)=0.2(2.303)\approx0.461.
$$

**Exam caution:** state the logarithm convention. A base-10 and a natural-log calculation give different values, so one convention must be used consistently.

### 5.3 N-grams

An **n-gram** is a sequence of $n$ consecutive tokens. Unigrams contain one word, bigrams contain two, and trigrams contain three.

Unlike ordinary BoW, n-grams preserve some local word order. For example, the bigram “machine learning” is more informative than treating “machine” and “learning” as unrelated words. The drawback is a larger and sparser feature space.

### Feature-selection overview

Very rare, extremely common, noisy, or unhelpful features may be removed. Frequency, document frequency, chi-square, mutual information, and information gain are possible selection criteria, but their detailed statistical calculations are not required by the supplied Unit 2 material.

## 6. Obtaining item features from tags

Tags are short labels that describe an item. They may come from:

- **users**, through community tagging;
- **providers**, through manually assigned metadata;
- **automatic systems**, through content analysis.

### Tag representations

Suppose the tag vocabulary is:

$$
[\text{action},\text{comedy},\text{romance},\text{sci-fi},\text{space}].
$$

An item tagged action, sci-fi, and space has the binary vector:

$$
[1,0,0,1,1].
$$

Possible representations are:

- **Binary tags:** 1 if a tag is present, otherwise 0.
- **Weighted tags:** assign different importance values to different tags.
- **Tag frequency:** use how often a tag is assigned.
- **Tag TF-IDF:** reduce the importance of tags that appear on almost every item.

$$
\operatorname{TFIDF}(\text{tag},i)=
\operatorname{TF}(\text{tag},i)\times\operatorname{IDF}(\text{tag}).
$$

### User profiles from tags

If a user interacted with items tagged:

$$
\{\text{sci-fi, space}\},\quad
\{\text{sci-fi, robot}\},\quad
\{\text{action, sci-fi}\},
$$

the profile contains sci-fi three times and the other tags once. After weighting or normalization, this tag profile can be compared with candidate-item tag vectors.

The process is:

$$
\text{Items}\rightarrow\text{Tags}\rightarrow\text{Item profiles}
$$

$$
\text{User interactions}\rightarrow\text{User tag profile}
\rightarrow\text{Similarity}\rightarrow\text{Recommendation}
$$

### Advantages and limitations of tags

**Advantages**

- Tags are interpretable and can support explanations.
- They are useful even for a newly added item.
- They may capture concepts not obvious from a short description.

**Limitations**

- Tags may be missing, inconsistent, subjective, noisy, or too general.
- Different users may use synonyms or different spellings.
- Very common tags have little power to distinguish items.

## 7. Learning user profiles

A user profile gives a weight to each item feature. It can be learned from explicit ratings, likes/dislikes, clicks, purchases, reading history, or other interactions.

### 7.1 Simple average profile

For the set $L_u$ of items liked by user $u$:

$$
P_u=\frac{1}{|L_u|}\sum_{i\in L_u}x_i.
$$

#### Worked numerical

Let:

$$
x_1=[1,0.8,0.2],\quad
x_2=[0.9,0.7,0.3],\quad
x_3=[0.8,0.9,0.1].
$$

Then:

$$
P_u=\frac{x_1+x_2+x_3}{3}
=\frac{[2.7,2.4,0.6]}{3}
=[0.9,0.8,0.2].
$$

This method is simple, but every liked item contributes equally.

### 7.2 Weighted and rating-centred profiles

If item $i$ has preference weight $w_{ui}$:

$$
P_u=\frac{\sum_i w_{ui}x_i}{\sum_i|w_{ui}|}.
$$

Larger positive weights give preferred items more influence. To reduce the effect of users who rate everything unusually high or low, use a centred weight:

$$
w_{ui}=r_{ui}-\bar r_u,
$$

where $r_{ui}$ is the user’s rating and $\bar r_u$ is the user’s mean rating.

A positive/negative profile can also be expressed as:

$$
P_u=P_u^+-\lambda P_u^-,
$$

where $P_u^+$ represents preferred content and $P_u^-$ represents disliked content.

**Important:** lack of interaction is not automatically dislike. A user may simply never have seen the item.

### 7.3 Dynamic profile update

Recent behaviour can update an existing profile:

$$
P_u^{(t+1)}=(1-\alpha)P_u^{(t)}+\alpha x_i,
\qquad 0\leq\alpha\leq1.
$$

A larger $\alpha$ makes the profile react more strongly to the new item.

#### Worked numerical: profile update

Given:

$$
P_u^{(t)}=[0.8,0.6,0.2],\quad x_i=[0.2,0.8,0.9],\quad\alpha=0.3,
$$

$$
P_u^{(t+1)}
=0.7[0.8,0.6,0.2]+0.3[0.2,0.8,0.9]
$$

$$
=[0.56,0.42,0.14]+[0.06,0.24,0.27]
=[0.62,0.66,0.41].
$$

The third feature becomes more important because it is strong in the new interaction.

### Short-term and long-term profiles

- A **long-term profile** represents stable interests.
- A **short-term profile** represents the current session or recent interest.

They may be combined as:

$$
P_u=\lambda P_u^{long}+(1-\lambda)P_u^{short}.
$$

## 8. Similarity-based retrieval

Similarity-based retrieval treats the user profile as a personalized query. It compares $P_u$ with candidate vectors $x_i$, ranks their scores, and returns the top-$k$ items.

### 8.1 Cosine similarity

Cosine similarity is the main measure for non-negative text or feature vectors:

$$
\operatorname{cosine}(A,B)=
\frac{A\cdot B}{\|A\|\|B\|}.
$$

It emphasizes direction rather than absolute vector size. A value near $1$ indicates high similarity.

#### Worked numerical: cosine similarity

Let:

$$
P_u=[0.8,0.6,0.2],\qquad x_i=[0.9,0.7,0.1].
$$

Dot product:

$$
P_u\cdot x_i=0.8(0.9)+0.6(0.7)+0.2(0.1)=1.16.
$$

Magnitudes:

$$
\|P_u\|=\sqrt{1.04},\qquad \|x_i\|=\sqrt{1.31}.
$$

Therefore:

$$
\operatorname{cosine}(P_u,x_i)
=\frac{1.16}{\sqrt{1.04}\sqrt{1.31}}\approx0.994.
$$

The item is highly similar to the user’s profile.

### 8.2 Euclidean distance

$$
d(A,B)=\sqrt{\sum_{j=1}^{m}(a_j-b_j)^2}.
$$

A smaller distance means greater similarity. If $A=[1,2]$ and $B=[2,3]$:

$$
d(A,B)=\sqrt{(1-2)^2+(2-3)^2}=\sqrt2\approx1.414.
$$

If a similarity-like score is required, one possible transformation is:

$$
\operatorname{Score}(A,B)=\frac{1}{1+d(A,B)}.
$$

### 8.3 Jaccard similarity

Jaccard similarity is useful for sets, especially binary tags:

$$
J(A,B)=\frac{|A\cap B|}{|A\cup B|}.
$$

For:

$$
A=\{\text{AI, ML, Healthcare}\},\quad
B=\{\text{AI, ML, Robotics}\},
$$

the intersection has 2 elements and the union has 4:

$$
J(A,B)=\frac24=0.5.
$$

### 8.4 Pearson similarity

Pearson similarity compares centred numerical patterns:

$$
r_{XY}=
\frac{\sum_i(x_i-\bar x)(y_i-\bar y)}
{\sqrt{\sum_i(x_i-\bar x)^2}\sqrt{\sum_i(y_i-\bar y)^2}}.
$$

It is useful when the relative pattern matters more than absolute levels, such as when users use rating scales differently. The supplied Unit 2 material gives the formula but no complete Pearson numerical.

### Choosing a measure

| Data | Suitable measure |
| --- | --- |
| TF-IDF or non-negative feature vectors | Cosine similarity |
| Numerical coordinates where absolute distance matters | Euclidean distance |
| Binary tag or feature sets | Jaccard similarity |
| Centred numerical or rating patterns | Pearson correlation |

## 9. Classification-based recommendation

Classification-based recommendation treats recommendation as a supervised-learning problem:

$$
y_{ui}=
\begin{cases}
1,&\text{item }i\text{ is relevant to user }u,\\
0,&\text{item }i\text{ is not relevant to user }u.
\end{cases}
$$

The pipeline is:

$$
\text{Interactions}\rightarrow\text{Labelled examples}\rightarrow
\text{Features}\rightarrow\text{Classifier training}\rightarrow
\text{Relevance prediction}\rightarrow\text{Ranking}
$$

Positive examples may come from likes or strong interactions. Negative examples require care because an unseen item is not necessarily disliked.

### 9.1 Naive Bayes

Naive Bayes predicts a class using Bayes’ theorem:

$$
P(C\mid X)=\frac{P(X\mid C)P(C)}{P(X)}.
$$

It assumes that features are conditionally independent given the class:

$$
P(X\mid C)=\prod_{j=1}^{m}P(x_j\mid C).
$$

For recommendation, calculate the probability of “relevant” and “not relevant” and select the class with the larger posterior probability. It is simple and useful for text represented by words or TF-IDF features, but the independence assumption is often only approximate.

### 9.2 K-Nearest Neighbours

KNN classifies a candidate according to nearby labelled examples:

1. Calculate its distance or similarity to training items.
2. Select the $k$ nearest items.
3. Assign the majority class, or use a distance-weighted vote.

If 3 of the 5 nearest items are relevant and 2 are irrelevant, KNN predicts **relevant**. It is easy to understand but can be slow at prediction time and depends on the feature representation and the choice of $k$.

### 9.3 Logistic regression

Logistic regression predicts a relevance probability:

$$
P(y=1\mid X)=\sigma(w^TX+b)
=\frac{1}{1+e^{-(w^TX+b)}}.
$$

With a threshold of $0.5$, the item is classified as relevant when the probability exceeds $0.5$.

#### Worked numerical

If $z=w^TX+b=2$:

$$
P(y=1\mid X)=\frac{1}{1+e^{-2}}\approx0.881.
$$

Since $0.881>0.5$, the item is predicted to be relevant.

### 9.4 Support Vector Machine

An SVM finds a separating hyperplane between relevant and irrelevant examples:

$$
f(X)=w^TX+b.
$$

The sign of $f(X)$ determines the predicted class. SVMs can work well with high-dimensional text features, but their explanation is less direct than a simple rule or decision tree.

### 9.5 Decision tree

A decision tree repeatedly tests feature values and follows branches to a class label. For example, it may test whether an item contains the feature “healthcare” and then whether it contains “deep learning.”

Decision trees are easy to interpret, but a deep tree may overfit the training examples. Detailed split calculations are not given in the supplied Unit 2 material.

## 10. Similarity retrieval versus classification

| Aspect | Similarity-based retrieval | Classification-based recommendation |
| --- | --- | --- |
| Main idea | Find items similar to the user profile | Predict an item’s relevance class or probability |
| Output | Similarity or distance score | Class label or probability |
| Training | Often little explicit training | Requires labelled examples and model fitting |
| Typical methods | Cosine, Euclidean, Jaccard, Pearson | Naive Bayes, KNN, logistic regression, SVM, decision tree |
| Negative examples | Not necessarily needed | Usually needed |
| Explainability | Often high through matching features | Depends on the classifier |

The two approaches can be combined:

1. Use similarity retrieval to generate a manageable candidate set.
2. Use a classifier to estimate relevance for those candidates.
3. Rank the candidates by probability or a combined score.

One possible combined score is:

$$
\operatorname{Score}(u,i)=
\alpha S_{sim}(u,i)+(1-\alpha)P_{rel}(u,i),
\qquad0\leq\alpha\leq1.
$$

For $S_{sim}=0.91$, $P_{rel}=0.93$, and $\alpha=0.6$:

$$
\operatorname{Score}=0.6(0.91)+0.4(0.93)=0.918.
$$

## 11. Advantages and drawbacks

### Advantages

- **User independence:** the system does not require similar users.
- **New-item support:** a new item can be recommended as soon as its features are available.
- **Explainability:** matching features can provide a natural reason for a recommendation.
- **Personalization:** each user can have a separate profile.
- **Domain adaptability:** it works wherever meaningful item features can be extracted.
- **Direct use of content:** useful even when rating data is limited.

### Drawbacks

- **Overspecialization:** repeatedly recommends items too similar to past choices.
- **Limited serendipity:** may fail to introduce useful but different topics.
- **Feature dependence:** poor or missing item features produce poor recommendations.
- **New-user cold start:** no reliable profile exists before the user provides information.
- **Limited semantic understanding:** simple keywords may miss synonyms or deeper meaning.
- **No collective taste information:** useful patterns from other users may be ignored.
- **Feedback ambiguity and bias:** clicks or displayed items do not always represent true preference.

Initial preference questions or category selection can reduce new-user cold start. Better feature representations can reduce keyword mismatch, but content-based systems still naturally favour similarity to the user’s known interests.

## 12. High-value exam comparisons

| Comparison | Key distinction |
| --- | --- |
| Content-based vs collaborative | Item-feature matching vs patterns across many users |
| Item profile vs user profile | Description of one item vs learned interests of one user |
| Feature extraction vs selection | Create features from raw data vs retain useful existing features |
| BoW vs TF-IDF | Raw/binary term representation vs importance adjusted by document frequency |
| Stemming vs lemmatization | Fast rule-based root vs linguistically valid base form |
| Cosine vs Euclidean | Directional similarity vs absolute geometric distance |
| Jaccard vs Pearson | Set overlap vs correlation of centred numerical patterns |
| Similarity retrieval vs classification | Direct profile–item matching vs learned relevance prediction |
| New item vs new user | Content features can describe a new item, but a new user lacks a profile |