# PageWhisperer

PageWhisperer is a lightweight Streamlit app + helper scripts that turn PDFs (and optionally web pages) into a conversational retrieval assistant. Upload PDFs, the app chunks and embeds the text, stores vectors in FAISS, and lets you ask questions about the document using an LLM-backed conversational chain.

## What this is
A simple document Q&A tool aimed at quickly building a chat interface over PDF content (and basic web scraping utilities). It's useful for exploring, searching, and asking conversational questions about the contents of one or more PDFs.

### Stack
- **Language(s):** Python
- **Framework / runtime:** Streamlit (for the UI)
- **Notable libraries:** langchain, PyPDF2, OpenAI (OpenAIEmbeddings / ChatOpenAI), FAISS (faiss-cpu)

## How it's organized
```
app.py                  # Streamlit app: upload PDFs, build vectorstore, conversational UI
htmlTemplates.py        # Simple HTML/CSS templates for chat messages + footer
requirements.txt        # Python dependencies
logo.jpeg               # Project logo used in the Streamlit UI
singlepage_scraping.py  # Minimal Selenium script: extract text from a single page
webscraping.py          # Selenium-based link extraction + page text collection
```

How it fits together: app.py is the runtime entrypoint — when you upload PDFs it extracts text (PyPDF2), splits into chunks (langchain CharacterTextSplitter), embeds chunks (OpenAIEmbeddings by default) and saves them into a FAISS vectorstore. A ConversationalRetrievalChain (langchain) wraps a ChatOpenAI model + conversation memory to answer user queries by retrieving from FAISS.

## Features
- Upload multiple PDFs and build a vector index for them
- Chunking and overlap to preserve context across boundaries
- OpenAI embeddings + ChatOpenAI conversational responses (configurable)
- Simple chat UI rendered with HTML templates inside Streamlit
- Basic Selenium-based utilities for scraping single pages or extracting site navigation links

## Requirements
See requirements.txt for the main packages. Minimal set:
- Python 3.8+
- pip
- OpenAI account + API key (or substitute a HuggingFace LLM/embedding if you prefer)

Example of relevant dependencies (from requirements.txt):
- langchain
- PyPDF2
- python-dotenv
- streamlit
- openai
- faiss-cpu
- pyngrok (optional)

## Environment variables
- OPENAI_API_KEY — required for the default OpenAI embeddings and ChatOpenAI usage.

The app checks that the key is available (app.init()). You can export it in your shell or add to a .env file and use python-dotenv.

## How to run (development)
1. Create a virtual environment and install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate    # or .venv\Scripts\activate on Windows
pip install -r requirements.txt
```

2. Set your OpenAI API key (one of):

```bash
export OPENAI_API_KEY="sk-..."          # macOS / Linux
setx OPENAI_API_KEY "sk-..."             # Windows (restart required)
# or create a .env file with OPENAI_API_KEY=...
```

3. Run the Streamlit app:

```bash
streamlit run app.py
```

4. In the left sidebar upload one or more PDF files and click "Process". After processing, ask questions using the chat input.

Notes:
- The first processing step builds FAISS in memory; no persistence is provided by default. If you restart the app you will need to re-upload and re-process PDFs.
- If you want to use Hugging Face embeddings or an HF LLM, uncomment and configure the relevant lines in app.py (already indicated in code comments).

## Using the scraping utilities
- singlepage_scraping.py: prompts for a URL and prints the extracted page text using Selenium + Chrome WebDriver.
- webscraping.py: prompts for a URL, collects page text and extracts navigation links from a number of likely navigation containers.

Both scripts require a working ChromeDriver installation and may need adjustments to driver location or Selenium options depending on your environment.

## Implementation details
- get_pdf_text(pdf_docs): reads uploaded file-like objects with PyPDF2.PdfReader and concatenates page text.
- get_text_chunks(text): uses CharacterTextSplitter with chunk_size=1000 and chunk_overlap=200 to create overlapping chunks which improve retrieval quality.
- get_vectorstore(text_chunks): builds OpenAIEmbeddings() and constructs a FAISS index from the text chunks. (There is commented code to use HuggingFaceInstructEmbeddings instead.)
- get_conversation_chain(vectorstore): wraps a ChatOpenAI model and ConversationBufferMemory into a ConversationalRetrievalChain. This allows follow-up questions to be answered in context.
- Frontend: Streamlit renders HTML templates from htmlTemplates.py to show user and bot messages in a styled chat format.

## Configuration & customization ideas
- Persist FAISS to disk or push to a more permanent vector DB for larger datasets.
- Add metadata to vectors (filename, page number) so responses can cite sources.
- Swap the embedding model to a local or self-hosted embedding (InstructorEmbedding / sentence-transformers) to avoid OpenAI costs.
- Use HuggingFaceHub LLMs if you want to avoid the ChatOpenAI dependency (commented lines in app.py show how to switch).

## Advantages, achievements & efficiency gains
Implementing PageWhisperer (or integrating similar document-embedding + retrieval chains) provides concrete productivity and operational benefits:

- Faster access to information: Instead of manually reading long PDFs, users can ask concise questions and get targeted answers, reducing time-to-insight from hours to minutes.
- Reduced context switching: Developers, analysts, or researchers can keep the app open and query documents directly rather than switching between viewers, notes, and search tools.
- Improved comprehension and recall: Chunking with overlap preserves contextual information across boundaries which improves retrieval relevance and reduces misinterpretation of isolated snippets.
- Scalable search over documents: FAISS vectorization supports quick nearest-neighbor search over thousands of chunks, enabling interactive exploration of large corpora.
- Reproducible, automatable workflow: The pipeline (extract -> chunk -> embed -> index -> query) is repeatable and can be automated for periodic updates or batch processing.
- Cost control options: By switching to local embeddings (sentence-transformers / InstructorEmbedding) and local LLMs, you can lower per-query cloud costs and data exposure.
- Better collaboration and knowledge sharing: The app centralizes the information in documents into a shareable interface which teams can use to onboard, review, or audit content faster.
- Prototype-to-production readiness: The code demonstrates a minimal but practical LangChain + FAISS pattern that can be hardened for production (persistence, metadata, citations).

Achievements you can attribute to this implementation:
- Proof-of-concept for conversational retrieval over PDFs.
- Demonstrates integration of LangChain primitives (text splitter, embeddings, vectorstore, conversational chain) in a concise app.
- Provides a foundation for adding source citation, multi-document merging, or web-crawling pipelines.

Quantifying efficiency gains (examples):
- Query latency: FAISS NN queries return results in milliseconds for moderate corpora (vs. full-text scans that take much longer).
- Human time saved: For a 300-page manual, targeted Q&A can reduce lookup time from ~30–90 minutes (manual scanning) to under 5 minutes for a typical factual question.
- Reprocessing time: Chunking + embedding a 100-page document is a one-time upfront cost; subsequent queries do not incur the embedding cost unless you re-index.

## Limitations & security
- The default uses OpenAI services — be mindful of API costs and data privacy when uploading sensitive documents.
- No authentication or access control is provided — do not expose the app to untrusted networks without adding protections.
- PDF text extraction depends on how text is encoded in PDFs; some PDFs (scanned images) need OCR before extracting useful text.

## Troubleshooting
- If Streamlit cannot find Chrome/Chromedriver for the scraping scripts, install ChromeDriver and make sure it's on PATH or configure a WebDriver manager.
- If embeddings or LLM calls fail, confirm OPENAI_API_KEY is set and has proper permissions and quota.

## License & Credit
MIT-style or choose your preferred license.

Built by LearnCodeWithRam. UI template and footer reference a contributor (Heflin Stephen Raj S) in htmlTemplates.py.
