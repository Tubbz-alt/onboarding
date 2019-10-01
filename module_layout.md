# Module Layout

Before we start, please open the 
[official module guide](https://github.com/cosmos/cosmos-sdk/blob/master/docs/building-modules/structure.md)
in your browser and follow along.

In this example, I am have an existing upgrade module that is laid out according to the old spec.
We don't need to look at the code samples, but rather follow how I place the existing code according
to this layout.

Before I started, I renamed `x/upgrade` to `x/_upgrade`, but if this is a new module, there is no
need to do that.

## Basic setup

### Create layout

```
mkdir -p x/upgrade
cd x/upgrade
mkdir exported
mkdir -p internal/types
mkdir -p internal/keeper
mkdir -p client/cli
mkdir -p client/rest
```

### Define Internal Types

These types have no dependencies on anything in this module, and are a good place to start

* `codec.go` is the same as previous top-level `codec.go`. You only need to define `RegisterCodec` there.
* `msgs.go` if you have any exported messages for which you want to register a handler
* `keys.go` this should `ModuleName`, `StoreKey`, `RouterKey`, `QuerierRoute` (all generally the same), and any custom keys/prefixes you need for kvstore access
* and one file per internal data structure, eg. in this case:
    * `plan.go` for the Plan that we serialize
    * `proposal.go` for the software upgrade proposals

An example can be seen in commit `af95b8c8a` (TODO: link)

### Define Keeper

A "Keeper" is more or less what is called a "Controller" in typical web app terminology.
It exposes all methods and business logic to query and modify state. However, it is
not connected to the api nor does it provide authorization checks (that is left
for a higher-level wrapper). We generally write tests on the Keeper to ensure everything
is working as intended, and then can just test the higher-level Handler to make sure
the authorization and wiring are done properly.

This is usually just placing the `keeper.go` and `keeper_test.go` file inside `internal/keeper`.
Note that we now need to update most imports to import from `internal/types` as the types are
no longer in the same directory.

**TODO** explain more about keepers

#### Exported

Many previous implementations defined `interface Keeper` and `struct keeper` which implements it.
The new format is to move `interface Keeper` into `exported` and make `struct keeper` -> `struct Keeper`
inside the `internal/keeper` package.

We also want to ensure that the structs defined in `internal/types` always do fulfill the 
interfaces in `exported`, even after a refactor. The typical way is to provide a type
assertion. (Note that exported should not pull in other modules to avoid circular imports).
`var _ exported.Keeper = (*Keeper)(nil)`

An example can be seen in commit `e06e69ee3` (TODO: link)

## Client

There are two types of clients we provide: cli and rest. Cli registers subcommands for creating transactions
and querying state, which are added to the standard cli tool when the module is enabled (`xrncli`, `gaiacli`, etc).
The rest package register rest endpoints that are exposed by the LCD (light client daemon). This is a bit
deep to go into, but basically, as historically there are no good js (or non-go) client libraries to parse go-amino
objects, the LCD provides a nice JSON interface to query the blockchain and handles the marshalling and unmarshalling
of all the custom binary encodings exposed in tendermint's `abci_query`. 

If they ever remove go-amino and use some standard encoding, client libraries will be able to do the binary encoding
directly and this whole LCD / rest server infrastructure will no longer be needed.

**TODO** link to docs

### Cli

For `query.go`, add commands like `GetPlanCmd` that register a name and then construct the proper key to
dispatch to `cliCtx.QueryStore()`. We then parse the binary response into a struct and send it
to `cliCtx.PrintOutput()` to display as either json or yaml. You can look at an 
[example command](https://github.com/cosmos/cosmos-sdk/blob/133739c1260c2be89b51efd134be293d1cd70327/x/upgrade/client/cli/query.go#L13-L42)

For `tx.go`, the point is to parse various command-line arguments to construct a valid `sdk.Msg`. This
message is then send to `utils.GenerateOrBroadcastMsgs(cliCtx, txBldr, []sdk.Msg{msg})` in order to
construct the final transaction, sign it, and possibly broadcast it (depending on other cli flags set).
This general flow can be seen in the [relatively simple cancel proposal tx](https://github.com/cosmos/cosmos-sdk/blob/133739c1260c2be89b51efd134be293d1cd70327/x/upgrade/client/cli/tx.go#L143-L178)

### Rest

**TODO** I'm still not sure best practices here

## Top Level Module

The main purpose of the top-level code is to import both the internal types and the client types (rest and cli)
and pull them into one Module than can be exported. Packages looking to use the functionality of your module
will import this top level. Those who want to require some interface (dependency injection), will just use
`exported`.

The reason for such a structure seems to be imports. `client/*` naturally uses code from `internal/*`,
which used to be top-level. However, they added an `AppModule` abstraction that exposes both internal
elements and client elements. If this module is in the top-level, it imports `client/*`, which in
turn imports the top-level module. Thus, this top-level package should avoid most business logic and just focus
on providing the "glue pieces" to create the external interfaces you need to construct `app.go`.

### Aliases

The top-level code will typically want to include much of the code from `internal/{types,keeper}` and to ease
imports, it has become a standard to create an `alias.go` file that pulls those types into the top-level package.
You can see [many](https://github.com/cosmos/cosmos-sdk/blob/master/x/staking/alias.go)
[examples](https://github.com/cosmos/cosmos-sdk/blob/master/x/supply/alias.go) that start with
`// autogenerated code using github.com/rigelrozanski/multitool`. So, let's do the same...

This is unfortunately used everywhere throughout the cosmos-sdk, but not even mentioned
in the [repo's README file](https://github.com/rigelrozanski/multitool/blob/master/README.md). Here is how.

```console
go get github.com/rigelrozanski/multitool/cmd/mt
mt --help
```

Unfortunately, `/bin/mt` is installed in most unix systems, so we get that, not the desired binary.
Let's make an alias. Assuming you have not set a custom GOPATH, compiled binaries will go to `$HOME/go/bin`.
So try:

```console
alias multitool="$HOME/go/bin/mt"
multitool create-alias --help
```

Okay, now we got this setup, let's make the alias file. Assuming we are in `x/upgrade`:

```console
multitool create-alias ./internal/types ./internal/keeper
less alias.go
go build .
```

(This fails due to a bug in mt. You need to edit the file and remove the trailing slash on the imports path.
eg. `.../internal/types/` -> `.../internal/types`. Also, this will fail if the name of the package is not
the same as the name of the directory. This should usually be the case, just a warning).

Hopefully now you got the `alias.go` file set up. This is a lot of time now, but hopefully will be a time savings
in the future, when you create more modules.
