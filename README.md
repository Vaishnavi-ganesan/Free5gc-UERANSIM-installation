### Free5gc and Ueransim installation
Two vms is used    
1- Free5gc vm add static ip in vagrantfile 192.168.60.187   
2- UERANSIM add static ip in vagrantfile 192.168.60.185
### Getting the UERANSIM
clone the repository: 
```
cd ~     
git clone https://github.com/aligungr/UERANSIM
```
Firstly it's better to update your apt repositories and upgrade the programs.
```
sudo apt update
sudo apt upgrade
```
Then here's the list of dependencies: (Built-in dependencies shipped with Ubuntu are not listed herein.) 
```
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```
```
cd ~/UERANSIM   
make
```
And that's it. After successfully compiling the project, output binaries will be copied to `~/UERANSIM/build` folder. 
### Getting the Free5gc
1. Check OS Versions  
First issue `uname -r`to see the Linux kernel version.The version looks like `5.4.x.`
Please make sure your kernel version is `5.0.0-23-generic `or newer than `5.4.0.`   
2. Install Basic Tools  
First make sure Golang (go) is not installed   
`go version`    
To install latest go,   
```
cd ~     
wget https://golang.org/dl/go1.15.8.linux-amd64.tar.gz  
sudo tar -C /usr/local -xzf go1.15.8.linux-amd64.tar.gz
```  
Then execute the following commands  
```
mkdir -p ~/go/{bin,pkg,src} 
echo 'export GOPATH=$HOME/go' >> ~/.bashrc          
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc 
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
echo 'export GO111MODULE=auto' >> ~/.bashrc  
source ~/.bashrc
```
And check if go is installed successfully.
`go version`  
Next, we will install MongoDb. Copy and paste:  
```
sudo apt -y update
sudo apt -y install mongodb
sudo systemctl start mongodb  
```
You can check if MongoDb is installed by `mongo`   
If you can enter MongoDB’s command shell, it is installed successfully. You can exit the command shell by enter exit, or just type Ctrl-D.
Next, let’s install other development tools. Copy and paste:     
```
sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
go get -u github.com/sirupsen/logrus
```
3. Setting up Networking
```
sudo sysctl -w net.ipv4.ip_forward=1     
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE   
sudo systemctl stop ufw     
```
Note: The name `enp0s8`is the network interface free5GC use to connect to Data Network (i.e. Internet). We already know how to get it by using `ifconfig `and `route`commands. You should change it to the name corresponding to the interface name that can reach internet.

Also note that these network settings will disappear after reboot. So make sure you run the above commands after each reboot. 
4. Install free5GC Core Network
```
cd ~  
git clone --recursive https://github.com/free5gc/free5gc.git
```
To build free5GC, do:  
```
cd ~/free5gc  
make
```
We also need to install kernel module gtp5g:     
```
cd ~ 
git clone https://github.com/free5gc/gtp5g.git  
cd gtp5g
make    
sudo make install
```
To check if gtp5g is installed successfully, see if the following command shows some information:   
`lsmod | grep gtp`    
 5. Testing free5GC
free5GC provides some testing procedures to make sure it works properly.   
```
cd ~/free5gc 
./test.sh TestRegistration
```
If everything runs properly without “red” error messages, and the word “PASS” appears near the end of the screen output, then free5GC is running properly.   

### Install Free5Gc webconsole
First SSH into free5gc
```
sudo apt remove cmdtest
sudo apt remove yarn
```
Then install Node.js and Yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install -y nodejs yarn
```
To build WebConsole
```
cd ~/free5gc
make webconsole
```
6.Use WebConsole to Add an UE
First start up the WebConsole server:
```
cd ~/free5gc/webconsole
go run server.go
```
The screen shows the port number :5000 at the end. Open your web browser from your host machine, and enter the URL http://172.16.9.8:5000 (here port forwarding is done)
On the login page, enter username admin and password free5gc
Once logged in, widen the page until you see “Subscribers” on the left-hand side column.
Choose Subscribers and create a new data:
Note that other than the “Operator Code Type” field which you should choose “OP” for now, leave other fields unchanged. This registration data is used for ease of testing and actual use later.
After the data is created successfully, you can press Ctrl-C on the terminal to quit WebConsole
7. Setting free5GC and UERANSIM Parameters  
In free5gc VM, we need to edit three files:  
~/free5gc/config/amfcfg.yaml   
~/free5gc/config/smfcfg.yaml   
~/free5gc/config/upfcfg.yaml
```
cd ~/free5gc
nano config/amfcfg.yaml
```
Replace ngapIpList IP from 127.0.0.1 to 192.168.60.187        
In,`nano config/smfcfg.yaml`
and in the entry inside userplane_information / up_nodes / UPF / interfaces / endpoints, change the IP from 127.0.0.8 to 192.168.60.187     
Finally, edit ~/free5gc/config/upfcfg.yaml，and change gtpu IP from 127.0.0.8 into 192.168.60.187     
Setting UERANSIM
In the ueransim VM, there are two files related to free5GC：  
~/UERANSIM/config/free5gc-gnb.yaml  
~/UERANSIM/config/free5gc-ue.yamll

The second file is for UE, which we don’t have to change if the data inside is consistent with the (default) registration data we set using WebConsole previously.

First SSH into ueransim, and edit the file ~/UERANSIM/config/free5gc-gnb.yaml, and change the ngapIp IP, as well as the gtpIp IP, from 127.0.0.1 to 192.168.60.185，and also change the IP in amfConfigs into 192.168.60.187, that is, from

Next we examine the file ~/UERANSIM/config/free5gc-ue.yaml，and see if the settings is consistent with those in free5GC (via WebConsole),
7. Testing UERANSIM against free5GC
SSH into free5gc. If you have rebooted free5gc, remember to do:
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```
Also, make sure you have make proper changes to the free5GC configuration files, then run ./run.sh
```
cd ~/free5gc
./run.sh
```
At this time free5GC has been started.

Next, prepare three additional SSH terminals from your host machine (if you know how to use tmux, you can use just one).

In terminal 1: SSH into ueransim, make sure UERANSIM is built, and configuration files have been changed correctly, then execute nr-gnb
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
In terminal 2, SSH into ueransim, and execute nr-ue with admin right:
```
cd ~/UERANSIM
sudo build/nr-ue -c config/free5gc-ue.yaml
```
In terminal 3, SSH into ueransim, and ping 192.168.56.101 to see free5gc is alive. Then, use ifconfig to see if the tunnel uesimtun0 has been created (by nr-ue):
Now use ping:
`ping -I uesimtun0 google.com`
If ping gets replies, then free5GC is running properly. Congratulations!


