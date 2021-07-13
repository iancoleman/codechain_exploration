# Test repo

This repo is to test [codechain](https://github.com/frankbraun/codechain)
and how it works alongside git.

The [codechain walkthrough](https://github.com/frankbraun/codechain/blob/master/doc/walkthrough.md)
is a useful first-stop for getting started, and is what was used to generate
this document you're reading now.

Initially in the repo there's no codechain release and no codechain signers.

Then one codechain signer is added:

`codechain start -s ~/.config/codechain/secrets/<myidentity>`

The existing code and codechain directory with one signer is commited via git.

We can see where git is at using `git log` and we can see where codechain is at
using `codechain status`
