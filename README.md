# local_repo_debian
Right now it's a guide for creating a local debian repository with apt-mirror. Will rewrite it in python some time later.

------------
In this case for no particular reason I'll create a local repo on fresh RedOS installation. This should apply to most RHEL based distributions such as CentOS.

1. Get apt-mirror: `wget https://raw.githubusercontent.com/Stifler6996/apt-mirror/master/apt-mirror`
2. Than you'll need to modify apt-mirror script to dowload files to prefered directory:

```
sed -i 's|"base_path"   => '\''/var/spool/apt-mirror'\''|"base_path"   => '\''NEW PATH'\''|g' apt-mirror # replase NEW PATH with your prefered path
sed -i 's|config_file = "/etc/apt/mirror.list"|config_file = "config"|g' apt-mirror # change script to use config file in this directory. We'll create config later.
sed -i 's/"run_postmirror"       => 1/"run_postmirror"       => 0/g' apt-mirror # remove post script run, we don't need it here.
```

3. Install requiered perl packages: `sudo dnf install perl-File-Copy perl-File-Compare `

4. Create config file. For example you can use this command:

```
tee <<EOF >/dev/null config
# Proxmox repo
deb-amd64 http://download.proxmox.com/debian/pve buster pve-no-subscription
deb-amd64 http://download.proxmox.com/debian/pbs buster pbs-no-subscription


# Debian bullseye repo
deb-amd64 http://deb.debian.org/debian bullseye main contrib non-free
deb-amd64 http://security.debian.org/debian-security bullseye-security main contrib non-free
deb-amd64 http://deb.debian.org/debian bullseye-updates main contrib non-free 


skip-clean http://deb.debian.org/debian/dists/bullseye/main/installer-amd64
skip-clean http://deb.debian.org/debian/dists/bullseye-updates/main/installer-amd64
clean http://deb.debian.org/debian
clean http://security.debian.org/debian-securi
EOF
```

5. Give script execution rights: `chmod +x apt-mirror`

Downloading all this files will take some time, so I recommend passing it to terminal multiplexer - `tmux new-session -d './apt-mirror'`

6. Install and configure nginx:

```
sudo dnf install nginx
sudo setenforce 0 # well need to disable selinux to be able to run nginx properly
```

Nginx config file should have something like this in it:

`sudo vi /etc/nginx/nginx.conf`


```
location / {
            root   /home/red/repo/mirror; # repo location
            index  index.html;
            try_files $uri $uri/ =404;
            autoindex on;
          }
```

Modify /etc/apt/sources.list in your debian bases system \(replace ip with your repo machine\):
```
# Proxmox
deb [arch=amd64] http://IP/download.proxmox.com/debian/pve/ buster pve-no-subscription
deb [arch=amd64] http://IP/download.proxmox.com/debian/pbs/ buster pbs-no-subscription

# Debian 11
deb [arch=amd64] http://IP/deb.debian.org/debian/  bullseye main contrib non-free
deb [arch=amd64] http://IP/security.debian.org/debian-security/ bullseye-security main contrib non-free
deb [arch=amd64] http://IP/deb.debian.org/debian/ bullseye-updates main contrib non-free
```

Please **don't forget to run `apt-get update`** after this.

And that's all, you done.