---
sidebar_position: 1
title: "Archivos PDF"
id: archivos-pdf
---

## Comprimnir con ghostscript

- [Fuentes](https://www.digitalocean.com/community/tutorials/reduce-pdf-file-size-in-linux)

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -dNOPAUSE -dQUIET -dBATCH -sOutputFile=output/output.pdf output.pdf
```

## Dividir

```bash
pdfseparate -f 1 -l 5 input.pdf output-page%d.pdf
```
