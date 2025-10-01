# Adding/Mounting Shared Drives(via SMB) to Ubuntu / Fedora
Distilled from the source listed below for my specific needs (protected access folders)

## Notes on guide
The latest version of this guide contains steps to use either CIFS or Systemd mount to mount the shared folder. The first 2 main steps are the same, and the second 2 will differ. Please note that for distros that use systemd (such as Ubuntu and Fedora), the recommended method is systemd mount.

Additional notes:
1. When item is in angled bracket(for example `<your_linux_username>`), replace it(including the angled bracket) with your version of the required item
2. This guide now walks you through forcing the latest samba protocol version (3.1.1) so ensure that your unraid server supports/has this enabled (see the Samba Hardening guide in this repository)
3. This guide assumes your share is password protected and will go through the steps accordingly
4. Some distros on systemd may start forcing you into the systemd method, so please follow that guide
5. Get your numeric uid and gid from via `id <your_linux_username` from the terminal

---

## Common steps
### Step 1: Install CIFS Utils package

Ubuntu: `sudo apt install cifs-utils`
Fedora: `sudo dnf install cifs-utils`

### Step 2: Create mount directories

For each folder you want to mount, create the mount folder. You can use media or home(`/home/<your_linux_username>`) folder for example as the root folder for your shares as an option:

`sudo mkdir /<folder>/<share_folder>` or `mkdir /home/<your_linux_username>/<share_folder>` (sudo not required if creating the folder in your home folder)

for example, you can have it as:
`sudo mkdir /media/linuxiso` or `mkdir /home/username/linuxiso`

### Step 3: Create a credentials file
We create a credentials file to prevent us from having to save the credentials directly in the fstab file. We will store this in the home folder

#### Open new file in editor
`nano ~/.smbcredentials`

#### Add username and password
```
username=myunraidusername
password=myunraiduserpassword
```
#### Save and exit
Save the file using `Ctrl`+`o`

and

Exit the editor using `Ctrl` + `x`

#### Change permissions
`chmod 600 ~/.smbcredentials`

---

## Systemd Method
### Step 1: Create mount setting file
#### Create a new mount file
Create a new mount setting file. Follow the naming convention where the name matches the mount point(`/etc/systemd/system/<folder>-<share_folder>.mount):

`sudo nano /etc/systemd/system/media-linuxiso.mount`

#### Input the mount settings
The following is list of items you would put in your share folder mount settings:
```
[Unit]
Description=<any description>

[Mount]
What=//<server_ip_or_hostname>/<sharename>
Where=/<folder>/<sharedfolder>
Type=cifs
Options=rw,file_mode=0700,dir_mode=0700,uid=<your_numeric_uid>,credentials=/home/<your_linux_username>/.smbcredentials
DirectoryMode=0700

[Install]
WantedBy=multi-user.target
```

for read-only shares, use the following options and director mode:
```
Options=ro,file_mode=0500,dir_mode=0500,uid=<your_numeric_uid>,credentials=/home/<your_linux_username>/.smbcredentials
DirectoryMode=0500
```

As an example, you can use:
```
[Unit]
Description=linux isos share folder on my unraid

[Mount]
What=//tower.local/linuxisos
Where=/media/linuxiso
Type=cifs
Options=rw,file_mode=0700,dir_mode=0700,uid=1000,credentials=/home/username/.smbcredentials
DirectoryMode=0700

[Install]
WantedBy=multi-user.target
```

#### Save and exit
Save the file using `Ctrl`+`o` and exit the editor using `Ctrl` + `x`

### Step 2: Create an automount setting file
#### Create a new automount file
Create a new automount setting file. Follow the naming convention where the name matches the mount point(`/etc/systemd/system/<folder>-<share_folder>.automount):

`sudo nano /etc/systemd/system/media-linuxiso.automount`



#### Input the automount settings
The following is list of items you would put in your share folder automount settings:

```
[Unit]
Description=<any description>

[Automount]
Where=/<folder>/<sharedfolder>
TimeoutIdleSec=120

[Install]
WantedBy=multi-user.target
```

as an example:
```
[Unit]
Description=automount linux isos share folder on my unraid

[Automount]
Where=/media/linuxiso
TimeoutIdleSec=120

[Install]
WantedBy=multi-user.target
```

#### Save and exit
Save the file using `Ctrl`+`o` and exit the editor using `Ctrl` + `x`

### Step 3: Enable and start the service
Enable the service using `sudo systemctl daemon-reload`.

Start the service using `sudo systemctl enable media-linuxiso.automount --now`

### Troubleshooting
Chec status:
Check mount status: `systemctl status media-linuxiso.mount`
Check automount status: `systemctl status media-linuxiso.automount`

Check logs:
View system logs: `journalctl -xe`

Hide duplicate of shares in file explorer in gnome:
Add `x-gvfs-hide` option to systemd options: `Options=rw,file_mode=0700,dir_mode=0700,uid=<your_numeric_uid>,credentials=/home/<your_linux_username>/.smbcredentials,x-gvfs-hide`

---

## FSTab Method
### Step 1: Add settings to fstab file
#### Open fstab file with root
`sudo nano /etc/fstab`

#### Step 2: Add entry
`//<ip>/<sharename> /<folder>/<sharedfolder> cifs uid=<your_numeric_uid>,gid=<your_numeric_gid>,credentials=/home/<your_linux_username>/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`

Examples supported include:
`//192.168.0.10/linuxisos /media/linuxiso cifs uid=1000,gid=1000,credentials=/home/username/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`
`//tower.local/linuxisos /media/linuxiso cifs uid=1000,gid=1000,credentials=/home/username/.smbcredentials,vers=3.1.1,sec=ntlmssp,cache=strict,rw,_netdev 0 0`
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


#### Step 3: Save and exit
Save the file using `Ctrl`+`o` and exit the editor using `Ctrl` + `x`

### Step 4: Mount folder
`sudo mount /<folder>/<share_folder>`

or to mount all folders at once:

`sudo mount -a`

### Optional guide: To unmount via shell
In case you would like to unmount your folder manually, you can use:

`sudo umount /<folder>/<share_folder>`

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
* [Systemd mount discussion](https://discussion.fedoraproject.org/t/suddenly-user-cifs-mounts-not-supported/78652/11)
* [Systemd mount docs](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html)
