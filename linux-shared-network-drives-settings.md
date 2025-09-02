# Adding/Mounting Shared Drives(via SMB) to Ubuntu / Fedora
Distilled from the source listed below for my specific needs (protected access folders)

## Notes on guide
1. When item is in angled bracket(for example `<yourlinuxusername>`), replace it(including the angled bracket) with your version of the required item
2. This guide now walks you through forcing the latest samba protocol version (3.1.1) so ensure that your unraid server supports/has this enabled (see the Samba Hardening guide in this repository)
3. This guide assumes your share is password protected and will go through the steps accordingly
4. Some distros on systemd may start forcing you into the systemd method, so please follow that guide

## FSTab Method
### Install CIFS Utils package

Ubuntu: `sudo apt-get install cifs-utils`
Fedora: `sudo dnf install cifs-utils`

### Create mount directories

For each folder you want to mount, create the mount folder. You can use media as the root folder for your shares as an option:

`sudo mkdir /<folder>/<yoursharedfolder>`

for example, you can have it as:
`sudo mkdir /media/linuxiso`

### Create a credentials file
We create a credentials file to prevent us from having to save the credentials directly in the fstab file. We will store this in the $HOME(`/home/<yourlinuxusername>`) folder

#### Open new file in editor
`nano ~/.smbcredentials`

#### Add username and password
```
username=myunraidusername
password=myunraiduserpassword
```
#### Save
`Ctrl`+`o`

#### Exit editor
`Ctrl` + `x`

#### Change permissions
`chmod 600 ~/.smbcredentials`

### Add settings to fstab file

#### Open fstab file with root
`sudo nano /etc/fstab`

#### Add entry
`//<ip>/<sharename> /<folder>/<sharedfolder> cifs uid=<yourlinuxusername>,gid=<yourlinuxusername>,credentials=/home/<yourlinuxusername>/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`

Examples supported include:
`//192.168.0.10/linuxisos /media/linuxiso cifs uid=username,gid=username,credentials=/home/username/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`
`//tower.local/linuxisos /media/linuxiso cifs uid=username,gid=username,credentials=/home/username/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`
(change to selected server name or put in IP address)

Note on options
`uid`:      Set user of the mounted drive. relevant for read/write
`gid`:      Set group for the mounted drive. Usually not necessary but included for when it may be needed
`vers`:     Enforces the samba version (latest default tries to negotiate the highest samba version 2.1 or above)
`sec`:      Enforce ntlmssp - use NTLMv2 password hashing encapsulated in Raw NTLMSSP message (technically this is already the default for newer kernels but we are making it explicit)
`cache`:    Strict enforces CIFS/SMB2 protocol strictly for cache coherency
`rw`:       Marks share as readable and writeable. Use `ro` for read-only shares
`_netdev`:  fstab setting to enable auto mount only after network is available

Other options that may be helpful:


#### Save
`Ctrl`+`o`

#### Exit editor
`Ctrl` + `x`

### Mount folder
`sudo mount /<folder>/<yoursharedfolder>`

or to mount all folders at once:

`sudo mount -a`

### To unmount via shell
`sudo umount /<folder>/<yoursharedfolder>`

** Might get this on Fedora:

```
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

```

### Troubleshooting "User CIFS Not Supported" error
If you encounter this error:
<p align="center"><img src="./images/user_cifs_not_supported_error.png" title="" alt="user_cifs_error_message"></p>

Change the following settings:

```
sudo chmod u+s /bin/mount
sudo chmod u+s /bin/umount
sudo chmod u+s /usr/sbin/mount.cifs
```


#### ** You can mount unprotected (guest) folders via the original guide below(not applicable if you have hardened your samba settings)


Source of notes:
* [Latest Linux mount.cifs manpage guide](https://www.mankier.com/8/mount.cifs)
* [Mounting CIFS Shares Permanently](https://ubuntu.com/server/docs/how-to-mount-cifs-shares-permanently)
* ["user" CIFS mounts not supported error](https://discussion.fedoraproject.org/t/suddenly-user-cifs-mounts-not-supported/78652)
* [Systemd Mount](https://discussion.fedoraproject.org/t/suddenly-user-cifs-mounts-not-supported/78652/11)
