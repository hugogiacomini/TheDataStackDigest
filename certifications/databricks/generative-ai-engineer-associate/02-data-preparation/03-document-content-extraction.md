# Document Content Extraction: Python Packages by Format

## Overview

Select appropriate Python packages to extract content from different source formats: PDF, DOCX, HTML, Markdown, images (OCR), spreadsheets, and more.

## Extraction Tools by Format

| Format | Recommended Package | Use Case | Example |
|--------|-------------------|----------|---------|
| **PDF** | `PyPDF2`, `pdfplumber`, `Unstructured` | Simple PDFs, tables, complex layouts | Extract text, preserve structure |
| **DOCX** | `python-docx`, `Unstructured` | Word documents | Paragraphs, headings, tables |
| **HTML** | `BeautifulSoup`, `lxml`, `Trafilatura` | Web scraping, articles | Clean HTML, extract main content |
| **Markdown** | `markdown`, `mistune` | Documentation, notes | Parse headers, code blocks |
| **Images** | `pytesseract` (OCR), `EasyOCR` | Scanned documents, photos | Text extraction from images |
| **Excel/CSV** | `pandas`, `openpyxl` | Spreadsheets | Tabular data |
| **JSON** | `json`, `jsonlines` | Structured data | API responses, logs |
| **Email** | `email`, `mailparser` | Email archives | Subject, body, attachments |

## Hands-on Examples

### PDF Extraction

```python
# Simple PDF (PyPDF2)
import PyPDF2

def extract_pdf_pypdf2(file_path):
    with open(file_path, 'rb') as file:
        reader = PyPDF2.PdfReader(file)
        text = ""
        for page in reader.pages:
            text += page.extract_text() + "\n\n"
    return text

# Complex PDF with tables (pdfplumber)
import pdfplumber

def extract_pdf_with_tables(file_path):
    content = {"text": [], "tables": []}
    
    with pdfplumber.open(file_path) as pdf:
        for page in pdf.pages:
            # Extract text
            content["text"].append(page.extract_text())
            
            # Extract tables
            tables = page.extract_tables()
            for table in tables:
                content["tables"].append(table)
    
    return content

# Production-grade (Unstructured)
from unstructured.partition.pdf import partition_pdf

elements = partition_pdf(
    "document.pdf",
    strategy="hi_res",  # Uses ML models for better accuracy
    infer_table_structure=True
)

for element in elements:
    print(f"{element.category}: {element.text[:100]}")
```

### DOCX Extraction

```python
from docx import Document

def extract_docx(file_path):
    doc = Document(file_path)
    content = {
        "paragraphs": [],
        "tables": [],
        "headings": []
    }
    
    for para in doc.paragraphs:
        if para.style.name.startswith('Heading'):
            content["headings"].append({
                "level": para.style.name,
                "text": para.text
            })
        else:
            content["paragraphs"].append(para.text)
    
    for table in doc.tables:
        table_data = []
        for row in table.rows:
            table_data.append([cell.text for cell in row.cells])
        content["tables"].append(table_data)
    
    return content

doc_content = extract_docx("document.docx")
```

### HTML Extraction

```python
from bs4 import BeautifulSoup
import requests

# Basic extraction
def extract_html(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Get main content (remove nav, footer, etc.)
    for tag in soup(['script', 'style', 'nav', 'footer', 'header']):
        tag.decompose()
    
    return soup.get_text(strip=True, separator='\n')

# Advanced: Trafilatura (article extraction)
import trafilatura

def extract_article(url):
    downloaded = trafilatura.fetch_url(url)
    text = trafilatura.extract(
        downloaded,
        include_comments=False,
        include_tables=True,
        output_format='txt'
    )
    return text
```

### Image OCR

```python
from PIL import Image
import pytesseract

def extract_text_from_image(image_path):
    image = Image.open(image_path)
    text = pytesseract.image_to_string(image, lang='eng')
    return text

# EasyOCR for better multilingual support
import easyocr

def extract_with_easyocr(image_path, languages=['en']):
    reader = easyocr.Reader(languages)
    results = reader.readtext(image_path)
    text = ' '.join([result[1] for result in results])
    return text
```

### Spreadsheet Extraction

```python
import pandas as pd

def extract_excel(file_path):
    # Read all sheets
    excel_file = pd.ExcelFile(file_path)
    sheets_data = {}
    
    for sheet_name in excel_file.sheet_names:
        df = pd.read_excel(file_path, sheet_name=sheet_name)
        # Convert to text representation
        sheets_data[sheet_name] = df.to_markdown(index=False)
    
    return sheets_data
```

### Unified Extraction with Unstructured

```python
from unstructured.partition.auto import partition

def extract_any_format(file_path):
    """Auto-detect format and extract content."""
    elements = partition(file_path)
    
    content = {
        "text": [],
        "tables": [],
        "metadata": []
    }
    
    for element in elements:
        if element.category == "Table":
            content["tables"].append(str(element))
        elif element.category in ["NarrativeText", "Title", "ListItem"]:
            content["text"].append(element.text)
        
        content["metadata"].append({
            "type": element.category,
            "page": getattr(element.metadata, 'page_number', None)
        })
    
    return content

# Works with PDF, DOCX, HTML, EPUB, etc.
content = extract_any_format("document.pdf")
```

## Extraction Pipeline for RAG

```python
from pathlib import Path
from typing import List, Dict
from langchain.schema import Document

class DocumentExtractor:
    def __init__(self):
        self.extractors = {
            '.pdf': self._extract_pdf,
            '.docx': self._extract_docx,
            '.html': self._extract_html,
            '.txt': self._extract_txt,
            '.md': self._extract_txt,
            '.png': self._extract_image,
            '.jpg': self._extract_image,
        }
    
    def extract(self, file_path: str) -> List[Document]:
        path = Path(file_path)
        suffix = path.suffix.lower()
        
        if suffix not in self.extractors:
            raise ValueError(f"Unsupported format: {suffix}")
        
        return self.extractors[suffix](file_path)
    
    def _extract_pdf(self, file_path: str) -> List[Document]:
        from unstructured.partition.pdf import partition_pdf
        elements = partition_pdf(file_path)
        return [Document(page_content=el.text, metadata={"source": file_path, "type": el.category}) 
                for el in elements]
    
    def _extract_docx(self, file_path: str) -> List[Document]:
        from docx import Document as DocxDocument
        doc = DocxDocument(file_path)
        return [Document(page_content=para.text, metadata={"source": file_path}) 
                for para in doc.paragraphs if para.text.strip()]
    
    def _extract_html(self, file_path: str) -> List[Document]:
        from bs4 import BeautifulSoup
        with open(file_path, 'r', encoding='utf-8') as f:
            soup = BeautifulSoup(f, 'html.parser')
            text = soup.get_text(separator='\n')
        return [Document(page_content=text, metadata={"source": file_path})]
    
    def _extract_txt(self, file_path: str) -> List[Document]:
        with open(file_path, 'r', encoding='utf-8') as f:
            text = f.read()
        return [Document(page_content=text, metadata={"source": file_path})]
    
    def _extract_image(self, file_path: str) -> List[Document]:
        import pytesseract
        from PIL import Image
        image = Image.open(file_path)
        text = pytesseract.image_to_string(image)
        return [Document(page_content=text, metadata={"source": file_path, "type": "ocr"})]

# Usage
extractor = DocumentExtractor()
documents = extractor.extract("data/document.pdf")
```

## Best Practices

- **Use Unstructured for production**: Handles multiple formats with consistent API.
- **Validate extracted text**: Check for garbled encoding, missing content.
- **Preserve structure metadata**: Keep page numbers, headings, table context.
- **Handle OCR errors**: Apply spell-checking and confidence filtering.
- **Batch processing**: Extract concurrently for large document sets.
- **Test on sample documents**: Different PDFs (scanned, native, complex layouts) behave differently.

## Sample Questions

1. Which package for scanned PDF extraction?
2. How extract tables from PDF?
3. Best library for HTML article extraction?
4. When use Unstructured vs PyPDF2?
5. How handle multilingual OCR?

## Answers

1. `pytesseract` or `EasyOCR` for OCR; `Unstructured` can handle both native and scanned PDFs.
2. `pdfplumber` (native PDFs) or `Unstructured` with `infer_table_structure=True`.
3. `Trafilatura` (purpose-built for article extraction, handles boilerplate well).
4. Unstructured for complex/mixed formats and production; PyPDF2 for simple, lightweight PDF text extraction.
5. Use `EasyOCR` with multiple language codes: `reader = easyocr.Reader(['en', 'es', 'fr'])`.

## References

- [Unstructured.io](https://unstructured.io/)
- [PyPDF2 Docs](https://pypdf2.readthedocs.io/)
- [pdfplumber](https://github.com/jsvine/pdfplumber)
- [Trafilatura](https://trafilatura.readthedocs.io/)
- [pytesseract](https://github.com/madmaze/pytesseract)

---

Previous: [Filtering Extraneous Content](./02-filtering-extraneous-content.md)  
Next: [Writing Chunks to Delta Lake in Unity Catalog](./04-writing-chunks-to-delta-lake.md)
