## Process this file with automake to create Makefile.in
## Configure input file for elfutils.
##
## Copyright (C) 1996-2002, 2003, 2004 Red Hat, Inc.
##
## This program is Open Source software; you can redistribute it and/or
## modify it under the terms of the Open Software License version 1.0 as
## published by the Open Source Initiative.
##
## You should have received a copy of the Open Software License along
## with this program; if not, you may obtain a copy of the Open Software
## License version 1.0 from http://www.opensource.org/licenses/osl.php or
## by writing the Open Source Initiative c/o Lawrence Rosen, Esq.,
## 3001 King Ranch Road, Ukiah, CA 95482.
##
DEFS = -D_GNU_SOURCE -DHAVE_CONFIG_H $(YYDEBUG) \
       -DSRCDIR=\"$(shell cd $(srcdir);pwd)\" -DOBJDIR=\"$(shell pwd)\"
if MUDFLAP
AM_CFLAGS = -Wall -Wshadow -std=gnu99 \
	    $(native_ld_cflags)
else
AM_CFLAGS = -Wall -Wshadow -std=gnu99 \
	    $(if $($(*F)_no_Werror),,-Werror) $(native_ld_cflags)
endif
if MUDFLAP
AM_LDFLAGS = -fmudflap
endif
INCLUDES = -I$(srcdir) -I$(srcdir)/../libelf -I$(srcdir)/../libebl -I$(srcdir)/../lib -I$(srcdir)/../libdw  -I..

YACC = @YACC@ -d
AM_YFLAGS = -pld
AM_LFLAGS = -Pld -olex.yy.c
## Uncomment to enable debugging of linker script parser
##YYDEBUG = -DYYDEBUG=1

native_ld = @native_ld@
base_cpu = @base_cpu@

bin_PROGRAMS = readelf nm size strip ld elflint


ld_dsos = libld_elf_i386_pic.a
if NATIVE_LD
noinst_LIBRARIES = libld_elf.a
native_ld_cflags = -DBASE_ELF_NAME=elf_$(base_cpu)
else
noinst_LIBRARIES = libld_elf.a $(ld_dsos)
noinst_PROGRAMS = $(ld_dsos:_pic.a=.so)
endif


ld_SOURCES = ld.c ldgeneric.c ldlex.l ldscript.y symbolhash.c sectionhash.c \
	     versionhash.c

noinst_HEADERS = ld.h symbolhash.h sectionhash.h versionhash.h \
		 ldscript.h xelf.h unaligned.h

EXTRA_DIST = elf32-i386.script libld_elf_i386.map $(ld_modules)
ld_modules = i386_ld.c

if MUDFLAP
libdw = ../libdw/libdw.a
libelf = ../libelf/libelf.a
else
libdw = ../libdw/libdw.so
libelf = ../libelf/libelf.so
endif
libebl = ../libebl/libebl.a
libeu = ../lib/libeu.a

readelf_LDADD = $(libdw) $(libebl) $(libelf) $(libeu) -ldl
nm_LDADD = $(libdw) $(libebl) $(libelf) $(libeu) -ldl
size_LDADD = $(libelf) $(libeu)
strip_LDADD = $(libebl) $(libelf) $(libeu) -ldl
ld_LDADD = $(libebl) $(libelf) $(libeu) -ldl
if NATIVE_LD
ld_LDADD += libld_elf.a
endif
ld_LDFLAGS = -rdynamic
elflint_LDADD  = $(libebl) $(libelf) $(libeu) -ldl

ldlex.o: ldscript.c
ldlex_no_Werror = yes
ldscript.h: ldscript.c

# Machine-specific linker code.
libld_elf_a_SOURCES = $(base_cpu)_ld.c

libld_elf_i386_pic_a_SOURCES =
am_libld_elf_i386_pic_a_OBJECTS = i386_ld.os

libld_elf_i386_so_SOURCES =
libld_elf_i386.so: libld_elf_i386_pic.a libld_elf_i386.map
	$(CC) -shared -o $@ -Wl,--whole-archive,$<,--no-whole-archive \
	      $(libelf) $(libeu) \
	      -Wl,--version-script,$(srcdir)/libld_elf_i386.map


%.os: %.c %.o
	if $(filter-out -fmudflap,$(COMPILE)) -c -o $@ -fpic -DPIC -DSHARED \
	  -MT $@ -MD -MP -MF "$(DEPDIR)/$*.Tpo" \
	  `test -f '$<' || echo '$(srcdir)/'`$<; \
	then cat "$(DEPDIR)/$*.Tpo" >> "$(DEPDIR)/$*.Po"; \
	     rm -f "$(DEPDIR)/$*.Tpo"; \
	else rm -f "$(DEPDIR)/$*.Tpo"; exit 1; \
	fi

# Special rule to make it possible to define libld_elf_a_SOURCES as we do.
# Otherwise make would complain.
.deps/none_ld.Po: none_ld.os
	-:


installcheck-binPROGRAMS: $(bin_PROGRAMS)
	bad=0; pid=$$$$; list="$(bin_PROGRAMS)"; for p in $$list; do \
	  case ' $(AM_INSTALLCHECK_STD_OPTIONS_EXEMPT) ' in \
	   *" $$p "* | *" $(srcdir)/$$p "*) continue;; \
	  esac; \
	  f=`echo "$$p" | \
	     sed 's,^.*/,,;s/$(EXEEXT)$$//;$(transform);s/$$/$(EXEEXT)/'`; \
	  for opt in --help --version; do \
	    if LD_LIBRARY_PATH=$(DESTDIR)$(libdir) \
	       $(DESTDIR)$(bindir)/$$f $$opt > c$${pid}_.out 2> c$${pid}_.err \
	         && test -n "`cat c$${pid}_.out`" \
	         && test -z "`cat c$${pid}_.err`"; then :; \
	    else echo "$$f does not support $$opt" 1>&2; bad=1; fi; \
	  done; \
	done; rm -f c$${pid}_.???; exit $$bad

CLEANFILES = none_ld.os $(ld_modules:.c=.os)
