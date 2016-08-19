---------------------------------
Running tripleo-quickstart on OVB
---------------------------------

By default, tripleo-quickstart uses libvirt to create virtual machines (VM) to serve
as undercloud and overcloud nodes for a TripleO deployment.
With some steps and modification, tripleo-quickstart can setup an undercloud and
deploy the overcloud on instances running on an OVB host cloud rather than libvirt VMs.

#####################
Beginning assumptions
#####################

This document details the workflow for running tripleo-quickstart on OVB.
The following are assumed to have been completed before following this document:

* A host cloud exists and has been set up and configured as described in
  <https://github.com/cybertron/openstack-virtual-baremetal#patching-the-host-cloud>
* The undercloud image under test has been uploaded to the host cloud
* A heat stack has been deployed with instances for the undercloud, bmc, and overcloud nodes
* The nodes.json file has been created (later to be copied to the undercloud as instackenv.json)

Below is an example env.yaml file used to create the heat stack that will support a
tripleo-quickstart undercloud and overcloud deployment with network isolation:

::

    parameters:
      os_user: admin
      os_password: password
      os_tenant: admin
      os_auth_url: http://10.10.10.10:5000/v2.0

      bmc_flavor: m1.medium
      bmc_image: 'bmc-base'
      bmc_prefix: 'bmc'

      baremetal_flavor: m1.large
      baremetal_image: 'ipxe-boot'
      baremetal_prefix: 'baremetal'

      key_name: 'key'
      private_net: 'private'
      node_count: {{ node_count }}
      public_net: 'public'
      provision_net: 'provision'

      # QuintupleO-specific params ignored by virtual-baremetal.yaml
      undercloud_name: 'undercloud'
      undercloud_image: '{{ latest_undercloud_image }}'
      undercloud_flavor: m1.xlarge
      external_net:  '{{ external_net }}'
      undercloud_user_data: |
            #!/bin/sh
            sed -i "s/no-port-forwarding.*sleep 10\" //" /root/.ssh/authorized_keys

    #parameter_defaults:
    ## Uncomment and customize the following to use an existing floating ip
    #  undercloud_floating_ip_id: 'uuid of floating ip'
    #  undercloud_floating_ip: 'address of floating ip'

    resource_registry:
    ## Uncomment the following to use an existing floating ip
    #  OS::OVB::UndercloudFloating: templates/undercloud-floating-existing.yaml

    ## Uncomment the following to use no floating ip
    #  OS::OVB::UndercloudFloating: templates/undercloud-floating-none.yaml

    ## Uncomment the following to create a private network
      OS::OVB::PrivateNetwork: {{ templates_dir }}/private-net-create.yaml

    ## Uncomment to create all networks required for network-isolation.
    ## parameter_defaults should be used to override default parameter values
    ## in baremetal-networks-all.yaml
      OS::OVB::BaremetalNetworks: {{ templates_dir }}/baremetal-networks-all.yaml
      OS::OVB::BaremetalPorts: {{ templates_dir }}/baremetal-ports-all.yaml

    ## Uncomment to deploy a quintupleo environment without an undercloud.
    #  OS::OVB::UndercloudEnvironment: OS::Heat::None


##################################
Setting up the undercloud instance
##################################

<pull in rst from setup-undercloud-connectivity.sh.j2>

################################################
Settings for deploying tripleo-quickstart on OVB
################################################

After the undercloud connectivity has been set up, the undercloud is installed and the
overcloud is deployed following the 'baremetal' workflow, using setting relevant to the
undercloud and baremetal nodes created on the OVB host cloud.

Below are a list of example settings (overwriting defaults) that would be passed to quickstart.sh:

::

    # undercloud.conf
    undercloud_network_cidr: 192.0.2.0/24
    undercloud_local_ip: 192.0.2.1/24
    undercloud_network_gateway: 192.0.2.1
    undercloud_undercloud_public_vip: 192.0.2.2
    undercloud_undercloud_admin_vip: 192.0.2.3
    undercloud_local_interface: eth1
    undercloud_masquerade_network: 192.0.2.0/24
    undercloud_dhcp_start: 192.0.2.5
    undercloud_dhcp_end: 192.0.2.24
    undercloud_inspection_iprange: 192.0.2.25,192.0.2.39

    overcloud_nodes:
    undercloud_type: ovb
    introspect: true

    # file locations to be copied to the undercloud (for network-isolation deployment)
    undercloud_instackenv_template: "{{ local_working_dir }}/instackenv.json"
    network_environment_file: "{{ local_working_dir }}/openstack-virtual-baremetal/network-templates/network-environment.yaml"
    baremetal_nic_configs: "{{ local_working_dir }}/openstack-virtual-baremetal/network-templates/nic-configs"

    network_isolation: true

    # used for access to external network
    external_interface: eth2
    external_interface_ip: 10.0.0.1
    external_interface_netmask: 255.255.255.0
    external_interface_hwaddr: fa:05:04:03:02:01

    # used for validation
    floating_ip_cidr: 10.0.0.0/24
    public_net_pool_start: 10.0.0.50
    public_net_pool_end: 10.0.0.100
    public_net_gateway: 10.0.0.1

########################################
Adjusting MTU values prior to deployment
########################################

.. note:: The MTUS of the interfaces need to be adjusted again prior to deployment
   because  undercloud installation step resets the  MTU value on the provisioning
   interface. The adjusted MTU value must be added to ``dnsmasq-ironic.conf`` to
   set the values for the overcloud NICs.

<see adjust mtus template.sh.j2>
