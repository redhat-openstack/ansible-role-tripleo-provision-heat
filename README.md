Role Name
=========

Ansible roles for deploying TripleO heat stack on existing OpenStack cloud using OpenStack Virtual Baremetal.

Requirements
------------

These roles assume that the host cloud has already been patched as per https://github.com/cybertron/openstack-virtual-baremetal/blob/master/README.rst#patching-the-host-cloud.

Role Variables
--------------

**Note:** Make sure to include all environment file and options from your [initial Overcloud creation](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/sect-Scaling_the_Overcloud.html)

Default values for the variables used in the roles are  included in https://github.com/redhat-openstack/ansible-role-tripleo-provision-heat/blob/master/roles/common-heat/defaults/main.yml.

To interact with the Openstack Virtual Baremetal host cloud, credentials are needed:
- os_username: <cloud_username>
- os_password: <user_password>
- os_tenant_name: <tenant_name>
- os_auth_url: <cloud_auth_url> # For example http://190.1.1.5:5000/v2.0


Dependencies
------------

The playbook included in this role calls https://github.com/redhat-openstack/ansible-role-tripleo-validate-ipmi and https://github.com/redhat-openstack/ansible-role-tripleo-baremetal-overcloud.

Example Playbook
----------------

Playbooks to call the deploy and cleanup related roles are included in this repo:

- https://github.com/redhat-openstack/ansible-role-tripleo-provision-heat/blob/master/playbooks/deploy-stack.yml
- https://github.com/redhat-openstack/ansible-role-tripleo-provision-heat/blob/master/playbooks/cleanup.yml
- https://github.com/redhat-openstack/ansible-role-tripleo-provision-heat/blob/master/playbooks/heat-provision-tripleo-quickstart.yml

License
-------

Apache

Author Information
------------------

RDO-CI Team

