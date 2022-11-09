# Установка

## Clouni client
Для установки клиента рекомендуется Python версии 3.6.9

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.
~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~

С помощью pip (без скриптов easy install):
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-client
~~~
Или:
~~~shell
git clone https://github.com/sadimer/clouni_client
~~~
Не рекомендутся использовать `$CLOUNI_HOME` в качестве `$VIRTUALENV_HOME`.
~~~shell
cd $CLOUNI_HOME
pip install -r requirements.txt
python setup.py install
~~~

## Easy install
Сразу после установки клиента можно установить все необходимые микросервисы (кроме API для хранения) запустив всего одну команду!

**Для удаленной установки:**

Заполните файл hosts.ini подобным образом:
~~~ini
[provider_tool_host]
1.1.1.1 ansible_user=ubuntu
[configuration_tool_host]
2.2.2.2 ansible_user=ubuntu
[grpc_cotea_host]
3.3.3.3 ansible_user=ubuntu
~~~
Затем выполните:
~~~shell
pip install ansible==2.9.19
ansible-galaxy collection install community.crypto
cd ansible_installer
ansible-playbook main.yaml -i hosts.ini
~~~

**ВАЖНО!** Для аутентификации в облаке на базе OpenStack Clouni использует файл clouds.yaml, его вы можете скачать у своего облачного провайдера. Необходимо разместить этот файл в /etc/openstack либо на компьютере клиента (в случае установки easy install, либо docker install), либо на сервере с grpc cotea. **Не забывайте вписать в него свой пароль/токен.** Аналогичная логика применима для файлов ~/.aws/credentials и ~/.aws/config в случае использования провайдера Amazon AWS.

**Для локальной установки:**

Замените все адреса на 127.0.0.1 или localhost в файле hosts.ini.
Пример:
~~~ini
[provider_tool_host]
localhost ansible_connection=local
~~~

## Docker install
Выполните:
~~~shell
cd compose_installer
docker-compose up -d
~~~
Также вы можете установить каждый из сервисов в Docker по отдельности:
~~~shell
docker run --name clouni-provider-tool -d -p <ip_address>:<port>:50051 clouni/clouni-provider-tool:latest
docker run --name clouni-configuration-tool -d -p <ip_address>:<port>:50052 clouni/clouni-configuration-tool:latest
docker run --name grpc-cotea -d -p <ip_address>:<port>:50151 clouni/grpc-cotea:latest
~~~

Или же собрать образ из Dockerfile (находятся в корневой папке любого из сервисов):
~~~shell
cd $CLOUNI_PROVIDER_TOOL_HOME
docker build -t clouni-provider-tool - < dockerfile
cd $CLOUNI_CONFIGURATION_TOOL_HOME
docker build -t clouni-configuration-tool - < dockerfile
~~~

## Clouni provider tool
Для установки provider tool требуется Python версии 3.6+

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.

Можно и не использовать скрипт из easy install и установить сервисы вручную.
Для этого на сервере, на котором вы хотите установить Clouni provider tool выполните следующее:

~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~
С помощью pip:
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-provider-tool
~~~
Или:
~~~shell
git clone https://github.com/sadimer/clouni_provider_tool
cd $CLOUNI_PROVIDER_TOOL_HOME
pip install -r requirements.txt
python setup.py install
~~~
Теперь можно запустить сервер:
~~~shell
clouni-provider-tool --host 0.0.0.0 -p 5000
~~~
## Clouni configuration tool
Для установки configuration tool требуется Python версии 3.6+

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.

Можно и не использовать скрипт из easy install и установить сервисы вручную.
Для этого на сервере, на котором вы хотите установить Clouni configuration tool выполните следующее:

~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~
С помощью pip:
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-configuration-tool
~~~
Или:
~~~shell
git clone https://github.com/sadimer/clouni_configuration_tool
cd $CLOUNI_CONFIGURATION_TOOL_HOME
pip install -r requirements.txt
python setup.py install
~~~
Теперь можно запустить сервер:
~~~shell
clouni-configuration-tool --host 0.0.0.0 -p 5000
~~~
## Grpc cotea
Можно и не использовать скрипт из easy install и установить сервисы вручную.
Для установки grpc-cotea требуется Python версии 3.8+.
~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
pip install grpcio==1.37.1
pip install grpcio-tools==1.37.1
pip install -i https://test.pypi.org/simple/ grpc-cotea==0.9.5
pip install ansible==2.9.19
pip install openstacksdk==0.46.1
pip install boto3 botocore urllib3 bs4
~~~
Теперь можно запустить сервер:
~~~shell
nohup grpc-cotea --host { please insert here host ip address } -p 5000 &
~~~
Создайте временную директорию для дополнительных модулей и артефактов (пользовательских скриптов):
~~~shell
mkdir -p /tmp/clouni/
~~~
**Не забывайте об аутентификации!** Большинству провайдеров требуется аутентификация для использования облачных ресурсов.
Аутентификация - это ответственность пользователей. Например, чтобы использовать OpenStack, вы должны загрузить свой RC-файл OpenStack, либо же использовать файл clouds.yaml на сервере где запущена grpc-cotea. Аналогичная логика применима для файлов ~/.aws/credentials и ~/.aws/config в случае использования провайдера Amazon AWS.

Для конфигурации програмнного обеспечения на виртуальных серверах Clouni создает ключевую пару используя путь к публичному ключу, который вы укажете в ```--public-key-path```. Важно что этот ключ должен находиться на сервере где запущена grpc-cotea.
Для его генерации можно использовать команду:
```shell
ssh-keygen
```

## Database API
Гайд по установке находится в README.md [здесь](https://github.com/DYDKA4/API_COURSE).

## Logs
Логи каждого из микросервисов находятся здесь:
1) provider tool - **$HOME/.clouni-provider-tool.log**
2) configuration tool - **$HOME/.clouni-configuration-tool.log**
3) grpc-cotea - **$HOME/cotea.log**