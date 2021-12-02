# Docker VPN Project: Setup Wireguard VPN server using Docker

### Created by: Samantha Phillips

Resources:

[https://thematrix.dev/setup-wireguard-vpn-server-with-docker/](https://thematrix.dev/setup-wireguard-vpn-server-with-docker/)

NOTE: I used a Ubuntu 20.04 (LTS) 64 droplet on DigitalOcean for this installation process.

## Step 1: Install Docker

NOTE: If you are not currently logged on as the root user, switch to the root user...it will save you from having to add sudo to almost all of the commands :)

Run the following two commands:

    apt update
    apt install docker.io
    
Notes: 
The first command updates apt, so it is pulling the latest installs from the apt repository.
The second command installs docker.


## Step 2: Install Docker Compose

First, install curl (if it is not already installed) with the following command:

    apt install curl
  
So, what is curl? Well, I'm glad you asked. Curl is a command line tool to transfer data to or from a server. We will be using it to install Docker Compose from Github. Docker Compose is a tool that was developed to help define and share multi-container applications.

Next, run the following commands to install docker-compose:

    curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    
The first command is transfering the docker-compose data found on the github site, that is shown in the command. The second command is modifying the permissions of the downloaded docker-compose data by making it able to be executed.   

After docker-compose is finished installing, run the following command to check the installation is successful and the version.

    docker-compose –version
    
Here is what my result looked like after running the command:


![Example One](/docs/assets/images/Picture1.png)

## Step 3: Create Directories

Run the following command to make two new directories (~/wireguard/ and ~/wireguard/config/) that will be used to setup Wireguard

    mkdir -p ~/wireguard/config
    
## Step 4: Create the .yml file

What is a .yml file?
A yml file allows for the deployment, combination and configuration of multiple docker containers at the same time.

What is its purpose for this setup?
The .yml file will be used to setup/configure the docker container that will host the wireguard VPN server.

First, install vim (or your preferred text editor).

Run the following command to install vim

    apt install vim
    
NOTE: You will need to know the IP Address of your host before continuing. You can use the command "ip addr" to find the ip address

Run the following command to create the docker-compose.yml file and begin editing it

    vim ~/wireguard/docker-compose.yml
    
Next copy and paste the content below into the file:

    version: '3.8'
    services:
      wireguard:
        container_name: wireguard
        image: linuxserver/wireguard
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Hong_Kong
          - SERVERURL=1.2.3.4
          - SERVERPORT=51820
          - PEERS=pc1,pc2,phone1
          - PEERDNS=auto
          - INTERNAL_SUBNET=10.0.0.0
        ports:
          - 51820:51820/udp
        volumes:
          - type: bind
            source: ./config/
            target: /config/
          - type: bind
            source: /lib/modules
            target: /lib/modules
        restart: always
        cap_add:
          - NET_ADMIN
          - SYS_MODULE
        sysctls:
          - net.ipv4.conf.all.src_valid_mark=1
      
    
A few changes will need to be made before you save the .yml file:

- TZ refers to timezone, so you will need to change it based on your timezone. Choose yours from TZ database name from [Wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). I chose America/Chicago for my installation.
- SERVERURL refers to the IP Address of the server you are using. Replace the 1.2.3.4 with whatever the IP address is of your host. My server IP is 104.248.6.164
- PEERS is the number of user-config-files that are generated when the docker container is created. You can add, remove, and rename the current PEERS options shown. I left mine as the default pc1,pc2,phone1 

NOTE: The following change is not necessary, but may be helpful

- Change the port numbers, so the VPN uses a differnet port (like port 80) that might not be blocked on Public Wi-Fi
    - Change the SERVERPORT to the port you would like to use. I used port 80.
    - Change the first port number under ports: to the same port number you changed SERVERPORT to. Look at my example .yml file that I used below for a better understanding.

Here is the docker-compose.yml file I used for my VPN installation:

![Example Two](/docs/assets/images/Picture2.png)
    
    
To save and exit the docker-compose.yml file in vim press "ESC", type :wq and then press enter :)

Run the following command to view your docker-compose.yml file to make sure it is correct:

    cat ~/wireguard/docker-compose.yml

## Step 5: Create the Wireguard VPN Server

First, make sure you are in the ~/wireguard/ directory. Run the following command to switch to that directory:

    cd ~/wireguard/
    
Next, run the following command to create/build the Wireguard VPN Docker container from the .yml file you created:

    docker-compose up -d
    
The command docker-compose up creates and starts containers and the -d option makes the container run in the background.

Once docker-compose finishes running, if the container creation was successful you will see:

![Example Three](/docs/assets/images/Picture3.png)

The Wireguard VPN server is now officially setup!!!

## Connect your phone to the Wireguard VPN

1. Run the following command:

        docker-compose logs -f wireguard
    
2. Open the Wireguard app on your phone
3. Click the "+" button
4. Choose the "Create from QR code" option 
5. Scan the QR code in the terminal for the PEER option you want to use
    - If you left the defualt PEERS=pc1,pc2,phone1 on the .yml file there will be 3 QR codes available to scan. Pick one of them to scan, I chose the phone1 option.

## Connect your computer to the Wireguard VPN

First, download/install the Wireguard application for your computer.

1. Run the following command to change directories:

        cd ~/wireguard/config
    
2. List the files/directories in the config directory

        ls
   
3. Decide which PEER option you want to use to set up the VPN on your computer. I chose peer_pc1.

4. Change directories into the peer option you choose. I will be using peer_pc1 in my example.

        cd peer_pc1
    
5. List the contents of the peer_pc1 directory

        ls
    
6. Run the following command to view the configuration file

        cat peer_pc1.conf
    
This is what my file looked like:

![Example Four](/docs/assets/images/Picture4.png)

NOTE: I deleted the droplet in Digital Ocean right after making this, so that is why I didn't redact the Private/Public key informaton.

 7. Open your Wireguard application and Add an empty tunnel...

![Example Five](/docs/assets/images/Picture5.png)

 8. Copy and paste the information from the peer_pc1.conf file into the new tunnel

Here is mine as an example:

![Example Six](/docs/assets/images/Picture6.png)

 9. Click "Save"


## Appendix

### Proof of Phone Installation

![Example Seven](/docs/assets/images/Picture7.PNG)
![Example Eight](/docs/assets/images/Picture8.PNG)
![Example Nine](/docs/assets/images/Picture9.PNG)

### Proof of Computer Installation

![Example Ten](/docs/assets/images/Picture10.png)


### Proof of using port 80 instead of port 51820

![Example Eleven](/docs/assets/images/Picture11.png)


