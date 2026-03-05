LATEXMK=latexmk
MAIN=src/main.tex

.PHONY: pdf clean glossary

pdf:
	$(LATEXMK) -lualatex -interaction=nonstopmode -file-line-error $(MAIN)

glossary:
	makeglossaries src/main

clean:
	$(LATEXMK) -C $(MAIN)
	rm -f src/main.acn src/main.acr src/main.alg src/main.glo src/main.gls src/main.ist
