---
layout: post
title: LaTeX installation handbook
subtitle: LaTeX安装手册
image: /ico/icon-latex.png
#bigimg: /img/background.png
tags: [latex]
comments: true
---

> This page is stil under construction...  
> 该页面仍在建设中...1

## TeX Live 2014 profile config 3

Add the following contents to *~/.profile*:
```
TEXLIVE_BASE='/usr/local/texlive/2014'
TEXLIVE_BIN="$TEXLIVE_BASE/bin/i386-linux"
TEXLIVE_DOC="$TEXLIVE_BASE/texmfdist/doc"
export PATH="$TEXLIVE_BIN:$PATH"
export MANPATH="$TEXLIVE_DOC/man:$MANPATH"
export INFOPATH="$TEXLIVE_DOC/info:$INFOPATH"
export OSFONTDIR='/usr/share/fonts'
export XINDY_LIBDIR="$TEXLIVE_BIN"
```

## Latex Beamer中文书签

beamer不能使用dvipdfmx来生成pdf所以对中文标签的支持不能通过dvipdfmx来完成。CJKutf8可以很好的完成中文标签tounicode的转换，但是beamer.cls中的定义存在问题。
打开*/usr/local/texlive/2015/texmf-dist/tex/latex/beamer/beamer.cls*，找到

```
\DeclareOptionBeamer{CJK}{\ExecuteOptionsBeamer{cjk}}
\DeclareOptionBeamer{cjk}{
\def\beamer@hypercjk{\hypersetup{CJKbookmarks=true}}
\def\beamer@activecjk{
% Activate all >128 characters.
\count@=127
\@whilenum\count@<255 \do{'%'
\advance\count@ by 1
\lccode`\~=\count@
\catcode\count@=\active
\lowercase{\def~{\kern1ex}}
}
}
}
```

在beamer3.06中是在178行，把
```latex
'%' Activate all >128 characters.
```
改成
```latex
'%' Activate all >＝0x80 characters.
```

然后在上文的最后一个}后加上下面几句：
```latex
\DeclareOptionBeamer{CJKutf8}{\ExecuteOptionsBeamer{cjkutf8}}
\DeclareOptionBeamer{cjkutf8}{'%'
\PassOptionsToPackage{unicode}{hyperref}
\def\beamer@activecjk{
'%' Activate all characters >= 0x80.
\count@=127
\@whilenum\count@<254 \do{'%'
\advance\count@ by 1
\lccode`\~=\count@
\catcode\count@=\active
\lowercase{\def~{\kern1ex}}
}
}
}
```

之后用\documentclass[CJKutf8]{beamer}调用beamer类，并用\usepackage{CJKutf8}来使用CJKutf8宏包，之后按常规使用中文环境，最后用pdflatex编译.tex文档两次即可。
