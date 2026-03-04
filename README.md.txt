# Layer10 Take-Home Project

### Grounded Long-Term Memory from Unstructured Communication

## Project Idea

Organizations generate a huge amount of knowledge through conversations: emails, issue trackers, chats, and documents. Over time, decisions change, discussions evolve, and it becomes difficult to trace **what was decided and why**.

The goal of this project was to build a small prototype of a **memory system** that can turn unstructured discussions into structured knowledge that can be queried later.

The system extracts entities, relationships, and supporting evidence from technical discussions and stores them in a **memory graph**.

---

## Dataset

For the prototype, I used **GitHub issues from the PyTorch repository**.
These issues contain real technical discussions about bugs, design choices, and implementation details.

The dataset is stored in:

```
data/issues.json
```

---

## System Pipeline

The pipeline implemented in the notebook follows this flow:

```
GitHub Issues
      ↓
LLM-based structured extraction
      ↓
Entity + claim processing
      ↓
Memory graph construction
      ↓
Embedding-based retrieval
      ↓
Interactive graph visualization
```

---

## Structured Extraction

Each issue is processed using a local language model (**Llama3 via Ollama**).
The model extracts:

**Entities**

Examples include:

* functions
* libraries
* components
* contributors

**Claims**

Claims represent relationships between entities.
For example:

```
constant() ignored dtype
```

**Evidence**

Whenever possible, the model also extracts a supporting text snippet from the issue discussion.

Example:

```
constant() ignored its dtype parameter and returned raw literals
```

In some cases the model does not explicitly return evidence for a claim.
Those claims are still preserved in the graph but the evidence field remains empty.

---

## Memory Graph

The extracted knowledge is stored as a **directed graph** using NetworkX.

* **Nodes** represent entities.
* **Edges** represent claims or relationships.
* **Edge metadata** stores supporting evidence.

Example:

```
constant()  ── ignored ──> dtype
```

This structure allows the system to track relationships and later retrieve relevant information.

The serialized graph is saved to:

```
outputs/memory_graph.json
```

---

## Retrieval

To make the graph searchable, claims are embedded using **sentence-transformer embeddings**.

When a question is asked:

1. The question is embedded.
2. It is compared to stored claim embeddings.
3. The most similar claims are returned with their evidence.

Example query:

```
Which functions ignore dtype?
```

Example output:

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
* see relationships
* explore the graph interactively

Open:

```
visualization/memory_graph.html
```

to view the graph.

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

Then open the visualization file.

---

## How This Could Extend to Layer10

In a real environment, the same pipeline could ingest data from sources like:

* email threads
* Slack / Teams conversations
* Jira or Linear issues
* internal documentation

Additional improvements would include:

* stronger entity canonicalization
* tracking revisions of decisions
* permission-aware retrieval
* continuous ingestion pipelines

---

## Technologies Used

* Python
* Ollama
* Llama3
* Sentence Transformers
* NetworkX
* PyVis

---

## Final Note

This project is intentionally lightweight but demonstrates the core idea of turning scattered discussions into **grounded, queryable organizational memory**.
