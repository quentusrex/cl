OSNAME := $(shell uname -s)
MACHINE := $(shell uname -m)
CC = gcc

ERLDIR	:= /usr/local/lib/erlang
ERL_C_INCLUDE_DIR := $(ERLDIR)/usr/include
OCL_DIR := $(shell cd ..; pwd)

MAC_OS_X  = No
WIN32     = No
LINUX     = No

ALTIVEC   = No
MMX       = No
SSE       = No
SSE2      = No
SSE3      = No
SSE4      = No

ifeq ($(TYPE), debug)
OBJDIR=../obj/debug
LIBDIR=../lib/debug
CFLAGS += -fPIC -Ddebug -DDEBUG -g -Wall -Wextra -Wswitch-default -Wswitch-enum -no-cpp-precomp -D_THREAD_SAFE -D_REENTRANT -fno-common 
endif

ifeq ($(TYPE), release)
OBJDIR=../obj/release
LIBDIR=../lib/release
CFLAGS += -fPIC -O3 -Wall -Wextra -Wswitch-default -Wswitch-enum -no-cpp-precomp -D_THREAD_SAFE -D_REENTRANT -fno-common
endif

ifeq ($(OSNAME), Linux)
LINUX = Yes
ifeq ($(WORDSIZE), 32)
CFLAGS += -m32 -DWORDSIZE=32
endif
ifeq ($(WORDSIZE), 64)
CFLAGS += -m64 -DWORDSIZE=64
endif
LD_SHARED	:= $(CC) -shared
endif

ifeq ($(OSNAME), Darwin)
MAC_OS_X = Yes
ifeq ($(WORDSIZE), 32)
CFLAGS += -m32 -DWORDSIZE=32
LD_SHARED	:= $(CC) -m32 -bundle -flat_namespace -undefined suppress
endif
ifeq ($(WORDSIZE), 64)
CFLAGS += -m64 -DWORDSIZE=64
LD_SHARED	:= $(CC) -m64 -bundle -flat_namespace -undefined suppress
endif
endif

ifeq ($(MACHINE), "Power Macintosh")
ALTIVEC   = Yes
endif

ifeq ($(MACHINE), i386)
MMX       = No
SSE       = No
SSE2      = Yes
SSE3      = No
endif

ifeq ($(MACHINE), i686)
MMX       = No
SSE       = No
SSE2      = Yes
SSE3      = No
endif

ifeq ($(MAC_OS_X),Yes)
	LDFLAGS += -framework OpenCL
endif

ifeq ($(WIN32),Yes)
	CFLAGS += -DWIN32
endif

ifeq ($(ALTIVEC),Yes)
	CFLAGS += -maltivec -DUSE_ALTIVEC
endif

ifeq ($(SSE),Yes)
	CFLAGS += -msse
endif

ifeq ($(SSE2),Yes)
	CFLAGS += -msse2 -DUSE_SSE2
	SIMD_OBJS += ocl_simd_sse2.$(BIT).o
endif

ifeq ($(SSE3),Yes)
	MCFLAGS += -msse3
endif

ifeq ($(MMX),Yes)
	MCFLAGS += -mmmx -DUSE_MMX
	SIMD_OBJS += ocl_simd_mmx.$(BIT).o
endif

CL_DRV = $(LIBDIR)/$(WORDSIZE)/cl_drv.so

CL_OBJS = \
	$(OBJDIR)/$(WORDSIZE)/cl_drv.o \
	$(OBJDIR)/$(WORDSIZE)/cl_hash.o

CL_ASM = \
	$(OBJDIR)/$(WORDSIZE)/cl_drv.s \
	$(OBJDIR)/$(WORDSIZE)/cl_hash.s

OCL_SIMD_OBJS = $(SIMD_OBJS:%=$(OBJDIR)/%)

all:	all32 all64

all32:	config.32.h
	$(MAKE) driver TYPE=release WORDSIZE=32
	$(MAKE) driver TYPE=debug   WORDSIZE=32

all64:	config.64.h
	$(MAKE) driver TYPE=release WORDSIZE=64
	$(MAKE) driver TYPE=debug   WORDSIZE=64

config.32.h:	configure
	./configure --with-wordsize=32

config.64.h:	configure
	./configure --with-wordsize=64

configure:	configure.in
	autoheader; autoconf

clean: 
	$(MAKE) clean_internal TYPE=release

# Make cl library
debug:
	$(MAKE) driver TYPE=debug

clean_debug:
	$(MAKE) clean_internal TYPE=debug

release:
	$(MAKE) driver TYPE=release

driver: $(CL_DRV)

cbuf_test: cbuf_test.o
	$(CC) -o $@ cbuf_test.o

cbuf_test.o:  cbufv2.h

depend:

clean_internal:
	-rm -f $(OBJDIR)/*.o

$(CL_DRV): $(OCL_LIB) $(CL_OBJS) # $(CL_ASM)
	$(LD_SHARED) -o $@ $(CL_OBJS) $(LDFLAGS)

$(OBJDIR)/$(WORDSIZE)/cl_drv.o: cbufv2.h cl_hash.h

$(OBJDIR)/$(WORDSIZE)/%.o: %.c
	$(CC) -c -o $@ $(CFLAGS) -I$(ERL_C_INCLUDE_DIR) $<

 $(OBJDIR)/$(WORDSIZE)/%.s: %.c
	$(CC) -S -o $@ $(CFLAGS) -I$(ERL_C_INCLUDE_DIR) $<

cbuf_test.o: cbuf_test.c
	$(CC) -c -o $@ -g -I$(ERL_C_INCLUDE_DIR) $<
