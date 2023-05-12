# Design Open-Source SDN-based 5G Standalone Testbed #

## Project Description ##
This project focuses on implementing Standalone (SA) mode in 5G Networks using SDN and NFV. We have utilized various open-source projects, including Open5Gs, srsRAN, UERANSIM, Opendaylight (ODL), and OpenvSwitch (OVS), to build and deploy our testbed. We wil provide a comprehensive instructions to replicate our deployed testbed. 


# Setup the SDN environment 

In this section, we provide a step-by-step guide on how to set up an SDN environment for the testbed in VMware ESXi hypervisor.
Our guide covers several essential steps, includig setting up the host networking, deploying an SDN controller, configuring SDN switches and testing the SDN environment. 




## VMware ESXi 

The ESXi deployment consists on several steps: 
1. Creating a virtual Switch 
2. Create Port Groups for the virtual switch 
3. Create and setup the Gateway VM 
4. Create and setup the Controller VM 
5. Create and setup Openvswitch VM 





### Create a virtual Switch 

There are two ways to create a virtual switch in the VMware ESXi: GUI and CLI. We provided the necessary steps in the following section. 

#### GUI 
1. In the Navigator pane on the left, click on the *Networking*  tab 
2. Click on *Virtual Switches* tab
3. To add a new switch, click on the *Add standard virtual switch* button
4. Enter a name for the new Switch in the *vSwitch Name* field. 

<p align="center">
  <img src="./images/VMware-ESXi/AddVirtualSwitch.png">
</p>





#### CLI 
1. Connect to the ESXi server using SSH. Make sure that the *ESXi SSH* is enabled on the host. 

```bash
ssh root@$ESXiIP
```

2. Use the esxcli command to create the virtual switch. For this scenario, the switch will have the default setting, but it's possible to set the MTU, Link Discovery, and Security.

```bash 
esxcli network vswitch standard add -v vSwitch
```
* **-v** &rarr; Specifies the vSwitch name.

### Create Port Groups for the created Virtual Switch 

#### GUI 
1. In the Navigator pane on the left, click on the *Networking*  tab 
2. Click on *Port Groups* tab
3. To add a new port group, click on the *Add port group* button
4. Enter the following attributes for the port group:
    1. Enter a name in the *Name* field
    2. Enter the corresponding VLAN ID in the *VLAN ID* field
    3. Select the created *Virtual Switch*
    4. Set all the *Security* policies to accept: *Promiscuous Mode*, *MAC adress changes*, and *Forged transmits*


 <p align="center">
  <img src="./images/VMware-ESXi/AddPortGroup.png">
</p>   

We are going to be creating a port group for each VM type. Use the following table as a reference to create the corresponding port groups. 

| Name             	| VLAN ID 	|
|------------------	|---------	|
| OVS-GW-1         	| 1       	|
| OVS-Controller-2 	| 2       	|
| OVS-CP-3         	| 3       	|
| OVS-UP-4         	| 4       	|
| OVS-gNB-5        	| 5       	|
| OVS-srsRAN-6     	| 6       	|
| OVS-UE-7         	| 7       	|

#### CLI 

1. Connect to the ESXi server using SSH. Make sure that the *ESXi SSH* is enabled on the host. 

```bash
ssh root@$ESXiIP
```

2. Use the esxcli command to create the port group. 


The following command allows to create new portgroups to a virtual switch in the ESXi envrionment

```bash 
esxcli network vswitch standard portgroup add --portgroup-name=OVS-GW-1 --vswitch-name=vSwitch
```

* **--portgroup-name** &#8594; Specifies the name of the new port group that you want to create
* **--vswitch-name** &#8594; Specifies the name of the vSwitch where you want to create the new port group 



The following command allows to set the VLAN ID of the recently created port group. 


 * **-p** &#8594; Specifies the name of the port group that you want to modify
* **--vlan-id** &#8594; sets the VLAN ID of the port group 

```bash 
esxcli network vswitch standard portgroup set -p OVS-GW-1 --vlan-id 1
```

The following command allows to set the security policies for a port group using the ESXi CLI. 
```bash
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-GW-1 -f true -m true -o true
```

* **--portgroup-name** &#8594; Specifies the name of the port group you want to configure.
* **-o true** &#8594; Sets the Promiscuous Mode policy which it's true in this case to accept. 
* **-m true** &#8594; Sets the MAC Address Changes policy, which in this case it's true to accept.
* **-f true** &#8594; Sets the Forged Transmits policy, which it's true in this case to accept.

We are going to be creating a port group for each time of VM. Use the following table as a reference to create the corresponding port groups. 

| Name             	| VLAN ID 	|
|------------------	|---------	|
| OVS-GW-1         	| 1       	|
| OVS-Controller-2 	| 2       	|
| OVS-CP-3         	| 3       	|
| OVS-UP-4         	| 4       	|
| OVS-gNB-5        	| 5       	|
| OVS-srsRAN-6     	| 6       	|
| OVS-UE-7         	| 7       	|

You can also use the following created commands to create all the required ports based on the table values

```bash
esxcli network vswitch standard portgroup add --portgroup-name=OVS-GW-1 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-GW-1 --vlan-id 1
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-GW-1 -f true -m true -o true
````

```bash 
esxcli network vswitch standard portgroup add --portgroup-name=OVS-Controller-2 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-Controller-2 --vlan-id 2
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-Controller-2 -f true -m true -o true
```

```bash
esxcli network vswitch standard portgroup add --portgroup-name=OVS-CP-3 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-CP-3 --vlan-id 3
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-CP-3 -f true -m true -o true
```

```bash 
esxcli network vswitch standard portgroup add --portgroup-name=OVS-UP-4 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-UP-4 --vlan-id 4
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-UP-4 -f true -m true -o true
```

```bash
esxcli network vswitch standard portgroup add --portgroup-name=OVS-gNB-5 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-gNB-5 --vlan-id 5
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-gNB-5 -f true -m true -o true
```

```bash
esxcli network vswitch standard portgroup add --portgroup-name=OVS-srsRAN-6 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-srsRAN-6 --vlan-id 6
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-srsRAN-6 -f true -m true -o true
```

```bash
esxcli network vswitch standard portgroup add --portgroup-name=OVS-UE-7 --vswitch-name=vSwitch
esxcli network vswitch standard portgroup set -p OVS-UE-7 --vlan-id 7
esxcli network vswitch standard portgroup policy security set --portgroup-name=OVS-UE-7 -f true -m true -o true
```

### Create Gateway VM


<p align="center">
  <img src="./images/VMware-ESXi/CreateVM.png">
</p>

#### GUI 
1. In the Navigator pane on the left, click on the *Virtual Machines*  tab 
2. Click on *Create/Register VM* tab
3. In the *New Virtual Machine* wizard, select the following:
    1. Select Creating Type > Create a new virtual machine
    2. Select a name and Guest OS > 
        1. *Name*: GW
        2. *Compatibility*: ESXi 7.0 U2 virtual machine 
        3. *Guest OS family*: Linux 
        4. *Guest OS version*: Ubuntu Linux (64 bit)
    3. Select the inventory location where the virtual machine should be created
    4. Select the customize settings as it shows the figure below. The ISO used is ubuntu 20.04
    5. Finish the wizard and  power on the VM 

<p align="center">
  <img src="./images/VMware-ESXi/GWsettings.png">
</p>    


The gateway has two Network Adapters. The first network adapter is coming from the virtual switch that is connected to the physical NIC of the device where Vmware ESXi has been installed. The second network adapter is coming the virtual switch that is not connected to a NIC to keep the testbed network isolated from the institution network. We will proceed to configure this Network Adapter in future steps after configuring the Ubuntu VM for the Gateway. 
    


### Setup the GW VM 

#### Modify the network configuration 

1. Select the GW virtual machine and *Open console in new window*
2. In the top right corner of the screen displayed on the console, click on the network icon
3. Click on *Settings* 
<p align="center">
  <img src="./images/VMware-ESXi/NetworkSettings.png">
</p>

4. In this case, the VM has two network connections. For the first network connection (ens160), we will leave the default settings.
5. For the second connection, we will click on the gear icon next to the connection name to access the settings for that connection and modify it based on the figure below. 
    1. *IPv4 Method*: Manual 
    2. Addresses: 
        1. *Address*: 192.168.233.1
        2. *Netmask*: 255.255.255.0
        3. *Gateway*: 
        4. *DNS*: 8.8.8.8,8.8.4.4

<p align="center">
  <img src="./images/VMware-ESXi/ModifyConnection.png">
</p>

```bash 
sudo apt update 
sudo apt install openssh-server -y 
```

Start a new ssh session with the GW VM. I'm using MobaXterm 

<p align="center">
  <img src="./images/VMware-ESXi/StartSSH.png">
</p>

<p align="center">
  <img src="./images/VMware-ESXi/SessionSettings.png">
</p>



Once the SSH session has been established, proceed to upgrade the system 

The following commands is to allow the second interface in the Gateway VM to allow traffic coming from the VMs connected to the OVS. 


```bash 
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.233.0/24 -o ens160 -j MASQUERADE
```

Also using this ssh session, it is possible to create another ssh session to access the VMs of our testbed. By using an ssh session, it allows us to have a centralized environment where we can copy and paste different commands. 






### Create the controller VM 


<p align="center">
  <img src="./images/VMware-ESXi/CreateVM.png">
</p>

#### GUI 
1. In the Navigator pane on the left, click on the *Virtual Machines*  tab 
2. Click on *Create/Register VM* tab
3. In the *New Virtual Machine* wizard, select the following:
    1. Select Creating Type > Create a new virtual machine
    2. Select a name and Guest OS > 
        1. *Name*: Controller
        2. *Compatibility*: ESXi 7.0 U2 virtual machine 
        3. *Guest OS family*: Linux 
        4. *Guest OS version*: Ubuntu Linux (64 bit)
    3. Select the inventory location where the virtual machine should be created
    4. Select the customize settings as it shows the figure below. The ISO used is ubuntu 20.04
    5. Finish the wizard and  power on the VM 
    
<p align="center">
  <img src="./images/VMware-ESXi/ControllerSettings.png">
</p>


### Setup the Controller VM 

#### Modify the network configuration 

1. Select the *Controller* virtual machine and *Open console in new window*
2. In the top right corner of the screen displayed on the console, click on the network icon
3. Click on *Settings* 
<p align="center">
  <img src="./images/VMware-ESXi/NetworkSettings.png">
</p>

4. In this case, the VM has two network connections. For the first network connection (ens160), we will leave the default settings.
5. For the second connection, we will click on the gear icon next to the connection name to access the settings for that connection and modify it based on the figure below. 
    1. *IPv4 Method*: Manual 
    2. Addresses: 
        1. *Address*: 192.168.230.1
        2. *Netmask*: 255.255.255.0
        3. *Gateway*: 
        4. *DNS*: 8.8.8.8,8.8.4.4

<p align="center">
  <img src="./images/VMware-ESXi/ControllerModifyConnection.png">
</p>

```bash 
sudo apt update 
sudo apt install openssh-server -y 
```

Start a new ssh session with the GW VM. We are using MobaXterm  

<p align="center">
  <img src="./images/VMware-ESXi/StartSSH.png">
</p>

<p align="center">
  <img src="./images/VMware-ESXi/ControllerSessionSettings.png">
</p>


The following commands is to allow the second interface in the Controller VM to allow traffic coming from the OVS. 


```bash 
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.230.0/24 -o ens160 -j MASQUERADE
```

Also using this ssh session, it is possible to create another ssh session to access the VMs of our testbed. By using an ssh session, it allows us to have a centralized environment where we can copy and paste different commands. 




    





#### Install Opendaylight 

```bash
sudo apt-get -y update
sudo apt-get -y upgrade
sudo apt-get -y install unzip
sudo apt-get -y install openjdk-8-jre
sudo update-alternatives --config java
ls -l /etc/alternatives/java
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre' >> ~/.bashrc
source ~/.bashrc
echo $JAVA_HOME
sudo apt install curl -y 
curl -XGET -O https://nexus.opendaylight.org/content/repositories/opendaylight.release/org/opendaylight/integration/karaf/0.8.4/karaf-0.8.4.zip
unzip karaf-0.8.4.zip
cd karaf-0.8.4/
./bin/karaf clean 

```

```bash 
opendaylight-user@root>feature:install odl-restconf  odl-mdsal-apidocs odl-dlux-core
```

**Connect to UI**
Connect to http://<IP>:8181/index.html#/login, using the credentials of admin/admin.




### Create a VM for the Openvswitch



<p align="center">
  <img src="./images/VMware-ESXi/CreateVM.png">
</p>


1. In the Navigator pane on the left, click on the *Virtual Machines*  tab 
2. Click on *Create/Register VM* tab
3. In the *New Virtual Machine* wizard, select the following:
    1. Select Creating Type > Create a new virtual machine
    2. Select a name and Guest OS > 
        1. *Name*: OVS
        2. *Compatibility*: ESXi 7.0 U2 virtual machine 
        3. *Guest OS family*: Linux 
        4. *Guest OS version*: Ubuntu Linux (64 bit)
    3. Select the inventory location where the virtual machine should be created
    4. Select the customize settings as it shows the figure below. The ISO used is ubuntu 20.04
    5. Finish the wizard and  power on the VM 
    
<p align="center">
  <img src="./images/VMware-ESXi/OVSsettings.png">
</p>





### Setup the OVS VM 


#### Modify the network configuration 

1. Select the *OVS* virtual machine and *Open console in new window*
2. In the top right corner of the screen displayed on the console, click on the network icon
3. Click on *Settings* 
<p align="center">
  <img src="./images/VMware-ESXi/OVSNetworkSettings.png">
</p>

4. In this case, the VM has seven network connections. Find the MAC address of the port that is connected to the SDN controller since it's the one that provides internet connection to the Openvswitch for install the necessary tools. In this case, our network connection is ens192
5. For the ens192, we will click on the gear icon next to the connection name to access the settings for that connection and modify it based on the figure below.
    1. *IPv4 Method*: Manual 
    2. Addresses: 
        1. *Address*: 192.168.230.5
        2. *Netmask*: 255.255.255.0
        3. *Gateway*:192.168.230.1 
        4. *DNS*: 8.8.8.8,8.8.4.4

<p align="center">
  <img src="./images/VMware-ESXi/OVSModifyConnection.png">
</p>


6. For the other six network connections, set them as *disable* as the *IPv4 Method*
7. After that reboot the vm to make the changes effective. 



    

```bash 
sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server -y 
```

Using the ssh session in the SDN controller session in MobaXterm, proceed to start an ssh session to access the Openvswitch and be able to setup the environment 

```bash
controller@controller:~$ ssh ovs@192.168.230.5
ovs@ovs:~$
```

**Instal Openvswitch and necessary network tools**

```bash
sudo apt install openvswitch-switch -y
sudo apt install -y net-tools iproute2 bridge-utils iputils-ping tcpdump traceroute
```

**Create and configure the bridge**

```bash
sudo ovs-vsctl add-br br0

 sudo ip addr add 192.168.233.1/24 dev br0
 sudo ip link set br0 up

sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.233.0/24 -o ens160 -j MASQUERADE
```

**Add interfaces to the switch**

```bash
sudo ovs-vsctl add-port br0 ens160
sudo ovs-vsctl add-port br0 ens161
sudo ovs-vsctl add-port br0 ens193
sudo ovs-vsctl add-port br0 ens224
sudo ovs-vsctl add-port br0 ens225
sudo ovs-vsctl add-port br0 ens256
```

**Verify that the interfaces have been added to the ovs**
```bash
sudo ovs-vsctl show 
```

**Set the external SDN controller**

```bash
sudo ovs-vsctl set-controller br0 tcp:192.168.230.1:6633
sudo ovs-vsctl set bridge br0 protocols=OpenFlow13
sudo ovs-ofctl add-flow br0 -O OpenFlow13 "table=0,priority=100,actions=normal"
sudo ovs-ofctl -O OpenFlow13 dump-flows br0
```

Reboot ovs VM after adding the ports, and proceed to add the following commands since it gets deleted after rebooting 


```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 192.168.233.0/24 -o ens160 -j MASQUERADE
```















# Open5GS - CP 

## Getting MongoDB

* Import the public key used by the package management system.

```bash
sudo apt update
sudo apt install wget gnupg -y
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

```

* Create the list file /etc/apt/sources.list.d/mongodb-org-6.0.list for your version of Ubuntu.

```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

```

* Install the MongoDB packages.

```bash
sudo apt update 
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```


## Getting Open5GS

* Ubuntu makes it easy to install Open5GS as shown below.

```bash 
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs -y
```

* Modify the config files for AMF and SMF. This is only for the control Plane

```bash
cd /etc/open5gs
```


```bash
sudo nano amf.yaml
```

```diff
amf:
    sbi:
      - addr: 127.0.0.5
        port: 7777
    ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.233.2
    metrics:
      - addr: 127.0.0.5
        port: 9090
    guami:
      - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
        amf_id:
          region: 2
          set: 1
    tai:
      - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
+        tac: 1
    plmn_support:
      - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
        s_nssai:
          - sst: 1
    security:
        integrity_order : [ NIA2, NIA1, NIA0 ]
        ciphering_order : [ NEA0, NEA1, NEA2 ]
    network_name:
        full: Open5GS
    amf_name: open5gs-amf0
```


```bash
sudo nano smf.yaml
```


```diff
smf:
    sbi:
      - addr: 127.0.0.4
        port: 7777
    pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.233.3
+      #- addr: ::1
    gtpc:
      - addr: 127.0.0.4
-      - addr: ::1
+      #- addr: ::1
    gtpu:
      - addr: 127.0.0.4
-      - addr: ::1
+      #- addr: ::1
    metrics:
      - addr: 127.0.0.4
        port: 9090
    subnet:
      - addr: 10.45.0.1/16
      - addr: 2001:db8:cafe::1/48
    dns:
      - 8.8.8.8
      - 8.8.4.4
      - 2001:4860:4860::8888
      - 2001:4860:4860::8844
    mtu: 1400
    ctf:
      enabled: auto
    freeDiameter: /home/cp/open5gs/install/etc/freeDiameter/smf.conf




upf:
    pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.233.4

```



* Restart the AMF and SMF service

```bash
sudo systemctl restart open5gs-amf.service
sudo systemctl restart open5gs-smf.service
```

* Check that the AMF and SMF service is running properly


```bash
sudo systemctl status open5gs-amf.service
sudo systemctl status open5gs-smf.service
```





## Building the WebUI of Open5GS

The WebUI allows you to interactively edit subscriber data. While it is not essential to use this, it makes things easier when you are just starting out on your Open5GS adventure. (A command line tool is available for advanced users).

Node.js is required to install the WebUI of Open5GS

Debian and Ubuntu based Linux distributions can install Node.js as follows:

```bash
sudo apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs
```

* You can now install WebUI of Open5GS.

```bash
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```



## Register Subscriber Information

Connect to http://127.0.0.1:3000 and login with admin account.

Username : admin
Password : 1423

You can change the password after login 

<p align="center">
  <img src="figures/login.png" alt="Image description">
</p>



<p align="center">
  <img src="figures/home.png" alt="Image description">
</p>


<p align="center">
  <img src="figures/createsubscriber.png" alt="Image description">
</p>

<p align="center">
  <img src="figures/createsubscriber2.png" alt="Image description">
</p>


<p align="center">
  <img src="figures/subscriberlist.png" alt="Image description">
</p>






























# Open5GS - UP 



```bash
sudo apt update and sudo apt upgrade -y 
sudo apt install openssh-server -y 
```

**Remember to make a snapshot**


## Getting Open5GS

* Ubuntu makes it easy to install Open5GS as shown below.

```bash 
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs -y
```


* Modify the config files for UPF. This is only for the User Plane

```bash
cd /etc/open5gs
```


```bash
sudo nano upf.yaml
```

```diff
upf:
    pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.233.4
    gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.233.4
    subnet:
      - addr: 10.45.0.1/16
      - addr: 2001:db8:cafe::1/48
    metrics:
      - addr: 127.0.0.7
        port: 9090
```


* Restart the UPF service

```bash
sudo systemctl restart open5gs-upfd.service
```

* Check that the upf service is running properly


```bash
sudo systemctl status open5gs-upfd.service
```

* Allow UE network traffic to access the internet. 

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```




















































































# UERANSIM - gNB

## Getting the UERANSIM

```bash
sudo apt update and sudo apt upgrade -y 
sudo apt install openssh-server -y 
sudo apt install git -y
```

* Clone repo 

```bash 
cd ~
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
```

* Install the required dependencies 

```bash 
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```

## Build UERANSIM

```bash 
cd ~/UERANSIM
make
```

## gNB Configuration

```bash 
cd config
sudo cp open5gs-gnb.yaml open5gs-gnb1.yaml
sudo nano open5gs-gnb1.yaml
```

```diff
-mcc: '999'          # Mobile Country Code value
+mcc: '001'          # Mobile Country Code value

-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)

nci: '0x000000010'  # NR Cell Identity (36-bit)
idLength: 32        # NR gNB ID length in bits [22...32]
+tac: 1              # Tracking Area Code

-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)

+linkIp: 192.168.233.5   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.233.5   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.233.5    # gNB's local IP address for N3 Interface (Usually same with local IP)


# List of AMF address information
amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.233.2
    port: 38412

# List of supported S-NSSAIs by this gNB
slices:
  - sst: 1

# Indicates whether or not SCTP stream number errors should be ignored.
ignoreStreamIds: true```


## Start using the gNB - UERANSIM 

After completing configurations and setups, now you can start using UERANSIM.
```

Run the following command to start the gNB:

```bash 
cd ..
./build/nr-gnb -c config/open5gs-gnb1.yaml
```















































# UERANSIM - UE

## Getting the UERANSIM

```bash
sudo apt update and sudo apt upgrade -y 
sudo apt install openssh-server -y 
sudo apt install git -y
```

* Clone repo 

```bash 
cd ~
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
```

* Install the required dependencies 

```bash 
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```

## Build UERANSIM

```bash 
cd ~/UERANSIM
make
```

## UE Configuration

```bash 
cd config
sudo cp open5gs-ue.yaml open5gs-ue1.yaml
sudo nano open5gs-ue1.yaml
```

```diff
# IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000001'
# Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
# Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'

# Permanent subscription key
key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
# Operator code (OP or OPC) of the UE
op: 'E8ED289DEBA952E4283B54E88E6183CA'
# This value specifies the OP type and it can be either 'OP' or 'OPC'
opType: 'OPC'
# Authentication Management Field (AMF) value
amf: '8000'
# IMEI number of the device. It is used if no SUPI is provided
imei: '356938035643803'
# IMEISV number of the device. It is used if no SUPI and IMEI is provided
imeiSv: '4370816125816151'

# List of gNB IP addresses for Radio Link Simulation
gnbSearchList:
-  - 127.0.0.1
+  - 192.168.233.5

# UAC Access Identities Configuration
uacAic:
  mps: false
  mcs: false

# UAC Access Control Class
uacAcc:
  normalClass: 0
  class11: false
  class12: false
  class13: false
  class14: false
  class15: false

# Initial PDU sessions to be established
sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1

# Configured NSSAI for this UE by HPLMN
configured-nssai:
  - sst: 1

# Default Configured NSSAI for this UE
default-nssai:
  - sst: 1
    sd: 1

# Supported integrity algorithms by this UE
integrity:
  IA1: true
  IA2: true
  IA3: true

# Supported encryption algorithms by this UE
ciphering:
  EA1: true
  EA2: true
  EA3: true

# Integrity protection maximum data rate for user plane
integrityMaxRate:
  uplink: 'full'
  downlink: 'full'

```


## Start using the UE - UERANSIM 

After completing configurations and setups, now you can start using UERANSIM.

Run the following command to start the UE:

```bash 
cd ..
sudo ./build/nr-ue -c config/open5gs-ue1.yaml
```


