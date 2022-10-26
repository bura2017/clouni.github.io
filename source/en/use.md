# CLI usage (deprecated version)

Execute
~~~shell
clouni --help
~~~
Output
~~~text
usage: clouni [-h] --template-file <filename> --cluster-name CLUSTER_NAME
              [--validate-only] [--delete] [--provider PROVIDER]
              [--output-file <filename>]
              [--configuration-tool CONFIGURATION_TOOL]
              [--extra KEY=VALUE [KEY=VALUE ...]] [--debug]
              [--log-level {debug,info,warning,error,critical}]
              [--host-parameter HOST_PARAMETER]
              [--public-key-path PUBLIC_KEY_PATH]

optional arguments:
  -h, --help            show this help message and exit
  --template-file <filename>
                        YAML template to parse.
  --cluster-name CLUSTER_NAME
                        Cluster name
  --validate-only       Only validate input template, do not perform
                        translation.
  --delete              Delete cluster
  --provider PROVIDER   Cloud provider name to execute ansible playbook in.
  --output-file <filename>
                        Output file
  --configuration-tool CONFIGURATION_TOOL
                        Configuration tool which DSL the template would be
                        translated to. Default value = "ansible"
  --extra KEY=VALUE [KEY=VALUE ...]
                        Extra arguments for configuration tool scripts
  --debug               Set debug level for tool
  --log-level {debug,info,warning,error,critical}
                        Set log level for tool
  --host-parameter HOST_PARAMETER
                        Specify Compute property to be used as host IP for
                        software components that hosted on the Compute. Valid
                        values: public_address and private_address
  --public-key-path PUBLIC_KEY_PATH
                        Set path to public key for configuration software on
                        cloud servers

~~~

Small example of input:
~~~yaml
tosca_definitions_version: tosca_simple_yaml_1_0

topology_template:
  node_templates:
    tosca_server_example:
      type: tosca.nodes.Compute
      properties:
        meta: "cube_master=true"
        public_address: 10.100.157.77
        networks:
          default:
            network_name: sandbox_net
      capabilities:
        host:
          properties:
            num_cpus: 1
            disk_size: 10 GiB
            mem_size: 1 GiB
        endpoint:
          properties:
            protocol: tcp
            port: 22
            initiator: target
            ip_address: 0.0.0.0
        os:
          properties:
            architecture: x86_64
            type: cirros
            version: 0.4.0
~~~

Topology template contains several node or relationship templates to create in a cloud.
Templates can be of different types. You can specify operations for creating/configuring/launching/deleting (etc.) cloud resources. Examples can be seen [here](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/TOSCA-Simple-Profile-YAML-v1.0.html ) and [here](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/TOSCA-Simple-Profile-YAML-v1.3.html). However, it is worth noting that there is no full support for TOSCA v1.3 in Clouni.

## Creating

~~~shell
clouni --template-file examples/tosca-server-example.yaml --cluster-name example --provider openstack
~~~

Clouni collects facts from the cloud you are using and generates Ansible playbooks according to a given TOSCA template, which are executed in parallel, considering the dependencies of nodes, relationships and their operations.
The following is an example of playbook for the Openstack-based cloud. Absolutely similar behavior is available for AWS Cloud.
It is also possible to use Clouni as a Kubernetes cluster management tool.

If you only want to generate a playbook without launching it, specify the ```--debug``` key.
~~~yaml
- name: 'Create OpenStack component openstack cluster: tosca_server_example_security_group:create'
  hosts: localhost
  tasks:
  - file:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      state: absent
  - file:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      state: touch
  - name: Create OpenStack component security group
    os_security_group:
      name: tosca_server_example_security_group
    register: tosca_server_example_security_group
  - set_fact:
      tosca_server_example_security_group_list: '{{ tosca_server_example_security_group_list
        | default([]) }} + [ "{{ item.id }}" ]'
    loop: '{{ tosca_server_example_security_group.results | flatten(levels=1)  }}'
    when: item.id  is defined
  - set_fact:
      tosca_server_example_security_group_list:
        tosca_server_example_security_group_ids: '{{ tosca_server_example_security_group_list
          }}'
    when: tosca_server_example_security_group_list is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: 'tosca_server_example_security_group_delete: {{ tosca_server_example_security_group.id
        }}'
    when: tosca_server_example_security_group.id is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: '{{ tosca_server_example_security_group_list | to_nice_yaml }}'
    when: tosca_server_example_security_group_list is defined
  - fail:
      msg: Variable tosca_server_example_security_group is undefined! So it will not
        be deleted
    when: tosca_server_example_security_group_list is undefined and tosca_server_example_security_group.id
      is undefined
    ignore_errors: true
- name: 'Create OpenStack component openstack cluster: tosca_server_example_keypair:create'
  hosts: localhost
  tasks:
  - name: Create OpenStack component keypair
    os_keypair:
      name: tosca_server_example_keypair
      public_key_file: ~/.ssh/id_rsa.pub
    register: tosca_server_example_keypair
  - set_fact:
      tosca_server_example_keypair_list: '{{ tosca_server_example_keypair_list | default([])
        }} + [ "{{ item.id }}" ]'
    loop: '{{ tosca_server_example_keypair.results | flatten(levels=1)  }}'
    when: item.id  is defined
  - set_fact:
      tosca_server_example_keypair_list:
        tosca_server_example_keypair_ids: '{{ tosca_server_example_keypair_list }}'
    when: tosca_server_example_keypair_list is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: 'tosca_server_example_keypair_delete: {{ tosca_server_example_keypair.id
        }}'
    when: tosca_server_example_keypair.id is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: '{{ tosca_server_example_keypair_list | to_nice_yaml }}'
    when: tosca_server_example_keypair_list is defined
  - fail:
      msg: Variable tosca_server_example_keypair is undefined! So it will not be deleted
    when: tosca_server_example_keypair_list is undefined and tosca_server_example_keypair.id
      is undefined
    ignore_errors: true
- name: 'Create OpenStack component openstack cluster: tosca_server_example_security_group_rule:create'
  hosts: localhost
  tasks:
  - name: Create OpenStack component security group rule
    os_security_group_rule:
      direction: ingress
      ethertype: IPv4
      port_range_max: '{{ port[item | int] | default(omit) }}'
      port_range_min: '{{ port[item | int] | default(omit) }}'
      protocol: '{{ protocol[item | int] | default(omit) }}'
      remote_ip_prefix: 0.0.0.0
      security_group: tosca_server_example_security_group
    register: tosca_server_example_security_group_rule
    with_sequence: start=0 end={{ [protocol | length, port | length] | max - 1 }}
      format=%d
    vars:
      protocol:
      - tcp
      port:
      - 22
  - set_fact:
      tosca_server_example_security_group_rule_list: '{{ tosca_server_example_security_group_rule_list
        | default([]) }} + [ "{{ item.id }}" ]'
    loop: '{{ tosca_server_example_security_group_rule.results | flatten(levels=1)  }}'
    when: item.id  is defined
  - set_fact:
      tosca_server_example_security_group_rule_list:
        tosca_server_example_security_group_rule_ids: '{{ tosca_server_example_security_group_rule_list
          }}'
    when: tosca_server_example_security_group_rule_list is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: 'tosca_server_example_security_group_rule_delete: {{ tosca_server_example_security_group_rule.id
        }}'
    when: tosca_server_example_security_group_rule.id is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: '{{ tosca_server_example_security_group_rule_list | to_nice_yaml }}'
    when: tosca_server_example_security_group_rule_list is defined
  - fail:
      msg: Variable tosca_server_example_security_group_rule is undefined! So it will
        not be deleted
    when: tosca_server_example_security_group_rule_list is undefined and tosca_server_example_security_group_rule.id
      is undefined
    ignore_errors: true
- name: 'Create OpenStack component openstack cluster: tosca_server_example_server:create'
  hosts: localhost
  tasks:
  - name: Create OpenStack component server
    os_server:
      auto_ip: false
      boot_from_volume: false
      reuse_ips: true
      config_drive: false
      meta: cube_master=true
      name: tosca_server_example
      nics:
      - net-name: sandbox_net
      key_name: tosca_server_example_keypair
      flavor: 3d7e6639-730d-4bac-aebd-60b1baa2dd70
      security_groups:
      - tosca_server_example_security_group
      image: cirros-0.4.0-x86_64
    register: tosca_server_example_server
  - set_fact:
      ansible_user: cirros
  - set_fact:
      host_ip: '{{ tosca_server_example_server.server.public_v4 }}'
  - set_fact:
      group: tosca_server_example_private_address
  - include: /tmp/clouni/example/artifacts/add_host.yaml
  - set_fact:
      tosca_server_example_server_list: '{{ tosca_server_example_server_list | default([])
        }} + [ "{{ item.id }}" ]'
    loop: '{{ tosca_server_example_server.results | flatten(levels=1)  }}'
    when: item.id  is defined
  - set_fact:
      tosca_server_example_server_list:
        tosca_server_example_server_ids: '{{ tosca_server_example_server_list }}'
    when: tosca_server_example_server_list is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: 'tosca_server_example_server_delete: {{ tosca_server_example_server.id
        }}'
    when: tosca_server_example_server.id is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: '{{ tosca_server_example_server_list | to_nice_yaml }}'
    when: tosca_server_example_server_list is defined
  - fail:
      msg: Variable tosca_server_example_server is undefined! So it will not be deleted
    when: tosca_server_example_server_list is undefined and tosca_server_example_server.id
      is undefined
    ignore_errors: true
- name: 'Create OpenStack component openstack cluster: tosca_server_example_floating_ip:create'
  hosts: localhost
  tasks:
  - name: Create OpenStack component floating ip
    os_floating_ip:
      floating_ip_address: 10.100.157.77
      server: tosca_server_example
      network: not_found
    register: tosca_server_example_floating_ip
  - set_fact:
      ansible_user: cirros
  - set_fact:
      host_ip: '{{ tosca_server_example_floating_ip.floating_ip.floating_ip_address
        }}'
  - set_fact:
      group: tosca_server_example_public_address
  - include: /tmp/clouni/example/artifacts/add_host.yaml
  - set_fact:
      tosca_server_example_floating_ip_dict:
        tosca_server_example_floating_ip_dict:
          floating_ip_address: '{{ tosca_server_example_floating_ip.floating_ip.floating_ip_address
            }}'
          server: '{{ tosca_server_example_floating_ip.floating_ip.port_details.device_id
            }}'
          purge: 'yes'
          state: absent
    when: tosca_server_example_floating_ip is defined
  - lineinfile:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      line: '{{ tosca_server_example_floating_ip_dict | to_nice_yaml }}'
    when: tosca_server_example_floating_ip_dict is defined
~~~
## Deleting
To generate playbook with the same input template, just add a ```--delete``` command
~~~shell
clouni --template-file examples/tosca-server-example.yaml --cluster-name example --provider openstack --delete
~~~
File with playbooks for deleting cloud resources:
~~~yaml
- name: 'Delete OpenStack component openstack cluster: tosca_server_example_security_group_rule:delete'
  hosts: localhost
  tasks:
  - include_vars: /home/sadimer/Desktop/clouni/id_vars_example.yaml
- name: 'Delete OpenStack component openstack cluster: tosca_server_example_floating_ip:delete'
  hosts: localhost
  tasks:
  - include_vars: /home/sadimer/Desktop/clouni/id_vars_example.yaml
  - name: Delete OpenStack component floating ip
    os_floating_ip:
      name: '{{ tosca_server_example_floating_ip_delete }}'
      state: absent
    when: tosca_server_example_floating_ip_delete is defined
    register: tosca_server_example_floating_ip_var
  - set_fact: tosca_server_example_floating_ip='{{ tosca_server_example_floating_ip_var
      }}'
    when: tosca_server_example_floating_ip_var.changed
  - name: Delete OpenStack component floating ip
    os_floating_ip:
      name: '{{ item }}'
      state: absent
    when: tosca_server_example_floating_ip_ids is defined
    loop: '{{ tosca_server_example_floating_ip_ids | flatten(levels=1) }}'
    register: tosca_server_example_floating_ip_var
  - set_fact: tosca_server_example_floating_ip='{{ tosca_server_example_floating_ip_var
      }}'
    when: tosca_server_example_floating_ip_var.changed
  - name: Delete OpenStack component floating ip
    os_floating_ip: '{{ tosca_server_example_floating_ip_dict }}'
    when: tosca_server_example_floating_ip_dict is defined
    register: tosca_server_example_floating_ip_var
  - set_fact: tosca_server_example_floating_ip='{{ tosca_server_example_floating_ip_var
      }}'
    when: tosca_server_example_floating_ip_var.changed
- name: 'Delete OpenStack component openstack cluster: tosca_server_example_server:delete'
  hosts: localhost
  tasks:
  - include_vars: /home/sadimer/Desktop/clouni/id_vars_example.yaml
  - name: Delete OpenStack component server
    os_server:
      name: '{{ tosca_server_example_server_delete }}'
      state: absent
    when: tosca_server_example_server_delete is defined
    register: tosca_server_example_server_var
  - set_fact: tosca_server_example_server='{{ tosca_server_example_server_var }}'
    when: tosca_server_example_server_var.changed
  - name: Delete OpenStack component server
    os_server:
      name: '{{ item }}'
      state: absent
    when: tosca_server_example_server_ids is defined
    loop: '{{ tosca_server_example_server_ids | flatten(levels=1) }}'
    register: tosca_server_example_server_var
  - set_fact: tosca_server_example_server='{{ tosca_server_example_server_var }}'
    when: tosca_server_example_server_var.changed
  - name: Delete OpenStack component server
    os_server: '{{ tosca_server_example_server_dict }}'
    when: tosca_server_example_server_dict is defined
    register: tosca_server_example_server_var
  - set_fact: tosca_server_example_server='{{ tosca_server_example_server_var }}'
    when: tosca_server_example_server_var.changed
- name: 'Delete OpenStack component openstack cluster: tosca_server_example_security_group:delete'
  hosts: localhost
  tasks:
  - include_vars: /home/sadimer/Desktop/clouni/id_vars_example.yaml
  - name: Delete OpenStack component security group
    os_security_group:
      name: '{{ tosca_server_example_security_group_delete }}'
      state: absent
    when: tosca_server_example_security_group_delete is defined
    register: tosca_server_example_security_group_var
  - set_fact: tosca_server_example_security_group='{{ tosca_server_example_security_group_var
      }}'
    when: tosca_server_example_security_group_var.changed
  - name: Delete OpenStack component security group
    os_security_group:
      name: '{{ item }}'
      state: absent
    when: tosca_server_example_security_group_ids is defined
    loop: '{{ tosca_server_example_security_group_ids | flatten(levels=1) }}'
    register: tosca_server_example_security_group_var
  - set_fact: tosca_server_example_security_group='{{ tosca_server_example_security_group_var
      }}'
    when: tosca_server_example_security_group_var.changed
  - name: Delete OpenStack component security group
    os_security_group: '{{ tosca_server_example_security_group_dict }}'
    when: tosca_server_example_security_group_dict is defined
    register: tosca_server_example_security_group_var
  - set_fact: tosca_server_example_security_group='{{ tosca_server_example_security_group_var
      }}'
    when: tosca_server_example_security_group_var.changed
- name: 'Delete OpenStack component openstack cluster: tosca_server_example_keypair:delete'
  hosts: localhost
  tasks:
  - include_vars: /home/sadimer/Desktop/clouni/id_vars_example.yaml
  - name: Delete OpenStack component keypair
    os_keypair:
      name: '{{ tosca_server_example_keypair_delete }}'
      state: absent
    when: tosca_server_example_keypair_delete is defined
    register: tosca_server_example_keypair_var
  - set_fact: tosca_server_example_keypair='{{ tosca_server_example_keypair_var }}'
    when: tosca_server_example_keypair_var.changed
  - name: Delete OpenStack component keypair
    os_keypair:
      name: '{{ item }}'
      state: absent
    when: tosca_server_example_keypair_ids is defined
    loop: '{{ tosca_server_example_keypair_ids | flatten(levels=1) }}'
    register: tosca_server_example_keypair_var
  - set_fact: tosca_server_example_keypair='{{ tosca_server_example_keypair_var }}'
    when: tosca_server_example_keypair_var.changed
  - name: Delete OpenStack component keypair
    os_keypair: '{{ tosca_server_example_keypair_dict }}'
    when: tosca_server_example_keypair_dict is defined
    register: tosca_server_example_keypair_var
  - set_fact: tosca_server_example_keypair='{{ tosca_server_example_keypair_var }}'
    when: tosca_server_example_keypair_var.changed
- name: Renew id_vars_example.yaml
  hosts: localhost
  tasks:
  - file:
      path: /home/sadimer/Desktop/clouni/id_vars_example.yaml
      state: absent
~~~
After the execution of creating playbooks, the file **id_vars_{cluster-name}.yaml** will be created. This file contains all resources ids. Deleting playbook uses ids from this file and after successful deleting removes file too.

**ATTENTION** Most of providers require authentication for using there resources.
Authentication is users responsibility. For example, to use OpenStack
you must download your OpenStack RC file and `source` it or use the clouds.yaml file.
After that user is able to use Clouni for creating and deleting cloud infrastructure.

During the Clouni execution it doesn't send or receive any information from cloud
or Internet.

During execution, Clouni takes only 2 steps:
1. Get cloud information and choosing specific cloud parameters for meeting requirements.
2. Create cloud resources

For example to create OpenStack server cloud image name and flavor must be specified.

For software configuration on virtual servers, Clouni creates a key pair using the path to the key that you specify in ```--public-key-path```.