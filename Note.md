# NOTES.md (Beginner Friendly RAG Notes)

This file contains beginner notes to understand how this RAG pipeline works.

---

## 1) What does `load_all_documents("data")` do?

`load_all_documents("data")` returns a list of **LangChain Document objects**.

Each LangChain `Document` contains:

- `page_content` → the actual text
- `metadata` → extra information (file name, page number, etc.)

Example (conceptually):

```python
Document(
    page_content="Sheikh Mujibur Rahman was ...",
    metadata={"source_file": "Sheikh_Mujibur_Rahman.pdf", "page": 3}
)

2) Why do we need Chunking (Splitting)?

You cannot embed a full PDF book at once because it is too large.

So we split text into smaller pieces called chunks.

Example:

A PDF page has 2000 characters

We choose chunk_size = 1000

That page becomes 2 chunks

Why do we use overlap?

Example:

chunk_size = 1000

chunk_overlap = 200

If we split exactly at 1000 characters, a sentence might break.

So overlap repeats the last 200 characters in the next chunk.
This helps keep meaning and context.

3) What is Embedding?

Embedding converts text into numbers (vectors).

Example:

Input text:

What is attention mechanism?


Output embedding:

[0.012, -0.04, 0.33, ...]


For the model all-MiniLM-L6-v2, the vector length is usually 384 numbers.

Embeddings allow semantic search (meaning-based search).

4) What is a Vector Store (FAISS / Chroma)?

A vector store is a database optimized for similarity search.

It stores embeddings and allows fast retrieval.

The process is:

User query text → converted into embedding

Compare query embedding with stored embeddings

Return the Top-K most similar chunks

FAISS and Chroma are both vector stores.

5) Pipeline Summary (Full Flow)

This is the complete RAG ingestion pipeline:

Load documents from data/

Convert files into LangChain Documents

Split documents into chunks

Create embeddings for each chunk

Store embeddings in FAISS or Chroma

Query FAISS/Chroma to retrieve relevant chunks

6) Folder Structure
RAG_ONE/
│── data/
│── notebooks/
│── src/
│   ├── data_loader.py
│   ├── embedding.py
│   ├── faiss_store.py
│── main.py
│── README.md
│── pyproject.toml

7) How to Run (Example)
Step 1: Load documents
from src.data_loader import load_all_documents

docs = load_all_documents("data")
print("Total loaded documents:", len(docs))

Step 2: Chunk + Embed
from src.embedding import EmbeddingPipeline

pipe = EmbeddingPipeline()
chunks = pipe.chunk_documents(docs)
embeddings = pipe.embed_chunks(chunks)

print("Total chunks:", len(chunks))
print("Embeddings shape:", embeddings.shape)

Step 3: Store embeddings in FAISS
from src.faiss_store import FaissVectorStore

store = FaissVectorStore("faiss_store")
store.build_from_documents(docs)

Step 4: Query FAISS
store.load()
results = store.query("Who is Sheikh Mujibur Rahman?", top_k=3)

for r in results:
    print(r)


---

✅ **Yes, you can paste this exactly as-is**  
✅ **Yes, it will render correctly on GitHub**  
✅ **Yes, this is beginner-friendly and correct**

If you want next:  
- a **simple diagram**  
- or a **README.md that links to NOTES.md**  

just tell me.

I prefer this response