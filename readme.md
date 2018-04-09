# Udocker

## Installation

Installation is quite simple. Since by default bunch of files are put in ```~/.udocker```, probably the best option is to leave the installation to the user.

To install, follow https://github.com/indigo-dc/udocker/blob/master/doc/installation_manual.md, though the simplest worked:
```
  curl https://raw.githubusercontent.com/indigo-dc/udocker/master/udocker.py > udocker
  chmod u+rx ./udocker
  ./udocker install
```

```~/.udocker``` contains bin and lib directories with dependencies, such as proot and runc, which provide the chroot capability. It also hosts the docker style local container repository.

User interfaces everything with a single file command, ```udocker`, wherever it was installed in the user space - therefore adding a path to PATH or creating personal module file would be recommended.

## Implementation

udocker is written in Python and runs on CentOS7 stock Python. 

udocker pulls and expands the Docker containers into a directory structure and then chroots to them. By default, it does chroot with [proot](https://proot-me.github.io/), which does not introduce any new SetUID or other escallation. Other options are Fakeroot, runC and Singularity, but, I would think that for our purposes the default should be sufficient. The chroot method can be changed with ```--execmode```. This can be also made into a new default, e.g. to change to runC:
```
$ ./udocker setup  --execmode=R1  myfed
```

## Running

Pulling containers from Docker hub is very easy:
```
$ ./udocker run ubuntu
...
 executing: bash
root@tallboy:/# 

$ ./udocker run --user=u0101881 --bindhome ubuntu
 executing: bash
u0101881@tallboy:/$ 
```
-> to bind a home directory in /uufs tree,  use ```--bindhome```
```
$ ./udocker run --user=u0101881 --bindhome --workdir=/uufs/chpc.utah.edu/common/home/u0101881 ubuntu
 executing: bash
u0101881@kingspeak1:/uufs/chpc.utah.edu/common/home/u0101881$ 
```
-> udocker by default starts with /, to start in ones home (or elsewhere), use ```--workdir```

To map /scratch, we need to add bind mount that as well:
```
./udocker run --user=u0101881 --bindhome --workdir=/uufs/chpc.utah.edu/common/home/u0101881 --volume=/scratch:/scratch ubuntu
```

To query available containers on dockerhub:
```
$./udocker search tensorflow
...
```

Users can also create containers based on Docker repos:
```
$./udocker create --name=mytf tensorflow/tensorflow
b3fdea2a-5b33-3283-aa85-6d0d777c1be0
```

and then run them (which will be faster than running from Dockerhub, even with the locally cached container image):
```
./udocker run --user=u0101881 --bindhome mytf python
...
>>> import tensorflow as tf
...
>>> sess=tf.Session()
2018-04-09 23:04:44.503864: I tensorflow/core/platform/cpu_feature_guard.cc:140] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 AVX512F FMA
```

The udocker documentation maintains that user can be root inside the container ```--user=root``` option and modify files as root in the container (e.g. install new packages), but, this runs into permission problems with Ubuntus apt-get. On the other hand, this has worked with Fedora, e.g.:
```
$ ./udocker run fedora:latest
a92387cd# which vim
bash: which: command not found
a92387cd# yum install vim
...
Complete!
a92387cd# which vim
/usr/bin/vim
```

But this change is not persistent...

This can be circumvented by creating own container:
```
$ ./udocker create --name=myfed fedora:latest
$ ./udocker run --user=root myfed
a92387cd# yum install -y vim
a92387cd# exit
$ ./udocker run --user=u0101881 --bindhome myfed
adba7d51$ which vim
/usr/bin/vim
```






