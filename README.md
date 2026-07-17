# PDF Version and Producer Identification

This project provides a simple Python utility for identifying technical information from a PDF file.

The script extracts:

* The PDF specification version declared in the file header
* The software or library registered as the PDF producer
* Basic PDF metadata using the `pypdf` library

## Features

* Reads the `%PDF-x.y` header directly from the PDF file
* Identifies versions such as PDF 1.4, 1.5, 1.6, or 1.7
* Reads the `/Producer` metadata property
* Handles missing version or producer information
* Can be executed locally or in Google Colab

## Requirements

* Python 3.8 or later
* pypdf

Install the required dependency with:

```bash
pip install pypdf
```

In Google Colab, run:

```python
!pip install pypdf
```

## Usage

Import the required modules:

```python
import re

from pypdf import PdfReader
```

Define the function responsible for reading the PDF version:

```python
def get_pdf_version(path):
    """
    Reads the PDF header and returns the declared PDF version.

    Expected header format:
        %PDF-x.y

    Returns:
        str | None: The detected version or None when the header
        cannot be identified.
    """
    with open(path, "rb") as file:
        header = file.read(16)

    match = re.search(br"%PDF-(\d\.\d)", header)

    return match.group(1).decode() if match else None
```

Define the function responsible for reading the producer metadata:

```python
def get_pdf_producer(path):
    """
    Reads the PDF metadata and returns the registered producer.

    Returns:
        str | None: The PDF producer or None when the metadata
        is unavailable.
    """
    reader = PdfReader(path)
    metadata = reader.metadata

    if metadata:
        return metadata.get("/Producer") or metadata.get("Producer")

    return None
```

Set the path of the PDF file:

```python
pdf_path = "/content/1e6cc754A8425000.pdf"
```

Extract and display the information:

```python
version = get_pdf_version(pdf_path)
producer = get_pdf_producer(pdf_path)

print(f"PDF version: {version or 'Not identified'}")
print(f"Producer: {producer or 'Not identified'}")
```

## Complete Example

```python
import re

from pypdf import PdfReader


def get_pdf_version(path):
    """
    Reads the first bytes of the PDF and extracts the version
    declared in the %PDF-x.y header.
    """
    with open(path, "rb") as file:
        header = file.read(16)

    match = re.search(br"%PDF-(\d\.\d)", header)

    return match.group(1).decode() if match else None


def get_pdf_producer(path):
    """
    Reads the PDF metadata and returns the Producer property.
    """
    reader = PdfReader(path)
    metadata = reader.metadata

    if metadata:
        return metadata.get("/Producer") or metadata.get("Producer")

    return None


pdf_path = "/content/1e6cc754A8425000.pdf"

version = get_pdf_version(pdf_path)
producer = get_pdf_producer(pdf_path)

print(f"PDF version: {version or 'Not identified'}")
print(f"Producer: {producer or 'Not identified'}")
```

## Expected Output

Example when both properties are available:

```text
PDF version: 1.7
Producer: Adobe PDF Library 17.0
```

Another possible result:

```text
PDF version: 1.4
Producer: Microsoft: Print To PDF
```

When the information cannot be identified:

```text
PDF version: Not identified
Producer: Not identified
```

## How PDF Version Detection Works

PDF files normally start with a header similar to:

```text
%PDF-1.7
```

The script opens the file in binary mode and reads its first 16 bytes:

```python
with open(path, "rb") as file:
    header = file.read(16)
```

A regular expression is then used to locate the version:

```python
re.search(br"%PDF-(\d\.\d)", header)
```

For the following header:

```text
%PDF-1.7
```

the returned version is:

```text
1.7
```

## How Producer Detection Works

The PDF producer is normally stored in the document information dictionary under the `/Producer` property.

The script uses `PdfReader` to access the metadata:

```python
reader = PdfReader(path)
metadata = reader.metadata
```

The Producer property may contain values such as:

```text
Adobe PDF Library
Microsoft Word
Microsoft Print to PDF
LibreOffice
iText
PDFium
ReportLab
Apache FOP
```

The code checks both forms of the metadata key:

```python
metadata.get("/Producer") or metadata.get("Producer")
```

## PDF Producer vs PDF Creator

The PDF metadata may contain two related but different properties.

### Producer

The `Producer` property normally identifies the application or library that generated the final PDF structure.

Examples:

```text
Adobe PDF Library
iText
ReportLab
Microsoft Print to PDF
```

### Creator

The `Creator` property normally identifies the application in which the original document was created.

Examples:

```text
Microsoft Word
Microsoft Excel
Adobe InDesign
Crystal Reports
```

The script can also retrieve the Creator property:

```python
def get_pdf_creator(path):
    reader = PdfReader(path)
    metadata = reader.metadata

    if metadata:
        return metadata.get("/Creator") or metadata.get("Creator")

    return None
```

Usage:

```python
creator = get_pdf_creator(pdf_path)

print(f"Creator: {creator or 'Not identified'}")
```

## Reading Additional Metadata

Other metadata properties can also be retrieved:

```python
def get_pdf_metadata(path):
    reader = PdfReader(path)
    metadata = reader.metadata

    if not metadata:
        return {}

    return {
        "title": metadata.get("/Title"),
        "author": metadata.get("/Author"),
        "subject": metadata.get("/Subject"),
        "creator": metadata.get("/Creator"),
        "producer": metadata.get("/Producer"),
        "creation_date": metadata.get("/CreationDate"),
        "modification_date": metadata.get("/ModDate"),
    }
```

Example:

```python
metadata = get_pdf_metadata(pdf_path)

for key, value in metadata.items():
    print(f"{key}: {value or 'Not identified'}")
```

Possible output:

```text
title: Invoice 12345
author: Finance Department
subject: Customer Invoice
creator: Microsoft Word
producer: Adobe PDF Library
creation_date: D:20260716150000
modification_date: Not identified
```

## Processing Multiple PDF Files

The utility can be extended to analyze every PDF in a directory:

```python
import os


directory = "/content/pdfs"

for filename in os.listdir(directory):
    if not filename.lower().endswith(".pdf"):
        continue

    pdf_path = os.path.join(directory, filename)

    try:
        version = get_pdf_version(pdf_path)
        producer = get_pdf_producer(pdf_path)

        print(f"\nFile: {filename}")
        print(f"PDF version: {version or 'Not identified'}")
        print(f"Producer: {producer or 'Not identified'}")

    except Exception as error:
        print(f"\nFile: {filename}")
        print(f"Error: {error}")
```

## Error Handling

The original example assumes that the provided file exists and is a valid PDF.

A safer implementation can handle common errors:

```python
import os
import re

from pypdf import PdfReader
from pypdf.errors import PdfReadError


def get_pdf_version(path):
    with open(path, "rb") as file:
        header = file.read(16)

    match = re.search(br"%PDF-(\d\.\d)", header)

    return match.group(1).decode() if match else None


def get_pdf_producer(path):
    reader = PdfReader(path)
    metadata = reader.metadata

    if metadata:
        return metadata.get("/Producer") or metadata.get("Producer")

    return None


def analyze_pdf(path):
    if not os.path.isfile(path):
        print(f"File not found: {path}")
        return

    try:
        version = get_pdf_version(path)
        producer = get_pdf_producer(path)

        print(f"PDF version: {version or 'Not identified'}")
        print(f"Producer: {producer or 'Not identified'}")

    except PermissionError:
        print(f"Permission denied: {path}")

    except PdfReadError as error:
        print(f"Invalid or damaged PDF: {error}")

    except OSError as error:
        print(f"File access error: {error}")


pdf_path = "/content/1e6cc754A8425000.pdf"

analyze_pdf(pdf_path)
```

## Important Notes

### Producer metadata may be missing

The `/Producer` property is optional. Some PDF files do not contain this information.

In that case, the script returns:

```text
Producer: Not identified
```

### Metadata is not guaranteed to be accurate

PDF metadata can be:

* Removed
* Modified
* Overwritten
* Copied from another document
* Manually changed

The Producer value should therefore be treated as an indication rather than definitive proof of which software created the file.

### Header version and internal PDF version may differ

The version declared in the `%PDF-x.y` header represents the base PDF version.

A PDF may use a catalog version entry that overrides the header version. Therefore, reading only the header may not always represent every feature used by the document.

### Encrypted PDFs

Encrypted or password-protected PDFs may prevent metadata access.

A password can be provided when necessary:

```python
reader = PdfReader(pdf_path)

if reader.is_encrypted:
    reader.decrypt("password")
```

### Damaged PDFs

A corrupted or incomplete file may still contain a valid `%PDF-x.y` header while failing during metadata extraction.

For this reason, version detection may work even when `PdfReader` cannot fully process the document.

## Limitations

* The script analyzes one PDF file at a time.
* It does not validate whether the file complies fully with the declared PDF version.
* The Producer value depends on optional and editable metadata.
* Password-protected files may require authentication.
* Damaged files may cause `PdfReader` to raise an exception.
* The basic version function only checks the beginning of the file.
* It does not identify PDF/A, PDF/X, or PDF/UA compliance.

## License

This project is available for technical analysis, document inspection, and educational purposes.
