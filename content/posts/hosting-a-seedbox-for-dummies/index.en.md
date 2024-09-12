+++
title = 'Hosting a Seedbox for Dummies'
date = 2024-09-09T01:24:56-03:00
draft = false
showSummary = true
summary = 'Setting up a private seedbox from the beggining: Setting up Windows, then Ubuntu Server, and finally Swizzin'
showBreadcrumbs = true
keywords = "ubuntu server, vps, torrent, seedbox, guide, swizzin, private tracker, hostbrr"
+++

With the influx of people wanting to set up their own seedboxes without being bothered by their ISPs, there's a growing need for them to consider setting their servers for themselves. This guide will tackle the basics, starting on how to use the terminal on Windows, choosing a hoster and installing (and learning) Swizzin.
## Part 1: Setting up Windows
### Step 1: Installing a good terminal

Using Command Prompt is outdated, boring and ugly. Powershell is no better. Time to update. For this, Windows Terminal will be installed and used. You can either get it for free on the [Microsoft Store](https://apps.microsoft.com/detail/9n0dx20hk701) or [install it with an alternative method](https://github.com/microsoft/terminal#installing-and-running-windows-terminal).  Cmder, Tabby are good alternatives, but we're going with the easiest option.

#### Why not PuTTY, KiTTY or *insert SSH client here*?
If you're reading this guide I'm having in consideration that you don't have extensive knowledge in setting up a Linux server. To learn this, you need to learn to work in a full fledged terminal, instead of having a program do the first part for you.

### Step 2: Installing Git for Windows
cmd sucks, Powershell sucks, Cygwin is stagnant. We'll install Git for Windows, which includes BASH emulation built in, to help familiarize yourself sooner with Linux's bash.

Get the 64-bit version of Git for Windows [here](https://git-scm.com/download/win). When installing, remember to check the "(NEW!) Add a Git Bash Profile to Windows Terminal" option in the _Select Components_ part. To make your life easier, in _Choosing the default editor used by Git_, scroll up the list and select "nano". No need to change anything else in the installation process.

Open Windows Terminal and you should see a _Git_ option when you click the arrow next to the new tab button. **You will be using this option starting from this point until the end of the tutorial.** You can also open settings (`CTRL + ,` or Downwards arrow > Settings) and select _Git_ as default profile. Remember to click **Save** at the end. Close your terminal and open another, you are set.

## Part 2: Getting a SSH key
We're going to create a key that's going to replace password login. Open a new terminal window and paste the following command, replacing _your-key-filename_ with the name you actually want.

```console
ssh-keygen -t ed25519 -f ~/.ssh/your-key-filename
```

It will ask you for a passphrase, you can either input your password here (hopefully you'll have a [good passphrase](https://xkcd.com/936/)) or leave it blank (not recommended) by pressing enter twice.

After you're done, if you check the .ssh folder (It's in your user folder, you might have to enable "View Hidden Folders") you'll see two files, one named like your key and one named the same, but with .pub at the end. The first one is your private key, do not **ever** share it with someone else, your VPS provider will never ask you for it! The second one is your public key, this one is the one that you will send over to your server.
## Part 3: Choosing a server provider
You'll hopefully choose a server provider that will provide
- A IPv4 address for you (IPv4 NAT is fine, but you'll have to set up a domain for it)
- Both SDD and HDD, with HDD space being at least 500GB (Pure HDD space is honestly fine, too)
- At least 1GB of RAM
- Will let you seed from public / private trackers (it's better to open a ticket and ask directly)
I've been using Hostbrr's Hybrid Storage VPS for almost a year, and it works perfectly for my usage. You can check their various offerings [here](https://my.hostbrr.com/order/main/index/storage?a=NjEw) ([here, without affiliate code](https://my.hostbrr.com/order/main/index/storage)), or ask whether their custom offer is still available (I'm using this):

```
2 vCore AMD EPYC 7502  
30 GB NVMe  
2 TB HDD  
6 TB Bandwidth @ 1Gbps 
IPv4+IPv6  
Helsinki, Finland  
$5.5/month
```

As an alternative, ServaRica has their Polar Bear Storage offer ([link, no aff](https://clients.servarica.com/store/black-friday-2023/polar-bear-storage-offer)) with:
```
2 cores Shared
2GB RAM
2TB HDD
Unlimited bandwidth @ 250Mbps + 1Mbps daily increase, limit 1Gbps OR
12TB Bandwidth @ 1Gbps
IPv4 + IPv6 (on request)
Montreal, Canada
$5/month
```


## Part 4: Setting up and securing your server
### Picking an OS
I personally use Ubuntu 22.04 for everything. I recommend you to do so, too.
### Making sure you can log in
In a new terminal window, write the following, replacing *root* in case your hoster gives you a different user (like *ubuntu* or *debian* or *user*) and replacing *your_server_ip* with your server IP address. 

{{< alert "circle-info" >}}
**Important** 22 is the default port for SSH. If you picked a server with IPv4 NAT (shared IP), this might be different, so make sure to change 22 to whatever was selected for you
{{< /alert >}}

```console
ssh root@your_server_ip -p 22
```

If it tells you "The authenticity of host \[...\] can't be established", it's safe to write "yes". You should now be inside your server.

### Creating a new user and adding it to sudoers
It's always a good idea to move from the default user to a new one created by yourself. In your terminal window, paste the following, changing "*ignacio*" to whatever user you want

{{< alert "circle-info" >}}
**Important** If you're on the user "root", you will get the error "*sudo: command not found*", remember to remove "sudo" on this step's commands
{{< /alert >}}

```console
sudo adduser ignacio
```

Type your (secure) password twice, and fill out the info (or press enter multiple times to leave blank), then type "Y" to accept. Now, you need to add this new user to the *sudo* group, giving it admin permissions basically
```console
sudo usermod -aG sudo ignacio
```

To check whether it was successfully added, do the following command
```console
sudo groups ignacio
```

and you should see something similar to this: `ignacio : ignacio sudo`

### Making sure you can log in as your new user
Send the command `exit` on your terminal (or close it and open it back again) and do the same SSH login command, but changing *root* to your newly created username
```console
ssh ignacio@your_server_ip -p 22
```

Provide the password, and if successful, you should've logged back in, this time with your new user. Cool!

### Sending your public key over to your server
Logging with a password is old and prone to password dictionary attacks, we will send your public key over to your server, as the first step of disabling password logins altogether.

In a new terminal window, do the following command. Remember to replace ignacio with your actual user!
```console
ssh-copy-id -i ~/.ssh/your-key-filename.pub -p 22 ignacio@your_server_ip
```

It will ask you for your password, provide it. If successful, you'll see the following
```bash
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ignacio@your_server_ip'"
and check to make sure that only the key(s) you wanted were added.
```

Cool! Let's log in with your keyfile now
```console
ssh ignacio@your_server_ip -p 22 -i ~/.ssh/your-key-filename
```

Provide your keyfile passphrase / password (in case you used one) and you should've logged in the server without using your actual user's password.

### Enabling firewall with UFW
Check UFW status with `sudo ufw status`, should reply with `Status: inactive`.
Before enabling, let's make sure we don't lock ourselves from accessing, so we need to allow port 22 (or your SSH port) to be open.
```console
sudo ufw allow 22
```

The response to that should be `Rules updated` and `Rules updated (v6)`. Now we can enable the firewall with
```console
sudo ufw enable
```

Accept the SSH disruption warning by typing "y" and pressing enter. Now, `sudo ufw status` should show allow rules to port 22.

### Changing your SSH port to a random one
Yes, we just allowed port 22 through our firewall, but changing the port to a less obvious one will reduce attempted logins by bots to nearly zero, and multiple failed attempts can make a less resource-heavy machines slower.

{{< alert "circle-info" >}}
**Important** If you're on IPv4 NAT or you had a different SSH port to begin with, you can skip this section.
{{< /alert >}}

Pick a random port, for this example I will be using port 566. Allow traffic through port 566 on UFW
```console
sudo ufw allow 566
```

Now change SSH port to your chosen one with this one-liner, remember to change 566 to your preferred port
```console
sudo sed -i 's/#Port 22/Port 566/' /etc/ssh/sshd_config
```

And restart the SSH service
```console
sudo systemctl restart sshd
```

In case you want to check, do `nano /etc/ssh/sshd_config` and it should look similar to this
```bash
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

Port 566
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
(...)
```

Open a new window and check if you can log in with your new port instead of the default one
```console
ssh ignacio@your_server_ip -p 566 -i ~/.ssh/your-key-filename
```

If you were successful, you can now deny the default SSH port with
```console
sudo ufw deny 22
```

### Disabling password logins and root logins
First copy and paste this command that will disable logging in as root
```console
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
```

Then copy and paste this command that will disable logging in with a password, hence only allowing to log in with the correct keyfile
```console
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
```

Once again, restart the SSH service
```console
sudo systemctl restart sshd
```

Now, root login is disabled and you will only be able to log in with your keyfile.

### Creating a SSH alias in your home device
Your SSH login command is getting insanely long. We can set up a file named "config" inside ~/.ssh in your machine, adding all those extra commands to an alias that will make it way easier.

Create a blank file on .ssh/ inside your user folder named "*config*", and paste the following, replacing "*seedbox*" with whatever alias you want, "*ignacio*" with your chosen user, *566* with the SSH port you chose and *"your-key-filename*" with the name of your private key.
```console
Host *
    ServerAliveInterval 40

Host seedbox
  HostName your_server_ip
  User ignacio
  Port 566
  IdentityFile ~/.ssh/your-key-filename
```

Save it, close your terminal and open a new one. Instead of doing the whole SSH command with your user, IP, port and keyfile, simply do:
```console
ssh seedbox
```

and you will be logged in.

### Enabling fail2ban and setting it up to the extreme
fail2ban is a handy program that will keep a list of IPs who failed to log in. You can customize it to a T, from how many tries to how many ban minutes / hours, etc. Since we already have everything set up to log in with a simple command, we'll set up fail2ban to
- Ban anyone who tries to log in as root permanently
- Ban anyone who doesn't provide a keyfile permanently
- Ban anyone who provides the wrong keyfile for 24 hours

Update your system package list
```console
sudo apt-get update
```

Upgrade any outdated package. If you get a pink screen asking you which services you want to restart, just press enter.
```console
sudo apt-get dist-upgrade
```

 Install fail2ban
```console
sudo apt-get install fail2ban
```

Create a new file with nano named jail.local
```console
sudo nano /etc/fail2ban/jail.local
```

Paste the following, remember to change 566 to your actual SSH port
```console
[sshd]
enabled = true
port = 566
filter = sshd
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-root]
enabled = true
port = 566
filter = sshd[mode=aggressive]
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-nokeyfile]
enabled = true
port = 566
filter = sshd-nokeyfile
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-wrongkeyfile]
enabled = true
port = 566
filter = sshd-wrongkeyfile
logpath = /var/log/auth.log
maxretry = 1
bantime = 86400
```

To save, press `CTRL + X`, press `Y` and then press enter.
Create the following filter by doing `sudo nano /etc/fail2ban/filter.d/sshd-nokeyfile.conf` and paste the contents:
```console
[Definition]
failregex = ^.*Did not receive identification string from <HOST>$
ignoreregex =
```

Save, and create the last filter by doing `sudo nano /etc/fail2ban/filter.d/sshd-wrongkeyfile.conf` and paste the following:
```console
[Definition]
failregex = ^.*error: Authentication failed for .* from <HOST>$
ignoreregex =
```

Save, then start and enable fail2ban
```console
sudo systemctl start fail2ban
```

```console
sudo systemctl enable fail2ban
```

To check the status of all jails (to check an individual jail, put the name of the jail after "status")
```console
sudo fail2ban-client status
```

To unban an IP
```console
sudo fail2ban-client unban <IP_ADDRESS>
```

### Enabling auto-updates
Install these packages
```console
sudo apt-get install unattended-upgrades update-notifier-common
```

Enable auto-updates (On the pink screen, make sure "Yes" is selected and press enter)
```console
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Do `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and make sure it looks like this
```shell
// Automatically upgrade packages from these (origin:archive) pairs
//
// Note that in Ubuntu security updates may pull in new dependencies
// from non-security sources (e.g. chromium). By allowing the release
// pocket these get automatically pulled in.
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        // Extended Security Maintenance; doesn't necessarily exist for
        // every release and this system may not have it installed, but if
        // available, the policy for updates is such that unattended-upgrades
        // should also install from here by default.
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
//      "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```

Also, check `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`, make sure it has these lines
```shell
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Enable and start the service
```console
sudo systemctl enable unattended-upgrades
```

```console
sudo systemctl start unattended-upgrades
```

## Part 5: Holy fuck, can we install Swizzin yet?
### Installing Swizzin

Enable access to ports 80 (HTTP) and 443 (HTTPS)
```console
sudo ufw allow 80 && sudo ufw allow 443
```

Switch to root
```console
sudo -i
```

Run the Swizzin installation wizard
```console
bash <(curl -sL s5n.sh) && . ~/.bashrc
```

Press "Yes" once it tells you to set up, then input your username. You will get a notice like this:
```shell
INFO    The user ignacio already appears to be present on this machine; however, it does not appear to be configured as a swizzin user.
        The user will be added to swizzin.
INPUT   Continue setting up user?
(Y/n) >
```

Input "Y" and press enter to confirm. On password, re-type the password you already set up for your user.

On "Install software", select the following (Move with up/down arrows, select with spacebar)
- nginx
- qBittorrent (You can select deluge or rtorrent, though)
- panel

Press the TAB button to select `<Ok>` and press ENTER.
On the "Make some more choices" screen, scroll down and select `letsencrypt`. Anything else can be installed later. Again, TAB, select `<Ok>` and press ENTER.

Select your preferred qBittorrent version. I'll pick 4.6 because I use RSS feeds. TAB and continue. This will start installing all your selected applications. In lower RAM servers, there might be some parts where it looks like everything's frozen. **Do not worry**, just leave it and let it load until it finishes.

While it finishes, let's continue with the following step

### Setting up your domain with Cloudflare
We will use Cloudflare for this. Domains are cheap for the first year, but if you don't want to spend any money, [Afraid's FreeDNS](https://freedns.afraid.org/), [DuckDNS](https://www.duckdns.org/), [deSEC](https://desec.io/) or [eu.org](https://nic.eu.org/) are free alternatives to get a subdomain.

First thing in Cloudflare: Select your domain, go to SSL/TLS > Overview > Configure and switch Custom SSL/TLS to "Full".
Now, in DNS > Records > Add Record:
- Select type `A`
- In name, if you want to use a subdomain (tee.yourdomain.com), then write "tee". If you want to use the full domain just put `@`
- IPV4 Address: Your seedbox IP
- Proxy status: Leave as Proxied (Orange cloud)

Then click "Save".


Lastly, on the top right, select the profile icon > My Profile, select API Tokens (Or [click here](https://dash.cloudflare.com/profile/api-tokens)), scroll down and click "View" on Global API Key, copy it; you will use it now.

### Setting up your domain in your server
Go back to your terminal, you should see the following:
```shell
SUCCESS Panel installed
INFO    Installing letsencrypt
DOCS    Further reference: https://swizzin.ltd/applications/letsencrypt
INPUT   Enter domain name to secure with LE
```

Enter your domain (or subdomain), when asked *"Do you want to apply this certificate to your swizzin default conf?"* type "y". Same with *"Is your DNS managed by CloudFlare?"* and *"Does the record for this subdomain already exist?"*. When asked for your CF API key, paste the global api key you copied early, then input your Cloudflare account email.

Finally, you'll see this on your screen
```shell
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2297    0  2297    0     0   7098      0 --:--:-- --:--:-- --:--:--  7089
...     Performing apt update
        ✔   Done
...     Performing installation of 1 apt packages
        (socat)
        ✔   Apt install complete
...     Installing ACME script
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1032    0  1032    0     0   7709      0 --:--:-- --:--:-- --:--:--  7701
        ✔   Done
...     Registering certificates
        ✔   Certificate acquired
...     Installing certificate
[Sun  8 Sep 05:47:14 UTC 2024] The domain "tee.yourdomain.com" seems to already have an ECC cert, let\'s use it.
[Sun  8 Sep 05:47:14 UTC 2024] Installing CA to: /etc/nginx/ssl/tee.yourdomain.com/chain.pem
[Sun  8 Sep 05:47:14 UTC 2024] Installing key to: /etc/nginx/ssl/tee.yourdomain.com/key.pem
[Sun  8 Sep 05:47:14 UTC 2024] Installing full chain to: /etc/nginx/ssl/tee.yourdomain.com/fullchain.pem
[Sun  8 Sep 05:47:14 UTC 2024] Running reload cmd: systemctl reload nginx
[Sun  8 Sep 05:47:14 UTC 2024] Reload successful
        ✔   Certificate installed
SUCCESS Letsencrypt installed
INFO    Package installation took 129 minutes and 16 seconds


SUCCESS Swizzin installation complete!
INFO    Seedbox can be accessed at https://server.ip
INFO    You can now use the box command to manage swizzin features, e.g. `box install nginx panel`
DOCS    Further reference: https://swizzin.ltd/getting-started/box-basics
INFO    Shell completions for the box command have been installed. They will be applied the next time you log into a bash shell
...     Executing post-install commands
        ✔   Post-install commands finished
```

In your browser, open your domain (or subdomain), log in and you should see Swizzin's panel. Remember to exit sudo user by simply typing `exit` on your terminal and pressing enter.

### Opening up your torrent port
In your panel, select qBittorrent at the left sidebar (or go directly to `https://tee.yourdomain.com/qbittorrent`), log in with the same info as before. Click on the settings icon (gear icon), select tab "Connection" and copy the port listed on "Port used for incoming connections" (or replace it for your preferred one). In my case, my port is 2549.

Scroll down, click "Save" and on your terminal do
```console
sudo ufw allow 2549
```

Check if your port is open with any port checker like [Dynu's Port Check](https://www.dynu.com/NetworkTools/PortCheck), wait until the "SUCCESS!" message appears. You're done! You now have a working seedbox available to you.

### Optional settings for qBittorrent

#### Disabling torrent limits
Go to the settings icon > Connection and **deselect** the following:
- Global maximum number of connections
- Maximum number of connections per torrent
- Global maximum number of upload slots
- Maximum number of upload slots per torrent
#### Setting up your seedbox for private tracker usage only
Go to the settings icon > Bittorrent and **deselect** the following:
- Enable DHT (decentralised network) to find more peers
- Enable Peer Exchange (PeX) to find more peers
- Enable Local Peer Discovery to find more peers}
### General maintenance (Good to do every few days / every week)
#### Update your package lists
```console
sudo apt-get update
```

#### Upgrade outdated packages
```console
sudo apt-get dist-upgrade
```

#### Update Swizzin to the latest release
```console
sudo box update
```