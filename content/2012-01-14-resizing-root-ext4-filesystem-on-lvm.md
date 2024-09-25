Title: Resizing Root ext4 filesystem on LVM
Date: 2012-01-14 11:58
Author: Ryan McKay
Tags: linux lvm ext4 lvextend resize2fs ubuntu10.04
Slug: resizing-root-ext4-filesystem-on-lvm
Status: draft

Resizing an ext4 filesystem on a lvm partition is pretty straightforward - you just extend the logical volume first, then resize the filesystem.  It gets a little more complicated if its the root filesystem that you want to resize, because resizing an online filesystem is riskier and probably not even supported for ext4 filesystems (see [resize2fs man page](http://linux.die.net/man/8/resize2fs)).  In this case, you can boot from a live cd to resize the unmounted root partition.  However, most live cds do not include lvm support.  Here is how to resize your root partition with a Ubuntu 10.04 live cd.  

  

First, you can resize the logical volume while you're still running off the root partition.  You can look in /etc/fstab to help figure out the logical volume name.  Look for the filesystem mounted on /.  My entry looked like:  

  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">/dev/mapper/lvmvolume-lucid64root /   ext4    errors=remount-ro 0 1</span>

</p>

<div>

</p>

The corresponding logical volume name is /dev/lvmvolume/lucid64root.  You can use lvdisplay to verify:  

> </p>
>
> <p>
>
> <span style="font-family: &#39;Courier New&#39;, Courier, monospace; font-size: x-small;">
>
> </p>
>
> ubuntu@ubuntu:~\$ sudo lvdisplay  
>
>   --- Logical volume ---  
>
>   LV Name                /dev/lvmvolume/lucid64root  
>
>   VG Name                lvmvolume  
>
>   LV UUID                xbW7iN-x9Ri-gGHG-rwpp-iLu1-gIsf-ycT6dc  
>
>   LV Write Access        read/write  
>
>   LV Status              NOT available  
>
>   LV Size                12.00 GiB  
>
>   Current LE             3072  
>
>   Segments               2  
>
>   Allocation             inherit  
>
>   Read ahead sectors     auto  
>
> </span>

</p>

You can extend the logical volume without unmounting (see [lvm howto](http://tldp.org/HOWTO/LVM-HOWTO/extendlv.html)):

</div>

</p>

<div>

</p>

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">ubuntu@ubuntu:~\$ sudo lvextend -L12G /dev/lvmvolume/lucid64root</span>

</p>

</div>

</p>

  

You'll need to use the same logical volume name later when you resize the filesystem.  Now boot into the live cd.  Since the Ubuntu 10.04 live cd I used doesn't have lvm support, the first step is to intall it:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">ubuntu@ubuntu:~\$ sudo apt-get install lvm2</span>

</p>

Then make your logical volumes available:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">ubuntu@ubuntu:~\$ sudo vgchange -a y</span>

</p>

Then resize the filesystem.  The default size is to just fill the partition.  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">ubuntu@ubuntu:~\$ sudo resize2fs /dev/lvmvolume/lucid64root</span>

</p>

It might ask you to run e2fck first:  

> </p>
>
> <span style="font-family: 'Courier New', Courier, monospace; font-size: x-small;">ubuntu@ubuntu:~\$ sudo e2fsck -f /dev/lvmvolume/lucid64root</span>

</p>

Done.  Now just reboot to your newly resized root partition.
