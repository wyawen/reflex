#shared lib
REFLEX   = ..
DPDK = $(REFLEX)/deps/dpdk
SPDK = $(REFLEX)/deps/spdk
DPDK_INC = -I$(DPDK)/x86_64-native-linuxapp-gcc/include -I$(DPDK)/lib/librte_eal/common
SPDK_INC = -idirafter$(SPDK)/include
INC     = -I$(REFLEX)/inc -I$(REFLEX)/inc/lwip -I$(REFLEX)/inc/lwip/ipv4 -I$(REFLEX)/inc/lwip/ipv6 $(DPDK_INC) $(SPDK_INC) -I$(REFLEX)/libix -I$(REFLEX)/dp/core


DPDK_MACHINE_FLAGS = -march=native -DRTE_MACHINE_CPUFLAG_SSE -DRTE_MACHINE_CPUFLAG_SSE2 -DRTE_MACHINE_CPUFLAG_SSE3 -DRTE_MACHINE_CPUFLAG_SSSE3 -DRTE_MACHINE_CPUFLAG_SSE4_1 -DRTE_MACHINE_CPUFLAG_SSE4_2 -DRTE_MACHINE_CPUFLAG_AES -DRTE_MACHINE_CPUFLAG_PCLMULQDQ -DRTE_MACHINE_CPUFLAG_AVX -DRTE_MACHINE_CPUFLAG_RDRAND -DRTE_MACHINE_CPUFLAG_FSGSBASE -DRTE_MACHINE_CPUFLAG_F16C -DRTE_MACHINE_CPUFLAG_AVX2
CFLAGS  = -msse4.1 -fPIC -g -w -fno-dwarf2-cfi-asm -fno-asynchronous-unwind-tables -O3 -mno-red-zone $(INC) $(EXTRA_CFLAGS) -DSPDK_STRING_H $(DPDK_MACHINE_FLAGS) -fPIC

PCIDMA  = $(REFLEX)/deps/pcidma
INC     += -I$(PCIDMA)

LDFLAGS = -T $(REFLEX)/dp/ix.ld -Wl,-rpath=$(DPDK)/x86_64-native-linuxapp-gcc/lib
LDLIBS  = -lrt -lpthread -lm -lnuma -ldl -lconfig -lpciaccess


DPDK_LIBS=
DPDK_LIB_LIST = rte_pmd_ixgbe rte_pmd_ena rte_ethdev rte_mbuf rte_hash rte_eal rte_mempool rte_ring rte_mempool_ring
DPDK_LIB_EXT = .so
DPDK_LIB = $(DPDK_LIB_LIST:%=$(DPDK)/x86_64-native-linuxapp-gcc/lib/lib%$(DPDK_LIB_EXT))

DPDK_LIBS+= -Wl,--no-as-needed -Wl,--start-group -Wl,--whole-archive $(DPDK_LIB) -Wl,--end-group -Wl,--no-whole-archive


SPDK_LIBS=$(SPDK)/build/lib/libspdk_nvme.a
SPDK_LIBS+=$(SPDK)/build/lib/libspdk_util.a
SPDK_LIBS+=$(SPDK)/build/lib/libspdk_env_dpdk.a
SPDK_LIBS+=$(SPDK)/build/lib/libspdk_log.a


IX_LIBS+=$(REFLEX)/libix/libix.so
IX_LIBS+=$(REFLEX)/dp/libixev.so


all: ixclient 

ixclient: reflex_ix_client.c 
	gcc $(CFLAGS) $(LDFLAGS) -o ixclient reflex_ix_client.c $(SPDK_LIBS) $(DPDK_LIBS) $(LDLIBS) $(IX_LIBS) 


clean:
	rm -r ixclient 
