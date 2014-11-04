# About texsvg

texsvg takes LaTeX-compatible equations and produces vector-based SVG images (via LaTeX / dvisvgm).

texsvg is based on texvc which was originally created by Tomasz Wegrzanowski for the [MediaWiki Math Extension](http://www.mediawiki.org/wiki/Extension:Math).

Please report bugs at: https://github.com/ckepper/texsvg/issues

The software is licensed under the GNU GPL license.

## Setup

### Requirements

To build and run the software, the following components are required:

- OCaml
- LaTeX
- dvisvgm

texsvg is written in OCaml. OCaml 3.06 or later is required to compile texsvg; this can be acquired from http://caml.inria.fr/ if your system doesn't have it available.

The makefile requires GNU make.

Image conversion is done via LaTeX and dvisvgm. These tools have to be installed and in the PATH: latex, dvisvgm

AMS* packages for LaTeX also need to be installed. Without AMS* some equations will render correctly while others won't render. Most distributions of TeX already contain AMS*. 

To maximize SVG file portability, all fonts/glyphs are converted to paths.

To work properly with rendering non-ASCII Unicode characters, a supplemental TeX package is needed (cjk-latex in Debian)

### Installation

#### Ubuntu

In Ubuntu, all dependencies can be installed using:

  	$ sudo apt-get install build-essential dvisvgm ocaml texlive-fonts-recommended texlive-lang-greek texlive-latex-recommended

#### Mac

On a Mac, OCaml can be installed quite easily with [Homebrew](http://brew.sh)

	$ brew install ocaml

Moreover, the installation does not require a full TeXLive installation - the [BasicTeX package](https://tug.org/mactex/morepackages.html) from MacTeX will do if you install a few additional modules.

	$ sudo tlmgr update --self
	$ sudo tlmgr install collection-fontsrecommended
	$ sudo tlmgr install dvipng
	$ sudo tlmgr install cancel
    $ sudo tlmgr install AMSmath
	$ sudo tlmgr install dvisvgm

Run 'make' (or 'gmake' if GNU make is not your default make) in the texsvg directory to produce the texsvg executable. Afterwards you probably want to put it in your path.


## Usage

texsvg can be run manually for testing or used in another app.


### Command-line parameters

    texsvg <temp directory> <output directory> <TeX code> <encoding> <color>

Be sure to properly quote the TeX code!

Example:

    texsvg ~/tmp ~/math "y=x+2" iso-8859-1 "rgb 1.0 1.0 1.0"

### Output format

Status codes and HTML/MathML transformations are returned on stdout.
A SVG file will be written to the output directory and named after the MD5 hash code of the TeX formula.

texsvg output format is like this:

    +%5		ok, but not html or mathml
    c%5%h	ok, conservative html, no mathml
    m%5%h	ok, moderate html, no mathml
    l%5%h	ok, liberal html, no mathml
    C%5%h\0%m	ok, conservative html, with mathml
    M%5%h\0%m	ok, moderate html, with mathml
    L%5%h\0%m	ok, liberal html, with mathml
    X%5%m	ok, no html, with mathml
    S		syntax error
    E		lexing error
    F%s		unknown function %s
    -		other error

    \0 - null character
    %5 - md5, 32 hex characters
    %h - html code, without \0 characters
    %m - mathml code, without \0 characters


## Troubleshooting

Try running texsvg from the command line to ensure that the software it relies upon is set up correctly.

Ensure that the temporary and output directories exist and can be written to by the user account - if no output files are created, you should check your permissions.

If some equations render correctly while others don't, you probably don't have AMS* packages for LaTeX installed. Most distributions of TeX come with AMS*.

In Debian/Ubuntu AMS* is in tetex-extra package.

To check if that is the problem you can try those two equations:

    x + y
    x \implies y

The first uses only standard LaTeX, while the second uses symbol \implies from AMS*.

If the first renders, but the second doesn't, you need to install AMS*.

## Hacking

Before you start hacking on the math package its good to know the workflow,
which is basically:

1. texvc gets called by Math/Math.body.php (check out the line begining with "$cmd")
2. texvc does its magic, which is basically to check for invalid latex code.
3. texvc takes the user input if valid and creates a latex file containing it, see
   get_preface in texutil.ml
4. dvipng(1) gets called to create a .png file
   See render.ml for this process (commenting out the removal of
   the temporary file is useful for debugging).
