# rdo-player
Collection of Ansible roles which deploy RDO Manager on a virtual environment and provide connectivity to the isolated networks through OpenVPN tunnel.

These are reflecting the steps in the Liberty docs at:
https://repos.fedorapeople.org/repos/openstack-m/rdo-manager-docs/liberty/

Prerequisites:
1. Key based SSH access as root user to the physical machine which will host the virtual environment
2. Ansible

How to:

1. Edit group_vars/all/vars.yml:

vpn:
  remote: yes           # install OpenVPN server on the physical host
  local: no             # install OpenVPN client on local machine; it's recommended to set this to no as I've only tested it on Fedora with passwordless sudo; 
                          the certificates and configuration file will be placed in the vpn-certs directory so you can manually install the client 

instack:
 destroy: no            # when set to yes it removes any existing instack and baremetalbrbm VMs from the physical host
 undercloud_node:
   vcpus: 8             # number of vCPUs assigned to the instack VM (undercloud)
   memory: 16384        # memory for the instack VM
 overcloud_nodes: 
   number: 15           # number of baremetalbrbm VMs (overcloud nodes)
   vcpus: 4             # number of vCPUs assigned to each of the overcloud nodes
   memory: 16384        # memory for each of the overcloud nodes

undercloud:
  install: yes          # installs undercloud 

overcloud:
  install: yes          # prepares overcloud deployment: nodes registration, introspection, images upload
  images_build: no      # builds images or downloads the prebuilt ones
  images_location: "https://repos.fedorapeople.org/repos/openstack-m/rdo-images-centos-liberty/"

2. Edit hosts file with the IP address or the name of the physical host that will host the virtual environment

3. Run the deployment:
ansible-playbook -i hosts tripleo.yml 

What next:

The undercloud node can be accessed via the physical host IP address on port 2900, user stack, password stack:
ssh stack@{PHYSICAL_HOST_IP_OR_NAME} -p 2900

After running a VPN deployment you should be able to reach the following networks:
ctlplane: 192.0.2.0/24      # provisioning network; undercloud node is at 192.0.2.1
external: 172.16.23.0/24    # external network

Files in /home/stack:

deploy.commands                      : examples of overcloud deploy commands
templates/my-overcloud               : copy of the default heat templates in /usr/share/openstack-tripleo-heat-templates
templates/network-environment.yaml   : example of environment file to be used in the virt env with single-nic-vlans templates to deploy isolated networks
templates/nic-configs                : copy of the single-nic-vlans templates in /usr/share/openstack-tripleo-heat-templates/network/config/single-nic-vlans
templates/firstboot-environment.yaml : example of a first boot script that sets up asymmetric routing on the controllers 
templates/ceph.yaml                  : example of environment file to activate ceph as storage backend
templates/snmpd.yaml                 : example of post deploy script that adds a snmp user on the overcloud nodes
post.deploy                          : commands for setting up networks, router and upload image to a deployed overcloud
