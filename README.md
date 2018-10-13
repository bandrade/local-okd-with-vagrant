Overview
--------

This is a Vagrant based project that demonstrates an advanced Origin Community Distribution of Kubernetes by using an Ansible playbook.

The documentation for the installation process can be found at:

https://docs.okd.io/latest/welcome/index.html


Pre-requisites
--------------

* Vagrant
* VirtualBox or Libvirt (--provider=libvirt)

Install the following vagrant plugins:

* landrush (1.1.2)
* vagrant-hostmanager (1.8.5)

------------
Define your disks location by exporting the DISKS_LOCATION environment variable, by default it uses the current directory

    export DISKS_LOCATION=<location>

Clone the repository 

    git clone https://github.com/bandrade/local-okd-with-vagrant.git
    cd vagrant
    vagrant up 

Three ansible playbooks will start on admin1 after it has booted. The first playbook bootstraps the pre-requisites for the Openshift install. The second playbook is the actual Openshift install. The inventory for the Openshift install is declared inline on inventory file.

The install comprises one master and two nodes and one infra-node. The NFS share gets created on admin1.

The inventory makes use of the 'openshift_ip' property to force the use of the eth1 network interface which is using the 192.168.50.x ip addresses of the vagrant private network.

Once complete AND after confirming that the docker-registry pod is up and running then

Logon to https://master1.example.com:8443 as admin/admin123, create a project test then

ssh to master1:

    ssh master1
    oc login -u=system:admin

On the host machine (the following assumes RHEL/Centos, other OS may differ) first verify the contents of /etc/dnsmasq.d/vagrant-landrush gives

    server=/example.com/127.0.0.1#10053

then update the dns entries thus

    vagrant landrush set apps.example.com 192.168.50.20

In the web console create a PHP app and wait for it to complete the deployment. Navigate to the overview page for the test app and click on the link for the service i.e.

    cakephp-example-test.apps.example.com

What has just been demonstrated? The new app is deployed into a project with a node selector which requires the region label to be 'primary', this means the app gets deployed to either node1 or node2. The landrush DNS wild card entry for apps.example.com points to master1 which is where the router is running, therefore being able to render the home page of the app means that the SDN of Openshift is working properly with Vagrant.

Notes
-----

The landrush plugin creates a small DNS server to that the guest VMs can resolve each others hostnames and also the host can resolve the guest VMs hostnames. The landrush DNS server is listens on 127.0.0.1 on port 10053. It uses a dnsmasq process to redirect dns traffic to landrush. If this isn't working verify that:

    cat /etc/dnsmasq.d/vagrant-landrush

gives

    server=/example.com/127.0.0.1#10053

and that /etc/resolv.conf has an entry

    # Added by landrush, a vagrant plugin 
    nameserver 127.0.0.1
