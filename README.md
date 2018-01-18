![openstack-logo-1-300x150](https://user-images.githubusercontent.com/13846660/34405179-33a09ef4-ebf5-11e7-8530-9d60a26683c8.png)
<!-- ![cloudshell_logo](https://user-images.githubusercontent.com/13846660/34405299-ec92e066-ebf5-11e7-86b2-0aa943395a14.png) -->

# Openstack_dev
Openstack intergration with Quali Cloudshell

# DevStack Installation

1) install

> install ubuntu 16.04 LTS <br>
> install virtualbox extension >> insert guest addition CD image <br>
> Virtualbox >> Devices >> Shared .. >> bidirectional <br>

2) add user named "devstack"
```sh
sudo adduser devstack
cat /etc/passwd | grep devstack
su -devstack
pwd
exit
```

3) add sudo privilege to user named "devstack", if you do not this, the openstack service could not start.
```sh
ls -l /etc/sudoers
sudo visudo -f /etc/sudoers
devstack ALL=(ALL) NOPASSWD: ALL => add this line end of file
```

4) set NIC on the virtualbox
> Setting for 2 NIC <br>
> Virtualbox >> settings >> network <br>
>  * adpater 1: NAT <br>
> &nbsp;&nbsp;&nbsp;&nbsp;Port Forwarding: SSH TCP 192.168.X.X 22 192.168.Y.Y 22 <br>
>  * adapter 2: Bridged Adapter <br>
> &nbsp;&nbsp;&nbsp;&nbsp;Promiscuous Mode: Allow All <br><br>
NAT(Network Address Translation): vRouter, VM ↔ Internet, different subnet, Port forwarding OK <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Bridged: vSwitch, same subnet, VM has it's own IP

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strong>or</strong> 

> Setting for 1 NIC <br>
> virtualbox >> settings >> network <br>
> * adpater 1: NAT <br>
> &nbsp;&nbsp;&nbsp;Port Forwarding: SSH TCP  2022  22 <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dashboard TCP  2080  80 <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console TCP  6080 6080<br>

5) update ubuntu
```sh
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

6) ssh & git
```sh
sudo apt-get install openssh-server git -y
```

7) change root pass
> Press ESC durling Boot <br>
> Press e on ubuntu <br>
```sh
lash $vt_handoff 1 => add num 1 end of lash $vt_handoff
```
> Ctrl + x => save <br>
> terminal>LANG=C passwd root <br>

8) clone from git
```sh
git clone https://git.openstack.org/openstack-dev/devstack
```

9) change privilege & check info
```sh
sudo chown -R devstack:devstack /path/devstack
sudo chmod 770 /path/devstack
ifconfig enp0s8 => checking host ip
ex) terminal>..inet addr: 192.168.0.168
```

10) make local.conf
```sh
cd devstack
vi local.conf
```

11) setting of local.conf

- Most simplest(=> Setting for 1 NIC) 

```sh
[[local|localrc]]
ALL_PASSWORD=a
ADMIN_PASSWORD=$ALL_PASSWORD
DATABASE_PASSWORD=$ALL_PASSWORD
RABBIT_PASSWORD=$ALL_PASSWORD
SERVICE_PASSWORD=$ALL_PASSWORD
HOST_IP=10.0.2.15
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strong>or</strong> 

- Setting for 2 NIC 

![2nic_setting_diagram](https://user-images.githubusercontent.com/13846660/35082412-082a2e2a-fc5d-11e7-9deb-5b02ea6c1680.png)

```sh
[[local|localrc]]
# NIC information of Ubuntu on the Virtualbox
# devstack@devstack-VirtualBox:~$ ifconfig
# enp0s3    Link encap:Ethernet  HWaddr 08:00:27:2f:07:ee  
#           inet addr:192.168.0.168  Bcast:192.168.0.255  Mask:255.255.255.0
# enp0s8    Link encap:Ethernet  HWaddr 08:00:27:a2:d6:dd  
#          inet addr:10.0.3.15  Bcast:10.0.3.255  Mask:255.255.255.0
# lo        Link encap:Local Loopback  
#          inet addr:127.0.0.1  Mask:255.0.0.0

# Host pc subnet, Bridged Interface
HOST_IP=192.168.0.168 

# NAT Interface
FLOATING_RANGE=10.0.3.0/24

#Internal Network range in the VM 
FIXED_RANGE=192.168.1.0/24

#Subnet mask 24 equals 256
FIXED_NETWORK_SIZE=256

# NAT Interface
FLAT_INTERFACE=enp0s8 
```

- Turnoff the nova (Optional) <br>

https://wiki.openstack.org/wiki/NeutronDevstack <br>
 neutron //support complex level network management <br>
 nova //simple network management <br>

```sh
[[local|localrc]]
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta

# Optional, to enable tempest configuration as part of devstack
enable_service tempest

# For Tempest
API_RATE_LIMIT=False 
```

12) run shell script
```sh
./stack.sh > setup_log_stack.log
```

13) login dashboard
```sh
http://192.168.0.168/dashboard
```
or
```sh
http://127.0.0.1/dashboard
```

14) setting for CLI command (openstack compute (nova) “error”)
> download openrc file from WEB UI

```sh
cd devstack
source [project_name]-openrc.sh [user_id]
ex) >>source alt-demo-openrc.sh admin
```

15) *** ***Don't reboot devstack*** ***
> After every reboot you need to run ./stack.sh.

# complete message
```sh
=========================
DevStack Component Timing
 (times are in seconds)  
=========================
run_process          156
test_with_retry       13
apt-get-update        57
pip_install          937
osc                  691
wait_for_service     153
git_timed            324
dbsync               145
apt-get              310
-------------------------
Unaccounted time     2105
=========================
Total runtime        4891

This is your host IP address: 192.168.0.168
This is your host IPv6 address: ::1
Horizon is now available at http://192.168.0.168/dashboard
Keystone is serving at http://192.168.0.168/identity/
The default users are: admin and demo
The password: a

WARNING: 
Using lib/neutron-legacy is deprecated, and it will be removed in the future

Services are running under systemd unit files.
For more information see: 
https://docs.openstack.org/devstack/latest/systemd.html

DevStack Version: queens
Change: c5c7d8f37eff14f2943c88cbce3c835b14237507 Merge "Switch to consolidated fetch-subunit-output role" 2018-01-17 20:31:33 +0000
OS Version: Ubuntu 16.04 xenial
```

# this will remove the installation of DevStack and dependancies

```sh
./clean.sh 
rm -rf /opt/stack
rm -rf /usr/local/bin 
```

# Tip

1) change mode in Ubuntu
> ctrl + alt + f1 : cmd mode <br>
> ctrl + alt + f7 : gui mode

2) setting for fixed ip
```sh
sudo vi /etc/network/interfaces
```

- The primary network interface
```sh
auto enp0s3
iface enp0s3 inet static
  address 192.168.0.19
  netmask 255.255.255.0
  gateway 192.168.0.1
  dns-nameservers 8.8.8.8
```
- restart NIC
```sh
sudo ip addr flush dev enp0s3
sudo ifdown enp0s3
sudo ifup enp0s3
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strong>or</strong> 
```sh
sudo systemctl restart networking
```
3) Vi Editor
- input mode
```sh
i on cursor
a after cursor
```
- delete
```sh
delete one char
x
-delete line
dd
```
- arrow key
```sh
h
j
k
l
```

4) basic command
```sh
ll
ls -l
rm -r mydir
rm -f sample.txt //確認なしで削除する場合。
mv /home/user/oldname /home/user/newname
cp -rp /home/user/oldname /home/user/newname
  => r: recursive / p: keep properties
ifconfig
cd 
pwd
chmod 777 mydir
mkdir myfolder
df -h  => disk usages

# remove repository
sudo add-apt-repository -r ppa:<ppa to remove>

# recursive mkdir : 
mkdir -p /opt/stack/logs
```

5) network command
```sh
nmcli dev status
nmcli dev show enp0s8 => check for gateway
ifconfig
```

6) find as a file name
```sh
find . -name "foo*"
```
7) scroll in cli
```sh
ls -l | more
```

8) Q: Virtualbox shared folder permissions? <br>
A: Try this (on the guest machine. i.e. the OS running in the Virtual box): <br>
```sh
 sudo adduser your-user vboxsf
```
Now reboot the OS running in the virtual box.

9) Lightweight Browser Midori <br>
Open terminal by pressing Ctrl+Alt+T and run the following commands, <br>
```sh
sudo apt-add-repository ppa:midori/ppa
sudo apt-get update
sudo apt-get install midori
```
or
```sh
sudo apt-get -f install
sudo dpkg -i midori-xxx.deb
```
or launch midori-xxx.deb on the gui


# Sample Topology Configuration in Openstack
```sh
1) add 2 private network
 Network >> Create subnet >> Subnet Name: private1_subnet >> Network Address Source :: Enter Network Address manually >> Network Address :: 10.0.0.0/24 >> IP Version :: IPv4 >> Enable DHCP
 Network >> Create subnet >> Subnet Name: private2_subnet >> Network Address Source :: Enter Network Address manually >> Network Address :: 10.0.1.0/24 >> IP Version :: IPv4 >> Enable DHCP

2) add 2 router
 Network >> Create rouer >> Router Name: router1 >> External Network: public
 Network >> Create rouer >> Router Name: router2 >> External Network: public

3) add security group
 Network >> Security Groups >> default >> Manage Rules >> Rule : All ICMP 
 Network >> Security Groups >> Add Rule >> Rule : SSH

4) Set a Floating IP Pool
 Login to admin account
 Network >> Floating IPs >> Allocate IP to Project >> Pool : public >> Allocate IP 

5) Create Instance & Allocate Floating IP to Instance
 Compute >> Instances >> Launch Instance >> Instance Name: demo1 / cirros-0.3.5-x86_64-disk / m1.tiny / private_1 or private_2 >> Actions : Associate Floating IP

6) Connect Instance from SSH
 For Japanese Layout Keyboard, Need to change Keyboard Layout to EN
  1)Windows Control Panel >> Add Keyboard >> English(US) 
  2)Compute >> Instance >> select demo1 >> Console
  3)demo1 login: cirros / Password: cubswin:) 
  4)Change Keyboard Layout to EN => For Input : Shift + ; / For Input ) Shift + 0  
```


# Intergration with Quali CloudShell
[Openstack Guide][df1] Add OpenStack Cloud Provider Resource

First Register Cloud controller and then Add App which wants to add.

1) Portal >> Inventory >> Add New >> Select Shell >> Openstack
```sh
Controller URL: http://192.168.0.157/identity/v3 
OpenStack Domain Name: default
OpenStack Project Name: demo
OpenStack Management Network ID:(public's subnet ID): 4ebe5af2-923b-47be-9645-3b93b54438d2	
OpenStack Reserved Networks : skip setting
VLAN Type: VXLAN
 => https://ask.openstack.org/en/question/51388/whats-the-difference-between-flat-gre-and-vlan-neutron-network-types/
Floating IP Subnet ID (Subnet ID which want to spawn(deploy), maybe public or private's subnet id): 76df00fa-96a6-45d9-8f2a-1dcf14378667
```

2) Managing Apps (Add Apps), maybe public or private's subnet id
```sh
 http://help.quali.com/Online%20Help/8.1.0.4291/Rm/Content/CSP/MNG/Mng-Apps.htm#Adding

 Portal >> Manage >> Apps >> Add >> Openstack Deploy From Glance Image >> Create

 DEPLOYMENT
  CLOUD PROVIDER: select one from drop-down list
  IMAGE ID: Select one from Openstack dashboard >> Project >> compute >> Images
  INSTANCE FLAVOR: m1.tiny 
  ADD FLOATING IP: True or False => I choose False. 
  FLOATING IP SUBNET ID: Subnet ID which is ip pool of public ip
```
- The Meaning of Floating IP in the Cloudshell is not only Netwrok::Floating IPs in the Openstack, it means SUBNET in the Openstack.

# Trouble Shooting
```sh
1) Set a Enviroment variable
> Download RC file from the Dashboard, And Run a [project_name]-openrc.sh

2) Permission denied on the Root privilege
 bash -x demo-openrc.sh //this command is not working

3) Missing value auth-url required for auth plugin password
 source demo-openrc.sh
 
# Setup endpoint URL (not necessary)

cat /etc/keystone/keystone.conf
- admin_endpoint = http://192.168.0.157/identity
```

# the difference between NAT / Bridged / Host-Only networking?
```sh
=> https://superuser.com/questions/227505/what-is-the-difference-between-nat-bridged-host-only-networking
   Host-Only: The VM will be assigned one IP, but it's only accessible by the box VM is running on. No other computers can access it.
   NAT: Just like your home network with a wireless router, the VM will be assigned in a separate subnet, like 192.168.6.1 is your host computer, and VM is 192.168.6.3, then your VM can access outside network like your host, but no outside access to your VM directly, it's protected.
   Bridged: Your VM will be in the same network as your host, if your host IP is 172.16.120.45 then your VM will be like 172.16.120.50. It can be accessed by all computers in your host network.

=> https://serverfault.com/questions/490043/differences-between-bridged-and-nat-networking
   Bridged connections are just that, essentially a virtual switch is connected between the VM and your physical network connection.
   NAT'd connections are also just that, instead of a switch a NAT router is between the VM and your physical network connection.  
```

# What IP address starts with 10?
```sh
=> The Internet Assigned Numbers Authority (IANA) has reserved thefollowing three blocks of the IP address space for private internets:

10.0.0.0 - 10.255.255.255 (10/8 prefix)
172.16.0.0 - 172.31.255.255 (172.16/12 prefix)
192.168.0.0 - 192.168.255.255 (192.168/16 prefix)
```

[df1]: <http://help.quali.com/Online%20Help/8.1.0.4291/Rm/Content/Admn/OpenStack-Cld-Prvdr-Rsc.htm>
