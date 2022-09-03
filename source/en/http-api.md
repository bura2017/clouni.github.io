# HTTP Restful API server

Simple one method microservice to control Clouni over HTTP.

## Start server

~~~shell
pip install -r requirements-rest.txt
export FLASK_APP=$CLOUNI_HOME/rest_clouni/app.py 
  && export FLASK_DEBUG=0 
  && export FLASK_ENV=development 
  && cd $CLOUNI_HOME 
  && python -m flask run --host=0.0.0.0
~~~

## API

Server accept POST-request for route `/`. Payload must be JSON-formatted. 
JSON must contain arguments for `translate()` function of the `toskatranslator`.

JSON format:
~~~JSON
{
  "template_file": "<TOSCA template (required)>",
  "validate_only": "<Boolean default 'False'>",
  "provider": "<'amazon' | 'openstack' | 'kubernetes'>",
  "configuration_tool": "ansible",
  "cluster_name": "<String>",
  "is_delete": "<Boolean default 'False'>",
  "extra": "<JSON default '{}'>",
  "log_level": "<Python logging.* log levels, e.g. 'debug'>",
  "host_ip_parameter": "'public_address' | 'private_address'",
  "debug": "<Boolean default 'False'>"
}
~~~

Response format:
- if `debug` mode off - just `toskatranslator` output as is.
- if `debug` mode on - JSON, containing info about all files generated:
~~~JSON
{
  "<File name>": "<File content>"
}
~~~

### Example

Request:

~~~JSON
{
	"template_file": "tosca_definitions_version: tosca_simple_yaml_1_0\n\ntopology_tem...",
	"configuration_tool": "ansible",
	"cluster_name": "qwerty",
	"extra": "{}",
	"is_delete": "True",
	"debug": "True"
}
~~~

Response:

~~~JSON
{
  "contains.yaml": "# input_facts and input_args are defined\n\n- name: Initialize input argument argv_0\n...",
  "equals.yaml": "# input_facts and input_args are defined\n# matched_object is required\n\n- ...",
  "playbook.yml": "- name: 'Delete OpenStack component openstack cluster: server_kube_master_serve..."
}
~~~

### Example (Parse error)

Request:

~~~JSON
{
	"template_file": "...",
	"provider": "AWS",
	"configuration_tool": "ansible",
	"cluster_name": "qwerty",
	"extra": "{}",
	"is_delete": "True",
	"debug": "13"
}
~~~

Response:

~~~JSON
{
	"debug": [
		"Not a valid boolean."
	],
	"provider": [
		"Must be one of: amazon, common, kubernetes, openstack."
	]
}
~~~

## Dockerfile

*clouni-server* can be started as container with help of Docker.
From *dockerfile* you can get image for clouni-server container:
~~~shell
cd $CLOUNI_HOME/rest_clouni
docker build -t clouni-api .
~~~
Then start the server on *IP_address:port* you needed:
~~~shell
docker run --name clouni-api-1 -itd -p IP_address:port:5000 clouni-api
~~~
