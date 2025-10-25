# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a tutorial book titled "R と RStudio" (R and RStudio) by 宮﨑憲治. The book provides a Japanese-language tutorial covering R programming basics and econometric regression analysis techniques.

The project supports both **Quarto** (recommended) and **R bookdown** build systems.

## Build System

### Quarto (Recommended)

The project has been converted to Quarto format using `.qmd` files.

#### Building with Quarto

To render all formats:
```bash
quarto render
```

To render only HTML:
```bash
quarto render --to html
```

To render only PDF:
```bash
quarto render --to pdf
```

To preview the book while editing (with hot-reload):
```bash
quarto preview
```

To render a single chapter:
```bash
quarto render 01-base.qmd
```

To clean the build:
```bash
quarto clean
```

#### Quarto configuration

- [`_quarto.yml`](_quarto.yml): Main configuration file controlling book structure, output formats, and language settings
- Output directory: [docs](docs/) (configured as GitHub Pages publishing source)
- Supports HTML and PDF outputs with Japanese language support
- Cross-references use Japanese labels (図, 表, 式, etc.) configured in `_quarto.yml`

### R Bookdown (Legacy)

The original R Markdown (`.Rmd`) files are still available for bookdown builds.

#### Building with bookdown

To render all formats:
```r
bookdown::render_book("index.Rmd")
```

To render only the GitBook HTML format:
```r
bookdown::render_book("index.Rmd", "bookdown::gitbook")
```

To render only the PDF format:
```r
bookdown::render_book("index.Rmd", "bookdown::pdf_book")
```

To preview with hot-reload while editing:
```r
bookdown::serve_book(dir = ".", output_dir = "docs")
```

#### Bookdown configuration

- [`_bookdown.yml`](_bookdown.yml): Controls book structure, output directory, chapter ordering, and language labels
- [`_output.yml`](_output.yml): Configures output formats (GitBook, PDF, ePub) and their rendering options
- [`Rforregression.Rproj`](Rforregression.Rproj): RStudio project file (Build Type: Website, LaTeX: XeLaTeX)

## Project Structure

### Dual format system

The book content exists in both formats:
- **Quarto (`.qmd`)**: Primary format, actively maintained
- **R Markdown (`.Rmd`)**: Legacy format, kept for compatibility

When making content changes, update the `.qmd` files. The `.Rmd` files may need manual synchronization if bookdown builds are required.

### Source chapters

The book consists of 11 chapters that compile into a comprehensive tutorial. Each chapter is available in both Quarto (`.qmd`) and R Markdown (`.Rmd`) formats:

1. [index.qmd](index.qmd) - Introduction, R installation, basic setup
2. [01-base.qmd](01-base.qmd) - R basics: calculator functions, types, variables, packages
3. [02-rstudio.qmd](02-rstudio.qmd) - RStudio IDE introduction
4. [03-vector.qmd](03-vector.qmd) - Vectors and data structures
5. [04-dataframe.qmd](04-dataframe.qmd) - Data frames
6. [05-datainput.qmd](05-datainput.qmd) - Data input/import
7. [06-datawrangling.qmd](06-datawrangling.qmd) - Data wrangling techniques
8. [07-regression1.qmd](07-regression1.qmd) - Classical OLS regression (simple and multiple)
9. [08-regression2.qmd](08-regression2.qmd) - Advanced regression topics
10. [09-ivreg.qmd](09-ivreg.qmd) - Instrumental variables and 2SLS regression
11. [10-panel.qmd](10-panel.qmd) - Panel data analysis (fixed effects, random effects)

### Key R packages used

Econometric analysis chapters rely on these packages:
- `AER` - Applied Econometrics with R
- `plm` - Panel data linear models
- `wooldridge` - Data sets from Wooldridge's econometrics textbooks
- `estimatr` - Fast estimators for design-based inference (robust standard errors)

### Supporting files

- [preamble.tex](preamble.tex), [afterbody.tex](afterbody.tex) - LaTeX configuration for PDF output
- [style.css](style.css), [toc.css](toc.css) - CSS styling for HTML output
- [jeconunicode.bst](jeconunicode.bst) - BibTeX style for Japanese economics citations
- [figs/](figs/) - Directory for figures
- [_bookdown_files/](_bookdown_files/) - Cache directory for bookdown
- [AGENTS.qmd](AGENTS.qmd) - Separate guidelines document (not part of book build)

## Language and Typography

The book is written in Japanese (lang: ja-JP) with the following specifications:
- Document class: bxjsbook (Japanese book class)
- LaTeX engine: XeLaTeX (for Japanese character support)
- Encoding: UTF-8

## Econometric Analysis Patterns

When working with regression analysis chapters ([07-regression1.qmd](07-regression1.qmd), [08-regression2.qmd](08-regression2.qmd), [09-ivreg.qmd](09-ivreg.qmd), [10-panel.qmd](10-panel.qmd)), note these patterns:

### Standard OLS workflow
```r
fm <- lm(y ~ x, data=df)
summary(fm)
coef(fm)           # Coefficients
resid(fm)          # Residuals
fitted(fm)         # Fitted values
deviance(fm)       # Residual sum of squares
```

### Instrumental variables regression
```r
library(AER)
fm <- ivreg(log(wage) ~ educ | fatheduc, data=df)  # Single IV
fm <- ivreg(y ~ x1 + x2 | z1 + z2 + x2, data=df)   # 2SLS with exogenous vars
summary(fm, diagnostics = TRUE)  # Includes weak instruments, Wu-Hausman, Sargan tests
```

### Panel data analysis
```r
library(plm)
pdata <- pdata.frame(df, index = c("firm", "year"))
gp <- plm(y ~ x, data=pdata, model="pooling")   # Pooled OLS
gi <- plm(y ~ x, data=pdata, model="within")    # Fixed effects
gr <- plm(y ~ x, data=pdata, model="random")    # Random effects
gf <- plm(y ~ x, data=pdata, model="fd")        # First differences
phtest(gi, gr)  # Hausman test
```

### Robust standard errors
```r
library(AER)
coeftest(fm, vcov=vcovHC)                    # Heteroskedasticity-robust
coeftest(fm, vcov=vcovHC(fm, type="sss"))    # Cluster-robust for panel data
summary(fm, vcov=vcovHC, df=Inf)             # In summary output
```

## Session Configuration

### Quarto
Quarto renders each chapter independently by default. Common knitr options are set globally in [`_quarto.yml`](_quarto.yml):
```yaml
knitr:
  opts_chunk:
    echo: true
    collapse: true
    warning: false
    message: false
```

Individual chapters can override these settings in their setup chunks if needed.

### Bookdown
The bookdown configuration uses `new_session: yes`, meaning each chapter is rendered in a fresh R session. This prevents variable conflicts but requires each chapter to load necessary packages independently.

Each chapter includes its own setup chunk:
```r
knitr::opts_chunk$set(echo = TRUE, collapse=TRUE)
```

## Deployment

The [docs](docs/) directory contains the rendered HTML output and is configured as the GitHub Pages publishing source. After running `quarto render`, commit the updated docs/ directory to deploy changes to the live site.
