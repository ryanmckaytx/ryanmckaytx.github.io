Title: Inexpensive Remote Backup with Synology DS110j
Date: 2011-11-27 09:48
Author: Ryan McKay
Tags: ssh, backup, ds110j, linux, rdiff-backup, synology
Slug: inexpensive-remote-backup-with-synology
Status: draft

<span class="Apple-style-span"></span>

> 

I wanted an offsite backup of my important data (~200GB), with more control and lower recurring cost than a service like dropbox. The [Synology DS110j](http://www.synology.com/us/products/DS110j/index.php) is an inexpensive, but full-featured, network-attached storage (NAS) device. It runs a Linux kernel with [BusyBox](http://busybox.net/) utilities. I used it to set up a low-power remote rdiff-backup target. I like [rdiff-backup](http://www.nongnu.org/rdiff-backup/) because it provides the best features of a mirror and an incremental backup.</span>

<div>

<span class="Apple-style-span">**  
**</span>

</div>

<div>

<span class="Apple-style-span">**Hardware**</span>

</div>

<div>

<span class="Apple-style-span">Synology DiskStation DS110j 1-Bay NAS - \$149.99 at Amazon.com</span>

</div>

<div>

<span class="Apple-style-span">Seagate Barracuda Green ST1500DL003 1.5TB 5900 RPM internal hd - \$129.99 at Newegg.com</span>

</div>

<div>

<span class="Apple-style-span">**  
**</span>

</div>

<div>

<span class="Apple-style-span">**Power Profile**</span>

</div>

<div>

<span class="Apple-style-span">I measured the power usage in various modes with a [KillAWatt](http://www.p3international.com/products/special/p4400/p4400-ce.html).  
</span>

|  |  |
|----|----|
| Mode | Power (Watts) |
| Standby (entered automatically after ~10 minutes of no activity) | 5.1 |
| Active w/o disk activity | 10.1 |
| During backup | 10.9 |

</span>

</div>

<div>

  

</div>

<div>

<span class="Apple-style-span">**Initial Setup**</span>

</div>

<div>

<span class="Apple-style-span">Follow included instructions to install hd and DiskStation Manager software.</span>

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">**Enable SSH**</span>

</div>

<div>

<span class="Apple-style-span">Use the web interface to [enable ssh](http://forum.synology.com/wiki/index.php/Enabling_the_Command_Line_Interface). I like to change the ssh port from the default. I've found that in practice, this cuts down almost all automated ssh-based hack attempts. To change this setting, ssh to the box as root (same pw as admin in the web interface), then edit DS:/etc/ssh/sshd_config to add the setting:</span>

</div>

<div>

    Port ####

<span class="Apple-style-span"> (whatever port you want to use). Also add to SOURCE:/root/.ssh/config</span>  

    Host DS # whatever the DS hostname is
        Port ####

</div>

<div>

After any change to sshd settings, you'll need to restart the ssh server. You can do this through the web interface by disabling and then re-enabling ssh.

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">**Enable Automated Login**</span>

</div>

<div>

<span class="Apple-style-span">My backup runs as a cron job in the middle of the night, so it needs to be able to ssh login to the DS without prompting for a password. Public key authentication is the way to do this. </span>

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">In my setup, I am backing up one source to multiple destinations (one local rdiff-backup and the DS for off-site rdiff-backup). Therefore, I prefer to put the rdiff-backup config on the source and push rather than pull my backups. Follow [these instructions](http://linuxproblem.org/art_9.html), where A is the SOURCE side (initiating the rdiff-backup session), and B is the DS side. I used the root account on both sides. </span>

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">Suppose there are multiple remote hosts for which you want automated logins. You can reuse the same key pair, or set up multiple pairs. Assuming the latter case, rename the private key to something like id_rsa_synology (should also rename public key similarly so you remember which keys are part of the same pair). Also add to SOURCE:/root/.ssh/config (under the DS host section):</span>  

    IdentityFile /root/.ssh/id_rsa_synology

</div>

<div>

<span class="Apple-style-span">You may also need to explicitly enable public key authentication on the DS. Again in DS:/etc/ssh/sshd_config:</span>

</div>

<div>

    PubkeyAuthentication yes           
    AuthorizedKeysFile      .ssh/authorized_keys

</div>

<div>

<span class="Apple-style-span">**Install Package Manager**</span>

</div>

<div>

<span class="Apple-style-span">Synology uses a package manager called [ipkg](http://en.wikipedia.org/wiki/Ipkg). Install it by following [these instructions](http://forum.synology.com/wiki/index.php/Overview_on_modifying_the_Synology_Server,_bootstrap,_ipkg_etc#How_to_install_ipkg).</span>

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">**Install rdiff-backup**</span>

</div>

<div>

<span class="Apple-style-span">Following [these instructions](http://forum.synology.com/wiki/index.php/Backup_rdiff-backup).</span>

</div>

<div>

<span class="Apple-style-span">  
</span>

</div>

<div>

<span class="Apple-style-span">**Setup Non-interactive Environment**</span>

</div>

<div>

<span class="Apple-style-span">When rdiff-backup connects to the DS over ssh, it is a non-interactive session. This mode omits a lot of the environment setup that normally happens when you log in interactively. In particular, the PATH is different:</span>

</div>

<div>

<span class="Apple-style-span">Interactive:</span>  

    DiskStation1> echo $PATH
    /opt/bin:/opt/sbin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/syno/bin:/usr/syno/sbin:/usr/local/bin:/usr/local/sbin

  
<span class="Apple-style-span">Non-interactive:</span>  

    root@local:~# ssh nas1 'echo $PATH'
    /usr/bin:/bin:/usr/sbin:/sbin:/usr/syno/bin

</div>

<div>

<span class="Apple-style-span">The critical difference in this case is that /opt/bin is missing in the non-interactive path. rdiff-backup requires the rdiff-backup executable to be in the path on the remote side. To set the path for non-interactive ssh sessions, add to DS:/etc/ssh/sshd_config</span>  

    PermitUserEnvironment yes

<span class="Apple-style-span">and create DS:~/.ssh/environment containing</span>  

    PATH=/opt/bin:/opt/sbin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/syno/bin

**<span class="Apple-style-span">DiskStation Hostname</span>**

</div>

<div>

Assuming you don't have a static IP to assign to your DS, you can use dynamic dns to maintain a consistent hostname by which you can reach it. This is functionality is built in to the DS web interface.

</div>

<div>

  

</div>

<div>

**<span class="Apple-style-span">Conclusion</span>**

</div>

<div>

Now just park the DS at a friend/family member's house, and viola, 1.5TB of online, offsite, secure storage, with no recurring costs for several years. You'll probably have to set up port forwarding on the router that the DS is connected to, so you can connect to it from the outside. Just use the same port you set up for sshd.

</div>

</p>
