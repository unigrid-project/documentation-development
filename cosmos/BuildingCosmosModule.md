# How to build a module in Cosmos SDK without using Ignite CLI
At the time of this writing, Ignite CLI supported only up to version v0.46.7 of Cosmos SDK and we had to write a module for version 0.47.x of Cosmos SDK.  This describes how to do it without Ignite CLI scaffolding support, and what to look out for in the process.

## Process

* [Create module folder structure](#folder-structure)
* [Create protobuf files](#protobuf-files)
* [Generate Go code from protobuf files](#generate-go-code)
* [Post API files to your repo](#api-files)
* [Add module to simapp](#add-module)
* [Continue to develop module](#develop)
* [Move module to your repo](#export-module)

### Folder structure
I'm not sure if it creates the folders in proto gen or if they need to be there already, but it's probably best to create these folders first.  Particularly important for v1 to have the proto folders as shown here.

- api
	- cosmos
		- `<module-name>`
- proto
	- cosmos
		- `<module-name>`
			- module
				- v1
					- `<module-name>.proto`
			- v1beta1
- x
	- `<module-name>`
		- client
			- cli
		- keeper
		- testutil
		- types
		- module.go
		- README.md

### Protobuf files
Protobuf files define three things basically.

* [The state data structure of your module](#protobuf-state).
* [The query api of your module for gRPC, Rest, and CLI](#protobuf-query)
* [The message api which can set parameters](#protobuf-messages)

I am no protobuf expert so I invite you to visit some of these links:

* Genesis proto in Cosmos SDK docs [Type Definition](https://docs.cosmos.network/v0.47/building-modules/genesis#type-definition)
* Query proto in Cosmos SDK docs [gRPC Queries](https://docs.cosmos.network/v0.47/building-modules/messages-and-queries#grpc-queries)
* Msg proto in Cosmos SDK docs [Msg Services](https://docs.cosmos.network/v0.47/building-modules/messages-and-queries#msg-services)
* [Messages and Queries](https://docs.cosmos.network/v0.47/building-modules/messages-and-queries)
* [Guidelines for protobuf message definitions](https://docs.cosmos.network/v0.47/core/encoding#guidelines-for-protobuf-message-definitions)

#### Protobuf State
There are generally one or two files for defining the module state:

- proto
	- cosmos
		- `<module-name>`
			- v1beta1
				- genesis.proto
				- `<module-name>.proto`

Genesis protobuf file defines the overall state and usually supporting data structures are defined in your `<module-name>.proto` file.

Example Genesis.proto:

``` protobuf
syntax = "proto3";
package cosmos.ugdmint.v1beta1;

import "gogoproto/gogo.proto";
import "cosmos/ugdmint/v1beta1/ugdmint.proto";
import "amino/amino.proto";

option go_package = "github.com/cosmos/cosmos-sdk/x/ugdmint/types";

// GenesisState defines the ugdmint module's genesis state.
message GenesisState {
  // minter is a space for holding current subsidy information.
  Minter minter = 1 [(gogoproto.nullable) = false, (amino.dont_omitempty) = true];

  // params defines all the parameters of the module.
  Params params = 2 [(gogoproto.nullable) = false, (amino.dont_omitempty) = true];
}
```

The option go_package refers to your modules types folder in x.  This is where proto gen will create your Go files,  It creates them locally and the module does not have to exist on the online repo.

Example `<module-name>.proto`:

``` protobuf
syntax = "proto3";
package cosmos.ugdmint.v1beta1;

import "gogoproto/gogo.proto";
import "cosmos_proto/cosmos.proto";
import "amino/amino.proto";

option go_package = "github.com/cosmos/cosmos-sdk/x/ugdmint/types";

// Minter represents the minting state.
message Minter {
  // current subsidy halving interval
  string subsidy_halving_interval = 1 [
    (cosmos_proto.scalar)  = "cosmos.Dec",
    (gogoproto.customtype) = "github.com/cosmos/cosmos-sdk/types.Dec",
    (gogoproto.nullable)   = false
  ];
}

// Params defines the parameters for the module.
message Params {
  option (gogoproto.goproto_stringer) = false;
  option (amino.name)                 = "cosmos-sdk/x/ugdmint/Params";

  // type of coin to mint
  string mint_denom = 1;
  // subsidy halving interval
  string subsidy_halving_interval = 2 [
    (cosmos_proto.scalar)  = "cosmos.Dec",
    (gogoproto.customtype) = "github.com/cosmos/cosmos-sdk/types.Dec",
    (gogoproto.nullable)   = false
  ];
  // goal of percent bonded atoms
  string goal_bonded = 3 [
    (cosmos_proto.scalar)  = "cosmos.Dec",
    (gogoproto.customtype) = "github.com/cosmos/cosmos-sdk/types.Dec",
    (gogoproto.nullable)   = false
  ];
  // expected blocks per year
  uint64 blocks_per_year = 4;
}]
```

I think this file is optional but recommended.  Additional data structures to support your state and query and messages would be defined here.

#### Protobuf Query
The query protobuf file defines the gRPC service and request and response data structures.  I believe this generates the REST and CLI as well.

Example query.proto

proto/cosmos/`<module-name`/v1beta1/query.proto
``` protobuf
syntax = "proto3";
package cosmos.ugdmint.v1beta1;

import "gogoproto/gogo.proto";
import "google/api/annotations.proto";
import "cosmos/ugdmint/v1beta1/ugdmint.proto";
import "amino/amino.proto";

option go_package = "github.com/cosmos/cosmos-sdk/x/ugdmint/types";

// Query defines the gRPC querier service.
service Query {
  // Parameters queries the parameters of the module.
  rpc Params(QueryParamsRequest) returns (QueryParamsResponse) {
    option (google.api.http).get = "/cosmos/ugdmint/v1beta1/params";
  }

  // Subsidy halving interval
  rpc SubsidyHalvingInterval(QuerySubsidyHalvingIntervalRequest) returns (QuerySubsidyHalvingIntervalResponse) {
    option (google.api.http).get = "/cosmos/ugdmint/v1beta1/subsidy_halving_interval";
  }
}

// QueryParamsRequest is request type for the Query/Params RPC method.
message QueryParamsRequest {}

// QueryParamsResponse is response type for the Query/Params RPC method.
message QueryParamsResponse {
  // params holds all the parameters of this module.
  Params params = 1 [(gogoproto.nullable) = false, (amino.dont_omitempty) = true];
}

// QuerySubsidyHalvingIntervalRequest is the request type for the Query/SubsidyHalvingInterval RPC method.
message QuerySubsidyHalvingIntervalRequest {}

// QuerySubsidyHalvingIntervalResponse is the response type for the Query/SubsidyHalvingInterval RPC method.
message QuerySubsidyHalvingIntervalResponse {
  // subsidy halving interval value.
  bytes subsidy_halving_interval = 1 [
    (gogoproto.customtype) = "github.com/cosmos/cosmos-sdk/types.Dec",
    (gogoproto.nullable)   = false,
    (amino.dont_omitempty) = true
  ];
}
```

#### Protobuf Messages
This defines the Msg service, and adds an rpc call.

Example tx.proto:

proto/cosmos/`<module-name>`/v1beta1/tx.proto
``` protobuf
syntax = "proto3";
package cosmos.ugdmint.v1beta1;

option go_package = "github.com/cosmos/cosmos-sdk/x/ugdmint/types";

import "cosmos/msg/v1/msg.proto";
import "amino/amino.proto";
import "cosmos/ugdmint/v1beta1/ugdmint.proto";
import "gogoproto/gogo.proto";
import "cosmos_proto/cosmos.proto";

// Msg defines the x/ugdmint Msg service.
service Msg {
  option (cosmos.msg.v1.service) = true;

  // UpdateParams defines a governance operation for updating the x/ugdmint module
  // parameters. The authority is defaults to the x/gov module account.
  //
  // Since: cosmos-sdk 0.47
  rpc UpdateParams(MsgUpdateParams) returns (MsgUpdateParamsResponse);
}

// MsgUpdateParams is the Msg/UpdateParams request type.
//
// Since: cosmos-sdk 0.47
message MsgUpdateParams {
  option (cosmos.msg.v1.signer) = "authority";
  option (amino.name)           = "cosmos-sdk/x/ugdmint/MsgUpdateParams";

  // authority is the address that controls the module (defaults to x/gov unless overwritten).
  string authority = 1 [(cosmos_proto.scalar) = "cosmos.AddressString"];

  // params defines the x/ugdmint parameters to update.
  //
  // NOTE: All parameters must be supplied.
  Params params = 2 [(gogoproto.nullable) = false, (amino.dont_omitempty) = true];
}

// MsgUpdateParamsResponse defines the response structure for executing a
// MsgUpdateParams message.
//
// Since: cosmos-sdk 0.47
message MsgUpdateParamsResponse {}
```

### Generate Go Code



### API files
I'm not sure entirely all of what the API generated files are for.  Might be a good research project.  But at least the `api/cosmos/<module-name>/module/v1/module.pulsar.go` file needs to be referenced for the whole thing to work.  I tied referencing it by local relative path but did not have success, so I pushed the api folder up to my repo to reference it there.

### Add module
Ignite scaffolding did a lot of this for us but we'll have to do it by hand. Our new module needs to be added to some if not all of the following files in `/simapp` folder:

### Develop
There is still much to do to develop the module out.  Go through the tutorials and documentation in [Cosmos SDK](docs.cosmos.network/v0.47) to learn more.  Definitely look at the source code of the other Cosmos modules and even copy code to help flesh out your module.  When the module code is ready to run, follow these general steps:

### Export Module
Once the module is developed and ready to publish, or at least a functional first draft, you might want to move the module into your repo if you don't want it committed directly into a cosmos-sdk fork.

