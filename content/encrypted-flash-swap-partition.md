Title: Encrypted Flash Swap Partition
Date: 2011-12-21 08:55
Author: Ryan McKay
Tags: linux encryption swap
Slug: encrypted-flash-swap-partition
Status: draft

Flash swap can improve performance on systems with low memory, and its pretty cheap.  My laptop has 4GB of memory, but I run a lot of applications and browser tabs, so I still end up swapping.  Using flash for swap doesn't make much if any difference while you're using a single application, but I do notice a significant speedup when switching to other applications that have been swapped out.  I'm running Ubuntu 10.04 with encrypted home directory, which also encrypts the swap partition, so I want my new flash swap encrypted as well.  I'm using a Verbatim Stay 'n Store 4GB drive ([\$9 at amazon](http://amzn.com/B004BLIQDC)) , which has a very small physical footprint, so I can just leave it in all the time.  

  

The first step is to set up a swap partition on your flash drive.  Try to pick the USB port where the drive is least likely to get dislodged.  Insert the drive, and linux should automatically recognize it and mount it.  You can use  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ mount</span>

</p>

to see what drives are mounted.  The newly added usb drive should be the last one.  For example, mine looked like this:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/sdc1 on /media/VERBATIM type vfat (rw,nosuid,nodev,uhelper=udisks,uid=1000,gid=1000,shortname=mixed,dmask=0077,utf8=1,flush)</span>

</p>

In order to create a swap partition, you need to unmount it first:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ umount /dev/sdc1</span>

</p>

Then create the swap partition:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo mkswap /dev/sdc1</span>

</p>

Then enable it with a high priority (so it gets used ahead of the hard disk swap partition):  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo swapon -p 32767 /dev/sdc1</span>

</p>

You can see that it has been added to the list of available swap partitions (cryptswap1 is the pre-existing hard disk encrypted swap partition):  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ cat /proc/swaps</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">Filename<span class="Apple-tab-span" style="white-space: pre;"> </span>Type<span class="Apple-tab-span" style="white-space: pre;"> </span>Size<span class="Apple-tab-span" style="white-space: pre;"> </span>Used<span class="Apple-tab-span" style="white-space: pre;"> </span>Priority</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap1                  partition<span class="Apple-tab-span" style="white-space: pre;"> </span>1949688<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>-1</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/sdc1                               partition<span class="Apple-tab-span" style="white-space: pre;"> </span>3875832<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>32767</span>

</p>

<div>

</p>

Note, if you did not install with the encrypted home directory option, you might need to install the crypto utilities:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo apt-get install cryptsetup ecryptfs-utils</span>

</p>

Now to encrypt the new swap partition:

</div>

</p>

<div>

</p>

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo ecryptfs-setup-swap  
> WARNING: \[/dev/mapper/cryptswap1\] already appears to be encrypted, skipping.  
> WARNING:  
> An encrypted swap is required to help ensure that encrypted files are not leaked to disk in an unencrypted format.  
> HOWEVER, THE SWAP ENCRYPTION CONFIGURATION PRODUCED BY THIS PROGRAM WILL BREAK HIBERNATE/RESUME ON THIS SYSTEM!  
> NOTE: Your suspend/resume capabilities will not be affected.  
> Do you want to proceed with encrypting your swap? \[y/N\]: y  
> INFO: Setting up swap: \[/dev/sdc1\]  
> \* Stopping remaining crypto disks...                                            
> \* cryptswap1 (busy)...                                                          
> \* cryptswap2 (stopped)...                                               \[ OK \]  
> \* Starting remaining crypto disks...                                            
> \* cryptswap1 (running)...                                                      
> \* cryptswap2 (starting)..  
> \* cryptswap2 (started)...                                               \[ OK \] </span>

</p>

</div>

</p>

<div>

</p>

<span style="font-family: inherit;">Now your /proc/swaps looks different:</span>

</div>

</p>

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">laptop:~\$ cat /proc/swaps</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">Filename<span class="Apple-tab-span" style="white-space: pre;"> </span>Type<span class="Apple-tab-span" style="white-space: pre;"> </span>Size<span class="Apple-tab-span" style="white-space: pre;"> </span>Used<span class="Apple-tab-span" style="white-space: pre;"> </span>Priority</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap1                  partition<span class="Apple-tab-span" style="white-space: pre;"> </span>1949688<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>-1</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap2                  partition<span class="Apple-tab-span" style="white-space: pre;"> </span>3875832<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>-2</span>

</p>

<div>

</p>

<div>

</p>

<span style="font-family: inherit;">Also an entry has been added to fstab:</span>

</div>

</p>

<div>

</p>

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ grep cryptswap /etc/fstab  
> /dev/mapper/cryptswap1 none swap sw 0 0  
> /dev/mapper/cryptswap2 none swap sw 0 0</span>

</p>

</div>

</p>

Notice that it did not preserve the priority setting. You can fix this for your current session by doing:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo swapoff /dev/mapper/cryptswap2  
> </span><span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ sudo swapon -p 32767 /dev/mapper/cryptswap2</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">user@laptop:~\$ cat /proc/swaps</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">Filename<span class="Apple-tab-span" style="white-space: pre;"> </span>Type<span class="Apple-tab-span" style="white-space: pre;"> </span>Size<span class="Apple-tab-span" style="white-space: pre;"> </span>Used<span class="Apple-tab-span" style="white-space: pre;"> </span>Priority</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap1                  partition<span class="Apple-tab-span" style="white-space: pre;"> </span>1949688<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>-1</span>  
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap2                  partition<span class="Apple-tab-span" style="white-space: pre;"> </span>3875832<span class="Apple-tab-span" style="white-space: pre;"> </span>0<span class="Apple-tab-span" style="white-space: pre;"> </span>32767</span>

</p>

<div>

</p>

And fix it for your next reboot forward by changing the entry in /etc/fstab to:  

<div>

</p>

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/cryptswap2 none swap sw,pri=32767 0 0</span>

</p>

</div>

</p>

<div>

</p>

  

</div>

</p>

  

<span style="font-family: inherit;">Sources:</span>  

<http://www.arsgeek.com/2008/07/24/readyboost-for-linux-a-quick-how-to/>  

<http://www.logilab.org/29155>  

<http://www.brighthub.com/computing/linux/articles/37236.aspx>

</div>

</p>

</div>

</p>
