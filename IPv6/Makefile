.PHONY: clean withversion run

INPUT=IPv6.md TP.md
OUTPUT=IPv6.pdf TP.pdf
IMG_JPG=$(wildcard img/*.jpg)
IMG_PNG=$(wildcard img/*.png)
IMG_HTML=$(wildcard img/*.html)
IMG_SVG=$(wildcard img/*.svg)
IMG_CSS=$(wildcard img/css/*.css)
IMG_GEN_SVG := $(subst img/, gen/, $(IMG_HTML:.html=.svg))
DEP=$(wildcard *.sty) $(IMG_JPG) $(IMG_PNG) $(IMG_HTML) $(IMG_SVG) $(IMG_CSS) $(IMG_GEN_SVG)
THEME=Custom
PANDOC=pandoc -V theme:$(THEME) -V lang:fr-FR --latex-engine xelatex --toc --variable mainfont="DejaVu Serif" --variable sansfont=Arial --filter ./pandoc-svg.py

gen/%.svg: img/%.html $(IMG_CSS)
	wkhtmltoimage -f svg --width 0 $< $@

#$(OUTPUT): $(INPUT) $(DEP)
%.pdf: %.md $(DEP)
	${PANDOC} -st beamer $< -o $@

pdf: $(OUTPUT)

latex: $(INPUT) $(DEP)
	${PANDOC} -st beamer $(INPUT) -o IPv6.tex

withversion: $(INPUT) $(DEP)
	<$(INPUT) ./inject_version | ${PANDOC} -o $(OUTPUT)

all: $(OUTPUT)

run: all
	impressive -t Crossfade --nologo $(OUTPUT)

clean:
	rm -f $(OUTPUT) img/gen/*
