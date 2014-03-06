pxc_pacemaker
=============


Basic steps to install this package::

  git clone <this repos git url>
  cd pxc_pacemaker
  git submodule init
  vagrant up
  vagrant ssh node1
  node1$ crm configure edit
  
You can mess with your own crm configuration, but here's a good starting point::

  primitive p_pxc ocf:percona:pxc_resource_agent \
        params enable_bootstrap="1" \
        op monitor interval="2s" role="Master" \
        op monitor interval="5s" role="Slave" \
        op start timeout="10000s" interval="0"
  ms ms_pxc p_pxc \
        meta master-max="3" clone-max="3" target-role="Started" is-managed="true" ordered="false" interleave="true" notify="false"

You'll probably also want to disable stonith and set quorum action to 'ignore' (PXC does its own quorum handling for split-brain issues)::

  crm_attribute --attr-name no-quorum-policy --attr-value ignore
  crm_attribute --attr-name stonith-enabled --attr-value false

