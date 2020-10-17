
# tty0tty - linux null modem emulator with fault injection v2.0


The tty0tty directory tree is divided in:

  **module** - linux kernel module null-modem  
  **pts** - null-modem using ptys (without handshake lines)


## Null modem pts (unix98): 

  When run connect two pseudo-ttys and show the connection names:
  
  (/dev/pts/1) <=> (/dev/pts/2)  

  the connection is:
  
  TX -> RX  
  RX <- TX  



## Module:

 The module is tested with kernel 5.4.0 (Ubuntu 20.04 LTS)

  When loaded, create 8 ttys interconnected:
  
  /dev/tnt0  <=>  /dev/tnt1  
  /dev/tnt2  <=>  /dev/tnt3  
  /dev/tnt4  <=>  /dev/tnt5  
  /dev/tnt6  <=>  /dev/tnt7  

  the connection is:
  
  TX   ->  RX  
  RX   <-  TX  
  RTS  ->  CTS  
  CTS  <-  RTS  
  DSR  <-  DTR  
  CD   <-  DTR  
  DTR  ->  DSR  
  DTR  ->  CD  
  

## Requirements:

  For building the module kernel-headers or kernel source are necessary.

## Installation:

Download the tty0tty package from one of these sources:
Clone the repo https://github.com/mbehr1/tty0tty

```
git clone https://github.com/mbehr1/tty0tty
```

Build the kernel module:

```
cd tty0tty/module
make
```

Copy the new kernel module into the kernel modules directory

```
sudo cp tty0tty.ko /lib/modules/$(uname -r)/kernel/drivers/misc/
```

Load the module

```
sudo depmod
sudo modprobe tty0tty
```

You should see new serial ports in ```/dev/``` (```ls /dev/tnt*```)
Give appropriate permissions to the new serial ports

```
sudo chmod 666 /dev/tnt*
```

You can now access the serial ports as /dev/tnt0 (1,2,3,4 etc) Note that the consecutive ports are interconnected. For example, /dev/tnt0 and /dev/tnt1 are connected as if using a direct cable.

Persisting across boot:

edit the file /etc/modules (Debian) or /etc/modules.conf

```
nano /etc/modules
```
and add the following line:

```
tty0tty
```

Note that this method will not make the module persist over kernel updates so if you ever update your kernel, make sure you build tty0tty again repeat the process.

## Fault injection

The module offers a parameter **ber** that allows to simulate bit-errors on the write->read path.
This is useful to e.g. test SW modules that communicate via UART/serial devices where noise/corruption can occur.

The **ber** value determines the amount of bit-error-rate per 2^32 bytes.

You can set it either as a parameter on loading the module add e.g. `ber=1000` or change it via proc-fs:

to read current **ber** value:

`
sudo cat /sys/module/tty0tty/parameters/ber
`

to change it:

`
sudo bash -c 'echo 1000 > /sys/module/tty0tty/parameters/ber'
`

Now around 1000 / 2^32 bytes will be randomly corrupted.

