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

## Writing Style Guidelines

When writing or editing content for this book, follow these style conventions observed throughout the `.qmd` files:

### Japanese Language Style

#### Sentence endings and tone
- Use です・ます調 (polite form) throughout: である, となる, されたい, すればよい, など
- Common sentence-ending patterns:
  - Explanatory: 〜である, 〜となる, 〜されている
  - Instructional: 〜する, 〜すればよい, 〜されたい
  - Conditional: 〜なら, 〜のとき, 〜場合
- Maintain neutral, instructional tone appropriate for technical educational material

#### Punctuation
- Use full-width Japanese punctuation: 。、
- Use full-width parentheses for Japanese text: ()
- Use half-width parentheses for inline code or English terms: `command()`
- Use full-width colon in Japanese sentences: :

#### Terminology
- Use katakana for foreign technical terms: パッケージ, コマンド, オプション, パラメータ, データフレーム
- Keep English terms in Roman alphabet when referring to code elements: `lm`, `data`, `model`
- Bold important technical terms on first introduction: **外生変数**, **操作変数**, **平均差分法**

### Content Structure

#### Section organization
- Use hierarchical headings: `#`, `##`, `###`
- Start chapters with level 1 heading (`#`) for main chapter title
- Use level 2 (`##`) for major sections
- Use level 3 (`###`) for subsections

#### Code explanation pattern
Follow this consistent pattern when introducing R commands:

1. **Mathematical/conceptual introduction** - Present the model or concept with equations when relevant
2. **Verbal explanation** - Explain what the code does before showing it
3. **Code block** - Show the R code
4. **Output interpretation** - Explain the results or what to observe

Example pattern:
```
単回帰モデルを考える。
$$
y = \alpha + \beta x + u
$$

R で回帰分析を実施するには `lm` 関数を使用する。
第一引数には回帰式を `y ~ x` という形式で指定し、`data` 引数にはデータフレームを指定する。

\```{r}
fm <- lm(y ~ x, data=df)
\```

推定された係数を取り出すには `coef` 関数を使用する。
```

#### Formula and equation formatting
- Use LaTeX math mode for equations: `$$...$$` for display math, `$...$` for inline math
- Use subscripts for indices: `x_i`, `y_{it}`
- Use `\alpha`, `\beta`, `\gamma` for parameters
- Label components clearly: ここで $x$ は説明変数、$y$ は被説明変数、$u$ は誤差項である

### Code Documentation Style

#### Function introduction
When introducing R functions, follow this pattern:
- Mention the function name in backticks: `lm` 関数
- Explain the purpose: 〜を実施するには〜を使用する
- Describe key arguments: 第一引数には〜, `data` 引数には〜
- Show usage example
- Explain how to extract results: 〜を取り出すには〜

Example:
```
`plm()` 関数で `model = "pooling"` を指定する。
`plm(inv ~ value + capital, data = pdata, model = "pooling")` コマンドで、inv を被説明変数、value と capital を説明変数としてプーリングOLS推定を行い、結果を `gp` に格納する。
```

#### Code block attributes
- Always include setup chunk at the beginning:
  ```{r}
  #| label: setup
  #| include: false
  knitr::opts_chunk$set(echo = TRUE, collapse=TRUE)
  ```
- Use `#| eval: false` for code that should not be executed
- Use `#| message: false` and `#| warning: false` when loading packages
- Use descriptive labels: `#| label: setup`

#### Comments in code
- Minimize inline comments in R code blocks
- Use Japanese comments when necessary: `# 係数の取得`
- Prefer explanatory text before or after code blocks rather than inline comments

### Explanatory Patterns

#### Introducing alternatives
When showing equivalent methods or alternative syntax:
```
`coef` 関数を使用する。
`coefficients` 関数でも同じ結果が得られる (コメントアウトしている).

\```{r}
coef(fm)
# coefficients(fm)
\```
```

#### Step-by-step procedures
When explaining multi-step procedures, use numbered lists:
```
以下の手順で実行する:

1. それぞれの内生変数を外生変数と操作変数に回帰させて、その予測値を得る。
2. 被説明変数を外生変数と内生変数の予測値に回帰させて、その係数を得る。
```

#### Assumptions and conditions
When listing assumptions, use bullet points with `+`:
```
以下の仮定を置く。

+ $(x_i, y_i)$ は独立同一分布に従う。
+ $E[u_i] = 0$ である。
+ $u_i$ と $x_i$ は独立である。
```

#### Technical definitions
Define technical terms clearly:
```
誤差項と相関が無い説明変数を **外生変数** といい、誤差項と相関がある説明変数を **内生変数** という。
```

### Cross-references and Links

#### Function references
- Use backticks for function names: `lm()`, `summary()`, `coef()`
- Include parentheses when referring to functions
- Mention package when first introducing specialized functions: `ivreg` コマンドは `AER` パッケージに含まれており

#### File and URL references
- Use full URLs for external references
- Provide Japanese context for English resources:
  ```
  英語であるがこの動画も有益である。

  https://www.datacamp.com/courses/working-with-the-rstudio-ide-part-1
  ```

### Consistency Points

#### Variable naming in examples
- Use consistent variable names across chapters:
  - `fm`, `fm0`, `fm1` for fitted models
  - `df` for data frames
  - `N` for sample size
  - `x`, `y`, `w` for variables
  - `pdata` for panel data frames

#### Output display
- Use `head()` when showing partial output: `head(resid(fm))`
- Note when commenting out alternative methods: `# coefficients(fm)`
- Use `with()` for accessing data frame columns: `with(df, cov(x,y)/var(x))`

#### Notes and attention markers
- Use parenthetical notes with `^[...]` for footnotes
- Use 注意 or ここで for calling attention
- Use たとえば for examples
- Use なお for additional notes

## Deployment

The [docs](docs/) directory contains the rendered HTML output and is configured as the GitHub Pages publishing source. After running `quarto render`, commit the updated docs/ directory to deploy changes to the live site.
