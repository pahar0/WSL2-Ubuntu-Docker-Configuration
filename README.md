# WSL2-Ubuntu-Docker-Configuration
Step-by-Step Guide for Setting Up and Using Docker on Ubuntu via Windows Subsystem for Linux (WSL2)

### Before continuing with the guide, disable and disconnect from any VPN you are connected to (Ivanti, Guardian, Netskope, etc...)

### Uninstall Docker Desktop if it is installed.
### Uninstall any other Ubuntu distribution that has been installed to avoid confusion.
- (Search in Add or Remove Programs for "Ubuntu" and uninstall it and/or launch `wsl.exe --unregister Ubuntu` to remove distributions in WSL)

# 0.
In case you have WSL installed, update it to make sure that version **2** is being used.
Launch in a PowerShell with administrator rights:

```
wsl.exe --update
```

# 1.
In a PowerShell with administrator rights, execute the following command:

```
wsl --install --web-download -d Ubuntu
```

(If the console returns that the `wsl` command does not exist, you may not be under version **2** of WSL.)

# 2.
Once the installation is finished, you will be asked to restart the system in case WSL was not activated and installed.

# 3.
An Ubuntu terminal will open and ask you for the username and password for the Ubuntu machine. 
(It is recommended to use the user `ubuntu`)

# 4.
Once inside the Ubuntu distribution, run the following command to create the WSL configuration file:

```
sudo nano /etc/wsl.conf
```


Paste the following code:

```
[boot]
systemd=true
```

Use `CTRL+O` to save it and then `CTRL+X` to exit.

# 5.
Within the Ubuntu distribution, run the following command to shut down WSL:

```
wsl.exe --shutdown
```

# 6.
Once WSL is shut down, open Windows File Explorer and type `%UserProfile%` in the address bar (or alternatively open the following path `C:\Users\<Username>\`).
Create the `.wslconfig` file (the file path should look like this: `C:\Users\<Username>\.wslconfig`). 
Write this code in the file to limit the memory usage of the WSL VM:

```
[wsl2]
memory=8GB
```

# 7.
Start the Ubuntu distribution (by launching `wsl.exe` in any terminal) and run the following commands to update the system and install necessary certificates:

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install ca-certificates curl gnupg lsb-release
```

# 8.
Add the official Docker GPG key:

```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```


# 9.
Set up the Docker repository:

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

# 10.
Update the `apt` package and install Docker:

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# 11.
When the installation is complete, assign Docker permissions to your user:

```
sudo usermod -aG docker $USER
```

# 12.
Stop the default Docker service and disable it so it does not start automatically:

```
sudo systemctl stop docker
sudo systemctl disable docker
```

# 13.
Create a specific service to run Docker with the TCP connection between Ubuntu and Windows.
Create the `dockerd.service` file with the following command:

```
sudo nano /etc/systemd/system/dockerd.service
```

Paste the following code:

```
[Unit]
Description=Docker Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service

[Service]
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://localhost:2375
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

[Install]
WantedBy=multi-user.target
```

Use `CTRL+O` to save it and then `CTRL+X` to exit.

# 14.
Restart `systemctl`, activate the new service and start it:

```
sudo systemctl daemon-reload
sudo systemctl enable dockerd.service
sudo systemctl start dockerd.service
```


# 15.
Open Visual Studio Code in Windows and install the `WSL` extension:

![](images/image12.png)

# 16.
Once the extension is installed, click on the button at the bottom left that says `Open a remote window` to open Visual Studio Code with a connection to the Ubuntu distribution of WSL and click on `Connect to WSL`:

![](images/image14.png) ![](images/image15.png)

# 17.
Install the Docker extension in WSL:

![](images/image16.png)

# 18.
In the settings of the Docker extension (by clicking on the gear icon), look for the `host` setting within the `Remote: [WSL: Ubuntu]` tab and add the environment variable `DOCKER_HOST=tcp://localhost:2375`:

![](images/image1.png)

# 19.
Install GIT:

```
sudo apt install git
```

# 20.
Once GIT is installed, open the `.gitconfig` file to save your credentials configuration:

```
sudo nano ~/.gitconfig
```


# 21.
Modify the following code with your USER, NAME and GITHUB email and paste it:

```
[user]
        email = github@email.com
        user = USER
        name = NAME
[credential]
        helper = store
[http]
        sslVerify = false
[core]
        autocrlf = true
```

Use `CTRL+O` to save it and then `CTRL+X` to exit.

# 22.
Download and install `wsl-vpnkit` to have a connection to the repositories through the VPN:

Download the release `0.4.1` from this link and save it in a simple path such as in `C:\`: https://github.com/sakai135/wsl-vpnkit/releases/download/v0.4.1/wsl-vpnkit.tar.gz

Import the `wsl-vpnkit` distribution by launching the following command in a PowerShell with administrator rights:

Remember to change the `C:\` path in case the file is in another location.

```
wsl.exe --import wsl-vpnkit --version 2 $env:USERPROFILE\wsl-vpnkit C:\wsl-vpnkit.tar.gz
```

# 23.
Create and activate a service for `wsl-vpnkit`:

In an Ubuntu terminal, launch the following commands:

```
wsl.exe -d wsl-vpnkit --cd /app cat /app/wsl-vpnkit.service | sudo tee /etc/systemd/system/wsl-vpnkit.service
sudo systemctl enable wsl-vpnkit
sudo systemctl start wsl-vpnkit
```

# 24.
Generate a new SSH key to be able to download the GitLab repositories:
In an Ubuntu terminal, launch the following command changing the email for your GitLab email (the same one you have put in the Git configuration)

```
ssh-keygen -t ed25519 -C "correo@gitlab.com"
```
Press enter 3 times to generate the SSH key.

Once generated, launch the following command (change the user if necessary) and copy the key (including email):

```
sudo cat /home/ubuntu/.ssh/id_ed25519.pub
```


# 25.
Go to the following page to add the SSH key:
https://github.com/settings/ssh/new
(or click on your profile icon at the top right > Settings > Click on `SSH and GPG keys` in the left sidebar and then on the `New SSH key` button)

Paste the SSH key into the `Key` field and press the `Add SSH key` button


