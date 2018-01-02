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

==============================================================================================================================

Role Bindings:

Set the role-binding in which ever the project you are currently in. 
Grant relevant access to user or group
Pre-project basis(-n)
per-cluster basis
Applied for all namespaces

Default Roles:

--> Admin
--> basic-user
--> cluster-admin
-->  edit
--> self-provisioner
--> view

oc adm policy who-can action [action] [resource]
oc adm policy add-role-to-uesr [role] [username]
oc adm policy remove-role-from-user [role][username]
oc adm ploicy remove-user[username]
oc adm is also used for per-cluster basis.
oc adm policy add-cluster-role-to-user[role][user]
oc get clusterroles
oc get /oc describe

Describes the information of roles, policy, clusterroles, clusterpolicy, clusterrolebindings, scc

======================================================================

Managing Users and Projects in Openshift ( Four users, and for projects, and set user privalages for roles for that project )

Note: Login to the Master Node via ssh root@MASTERIP 

    oc whoami

Create users know

    htpasswd -b /etc/origin/openshift-passwd law openshift --> An user law is created

    for i in shen jack chi
    d
    htpasswd -b /etc/origin/openshift-passwd $i openshift
    done                              

Note: The remaining three users will be created at a time( shen, jack, chi )

    systemctl restart atomic-openshift-master

Open New Terminal and check that the users that you have created  has access to the master account by using the below command

    oc login https://master.<hostname>:8443 -u law
    
    oc whoami --> And check that your are logged in with the user law

Note: Repeat the same proccess with the other three of them shen, jack, chi

Exit from the terminal

=========================================

Note: you should be master Now, in the root directory

Now we are going to add the label for the users that you created.

    oc label user law org=PorkChopExpress ---> Now you labeled for the user law

Do the same for all the other three users like as fallows:

    for i in chi shen jack
    do
    oc label user $i org=PorkChopExpress
    done

For the above four command the labels are assigned for all the four of the users

In order to check use the below command

    oc describe user chi  --> check that your user chi has a label 

==================================

To create a Project in the openshift cluster:

Login to the master Node as root
 
    oc new-project dev --description="PorkChop Express engineering team"
    oc describe project dev  --> dev is the name of the project

We can create multipe projects for the users you created Just use as follows

    for i engineering content prod
    do
    oc new-project $i --description="PorkChop Express engineering team"
    done

Note: What happens here is for the every user you created an Project "prod" will be assigned for that users you created before

you can list all the projects know:

    oc projects --> shows the projects of dev, engineering, content, prod all are exists

===================================

Granting the Admin privilages for dev, engineering, prod 

    oadm policy add-role-to-user admin shen -n engineering 
    oadm policy add-role-to-user admin shen -n dev
    oadm policy add-role-to-user admin shen -n prod 

Note: You are giving an admin permissions for the user "shan" for the engineerin, dev, prod projects. -n is the namespaces.

If we want the other user "chi" has get the permission as the basic-user. The command as fallows.

    oadm policy add-role-to-user basic-user chi -n dev
    oadm policy add-role-to-user basic-user chi -n engineering

Note: Here you have given an "basic-user" permission for the user "chi" for the projects dev, engineering

    oadm policy add-role-to-user admin law -n content 

Note : The command above is used to give admin policy for the user "law" for project "content" of Cluster.

    oadm policy add-role-to-user cluster-status law -n engineering
    oadm policy add-role-to-user cluster-status law -n prod

Note: The command above is used for give the admin policy role for the user "law" to see the cluster-status for the projects engineering and prod

    oadm policy add-role-to-user view law -n dev

Note: The command above is used to give the admin policy role for the user "law" to view the project "dev"

If you want to do the view admin policy role for the user "jack" for all the three projects dev, prod, content use the below command

    for i in engineering dev prod content
    do
    oadm policy add-role-to-user view jack -n $i
    done

Note: what happens here it set's all the roles for the user jack.

You can verify by using 

    oc describe policyBindings :default -n engineering
    oc describe policyBindings :default -n content
    oc describe policyBindings :default -n dev
    oc describe policyBindings :default -n prod

Note: The above four commands describes all the what user has what permissions, and roles are assigned for that user for the engineering, content, dev, prod projects.

========================================================================

Login to the Master 

    oc whoami --> can see that you are the sytem:admin
    oc describe clusterPolicy default --> Tells you to view the cluster policys

    oc describe clusterPolicyBindings :default

    oc get projects --> Get all default, kube-system,logging,management-infra,openshift,openshift-infra projects by default in the openshift.

    oc describe policyBindings :default -n openshift  --> gives you all the policy information,users,groups,imagepullers, about the "openshift" project or namespace.

================================

create a new-project by any name you need

    oc new-project <new-projectname>

Just want to know the newly created policy binding

    oc describe policyBindings :default -n <new-projectname> --> Tells all the information like it don't have noting like no labels.

    oc adm policy add-role-to-user admin <student> -n <new-projectname> 

Note: we can check that user "student" has a admin policy for the new-project that we created. Here what happens is that we are managing privilazes per the namespaces or per project bases.

what about on the Cluster basis ?

    oc adm policy add-cluster-role-to-use cluster-admin <student>

Note: what happens for the above command is we can give a user "student" a full cluster-admin access.

    oc describe clusterPolicyBindings :default | grep cluster-admin -A4 --> Describes the complete information of the cluster, users, groups.

====================================

    oc adm policy remove-user <student> -n <projectname>

Note: For the above we can remove the user student for that particular project.

==============================

    oc adm policy remove-cluster-role-from-user cluster-admin <student>

Note: want it does is that it removes the admin policy for the cluster role form that particular user.

=================================================================================================================================

Configuring Authentication:

Login to Master

Basic Configurating Authentication In order to know.

Using htpasswd with the master config file.

    yum -y install httpd-tools

Edit the master configuration file.

    vim /etc/origin/master/master-config.yaml

Look for the oauthConfig, and change "kind" attibute in the apiversion from DenyAll to HTPasswd, directly below that add a new attribute file attribute as fallows:

    kind: HTPasswdPasswordIdentityProvider
    file: /etc/origin/openshift-passwd

Create a openshift passwd file Now.

    touch /etc/origin/openshift-passwd

    htpasswd -b /etc/origin/openshift-passwd <student> <openshift>

Note: An user "student" and the password "openshift" will be created.

Now restart the service by using the below command

    systemctl restart atomic-openshift-master.service

Now a user "student" and passwd is configured, Now we have to setup the hostname

Exit from the Master and go back to the Main terminal

Go back to terminal: In the main terminal, not Master or Node

Edit 

    vim /etc/hosts

    # OCP
    <MasterIPAddress> <Hostname> <MasterDomain Name>

save it

===========================================================================

Now you are ready to open up the new browser window in the deskop

    https://<HostName>:8443 

Note: Add Exaception, and do permentently added not to get this error again.


we can see the openshift configured and setup for the user "student". So the user student can you openshift.

=====================================================================================================================================
Creating a POD:

we have to create an sample ruby Application in an openshift container platform.

Login to the cluster of master Node.

ssh root@MasterIp

oc whoami --> system:admin

oc projects --> all the project

oc project --> we have the default project

oc get pods --> we can see that our docker-registry, router, registry console working under the default projects.

=================

Create a New Project:

    oc new-project <myproject>

we can do an sample Example Ruby-Application from the default git hub repository as fallows

    oc new-app <centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git>

watch oc get pods --> can watch the build progress for the pod, Control+c to get out of this.

    oc get pods --> All the pods
    oc describe pod <podname> --> we are most conserend about IP Address.

we has to set the Router. In order to access for that Application 

    curl <IPADdress of pod we get this in pod decribe>

Note: We get this in our Terminal, Internally.

What if we want to get application in the webbrowser ?

we has to Expose the service.

    oc expose svc ruby-ex --> If the application is ruby can use that command
    oc get routes  --> get the hostname name of the project.

    curl <Hostname of the project for that route we get from above>

Note: we get ruby application running with openshift cloud.

    oc delete project <myproject>

===================================================================================================


Enable TLS Termination on Openshift routes: (Secureing a POD with TLS Termination)

Example: Do with Ruby application that we can get it from github repository.

Login to Master.

    oc whoami --> we has to make sure that we are sytem:admin

    oc new-project <secure>
    oc new-app <can paste from the above output as sample Ruby application>

    oc get builds
    watch oc get builds --> can see what going on for build. created or not takes a time of 49s
    oc get svc --> get all the services

    oc get routes --> get the routes of the application 

Copy the hostname of the new one created and paste this in browser, we can see the webpage.

what if we want to secure that application ?

we have to edit our route as fallows:

    oc edit route <route-name>

Here we get the YAML file, If we want to edit that Go directly to the ":set"  In the "spec" section and add tls, and termination: edge as fallows:

    spec:
      host: ruby-ex-secure.ocp.master.academybytes.com
      tls:
      termination: edge

If we want to edit the Port, we can do but here we are not goind to edit that, if we edit that we has to change the deployment config, build config also.

save the file 

    oc get route

we get the temination edge standard enabled. Now Copy the Rout Hostname and paste in the web browser with https.

==========================================================================================================

we can do all this with the openshift console very easily also

--> create New project by giving name, displayname, Description.
--> open catalog --> chosse python --> sample django project, --> Advanced options --> Under Routing we can automatically secure by clicking check mark secure
--> TLS Termination(Edge) --> Insecure Traffic (None) --> create. --> overview --> see over pod is ready.

--> open up near the pod hostname --> SSL warning --> Add exception --> Click the check mark Permenently store --> comfirm security group.

================================================================





     
