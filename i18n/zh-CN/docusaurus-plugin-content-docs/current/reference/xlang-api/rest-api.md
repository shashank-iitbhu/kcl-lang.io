---
sidebar_position: 2
---

# Rest API

## 1. 启动 REST 服务

通过以下方式可以启动 RestAPI 服务：

```shell
kcl server
```

然后可以通过 POST 协议请求服务：

```shell
curl -X POST http://127.0.0.1:2021/api:protorpc/BuiltinService.Ping --data '{}'
```

期望输出为

```shell
{
	"error": "",
	"result": {}
}
```

其中 POST 请求和返回的 JSON 数据和 Protobuf 定义的结构保持一致。

## 2. `BuiltinService` 服务

其中 `/api:protorpc/BuiltinService.Ping` 路径表示 `BuiltinService` 服务的 `Ping` 方法。

完整的 `BuiltinService` 由 Protobuf 定义：

```protobuf
service BuiltinService {
	rpc Ping(Ping_Args) returns(Ping_Result);
	rpc ListMethod(ListMethod_Args) returns(ListMethod_Result);
}

message Ping_Args {
	string value = 1;
}
message Ping_Result {
	string value = 1;
}

message ListMethod_Args {
	// empty
}
message ListMethod_Result {
	repeated string method_name_list = 1;
}
```

其中 `Ping` 方法可以验证服务是否正常，`ListMethod` 方法可以查询提供的全部服务和函数列表。

## 3. `KclvmService` 服务

`KclvmService` 服务是和 KCL 功能相关的服务。用法和 `BuiltinService` 服务一样。

比如有以下的 `Person` 结构定义：

```python
schema Person:
    key: str

    check:
        "value" in key  # 'key' is required and 'key' must contain "value"
```

然后希望通过 `Person` 来校验以下的 JSON 数据：

```json
{ "key": "value" }
```

可以通过 `KclvmService` 服务的 `ValidateCode` 方法完成。参考 `ValidateCode` 方法的 `ValidateCode_Args` 参数结构：

```protobuf
message ValidateCode_Args {
	string data = 1;
	string code = 2;
	string schema = 3;
	string attribute_name = 4;
	string format = 5;
}
```

根据 `ValidateCode_Args` 参数结构构造 POST 请求需要的 JSON 数据，其中包含 `Person` 定义和要校验的 JSON 数据：

```json
{
  "code": "\nschema Person:\n    key: str\n\n    check:\n        \"value\" in key  # 'key' is required and 'key' must contain \"value\"\n",
  "data": "{\"key\": \"value\"}"
}
```

将该 JSON 数据保存到 `vet-hello.json` 文件，然后通过以下命令进行校验：

```shell
curl -X POST \
    http://127.0.0.1:2021/api:protorpc/KclvmService.ValidateCode \
    -H  "accept: application/json" \
    --data @./vet-hello.json
```

如果看到输出

```json
{
  "error": "",
  "result": {
    "success": true
  }
}
```

说明校验成功。

## 4. 完整的 Protobuf 服务定义

跨语言的 API 通过 Protobuf 定义([https://github.com/kcl-lang/kcl-go/blob/main/pkg/spec/gpyrpc/gpyrpc.proto](https://github.com/kcl-lang/kcl-go/blob/main/pkg/spec/gpyrpc/gpyrpc.proto))：

```protobuf
// Copyright The KCL Authors. All rights reserved.
//
// This file defines the request parameters and return structure of the KCL RPC server.

syntax = "proto3";

package gpyrpc;

// kcl main.k -E pkg_name=pkg_path
message CmdExternalPkgSpec {
	string pkg_name = 1;
	string pkg_path = 2;
}

// kcl main.k -D name=value
message CmdArgSpec {
	string name = 1;
	string value = 2;
}

// kcl main.k -O pkgpath:path.to.field=field_value
message CmdOverrideSpec {
	string pkgpath = 1;
	string field_path = 2;
	string field_value = 3;
	string action = 4;
}

// ----------------------------------------------------------------------------
// Error types
// ----------------------------------------------------------------------------

message Error {
	string level = 1;
	string code = 2;
	repeated Message messages = 3;
}

message Message {
	string msg = 1;
	Position pos = 2;
}

// ----------------------------------------------------------------------------
// service request/response
// ----------------------------------------------------------------------------

// gpyrpc.BuiltinService
service BuiltinService {
	rpc Ping(Ping_Args) returns(Ping_Result);
	rpc ListMethod(ListMethod_Args) returns(ListMethod_Result);
}

// gpyrpc.KclvmService
service KclvmService {
	rpc Ping(Ping_Args) returns(Ping_Result);

	rpc ExecProgram(ExecProgram_Args) returns(ExecProgram_Result);
	rpc BuildProgram(BuildProgram_Args) returns(BuildProgram_Result);
	rpc ExecArtifact(ExecArtifact_Args) returns(ExecProgram_Result);

	rpc ParseFile(ParseFile_Args) returns(ParseFile_Result);
	rpc ParseProgram(ParseProgram_Args) returns(ParseProgram_Result);
	rpc LoadPackage(LoadPackage_Args) returns(LoadPackage_Result);

	rpc FormatCode(FormatCode_Args) returns(FormatCode_Result);
	rpc FormatPath(FormatPath_Args) returns(FormatPath_Result);
	rpc LintPath(LintPath_Args) returns(LintPath_Result);
	rpc OverrideFile(OverrideFile_Args) returns (OverrideFile_Result);

	rpc GetSchemaType(GetSchemaType_Args) returns(GetSchemaType_Result);
	rpc GetFullSchemaType(GetFullSchemaType_Args) returns(GetSchemaType_Result);
	rpc GetSchemaTypeMapping(GetSchemaTypeMapping_Args) returns(GetSchemaTypeMapping_Result);
	rpc ValidateCode(ValidateCode_Args) returns(ValidateCode_Result);

	rpc ListDepFiles(ListDepFiles_Args) returns(ListDepFiles_Result);
	rpc LoadSettingsFiles(LoadSettingsFiles_Args) returns(LoadSettingsFiles_Result);

	rpc Rename(Rename_Args) returns(Rename_Result);
	rpc RenameCode(RenameCode_Args) returns(RenameCode_Result);

	rpc Test(Test_Args) returns (Test_Result);
}

message Ping_Args {
	string value = 1;
}
message Ping_Result {
	string value = 1;
}

message ListMethod_Args {
	// empty
}
message ListMethod_Result {
	repeated string method_name_list = 1;
}

message ParseFile_Args {
	string path = 1;
	string source = 2;
	repeated CmdExternalPkgSpec external_pkgs = 3;  // External packages path
}

message ParseFile_Result {
	string ast_json = 1;         // JSON string value
	repeated string deps = 2;    // file dependency paths
	repeated Error errors = 3;    // Parse errors
}

message ParseProgram_Args {
	repeated string paths = 1;
	repeated string sources = 2;
	repeated CmdExternalPkgSpec external_pkgs = 3;  // External packages path
}

message ParseProgram_Result {
	string ast_json = 1;            // JSON string value
	repeated string paths = 2;      // Returns the files in the order they should be compiled
	repeated Error errors = 3;     // Parse errors
}

message LoadPackage_Args {
	ParseProgram_Args parse_args = 1;
	bool resolve_ast = 2;
	bool load_builtin = 3;
	bool with_ast_index = 4;
}

message LoadPackage_Result {
	string program = 1;          // JSON string value
	repeated string paths = 2;   // Returns the files in the order they should be compiled
	repeated Error parse_errors = 3;     // Parse errors
	repeated Error type_errors = 4;      // Type errors
	map<string, Scope> scopes = 5;    // Map key is the ScopeIndex json string.
	map<string, Symbol> symbols = 6;  // Map key is the SymbolIndex json string.
	map<string, SymbolIndex> node_symbol_map = 7;  // Map key is the AST index UUID string.
	map<string, string> symbol_node_map = 8;       // Map key is the SymbolIndex json string.
	map<string, SymbolIndex> fully_qualified_name_map = 9;  // Map key is the fully_qualified_name e.g. `pkg.Name`
	map<string, ScopeIndex> pkg_scope_map = 10;      // Map key is the package path.
}

message Symbol {
	KclType ty = 1;
	string name = 2;
	SymbolIndex owner = 3;
	SymbolIndex def = 4;
	repeated SymbolIndex attrs = 5;
	bool is_global = 6;
}

message Scope {
	string kind = 1;
	ScopeIndex parent = 2;
	SymbolIndex owner = 3;
	repeated ScopeIndex children = 4;
	repeated SymbolIndex defs = 5;
}

message SymbolIndex {
	uint64 i = 1;
	uint64 g = 2;
	string kind = 3;
}

message ScopeIndex {
	uint64 i = 1;
	uint64 g = 2;
	string kind = 3;
}

message ExecProgram_Args {
	string work_dir = 1;

	repeated string k_filename_list = 2;
	repeated string k_code_list = 3;

	repeated CmdArgSpec args = 4;
	repeated CmdOverrideSpec overrides = 5;

	bool disable_yaml_result = 6;

	bool print_override_ast = 7;

	// -r --strict-range-check
	bool strict_range_check = 8;

	// -n --disable-none
	bool disable_none = 9;
	// -v --verbose
	int32 verbose = 10;

	// -d --debug
	int32 debug = 11;

	// yaml/json: sort keys
	bool sort_keys = 12;

	// -E --external : external packages path
	repeated CmdExternalPkgSpec external_pkgs = 13;

	// Whether including schema type in JSON/YAML result
	bool include_schema_type_path = 14;

	// Whether only compiling the program
	bool compile_only = 15;

	// Show hidden attributes
	bool show_hidden = 16;

	// -S --path_selector
	repeated string path_selector = 17;
}

message ExecProgram_Result {
	string json_result = 1;
	string yaml_result = 2;
	string log_message = 3;
	string err_message = 4;
}

message BuildProgram_Args {
	ExecProgram_Args exec_args = 1;
	string output = 2;
}

message BuildProgram_Result {
	string path = 1;
}

message ExecArtifact_Args {
	string path = 1;
	ExecProgram_Args exec_args = 2;
}

message ResetPlugin_Args {
	string plugin_root = 1;
}
message ResetPlugin_Result {
	// empty
}

message FormatCode_Args {
	string source = 1;
}

message FormatCode_Result {
	bytes formatted = 1;
}

message FormatPath_Args {
	string path = 1;
}

message FormatPath_Result {
	repeated string changed_paths = 1;
}

message LintPath_Args {
	repeated string paths = 1;
}

message LintPath_Result {
	repeated string results = 1;
}

message OverrideFile_Args {
	string file = 1;
	repeated string specs = 2;
	repeated string import_paths = 3;
}

message OverrideFile_Result {
	bool result = 1;
}

message GetFullSchemaType_Args {
	ExecProgram_Args exec_args = 1;
	string schema_name = 2;
}

message GetSchemaType_Args {
	string file = 1;
	string code = 2;
	string schema_name = 3;
}
message GetSchemaType_Result {
	repeated KclType schema_type_list = 1;
}

message GetSchemaTypeMapping_Args {
	string file = 1;
	string code = 2;
	string schema_name = 3;
}
message GetSchemaTypeMapping_Result {
	map<string, KclType> schema_type_mapping = 1;
}

message ValidateCode_Args {
	string data = 1;
	string file = 2;
	string code = 3;
	string schema = 4;
	string attribute_name = 5;
	string format = 6;
}

message ValidateCode_Result {
	bool success = 1;
	string err_message = 2;
}

message Position {
	int64 line = 1;
	int64 column = 2;
	string filename = 3;
}

message ListDepFiles_Args {
	string work_dir = 1;
	bool use_abs_path = 2;
	bool include_all = 3;
	bool use_fast_parser = 4;
}

message ListDepFiles_Result {
	string pkgroot = 1;
	string pkgpath = 2;
	repeated string files = 3;
}

// ---------------------------------------------------------------------------------
// LoadSettingsFiles API
//    Input work dir and setting files and return the merged kcl singleton config.
// ---------------------------------------------------------------------------------

message LoadSettingsFiles_Args {
	string work_dir = 1;
	repeated string files = 2;
}

message LoadSettingsFiles_Result {
	CliConfig kcl_cli_configs = 1;
	repeated KeyValuePair kcl_options = 2;
}

message CliConfig {
	repeated string files = 1;
	string output = 2;
	repeated string overrides = 3;
	repeated string path_selector = 4;
	bool strict_range_check = 5;
	bool disable_none = 6;
	int64 verbose = 7;
	bool debug = 8;
	bool sort_keys = 9;
	bool show_hidden = 10;
	bool include_schema_type_path = 11;
}

message KeyValuePair {
	string key = 1;
	string value = 2;
}

// ---------------------------------------------------------------------------------
// Rename API
//    find all the occurrences of the target symbol and rename them. This API will rewrite files if they contain symbols to be renamed.
// ---------------------------------------------------------------------------------

message Rename_Args {
	string package_root = 1;              // the file path to the package root
	string symbol_path = 2;               // the path to the target symbol to be renamed. The symbol path should conform to format: `<pkgpath>:<field_path>` When the pkgpath is '__main__', `<pkgpath>:` can be omitted.
	repeated string file_paths = 3;       // the paths to the source code files
	string new_name = 4;                  // the new name of the symbol
}

message Rename_Result {
	repeated string changed_files = 1;    // the file paths got changed
}

// ---------------------------------------------------------------------------------
// RenameCode API
//    find all the occurrences of the target symbol and rename them. This API won't rewrite files but return the modified code if any code has been changed.
// ---------------------------------------------------------------------------------

message RenameCode_Args {
	string package_root = 1;              // the file path to the package root
	string symbol_path = 2;               // the path to the target symbol to be renamed. The symbol path should conform to format: `<pkgpath>:<field_path>` When the pkgpath is '__main__', `<pkgpath>:` can be omitted.
	map<string, string> source_codes = 3; // the source code. a <filename>:<code> map
	string new_name = 4;                  // the new name of the symbol
}

message RenameCode_Result {
	map<string, string> changed_codes = 1; // the changed code. a <filename>:<code> map
}

// ---------------------------------------------------------------------------------
// Test API
//    Test KCL packages with test arguments
// ---------------------------------------------------------------------------------

message Test_Args {
	ExecProgram_Args exec_args = 1;      // This field stores the execution program arguments.
	repeated string pkg_list = 2;        // The package path list to be tested e.g., "./...", "/path/to/package/", "/path/to/package/..."
	string run_regexp = 3;               // This field stores a regular expression for filtering tests to run.
	bool fail_fast = 4;                  // This field determines whether the test run should stop on the first failure.
}

message Test_Result {
	repeated TestCaseInfo info = 2;
}

message TestCaseInfo {
	string name = 1;          // Test case name
	string error = 2;
	uint64 duration = 3;         // Number of whole microseconds in the duration.
	string log_message = 4;
}

// ----------------------------------------------------------------------------
// KCL Type Structure
// ----------------------------------------------------------------------------

message KclType {
	string type = 1;                     // schema, dict, list, str, int, float, bool, any, union, number_multiplier
	repeated KclType union_types = 2 ;   // union types
	string default = 3;                  // default value

	string schema_name = 4;              // schema name
	string schema_doc = 5;               // schema doc
	map<string, KclType> properties = 6; // schema properties
	repeated string required = 7;        // required schema properties, [property_name1, property_name2]

	KclType key = 8;                     // dict key type
	KclType item = 9;                    // dict/list item type

	int32 line = 10;

	repeated Decorator decorators = 11;  // schema decorators

	string filename = 12;                // `filename` represents the absolute path of the file name where the attribute is located.
	string pkg_path = 13;                // `pkg_path` represents the path name of the package where the attribute is located.
	string description = 14;             // `description` represents the document of the attribute.
	map<string, Example> examples = 15;  // A map object to hold examples, the key is the example name.
}

message Decorator {
	string name = 1;
	repeated string arguments = 2;
	map<string, string> keywords = 3;
}

message Example {
	string summary = 1;                // Short description for the example.
	string description = 2;            // Long description for the example.
	string value = 3;                  // Embedded literal example.
}

// ----------------------------------------------------------------------------
// END
// ----------------------------------------------------------------------------
```
