all: spec.html framing.html

clean:
	rm spec.html framing.html flow0.svg

spec.html: the-spec.md flow0.svg
	pandoc --toc -c swiss.css -o spec.html -s the-spec.md

framing.html: framing.md
	pandoc -c swiss.css -o framing.html -s framing.md

flow0.svg: flow0.dot
	dot -Tsvg $< > $@
