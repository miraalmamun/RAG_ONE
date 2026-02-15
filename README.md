✅ go into a folder (like data/)
✅ find files (pdf, txt, csv, xlsx, docx, json)
✅ read them
✅ convert them into LangChain Document objects
✅ return one big list: documents
========================================================
That list is what you later split into chunks → create embeddings → store in FAISS/Chroma.

1) What is a “Document” in LangChain?

A LangChain Document is basically:

page_content → the text

metadata → extra info (file name, page number, etc.)

Example (conceptually):

Document(
  page_content="Sheikh Mujibur Rahman was ...",
  metadata={"source": "data/Sheikh_Mujibur_Rahman.pdf", "page": 3}
)


For a PDF, you usually get one Document per page.

So if your PDF has 51 pages:
➡️ you get 51 Document objects.
==============================================
2) What your function does (high level)
Function name
def load_all_documents(data_dir: str) -> List[Any]:


data_dir is the folder name ("data")

It returns List[Any] (a list of documents).
(Better: List[Document], but beginner-friendly is fine.)

==============================================
3) Path and why you use it
data_path = Path(data_dir).resolve()


Path("data") means “a folder named data”.
.resolve() converts it to the full absolute path.

Example:

you pass "data"

it becomes something like:
C:\Users\miraa\workspace\RAG\RAG_ONE\data

That’s why you print:

print(f"[DEBUG] Data path: {data_path}")


So you can confirm you’re loading from the correct place.

4) How it finds files (the most important line)

Example for PDFs:

pdf_files = list(data_path.glob('**/*.pdf'))

What glob('**/*.pdf') means

*.pdf = any file ending with .pdf

**/ = search recursively (inside subfolders too)

So it finds:

data/a.pdf

data/books/history.pdf

data/pdf/Sheikh_Mujibur_Rahman.pdf

Everything.

Same idea for txt/csv/xlsx/docx/json.

5) How it loads each file type
PDF part
loader = PyPDFLoader(str(pdf_file))
loaded = loader.load()
documents.extend(loaded)


What happens:

Create a loader for the PDF

load() reads it

loaded is a list of Documents (often 1 per page)

.extend() adds all of them into your main list

✅ Example:
If PDF = 51 pages
loaded length = 51
documents grows by 51

TXT part
loader = TextLoader(str(txt_file))
loaded = loader.load()


Usually TXT becomes one Document total.

✅ Example:
notes.txt → 1 Document

CSV part
loader = CSVLoader(str(csv_file))
loaded = loader.load()


Depends on loader settings, but often it makes:

1 Document per row, or

1 document with the whole file

So if your CSV has 500 rows, you might get 500 Documents.

Excel part
loader = UnstructuredExcelLoader(str(xlsx_file))
loaded = loader.load()


Excel is tricky:

It tries to extract text from sheets and cells

Output can be messy but works for many cases

If you only have tables, sometimes a better choice is to convert to CSV first.

Word docx part
loader = Docx2txtLoader(str(docx_file))
loaded = loader.load()


Usually 1 Document.

JSON part
loader = JSONLoader(str(json_file))
loaded = loader.load()


⚠️ This is the one that often fails for beginners.

Because many JSON loaders need to know which key to extract text from.

Example JSON:

[
  {"title": "A", "content": "hello"},
  {"title": "B", "content": "world"}
]


You usually must tell it: use content.

So in real life you often do something like (depends on LangChain version):

JSONLoader(
  file_path,
  jq_schema=".[] | .content",
  text_content=False
)


If your JSON part crashes, it’s usually because of this.

6) Why you use try/except everywhere

Example:

try:
    loader = PyPDFLoader(str(pdf_file))
    loaded = loader.load()
    documents.extend(loaded)
except Exception as e:
    print(f"[ERROR] Failed to load PDF {pdf_file}: {e}")


Reason:

One bad file should NOT crash your whole pipeline.

You want it to continue loading other files.

✅ Good practice.

7) Why print so many debug statements?

Because in ingestion pipelines, beginners get stuck on:

wrong folder path

files not found

one loader failing silently

empty documents

Your debug prints help you see:

how many files found

which file is being loaded

how many docs came from each file

total docs at end

That’s perfect.

8) What will docs look like in your code?

You do:

docs = load_all_documents("data")
print("Example document:", docs[0])


A sample docs[0] could print like:

page_content='...'

metadata={'source': '...pdf', 'page': 0}

That metadata is SUPER important later for citations.

9) Beginner examples to understand the output
Example A: Only 1 TXT file

data/a.txt contains: “Hello world”

Result:

docs length = 1

docs[0].page_content = “Hello world”

docs[0].metadata["source"] might contain the file path

Example B: 2 PDFs (your case)

history.pdf = 51 pages

mujib.pdf = 43 pages

Result:

total docs = 51 + 43 = 94 ✅ (exactly what you saw)

Example C: PDF inside subfolder

If you have:
data/pdf/book.pdf

Your glob finds it because of **/*.pdf

10) Important improvement (so your FAISS results show file/page)

Right now, many loaders store metadata keys like:

"source" not "source_file"

"page" for pdf

To make everything consistent, after loading each doc you can set:

for d in loaded:
    d.metadata["source_file"] = pdf_file.name


Do that for each file type too.

This way your search results always show:

source_file

file_type

page (for pdf)