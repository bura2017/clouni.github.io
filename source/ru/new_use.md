# Использование клиента clouni 

## Перед использованием
Для доступа к микросервисам Clouni необходимо заполнить некоторые environment переменные:
~~~shell
export CLOUNI_CONFIGURATION_TOOL_ENDPOINT=1.1.1.1:5000
export CLOUNI_PROVIDER_TOOL_ENDPOINT=2.2.2.2:5000
~~~
Для доступа к фактам из облака и обеспечения запуска Ansible скриптов используется [grpc-cotea](https://github.com/ispras/cotea):
~~~shell
export GRPC_COTEA_ENDPOINT=3.3.3.3:5000
~~~
Также в Clouni предусмотрена возможность сохранять развернутые шаблоны топологий облачных приложений, для этого используется специальное [API](https://github.com/DYDKA4/API_COURSE) графовой СУБД [nebula](https://www.nebula-graph.io/):
~~~shell
export DATABASE_API_ENDPOINT=4.4.4.4:5000
~~~
Если же данные переменные не указаны, Clouni использует параметры по умолчанию **localhost:50051** для provider tool, **localhost:50052** для configuration tool. 
Вы также можете поменять эти endpoints флагами ```--provider-tool-endpoint```, ```--configuration-tool-endpoint```, ```--database-api-endpoint```, ```--grpc-cotea-endpoint```.

Если **GRPC_COTEA_ENDPOINT** не указан, то Clouni работает в режиме генератора Ansible плейбуков: не собирает факты из облака и не запускает Ansible скрипты для развертывания инфраструктуры.

После выполнения плейбуков будет создан файл **id_vars_{cluster-name}.yaml**. Этот файл содержит идентификаторы всех ресурсов. Clouni использует этот файл для удаления соответствующих облачных ресурсов.

Если **DATABASE_API_ENDPOINT** не указан, Clouni сохраняет только id разворачиваемых ресурсов в файле **id_vars_{cluster-name}.yaml**, иначе сохраняет узлы и отношения в полноценном виде в базе данных.

**ВАЖНО!**
Чтобы использовать пользовательские Ansible скрипты для настройки ПО и дополнительной настройки инфраструктуры необходимо положить их в папку artifacts и переустановить клиент clouni.

При запуске клиента с установленной environment переменной **GRPC_COTEA_HOST_USER** клиент подключается к серверу grpc-cotea по SSH (или копирует модули в нужное место при локальном использовании grpc-cotea) и передает туда дополнительные Ansible модули которые использует Clouni и пользовательские скрипты из папки artifacts. Достаточно использовать эту переменную только при первом запуске клиента.
~~~shell
export GRPC_COTEA_HOST_USER=ubuntu
~~~
Команда:
~~~shell
clouni --help
~~~
Вывод:
~~~
usage: clouni [-h] --template-file <filename> --cluster-name <unique_name>
              [--validate-only] [--delete]
              [--provider {openstack,amazon,kubernetes}]
              [--output-file <filename>]
              [--configuration-tool {ansible,kubernetes}]
              [--extra KEY=VALUE [KEY=VALUE ...]] [--debug]
              [--log-level {debug,info,warning,error,critical}]
              [--host-parameter <parameter>] [--public-key-path <path>]
              [--provider-tool-endpoint <host:port>]
              [--configuration-tool-endpoint <host:port>]
              [--database-api-endpoint <host:port>]
              [--grpc-cotea-endpoint <host:port>]
              <MODULE>

positional arguments:
  <MODULE>              use provider_tool for creating provider template and writing it to stdout or file 
                        use configuration_tool for translating and deploying ansible generated from provider template (or getting ansible playbook with --debug) 
                        use deploy/delete for deploying/deleting TOSCA normative template (or getting ansible playbook with --debug) 

optional arguments:
  -h, --help            show this help message and exit
  --template-file <filename>
                        YAML template to parse
  --cluster-name <unique_name>
                        cluster name
  --validate-only       only validate input template, do not perform translation
  --delete              delete cluster
  --provider {openstack,amazon,kubernetes}
                        cloud provider name to execute ansible playbook in
  --output-file <filename>
                        output file
  --configuration-tool {ansible,kubernetes}
                        configuration tool which DSL the template would be translated toDefault value = "ansible"
  --extra KEY=VALUE [KEY=VALUE ...]
                        extra arguments for configuration tool scripts
  --debug               set debug level for tool
  --log-level {debug,info,warning,error,critical}
                        set log level for tool
  --host-parameter <parameter>
                        specify Compute property to be used as host IP for software components that hosted on the Compute
                        valid values: public_address and private_address
  --public-key-path <path>
                        set path to public key for configuration software on cloud servers
  --provider-tool-endpoint <host:port>
                        endpoint of provider tool server, default - localhost:50051
  --configuration-tool-endpoint <host:port>
                        endpoint of configuration tool server, default - localhost:50052
  --database-api-endpoint <host:port>
                        endpoint of database api for tosca template storage, default - localhost:5000
  --grpc-cotea-endpoint <host:port>
                        endpoint of grpc-cotea, default - localhost:50151
~~~

Небольшой пример входных данных:
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

Шаблон топологии содержит несколько шаблонов узлов или отношений для создания инфраструктуры в облаке.
Шаблоны могут быть разных типов. Можно задавать операции для создания/конфигурации/запуска/удаления (и т.д.) облачных ресурсов. Примеры можно увидеть [здесь](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/TOSCA-Simple-Profile-YAML-v1.0.html) и [здесь](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.3/TOSCA-Simple-Profile-YAML-v1.3.html). Однако стоит отметить что полной поддержки TOSCA v1.3 в Clouni нет.

## Создание

~~~shell
clouni deploy --template-file examples/tosca-server-example-scalable.yaml --provider openstack --cluster-name test
~~~
Clouni собирает факты из используемого вами облака, затем генерирует внутренние TOSCA типы для указанного провайдера,
а затем генерирует по заданному TOSCA шаблону Ansible плейбуки, которые выполняются параллельно, учитывая зависимости узлов, отношений и их операций. 
Далее приведен пример плейбука для облака на базе Openstack. Абсолютно аналогичное поведение доступно для облаков AWS Cloud. 
Также есть возможность использовать Clouni как инструмент управления кластером Kubernetes.

Если вы хотите только сгенерировать плейбуки, не запуская их - укажите ключ ```--debug```.
~~~yaml
- hosts: localhost
  name: 'Create OpenStack component openstack cluster: testing_tosca_security_group:create'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Create OpenStack component security group
    os_security_group:
      name: testing_tosca_security_group
    register: testing_tosca_security_group
  - loop: '{{ testing_tosca_security_group.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_security_group_list: '{{ testing_tosca_security_group_list | default([])
        }} + [ "{{ item.id }}" ]'
    when: item.id  is defined
  - loop: '{{ testing_tosca_security_group.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_security_group_list: '{{ testing_tosca_security_group_list | default([])
        }} + [ "{{ item.security_group_ids[0] }}" ]'
    when: item.security_group_ids is defined
  - set_fact:
      testing_tosca_security_group_list:
        testing_tosca_security_group_ids: '{{ testing_tosca_security_group_list }}'
    when: testing_tosca_security_group_list is defined
  - lineinfile:
      line: 'testing_tosca_security_group_delete: {{ testing_tosca_security_group.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group.id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_delete: {{ testing_tosca_security_group.security_group_id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group.security_group_id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_delete: {{ testing_tosca_security_group.security.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group.security.id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_delete: {{ testing_tosca_security_group.group.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group.group.id is defined
  - lineinfile:
      line: '{{ testing_tosca_security_group_list | to_nice_yaml }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_list is defined
  - fail:
      msg: Variable testing_tosca_security_group is undefined! So it will not be deleted
    ignore_errors: true
    when: testing_tosca_security_group_list is undefined and testing_tosca_security_group.id
      is undefined
- hosts: localhost
  name: 'Create OpenStack component openstack cluster: testing_tosca_keypair:create'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Create OpenStack component keypair
    os_keypair:
      name: testing_tosca_keypair
      public_key: '{{ lookup(''file'', ''~/.ssh/id_rsa.pub'') }}'
    register: testing_tosca_keypair
  - loop: '{{ testing_tosca_keypair.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_keypair_list: '{{ testing_tosca_keypair_list | default([]) }}
        + [ "{{ item.id }}" ]'
    when: item.id  is defined
  - loop: '{{ testing_tosca_keypair.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_keypair_list: '{{ testing_tosca_keypair_list | default([]) }}
        + [ "{{ item.keypair_ids[0] }}" ]'
    when: item.keypair_ids is defined
  - set_fact:
      testing_tosca_keypair_list:
        testing_tosca_keypair_ids: '{{ testing_tosca_keypair_list }}'
    when: testing_tosca_keypair_list is defined
  - lineinfile:
      line: 'testing_tosca_keypair_delete: {{ testing_tosca_keypair.id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_keypair.id is defined
  - lineinfile:
      line: 'testing_tosca_keypair_delete: {{ testing_tosca_keypair.keypair_id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_keypair.keypair_id is defined
  - lineinfile:
      line: 'testing_tosca_keypair_delete: {{ testing_tosca_keypair.keypair.id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_keypair.keypair.id is defined
  - lineinfile:
      line: '{{ testing_tosca_keypair_list | to_nice_yaml }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_keypair_list is defined
  - fail:
      msg: Variable testing_tosca_keypair is undefined! So it will not be deleted
    ignore_errors: true
    when: testing_tosca_keypair_list is undefined and testing_tosca_keypair.id is
      undefined
- hosts: localhost
  name: 'Create OpenStack component openstack cluster: testing_tosca_security_group_rule:create'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Create OpenStack component security group rule
    os_security_group_rule:
      direction: '{{ initiator[item | int] | default(omit) }}'
      ethertype: IPv4
      port_range_max: '{{ port[item | int] | default(omit) }}'
      port_range_min: '{{ port[item | int] | default(omit) }}'
      protocol: '{{ protocol[item | int] | default(omit) }}'
      remote_ip_prefix: 0.0.0.0/0
      security_group: testing_tosca_security_group
    register: testing_tosca_security_group_rule
    vars:
      initiator:
      - ingress
      port:
      - 22
      protocol:
      - tcp
    with_sequence: start=0 end={{ [protocol | length, port | length, initiator | length]
      | max - 1 }} format=%d
  - loop: '{{ testing_tosca_security_group_rule.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_security_group_rule_list: '{{ testing_tosca_security_group_rule_list
        | default([]) }} + [ "{{ item.id }}" ]'
    when: item.id  is defined
  - loop: '{{ testing_tosca_security_group_rule.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_security_group_rule_list: '{{ testing_tosca_security_group_rule_list
        | default([]) }} + [ "{{ item.security_group_rule_ids[0] }}" ]'
    when: item.security_group_rule_ids is defined
  - set_fact:
      testing_tosca_security_group_rule_list:
        testing_tosca_security_group_rule_ids: '{{ testing_tosca_security_group_rule_list
          }}'
    when: testing_tosca_security_group_rule_list is defined
  - lineinfile:
      line: 'testing_tosca_security_group_rule_delete: {{ testing_tosca_security_group_rule.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule.id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_rule_delete: {{ testing_tosca_security_group_rule.security_group_rule_id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule.security_group_rule_id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_rule_delete: {{ testing_tosca_security_group_rule.security.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule.security.id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_rule_delete: {{ testing_tosca_security_group_rule.group.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule.group.id is defined
  - lineinfile:
      line: 'testing_tosca_security_group_rule_delete: {{ testing_tosca_security_group_rule.rule.id
        }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule.rule.id is defined
  - lineinfile:
      line: '{{ testing_tosca_security_group_rule_list | to_nice_yaml }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_security_group_rule_list is defined
  - fail:
      msg: Variable testing_tosca_security_group_rule is undefined! So it will not
        be deleted
    ignore_errors: true
    when: testing_tosca_security_group_rule_list is undefined and testing_tosca_security_group_rule.id
      is undefined
- hosts: localhost
  name: 'Create OpenStack component openstack cluster: testing_tosca_server:create'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Create OpenStack component server
    os_server:
      auto_ip: false
      boot_from_volume: false
      config_drive: false
      flavor: 3d7e6639-730d-4bac-aebd-60b1baa2dd70
      image: Ubuntu-Server-18.04-LTS-Bionic-Beaver
      key_name: testing_tosca_keypair
      meta: cube_master=true
      name: testing_tosca_{{ item }}
      nics:
      - net-name: test_ext
      reuse_ips: true
      security_groups:
      - testing_tosca_security_group
    register: testing_tosca_server
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      ansible_user: ubuntu
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      group: testing_tosca_server_private_address
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      host_ip: '{{ host_ip | default([]) + [[ "testing_tosca_private_address_" + item,
        testing_tosca_server.results[item | int - 1].server.public_v4 ]] }}'
    with_sequence: start=1 end=3 format=%d
  - include: /tmp/clouni/artifacts/add_host.yaml
  - loop: '{{ testing_tosca_server.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_server_list: '{{ testing_tosca_server_list | default([]) }} +
        [ "{{ item.id }}" ]'
    when: item.id  is defined
  - loop: '{{ testing_tosca_server.results | flatten(levels=1)  }}'
    set_fact:
      testing_tosca_server_list: '{{ testing_tosca_server_list | default([]) }} +
        [ "{{ item.server_ids[0] }}" ]'
    when: item.server_ids is defined
  - set_fact:
      testing_tosca_server_list:
        testing_tosca_server_ids: '{{ testing_tosca_server_list }}'
    when: testing_tosca_server_list is defined
  - lineinfile:
      line: 'testing_tosca_server_delete: {{ testing_tosca_server.id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_server.id is defined
  - lineinfile:
      line: 'testing_tosca_server_delete: {{ testing_tosca_server.server_id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_server.server_id is defined
  - lineinfile:
      line: 'testing_tosca_server_delete: {{ testing_tosca_server.server.id }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_server.server.id is defined
  - lineinfile:
      line: '{{ testing_tosca_server_list | to_nice_yaml }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_server_list is defined
  - fail:
      msg: Variable testing_tosca_server is undefined! So it will not be deleted
    ignore_errors: true
    when: testing_tosca_server_list is undefined and testing_tosca_server.id is undefined
- hosts: localhost
  name: 'Create OpenStack component openstack cluster: testing_tosca_floating_ip:create'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Create OpenStack component floating ip
    os_floating_ip:
      floating_ip_address: 10.100.150.11
      network: ispras
      server: testing_tosca_{{ item }}
    register: testing_tosca_floating_ip
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      ansible_user: ubuntu
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      group: testing_tosca_server_public_address
    with_sequence: start=1 end=3 format=%d
  - set_fact:
      host_ip: '{{ host_ip | default([]) + [[ "testing_tosca_public_address_" + item,
        testing_tosca_floating_ip.results[item | int - 1].floating_ip.floating_ip_address
        ]] }}'
    with_sequence: start=1 end=3 format=%d
  - include: /tmp/clouni/artifacts/add_host.yaml
  - register: tmp
    set_fact:
      testing_tosca_floating_ip_dict:
        floating_ip_address: '{{ item.floating_ip.floating_ip_address }}'
        purge: 'yes'
        server: '{{ item.floating_ip.port_details.device_id }}'
        state: absent
    when: testing_tosca_floating_ip is defined
    with_items: '{{ testing_tosca_floating_ip.results }}'
  - set_fact:
      testing_tosca_floating_ip_dicts: '{{ testing_tosca_floating_ip_dicts | default([])
        + [item.ansible_facts.testing_tosca_floating_ip_dict] }}'
    when: testing_tosca_floating_ip_dict is defined
    with_items: '{{ tmp.results }}'
  - lineinfile:
      line: 'testing_tosca_floating_ip_dicts:

        {{ testing_tosca_floating_ip_dicts | to_nice_yaml }}'
      path: /home/ubuntu/id_vars_test.yaml
    when: testing_tosca_floating_ip_dicts is defined
- hosts: testing_tosca_server_public_address
  name: 'Create OpenStack component openstack cluster: software_for_cumulus_software_component:create'
  tasks:
  - set_fact:
      test: test
  - include: /tmp/clouni/artifacts/ansible-operation-example.yaml
~~~
## Удаление
Чтобы создать и запустить плейбуки для удаления того же шаблона можно использовать модуль delete или же просто указать флаг ```--delete```. Они будут выполняться так же параллельно, с учетом всех графовых зависимостей.
Если вы хотите только сгенерировать плейбуки, не запуская их - укажите ключ ```--debug```.
~~~shell
clouni delete --template-file examples/tosca-server-example-scalable.yaml --provider openstack --cluster-name test
or
clouni deploy --template-file examples/tosca-server-example-scalable.yaml --provider openstack --cluster-name test --delete
~~~
Файл с плейбуками для удаления облачных ресурсов:
~~~yaml
- hosts: localhost
  name: 'Delete OpenStack component openstack cluster: testing_tosca_floating_ip:delete'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Delete OpenStack component floating ip
    os_floating_ip:
      name: '{{ testing_tosca_floating_ip_delete }}'
      state: absent
    register: testing_tosca_floating_ip_var
    when: testing_tosca_floating_ip_delete is defined
  - set_fact: testing_tosca_floating_ip='{{ testing_tosca_floating_ip_var }}'
    when: testing_tosca_floating_ip_var.changed
  - loop: '{{ testing_tosca_floating_ip_ids | flatten(levels=1) }}'
    name: Delete OpenStack component floating ip
    os_floating_ip:
      name: '{{ item }}'
      state: absent
    register: testing_tosca_floating_ip_var
    when: testing_tosca_floating_ip_ids is defined
  - set_fact: testing_tosca_floating_ip='{{ testing_tosca_floating_ip_var }}'
    when: testing_tosca_floating_ip_var.changed
  - loop: '{{ testing_tosca_floating_ip_dicts }}'
    name: Delete OpenStack component floating ip
    os_floating_ip: '{{ item }}'
    register: testing_tosca_floating_ip_var
    when: testing_tosca_floating_ip_dicts is defined
  - set_fact: testing_tosca_floating_ip='{{ testing_tosca_floating_ip_var }}'
    when: testing_tosca_floating_ip_var.changed
- hosts: localhost
  name: 'Delete OpenStack component openstack cluster: testing_tosca_server:delete'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Delete OpenStack component server
    os_server:
      name: '{{ testing_tosca_server_delete }}'
      state: absent
    register: testing_tosca_server_var
    when: testing_tosca_server_delete is defined
  - set_fact: testing_tosca_server='{{ testing_tosca_server_var }}'
    when: testing_tosca_server_var.changed
  - loop: '{{ testing_tosca_server_ids | flatten(levels=1) }}'
    name: Delete OpenStack component server
    os_server:
      name: '{{ item }}'
      state: absent
    register: testing_tosca_server_var
    when: testing_tosca_server_ids is defined
  - set_fact: testing_tosca_server='{{ testing_tosca_server_var }}'
    when: testing_tosca_server_var.changed
  - loop: '{{ testing_tosca_server_dicts }}'
    name: Delete OpenStack component server
    os_server: '{{ item }}'
    register: testing_tosca_server_var
    when: testing_tosca_server_dicts is defined
  - set_fact: testing_tosca_server='{{ testing_tosca_server_var }}'
    when: testing_tosca_server_var.changed
- hosts: localhost
  name: 'Delete OpenStack component openstack cluster: testing_tosca_security_group:delete'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Delete OpenStack component security group
    os_security_group:
      name: '{{ testing_tosca_security_group_delete }}'
      state: absent
    register: testing_tosca_security_group_var
    when: testing_tosca_security_group_delete is defined
  - set_fact: testing_tosca_security_group='{{ testing_tosca_security_group_var }}'
    when: testing_tosca_security_group_var.changed
  - loop: '{{ testing_tosca_security_group_ids | flatten(levels=1) }}'
    name: Delete OpenStack component security group
    os_security_group:
      name: '{{ item }}'
      state: absent
    register: testing_tosca_security_group_var
    when: testing_tosca_security_group_ids is defined
  - set_fact: testing_tosca_security_group='{{ testing_tosca_security_group_var }}'
    when: testing_tosca_security_group_var.changed
  - loop: '{{ testing_tosca_security_group_dicts }}'
    name: Delete OpenStack component security group
    os_security_group: '{{ item }}'
    register: testing_tosca_security_group_var
    when: testing_tosca_security_group_dicts is defined
  - set_fact: testing_tosca_security_group='{{ testing_tosca_security_group_var }}'
    when: testing_tosca_security_group_var.changed
- hosts: localhost
  name: 'Delete OpenStack component openstack cluster: testing_tosca_keypair:delete'
  tasks:
  - include_vars: /home/ubuntu/id_vars_test.yaml
  - name: Delete OpenStack component keypair
    os_keypair:
      name: '{{ testing_tosca_keypair_delete }}'
      state: absent
    register: testing_tosca_keypair_var
    when: testing_tosca_keypair_delete is defined
  - set_fact: testing_tosca_keypair='{{ testing_tosca_keypair_var }}'
    when: testing_tosca_keypair_var.changed
  - loop: '{{ testing_tosca_keypair_ids | flatten(levels=1) }}'
    name: Delete OpenStack component keypair
    os_keypair:
      name: '{{ item }}'
      state: absent
    register: testing_tosca_keypair_var
    when: testing_tosca_keypair_ids is defined
  - set_fact: testing_tosca_keypair='{{ testing_tosca_keypair_var }}'
    when: testing_tosca_keypair_var.changed
  - loop: '{{ testing_tosca_keypair_dicts }}'
    name: Delete OpenStack component keypair
    os_keypair: '{{ item }}'
    register: testing_tosca_keypair_var
    when: testing_tosca_keypair_dicts is defined
  - set_fact: testing_tosca_keypair='{{ testing_tosca_keypair_var }}'
    when: testing_tosca_keypair_var.changed
- hosts: localhost
  name: Renew id_vars_example.yaml
  tasks:
  - file:
      path: /home/ubuntu/id_vars_test.yaml
      state: absent
~~~

## Обращение только к provider tool
Вы можете обращаться напрямую к отдельным микросервисам, например указав модуль provider_tool вы получите внутреннее представление TOSCA шаблона для выбранного облачного провайдера. 
Это может быть полезно для отладки ваших шаблонов или же для хранения. 

**ВАЖНО!** Чтобы получить вывод в файл используйте опцию ```--output-file```, чтобы получить вывод в stdin - опцию ```--debug```, иначе вывода не будет.
~~~shell
clouni provider_tool --template-file examples/tosca-server-example-scalable.yaml --provider openstack --cluster-name test --debug --output-file test.yaml
~~~
Файл с внутренним представлением:
~~~yaml
topology_template:
  node_templates:
    software_for_cumulus_software_component:
      interfaces:
        Standard:
          create:
            implementation: ansible-operation-example.yaml
            inputs: {test: test}
      requirements:
      - host: {node: testing_tosca_server}
      - dependency: {node: testing_tosca_security_group_rule}
      - dependency: {node: testing_tosca_floating_ip}
      - dependency: {node: testing_tosca_keypair}
      - dependency: {node: testing_tosca_security_group}
      - dependency: {node: testing_tosca_server}
      type: tosca.nodes.SoftwareComponent
    testing_tosca_floating_ip:
      interfaces:
        Standard:
          create:
            implementation: add_host.yaml
            inputs: {ansible_user: ubuntu, group: testing_tosca_server_public_address,
              host_ip: '{{ host_ip | default([]) + [[ "testing_tosca_public_address_"
                + item, testing_tosca_floating_ip.results[item | int - 1].floating_ip.floating_ip_address
                ]] }}'}
      properties: {floating_ip_address: 10.100.150.11}
      requirements:
      - server: {node: testing_tosca_server}
      - network:
          node_filter:
            properties:
            - {name: ispras}
      type: openstack.nodes.FloatingIp
    testing_tosca_keypair:
      properties: {name: testing_tosca_keypair, public_key: '{{ lookup(''file'', ''~/.ssh/id_rsa.pub'')
          }}'}
      type: openstack.nodes.Keypair
    testing_tosca_security_group:
      properties: {name: testing_tosca_security_group}
      type: openstack.nodes.SecurityGroup
    testing_tosca_security_group_rule:
      properties: {direction: '{{ initiator[item | int] | default(omit) }}', port_range_max: '{{
          port[item | int] | default(omit) }}', port_range_min: '{{ port[item | int]
          | default(omit) }}', protocol: '{{ protocol[item | int] | default(omit)
          }}', remote_ip_prefix: 0.0.0.0/0}
      requirements:
      - security_group: {node: testing_tosca_security_group}
      type: openstack.nodes.SecurityGroupRule
    testing_tosca_server:
      interfaces:
        Standard:
          create:
            implementation: add_host.yaml
            inputs: {ansible_user: ubuntu, group: testing_tosca_server_private_address,
              host_ip: '{{ host_ip | default([]) + [[ "testing_tosca_private_address_"
                + item, testing_tosca_server.results[item | int - 1].server.public_v4
                ]] }}'}
      properties:
        auto_ip: false
        meta: cube_master=true
        name: testing_tosca_{{ item }}
        nics:
        - {net-name: test_ext}
      requirements:
      - key_name: {node: testing_tosca_keypair}
      - flavor:
          node_filter:
            properties:
            - {vcpus: 1}
            - {disk: 10.0}
            - {ram: 1024.0}
      - security_groups: {node: testing_tosca_security_group}
      - image:
          node_filter:
            properties:
            - {name: Ubuntu-Server-18.04-LTS-Bionic-Beaver}
      type: openstack.nodes.Server
tosca_definitions_version: tosca_simple_yaml_1_0
~~~
Данное представление можно сохранить, внести правки (если необходимо), и подавать на вход в configuration tool для развертывания.

## Обращение только к configuration tool
Вы можете обращаться напрямую к отдельным микросервисам, например указав модуль configuration_tool вы сможете развернуть виртуальную инфраструктуру 
по заданному провайдерскому представлению (см. выше) или получить Ansible плейбуки при указании флага ```--debug```.
~~~shell
clouni configuration_tool --template-file test.yaml --provider openstack --cluster-name test
~~~
## Важная информация
**ВНИМАНИЕ!** Большинству провайдеров требуется аутентификация для использования облачных ресурсов.
Аутентификация - это ответственность пользователей. Например, чтобы использовать OpenStack, вы должны загрузить свой RC-файл OpenStack, либо же использовать файл clouds.yaml на сервере где запущена grpc-cotea.
После этого вы можете пользоваться Clouni.

Во время выполнения Clouni не отправляет и не получает никакой информации из облака или Интернета.

Во время выполнения Clouni делает только 2 шага:
1. Получить информацию об облаке и выбрать конкретные параметры облака для удовлетворения заданных требований.
2. Создать облачные ресурсы.
Например, для создания образа сервера в облачной среде OpenStack необходимо указать имя и тип образа.

Для конфигурации програмнного обеспечения на виртуальных серверах Clouni создает ключевую пару используя путь к публичному ключу, который вы укажете в ```--public-key-path```. Важно что этот ключ должен находиться на сервере где запущена grpc-cotea.