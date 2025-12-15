# Image Folder to OCR PDF Converter

This script converts folders of images into size-limited, OCR-processed PDFs. It is designed for large image collections and produces searchable, final PDFs while keeping intermediate artifacts isolated and disposable.

## Overview

Given a folder containing images, the script:

* Recursively discovers supported image files
* Converts each image into a single-page PDF
* Merges PDFs into size-capped batches
* Applies OCR to all content (mandatory)
* Produces one or more final, size-limited PDFs in the original folder

All intermediate processing occurs in a per-folder `_work` directory, which is automatically cleaned up on success.

## Processing Pipeline

For each input folder, the script performs the following steps:

1. **Folder traversal**
   Recursively walk the input directory and detect supported image files.
   Any folder whose name starts with `_` is skipped.

2. **Image to single-page PDF conversion**
   Each image is converted into a single-page PDF using ImageMagick.
   Output location:

   ```
   <folder>/_work/single_pages
   ```

3. **Pre-OCR PDF merging**
   Single-page PDFs are merged into multi-page PDFs, each capped at **50 MB** (pre-OCR).
   Output location:

   ```
   <folder>/_work/pre_ocr
   ```

4. **OCR processing**
   OCR is applied to each pre-OCR PDF using OCRmyPDF.
   Output location:

   ```
   <folder>/_work/ocr
   ```

5. **Final merge**
   All OCR-processed PDFs are merged into a single PDF named:

   ```
   <foldername>.pdf
   ```

   Output location:

   ```
   <folder>/_work/merged
   ```

6. **Final size enforcement**
   If the merged PDF exceeds `--max-mb`, it is split into parts:

   ```
   <foldername>-part01.pdf
   <foldername>-part02.pdf
   ...
   ```

   Each part is ≤ `--max-mb`.
   The original oversized PDF is removed if splitting occurs.

7. **Final output placement**
   The final PDF(s) are moved into the original image folder:

   ```
   <folder>/
   ```

   Existing files with the same names are overwritten.

8. **Cleanup**

   * On success: `<folder>/_work` is deleted
   * On error: `<folder>/_work` is preserved for debugging

## Usage

```bash
python image_to_pdf.py <folder> [options]
```

## Options

| Option               | Description                                         | Default   |
| -------------------- | --------------------------------------------------- | --------- |
| `--max-mb=<size>`    | Maximum size (in MB) of final PDFs before splitting | `100`     |
| `--ocr-lang=<langs>` | Languages passed to OCRmyPDF (`-l`)                 | `deu+eng` |

Language examples:

* `eng`
* `deu`
* `eng+deu`

## Examples

Process a folder with default settings:

```bash
python image_to_pdf.py /path/to/images
```

Use English-only OCR:

```bash
python image_to_pdf.py /path/to/images --ocr-lang=eng
```

Increase the final PDF size limit to 150 MB:

```bash
python image_to_pdf.py /path/to/images --max-mb=150
```

## Dependencies

The script relies on both system-level tools and Python packages.

### macOS Installation Instructions

**ImageMagick and Ghostscript**

```bash
brew install imagemagick
brew install ghostscript
```

**OCRmyPDF and Tesseract language data**

```bash
brew install ocrmypdf
brew install tesseract-lang
```

**Python dependency**

```bash
pip install pikepdf
```

## Notes

* OCR is always applied; there is no non-OCR mode.
* Intermediate files are intentionally isolated in `_work` to avoid polluting the source folder.
* Any folder starting with `_` is ignored during traversal.
