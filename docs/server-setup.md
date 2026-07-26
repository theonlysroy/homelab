# Setting up a Server from scratch

> Since this server is setup on a Laptop, there will be some additional steps to follow.

## Phase - 1

### 1. Installation

- OS: Debian 13 (w/o GUI)
- On *Software Installation* section of the installer (very last step), we need to check only **standard system utilities** to install the OS without any *Debian Desktop Environment* and *DE Task (Gnome, Mate, etc.)*
- Set *root* user password and a *non root user* as instructed
- Complete the installation

### 2. System update

- After fresh installation, login to `root` account, reboot and update the system

```bash
apt update && apt upgrade -y
```

### 3. Network Manager setup (optional)

> To use WiFi we need to follow these steps.

- Log in to the `root` account
- Install *network-manager*.

    ```bash
    apt update && apt upgrade
    apt install network-manager
    ```
- Check Wifi Interface and make sure wifi is on

    ```bash
    ip a
    
    # to make sure wifi radio is on
    nmcli radio wifi on
    ```
- Scan for Available Networks

    ```bash
    nmcli device wifi list
    ```
- Connect to the wifi network

    ```bash
    nmcli device wifi connect "SSID" password "PASSWORD"
    ```
- Verify connection

    ```bash
    ip a
    ping -c 3 google.com 
    ```

### 4. Setup ssh (optional)

> This step is required if the ssh server is not already installed. During the *installation* we didn't check the *ssh server* option on the list of software installation; so we need to proceed with these steps.

After we start the ssh server, we can connect the server from the local machine using ssh and continue the server setup.

- Install OpenSSH server

    ```bash
    apt install openssh-server -y
    systemctl enable --now ssh
    systemctl status ssh
    ```
- Confim ssh access to the server

    ```bash
    ssh root@<server-ip>
    ```

### 5. Allow Root Login with Password (optional)

> [!WARNING]
> Allowing Root login with password is highly security anti-pattern. This action is not recommended for long term usage and we'll update these settings soon.

Debian by default restricts `root` user login with only password over ssh. So there's two options now:

1. **Setting up a non root user (Recommended)**
    - Add a non-root user, say *devadmin*.

        ```bash
        adduser devadmin
        ```

    - Install `sudo` (as it doesn't come pre-installed)
        ```bash
        apt install sudo -y
        ```

    - Add the non-root user to the sudo group

        ```bash
        usermod -aG sudo devadmin
        ```

    - SSH using the new user

        ```bash
        ssh devadmin@<server-ip>
        ```

2. **Allow root user login for a limited time (HIGH RISK)**

    ```bash
    # update sshd_config
    apt install vim
    vim /etc/ssh/sshd_config

    # Change `PermitRootLogin` option, remove "#" and add "yes"
    PermitRootLogin yes
    
    # Restart ssh service
    systemctl restart ssh
    ```

> Now we have configured all the required settings on the server (our laptop) directly from the device and can now ssh into it and use it remotely as a proper server.

---

## Phase - 2

### 6. Setup non-root user, *if skipped in last step*

> If in the last step, used the Root login method, this step is required.

- ssh in to the server with the root user

    ```bash
    ssh root@<ip>
    ```

- Install sudo, create a non-root user, add the user to the sudo group
    
    ```bash
    # update and install sudo
    apt update && apt install sudo -y

    # will prompt to add password
    useradd <user_name> 

    # adds the user to the sudo group;
    # essential for operations requiring sudo priviledges
    usermod -aG sudo <user_name>
    ```

### 7. Disable Remote Root Login

> Once the non-root user ssh login is confirmed and working with sudo priviledges, run this.
> THIS STEP IS ABSOLUTELY NECESSARY, DO NOT SKIP

To check if the non root user ssh access is working or not, ssh login with that user and run any sudo command, say `sudo apt update`. If success, then good to go.

- Edit */etc/ssh/sshd_config*
    ```bash
    sudo vim /etc/ssh/sshd_config
    ```

    ```
    # Update the PermitRootLogin
    PermitRootLogin no
    ```

- Restart SSH

    ```bash
    sudo systemctl restart ssh
    ```

### 8. SSH Login with Public/Private Keys (non-root user)

- Create SSH keys on local machine
    `-C` is used for a comment to remember what the key is about

    ```bash
    ssh-keygen -t ed25519 -C "<user>@<serverip/server-name>"
    ```
    Press enter to accept the default path `~/.ssh/id_ed25519` or change the path if already that path exists with something like `~/.ssh/id_ed25519_devadmin_server`

    Following this pattern - `id_<key_algo>_<user>_<purpose>`

- Copy the key to the remote server
    *filepath* will be the public key file path. Say if the key name is default `id_ed25519` then there will be another file `id_ed25519.pub` in the `~/.ssh` directory

    ```bash
    ssh-copy-id -i <filepath> <user>@<serverip>
    ```

- Test login with SSH key
    ```bash
    ssh <user>@<serverip> 
    ```

- (Recommended) Disable password authentication entirely

    Once key login is confirmed, update in `/etc/ssh/sshd_config`:
    ```bash
    PasswordAuthentication no
    PubkeyAuthentication yes
    ```
    ```bash
    # restart ssh service
    sudo systemctl restart ssh
    ```

### 9. Update and Upgrade the System

- Update the system

    ```bash
    sudo apt update
    sudo apt full-upgrade -y
    sudo apt autoremove -y
    sudo apt autoclean
    ```

- Reboot if any kernel update was applied
    ```bash
    sudo reboot
    ```

### 10. Install utilities

```bash
sudo apt install vim curl htop fastfetch -y
```

### 11. Install and setup Nginx

- Installation, Enable nginx start on system startup

    ```bash
    sudo apt install nginx -y
    sudo systemctl enable --now nginx
    ```

- Create Virtual subdomains (directory + index)

    ```bash
    sudo mkdir -p /var/www/sub.example.com/html

    # Add user permissions on the directories, recursively
    sudo chown -R $USER:www-data /var/www/sub.example.com/html

    # change file permissions, rwx for user, rw for group and other (public view)
    sudo chmod -R 755 /var/www/sub.example.com
    ```

- Add an index file

    ```bash
    sudo vim /var/www/sub.example.com/html/index.html
    ```

    ```html
   <!DOCTYPE html>
    <html>
    <head><title>sub.example.com</title></head>
    <body><h1>It works - sub.example.com</h1></body>
    </html>   
    ```

- Add nginx *server block* under `sites-available`

    ```bash
    sudo vim /etc/nginx/sites-available/sub.example.com
    ```
    ```bash
    server {
        listen 80;
        listen [::]:80;
        
        root /var/www/sub.example.com/html;
        index index.html index.htm;

        server_name sub.example.com;
        
        # this is can be made separate modules
        location / {
            try_files $uri $uri/ /index.html =404;
        }
    }
    ```

- Symlink to `sites-enabled`

    ```bash
    sudo ln -s /etc/nginx/sites-available/sub.example.com /etc/nginx/sites-enabled/
    # check the configurations
    sudo nginx -t
    
    # restart nginx
    sudo systemctl reload nginx
    # or
    sudo nginx -s reload
    ```
    > To get this working we need to point the subdomain's DNS A/AAAA record at the server's IP. Since we are working locally without an actual domain, we need extra steps

    **Extra steps to work with sub-domains**
    
    - Edit `/etc/hosts` file on the local system
    
         ```bash
        sudo vim /etc/hosts
         ```

        ```bash
        # Add this lines
        <server-ip> sub.example.com
        ```

        This will register the subdomain to the server ip on the local machine

### 12. Firewall setup with `ufw`

- Installation
    ```bash
    sudo apt install ufw -y
    ```

- Allow required services
    ```bash
    sudo ufw allow OpenSSH
    sudo ufw allow 'Nginx HTTP'
    sudo ufw allow 'Nginx HTTPS'
    ```

- Allow any custom development port
    ```bash
    sudo ufw allow 3000/tcp # e.g. Node dev server
    ```

- Enable ufw and verify
    ```bash
    sudo ufw enable
    sudo ufw status verbose
    ```

### 13. Fail2Ban Setup

- Installation
    ```bash
    sudo apt install fail2ban -y
    ```

- Copy a local override for updating `jail.conf`

    ```bash
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    sudo vim /etc/fail2ban/jail.local
    ```

    Key settings to check/set under `[DEFAULT]`:

    ```bash
    [DEFAULT]
    bantime = 1h
    findtime = 10m
    maxretry = 5
    ignoreip = 127.0.0.1/8 ::1
    
    [sshd]
    enable = true
    port = ssh
    filter = sshd
    logpath = %(sshd_log)s
    backend = %(sshd_backend)s    
    ```

- Enable and start
    ```bash
    sudo systemctl enable --now fail2ban
    sudo fail2ban-client status
    sudo fail2ban-client status sshd
    ```
---

