# Types of Retrieval-Augmented Generation (RAG) Architectures

Retrieval-Augmented Generation (RAG) enhances Large Language Models (LLMs) by grounding responses in retrieved external knowledge. As domain requirements become more complex, RAG has evolved from simple search-and-generate pipelines into sophisticated, agentic, and graph-based systems.

## Overview Comparison Matrix

| **RAG Type** | **Retrieval Precision** | **Latency / Cost** | **Implementation Complexity** | **Best Use Case** | 
| **Naïve RAG** | Low – Medium | Low | Low | Basic Q&A, static document queries | 
| **Advanced RAG** | High | Medium | Medium | Enterprise search, technical docs | 
| **Modular RAG** | High | Medium – High | Medium – High | Dynamic workflows, multi-source search | 
| **GraphRAG** | Very High (Relational) | High | High | Complex, interconnected knowledge bases | 
| **Agentic RAG** | Very High | High | Very High | Multi-step reasoning, complex research | 
| **Corrective / Self-RAG** | Very High | High | High | High-accuracy domains (medical, legal) | 

## 1. Naïve RAG (Standard RAG)

Naïve RAG follows a straightforward sequential process: **Retrieve** $\rightarrow$ **Read** $\rightarrow$ **Generate**.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Embedding Model]
    B --> C[Vector Database Search]
    C --> D[Top-K Document Chunks]
    D --> E[Combine: Query + Chunks]
    E --> F[LLM]
    F --> G[Final Response]

```

### Visual Illustration

```
  [ User Query ]
        │
        ▼
  [ Embedding ] ──► [ Vector Store ]
                         │
                         ▼ (Top-K Chunks)
  [ Prompt Context ] ◄───┘
        │
        ▼
     [ LLM ] ──► [ Output ]

```

### Key Mechanics

1. **Indexing:** Documents are split into fixed-size chunks, embedded, and stored in a vector database.

2. **Retrieval:** The query is embedded using the same model to find top-$K$ similar chunks via cosine similarity or Euclidean distance.

3. **Generation:** Retrieved chunks are concatenated into the prompt context for the LLM.

### Trade-offs & Analysis

* **Benefits:** Simple to implement, low latency, low API/compute costs.

* **Drawbacks:** Sensitive to chunk size, susceptible to hallucination if retrieval fails, lacks re-ranking or query expansion.

## 2. Advanced RAG

Advanced RAG introduces **Pre-Retrieval** (e.g., query rewriting, expansion) and **Post-Retrieval** (e.g., re-ranking, chunk compression) optimization steps to address the limitations of Naïve RAG.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Query Rewriting & Expansion]
    B --> C[Vector / Hybrid Search]
    C --> D[Raw Context Chunks]
    D --> E[Re-Ranker Model]
    E --> F[Context Compression / Filtering]
    F --> G[LLM Prompting]
    G --> H[Final Response]

```

### Visual Illustration

```
                   [ User Query ]
                         │
                         ▼
             [ Query Expansion / Rewrite ]
                         │
                         ▼
  [ Sparse (BM25) ] ─── Hybrid ─── [ Dense Vector ]
                         │
                         ▼
                   [ Raw Chunks ]
                         │
                         ▼
                   [ Re-Ranker ] (e.g., Cross-Encoder)
                         │
                         ▼
             [ Context Compression ]
                         │
                         ▼
                     [ LLM ] ──► [ Output ]

```

### Key Mechanics

* **Pre-Retrieval:** Techniques like Hypothetical Document Embeddings (HyDE) or query decomposition generate multiple query variants to improve retrieval recall.

* **Hybrid Search:** Combines keyword search (BM25) with vector similarity.

* **Post-Retrieval:** Cross-encoder models re-rank retrieved chunks based on semantic relevance to prune noisy context before hitting the LLM context window.

### Trade-offs & Analysis

* **Benefits:** Dramatically reduces hallucinations, improves precision, handles ambiguous queries better.

* **Drawbacks:** Increased latency due to re-ranking/rewriting passes; higher compute costs.

## 3. Modular RAG

Modular RAG breaks the pipeline into independent, reconfigurable modules (e.g., Search Module, Memory Module, Routing Module), allowing dynamic execution flows based on query intent.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Router Module]
    B -->|Fact-Based| C[Vector Retrieval Module]
    B -->|Summarization| D[Full Document Storage]
    B -->|Web Search| E[External API Module]
    C --> F[Fusion / Synthesis Module]
    D --> F
    E --> F
    F --> G[LLM Generation]

```

### Visual Illustration

```
                   [ User Query ]
                         │
                         ▼
                [ Dynamic Router ]
            ┌────────────┼────────────┐
            ▼            ▼            ▼
     [ Vector DB ]  [ Web API ]  [ Memory ]
            └────────────┬────────────┘
                         │
                         ▼
                 [ Fusion Module ]
                         │
                         ▼
                      [ LLM ]

```

### Key Mechanics

* **Routing:** Selects execution paths (e.g., query database vs. web search vs. cached memory).

* **Flexibility:** Modules can be swapped, looped, or bypassed dynamically.

### Trade-offs & Analysis

* **Benefits:** Highly extensible, easily customizable across different enterprise workloads.

* **Drawbacks:** High operational complexity, potential routing bottlenecks or errors.

## 4. GraphRAG (Knowledge Graph RAG)

GraphRAG combines vector search with explicit **Knowledge Graphs (KGs)**. It models entities and relationships, enabling holistic reasoning across structured and unstructured data.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Entity Extraction]
    B --> C[Graph Database Traversal]
    B --> D[Vector Similarity Search]
    C --> E[Sub-graph / Relationship Context]
    D --> F[Text Chunks Context]
    E --> G[Context Fusion]
    F --> G
    G --> H[LLM Generation]

```

### Visual Illustration

```
                  [ User Query ]
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      [ Entity Extraction ]  [ Vector Embedding ]
             │                     │
             ▼                     ▼
     (Graph Database)       (Vector DB)
    (Node A)->[Rel]->(Node B)   [ Chunk 1, Chunk 2 ]
             │                     │
             └──────────┬──────────┘
                        ▼
                 [ Merged Graph Context ]
                        │
                        ▼
                     [ LLM ]

```

### Key Mechanics

* **Knowledge Graph Construction:** Extracts entities and relationships during indexing to form an explicit network.

* **Community Summarization:** Summarizes graph clusters to answer broad global queries (e.g., "What are the key themes across all documents?").

### Trade-offs & Analysis

* **Benefits:** Excels at global dataset reasoning, relationship extraction, and multi-hop questions where pure vector similarity fails.

* **Drawbacks:** High indexing costs (requires significant LLM usage to build graph networks), complex graph maintenance.

## 5. Agentic RAG

Agentic RAG empowers an autonomous agent driven by an LLM to determine *how* and *when* to search, evaluate retrieved context, formulate sub-queries, and iteratively gather information until it has sufficient knowledge to answer.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Agent Controller / LLM]
    B --> C{Sufficient Info?}
    C -->|No| D[Select Tool / Formulate Query]
    D --> E[Execute Retrieval Tool]
    E --> F[Observe Tool Results]
    F --> B
    C -->|Yes| G[Synthesize Final Response]

```

### Visual Illustration

```
              ┌──────────────────────────┐
              ▼                          │
       [ Agentic Reasoning ]             │ (Iteration Loop)
              │                          │
              ▼                          │
    { Sufficient Knowledge? }            │
         │              │                │
       (Yes)           (No)              │
         │              │                │
         ▼              ▼                │
     [ Output ]   [ Select Tool ]        │
                        │                │
                        ▼                │
               [ Execute Retrieval ] ────┘

```

### Key Mechanics

* **Reasoning Loop:** Uses frameworks like ReAct (Reason + Act) to loop continuously until the answer is complete.

* **Multi-Tool Access:** Agents can call vector databases, SQL databases, code execution sandboxes, and search engines.

### Trade-offs & Analysis

* **Benefits:** Outstanding for complex, multi-part, and open-ended research queries.

* **Drawbacks:** High latency and potential token cost explosion due to multi-step reasoning loops; risk of getting stuck in infinite execution loops.

## 6. Corrective RAG (CRAG) & Self-RAG

Corrective and Self-Reflective RAG architectures evaluate the quality of retrieved chunks using explicit evaluation models or self-reflection tokens. If retrieval is inadequate, the system dynamically falls back to alternative sources or query corrections.

### Architecture Workflow (Mermaid)

```
flowchart TD
    A[User Query] --> B[Retrieve Context Chunks]
    B --> C[Evaluator Model]
    C -->|Correct| D[Generate Output directly]
    C -->|Ambiguous| E[Refine Query + Augment Search]
    C -->|Incorrect| F[Fallback: Web Search / Knowledge Base]
    E --> G[LLM Generation]
    F --> G
    D --> G

```

### Visual Illustration

```
                 [ User Query ]
                       │
                       ▼
               [ Vector Retrieval ]
                       │
                       ▼
           [ Quality Evaluator Check ]
             /         │          \
      (Correct)   (Ambiguous)   (Incorrect)
         │             │             │
         ▼             ▼             ▼
      [ Pass ]    [ Refine ]     [ Web Search ]
         │             │             │
         └─────────────┼─────────────┘
                       ▼
                    [ LLM ]

```

### Key Mechanics

* **Quality Scoring:** Evaluates retrieved chunks as *Correct*, *Ambiguous*, or *Incorrect*.

* **Dynamic Fallback:** Triggers alternative pipelines (such as web search APIs) when internal database searches return low confidence scores.

### Trade-offs & Analysis

* **Benefits:** Prevents hallucination when vector databases lack relevant context; ensures high reliability for critical domain queries.

* **Drawbacks:** Requires fine-tuned evaluator models or extra evaluation API calls, increasing overall system latency.

## Summary Trade-off Matrix

```
    High Accuracy / Reasoning Capabilities
                  ▲
                  │                                  [ Agentic RAG ]
                  │                  [ GraphRAG ]
                  │       [ CRAG / Self-RAG ]
                  │    [ Modular RAG ]
                  │ [ Advanced RAG ]
                  │
  [ Naïve RAG ]  │
                  └─────────────────────────────────────────────────► High Latency & Cost

```
