# UERANSIM #

The following commands are to download and build the UERANSIM project 

``` bash

sudo apt update
sudo apt upgrade

sudo apt install make
sudo apt install gcc -y
sudo apt install g++ -y
sudo apt install libsctp-dev lksctp-tools -y
sudo apt install iproute2
sudo snap install cmake --classic

cd ~
sudo apt install git -y
git clone https://github.com/aligungr/UERANSIM


cd ~/UERANSIM
make

```
