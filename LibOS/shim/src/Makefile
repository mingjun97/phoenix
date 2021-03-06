RUNTIME_DIR = $(CURDIR)/../../../Runtime

include ../../../Pal/src/Makefile.Host

CC	= gcc
AS	= gcc
AR	= ar rcs
LD	= ld

OMIT_FRAME_POINTER = no

CFLAGS	= -Wall -fPIC -std=c11 -Winline -Wwrite-strings \
	  -fmerge-all-constants -Wstrict-prototypes \
	  -Werror=implicit-function-declaration \
	  -fno-stack-protector -fno-builtin -Wno-inline \
	  -I../../../bearssl-0.6/inc \
	  -I../include -I../../../Pal/lib -I../../../Pal/include/pal

EXTRAFLAGS = -Wextra -Wno-unused-parameter -Wno-sign-compare

CFLAGS += $(EXTRAFLAGS)

ifeq ($(OMIT_FRAME_POINTER),yes)
CFLAGS += -DOMIT_FRAME_POINTER=1
else
CFLAGS += -fno-omit-frame-pointer -DOMIT_FRAME_POINTER=0
endif
ASFLAGS	= -Wa,--noexecstack -x assembler-with-cpp -I../include

LDFLAGS	= -shared -nostdlib --version-script shim.map -T shim.lds \
	  -z combreloc -z relro -z defs \
	  -dynamic-link=libpal.so \
	  -rpath-link=$(abspath $(RUNTIME_DIR))
LDFLAGS-debug = $(patsubst shim.map,shim-debug.map,$(LDFLAGS))
ARFLAGS	=

ifeq ($(WERROR),1)
CFLAGS += -Werror
endif

files_to_build = libsysdb.a libsysdb.so libsysdb_debug.so
files_to_install = $(addprefix $(RUNTIME_DIR)/,$(files_to_build))

defs	= -DIN_SHIM
fs	= chroot str pipe socket proc dev nextfs smdish smuf smc
ipcns	= pid sysv
objs	= $(addprefix bookkeep/shim_,handle vma thread signal) \
	  $(patsubst %.c,%,$(wildcard utils/*.c)) \
	  $(addprefix fs/shim_,dcache namei fs_hash fs) \
	  $(patsubst %.c,%,$(foreach f,$(fs),$(wildcard fs/$(f)/*.c))) \
	  $(addprefix ipc/shim_,ipc ipc_helper ipc_child) \
	  $(addprefix ipc/shim_ipc_,$(ipcns)) \
	  elf/shim_rtld \
	  $(addprefix shim_,init table syscalls checkpoint malloc \
	  async parser debug) syscallas start \
	  $(patsubst %.c,%,$(wildcard sys/*.c))
bearssl_lib = ../../../bearssl-0.6/build/libbearssl.a
graphene_lib = .lib/graphene-lib.a
pal_lib = $(RUNTIME_DIR)/libpal-$(PAL_HOST).so
headers = ../../../bearssl-0.6/inc/*.h ../include/*.h ../../../Pal/lib/*.h ../../../Pal/include/pal/*.h

all: $(files_to_build) $(files_to_install)

ifeq ($(DEBUG),1)
CC += -gdwarf-2 -g3
CFLAGS += -DDEBUG
ASFLAGS += -DDEBUG
endif
export DEBUG

ifeq ($(PROFILING), 1)
CFLAGS += -DPROFILE
endif

$(files_to_install): $(RUNTIME_DIR)/%: %
	$(call cmd,ln_sf)

ifeq ($(findstring x86_64,$(SYS))$(findstring linux,$(SYS)),x86_64linux)
libsysdb.so: $(addsuffix .o,$(objs)) $(filter %.map %.lds,$(LDFLAGS)) \
	     $(bearssl_lib) $(graphene_lib) $(pal_lib)
	@echo [ $@ ]
	$(LD) $(LDFLAGS) -o $@ $(filter-out %.map %.lds,$^) -soname $@

libsysdb_debug.so: $(addsuffix .o,$(filter-out syscallas,$(objs))) \
		   $(filter %.map %.lds,$($LDFLAGS-debug)) \
		   $(bearssl_lib) $(graphene_lib) $(pal_lib)
	@echo [ $@ ]
	$(LD) $(LDFLAGS-debug) -o $@ $(filter-out %.map %.lds,$^) -soname $@

.lib/host_endian.h: ../../../Pal/src/host/$(PAL_HOST)/host_endian.h
	@mkdir -p .lib
	$(call cmd,ln_sf)

.PHONY: $(graphene_lib)
$(graphene_lib): .lib/host_endian.h
	$(MAKE) -C ../../../Pal/lib target=$(abspath .lib)/

libsysdb.a: $(addsuffix .o,$(objs))
	@echo [ $@ ]
	$(AR) $(ARFLAGS) $@ $^
else
.IGNORE: $(shim_target)
$(shim_target):
endif

%.asm: %.c $(headers)
	@echo [ $@ ]
	@$(CC) $(CFLAGS) $(defs) -c $< -o $<.o
	@objdump -S $<.o > $@
	@rm $<.o

$(addsuffix .o,$(addprefix ipc/shim_ipc_,$(ipcns))): $(wildcard ipc/*.h)
elf/shim_rtld.o: $(wildcard elf/*.h)


%.o: %.c $(headers)
	@echo [ $@ ]
	@$(CC) $(CFLAGS) $(defs) -c $< -o $@

%.e %.i: %.c $(headers)
	@echo [ $@ ]
	@$(CC) $(CFLAGS) $(defs) -E $< -o $@

%.s: %.c $(headers)
	@echo [ $@ ]
	@$(CC) $(CFLAGS) $(defs) -S $< -o $@

%.o: %.S $(headers)
	@echo [ $@ ]
	@$(AS) $(ASFLAGS) $(defs) -c $< -o $@

%.e %.s: %.S $(headers)
	@echo [ $@ ]
	@$(AS) $(ASFLAGS) $(defs) -E $< -o $@

syscallas.S: asm-offsets.h

include ../../../Makefile.rules

clean:
	rm -rf $(addsuffix .o,$(objs)) $(shim_target) $(files_to_build) .lib $(CLEAN_FILES)
