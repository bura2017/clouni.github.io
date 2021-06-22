# Установка

Для установки потребуется Python версии 3.6

Рекомендуется использовать виртуальное окружение для облегчения установки зависимостей.
~~~shell
python3.6 -m venv $VIRTUALENV_HOME/clouni
~~~
Не рекомендутся использовать `$CLOUNI_HOME` в качестве `$VIRTUALENV_HOME`.

Если вы планируете использовать виртуальное окружение для выполнения playbook-ов, установите зависимости Clouni.

~~~shell
source $VIRTUALENV_HOME/clouni/bin/activate
cd $CLOUNI_HOME
pip install -r requirements.txt
~~~

Для установки требуется OpenStack TOSCA Parser. Текущая версия не имеет необходимых изменений, используйте временный [fork repository](https://github.com/bura2017/tosca-parser.git).

~~~
cd $CLOUNI_HOME/../
git clone https://github.com/bura2017/tosca-parser.git
cd tosca-parser
git checkout develop
pip install -r requirements.txt
python setup.py install
~~~

Команда установки
~~~shell
cd $CLOUNI_HOME
python setup.py install
~~~
