######
#
# A simple Makefile
#
######

######
all::
######
all:: syllabus-old
######
all:: syllabus-alt
######
all:: faster-paced
######

syllabus-old: ; \
pdflatex syllabus-old.tex

syllabus-alt: ; \
pdflatex syllabus-alt.tex

faster-paced: ; \
pdflatex faster-paced.tex

######

clean:: ; rm -f *~
clean:: ; rm -f syllabus-old.aux
clean:: ; rm -f syllabus-old.log
clean:: ; rm -f syllabus-alt.aux
clean:: ; rm -f syllabus-alt.log
clean:: ; rm -f faster-paced.aux
clean:: ; rm -f faster-paced.log
clean:: ; rm -f faster-paced.out
clean:: ; rm -f faster-paced.nav
clean:: ; rm -f faster-paced.snm
clean:: ; rm -f faster-paced.toc
clean:: ; rm -f faster-paced.run.xml  
clean:: ; rm -f faster-paced-blx.bib

######

cleanall:: clean

############ end of [Makefile] ############
