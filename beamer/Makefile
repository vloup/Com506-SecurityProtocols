all: beamer.tex
	latexrun beamer.tex
	rm -r latex.out/

help:
	@echo "Usage:"
	@echo "make [all]	Compile LaTeX"
	@echo "make help	Show this help"
	@echo "make clean	Clean generated files"

clean:
	@rm -rf latex.out
