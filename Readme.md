# Ansible-openstack-admin

The Osadmin role can manage a whole openstack cloud, including:

* Users
* Networks
* Routers
* Images
* Keypairs
* Flavors
* Security groups
* Projects
* Quotas (>=ansible 2.3)
* Host aggregate/Availibility zones (>=ansible 2.3)

Projects can be initialized with default ou specified resources. A user named after the project name is always created.

## Usage

```bash
$ virtualenv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
$ ansible-playbook -i hosts osadmin.yml
```

Osadmin.yml Example:

```yml
---
- hosts:
    - localhost
  vars:
    osadmin_projects:
      - name: client2
        description: "Client 2"
        autocreate_network: yes
        autocreate_router: yes
        autocreate_keypair: yes
        autocreate_sg: yes
        keypair_user_assos: admin  # Associate the created keypairs with an existing cloud user
        more_users:  # Associate existing user to the project
          - name: cloudkitty
            role: rating
        networks:
          - name: 'net'
            cidr: 192.168.5.0/24
            project: client2
        routers:
          - name: router
            cloud: admin
            project: client2
            network: public
            interfaces:
              - subnet-client2
        security_groups:
          - name: Remote_access
            description: "Allow SSH from internet"
            rules:
              - protocol: tcp 
                port_range_min: 22
                port_range_max: 22
                remote_ip_prefix: 0.0.0.0/0
        keypairs:  # If no pubkey is specified, a new private/public key pair will be generated
                   # The private key will be stored in the credentials directory, keep it safe !
          - name: test
            public_key_file: roles/osadmin/files/keys/client2/client2.pub
        quotas:  # requires ansible >= 2.3
          cores: 50
          ram: 102400
        state: present
    osadmin_flavors:
      - name: XS10M
        vcpus: 1
        ram: 512
        disk: 1
        limit_bandwidth: yes  # Metadata to limit bandwidth can be added
        outbound_peak: 1250
        outbound_burst: 1250
        outbound_average: 1250
        inbound_average: 1250
        inbound_peak: 1250
        inbound_burst: 1250
    osadmin_host_aggregate:  # requires ansible >= 2.3
      - name: aggr1
        availability_zone: az1
        hosts:
          - nova1
          - nova2
        metadata:
          os_type: windows
    osadmin_images:
      - name: ubuntu14.04
        disk_format: raw
        filename: ~/images/trusty-server-cloudimg-amd64-disk1.raw
        min_disk: 10

    # Variables used when the default clouds.yaml auth cannot be used (for keypairs and sec group)
    # users generated by this playbook will be used for username and password variables
    osadmin_default_auth_url: https://cloud:5000/v3
    osadmin_default_project_domain_id: 'default'
    osadmin_default_user_domain_id: 'default'
```

## Auth management

One of the following files must contain a cloud named admin with the admin credentials of the cloud:

* ~/.config/openstack/clouds.yaml
* /etc/openstack/clouds.yaml
* /etc/ansible/openstack.yaml

Clouds.yaml examples:

Keystone v3:

```yml
clouds:
  admin:
    auth:
      username: admin
      password: adminpass
      project_name: admin
      auth_url: https://cloud:5000/v3
      user_domain_id: default
      project_domain_id: default
    identity_api_version: 3
```

Keystone v2:

```yml
clouds:
  admin:
    auth:
      username: admin
      password: adminpass
      project_name: admin
      auth_url: https://cloud:5000/v2.0
```

### Note
The credentials directory contains the passwords and the ssh private keys of the users generated with this playbook, keep it safe !