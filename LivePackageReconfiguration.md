techniques to live change package parameters

# Introduction #

You can do the following changes while a package is running:
  * change timeouts values
  * add resources (IP, disks, ...)
  * delete resources (IP, disks, ...)

# Procedures Details #

Let's start saying that on the stand-by node this is simply done by modifying the package script, kill the back-to-work monitor and restart it. Can't be more simple. For the active node follow this procedure:

## Change Timeouts on Active node ##

Create a copy of your package script, edit it with your new timeouts values and then substitute the old package script with the new one. Suppose you have pkg1 script configured:
```
mv packages/pkg1.new packages/pkg1
```

Now make back-to-work re-read the new values sending a SIGUSR1 to it:
```
pkill -SIGUSR1 -f "back-to-work pkg1"
```


## Adding a Resource to a running package ##

Create a copy of your package script, edit it adding new resource and calling the right functions to handle them. Manually start the new resources (like configuring a new ip with ifconfig or mount a new file system).
Test that the new package configuration works (actually you can only test the status function):
```
packages/pkg1.new status ; echo $?
```

If the result is `0` then you can submit the new package configuration:
```
mv packages/pkg1.new packages/pkg1
```

## Deleting a Resource from a running package ##

Create a copy of your package script, edit it deleting resources and comment out no more needed functions that handle them. Test that the new package configuration works (actually you can only test the status function):
```
packages/pkg1.new status ; echo $?
```

If the result is `0` then you can submit the new package configuration:
```
mv packages/pkg1.new packages/pkg1
```

Now manually stop the deleted resources (like unconfiguring an ip with ifconfig or umount a old file system).