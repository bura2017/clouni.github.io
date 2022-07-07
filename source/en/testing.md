# Testing

# Openstack
To run Clouni tests for an Openstack-based cloud, you need to download the RC file for your Openstack cloud and 'source' it, or download the clouds.yaml file and put it in /etc/openstack.
Next, you need to run the following command:

~~~shell
python -m unittest testing.test_openstack in $CLOUNI_HOME
~~~

# Amazon
To run Clouni tests for an Amazon AWS-based cloud, run the following command:

~~~shell
python -m unittest testing.test_amazon in $CLOUNI_HOME
~~~


# Kubernetes
To run Clouni tests as a Kubernetes cluster management tool, run the following command:

~~~shell
python -m unittest testing.test_kubernetes in $CLOUNI_HOME
~~~