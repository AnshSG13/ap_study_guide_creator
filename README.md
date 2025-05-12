# PDF Scraper and Retrieval-Augmented Generator (RAG) System


This project automates the pipeline for scraping AP (Advanced Placement) exam PDFs, extracting their content, splitting the text into semantic chunks, generating vector embeddings, and storing them in a searchable vector store (MongoDB Atlas). The system integrates with OpenAI's GPT models to enable **Retrieval-Augmented Generation (RAG)**, allowing users to query the dataset and receive grounded, contextualized answers.
 It uses **Poetry** for dependency management.

## How It Works

1. **Initial Setup**
   - The user provides a list of AP website URLs in `input_websites.csv`.
   - The project’s main script, `main.py`, orchestrates the entire workflow by calling two modules:
     - **Web Scraping Module:** Downloads PDFs from the provided AP websites.
     - **PDF Processing Module:** Extracts text from the downloaded PDFs, chunks the text, converts each chunk into a vector embedding using a SentenceTransformer model, and indexes these embeddings in a FAISS vector database.
   - Ensure that the required folder structure is in place (e.g., folders for downloaded PDFs, saved indexes, and metadata).

2. **Web Scraping**
   - The web scraping code loads each URL from `input_websites.csv`, navigates the site using Selenium, and downloads any PDF files found on those pages.
   - Downloaded PDFs are saved in the designated folder (e.g., `downloaded_files/`).

3. **PDF Processing and Vector Indexing**
   - The PDF processing module reads the downloaded PDFs and extracts text using PyPDF2.
   - The extracted text is split into manageable chunks to preserve context.
   - Each text chunk is converted into a vector embedding using a pre-trained model (e.g., `all-MiniLM-L6-v2` from SentenceTransformer).

4. **Vector Database (MongoDB Atlas)**
   - Each chunk, along with its embedding and metadata (e.g., source PDF, chunk index), is stored in a **MongoDB collection**.
   - A sample document in the database includes:
     ```json
     {
       "pdf_file": "2020_APUSH_DBQ.pdf",
       "chunk_index": 3,
       "chunk_text": "...",
       "embedding": [0.021, -0.004, ..., 0.108]
     }
     ```

5. **Retrieval-Augmented Generation (RAG)**
   - When a user submits a query, the system:
     - Embeds the query using the same SentenceTransformer model.
     - Retrieves top-matching chunks from MongoDB using cosine similarity.
     - Sends the chunks and the user query to OpenAI’s GPT model (e.g., `gpt-4o-mini`) to generate a context-aware response.


## Setup Instructions


python3 -m venv .venv
source .venv/bin/activate      # on Windows use: .venv\Scripts\activate
pip install -r requirements.txt
cd frontend
npm install
cd ..
cat <<EOF > .env
OPENAI_API_KEY=your_openai_api_key_here
MONGODB_URI=your_mongodb_connection_string_here
EOF
cd backend
uvicorn app:app --reload
# in a separate terminal/window, run:
cd frontend
npm start


## Notes

- **PDF Warnings:**  
  You may see warnings such as `unknown widths` or `ignore '/Perms' verify failed` during PDF processing. These are common with PyPDF2 when encountering non-standard PDF structures or permission entries. They can typically be ignored unless they affect the text extraction quality.

- **Modularity:**  
  The project has been modularized so that web scraping and PDF processing are contained in separate modules. This makes it easier to test, maintain, and extend the functionality.

- **Customization:**  
  You can customize parameters such as the maximum chunk size or the model used for vector embeddings by modifying the respective modules.
