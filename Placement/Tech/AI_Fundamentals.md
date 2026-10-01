# AI Fundamentals for Placements

<aside>
🎯

This is the **running AI fundamentals page for placement preparation**. Future AI study sessions should be appended here. The goal is not to collect buzzwords, but to understand each idea well enough to explain what it does, why it matters, and where it appears in a real software system.

</aside>

## Session 1 — The basic language of modern AI

Before studying RAG pipelines, agents, vector databases, or model APIs, it helps to understand the small set of ideas that everything else builds on. These concepts are especially useful in software-engineering interviews because they let you discuss AI features without pretending that an LLM is a mysterious black box.

### Training and inference

**Training** is the learning phase. A model is shown enormous amounts of data and repeatedly adjusts its internal numerical parameters, called weights, so that its predictions become better. This is computationally expensive and is usually performed by the organization building the model rather than by an ordinary application developer.

**Inference** is the usage phase. Once the model has been trained, we give it a new input and it uses the weights it already learned to produce an output. When an application sends a prompt to an LLM API and receives a response, that is inference. The model is using what it learned during training; it is not normally retraining itself on that individual request.

> **Placement connection:** If an interviewer asks what happens when your application calls an LLM API, the answer is primarily inference. Training is how the underlying model was created; inference is how your product uses it.
> 

### Tokens and the context window

An LLM does not directly process text as complete sentences or even always as complete words. Text is first broken into **tokens**, which are smaller units that may represent a whole word, part of a word, punctuation, or another text fragment. The model receives tokens as input and also generates its answer token by token.

This matters because model limits and API usage are usually measured in tokens rather than characters. The input side can include the system instructions, current prompt, previous chat messages, tool results, and retrieved RAG passages. The model's generated answer also uses tokens.

The **context window** is the total token budget available for one model interaction. It is not only an input limit. The input/context and the generated output must fit within the model's available context budget.

For example, suppose a model has a hypothetical 128,000-token context window. If the supplied instructions, conversation history, prompt, and retrieved documents already occupy 120,000 tokens, only the remaining capacity is available for generation and any other required context. An application therefore has to manage both sides: how much information it sends to the model and how much room it leaves for the answer.

A useful mental model is:

```
context window
=
input/context tokens
+
generated output tokens
```

If an application keeps adding conversation history or retrieved documents, it may eventually need to remove, summarize, or selectively retrieve information so that enough space remains for the model to produce a useful response.

> **Interview connection:** A larger context window lets the model consider more information and potentially generate longer outputs in the same interaction, but it does not mean every token should be filled or that the model will use all supplied information equally well.
> 

### Embeddings

An **embedding** is a numerical representation of information, commonly represented as a vector: a long list of numbers. The useful property is that content with similar meaning tends to receive embeddings that are close to one another in this numerical space.

For example, a search for “reset my account password” can be matched with a document titled “How to recover login access” even though the wording is different. Ordinary keyword matching may miss that relationship, while embeddings can capture semantic similarity.

This is why embeddings appear so often in AI applications. Documents can be split into chunks, converted into embeddings, stored in a vector database, and later searched by meaning. The  retrieved text can then be given to an LLM as useful context.

### Hallucination

A **hallucination** occurs when a model produces information that sounds convincing but is incorrect, invented, or unsupported by the available evidence. This happens because an LLM is fundamentally generating a likely continuation based on learned patterns. Fluent language is therefore not proof that the underlying statement is true.

In production systems, this is why developers often provide trusted source material, validate important outputs, restrict what actions an AI system may perform, or require human approval for high-impact decisions. RAG can reduce hallucinations caused by missing knowledge, but it does not guarantee that every answer will be correct.

> **Strong interview framing:** “I would treat an LLM response as generated output, not as an automatically verified fact. If correctness matters, I would ground the model with reliable data and validate important results.”
> 

### RAG

**Retrieval-Augmented Generation, or RAG**, gives the model relevant external information at the time a request is made. A typical RAG system first searches a knowledge source for information related to the user's question, then inserts the useful results into the model's context before asking it to answer.

A simplified flow is:

```
User question
    ↓
Retrieve relevant documents
    ↓
Add retrieved text to the prompt/context
    ↓
LLM generates an answer using that information
```

RAG is particularly useful when an application needs knowledge that changes frequently or belongs to a private data source, such as company documentation, support articles, policies, or product information. The model itself does not need to be retrained whenever those documents change.

### Fine-tuning

**Fine-tuning** means performing additional training on an existing model using carefully selected examples. Unlike RAG, which changes the information supplied to the model during inference, fine-tuning actually changes the model's weights.

Fine-tuning is therefore useful when the goal is to make a model behave more consistently for a particular task, style, structure, or domain pattern. It is usually not the first solution for simply giving a model access to changing factual information. For that situation, retrieval is often easier to update and easier to inspect.

<aside>
💡

**The distinction worth remembering:** RAG changes the **context the model receives**. Fine-tuning changes the **model itself**.

</aside>

## How these ideas connect

A useful mental model is to connect the concepts instead of memorizing them separately. A trained model is used during **inference**. Both the input supplied to the model and the output it generates are represented as **tokens**, and together they must fit within the model's **context window**. If the model needs outside knowledge, **embeddings** can help retrieve semantically relevant information for **RAG**. Even with good context, the model can still **hallucinate**, so important outputs may need grounding or validation. If repeated examples are needed to alter the model's behaviour itself, **fine-tuning** becomes an option.

## Placement checkpoint

By the end of this topic, you should be able to explain these without memorized definitions:

1. What is the difference between training and inference?
2. Why do tokens matter to an application developer?
3. What does a context-window limit affect?
4. Why are embeddings useful for semantic search?
5. Why can an LLM confidently produce a wrong answer?
6. When would you choose RAG instead of fine-tuning?

If you can answer those in your own words and give one practical example, you know enough to move into RAG, vector databases, model APIs, and agents without the terminology becoming noise.

---

## Session 2 — Retrieval, Embeddings and Vector Databases

<aside>
🧭

Today's goal is to understand the **retrieval side of RAG** well enough to explain how an application finds useful information before an LLM generates an answer. The placement-level focus is the difference between keyword search and semantic search, what embeddings represent, how similarity is measured, and why a vector database is useful.

</aside>

### RAG: the part before generation matters

A RAG system is not simply “an LLM connected to documents.” The important engineering step is **retrieval**: given a user question, the application must first identify which pieces of external information are actually relevant. Those retrieved passages are then placed into the model's context so the LLM can answer using information it did not need to memorize during training.

A useful mental model is:

```
User question
    ↓
Search / retrieval
    ↓
Relevant documents or passages
    ↓
Add them to the model context
    ↓
LLM generates the final answer
```

If retrieval is poor, generation will also suffer because the model is being grounded with the wrong information. In interviews, it is therefore useful to describe RAG as a **retrieval problem followed by a generation problem**, rather than treating the LLM as the whole system.

### Keyword retrieval: TF-IDF and BM25

Traditional search methods such as **TF-IDF** and **BM25** work primarily from the words that appear in the query and documents. They are useful when exact terminology matters, such as product codes, error messages, names, technical terms, or documentation where users tend to search using the same vocabulary as the source.

**TF-IDF** gives more importance to words that are frequent inside one document but uncommon across the overall document collection. A term such as “database” appearing everywhere is less informative than a rarer term that strongly distinguishes one document from the others.

**BM25** follows the same broad keyword-relevance idea but improves the scoring model. It avoids letting repeated occurrences of one word increase relevance forever and also accounts for document length. For placement interviews, you do not need to derive the formula; the useful distinction is that **BM25 is a strong lexical search algorithm that ranks documents according to query-term relevance**.

The limitation of lexical search is wording. A query such as “recover my login access” may fail to strongly match a document called “reset your account password” if the important words do not overlap enough, even though the meanings are closely related.

### Semantic search and embeddings

**Semantic search** tries to retrieve information by meaning rather than only by exact word overlap. This is where embeddings become useful.

An **embedding model** converts text into a numerical vector. The individual numbers are not normally interpreted by a developer as named features. What matters is the geometry of the resulting space: text with similar meanings tends to be represented by vectors that are relatively close to each other.

For example, “How can I reset my password?” and “I cannot access my account” use different words, but an embedding model may place them close together because they express related intent. The application can therefore retrieve semantically related documents even when keyword overlap is weak.

> **Placement distinction:** Keyword search asks, “Do these texts contain similar words?” Semantic search asks, “Do these texts represent similar meaning?” Real systems can also combine both approaches when exact terminology and semantic meaning are both valuable.
> 

### Similarity, vectors and the dot product

Once the query and documents have embeddings, the system needs a way to compare them. Similarity measures turn the relationship between two vectors into a score that can be used to rank candidate documents.

The **dot product** is one such operation and is commonly discussed because many embedding-retrieval systems use a vector similarity measure based on geometric closeness. Depending on the embedding model and database configuration, systems may use dot product, cosine similarity, or distance-based measures.

For an interview, the important idea is not performing vector arithmetic by hand. It is being able to explain that **the user's query is embedded using the same embedding model as the documents, and the system searches for document vectors that are most similar to the query vector**.

### Why a vector database exists

A normal application could store embedding arrays somewhere and compare the query against every vector manually, but that becomes increasingly inefficient as the collection grows. A **vector database** is designed to store embeddings and efficiently retrieve the vectors that are closest to a query vector.

The stored item usually needs more than just the vector. A useful record can contain the embedding together with the original text or an identifier and metadata such as document name, source, category, permissions, or timestamps. After similarity search finds the best matches, the application retrieves the associated text and supplies it to the LLM.

The placement-level flow is:

```
Documents → embedding model → vectors → vector database

Question → same embedding model → query vector
                              ↓
                    similarity search
                              ↓
                    top relevant documents
                              ↓
                          LLM context
```

### ChromaDB in today's lab

**ChromaDB** is the vector-store tool used in today's lab. Its purpose in this exercise is to make the abstract vector-search pipeline concrete: add documents, create or store their embeddings, and then query the collection for semantically similar content.

The important lesson is not memorizing a ChromaDB API. What should remain after the lab is the architecture: **text is represented as embeddings, embeddings are indexed in a vector store, and a query retrieves the most semantically relevant items before generation occurs**. If you can rebuild this flow using another vector database later, you have understood the concept rather than the library.

### What to say in an interview

If asked to explain a basic RAG retrieval pipeline, a strong answer is:

> “I would first index the knowledge base by converting its documents or passages into embeddings and storing them in a vector database. When a user asks a question, I create an embedding for the query using the same model and run a similarity search to retrieve the most relevant passages. Those passages are added to the LLM's context, and the model then generates an answer grounded in the retrieved information.”
> 

If asked **BM25 versus embeddings**, explain that BM25 is lexical and strong when exact terms matter, while embeddings support semantic matching when different wording can express the same idea. Neither is universally superior; the correct retrieval method depends on the data and query pattern.

## Placement checkpoint — Session 2

By the end of today's block, you should be able to explain why RAG needs a retrieval stage, how TF-IDF/BM25 differ from semantic search, what an embedding represents, how query and document vectors are compared, and what problem a vector database such as ChromaDB solves. You should also be able to trace one query from **user text → query embedding → similarity search → retrieved text → LLM context** without relying on memorized buzzwords.

## PROVE checkpoint — 1 Sep 2026: **PASS**

Closed-book recall was strong across training vs inference, tokens/context window, embeddings, lexical vs semantic retrieval, vector databases, hallucination, and RAG.

One genuine repair:

- fine-tuning was initially framed too narrowly as mainly personality/style;
- corrected mental model: **RAG changes the context supplied at inference; fine-tuning changes model weights through additional training and can alter broader task/domain behavior.**

The context-window note was also clarified so that **input/context tokens and generated output tokens together consume the available context budget**.

**Status:** MAINTAIN. No broad relearning needed.