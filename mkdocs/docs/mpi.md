## Parrallel Programming
Each Raspberry Pi is a small unit of compute, one of my goals is to understand how operating many in a cluster. There are various approaches to parallel computational models, message-passing has proven to be an effective one. MPI the Message Passing Interface, is a standardized and portable message-passing system designed to function on a wide variety of parallel computers. 

My prefered language is Python and usefully there is [MPI for Python](https://mpi4py.readthedocs.io/en/stable/install.html).

```
sudo apt-get install -y libopenmpi-dev python-dev pip
sudo pip install mpi4py
mpirun --version
```

All the RPIs in the cluster will access each other via SSH, this communication needs to be passwordless. The first thing you need to do is generate an SSH key pair on first host. 

```
ssh-keygen -t rsa -b 4096
```

Once key generated to enable passwordless access, upload a copy of the public key to the other three servers.

```
ssh-copy-id ubuntu@[server_ip_address]
```

Repeat keygen and copy public key process on all four hosts.

In order for mpi to distribute workload the node where execution occurs needs to understand which nodes are available.  The machinename parameter can be used to point to a text file containing list of nodes.

In order name we can use names we can add entries to hosts file.

```
sudo sh -c 'echo "192.168.1.100 rpi-0" >> /etc/hosts'
sudo sh -c 'echo "192.168.1.101 rpi-1" >> /etc/hosts'
sudo sh -c 'echo "192.168.1.102 rpi-2" >> /etc/hosts'
sudo sh -c 'echo "192.168.1.103 rpi-3" >> /etc/hosts'
```

We can then create a file in home directory listing nodes.

```
cat <<EOF >> ~/machinefile
rpi-0
rpi-1
rpi-2
rpi-3
EOF
```

The easiest way to run commands or code is with the mpirun command. This command, run in a shell, will launch multiple copies of your code, and set up communications between them. As each Pi has four cores and we have four we can specify number of processors to run as sixteen.

```
mpirun -np 16 -machinefile ~/machinefile vcgencmd measure_temp
```

![MPIRUN Measure Temparature](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/mpirun_temp.png)

Scatter is a way that we can take a bunch of elements, like those in a list, and "scatter" those elements around to the processing nodes.

```
cat <<EOF >> ~/scatter.py
from mpi4py import MPI

comm = MPI.COMM_WORLD
size = comm.Get_size()
rank = comm.Get_rank()

if rank == 0:
   data = [(x+1)**x for x in range(size)]
   print ('we will be scattering:',data)
else:
   data = None
   
data = comm.scatter(data, root=0)
print ('rank',rank,'has data:',data)
EOF
```

Once script created execute:

```
mpirun -np 16 -machinefile ~/machinefile python3 ~/scatter.py
```

![MPIRUN Scatter](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/mpirun_scatter.png)

I thought it would be nice for my Pi cluster to calculate Pi with MPI and found the following script.

```
wget -O ~/pi.py https://raw.githubusercontent.com/darrylcauldwell/piClusterRack/main/pi.py
mpirun -np 16 -machinefile ~/machinefile python3 pi.py
```

![Pi with MPI on Pi](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/pi_with_mpi_on_pi.png)
