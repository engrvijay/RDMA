### RDMA setting up on linux 
### I am using Ubuntu24 linux kernel=6.14 
### SoftiWARP is a software RDMA device that attaches with the active network cards to enable RDMA programming. 
### Soft-iWARP is a software-based iWARP stack that runs at reasonable performance levels and seamlessly fits into the OFA RDMA environment provides several benefits. 

1. Setting up RDMA environment (lib, verbs , driver)<br>
$sudo apt-get install build-essential libelf-dev cmake<br>

$sudo apt-get install libibverbs1 libibverbs-dev librdmacm1 \
librdmacm-dev rdmacm-utils ibverbs-utils<br>

2. Install common RDMA kernel modules<br>
$sudo modprobe ib_core<br>
$sudo modprobe rdma_ucm<br>

Check the installtion, with following command you can see o/p like below<br>
$sudo lsmod | grep rdma<br> 
rdma_ucm                28672   0<br>
ib_uverbs              126976   1 rdma_ucm<br>
rdma_cm                 61440   1 rdma_ucm<br>
iw_cm                   49152   1 rdma_cm<br>
ib_cm                   57344   1 rdma_cm<br>
ib_core                311296   5 rdma_cm,iw_cm,rdma_ucm,ib_uverbs,ib_cm<br>

3. Now set up some library for the userspace libs<br>
$sudo apt-get install build-essential cmake gcc libudev-dev libnl-3-dev \
libnl-route-3-dev ninja-build pkg-config valgrind<br>

4. Install SoftiWARP<br>
$sudo modprobe siw

Check the module will show o/p like following.<br>
$sudo lsmod | grep siw<br>
siw                   188416  0<br>
ib_core               311296  6 rdma_cm,iw_cm,rdma_ucm,ib_uverbs,siw,ib_cm<br>
libcrc32c<br> 
 
--list the Infiniband device present  
$ls /dev/infiniband 
rdma_cm

If it doesn't exist you need to create  /etc/udev/rules.d/90-ib.rules directory containing the below entries : 

 ####  /etc/udev/rules.d/90-ib.rules  #### 
 KERNEL=="umad*", NAME="infiniband/%k" <br>
 KERNEL=="issm*", NAME="infiniband/%k" <br>
 KERNEL=="ucm*", NAME="infiniband/%k", MODE="0666" <br>
 KERNEL=="uverbs*", NAME="infiniband/%k", MODE="0666" <br>
 KERNEL=="uat", NAME="infiniband/%k", MODE="0666"<br>
 KERNEL=="ucma", NAME="infiniband/%k", MODE="0666"<br>
 KERNEL=="rdma_cm", NAME="infiniband/%k", MODE="0666"<br>
 ########

 5. Setup the SIW interface
  set up the loopback and/or a standard eth interface as RDMA device: for me the eth interface is:enp0s3<br>
$sudo rdma link add siw0 type siw netdev enp0s3<br>
$sudo rdma link add siw0 type siw netdev lo

using ivc_devices and ibv_devinfo command see the result<br>
$ ibv_devices<br>
$ibv_devinfo<br>

Testing: need to open two terminal in one terminal type<br>
$rping -s -a <serverIP><br>
on another terminal type following:<br>
$rping -c -a <serverIP> -v<br>

the flow of data should continue until you kill one of the terminal process.


