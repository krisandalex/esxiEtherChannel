# from here https://serverfault.com/questions/1093203/what-is-wrong-with-esxi-vmkernel-port-in-active-active-configuration

The management vmk in ESXI assumes the MAC address of the Nic in the first PCI slot during the initial set-up. This is how it has worked forever. This can break things only when the physical device also starts sending packets. This normally does not happen, physical Nics do not send traffic, they pass traffic along. This behavior also needs to be paid attention to if you decide to move physical Nics from one host to another, this brings down 2 host connections when the physical switch freaks out. My guess is that this Nic started reporting CDP/LLDP traffic and this is when your switch sees the MAC duplication. The easiest solution is to rebuild the vmk through the command line. This will need to be done from a direct console access (DCUI) (KVM, ILO, IDRAC, etc...).

Here are the commands; (Adjust the IP's/subnet mask/portgroup name etc... to fit your needs.)

esxcli network ip interface remove --interface-name=vmk0

esxcli network vswitch standard portgroup add -p Management_Network -v vSwitch0

esxcli network ip interface add --interface-name=vmk0 --portgroup-name=Management_Network

esxcli network vswitch standard portgroup set -p Management_Network --vlan-id 50

esxcli network ip interface ipv4 set --interface-name=vmk0 --ipv4=192.168.50.116 --netmask=255.255.255.0 --gateway=192.168.50.1 --type=static

esxcli network ip interface tag add -i vmk0 -t Management
