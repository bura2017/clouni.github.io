# О проекте
## Clouni
**Cloud Unifier Tool for Service Orchestration**

Clouni - это инструмент управдния облачными приложениями, основанный на стандарте OASIS
[TOSCA](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/TOSCA-Simple-Profile-YAML-v1.0.html).

Основные характеристики мультиоблачного оркестратора Clouni:
1) Отсутствие зависимости от используемой облачной платформы. Сейчас поддерживается Openstack, Amazon AWS и частично Kubernetes, в планах - Google cloud и другие платформы. 
2) Тонкая настройка параметров виртуальных машин, групп безопасности, портов и сетей. Возможность выполнения в ходе развертывания пользовательских Ansible сценариев описанных в TOSCA interface operations, в том числе для настройки ПО на виртуальных машинах.
3) Микросервисная архитектура, позволяющая разделить процессы трансляции нормативных TOSCA шаблонов во внутреннее представление, хранения состояния разворачиваемых кластеров, генерации Ansible сценариев и контроля их выполнения.
4) Параллельное развертывание независимых узлов с соблюдением графовой иерархии описанной в TOSCA Simple Profile.
5) Поддержка TOSCA Simple Profile v1.0 и частично v1.3.

Сейчас все компоненты Clouni можно найти по ссылкам:

[Клиент](https://github.com/sadimer/clouni_client)

[Provider tool](https://github.com/sadimer/clouni_provider_tool)

[Configuration tool](https://github.com/sadimer/clouni_configuration_tool)

[Grpc cotea](https://test-files.pythonhosted.org/packages/22/24/fd0160b1dfcfcc21e6c2d2a263e60e9c46787b68c565fa8156af41cc5e92/grpccotea-0.9.3-py3-none-any.whl)

[Database API](https://github.com/DYDKA4/API_COURSE)

Неподдерживаемая, но рабочая монолитная версия:

[Clouni (deprecated version)](https://github.com/ispras/clouni)