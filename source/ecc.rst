Elastic Cloud Compute
=====================

Elastic Cloud Compute for usegalaxy.no

ECC is a cloud based HPC where the number of compute nodes that scales according
to a workload. This ensures that resources are not wasted when during
computational lulls.

ECC is implemented in python 3, and extends the usegalaxy.no Slurm cluster.

Installation
------------

OpenStack and centos system user ssh-key (if needed)
----------------------------------------------------

.. code-block:: bash

    # Make a ssh-key for the user who will be running the daemon (no passphrase)
    ssh-keygen 

    # add the generated ssh key to authorized_keys on head-node and the static slurm nodes

    # install openstack client
    pip install --upgrade --requirement https://raw.githubusercontent.com/platform9/support-locker/master/openstack-clients/requirements.txt --constraint https://raw.githubusercontent.com/openstack/requirements/stable/pike/upper-constraints.txt

    # add ssh public key to OpenStack, if needed, to be used for centos@ ssh access
    openstack keypair list
    openstack keypair create --public-key ~/.ssh/id_rsa.pub usegalaxy # use usegalaxy-test on test.usegalaxy.no


Manual configuration and running EHOS daemon
--------------------------------------------

.. code-block:: bash

    # (replace env/test/ with env/main/ for the production environment)

    sudo mkdir /srv/ecc
    sudo chown sysadmin /srv/ecc

    # create virtualenv and activate it
    cd /srv/ecc
    virtualenv -p /usr/bin/python3 venv
    source venv/bin/activate
    pip install --upgrade pip

    # install Ansible
    pip install ansible

    # Install ECC and the Ansible playbook
    pip install git+https://github.com/usegalaxy-no/ecc.git
    git clone https://github.com/usegalaxy-no/infrastructure-playbook.git

    # add ansible vault password to infrastructure-playbook/env/test/vault_password

    # make a symlink for ecc.yml into Ansible playbooks folder
    ln -s /srv/ecc/ecc.yaml /srv/ecc/infrastructure-playbook/env/test/ecc.yaml

    # make a config file:
    ecc-cli init
    # edit in password and suitable paths

    # fetch the cloud-init sample file:
    wget -O ecc_node.yaml https://raw.githubusercontent.com/usegalaxy-no/ecc/master/ecc_node.yaml.sample 

    # Add ssh-key generated above

    # modify the ecc.yaml file with correct info

    # check if daemon works
    bin/eccd.py ecc.yaml

    # run cli
    ecc-cli -c ecc.yaml help

    # add eccd.py to systemd, or run it within a Screen session

Example config file
-------------------
    
.. code-block:: bash

    cat /srv/ecc/ecc.yaml
    openstack:
        auth_url: https://api.uh-iaas.no:5000/v3
        image: 578114a1-3b45-4367-8130-b46fdea32820
        project_domain_name: dataporten
        # production env:
        project_name: elixir-nrec-prod-backend
        # test env:
        #project_name: uib-ii-usegalaxy
        region_name: bgo
        user_domain_name: dataporten
        username: feide-id
        password: nrec-api-password # https://access.nrec.no (reset api password)

    ecc:
        log: ecc.log
        nodes_max: 6
        nodes_min: 1
        nodes_spare: 1
        sleep: 30
        flavor: shpc.m1a.8xlarge
        image: GOLD CentOS 7
        key: usegalaxy # usegalaxy-test for test env
        network: dualStack
        security_groups: slurm-node
        name_template: "ecc{}.usegalaxy.no" # ecc{}.test.usegalaxy.no for test env
        cloudflare_apikey: api-key
        cloudflare_email: dnsadmin@ii.uib.no
        cloud_init: /srv/ecc/ecc_node.yaml
        ansible_cmd: "/srv/ecc/venv/bin/ansible-playbook -i '/srv/ecc/bin/ecc_nodes.py' slurm.yml -e'ansible_user=centos'"
        ansible_dir: /srv/ecc/infrastructure-playbook/env/main
        