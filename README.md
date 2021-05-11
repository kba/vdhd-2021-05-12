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

This workflow uses `ocrd-olena-binarize` (with the `sauvola-ms-split`
algorithm) to binarize the images. The images are processed by two runs with
tesseract (once `Fraktur_GT4HistOCR`, once `deu`) and once with calamari (with
the `qurator-gt4histocr-1.0` model).

```sh
ocrd process -m data/mets.xml \
  "olena-binarize -I OCR-D-GT-SEG-LINE -O BIN" \
  "tesserocr-recognize -P segmentation_level word -P textequiv_level line -P find_tables true -P model Fraktur_GT4HistOCR -I BIN -O TESS-GT4HIST" \
  "tesserocr-recognize -P segmentation_level word -P textequiv_level line -P find_tables true -P model deu -I BIN -O TESS-DEU" \
  "calamari-recognize -P checkpoint_dir qurator-gt4histocr-1.0 -I BIN -O CALA-GT4HIST"
```

This allows us to compare the files in `TESS-GT4HIST`, `TESS-DEU` and
`CALA-GT4HIST` with each other and with the GT in `OCR-D-GT-SEG-LINE`.

## Compare all the OCR results with the GT using ocrd-cor-asv-ann-evaluate

```sh
ocrd-cor-asv-ann-evaluate -m data/mets.xml -I OCR-D-GT-SEG-LINE,TESS-GT4HIST,TESS-DEU,CALA-GT4HIST -O EVAL-ASV
```

The results are JSON files in the `EVAL-ASV` filegroup with line-by-line distance measures between all the engine.

`data/EVAL-ASV/EVAL-ASV.json` contains the metrics (mean CER and variance) for the full workspace:

```json
{
  "OCR-D-GT-SEG-LINE,TESS-GT4HIST": {
    "length": 110,
    "distance-mean": 0.032638315863287554,
    "distance-varia": 0.010120613640730372
  },
  "OCR-D-GT-SEG-LINE,TESS-DEU": {
    "length": 110,
    "distance-mean": 0.17414861150552538,
    "distance-varia": 0.030377095996637286
  },
  "OCR-D-GT-SEG-LINE,CALA-GT4HIST": {
    "length": 110,
    "distance-mean": 0.044792427193718676,
    "distance-varia": 0.01339440642349274
  }
}
```

`data/EVAL-ASV/EVAL-ASV_0003.json` contains the metrics for the third page

## Compare Calamari output with GT using dinglehopper

```sh
ocrd-dinglehopper -m data/mets.xml -P textequiv_level line -I OCR-D-GT-SEG-LINE,CALA-GT4HIST -O EVAL-DINGLE
```

The result are HTML files (Diff View) and JSON files (with CER and WER).

## Visualize with browse-ocrd

> Show diff view in browse-ocrd (https://github.com/hnesk/browse-ocrd/tree/diff-view)
