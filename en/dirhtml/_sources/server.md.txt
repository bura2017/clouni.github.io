# Client-server model
## Usage of server

You need to implement client using specification
located at *toscatranslator/api.proto*, or use CLI-client *clouni-client*

Supported clouni request options are identical to the CLI version
except not supported `--output-file` option and `--template-file` option,
replaced by `template_file_content`

Log file is `./.clouni-server.log`

~~~shell
clouni-server --help
~~~
Output:
~~~
usage: clouni-server [-h] [--max-workers <number of workers>]
                     [--host <host_name/host_address>] [--port <port>]
                     [--verbose] [--no-host-error] [--stop] [--foreground]

optional arguments:
  -h, --help            show this help message and exit
  --max-workers <number of workers>
                        Maximum of working gRPC threads, default 10
  --host <host_name/host_address>
                        Hosts on which server will be started, may be more
                        than one, default [::]
  --port <port>, -p <port>
                        Port on which server will be started, default 50051
  --verbose, -v         Logger verbosity, default -vvv
  --no-host-error       If unable to start server on host:port and this option
                        used, warning will be logged instead of critical error
  --stop                Stops all working servers and exit
  --foreground          Makes server work in foreground
~~~
### Starting server
~~~shell
clouni-server --max-workers 20 --host 127.0.0.1 --host 20.20.20.20 --port 50051 -vv --no-host-error
~~~
Server will be started on 127.0.0.1:50051 with 'warning' logger verbosity level
Warning about unability to start server on 20.20.20.20 will be logged

By default, server works in background
## Usage of CLI gRPC client
~~~shell
clouni-client --help
~~~
Output:
~~~
usage: clouni-client [-h] --template-file <filename> --cluster-name
                     CLUSTER_NAME [--validate-only] [--delete]
                     [--provider PROVIDER] [--output-file <filename>]
                     [--configuration-tool CONFIGURATION_TOOL] [--async]
                     [--extra KEY=VALUE [KEY=VALUE ...]]
                     [--host <host_name/host_address>] [--port <port>]

optional arguments:
  -h, --help            show this help message and exit
  --template-file <filename>
                        YAML template to parse.
  --cluster-name CLUSTER_NAME
                        Cluster name
  --validate-only       Only validate input template, do not perform
                        translation.
  --delete              Delete cluster
  --provider PROVIDER   Cloud provider name to execute ansible playbook in.
  --output-file <filename>
                        Output file
  --configuration-tool CONFIGURATION_TOOL
                        Configuration tool which DSL the template would be
                        translated to. Default value = "ansible"
  --async               Provider nodes should be created asynchronously
  --extra KEY=VALUE [KEY=VALUE ...]
                        Extra arguments for configuration tool scripts
  --host <host_name/host_address>
                        Host of server, default localhost
  --port <port>, -p <port>
                        Port of server, default 50051
~~~

Usage is the same as *clouni* usage, with additional *host* and *port* options

## Examples of using server and client
~~~shell
clouni-server
clouni-client --template-file examples/tosca-server-example.yaml --cluster-name example --provider openstack
~~~
Output:
~~~
Server started
* Status *

OK

* Error *



* Content *

...

~~~


~~~shell
clouni-server
clouni-client --template-file examples/tosca-server-example.yaml --cluster-name example --provider kubernetes
~~~
Output:
~~~
* Status *

ERROR

* Error *

UnsupportedToscaParameterUsage: Unable to use unsupported TOSCA parameter: networks not supported in Kubernetes
		File /usr/local/lib/python3.6/threading.py, line 884, in _bootstrap
			self._bootstrap_inner()
		File /usr/local/lib/python3.6/threading.py, line 916, in _bootstrap_inner
			self.run()
		File /usr/local/lib/python3.6/threading.py, line 864, in run
			self._target(*self._args, **self._kwargs)
		File /usr/local/lib/python3.6/concurrent/futures/thread.py, line 69, in _worker
			work_item.run()
		File /usr/local/lib/python3.6/concurrent/futures/thread.py, line 56, in run
			result = self.fn(*self.args, **self.kwargs)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/grpc/_server.py, line 553, in _unary_response_in_pool
			argument, request_deserializer)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/grpc/_server.py, line 435, in _call_behavior
			response_or_iterator = behavior(argument, context)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/grpc_clouni/clouni_server.py, line 58, in Clouni
			response.content = TranslatorServer(args).output
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/grpc_clouni/clouni_server.py, line 39, in __init__
			extra={'global': self.extra}, a_file=False)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/common/translator_to_configuration_dsl.py, line 48, in translate
			tosca = ProviderToscaTemplate(tosca_parser_template_object, provider, cluster_name)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/providers/common/tosca_template.py, line 65, in __init__
			self.topology_template = self.translate_to_provider()
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/providers/common/tosca_template.py, line 384, in translate_to_provider
			self.tosca_elements_map_to_provider(), self.tosca_topology_template)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/providers/common/translator_to_provider.py, line 830, in translate
			tpl_structure = translate_node_from_tosca(restructured_mapping, element.name, self)
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/providers/common/translator_to_provider.py, line 483, in translate_node_from_tosca
			self=self
		File /home/winking-maniac/test/venv/clouni/lib/python3.6/site-packages/clouni-0.0.1-py3.6.egg/toscatranslator/providers/common/translator_to_provider.py, line 139, in restructure_value
			what=flat_mapping_value.get(REASON).format(self=self)


* Content *


~~~

## Dockerfile

*clouni-server* can be started as container with help of Docker.
From *dockerfile* you can get image for clouni-server container:
~~~
cd $CLOUNI_HOME
sudo docker build -t clouni - < dockerfile
~~~
Then start the server on *IP_address:port* you needed:
~~~
sudo docker run --name clouni-server -d -p IP_address:port:50051 clouni
~~~
