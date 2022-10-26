# About

## Clouni
**Cloud Unifier Tool for Service Orchestration**

Clouni is a cloud application management tool based on OASIS standard
[TOSCA](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/TOSCA-Simple-Profile-YAML-v1.0.html>).

Main features of the Clouni multicloud orchestrator:
1) Independence of the used cloud platform. Currently, Openstack, Amazon AWS and partially Kubernetes are supported, Google cloud and other platforms are planned. 
2) Fine-tuning the parameters of virtual machines, security groups, ports and networks. The ability to execute custom Ansible scripts described in TOSCA interface operations during deployment. With this, user can even configure software on virtual machines.
3) A microservice architecture that allows to separate the processes of translating normative TOSCA templates into an internal representation, storing the state of deployed clusters, generating Ansible scripts and deployment control.
4) Parallel deployment of independent nodes in compliance with the graph hierarchy described in TOSCA Simple Profile.
5) TOSCA Simple Profile v1.0 and partially v1.3 are supported.

Now all the components of Clouni can be found at the links:

[Client](https://github.com/sadimer/clouni_client)

[Provider tool](https://github.com/sadimer/clouni_provider_tool)

[Configuration tool](https://github.com/sadimer/clouni_configuration_tool)

[Grpc cotea](https://test-files.pythonhosted.org/packages/22/24/fd0160b1dfcfcc21e6c2d2a263e60e9c46787b68c565fa8156af41cc5e92/grpccotea-0.9.3-py3-none-any.whl)

[Database API](https://github.com/DYDKA4/API_COURSE)

Unsupported, but working monolithic version:

[Clouni (deprecated version)](https://github.com/ispras/clouni)