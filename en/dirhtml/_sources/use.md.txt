# CLI usage

Execute
~~~shell
clouni --help
~~~
Output
~~~text
usage: clouni [-h] --template-file <filename> --cluster-name CLUSTER_NAME
              [--validate-only] [--delete] [--provider PROVIDER]
              [--output-file <filename>]
              [--configuration-tool CONFIGURATION_TOOL] [--async]
              [--extra KEY=VALUE [KEY=VALUE ...]] [--debug]
              [--log-level {debug,info,warning,error,critical}]

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
  --async               Provider nodes should be created asynchronously
  --extra KEY=VALUE [KEY=VALUE ...]
                        Extra arguments for configuration tool scripts
  --debug               Set debug level for tool
  --log-level {debug,info,warning,error,critical}
                        Set log level for tool
~~~

## Examples

### Check full example of Clouni possibilities for OpenStack provider
Small example of input:
~~~yaml
tosca_definitions_version: tosca_simple_yaml_1_0

topology_template:
  node_templates:
    server_kube_master:
      type: tosca.nodes.Compute
      capabilities:
        os:
          properties:
            architecture: x86_64
            type: ubuntu
            distribution: xenial
            version: 16.04
~~~

Topology template contains several node or relationship templates to create in a cloud.
Templates can be of different types.
The only type supported by Clouni is `Compute` as in the example.
Other type are planned to be supported in the future.

### Creating

~~~shell
clouni --template-file examples/tosca-server-example.yaml --cluster-name example --provider openstack
~~~
Clouni output is Ansible playbook.
~~~yaml
- hosts: localhost
  name: Create openstack cluster
  tasks:
  - os_image_facts: {}
    register: facts_result
  - register: tmp_value
    set_fact:
      target_objects: '{{ facts_result["ansible_facts"]["openstack_image"] }}'
  - set_fact:
      target_objects_8838: '{{ target_objects }}'
  - set_fact:
      input_facts: '{{ target_objects_8838 }}'
  - set_fact:
      input_args_5697:
      - - name
        - properties
      - architecture: x86_64
        distribution: xenial
        type: ubuntu
        version: 16.04
  - set_fact:
      input_args: '{{ input_args_5697 }}'
  - include: contains.yaml
  - register: tmp_value
    set_fact:
      name: '{{ matched_object["name"] }}'
  - set_fact:
      name_1511: '{{ name }}'
  - file:
      path: '{{ playbook_dir }}/id_vars_example.yaml'
      state: absent
  - file:
      path: '{{ playbook_dir }}/id_vars_example.yaml'
      state: touch
  - name: Create OpenStack component server
    os_server:
      config_drive: false
      image: '{{ name_1511 }}'
      name: server_kube_master
    register: server_kube_master_server
  - lineinfile:
      line: 'server_kube_master_server: {{ server_kube_master_server.id }}'
      path: '{{ playbook_dir }}/id_vars_example.yaml'
    when: server_kube_master_server.id is defined
  - fail:
      msg: Variable server_kube_master_server.id is undefined! So it will not be deleted
    ignore_errors: true
    when: server_kube_master_server.id is undefined
~~~
### Deleting
To generate playbook with the same input template, just add a --delete command
~~~shell
clouni --template-file examples/tosca-server-example.yaml --cluster-name example --provider openstack --delete
~~~
Clouni output
~~~yaml
- hosts: localhost
  name: Delete openstack cluster
  tasks:
  - include_vars: '{{ playbook_dir }}/id_vars_example.yaml'
  - name: Delete OpenStack component server
    os_server:
      name: '{{ server_kube_master_server }}'
      state: absent
    when: server_kube_master_server is defined
  - file:
      path: '{{ playbook_dir }}/id_vars_example.yaml'
      state: absent
~~~
Ansible playbook can be executed to create or delete an instance
~~~shell
ansible-playbook <playbook_name>.yaml
~~~
After the execution of creating playbook, the file **id_vars_<cluster-name>.yaml** will be created. This file contains all resources ids. Deleting playbook uses ids from this file and after successful deleting removes file too.

**ATTENTION** Most of providers require authentication for using there resources.
Authentication is users responsibility. For example, to use OpenStack
you must download your OpenStack RC file and `source` it.
After that user is able to execute Ansible playbook.

During the Clouni execution it doesn't send or receive any information from cloud
or Internet.

Generated script consists of two parts:
1. Get cloud information and choosing specific cloud parameters for meeting requirements.
1. Create cloud resources

For example to create OpenStack server cloud image name and flavor must be specified.
