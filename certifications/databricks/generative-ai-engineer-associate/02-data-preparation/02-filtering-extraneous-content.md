# Filtering Extraneous Content for RAG Quality

## Overview

Remove irrelevant, low-quality, or degrading content (boilerplate, ads, navigation, footers) from source documents before chunking and embedding to improve RAG application quality.

## Content to Filter

- **Boilerplate**: Headers, footers, copyright notices, disclaimers.
- **Navigation**: Menus, breadcrumbs, "back to top" links.
- **Ads and Promotions**: Marketing content, popups, banners.
- **Metadata Noise**: Hidden tags, CSS/JavaScript in scraped HTML.
- **Duplicates**: Repeated sections, quoted text.
- **Low-Quality Text**: Gibberish, placeholder content, Lorem ipsum.

## Hands-on Examples

### HTML Cleaning

```python
from bs4 import BeautifulSoup
import re

def clean_html(html_content):
    soup = BeautifulSoup(html_content, 'html.parser')
    
    # Remove script and style elements
    for script in soup(["script", "style", "nav", "header", "footer", "aside"]):
        script.decompose()
    
    # Remove HTML comments
    for comment in soup.find_all(text=lambda text: isinstance(text, Comment)):
        comment.extract()
    
    # Get text
    text = soup.get_text()
    
    # Clean whitespace
    lines = (line.strip() for line in text.splitlines())
    chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
    text = '\n'.join(chunk for chunk in chunks if chunk)
    
    return text

html = """
<html>
<head><script>analytics();</script></head>
<body>
  <nav>Home | About | Contact</nav>
  <header>Company Logo</header>
  <main>
    <h1>Important Content</h1>
    <p>This is the actual useful information.</p>
  </main>
  <footer>© 2024 Company</footer>
</body>
</html>
"""

clean_text = clean_html(html)
print(clean_text)  # Only: "Important Content\nThis is the actual useful information."
```

### PDF Cleaning with Unstructured

```python
from unstructured.partition.pdf import partition_pdf
from unstructured.cleaners.core import clean, clean_extra_whitespace

elements = partition_pdf("document.pdf")

cleaned_elements = []
for element in elements:
    # Skip headers/footers (heuristic: very short text at top/bottom of page)
    if len(element.text) < 20 and element.metadata.page_number > 1:
        continue
    
    # Clean text
    cleaned_text = clean_extra_whitespace(element.text)
    cleaned_text = clean(cleaned_text, bullets=True, extra_whitespace=True)
    
    # Filter low-quality text
    if len(cleaned_text.split()) > 3:  # At least 3 words
        element.text = cleaned_text
        cleaned_elements.append(element)
```

### Regex-Based Content Filtering

```python
import re

def filter_boilerplate(text):
    """Remove common boilerplate patterns."""
    
    # Remove copyright notices
    text = re.sub(r'©\s*\d{4}.*?All rights reserved\.?', '', text, flags=re.IGNORECASE)
    
    # Remove "back to top" links
    text = re.sub(r'back to top', '', text, flags=re.IGNORECASE)
    
    # Remove email/contact footers
    text = re.sub(r'contact us at:?\s*[\w\.-]+@[\w\.-]+', '', text, flags=re.IGNORECASE)
    
    # Remove excessive whitespace
    text = re.sub(r'\n\s*\n', '\n\n', text)
    
    return text.strip()
```

### Duplicate Detection

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

def remove_duplicates(chunks, similarity_threshold=0.95):
    """Remove near-duplicate chunks using TF-IDF similarity."""
    
    if len(chunks) < 2:
        return chunks
    
    vectorizer = TfidfVectorizer()
    tfidf_matrix = vectorizer.fit_transform(chunks)
    
    similarity_matrix = cosine_similarity(tfidf_matrix)
    
    unique_chunks = []
    seen_indices = set()
    
    for i in range(len(chunks)):
        if i in seen_indices:
            continue
        
        unique_chunks.append(chunks[i])
        seen_indices.add(i)
        
        # Mark similar chunks as seen
        for j in range(i + 1, len(chunks)):
            if similarity_matrix[i, j] > similarity_threshold:
                seen_indices.add(j)
    
    return unique_chunks

# Example usage
chunks = [
    "Apache Spark is a unified analytics engine.",
    "Apache Spark is a unified analytics engine for big data.",  # Near duplicate
    "Delta Lake provides ACID transactions.",
]

unique = remove_duplicates(chunks)
print(f"Reduced from {len(chunks)} to {len(unique)} chunks")
```

### LangChain Document Filters

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.schema import Document

def filter_low_quality(documents, min_words=10, min_chars=50):
    """Filter documents by quality heuristics."""
    
    filtered = []
    for doc in documents:
        text = doc.page_content
        word_count = len(text.split())
        char_count = len(text)
        
        # Skip short documents
        if word_count < min_words or char_count < min_chars:
            continue
        
        # Skip documents with excessive special characters (likely noise)
        special_char_ratio = len(re.findall(r'[^a-zA-Z0-9\s]', text)) / len(text)
        if special_char_ratio > 0.3:
            continue
        
        filtered.append(doc)
    
    return filtered

documents = [
    Document(page_content="This is useful content with sufficient length and quality."),
    Document(page_content="OK"),  # Too short
    Document(page_content="!@#$%^&*()_+{}|:<>?"),  # Too many special chars
]

clean_docs = filter_low_quality(documents)
```

## Content Quality Pipeline

```python
from typing import List
from langchain.schema import Document

class ContentCleaningPipeline:
    def __init__(self):
        self.filters = []
    
    def add_filter(self, filter_func):
        self.filters.append(filter_func)
        return self
    
    def process(self, documents: List[Document]) -> List[Document]:
        """Apply all filters sequentially."""
        result = documents
        for filter_func in self.filters:
            result = filter_func(result)
            print(f"After {filter_func.__name__}: {len(result)} docs")
        return result

# Build pipeline
pipeline = (ContentCleaningPipeline()
    .add_filter(lambda docs: [d for d in docs if len(d.page_content) > 50])
    .add_filter(filter_low_quality)
    .add_filter(lambda docs: remove_duplicates([d.page_content for d in docs]))
)

# cleaned_docs = pipeline.process(raw_documents)
```

## Best Practices

- **Profile source data**: Identify common noise patterns before filtering.
- **Preserve semantic boundaries**: Don't break sentences or paragraphs during cleaning.
- **Log filtered content**: Track what's removed for audit and tuning.
- **Balance aggressiveness**: Over-filtering loses context; under-filtering adds noise.
- **Test retrieval quality**: Validate that filtering improves relevance metrics.
- **Document-specific rules**: PDFs, HTML, and Markdown require different cleaning strategies.

## Sample Questions

1. Why filter boilerplate before embedding?
2. How detect near-duplicate chunks?
3. Common HTML elements to remove?
4. Risk of aggressive filtering?
5. How validate filtering effectiveness?

## Answers

1. Reduces noise in vector space; improves retrieval precision; lowers embedding costs.
2. TF-IDF + cosine similarity; embedding similarity; exact string matching.
3. `<script>`, `<style>`, `<nav>`, `<header>`, `<footer>`, `<aside>`, comments.
4. May remove relevant context; breaks semantic coherence; requires careful threshold tuning.
5. Compare retrieval metrics (precision@k, MRR) before/after filtering; manual review of samples.

## References

- [BeautifulSoup Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [Unstructured.io Cleaners](https://unstructured-io.github.io/unstructured/core/cleaning.html)
- [Text Preprocessing Best Practices](https://towardsdatascience.com/text-preprocessing-in-natural-language-processing-using-python-6113ff5decd8)

---

Previous: [Chunking Strategies](./01-chunking-strategies.md)  
Next: [Document Content Extraction](./03-document-content-extraction.md)
