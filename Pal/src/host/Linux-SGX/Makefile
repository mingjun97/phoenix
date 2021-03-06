include Makefile.am

CFLAGS	+= -I. -Iinclude -I../.. -I../../../include -I../../../lib -Isgx-driver
ASFLAGS += -I. -I../.. -I../../../include

host_files = libpal-Linux-SGX.a pal-sgx debugger/sgx_gdb.so generated_offsets.py

defs	= -DIN_PAL -DPAL_DIR=$(PAL_DIR) -DRUNTIME_DIR=$(RUNTIME_DIR)
enclave-objs = $(addprefix db_,files devices pipes sockets streams memory \
		 threading mutex events process object main rtld \
		 exception misc ipc spinlock) \
	       $(addprefix enclave_,ocalls ecalls framework pages untrusted rpcqueue)
enclave-asm-objs = enclave_entry 
urts-objs = $(addprefix sgx_,enclave framework main rtld thread process exception graphene rpcqueue)
urts-asm-objs = sgx_entry
graphene_lib = .lib/graphene-lib.a
headers	= $(wildcard *.h) $(wildcard ../../*.h) $(wildcard ../../../lib/*.h) \
	  $(addprefix sgx-driver/,graphene-sgx.h isgx_version.h)

.PHONY: all
all: $(host_files)

ifeq ($(DEBUG),1)
CC += -gdwarf-2 -g3
CFLAGS += -DDEBUG
export DEBUG
endif

../../host_endian.h: host_endian.h
	$(MAKE) -C ../../ $<

libpal-Linux-SGX.a: $(addsuffix .o,$(enclave-objs) $(enclave-asm-objs))
	@echo [ host/Linux-SGX/$@ ]
	$(AR) $(ARFLAGS) $@ $^

$(addsuffix .o,$(enclave-objs)): %.o: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(CFLAGS) $(defs) -DIN_ENCLAVE -c $< -o $@

$(addsuffix .e,$(enclave-objs)): %.e: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(CFLAGS) $(defs) -DIN_ENCLAVE -E $< -o $@

$(addsuffix .i,$(enclave-objs)): %.i: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(CFLAGS) $(defs) -DIN_ENCLAVE -E $< -o $@

$(addsuffix .s,$(enclave-objs)): %.s: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(CFLAGS) $(defs) -DIN_ENCLAVE -S $< -o $@

$(addsuffix .o,$(enclave-asm-objs)): %.o: %.S $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(AS) $(ASFLAGS) $(defs) -DIN_ENCLAVE -c $< -o $@

$(addsuffix .s,$(enclave-asm-objs)): %.s: %.S $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(AS) $(ASFLAGS) $(defs) -DIN_ENCLAVE -E $< -o $@

$(addsuffix .o,$(urts-objs)): %.o: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(filter-out -DIN_ENCLAVE,$(CFLAGS)) $(defs) -c $< -o $@

$(addsuffix .e,$(urts-objs)): %.e: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(filter-out -DIN_ENCLAVE,$(CFLAGS)) $(defs) -E $< -o $@

$(addsuffix .i,$(urts-objs)): %.i: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(filter-out -DIN_ENCLAVE,$(CFLAGS)) $(defs) -E $< -o $@

$(addsuffix .s,$(urts-objs)): %.s: %.c $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(filter-out -DIN_ENCLAVE,$(CFLAGS)) $(defs) -S $< -o $@

$(addsuffix .s,$(urts-asm-objs)): %.s: %.S $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(AS) $(filter-out -DIN_ENCLAVE,$(ASFLAGS)) $(defs) -E $< -o $@

$(addsuffix .o,$(urts-asm-objs)): %.o: %.S $(headers)
	@echo [ host/Linux-SGX/$@ ]
	@$(AS) $(filter-out -DIN_ENCLAVE,$(ASFLAGS)) $(defs) -c $< -o $@

pal-sgx: $(addsuffix .o,$(urts-objs) $(urts-asm-objs)) $(graphene_lib)
	@echo [ host/Linux-SGX/$@ ]
	@$(CC) $(CFLAGS) -pie $^ -lc -pthread -o $@

debugger/sgx_gdb.so: debugger/sgx_gdb.c debugger/sgx_gdb.h sgx_arch.h
	@echo [ host/Linux-SGX/$@ ]
	$(CC) -Wall -fPIC -O2 -std=c11 -c debugger/sgx_gdb.c -o debugger/sgx_gdb.o
	$(LD) -shared debugger/sgx_gdb.o -o debugger/sgx_gdb.so -lc

enclave_entry.o sgx_entry.o: asm-offsets.h

sgx-driver/isgx_version.h:
	$(MAKE) -C sgx-driver $(notdir $<)

include ../../../../Makefile.rules

CLEAN_FILES += $(notdir $(pal_static) $(pal_lib) $(pal_sec) $(pal_loader))
CLEAN_FILES += debugger/sgx_gdb.o

.PHONY: clean
clean:
	rm -f *.o *.e *.i *.s $(host_files) $(CLEAN_FILES)
