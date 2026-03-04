# Layer10 Take-Home Project

### Grounded Long-Term Memory from Unstructured Communication

## Overview

Organizations generate a large amount of knowledge through discussions in emails, issue trackers, chats, and documents. Over time it becomes difficult to trace **what was decided, who proposed it, and what evidence supports it**.

This project implements a small prototype of a system that converts unstructured discussions into a **structured memory graph** that can be queried later.

The system extracts entities, relationships (claims), and supporting evidence from GitHub issue discussions and stores them in a graph representation. The graph can then be queried to retrieve evidence-backed answers.

---

## Dataset

For this prototype, I used **GitHub issues from the PyTorch repository** as the public corpus. These issues contain real technical discussions about bugs, features, and implementation details.

The dataset is stored in:

```
data/issues.json
```

---

## Pipeline Overview

The system follows the pipeline below:

```
GitHub Issues
      ↓
LLM Structured Extraction (Llama3 via Ollama)
      ↓
Entity and Claim Processing
      ↓
Memory Graph Construction
      ↓
Semantic Retrieval
      ↓
Graph Visualization
```

---

## Structured Extraction

Each issue is processed using a **local language model (Llama3 via Ollama)**.

The model extracts three types of structured information.

### Entities

Entities represent important objects mentioned in discussions such as:

* functions
* components
* libraries
* contributors
* technologies

Example:

```
constant()
dtype
Triton
CUDA
```

---

### Claims

Claims represent relationships between entities.

Example claim:

```
constant() ignored dtype
```

This can be interpreted as:

```
subject → predicate → object
```

Example:

```
constant()  → ignored → dtype
```

---

### Evidence

Whenever possible, the system also extracts a supporting text snippet from the issue discussion.

Example evidence:

```
constant() ignored its dtype parameter and returned raw literals
```

In some cases the language model does not explicitly return evidence for a claim.
Those claims are still stored in the graph but their evidence field remains empty.
This preserves the extracted relationship while acknowledging that the supporting snippet was not returned by the model.

---

## Memory Graph Design

The extracted knowledge is stored as a **directed multi-graph** using NetworkX.

### Nodes

Nodes represent extracted entities.

### Edges

Edges represent claims or relationships between entities.

Each edge also stores metadata:

* relationship type
* supporting evidence

Example representation:

```
constant()  ── ignored ──> dtype
```

The serialized graph is stored in:

```
outputs/memory_graph.json
```

---

## Retrieval System

To allow querying the extracted memory, claims are embedded using **sentence-transformer embeddings**.

When a user asks a question:

1. The question is converted into an embedding.
2. The embedding is compared to stored claim embeddings.
3. The most similar claims are returned with their evidence.

Example query:

```
Which functions ignore dtype?
```

Example result:

```
Claim: constant() ignored dtype
Evidence: constant() ignored its dtype parameter and returned raw literals.
```

Example retrieval outputs are saved in:

```
outputs/context_packs.json
```

---

## Visualization

To make the extracted memory easier to explore, the graph is visualized using **PyVis**.

The visualization allows users to:

* inspect entities
* view relationships
* explore the memory graph interactively

Open the visualization here:

```
visualization/memory_graph.html
```

---

## Project Structure

```
layer10-memory-system
│
├── layer10_pipeline.ipynb      # Main pipeline implementation
│
├── data
│   └── issues.json             # Downloaded dataset
│
├── outputs
│   ├── memory_graph.json       # Serialized memory graph
│   └── context_packs.json      # Example retrieval outputs
│
├── visualization
│   └── memory_graph.html       # Interactive graph visualization
│
└── README.md
```

---

## Running the Project

Install dependencies:

```
pip install requests networkx pyvis sentence-transformers tqdm
```

Run the notebook:

```
layer10_pipeline.ipynb
```

Then open the visualization file:

```
visualization/memory_graph.html
```

---

## Example Queries

Example questions that can be asked to the system:

```
Which functions ignore dtype?
What issues mention CUDA?
Which components use Triton?
What serialization components exist?
```

Example response:

```
Claim: constant() ignored dtype
Evidence: constant() ignored its dtype parameter and returned raw literals.
```

---

## Adapting the System to Layer10

In a real organizational environment, the same pipeline could ingest information from sources such as:

* email threads
* Slack or Teams discussions
* Jira or Linear issues
* internal documentation

Additional improvements would include:

* stronger entity canonicalization
* decision revision tracking
* permission-aware retrieval
* continuous incremental ingestion pipelines

---

## Technologies Used

* Python
* Ollama
* Llama3
* Sentence Transformers
* NetworkX
* PyVis

---

## Summary

This project demonstrates how scattered technical discussions can be transformed into a **grounded, queryable memory graph** where relationships between entities are stored along with supporting evidence and can be retrieved through semantic search.
