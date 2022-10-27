# Using Clouni over the gRPC protocol

Instead of using the standard Clouni client, the user can write their own. User can also use Clouni API to write their application.

To ensure this possibility, Clouni has gRPC API.
## Clouni provider tool and configuration tool
Proto file describing the Clouni API calls:
~~~protobuf
syntax = "proto3";

service ClouniProviderTool {
    rpc ClouniProviderTool(ClouniProviderToolRequest) returns (ClouniProviderToolResponse) {}
}

service ClouniConfigurationTool {
    rpc ClouniConfigurationTool(ClouniConfigurationToolRequest) returns (ClouniConfigurationToolResponse) {}
}

// ClouniProviderTool request
// Fields are specified in Clouni help

message ClouniProviderToolRequest {
    string template_file_content = 1;
    string cluster_name = 2;
    bool validate_only = 3;
    bool delete = 4;
    string provider = 5;
    string configuration_tool = 6;
    string extra = 7;
    string log_level = 8;
    bool debug = 9;
    string host_parameter = 10;
    string public_key_path = 11;
    string configuration_tool_endpoint = 12;
    string grpc_cotea_endpoint = 13;
    string database_api_endpoint = 14;
}

// ClouniProviderTool response
//      Status: TEMPLATE_VALID - returned for validate-only requests if template is valid
//              TEMPLATE_INVALID - returned for validate-only requests if template is invalid
//              OK - returned for normal execution of non-validate-only request
//              ERROR - returned if any error occured
//      Error: error description(only with ERROR status)
//      Content: content of proceeded template file(only with OK status)

message ClouniProviderToolResponse {
    enum Status {
        TEMPLATE_VALID = 0;
        TEMPLATE_INVALID = 1;
        OK = 2;
        ERROR = 3;
    }
    Status status = 1;
    string error = 2;
    string content = 3;
}

// ClouniProviderTool request
// Fields are specified in Clouni help

message ClouniConfigurationToolRequest {
    string provider_template = 1;
    string cluster_name = 2;
    bool validate_only = 3;
    bool delete = 4;
    string configuration_tool = 5;
    string extra = 6;
    string log_level = 7;
    bool debug = 8;
    string database_api_endpoint = 9;
    string grpc_cotea_endpoint = 10;
}

message ClouniConfigurationToolResponse {
    enum Status {
        TEMPLATE_VALID = 0;
        TEMPLATE_INVALID = 1;
        OK = 2;
        ERROR = 3;
    }
    Status status = 1;
    string error = 2;
    string content = 3;
}
~~~
## Grpc-cotea
Proto file describing the grpc-cotea API calls:
~~~protobuf
syntax = "proto3";

service CoteaGateway {
   rpc StartSession(EmptyMsg) returns (StartSessionMSG) {}
   rpc InitExecution(Config) returns (Status) {}
   rpc RunTask(Task) returns (TaskResults) {}
   rpc StopExecution(SessionID) returns (Status) {}
   // rpc RestartExecution(SessionID) returns (Status) {}
}

service CoteaWorker {
   rpc InitExecution(WorkerConfig) returns (Status) {}
   rpc RunTask(WorkerTask) returns (TaskResults) {}
   rpc StopExecution(EmptyMsg) returns (Status) {}
   rpc HealthCheck(EmptyMsg) returns (WorkerHealthStatus) {}
   // rpc RestartExecution() returns (Status) {}
}

message StartSessionMSG {
   bool ok = 1;
   string ID = 2;
   string error_msg = 3;
}

message SessionID {
   string session_ID = 1;
}

message EmptyMsg {}

message MapFieldEntry {
   string key = 1;
   string value = 2;
} 

message Config {
   string session_ID = 1;
   string hosts = 2;
   string inv_path = 3;
   string extra_vars = 4;
   repeated MapFieldEntry env_vars = 5;
   string ansible_library = 6;
   bool not_gather_facts = 7;
}

message WorkerConfig {
   string hosts = 1;
   string inv_path = 2;
   string extra_vars = 3;
   repeated MapFieldEntry env_vars = 4;
   string ansible_library = 5;
   bool not_gather_facts = 6;
}

message Task {
   string session_ID = 1;
   string task_str = 2;
   bool is_dict = 3;
}

message WorkerTask {
   string task_str = 1;
   bool is_dict = 2;
}

message TaskResult {
   bool ok = 1;
   string results_dict_str = 2;
   string task_name = 3;
   bool is_changed = 4;
   bool is_failed = 5;
   bool is_skipped = 6;
   bool is_unreachable = 7;
   bool is_ignored_errors = 8;
   bool is_ignored_unreachable = 9;
   string stdout = 10;
   string stderr = 11;
   string msg = 12;
}

message TaskResults {
   bool task_adding_ok = 1;
   string task_adding_error = 2;
   repeated TaskResult task_results = 3;
}

message Status {
   bool ok = 1;
   string error_msg = 2;
}

message WorkerHealthStatus {
   bool ok = 1;
   int32 executions_count = 2;
   int32 executed_tasks_count = 3;
}
~~~