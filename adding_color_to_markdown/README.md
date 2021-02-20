# Adding colors to Pandoc's markdown

I often find myself in a position where I would like to add some color to my
markdown files. Unfortunately, the only way you can do that in Pandoc's
markdown is to embed HTML or LaTex in your file. While this is indeed a
solution, color information gets lost when converting to different formats. For
example, if you use HTML to add color to text objects and you compile your file
to PDF, the color information is lost.

There are various ways we can work around this issue. The most straight-forward
way is to make use of Pandoc filters, specially Lua filters, since Pandoc comes
preinstalled with a Lua interpreter.

This script solves two issues:

1. It makes it possible to preserve color information when converting from
   markdown to HTML and LaTex.

2. It provides a more convenient syntax to add color information compared to
   both HTML and LaTex.

## Syntax

Color information can be added in three ways:

1. `[text_to_color]{fg=color_name}` - This changes the foreground color to
   `color_name`.
2. `[text_to_color]{bg=color_name}` - This changes the background color to
   `color_name`.
3. `[text_to_color]{fg=color1 bg=color2}` - This changes the foreground color
   to `color1` and background color to `color2`.

### Dependencies

Setting the background color in LaTex requires more work than necessary
especially when the text spans multiple lines. To keep things simple, I chose
to use the `soul` package which supports line wrapping. You will need to add
`soul` and `xcolor` to your LaTex distribution, if they're not already
included. Furthermore, you will have to add the snippet below in your
`preamble` for the compiler to know what to do with these new commands.

```tex
\usepackage{soul}
\usepackage{xcolor}

\newcommand{\bgcolor}[2]{
    \begingroup
    \definecolor{hlcolor}{HTML}{#1}
    \sethlcolor{hlcolor}
    \hl{#2}
    \endgroup
}

\newcommand{\fgcolor}[2]{
    \begingroup
    \definecolor{mycolor}{HTML}{#1}
    \textcolor{mycolor}{#2}
    \endgroup
}

\newcommand{\bgfgcolor}[3]{
    \begingroup
    \definecolor{hlcolor}{HTML}{#1}
    \definecolor{mycolor}{HTML}{#2}
    \sethlcolor{hlcolor}
    \textcolor{mycolor}{\hl{#3}}
    \endgroup
}
```

## Usage Example

Assuming the Lua script is located at `path_to_lua_script` and your
`preamble.tex` is located at `path_to_preamble`, this is how you can use the
script (without the linebreaks).

### Converting to HTML

```sh
pandoc -s
       -L path_to_lua_script
       your_file.md -o your_file.html
```

### Converting to PDF

Note that I am using `xelatex` as my pdf-engine. You can replace it with the
one you're using.

```sh
pandoc -H path_to_preamble
       --pdf-engine=xelatex --pdf-engine-opt=--shell-escape --pdf-engine-opt=-output-directory=temp
       -L path_to_lua_script
       your_file.md -o your_file.pdf
```
