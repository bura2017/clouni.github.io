# Тестирование

## Openstack
Для запуска тестов Clouni для облака на базе Openstack необходимо скачать и установить в 'source' файл RC для вашего облака Openstack, либо скачать файл clouds.yaml и поместить его в /etc/openstack.
Далее необходимо запустить следующую команду:

~~~shell
python -m unittest testing.test_openstack in $CLOUNI_HOME
~~~

## Amazon
Для запуска тестов Clouni для облака на базе Amazon AWS необходимо запустить следующую команду:

~~~shell
python -m unittest testing.test_amazon in $CLOUNI_HOME
~~~


## Kubernetes
Для запуска тестов Clouni в качестве инструмента управления кластером Kubernetes необходимо запустить следующую команду:

~~~shell
python -m unittest testing.test_kubernetes in $CLOUNI_HOME
~~~