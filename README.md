# Openshift
A complete Linux Redhat 7 Openshift


First In order to get ahead of Openshift, First of all we has to set up a KVM for master, and also some Nodes. And it has to be done in the Linux environment Redhat7. ISO Image.

Login as a Root User and also Install the fallowing command for, Also create an Account in Redhat.


                    subscription-manager register (Enter username and passwd)

In order to attach Entitilments to attachneeded:

    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A18 "Red Hat Enterprise Linux Server Entry Level, Self-support" \ | grep "Pool ID" | awk '{print$3}')


In order to Redhat openshift attach:

    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A32 "Red Hat OpenShift Container Platform" \ | grep "Pool ID" | awk '{print$3}')


Note: We can play with 10 licences

we have to assign the Redhat Enterprise openstack autonomus for the server, that was specificly openB switch, for those who are launching on redhat Enterprise as linux or community versions
If you don't have access to the RHOHP licencse, we can just enable Apple Repository, Extra packages for Enterprise linux from there we can find openb switch. 

If you have openshift licence available, you can go ahead and use it below command

    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A29 "Red Hat OpenStack Platform" \ |grep -B3 "Unlimited" \ | grep "Pool ID" | awk '{print$3}')


First thing we have to do disable the current repos:

    subscription-manager repos --disable=*

Enable our Repository's

    subscription-manager repos --enable=rhel-7-server-rpms \ --enable=rhel-7-server-optional-rpms \ --enable=rhel-7-server-extras-rpms \ --enable=rhel-7-server-ose-3.5-rpms \ --enable=rhel-7-server-openstack-10-rpms

    yum repolist (confirm with yum repo list)

    yum -y update

    systemctl reboot

    logout & login again
    
Note : Now our Master Node is Configured.

======================================================================================================================================


KVM Setup for Node1:

Setup a 4 Gb, 4096 MB RAM, CPU 2, storage from EXTK3DK available, Give server name : Node1, Virtual network:


Configuring the openshift node: 40 GB for Virtual Block storage, Any Network DNS name, give the Manual IP, Launch the Installation 

Add a Block storage Volume for docker storage, 40 GB for Docker Storage. 

Login to the terminal window:

      fdisk -l

Check that block storage volume is attached to the volume.

Setup 

    Subscription-manager register ( Give credentials and login)


In order to attach Entitilments to attachneeded:

    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A18 "Red Hat Enterprise Linux Server Entry Level, Self-support" \ | grep "Pool ID" | awk '{print$3}')


In order to Redhat openshift attach:

    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A32 "Red Hat OpenShift Container Platform" \ | grep "Pool ID" | awk '{print$3}')

If you have openshift licence available, you can go ahead and use it below command


    subscription-manager attach --pool=$(subscription-manager list --available \ | grep -A29 "Red Hat OpenStack Platform" \ |grep -B3 "Unlimited" \ | grep "Pool ID" | awk '{print$3}')


First thing we have to do disable the current repos:

    subscription-manager repos --disable=*

Enable our Repository's

    subscription-manager repos --enable=rhel-7-server-rpms \ --enable=rhel-7-server-optional-rpms \ --enable=rhel-7-server-extras-rpms \ --enable=rhel-7-server-ose-3.5-rpms \ --enable=rhel-7-server-openstack-10-rpms

    yum repolist (confirm with yum repo list)

    yum -y update

    systemctl reboot

    logout & login


 Note : The configuration is same as the Master.
 
 =============================================================================================================================
 
Configure DNS Resolution with DNSMASQ MASTER & NODES

    yum -y install dnsmasq bind-utils

hostname

    cp /etc/dnsmasq.conf /etc/dnsmasq.conf.orig

ADD the custom address for OCP lab

    vim /etc/dnsmasq.conf

    # address=/double-click.net/127.0.0.1

    address=/ocp.<hostname>/192.168.10.162  

    # Setup the custom resolve file

    resolv-file=/etc/resolv.dnsmasq

Save the file and that's it DNS Mask is updated.

Next is add to 

    vim /etc/resolv.dnsmasq 

    # ADD THE NAMESERVER AS OUR GATEWAY IP FOR OUR SYSTEM.
    nameserver 192.168.10.1


Edit the resolv.conf file by going to

    vim /etc/resolv.conf

    # Generated by NetworkManager
    search <hostname>

    # nameserver 192.168.10.1

    # ADD THE BELOW LINE OF NAMESERVER

    nameserver 127.0.0.1


Stop & Disable the service of firewalld & Enable dnsmasq

    systemctl stop firewalld && systemctl disable firewalld

    systemctl enable dnsmasq && systemctl start dnsmasq


ADD The custom IP Address to our host file as fallows

    vim /etc/hosts

    <IPADDRESS> <MASTER> <HOSTNAME>
    <IPADDRESS> <NODE1> <HOSTNAME>

Test our DNS Resolution 

    host $(hostname) -----> It should return the IPADDress that you configured.

Check Internally

    ping -c 1 test.ocp.<hostname> --> It should ping to the IPADDress to Master

Check Externally

    ping -c1 google.com 
    
    
==============================================================================================================================

Configuring Docker For OpenShift Container Platform for Both Master & Nodes

In order to connect Master server to Node server we has to connect via "ssh" via in a single terminal 

Note: Login from Terminal to Master first via ssh root@MASTERIP
 
Below are the steps to connect from master to Node

    ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
    ll .ssh/
    ssh-copy-id root@NodeIP
    shh root@NODEIP --> with password we can login to Node


The same thing that we has to do in the Node also, Because Node has to connect to Master

Login form Terminal via ssh root@NODEIP

    ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
    ll .ssh/
    ssh-copy-id root@MASTERIP
    ssh root@MASTERIP --> with password we can connect to Master from Node

===========================================================================================

Setup Docker in both MASTER & NODES ALSO: 

    systemctl status firewall (Make sure it has to be disabled)

Install Specific version of docker(1.12.6) only for docker to communicate with openshift:

    yum -y install docker-1.12.6

    cp /etc/sysconfig/docker-storage-setup /etc/sysconfig/docker-storage-setup.orig

    vim /etc/sysconfig/docker-storage-setup

Remove everything in this file (Don't bother remove everything) and add only the below lines

    DEVS=vdb
    VG=docker-vg 

Save it

    lvmconf --disable-cluster 

Run the Docker storage setup

    docker-storage-setup  --> docker-pool will be create & docker-vg/docker-pool changed
    lvs --> can see the docker-pool, Volume Group, LSize etc

    cat /etc/sysconfig/docker-storage
    
    systemctl enable docker
    systemctl start docker 

Note: Do the same setup for Nodes also.





     
