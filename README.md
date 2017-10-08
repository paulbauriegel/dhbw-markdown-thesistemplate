# Markdown-Pandoc Template for DHBW papers

## About

This repository contains my template for all the papers I write at the DHBW Mannheim. The documents are written in markdown syntax and converted by pandoc in a LaTeX PDF file.

The code is based on [the LaTeX Template from dhbw-horb](https://github.com/dhbw-horb/latexVorlage).

## Create the document

```
pandoc --from=markdown+hard_line_breaks+backtick_code_blocks+abbreviations --top-level-division=chapter --template=template.tex --biblatex --latex-engine=lualatex --listings --filter=pandoc-crossref input.md -o output.tex
```

## Syntax

**References**: 

- Literatur:   ```[@BibTexKey] z.B. [@Saha.2016] ```
- Figures: ````[@fig:ImgID] z.B. [@fig:NLQ] ````

**Image**:   ```![Natural Language Querying Application Architecture](images/NLQ-Architecture.png){#fig:NLQ caption="Natural Language Querying Application Architecture" width=80%}```

**Math**: ```$A^{-}$ ```

**CodeBlock**: 

```
​```{caption="Data Transformation Logic: An example"}
SELECT DISTINCT GENERATE_UNIQUE() OVER() ID,
      SEC.Address.country as hasName
INTO FIBO.Country
FROM SEC.Address
​```
```



