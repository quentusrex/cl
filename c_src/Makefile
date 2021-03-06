OSNAME := $(shell uname -s)
MACHINE := $(shell uname -m)
CC = gcc

ERLDIR	:= $(shell erl -noshell -eval "io:format([126,115,126,110],[code:root_dir()])" -s erlang halt)
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
EXT       = so

ifeq ($(WORDSIZE), 64)
include config.64.build
endif
ifeq ($(WORDSIZE), 32)
include config.32.build
endif

ifeq ($(TYPE), debug)
OBJDIR=../obj/debug
LIBDIR=../lib/debug
CFLAGS += -Ddebug -DDEBUG -g -Wall -Wextra -Wswitch-default -Wswitch-enum -D_THREAD_SAFE -D_REENTRANT -fno-common 
endif

ifeq ($(TYPE), release)
OBJDIR=../obj/release
LIBDIR=../lib/release
CFLAGS += -Wall -Wextra -Wswitch-default -Wswitch-enum -D_THREAD_SAFE -D_REENTRANT -fno-common
endif

ifeq ($(OSNAME), Linux)
LINUX = Yes
ifeq ($(WORDSIZE), 32)
CFLAGS += -O3 -fPIC -m32 -DWORDSIZE=32
endif
ifeq ($(WORDSIZE), 64)
CFLAGS += -O3 -fPIC -m64 -DWORDSIZE=64
endif
LD_SHARED	:= $(CC) -shared -lOpenCL
endif

ifeq ($(OSNAME), Darwin)
MAC_OS_X = Yes
ifeq ($(WORDSIZE), 32)
CFLAGS += -O3 -fPIC -m32 -DWORDSIZE=32 -DDARWIN -no-cpp-precomp
LD_SHARED	:= $(CC) -m32 -bundle -flat_namespace -undefined suppress
endif
ifeq ($(WORDSIZE), 64)
CFLAGS += -O3 -fPIC -m64 -DWORDSIZE=64 -DDARWIN -no-cpp-precomp
LD_SHARED	:= $(CC) -m64 -bundle -flat_namespace -undefined suppress
endif
endif

ifeq ($(OSNAME), MINGW32_NT-6.0)
MINGW = Yes
endif
ifeq ($(OSNAME), MINGW32_NT-6.1)
MINGW = Yes
endif

ifeq ($(MINGW), Yes)
WIN32 = Yes
EXT   = dll
CFLAGS += -D__WIN32__
ifeq ($(WORDSIZE), 32)
CFLAGS += -shared -I/opt/local/include -m32 -DWIN32 -DWORDSIZE=32
endif
ifeq ($(WORDSIZE), 64)
CFLAGS += -shared -I/opt/local/include -m64 -DWIN32 -DWORDSIZE=64
endif

## Optimizations is broken on mingw 4.4.0 (it crashes with it on)
GCC_VERSION = $(shell gcc -dumpversion)
ifneq ($(GCC_VERSION), 4.4.0)
CFLAGS += -O3
endif

LD_SHARED	:= $(CC) -shared 
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
	LDFLAGS += -Wl--enable-stdcall-fixup -L/opt/local/lib -lOpenCL
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

CL_DRV = $(LIBDIR)/$(WORDSIZE)/cl_drv.$(EXT)

CL_NIF = $(LIBDIR)/$(WORDSIZE)/cl_nif.$(EXT)

CL_DRV_OBJS = \
	$(OBJDIR)/$(WORDSIZE)/cl_drv.o \
	$(OBJDIR)/$(WORDSIZE)/cl_hash.o

CL_NIF_OBJS = \
	$(OBJDIR)/$(WORDSIZE)/cl_nif.o \
	$(OBJDIR)/$(WORDSIZE)/cl_hash.o

CL_ASM = \
	$(OBJDIR)/$(WORDSIZE)/cl_drv.s \
	$(OBJDIR)/$(WORDSIZE)/cl_hash.s

OCL_SIMD_OBJS = $(SIMD_OBJS:%=$(OBJDIR)/%)

all:	all32 all64

nif32:	config.32.h
	$(MAKE) nif TYPE=release WORDSIZE=32

all32:	config.32.build
	$(MAKE) driver TYPE=release WORDSIZE=32
	$(MAKE) driver TYPE=debug   WORDSIZE=32

all64:	config.64.build
	$(MAKE) driver TYPE=release WORDSIZE=64
	$(MAKE) driver TYPE=debug   WORDSIZE=64

config.32.build:	configure
	./configure --with-wordsize=32

config.64.build:	configure
	./configure --with-wordsize=64

configure:	configure.in
	autoheader; autoconf

clean: 
	$(MAKE) clean_internal TYPE=release WORDSIZE=32
	$(MAKE) clean_internal TYPE=debug   WORDSIZE=32
	$(MAKE) clean_internal TYPE=release WORDSIZE=64
	$(MAKE) clean_internal TYPE=debug   WORDSIZE=64

# Make cl library
debug:
	$(MAKE) driver TYPE=debug

clean_debug:
	$(MAKE) clean_internal TYPE=debug

release:
	$(MAKE) driver TYPE=release

ifeq ($(CL_BUILD), false)
driver: 
	@echo 
	@echo Skipping driver for wordsize=$(WORDSIZE), not supported on this system
	@echo 
else
driver: $(CL_DRV)
endif

nif: $(CL_NIF)

cbuf_test: cbuf_test.o
	$(CC) -o $@ cbuf_test.o

cbuf_test.o:  cbufv2.h

depend:

clean_internal:
	-rm -f $(OBJDIR)/$(WORDSIZE)/*.o

$(CL_DRV): $(OCL_LIB) $(CL_DRV_OBJS) # $(CL_ASM)
	$(LD_SHARED) -o $@ $(CL_DRV_OBJS) $(LDFLAGS)

$(CL_NIF): $(OCL_LIB) $(CL_NIF_OBJS) # $(CL_ASM)
	$(LD_SHARED) -o $@ $(CL_NIF_OBJS) $(LDFLAGS)

$(OBJDIR)/$(WORDSIZE)/cl_drv.o: cbufv2.h cl_hash.h

$(OBJDIR)/$(WORDSIZE)/%.o: %.c
	$(CC) -c -o $@ $(CFLAGS) -I$(ERL_C_INCLUDE_DIR) $<

$(OBJDIR)/$(WORDSIZE)/%.s: %.c
	$(CC) -S -o $@ $(CFLAGS) -I$(ERL_C_INCLUDE_DIR) $<

cbuf_test.o: cbuf_test.c
	$(CC) -c -o $@ -g -I$(ERL_C_INCLUDE_DIR) $<

