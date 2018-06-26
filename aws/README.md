LAMP Stack + HAProxy: Example Playbooks for Amazon Web Services
-----------------------------------------------------------------------------

- Requires Ansible 1.2
- Expects CentOS/RHEL 6 hosts

This example is an extension of the simple LAMP deployment. Here we'll install
and configure a web server with an HAProxy load balancer in front, and deploy
an application to the web servers. This set of playbooks also have the
capability to dynamically add and remove web server nodes from the deployment.
It also includes examples to do a rolling update of a stack without affecting
the service.

You can also optionally configure a Nagios monitoring node.

### Initial Site Setup

First, we provision the hosts neccessary for this demonstration using the included playbook, "demo-aws-launch.yml". This will provision the following instances, with the group structure specified below. The hosts are tagged via AWS EC2 tagging and the Ansible inventory sync script (or Tower) will create the appropriate groups from these tags.

		[tag_ansible_group_webservers]
		webserver1
		webserver2
		
		[tag_ansible_group_dbservers]
		dbserver
		
		[tag_ansible_group_lbservers]
		lbserver
		
		[tag_ansible_group_monitoring]
		nagios

=================================================================================

Should you decide to run Ansible on your workstation, you’ll need to set environment variables for your Secret and Access key:

$ export AWS_ACCESS_KEY_ID='YOUR_AWS_API_KEY'
$ export AWS_SECRET_ACCESS_KEY='YOUR_AWS_API_SECRET_KEY'

Download ec2.py and ec2.ini from the following URLS

wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini

Now you’ll need to set a few more environment variables for the inventory management script –

 $ export ANSIBLE_HOSTS=/etc/ansible/ec2.py  

Open up ec2.py in a text editor and make sure that the path to the ec2.ini config file is defined correctly at the top of the script:

$ export EC2_INI_PATH=/etc/ansible/ec2.ini  

Using an SSH agent is the best way to authenticate with your end nodes, as this alleviates the need to copy your .pem files around. To add an agent, do


$ ssh-agent bash 
$ ssh-add ~/.ssh/keypair.pem

If you call the Amazon EC2 inventory script directly, you’ll see your Amazon EC2 inventory broken down and grouped by a variety of factors. To try this, 
run 

$ /etc/ansible/ec2.py --list

For a useful example of leveraging the Amazon EC2 instance variables, I currently have two instances with the tag “Ansible Slave” applied to them. Let’s ping them real quickly.


$ ansible -m ping tag_Ansible_Slave



After which we execute the following command to deploy the site:

		ansible-playbook -i ec2.py site.yml

The deployment can be verified by accessing the IP address of your load
balancer host in a web browser: http://<ip-of-lb>:8888. Reloading the page
should have you hit different webservers.

The Nagios web interface can be reached at http://<ip-of-nagios>/nagios/

The default username and password are "nagiosadmin" / "nagiosadmin".




