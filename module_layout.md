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
mkdir external
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

And example can be seen in commit `af95b8c8a` (TODO: link)

### Define Keeper