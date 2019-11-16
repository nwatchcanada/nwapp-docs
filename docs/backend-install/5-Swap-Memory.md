## Create Swap

Start by running the following:

```
swapon -s
```


Check the system overall space use this following use the command:

```
free -m
```

before allocating swap, check the space on availability of drive use the command

```
df -h
```

The fastest and easiest way to create a swap file is by using fallocate. This command creates a file of a preallocated size instantly. We can create a 4 gigabyte file by typing:

```
sudo dd if=/dev/zero of=/swapfile count=4096 bs=1MiB
```

Developers Note: If you want 8 gigabytes:

```
sudo dd if=/dev/zero of=/swapfile count=8192 bs=1MiB
```

After entering your password to authorize sudo privileges, the swap file will be created almost instantly, and the prompt will be returned to you. We can verify that the correct amount of space was reserved for swap by using ls:

```
ls -lh /swapfile
```

## Enable Swap

adjust the permissions on our swap file so that it isn't readable by anyone besides the root accoun

```
sudo chmod 600 /swapfile
```

This will restrict both read and write permissions to the root account only. We can verify that the swap file has the correct permissions by using ``ls -lh`` again:

```
ls -lh /swapfile
```

Now that our swap file is more secure, we can tell our system to set up the swap space for use by typing:

```
sudo mkswap /swapfile
```

Our swap file is now ready to be used as a swap space. We can begin using it by typing:

```
sudo swapon /swapfile
```

verify that the procedure was successful, we can check whether our system reports swap space now:

```
swapon -s
```

Verify our swap memory has been used.

```
free -m
```

### Make the Swap File Permanent

Open

```
sudo vi /etc/fstab
```


Append file.

```
/swapfile   swap    swap    sw  0   0
```

Congradulations your swap file has been intergrated with the OS.


## Special Thanks
Special thanks to the following articles:

* [Increase Swap Memory CentOS 7](https://www.vembu.com/blog/increase-swap-memory-centos-7/)

* [How to Add Swapp on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-centos-7)
