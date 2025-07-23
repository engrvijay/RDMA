# RDMA setting up on linux with 
# I am using Ubuntu24 linux kernel=6.14 
# SoftiWARP is a software RDMA device that attaches with the active network cards to enable RDMA programming. 
# Soft-iWARP is a software-based iWARP stack that runs at reasonable performance levels and seamlessly fits 
# into the OFA RDMA environment provides several benefits. 

1. Setting up RDMA environment (lib, verbs , driver)
$sudo apt-get install build-essential libelf-dev cmake

$sudo apt-get install libibverbs1 libibverbs-dev librdmacm1 \
librdmacm-dev rdmacm-utils ibverbs-utils

2. Install common RDMA kernel modules
$sudo modprobe ib_core
$sudo modprobe rdma_ucm

Check the installtion, with following command you can see o/p like below
$sudo lsmod | grep rdma 
rdma_ucm               28672  0
ib_uverbs             126976  1 rdma_ucm
rdma_cm                61440  1 rdma_ucm
iw_cm                  49152  1 rdma_cm
ib_cm                  57344  1 rdma_cm
ib_core               311296  5 rdma_cm,iw_cm,rdma_ucm,ib_uverbs,ib_cm

3. Now set up some library for the userspace libs
sudo apt-get install build-essential cmake gcc libudev-dev libnl-3-dev \
libnl-route-3-dev ninja-build pkg-config valgrind

4. Install SoftiWARP
$sudo modprobe siw

Check the module will show o/p like following.
$sudo lsmod | grep siw
siw                   188416  0
ib_core               311296  6 rdma_cm,iw_cm,rdma_ucm,ib_uverbs,siw,ib_cm
libcrc32c 
 
--list the Infiniband device present  
$ls /dev/infiniband 
rdma_cm

If it doesn't exist you need to create  /etc/udev/rules.d/90-ib.rules directory containing the below entries : 

 ####  /etc/udev/rules.d/90-ib.rules  ####
 KERNEL=="umad*", NAME="infiniband/%k"
 KERNEL=="issm*", NAME="infiniband/%k"
 KERNEL=="ucm*", NAME="infiniband/%k", MODE="0666"
 KERNEL=="uverbs*", NAME="infiniband/%k", MODE="0666"
 KERNEL=="uat", NAME="infiniband/%k", MODE="0666"
 KERNEL=="ucma", NAME="infiniband/%k", MODE="0666"
 KERNEL=="rdma_cm", NAME="infiniband/%k", MODE="0666"
 ########

 5. Setup the SIW interface
  set up the loopback and/or a standard eth interface as RDMA device: for me the eth interface is:enp0s3
$sudo rdma link add siw0 type siw netdev enp0s3
$sudo rdma link add siw0 type siw netdev lo

using ivc_devices and ibv_devinfo command see the result
$ ibv_devices
$ibv_devinfo

Testing: need to open two terminal in one terminal type
$rping -s -a <serverIP>
on another terminal type following:
$rping -c -a <serverIP> -v

the flow of data should continue until you kill one of the terminal process.


