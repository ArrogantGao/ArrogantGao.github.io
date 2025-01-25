# Install slurm on Ubuntu 22.04

This blog is a technical note for the installation of slurm on Ubuntu 22.04.

## Before this note

First, I would like to claim that there are already lots of good turtorials online. However, when I try to install slurm, I found most of these turtorials are not working very well due to the changes of the system and tools. The purpose of this note is to record my experience and provide a detailed guide for the installation of slurm 23.11.1 on Ubuntu 22.04. (The reason I did not use ubuntu 24.04 is that I found the change is very huge between 22 and 24, harder to follow the tutorial.)

Here are some main tutorials I followed (they are in Chinese):
* Slurm: [https://scc.ustc.edu.cn/hmli/doc/linux/slurm-install/slurm-install.html](https://scc.ustc.edu.cn/hmli/doc/linux/slurm-install/slurm-install.html) (very detailed but has some mistakes)
* MaridDB: [https://cn.linux-console.net/?p=12420](https://cn.linux-console.net/?p=12420)
* NFS: [https://cn.linux-console.net/?p=14769](https://cn.linux-console.net/?p=14769)
* NIS: [https://blog.csdn.net/davei1988/article/details/141639252](https://blog.csdn.net/davei1988/article/details/141639252)

Some other tools:
* Run command at system start: [https://blog.csdn.net/qq_54529791/article/details/135826187](https://blog.csdn.net/qq_54529791/article/details/135826187)
* Other turtorials (recommand to read, but don't use them directly): [https://github.com/mknoxnv/ubuntu-slurm](https://github.com/mknoxnv/ubuntu-slurm), [https://github.com/nateGeorge/slurm_gpu_ubuntu](https://github.com/nateGeorge/slurm_gpu_ubuntu), [https://github.com/SergioMEV/slurm-for-dummies](https://github.com/SergioMEV/slurm-for-dummies)

## The process

1. Install the system (ubuntu 22.04).
2. Install the tools:
    * MaridDB: slurm need to use maridDB to store the job information.
    * NFS: network file system, to share the data between the nodes, this allow me to use the same `/storage`, `/opt` and `/home` directory on all the nodes.
    * NIS: network information service, to manage the user and group information on all the nodes. With NIS, I can create a user on master node, and then use the same user on other nodes.
3. Install munge: munge is used to secure the communication between the nodes.
4. Install and set up slurm

### Install the system

It is easy to install the system, just follow the instruction on the [ubuntu website](https://ubuntu.com/tutorials/install-ubuntu-desktop). 
Also remember to install the `openssh-server` on all the nodes.

If you have a GPU, you can also install the `nvidia-driver-*` on all the nodes (I use `nvidia-driver-550`).

For network, the master node is connected to the internet, and all nodes are connected to the master node (it has two network cards) with a switch to construct a same subnet.

![](/assets/slurm_figs/nodes.jpeg)

**All the following steps should be done by the root user.**

### Install the tools

Prepare the environment on all the nodes, make sure all is set up correctly before installing munge and slurm.

#### MaridDB

Just follow the [MaridDB tutorial](https://cn.linux-console.net/?p=12420).

#### NFS

<!-- Install the NFS on all nodes, use the master node as the server and the other nodes as the client.
```bash
sudo apt install nfs-kernel-server
```
Here I use `/storage` as the shared directory, for other directories, the operation is similar.
```bash
sudo mkdir -p /storage
sudo chown -R nobody:nogroup /storage
sudo chmod 777 /storage
```
Then change `/etc/exports` to add the shared directory.
```bash
echo "/storage 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
```
here `192.168.1.0/24` is the subnet of all the nodes, change it to your own subnet.

Then restart the NFS service.
```bash
sudo exportfs -a
sudo systemctl restart nfs-server
```

For the client nodes, you need to mount the shared directory.
```bash
sudo apt install nfs-common
sudo mount -t nfs 192.168.1.1:/storage /storage
```
here `192.168.1.1` is the IP address of the master node. -->

Follow this [guide](https://cn.linux-console.net/?p=14769) to install the NFS.
Remember to change the subnet to your own subnet. I directly closed the `ufw` service on all the nodes (not recommand).
I shared `/storage`, `/opt` and `/home` directory on all the nodes.

You can also start the service automatically when the node is booted by make this as a systemd service (in my case, I make it sleep for 10 seconds after the node is booted to make sure the network is ready).

#### NIS

Install the NIS according to the [NIS tutorial](https://blog.csdn.net/davei1988/article/details/141639252).
One can also use OpenLDAP to replace NIS, but I think NIS is easier to use.

Then run the following command on the master node to create a user.
```bash
adduser slurm
adduser munge
```
and update the NIS database.
```bash
cd /etc/yp
make
```
remember each time you change the user, you need to update the NIS database.

### Install munge

Follows 4.4 of this [guide](https://scc.ustc.edu.cn/hmli/doc/linux/slurm-install/slurm-install.html), just remember that create the `munge` user and group before installing munge!

### Install slurm

Follows chapter 3 and 4 of this [guide](https://scc.ustc.edu.cn/hmli/doc/linux/slurm-install/slurm-install.html), also you need to create the `slurm` user and group before doing so.

A mistake in the tutorial is that in the `cgroup.conf` file should be modified as:
```
# CgroupAutomount=yes # this is not needed
CgroupMountpoint=/sys/fs/cgroup
# CgroupPlugin=

ConstrainCores=yes
ConstrainDevices=yes
ConstrainRAMSpace=yes

# MaxRAMPercent=98 
# AllowedRAMSpace=96
```
others should be the same as the tutorial.

## Results

After the installation, you can enjoy the slurm if you successfully run the following commands.
```bash
# xzgao @ master in ~ [1:02:33]
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
i96c256g*    up   infinite      3   idle node[1-3]

# xzgao @ master in ~ [1:02:49]
$ srun -N3 hostname
node1
node2
node3
```

Here is an example of the slurm script, where I run a `julia` script on 1 node with 96 cores, 192 threads.
```bash
# xzgao @ master in ~/work/julia_test [1:07:31]
$ cat blas.jl
using BLASBenchmarksCPU, CSV, DataFrames

libs = [:Gaius, :Octavian, :OpenBLAS]
threaded = true
benchmark_result = runbench(Float64; libs, threaded)

df = benchmark_result_df(benchmark_result)
CSV.write("/home/xzgao/work/julia_test/blas_results.csv", df)

# xzgao @ master in ~/work/julia_test [1:05:11]
$ cat run.slurm
#!/bin/bash

#SBATCH --job-name=test
#SBATCH --partition=i96c256g
#SBATCH -n 96
#SBATCH --ntasks-per-node=192
#SBATCH --output=%j.out
#SBATCH --error=%j.err

julia --project=/home/xzgao/work/julia_test -t 192 /home/xzgao/work/julia_test/blas.jl

# xzgao @ master in ~/work/julia_test [1:04:17]
$ sbatch run.slurm
Submitted batch job 26

# xzgao @ master in ~/work/julia_test [1:04:31]
$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                26  i96c256g     test    xzgao  R       0:01      1 node1

# xzgao @ master in ~/work/julia_test [1:04:32]
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
i96c256g*    up   infinite      1  alloc node1
i96c256g*    up   infinite      2   idle node[2-3]
```

Since I use NFS to share `/home` directory, the program can be run on any nodes once the user install software under the `/home` directory.