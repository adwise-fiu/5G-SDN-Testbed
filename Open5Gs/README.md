# Open5Gs #

The following commands are to install Open5Gs project. We are using the snapshot version which is the stable version. 

```bash
 sudo apt update
 sudo apt install wget gnupg -y
 wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
 sudo apt-get install -y mongodb-org
 sudo systemctl start mongod 
 sudo systemctl enable mongod 

sudo add-apt-repository ppa:open5gs/snapshot
sudo apt update
 sudo apt install open5gs -y

 sudo apt update
 sudo apt install curl
 curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
 sudo apt install nodejs

 curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -

sudo apt install mongodb-clients -y

 curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -


```
