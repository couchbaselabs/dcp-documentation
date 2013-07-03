all: spec.html framing.html

clean:
	rm spec.html framing.html flow0.dot.svg

specdias = flow0.dot.svg

spec.html: the-spec.md $(specdias)
	pandoc --toc -o spec.html -s the-spec.md

framing.html: framing.md
	pandoc -o framing.html -s framing.md

%.dot.svg: %.dot
	dot -Tsvg $< > $@
