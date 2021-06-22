# Installation
Clouni requires Python 3.6 to be used.

To install Clouni is recommended to create virtual environment and install
requirements in your environment.
~~~shell
python3.6 -m venv $VIRTUALENV_HOME/clouni
~~~
It's recommended not to use `$CLOUNI_HOME` as your `$VIRTUALENV_HOME`

Install Clouni requirements in you virtual environment if you will use this virtual
environment to execute playbooks.

~~~shell
source $VIRTUALENV_HOME/clouni/bin/activate
cd $CLOUNI_HOME
pip install -r requirements.txt
~~~

Install required OpenStack TOSCA Parser in your virtual environment. Current version
of TOSCA Parser doesn't have required changes. Use temporary [fork repository](https://github.com/bura2017/tosca-parser.git).

~~~
cd $CLOUNI_HOME/../
git clone https://github.com/bura2017/tosca-parser.git
cd tosca-parser
git checkout develop
pip install -r requirements.txt
python setup.py install
~~~

Installation command
~~~shell
cd $CLOUNI_HOME
python setup.py install
~~~
