# Onboarding Developers

This is intended as a quick tutorial and reference guide for new developers
who are solid with go, but want to get up to speed quickluy with SDK programming.
We are focusing here mainly on how to build modules, as that is the majority of
the blockchain work at Regen.

## Public References

In order to get a high-level overview of building cosmos-sdk apps, I can recommend
you [watch this video](https://www.youtube.com/watch?v=pyAmxlzVdqM) if you like that.
Then [go through this tutorial](https://cosmos.network/docs/tutorial/) to get a general
feel of how to tie together all the pieces, and what these words like "Module",
"Handler", "Keeper", "Codec" means in the cosmos-sdk.

To get some better context of how this sdk modules fit into a blockchain, you should
read a bit of [the sdk architecture overview](https://cosmos.network/docs/intro/) and
also an [introduction to tendermint](https://tendermint.com/docs/introduction/what-is-tendermint.html#tendermint-vs-x).

## Module strucutre

As of cosmos-sdk 0.36, all new modules are supposed to follow the
[following structure](https://github.com/cosmos/cosmos-sdk/blob/master/docs/building-modules/structure.md).
I will go through step-by-step as I build out a new module based on existing code, to demonstrate how to
create this.

Check out the [step by step guide](./module_layout.md) if you are interested in the details.