# Installation 

## Clouni client
Clouni requires Python 3.6+ to be used.

To install Clouni client is recommended to create virtual environment and install
requirements in your environment.

~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~

With pip (without easy install scripts):
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-client
~~~
Or:
~~~shell
git clone https://github.com/sadimer/clouni_client
~~~
It's recommended not to use `$CLOUNI_HOME` as your `$VIRTUALENV_HOME`.
~~~shell
cd $CLOUNI_HOME
pip install -r requirements.txt
python setup.py install
~~~

## Easy install
After installing the client, you can install all the necessary microservices (except the storage API) by running just **one command!**

**Steps for installation on remote server:**

Fill the hosts.ini file like this:
~~~ini
[provider_tool_host]
1.1.1.1 ansible_user=ubuntu
[configuration_tool_host]
2.2.2.2 ansible_user=ubuntu
[grpc_cotea_host]
3.3.3.3 ansible_user=ubuntu
~~~
Then run:
~~~shell
pip install ansible==2.9.19
ansible-galaxy collection install community.crypto
cd ansible_installer
ansible-playbook main.yaml -i hosts.ini
~~~

**IMPORTANT!** For authentication in the cloud based on OpenStack Clouni uses the clouds.yaml file. You can download it from your cloud provider site. It is necessary to place this file in /etc/openstack dir on the client's computer (in the case of easy install or docker install), or on a server with grpc cotea. The same logic applies to the ~/.aws/credentials and ~/.aws/config files when using the Amazon AWS provider.
**Do not forget to enter your password/token in this file.**

**For local deployment:**

Replace addresses to 127.0.0.1 or localhost in hosts.ini. Example:
~~~ini
[provider_tool_host]
localhost ansible_connection=local
~~~

## Docker install
Run following command:
~~~shell
cd compose_installer
docker-compose up -d
~~~
You can also install each of the services in Docker separately:
~~~shell
docker run --name clouni-provider-tool -d -p <ip_address>:<port>:50051 clouni/clouni-provider-tool:latest
docker run --name clouni-configuration-tool -d -p <ip_address>:<port>:50052 clouni/clouni-configuration-tool:latest
docker run --name grpc-cotea -d -p <ip_address>:<port>:50151 clouni/grpc-cotea:latest
~~~

Or build an image from Dockerfile (located in the root folder of services):
~~~shell
cd $CLOUNI_PROVIDER_TOOL_HOME
docker build -t clouni-provider-tool - < dockerfile
cd $CLOUNI_CONFIGURATION_TOOL_HOME
docker build -t clouni-configuration-tool - < dockerfile
~~~

## Clouni provider tool
Clouni provider tool requires Python 3.6+ to be used.

To install Clouni provider tool is recommended to create virtual environment and install
requirements in your environment.

The user can install the services manually.
To do this, run the following commands on the Clouni provider tool server.
~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~
With pip:
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-provider-tool
~~~
Or:
~~~shell
git clone https://github.com/sadimer/clouni_provider_tool
cd $CLOUNI_PROVIDER_TOOL_HOME
pip install -r requirements.txt
python setup.py install
~~~
Now you can start the server:
~~~shell
clouni-provider-tool --host 0.0.0.0 -p 5000
~~~
## Clouni configuration tool
Clouni configuration tool requires Python 3.6+ to be used.

To install Clouni configuration tool is recommended to create virtual environment and install
requirements in your environment.

The user can install the services manually.
To do this, run the following commands on the Clouni configuration tool server.
~~~shell
python3 -m venv $VIRTUALENV_HOME/clouni
source $VIRTUALENV_HOME/clouni/bin/activate
~~~
With pip:
~~~shell
pip install --extra-index-url https://test.pypi.org/simple/ clouni-configuration-tool
~~~
Or:
~~~shell
git clone https://github.com/sadimer/clouni_configuration_tool
cd $CLOUNI_CONFIGURATION_TOOL_HOME
pip install -r requirements.txt
python setup.py install
~~~
Now you can start the server:
~~~shell
clouni-configuration-tool --host 0.0.0.0 -p 5000
~~~
## Grpc cotea
The user can install the services manually.
Grpc-cotea requires Python 3.8+ to be used.
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
Now you can start the server:
~~~shell
nohup grpc-cotea --host { please insert here host ip address } -p 5000 &
~~~
Create directory for additional modules and artifacts (custom Ansible scripts):
~~~shell
mkdir -p /tmp/clouni/
~~~
**Don't forget about authentication!** Most of providers require authentication for using there resources.
Authentication is users responsibility. For example, to use OpenStack
you must download your OpenStack RC file and *source* it or use the clouds.yaml file on the server where grpc-cotea is installed. The same logic applies to the ~/.aws/credentials and ~/.aws/config files when using the Amazon AWS provider.

For software configuration on virtual servers, Clouni creates a key pair using the path to the key on the server where grpc-cotea is installed that you specify in ```--public-key-path```.
For creating key run:
```shell
ssh-keygen
```
## Database API
The installation guide is located in [README.md](https://github.com/DYDKA4/API_COURSE).

## Logs
Logs of each of the microservices can be found here:
1) provider tool - **$HOME/.clouni-provider-tool.log**
2) configuration tool - **$HOME/.clouni-configuration-tool.log**
3) grpc-cotea - **$HOME/cotea.log**