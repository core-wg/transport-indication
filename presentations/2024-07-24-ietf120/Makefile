include shared/Makefile

.generated/components-clip-%.svg: components.svg
	inkscape -o $@ $< --export-id=$* --export-id-only --export-area-page
