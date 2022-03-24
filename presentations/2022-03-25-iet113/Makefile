slides-for-meetecho.pdf: slides.pdf
	# Workaround for meetecho doing slides at fixed DPI rather than fixed
	# size in pixels
	#
	# Thanks Kurt Pfeifle at https://stackoverflow.com/a/7507511
	gs \
	 -o $@ \
	 -sDEVICE=pdfwrite \
	 -g16000x9000 \
	 -dFIXEDMEDIA \
	 -dPDFFitPage \
	 -dCompatibilityLevel=1.4 \
	  $<

IMG_20220323_160047-bright.jpg: IMG_20220323_160047.jpg
	convert $< -brightness-contrast 20x20 $@

slides.pdf: slides.tex IMG_20220323_160047-bright.jpg
	lualatex $<
