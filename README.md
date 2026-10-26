# Regulatory Document QA

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/chilukuru-saikrishna/regulatory-document-qa/blob/main/Regulatory_Document_QA.ipynb)

A retrieval-augmented generation (RAG) agent that answers questions over long iGaming regulatory PDFs and **cites the document and page for every claim**.

Compliance teams spend hours searching hundreds of pages of licence conditions and directives for one requirement. This tool lets them ask in plain English and get an answer they can verify straight away, because every statement points back to its source page.

## What it does

- Ask a question such as *"What must an operator do when a customer self-excludes?"*
- The agent retrieves the most relevant passages from the regulatory documents
- Gemini writes an answer using **only** those passages, citing document and page for each claim
- A sources table shows exactly which pages were used, so the answer can be checked against the original

If the documents don't contain the answer, it says so instead of guessing.

## Documents covered

| Regulator | Document |
|---|---|
| UK Gambling Commission | Licence Conditions and Codes of Practice (LCCP) |
| Malta Gaming Authority | Player Protection Directive |
| FIAU (Malta) | Implementing Procedures Part II (Remote Gaming Sector) |

The PDFs are **not included** in this repository. They are public documents; download the current versions from each regulator's website.

## How it works

| Step | What happens |
|---|---|
| Extract | PyMuPDF reads each PDF page by page, keeping the page number with the text |
| Chunk | Text is split into overlapping chunks so no requirement is cut off mid-sentence |
| Embed | Each chunk is turned into a vector with Gemini embeddings (`gemini-embedding-001`), using separate task types for documents and for questions |
| Store | Vectors and their source/page metadata are saved in ChromaDB on Google Drive, so the index is built once and reused |
| Retrieve | A question is embedded and matched against the index to find the most relevant chunks |
| Answer | Gemini answers from the retrieved chunks only, with a citation for each claim |
| Interface | A Gradio app with example questions and a sources table |

## Engineering details

Running on the free Gemini tier meant handling real-world API limits:

- **Rate limiting:** chunks are embedded in batches with pauses between them, with retries and exponential backoff on `429` errors
- **Resumable ingestion:** already-embedded chunks are skipped, so indexing can stop and continue later (including across days when the daily quota runs out) without re-embedding anything
- **Model fallback:** if a Gemini model is overloaded (`503`), the agent automatically tries other available Flash models
- **Duplicate protection:** documents can be removed from the index by source and re-ingested cleanly

## Example questions

- What must an operator do when a customer self-excludes?
- When must customer due diligence be carried out for remote gaming customers?
- What are the requirements for reality checks or time-based reminders?
- How should operators handle customers showing signs of gambling harm?

## Run it

1. Click **Open in Colab** above.
2. Get a free Gemini API key from Google AI Studio and add it in Colab's 🔑 **Secrets** panel as `GEMINI_API_KEY`, with notebook access turned on. Never paste the key into a cell.
3. Download the regulatory PDFs and follow the notebook's upload step; they are saved to your Google Drive.
4. Run the cells in order. The first run builds the index; later sessions reuse it from Drive (the notebook's intro says which cells to skip).
5. The last cell launches the Gradio app.

## Limitations

- Answers are only as current as the PDFs loaded; regulations change, so re-ingest new versions.
- Retrieval can miss a relevant passage if the question is worded very differently from the regulation.
- This is a research aid, not legal advice. Always check the cited page before relying on an answer.

## Stack

Python · PyMuPDF · ChromaDB · Gemini API · Gradio · Google Colab

## License

MIT
