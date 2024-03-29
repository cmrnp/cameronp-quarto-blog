---
title: "Some Quarto PDF formatting tips, with particular reference to thesis writing"
author: "Cameron Patrick"
date: "2023-07-17"
draft: false
categories: [quarto, latex]
format:
  html:
    toc: true
abstract: |
  I've just spent most of a day faffing around trying to get Quarto to produce
  a nice PDF file that I like the look of and which meets my university's 
  formatting requirements for a PhD thesis.
  Maybe my pain and suffering can help reduce yours.
---

I decided to write my PhD thesis in [Quarto](https://quarto.org/). Between my  idiosyncratic standards for the PDF output and the university's rigid requirements for what a PhD thesis ought to look like, I knew that some degree of tweaking the formatting settings would be required. I've never really used Quarto for PDF output before, but since I'm fairly confident using LaTeX, I figured "how hard could it possibly be".

This post is a slightly elaborated version of the notes I made for myself during the process, shared in the hope that it may be useful for others who want to use Quarto to write reports or theses with idiosyncratic styling.

The one piece of this post that I haven't seen spelled out anywhere else on the web is the section on [additional front matter before and after the table of contents](#sec-additional-front-matter), which took a bit of LaTeX trickery to achieve. The solution I found is better than the other alternatives I've seen (for my purposes, anyway), because it allows you to write the front matter in Markdown instead of LaTeX and have it present in the HTML output as well as PDF.

## Basic principles

- Start by creating a "Quarto book project" using your favourite IDE (RStudio)
- It's probably a good idea to use git and periodically commit as you mess with stuff, but I just YOLO'ed it until I had something I was happy with
- You should probably [set up `renv`](https://rstudio.github.io/renv/articles/renv.html) so your package versions are tracked and reproducible. `renv::init()`
- Most of the configuration begins in editing the YAML file, `_quarto.yml`
- The most basic but also most important options are documented in the [Quarto PDF Options](https://quarto.org/docs/reference/formats/pdf.html) section of the Quarto manual
- Achieving more complex goals requires writing chunks of LaTeX code which you reference in the YAML (or sometimes include inline in your chapter `.qmd` files)
- Some of what I'm explaining here wasn't documented and required reading the Quarto source code. I am not to be held responsible for any voided warranty or code that stops working with the next release of Quarto
- I'm assuming you have some familiarity with YAML, Markdown, and LaTeX; but not necessarily with Quarto. In other words, I'm writing for me-last-week. Good luck

## Bibliography workflow

Thanks to [Brenton Wiernik](https://twitter.com/bmwiernik/status/1674046248041721856) and [Matthijs Hollanders](https://twitter.com/realHollanders/status/1673994887736487939) on Twitter for pointing me in the right direction here.

- Use [Zotero](https://www.zotero.org/) and the [Better BibTeX for Zotero](https://retorque.re/zotero-better-bibtex/) plugin
- (Aside: the [ZotFile](http://zotfile.com/) plugin may also be useful if/when I exceed my free Zotero PDF storage limit, although I like Zotero enough I may well just give them cash for cloud storage.)
- Set a sensible schema for naming your reference keys in the Better BibTeX settings. I'm using `auth.lower + year + shorttitle.lower` which generates keys like `@rubin1974estimatingcausaleffects`
- Set Zotero "quick copy" format to Better BibTeX Citation Key and set it to use Markdown, so you can hit Command-Shift-C to copy the citation in Markdown format
- Export using the Better CSL YAML plugin (you can also set it to automatically update when your Zotero library changes), or use the [R Better BibTeX](https://github.com/paleolimbot/rbbt) package to get an automatically updated bibliography export every time you knit your document (I haven't tried this yet)
- Grab a CSL file for your preferred bibliography and citation format and copy it into your Quarto project
- Add `bibliography` and `csl` top-level keys to your YAML:

```yaml
bibliography: references.yaml
csl: apa.csl
```

I've also edited my `references.qmd` Markdown file which generates the bibliography so it has a ragged right margin (i.e. left justified, not full justified):

```markdown
# References {.unnumbered}

\begingroup
\raggedright
::: {#refs}
:::
\endgroup
```

## Yeeting the RStudio visual editor

If you're like me, and can't stand the RStudio visual Markdown editor but accidentally clicked the "Visual editor" check box when creating the project, change your YAML to say:

```yaml
editor: source
```

Instead of `editor: visual`. Or vice versa, if you prefer the visual editor.

## Easy PDF tweaks (only YAML needed)

All of these go inside the `format:` → `pdf:` chunk of the YAML. They're documented in the Quarto manual but that's very long and sometimes unclear (partly because Quarto can produce PDF output both via LaTeX and via HTML, and different options apply to each). Here's what I cared about enough to mess with:

- LaTeX document class: Quarto's default templates will take advantage of the KOMA Script classes if you use those, and they seem to make some customisation nicer than the LaTeX packages I've used previously. So `documentclass: scrbook` for two-sided or `documentclass: scrreprt` for single-sided (note the lack of "o" in "scrreprt")
- You can change the name of the PDF file produced, e.g. `output-file: "FirstnameLastname_thesis.pdf"` (this one goes under the `book:` top-level YAML section, not under `format: pdf:`)
- Set `keep-tex: true` so you can take a squiz at the generated TeX file. I found this helped when figuring out what I needed to change to bend the output to my whims
- Enable Table of Contents, List of Figures, List of Tables; `toc-depth` of three means that the Table of Contents will show up to `\subsection` (or `###` headings in Markdown):
```yaml
    toc: true
    toc-depth: 3
    toc-title: "Table of contents"
    lof: true
    lot: true
```
- Section numbering: `number-depth` appears to count differently from `toc-depth`, so the below will have numbered `\subsection` (`###`) but not `\subsubsection` (`####`):
```yaml
    number-sections: true
    number-depth: 2
```
- Paper size: `papersize: a4` or you'll get US Letter
- Margins: you can either use the KOMA Script options (see below) which use some kind of fancy formula to derive margins, or you can specify options to the LaTeX `geometry` package in your YAML. Below is what I'm currently using, copied from my MSc thesis LaTeX preamble. I can't remember what the header and footer bits do exactly. `heightrounded` helps prevent "underfull vbox" warnings by making sure the text height is a multiple of the line height

```yaml
    geometry:
      - inner=3cm
      - outer=4cm
      - top=3cm
      - bottom=4cm
      - headsep=22pt
      - headheight=11pt
      - footskip=33pt
      - ignorehead
      - ignorefoot
      - heightrounded
```

- Indented paragraphs vs space between paragraphs: `indent: true` or `indent: false`. You can use KOMA Script options for greater control over the indent or skip distances but the defaults look fine to me
- Spacing between lines: you can use e.g. `linestretch: 1.25` or `linestretch: 1.5` to get increased line spacing
- As far as I can tell, you can't choose between full justified and ragged right (left justified) in the YAML, you'll need to add LaTeX commands to the preamble (see below)
- Font size: set the base font size used for body text, e.g. `fontsize: 11pt`
- I prefer the XeLaTeX engine: `pdfengine: xelatex` because...
- If you're using the XeLaTeX engine, you can specify any (Unicode, TrueType/OpenType) system font if you don't like the standard Computer Modern Roman look that screams "my document was made with TeX". The [TeX Gyre Math](https://www.gust.org.pl/projects/e-foundry/tg-math) font families provide OpenType math fonts compatible with XeLaTeX that fit well with Times and Palatino, amongst others. Here's an example of using Times New Roman and other common Microsoft fonts, alongside TeX Gyre Termes Math which provides mathematical symbols which blend in nicely with these fonts:
```yaml
    mainfont: "Times New Roman"
    sansfont: "Arial"
    monofont: "Courier New"
    mathfont: "TeX Gyre Termes Math"
```

## KOMA Script options: fonts, headings, headers, and footers

The [KOMA Script manual](https://mirror.aarnet.edu.au/pub/CTAN/macros/latex/contrib/koma-script/doc/scrguide-en.pdf) is comprehensive but inscrutable, and it takes a bit of messing around to find out where to put the options anyway. 

To set these options, you'll need to add them to what Quarto calls the LaTeX header (which I've always known as the LaTeX preamble). That means you need to add a line like `include-in-header: include-in-header.tex` to the PDF format options in your YAML, and then add the code here to a file called `include-in-header.tex`.

Here are a few things I did here:

- Make the headings the same font as the rest of the document, instead of sans serif: `\addtokomafont{disposition}{\rmfamily}`
- Restore the classic LaTeX chapter headings that are two lines, the first saying e.g. "Chapter 2" on a line before the chapter title: `\KOMAoptions{chapterprefix=true,appendixprefix=true}`
- Smaller fonts for headings: `\KOMAoptions{headings=small}`
- If you're fussy about the size of the indent or spacing between paragraphs, you would do that here too
- If you prefer left-justified (ragged right margin) instead of full-justified, you can do that here: `\raggedright`
- Header and footer fonts: normal upright font (instead of slanted, the KOMA Script default) and a smaller size. See [this handy web site](https://www.sascha-frank.com/latex-font-size.html) for more info on LaTeX relative font size commands like `\footnotesize`.
```tex
\setkomafont{pageheadfoot}{\normalfont\normalcolor\footnotesize}
\setkomafont{pagenumber}{\normalfont\normalcolor\footnotesize}
```
- Headers and footers! For this we'll need the `scrlayer-scrpage` package, and commands like `\lefoot[]{}` where "l" starts for left (there's also "c" and "r"), "e" starts for even page (there's also "o"), "foot" for footer (there's also "head"). Inside the square brackets you put what you want on "plain" pages (start of chapter) and inside the curly braces you put what want on pages with a running-head (inside chapters). Here's an example that gives output similar to many technical books: (1) centred page numbers (`\pagemark`) in the footer on the first page of a chapter; (2) page numbers on the outside edge of the header inside chapters; (3) chapter and section titles ("running heads", `\leftmark` and `\rightmark`) on the inside margins:
```tex
\usepackage{scrlayer-scrpage}
\lefoot[]{}
\cefoot[\pagemark]{}
\refoot[]{}
\lofoot[]{}
\cofoot[\pagemark]{}
\rofoot[]{}
\lehead[]{\pagemark}
\cehead[]{}
\rehead[]{\leftmark}
\lohead[]{\rightmark}
\cohead[]{}
\rohead[]{\pagemark}
```
- Headers and footers! If you're using single-sided output, beware: all of your pages will be considered "odd", for some odd reason.

## Title page

My university has a specific requirement for the formatting of title pages, and even if it didn't I'd still want to change the default Quarto title page because it's kind of ugly.

To do this, we will once again need to write some LaTeX code. This time, rather than just adding extra code to the preamble, we'll be replacing some of the built-in [Quarto Pandoc templates](https://quarto.org/docs/journals/templates.html). You can see [what templates are available and what they contain](https://github.com/quarto-dev/quarto-cli/tree/main/src/resources/formats/pdf/pandoc) by looking at the Quarto source code (!). Ignore the seductively-named `title.tex` because to edit the title page you will need to replace `before-body.tex`. Start by adding `before-body.tex` to the `template-partials` list under `format: pdf:` in your YAML (yeesh).

```yaml
    template-partials:
      - before-body.tex
```

Then, in `before-body.tex`, we'll add the code to create the title page. You'll notice that this isn't quite normal LaTeX, there's some kind of crazy templating language going on, with directives inside pairs of `$` signs. I'm not aware of any documentation on this, I just pieced it together from reading other Quarto templates.

The first few lines (copied from the [standard Quarto template](https://github.com/quarto-dev/quarto-cli/blob/main/src/resources/formats/pdf/pandoc/before-body.tex)) enable *front matter* mode in LaTeX, which causes pages to be numbered in roman numerals instead of normal (arabic) digits. Later on, `\mainmatter` will cause the page numbering to restart from 1. The remainder of the code is LaTeX code to generate the title page, with a bit of cleverness to pull the title and author information out of the YAML. Instead of just using `\maketitle` built into LaTeX, we'll make our own title page from scratch.

```tex
$if(has-frontmatter)$
\frontmatter
$endif$
$if(title)$
\cleardoublepage
\thispagestyle{empty}
{\centering
\hbox{}\vskip 0cm plus 1fill
{\Huge\bfseries $title$ \par}
$if(subtitle)$
\vspace{3ex}
{\Large\bfseries $subtitle$ \par}
$endif$
\vspace{12ex}
$for(by-author)$
{\Large\bfseries $by-author.name.literal$ \par}
\vspace{3ex}
{\Large ORCID: $by-author.orcid$ \par}
\vskip 0cm plus 2fill
{\bfseries\large Doctor of Philosophy \par}
\vspace{3ex}
{\bfseries\large $date$ \par}
\vspace{12ex}
$for(by-author.affiliations)$%
$if(it.department)$%
{\bfseries\large $it.department$ \par}
\vspace{3ex}
$endif$%
{\bfseries\large $it.name$ \par}
$endfor$$endfor$%
\vspace{12ex}
{\small Submitted in total fulfilment of the requirements
of the degree of Doctor of Philosophy \par}
}
$endif$
```

To go along with this, you'll also need to provide appropriate information in the `book:` section of the YAML:

```yaml
book:
  title: "PhD thesis main title"
  subtitle: "Clever subtitle probably with a pun"
  author: 
    - name: "Cameron James Patrick"
      orcid: "0000-0002-4677-535X"
      affiliations:
        - name: "The University of Melbourne"
          department: "Department of Paediatrics"
  date: "01 June 2026"
  date-format: "MMMM YYYY"
```

![Here's one I prepared earlier. Or will eventually have prepared, or something. Still have to write the damned thing.](title-page.png)

## Additional front matter before and after the table of contents {#sec-additional-front-matter}

According to my university, a PhD thesis needs to contain the following items, in this order: title page, abstract, authorship declaration, preface, acknowledgements, table of contents, list of tables, list of figures, abbreviations; the body of the thesis; references; and finally appendices.

Unfortunately, the default Quarto template places the table of contents immediately after the title page. To change this, we'll need to edit another "template partial", this time `toc.tex`. In the YAML:

```yaml
format:
  pdf:
    template-partials:
      - before-body.tex
      - toc.tex
```

Then in `toc.tex`:

```tex
$if(toc)$
$if(toc-title)$
\renewcommand*\contentsname{$toc-title$}
$endif$
$if(colorlinks)$
\hypersetup{linkcolor=$if(toccolor)$$toccolor$$else$$endif$}
$endif$
\setcounter{tocdepth}{$toc-depth$}
$endif$
\renewcommand*\listfigurename{List of figures}
\renewcommand*\listtablename{List of tables}
```

If you compare the above to the [standard Quarto `toc.tex` template](https://github.com/quarto-dev/quarto-cli/blob/main/src/resources/formats/pdf/pandoc/toc.tex) you'll see that I've removed a heap of code. Some of that code was for presentations but most importantly I removed the `\tableofcontents`, `\listoffigures` and `\listoftables` commands which actually produce the table of contents and lists of figures and tables. (I've also added some bonus code to make the "List of figures" and "List of tables" headings in sentence case instead of title case, all modern-like.)

We'll also need to stop Quarto from switching from `\frontmatter` to `\mainmatter` at around this point. I couldn't find the partial template that was responsible for this, so instead I added these two lines of LaTeX to the end of `before-body.tex`:

```tex
\let\mainmatterreal\mainmatter
\let\mainmatter\relax
```

The above code effectively neuters the `\mainmatter` command, until we're ready to bring it back to life.

Now we can write the 'chapters' that make our extra front matter. These can be written just like normal Quarto chapters, though I added `{.unnumbered .unlisted}` to the end of the chapter headings so they don't get chapter numbers and aren't included in the table of contents.

At the end of the last section before the table of contents should appear --- `acknowledgements.qmd` in my case --- add the following LaTeX code to generate table of contents (and list of tables and list of figures, if desired):

```tex
\tableofcontents
\listoftables
\listoffigures
```

Finally, at the end of the last front matter section --- `abbreviations.qmd` in my case --- add this code to return the `\mainmatter` command to life and run it, causing the main section of the document to have ordinary page numbers, starting from 1 again:

```tex
\let\mainmatter\mainmatterreal
\mainmatter
```

## HTML output

One nice thing about Quarto is that it can produce multiple output formats from the same input. I find the HTML output particularly convenient for on-screen previewing. I haven't messed with the appearance of the HTML output much, but here's the YAML chunk I'm using at the moment:

```yaml
format:
  html:
    theme: simplex
    fontsize: 1.2em
    linestretch: 1.7
    mainfont: Helvetica Neue, Helvetica, Arial, sans
    monofont: Cascadia Mono, Menlo, Consolas, Courier New, Courier
    backgroundcolor: "white"
    fontcolor: "black"
    knitr:
      opts_chunk:
        dev: "ragg_png"
```

I will probably end up writing a CSS stylesheet at some point as a form of procrastination.

### Setting the output directory

You can change the directory that the HTML and other outputs are saved to. This may be useful if, for example, you want to use GitHub Pages to publish your document as a web site. GitHub Pages expects the HTML to either be in the repository root or a "docs" subdirectory:

```yaml
project:
  type: book
  output-dir: "docs"
```
