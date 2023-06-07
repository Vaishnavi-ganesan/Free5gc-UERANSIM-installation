# Getting the UERANSIM
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
