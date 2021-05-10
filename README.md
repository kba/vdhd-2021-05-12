# vDHd Demo Evaluation

> Download GT, process with calamari, evaluate with dinglehopper and ocrd-cor-asv-ann-evaluate

**NOTE** This demo is just to show how to do the evaluation. The choice of OCR
engines, evaluation processors and models is entirely arbitrary and should not
be construed as approval or disapproval.

## Browse to and download from OCR-D GT Repo

* Go to https://ocr-d.de/gt-repo
* Copy link to https://ocr-d-repo.scc.kit.edu/api/v1/dataresources/dda89351-7596-46eb-9736-593a5e9593d3/data/luz_blitz_1784.ocrd.zip

```sh
wget https://ocr-d-repo.scc.kit.edu/api/v1/dataresources/dda89351-7596-46eb-9736-593a5e9593d3/data/luz_blitz_1784.ocrd.zip
```

## Extract the OCRD-ZIP

Extract the `data` subdirectory of the ZIP (which contains the workspace)

```sh
unzip luz_blitz_1784.ocrd.zip 'data/*'
```

## Run small workflows for OCR results with tesseract and calamari, compare output

<!--This executes tesseract twice (once `Fraktur_GT4HistOCR`, once `deu`) with the-->
<!--minimalist call on the segmentation provided by the GT.. Then it runs-->
<!--`ocrd-tesserocr-recognize` and calamari with `qurator-gt4histocr-1.0` because-->
<!--calamari does not like the GT segmentation (we will investigate).-->

This runs tesseract's segmentation, followed by two tesseract recognition runs (once with `Fraktur_GT4HistOCR`, once with `deu`)
and finally calamari with the `qurator-gt4histocr-1.0` model.

```sh
ocrd process -m data/mets.xml \
  "tesserocr-segment -I OCR-D-IMG -O SEG-TESS" \
  "tesserocr-recognize -P segmentation_level word -P textequiv_level line -P find_tables true -P model Fraktur_GT4HistOCR -I SEG-TESS -O TESS-GT4HIST" \
  "tesserocr-recognize -P segmentation_level word -P textequiv_level line -P find_tables true -P model deu -I SEG-TESS -O TESS-DEU" \
  "calamari-recognize -P checkpoint_dir qurator-gt4histocr-1.0 -I SEG-TESS -O CALA-GT4HIST"
```

This allows us to compare the files in `TESS-GT4HIST`, `TESS-DEU` and
`CALA-GT4HIST` with each other and with the GT in `OCR-D-GT-SEG-LINE` (which
also contains text).

## Compare all the OCR results with one another using ocrd-cor-asv-ann-evaluate

```sh
ocrd-cor-asv-ann-evaluate -m data/mets.xml -I TESS-GT4HIST,TESS-DEU,CALA-GT4HIST -O EVAL-ASV-OCR
```

The results are JSON files in the `EVAL-ASV-OCR` filegroup with line-by-line distance measures between all the engine.

## Run tesseract on the GT segmentation

Now we want to compare not the output with the ground truth. We'll reuse the segmentation of the GT for this

```sh
ocrd-tesserocr-recognize -m data/mets.xml -I OCR-D-GT-SEG-LINE -O GT-TESS -P model Fraktur_GT4HistOCR
```

And we'll compare them with dinglehopper:

```
ocrd-dinglehopper -m data/mets.xml -P textequiv_level line -I GT-TESS,OCR-D-GT-SEG-LINE -O EVAL-DINGLE-GT-TESS
```

The result are HTML files (Diff View) and JSON files (with CER and WER).

## Visualize with browse-ocrd

> Show diff view in browse-ocrd (https://github.com/hnesk/browse-ocrd/tree/diff-view)
