# Установка

## Clouni client
Для установки клиента рекомендуется Python версии 3.6+
~~~shell
git clone https://github.com/sadimer/clouni_client
~~~
Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.
~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
~~~
Не рекомендутся использовать `$CLOUNI_HOME` в качестве `$VIRTUALENV_HOME`.
~~~shell
source $VIRTUALENV_HOME/clouni/bin/activate
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
ansible-playbook main.yaml -i hosts.ini
~~~

**Для локальной установки:**

Замените все адреса на 127.0.0.1 или localhost в файле hosts.ini.

## Clouni provider tool
Для установки provider tool требуется Python версии 3.6.

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.

Можно и не использовать скрипт из easy install и установить сервисы вручную.
Для этого на сервере, на котором вы хотите установить Clouni provider tool выполните следующее:
~~~shell
git clone https://github.com/bura2017/tosca-parser.git
git clone https://github.com/sadimer/clouni_provider_tool
python3.6 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
cd tosca-parser
git checkout develop
pip install -r requirements.txt
python setup.py install
cd $CLOUNI_PROVIDER_TOOL_HOME
pip install -r requirements.txt
python setup.py install
~~~
Теперь можно запустить сервер:
~~~shell
clouni-provider-tool --host 0.0.0.0 -p 5000
~~~
## Clouni configuration tool
Для установки configuration tool требуется Python версии 3.6.

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.

Можно и не использовать скрипт из easy install и установить сервисы вручную.
Для этого на сервере, на котором вы хотите установить Clouni configuration tool выполните следующее:
~~~shell
git clone https://github.com/bura2017/tosca-parser.git
git clone https://github.com/sadimer/clouni_configuration_tool
python3.6 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
cd tosca-parser
git checkout develop
pip install -r requirements.txt
python setup.py install
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
Для установки grpc-cotea требуется Python версии 3.8.
~~~shell
python3.8 -m venv $VIRTUALENV_HOME/clouni
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
Создайте директории для дополнительных модулей и артефактов (пользовательских скриптов):
~~~shell
mkdir -p /tmp/clouni/ansible
mkdir -p /tmp/clouni/artifacts
~~~
**Не забывайте об аутентификации!** Большинству провайдеров требуется аутентификация для использования облачных ресурсов.
Аутентификация - это ответственность пользователей. Например, чтобы использовать OpenStack, вы должны загрузить свой RC-файл OpenStack, либо же использовать файл clouds.yaml на сервере где запущена grpc-cotea.

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