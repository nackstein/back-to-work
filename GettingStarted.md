introduction to back-to-work

# Introduction #

back-to-work is a failover cluster manager written in POSIX shell scripting language. It aims to be able to move resources between a set of servers in case of hardware failure or other configurable events. AFAIK there are no other maintained failover cluster software in public domain and one can think that POSIX shell (or other shell scripting language) is not suited for this kind of task. But on the pros side shell scripting is one of the most well known language by UNIX sysadmin (author included) and this is why it's the chosen language for back-to-work (it's written by a sysadmin for sysadmins :D). With back-to-work you will have a working program with few line of codes and very likely you will be able to read it, understand it and debug it if you find problems or customize it to fit your needs!
back-to-work is very portable and run on Linux, HP-UX, OpenBSD, CYGWIN, ESXi.

# Installation #

Installing back-to-work is simple as downloading the source (tar or zip) and explode it in the directory of choice on every server of your cluster. You will need another program to make back-to-work running: dex-lock. You can download dex-lock here:
https://code.google.com/p/dex-lock/

Usually you will explode back-to-work and dex-lock in the same directory (the author likes the directory `/opt/shell-cluster-suite` :D)

If you choose to install back-to-work and dex-lock in the same path you can avoid to configure dex-lock location otherwise edit the file `conf/lock_path` with the path you choose for dex-lock.

Remember: a good configuration is one with at least 3 lock server. You can make a cluster of 2 failover nodes + 1 lock server with all 3 servers acting as lock server. Less than 3 lock server will expose you to cluster failure in the event of one server failure (thus defeating the cluster role).

# Configuration #

Once you have installed back-to-work you have to configure 2 thing:
  * the shell interpreter
  * the list of cluster nodes

To configure the shell interpreter move to the directory of back-to-work and run the script `utils/set-shell /bin/sh` (change the /bin/sh parameter with the full path to the shell you want to use, it must be a POSIX compliant  shell, dash is a good options since it's very fast, bash is slower but ok, ash is ok, other POSIX shell are ok :D). Then edit the file `conf/cluster_nodes` with the space separeted list host hostname of your cluster member. This list is used only to query the cluster status. If you forget to edit it you will survive anyway.

Remember to configure dex-lock too. Read the wiki on the dex-lock project for this.

# Using it #

For a first test you can move to the back-to-work main directory and run:
```
verbose=1 ./back-to-work dummy
```

execute the same command on every cluster node. You will see that only one will be able to start the package dummy. On the terminale of the active node hit CTRL-C to stop the package, wait some time and see another node becoming the active one!

# Bla bla bla #

wiki incomplete. read the code :D