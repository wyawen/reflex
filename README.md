# ReFlex

ReFlex is a software-based system that provides remote access to Flash with performance nearly identical to local Flash access. ReFlex closely integrates network and storage processing in a dataplane  kernel to achieve low latency and high throughput at low resource requirements. ReFlex uses a novel I/O scheduler to enforce tail latency and throughput service-level objectives (SLOs) for multiple clients sharing a remote Flash device.

ReFlex is an extension of the [IX dataplace operating system](https://github.com/ix-project/ix). ReFlex uses Intel [DPDK](http://dpdk.org) for fast packet processing and Intel [SPDK](http://www.spdk.io) for high performance access to NVMe Flash. There are two implementations of ReFlex currently available: a kernel implementation and a userspace implementation. 

In the kernel implementation (available in the [master](https://github.com/stanford-mast/reflex/tree/master) branch of this repository), the ReFlex kernel runs as a guest OS in Linux and relies on the [Dune](https://github.com/project-dune/dune) kernel for memory management and direct access to hardware (e.g., NIC and NVMe Flash queues) from userspace applications. This is the original implementation of ReFlex, as presented in the [paper](https://web.stanford.edu/group/mast/cgi-bin/drupal/system/files/reflex_asplos17.pdf) published at ASPLOS'17. 

In the userspace implementation (available in the [userspace](https://github.com/stanford-mast/reflex/tree/userspace) branch), network and storage processing is implemented in userspace and ReFlex uses the standard `igb_uio` module to bind a network device to a DPDK-provided network device driver. The userspace version of ReFlex does not require the Dune kernel module to be loaded. This means the userspace version of ReFlex is simpler to deploy.  


## Requirements for userspace version of ReFlex

ReFlex requires a NVMe Flash device and a [network interface card supported by Intel DPDK](http://dpdk.org/doc/nics). We have tested ReFlex with the Intel 82599 10GbE NIC and the following NVMe SSDs: Samsung PM1725 and Intel P3600. We have also tested ReFlex on Amazon Web Services EC2 instances i3.4xlarge (see end of setup instructions below for AWS EC2 instance setup).

ReFlex has been successfully tested on Ubuntu 16.04 LTS with kernel 4.4.0.

**Note:** ReFlex provides an efficient *dataplane* for remote access to Flash. To deploy ReFlex in a datacenter cluster, ReFlex should be combined with a control plane to manage Flash resources across machines and optimize the allocation of Flash IOPS and capacity.  


## Instructions for Building ReFlex as a Shared Library 

User-level ReFlex can be built as a shared library "libixev.so" that uses libix and DPDK to allow fast packet processing.

1. Obtain ReFlex source code and fetch dependencies:

   ```
   git clone https://github.com/wyawen/reflex.git
   cd reflex
   git checkout sharedlib
   ./deps/fetch-deps.sh
   ```

2. Install library dependencies: 

   ```
   sudo apt-get install libconfig-dev libnuma-dev libpciaccess-dev libaio-dev libevent-dev g++-multilib
   ```

3. Build the dependecies:

   ```
   sudo chmod +r /boot/System.map-`uname -r`
   make -sj64 -C deps/dpdk config T=x86_64-native-linuxapp-gcc 
   make -sj64 -C deps/dpdk
   make -sj64 -C deps/dpdk install T=x86_64-native-linuxapp-gcc DESTDIR=. CONFIG_RTE_BUILD_SHARED_LIB=y
   
   export REFLEX_HOME=`pwd`	
   ```

4. Specify the absolute path to ix.conf in dp/core/cfg.c 
   ```
   #define DEFAULT_CONF_FILE "/absolute/path/to/reflex/ix.conf"
   ```

5. Build ReFlex as a shared library:

   ```
   make -sj64
   ```

6. Run the script to add the built library to /usr/local/lib
   ```
   sudo ./updatelib.sh 
   ```



