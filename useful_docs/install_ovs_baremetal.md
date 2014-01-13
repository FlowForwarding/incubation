# Installing OpenVSwitch on bare metal

## Download and Install 
Debian Wheezy v7.1 or later on Intel x86_64 from http://www.debian.org/CD/http-ftp/

```bash
# mkdir -p /usr/local/src/ovs
# cd  /usr/local/src/ovs
# wget http://openvswitch.org/releases/openvswitch-2.0.0.tar.gz
# tar zxvf openvswitch-2.0.0.tar.gz 
# cd openvswitch-2.0.0
# apt-get install build-essential fakeroot
# apt-get install debhelper autoconf automake libssl-dev bzip2 openssl graphviz python-all procps python-qt4 python-zopeinterface python-twisted-conch
# dpkg-checkbuilddeps
# fakeroot debian/rules binary
# apt-get -f install
# apt-get -f install dkms
# cd /usr/local/src/ovsdpkg -i openvswitch-common_2.0.0-1_amd64.deb openvswitch-controller_2.0.0-1_amd64.deb openvswitch-datapath-dkms_2.0.0-1_all.deb openvswitch-ipsec_2.0.0-1_amd64.deb openvswitch-dbg_2.0.0-1_amd64.deb openvswitch-pki_2.0.0-1_all.deb openvswitch-switch_2.0.0-1_amd64.deb ovsdbmonitor_2.0.0-1_all.deb python-openvswitch_2.0.0-1_all.deb 
To check if packages are installed, type -  
# dpkg -l openvswitch*
```

## Verify OVS installation

```bash
# ps -ea | grep ovs

 3814 ?        00:00:00 ovs_workq
 3877 ?        00:00:04 ovsdb-server
 3888 ?        00:00:00 ovs-vswitchd
 4216 ?        00:00:00 ovs-monitor-ips
 4217 ?        00:00:18 ovs-monitor-ips
 30364 ?        00:00:00 ovs-controller
```
Check if bridges are available
```bash
# ovs-vsctl show

 34c6538a-371f-4e0a-8469-6e82ebdbd508
 ovs_version: "2.0.0"
# 
```
The above result states that we need to create bridges.  On my machine, I have 15 network interfaces.  Hence, I created a 15-port switch by creating 1 bridge and adding the 15 network interfaces to the same.
```bash
# ovs-vsctl add-br br0
# for i in {1..15}; do ovs-vsctl add-port br0 eth$i; done
```

In order to make this work as OpenFlow v1.3 Switch, the configuration command is:
```bash
# ovs-vsctl set bridge br0 protocols=OpenFlow13
```

### Configuring Controllers:
To configure 1 controller to the switch, the following command is used:
```bash
# ovs-vsctl set-controller br0 tcp:10.48.2.115:6633
```

To configure more than 1 controllers to the switch, the following command is used:
```bash
# ovs-vsctl set-controller br0 tcp:10.48.2.115:6633 tcp:127.0.0.1:6633 tcp:10.102.28.116:6633
```
To check if controllers are configured correctly, run the following command: (note that if the switch is connected to controller, then, is_connected value is noted)
```bash
# ovs-vsctl show
34c6538a-371f-4e0a-8469-6e82ebdbd508
    Bridge "br0"
        Controller "tcp:127.0.0.1:6633"
        Controller "tcp:10.48.2.115:6633"
            is_connected: true
        Controller "tcp:10.102.28.116:6633"
        Port "eth15"
            Interface "eth15"
        Port "eth2"
            Interface "eth2"
        Port "eth12"
            Interface "eth12"
        Port "eth4"
            Interface "eth4"
        Port "br0"
            Interface "br0"
                type: internal
        Port "eth7"
            Interface "eth7"
        Port "eth5"
            Interface "eth5"
        Port "eth11"
            Interface "eth11"
        Port "eth1"
            Interface "eth1"
        Port "eth13"
            Interface "eth13"
        Port "eth6"
            Interface "eth6"
        Port "eth14"
            Interface "eth14"
        Port "eth3"
            Interface "eth3"
        Port "eth9"
            Interface "eth9"
        Port "eth8"
            Interface "eth8"
        Port "eth10"
            Interface "eth10"
    ovs_version: "2.0.0"
```



## Running OpenVSwitch
OpenVSwitch running config is in /var/run/openvswitch and /etc/openvswitch directories.
```bash
# ps -ea | grep ovs
 3814 ?        00:00:00 ovs_workq
 3877 ?        00:00:04 ovsdb-server
 3888 ?        00:00:00 ovs-vswitchd
 4216 ?        00:00:00 ovs-monitor-ips
 4217 ?        00:00:18 ovs-monitor-ips
 30364 ?        00:00:00 ovs-controller
```

## Working with Flows
OVS enables you to work with the Switch and add/view/delete flows.  Here are some commands for the same:
Dump flows on the Switch
```bash
# ovs-ofctl dump-flows br0
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=5.479s, table=0, n_packets=0, n_bytes=0, idle_age=5, in_port=2 actions=output:3
```

Add Flows (here 2 flows are added - anything on port 2, copy it to ports 1 & 3; anything on port 1, copy it to ports 2 & 3)
```bash
# ovs-ofctl add-flow br0 "in_port=2,actions=output:1,3"
# ovs-ofctl add-flow br0 "in_port=1,actions=output:2,3"

# ovs-ofctl dump-flows br0
 NXST_FLOW reply (xid=0x4):
  cookie=0x0, duration=3.632s, table=0, n_packets=0, n_bytes=0, idle_age=3, in_port=1 actions=output:2,output:3
  cookie=0x0, duration=14.343s, table=0, n_packets=0, n_bytes=0, idle_age=14, in_port=2 actions=output:1,output:3
# 
```
