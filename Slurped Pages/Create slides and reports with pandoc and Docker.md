---
link: https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker
excerpt: Pandoc is a free and open-source document converter that can convert various markup languages into different formats. It supports conversion from and to formats such as Markdown, reStructuredText, LaTeX, HTML, Word docx, and more. It is an incredibly powerful tool, which we use extensively to create PDF slides or reports from easy to maintain markdown sources. Pandoc is also highly customizable through the use of templates and filters.
slurped: 2026-05-19T08:32
title: Create slides and reports with pandoc and Docker
tags:
  - docker
---

Pandoc is a free and open-source document converter that can convert various markup languages into different formats. It supports conversion from and to formats such as Markdown, reStructuredText, LaTeX, HTML, Word docx, and more. It is an incredibly powerful tool, which we use extensively to create PDF slides or reports from easy to maintain markdown sources. Pandoc is also highly customizable through the use of templates and filters.

Under the hood, pandoc uses LaTex to produce PDF outputs. So the installation may be a little tricky as it requires to install LaTex, pandoc itself plus filters with matching versions. To make processing easier, we use a custom docker image that contains pandoc, LaTex and filters. Here is how to use this Docker image to produce slides and reports.
## The Docker image[](https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker#the-docker-image)

`cylab/pandoc` is a custom Docker image that contains pandoc (obviously) plus some useful filters (extensions):

- `pandoc-include` allows to split the source between multiple files;
- `pandoc-crossref` allows to create cross-references in the document;
- `pandoc-latex-fontsize` allows to modify the size of some elements;
- `pandoc-latex-environment` allows to create code blocks.

The image also includes a few additional tools:

- `watch` and `make`
- `pdftk` can be used to merge or modify the produced PDF;
- `plantuml` allows to create UML graphs.

Here is how to use the image to produce reports and slides…

## Report[](https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker#report)

Here is a very simple **report.md** file that can be used to write a report:

```
---
title: My report
author: Thibault Debatty
date: \today

documentclass: report
bibliography: bibliography.bib
geometry:
  - margin=2cm
  - b5paper
fontfamily: mathptmx
fontsize: 12pt
toc: true
codeBlockCaptions: True

pandoc-latex-fontsize:
  - classes: [tiny]
    size: tiny
  - classes: [scriptsize]
    size: scriptsize
  - classes: [footnotesize]
    size: footnotesize
  - classes: [small]
    size: small

pandoc-latex-environment:
  noteblock: [note]
  tipblock: [tip]
  warningblock: [warning]
  cautionblock: [caution]
  importantblock: [important]
header-includes:
  - \usepackage{awesomebox}
---

# Usage

## With a makefile
```

You can compile this to a PDF with the following command:

```
docker run --rm -v ${PWD}:/work -w /work \
  -u $(id -u ${USER}):$(id -g ${USER}) cylab/pandoc  \
  pandoc --number-sections -t latex --top-level-division part --toc-depth 2 \
  -F pandoc-include -F pandoc-crossref -F pandoc-latex-fontsize -F pandoc-latex-environment \
  --citeproc --fail-if-warnings \
  -o report.pdf report.md
```

[![pandoc-report.png](https://cylab.be/storage/blog/424/files/29mjevlUCXm6yR8m/pandoc-report.png)](https://cylab.be/storage/blog/424/files/29mjevlUCXm6yR8m/pandoc-report.png)

As this is obviously a long command line, it is usually easier to **create a Makefile**

```
#
# Use docker image cylab/docker to create a PDF report
# https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker
#
 
default: docker

docker:
	docker run --rm -v ${PWD}:/work -w /work -u $(id -u ${USER}):$(id -g ${USER}) cylab/pandoc make report.pdf

report.pdf: report.md
	pandoc --number-sections -t latex --top-level-division part --toc-depth 2 -F pandoc-include -F pandoc-crossref -F pandoc-latex-fontsize -F pandoc-latex-environment --citeproc --fail-if-warnings -o report.pdf report.md
```

Which means you can now compile your report with a simple

```
make
```

## Slides[](https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker#slides)

Likewise, pandoc and the pandoc image can be used to create slides. Here is an example **slides.md**

```
---
title: "Example slides"
subtitle: My awesome slides
author: Thibault Debatty
date: \today
toc: true

pandoc-latex-fontsize:
  - classes: [tiny]
    size: tiny
  - classes: [scriptsize]
    size: scriptsize
  - classes: [footnotesize]
    size: footnotesize
  - classes: [small]
    size: small

pandoc-latex-environment:
  noteblock: [note]
  tipblock: [tip]
  warningblock: [warning]
  cautionblock: [caution]
  importantblock: [important]
header-includes:
  - \usepackage{awesomebox}
---

# Introduction

## Slide 1

My first slide...
```

Which you can compile with:

```
docker run --rm -v ${PWD}:/work -w /work \
  -u ${UID}:${GID} cylab/pandoc pandoc \
  --slide-level 2 -f markdown -t beamer --fail-if-warnings \
  -V classoption:aspectratio=169 \
  --filter pandoc-include --filter pandoc-crossref --filter pandoc-latex-environment \
  --filter pandoc-latex-fontsize \
  -o slides.pdf slides.md
```

Similarly, you can create a **Makefile** to help you:

```
#
# Use docker image cylab/docker to create PDF slides
# https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker
#
 
default: docker

docker:
	docker run --rm -v ${PWD}:/work -w /work -u $(id -u ${USER}):$(id -g ${USER}) cylab/pandoc make slides.pdf

slides.pdf: slides.md
	pandoc --slide-level 2 -f markdown -t beamer --fail-if-warnings -V classoption:aspectratio=169 --filter pandoc-include --filter pandoc-crossref --filter pandoc-latex-environment --filter pandoc-latex-fontsize -o slides.pdf slides.md
```

## Filters[](https://cylab.be/blog/424/create-slides-and-reports-with-pandoc-and-docker#filters)

As mentioned, the Docker image also has some pre-installed pandoc filters. Here is a quick reference on how to use them…

### pandoc-include

`pandoc-include` allows to split the content of your document in multiple file. You can then include a file with

```
!include section-01.md
```

[https://github.com/DCsunset/pandoc-include](https://github.com/DCsunset/pandoc-include)

### pandoc-latex-environment

`pandoc-latex-environment` allows to create LaTeX environement blocks. This allows to add attention blocks in the document.

For example, the following code:

```
## Slide 1

My first slide...


::: note
Read more online : [https://cylab.be](https://cylab.be)
:::
```

Will be rendered as

[![pandoc-latex-environment.png](https://cylab.be/storage/blog/424/files/Gys084ARTYJe2GPu/pandoc-latex-environment.png)](https://cylab.be/storage/blog/424/files/Gys084ARTYJe2GPu/pandoc-latex-environment.png)

By default the following environment are defined, but you can create others:

- `note`
- `tip`
- `warning`
- `caution`
- `important`

[https://github.com/chdemko/pandoc-latex-environment](https://github.com/chdemko/pandoc-latex-environment)

### pandoc-crossref

`pandoc-crossref` allows to attach labels to figures, equations, tables or sections, and cite them from somewhere else in the document:

```
## Docker {#sec:docker}

## Image

![Caption](image-01.jpg){#fig:image-01}

## Equation

$$ math $$ {#eq:equation-01}

## Table

a   b   c
--- --- ---
1   2   3
4   5   6

: Caption {#tbl:table-01}

## References

See @fig:image-01 and [@sec:docker; @eq:equation-01; @tbl:table-01].
```

[![pandoc-crossref.png](https://cylab.be/storage/blog/424/files/Hf6KT5r2tKhTKj6F/pandoc-crossref.png)](https://cylab.be/storage/blog/424/files/Hf6KT5r2tKhTKj6F/pandoc-crossref.png)

[https://github.com/lierdakil/pandoc-crossref](https://github.com/lierdakil/pandoc-crossref)

### pandoc-latex-fontsize

`pandoc-latex-fontsize` allows to modify the size of some elements. This can be useful for code blocks for example.

````
```tiny
some tiny code
``

```scriptsize
some scriptsize code
``

```footnotesize
some footnotesize code
``

```small
some small code
``
````

You can also combine a custom size with syntax highlighting using the following syntax:

````
``` {.php .scriptsize}
$var = "Hello";
echo $var . " CYLAB!"
``
````

[![pandoc-latex-fontsize.png](https://cylab.be/storage/blog/424/files/eNLVMrYUxZAPbniu/pandoc-latex-fontsize.png)](https://cylab.be/storage/blog/424/files/eNLVMrYUxZAPbniu/pandoc-latex-fontsize.png)

[https://github.com/chdemko/pandoc-latex-fontsize](https://github.com/chdemko/pandoc-latex-fontsize)
