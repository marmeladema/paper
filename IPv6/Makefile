all: IPv6.pdf

IPv6.pdf: IPv6.html
	./wkhtmltopdf-amd64				\
		--margin-top 0				\
		--margin-left 0				\
		--margin-bottom 0			\
		--margin-right 0			\
		--orientation Landscape		\
		--page-height 128.0mm		\
		--page-width 96.0mm			\
		--dpi 300					\
		--footer-right	[page]/[topage] \
		IPv6.html IPv6.pdf

clean:
	rm -rf IPv6.pdf

#--footer-right '[page]/[toPage]'\
