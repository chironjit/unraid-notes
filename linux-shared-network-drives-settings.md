# Adding/Mounting Shared Drives(via SMB) to Ubuntu / Fedora
Distilled from the source listed below for my specific needs (protected access folders)

## Install CIFS Utils package

Ubuntu: `sudo apt-get install cifs-utils`
Fedora: `sudo dnf install cifs-utils`

## Create mount directories

For each folder you want to mount, create the mount folder

`sudo mkdir /media/sharedfolder`

## Create a credentials file

### Open new file in editor
`nano ~/.smbcredentials`

### Add username and password
```
username=myunraidusername
password=myunraiduserpassword
```
### Save
`Ctrl`+`o`

### Exit editor
`Ctrl` + `x`

### Change permissions
`chmod 600 ~/.smbcredentials`

## Add settings to fstab file

### Open fstab file with root
`sudo nano /etc/fstab` 

### Add entry
`//tower.local/sharename /media/sharedfolder cifs credentials=/home/ubuntuusername/.smbcredentials 0 0`
(change to selected server name or put in IP address)

### Save
`Ctrl`+`o`

### Exit editor
`Ctrl` + `x`

## Mount folder
`sudo mount /media/sharedfolder`

** Might get this on Fedora:

```
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

```


* You can mount unprotected (guest) folders via the original guide below

Source of notes: [Mounting CIFS Shares Permanently](https://ubuntu.com/server/docs/how-to-mount-cifs-shares-permanently)

