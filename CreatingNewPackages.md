how to create a new package and run it

# Introduction #

After configuring back-to-work to run the dummy package you will want to create you own package. Let's start saying that back-to-work inherit the term _package_ from HP ServiceGuard. A package is a collection of resource handled though a script. All those resource move from one node to another by stopping and starting the package script. The package script can be written in any language you like and even be a compiled program but if you choose to write it in POSIX shell you will find some useful modules in the modules directory (packages/modules).

# Creating you package script #

Move to the back-to-work top directory. From now on we will assume you will use this path as your working directory.
Copy a template like linux-test into your package script file
```
cp packages/linux-test packages/mypkg1
```

# Anatomy of a package script #

Edit your new package script and you will see it's divided in 2 section:
  * configuration section
  * code section

the code section have 3 important part you will have to edit and those are:
  * start case
  * stop case
  * status case

these 3 part are actually the cases of a _case_ statement:
```
case $1 in
 echo)
   eval echo \$$2
 ;;

 start)
 ...snip...
 ;;

 stop)
 ...snip...
 ;;

 status)
 ...snip...
 ;;
esac
```

What you need to do is to edit the configuration section with the parameters used by the modules that you will configure in each of the 3 cases.
The configuration section contain even other parameters that are not specific of a single module but are generic for the whole package. Those parameters are:
  * pace
  * timeout
  * start\_timeout
  * stop\_timeout

_pace_ is the number of second used by back-to-work for it's main loop, it represent the frequency at with back-to-work have to check for the status of the package. On fast server with a low number of monitored resource 4 seconds should be fine. If you see that the overhead of back-to-work monitoring start to be too much raise it a little up to the range of 10 seconds.

_timeout_ is the number of seconds after which if the package have trouble the active node will stop it.

_start\_timeout_ is the number of seconds after which if the package is still starting it will be considered to have failed the start procedure and will be stopped

_stop\_timeout_ is the number of seconds which the active node wait before giving up while stopping a package. If the active node give up it will exist instead of returning into the stand-by status.

the sum of _timeout_+_stop\_timeout_ is the numbers of seconds a stand-by node will wait before starting a race to become active and start the package on his own node. The active node while keeping the lock (with dex-lock) will keep at bay the stand-by node having this countdown restarting.

# A simple package example #

If you want to use the IP modules you will have to edit the configuration section like this:
```
# start of configuration section

pace=4
timeout=30
start_timeout=10
stop_timeout=10

vip_ip_0=172.24.49.200; vip_prefix_0=26; vip_network_0=172.24.49.192

# end of configuration section
```

and than edit the code section to make it use the vip module:
```
case $1 in
 echo)
   eval echo \$$2
 ;;

 start)
   echo "$(date) ======== $0 start ========"
   . modules/vip.$os
 ;;

 stop)
   echo "$(date) ======== $0 stop ========="
   . modules/vip.$os
 ;;

 status)
   . modules/vip.$os
 ;;
esac
```

note: I suggest to not put an echo statement in the _status_ case since this will log a message every _pace_ seconds

# Test it #

Before running the new package with back-to-work you can test that it work by directly running it:
```
packages/pkg1 start
```

wait for it to complete (maybe check that it start in less than _start\_timeout_ seconds). Check that it exit with value 0 and then query it's status:

```
packages/pkg1 status
```

if it's running it's status should be now be 0 (check the exist value of the status function with `echo $?`)

now stop it:
```
packages/pkg1 stop
```

Check the exit value, should be 0 and then query the status again, this time the exit value should be 1.

note: the status function should return a value between:
  * 0 for up and running
  * 1 down and halted
  * 2 for degraded or partially running
  * 3 for any critical error that require immediate stop and manual check.

# Run it #
execute on the preferred node:
```
./service start pkg1
```

this command is a shorthand for:
```
nohup ./back-to-work pkg1 > log/back-to-work.pkg1.log 2>&1 &
```

and then the same command on every stand-by node.

# Modules dependencies #

back-to-work does not implement any mechanism to bind resources between them so you have to take care of the order in which you configure your modules.
Usually the start order is something like:
  * pr (SCSI-3 Persistent Reservation)
  * fs (file system)
  * vip (virtual IP)
  * programs (your programs, like apache or another daemon)

and the stop case is the reverse:
  * programs (your programs, like apache or another daemon)
  * vip (virtual IP)
  * fs (file system)
  * pr (SCSI-3 Persistent Reservation)

Common sense should guide you here. You can't start programs that require a clustered file system before mounting it and so you should never access a shared disk without worrying about fencing (Persistent Reservation) to be sure the old active node is not accessing data in any way (maybe it's stale).
For now the only fencing mechanism provides in the modules is SCSI-3 Persistent Reservation.