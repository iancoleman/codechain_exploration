# Test repo

This repo is to test [codechain](https://github.com/frankbraun/codechain)
and how it works alongside git.

The [codechain walkthrough](https://github.com/frankbraun/codechain/blob/master/doc/walkthrough.md)
is a useful first-stop for getting started, and is what was used to generate
this document you're reading now.

Initially in the repo there's no codechain release and no codechain signers.

Git is initialized:

`git init`

This creates a `.git` subdirectory containing all the git-related metadata.

Then codechain is initialized with one signer:

`codechain start -s ~/.config/codechain/secrets/<myidentity>`

This creates a `.codechain` subdirectory containing all the codechain metadata.

The existing code and codechain directory with one signer is commited via git.

We can see where git is at using `git log` and we can see where codechain is at
using `codechain status`

It's also easy to start codechain again from scratch:

`rm -r .codechain`

`codechain start -s ~/.config/codechain/secrets/<myidentity>`

At this stage there's no point making a release with only 1 signer
(confirm this is true?)
so we can push the git to a repo and start to get others involved in the git
commits and the codechain.

## Adding another signer

A second person can clone the repository

`git clone <the repository>`

In this case the second person is my desktop (first person is my laptop).

The desktop doesn't have codechain installed so I'll get the binary that was
compiled on my laptop.

[mega.nz#codechain](https://mega.nz/file/Zdc0nTQT#L454w2wxGn7QYGpST2Lh68dO1QmT7sm6MVoemn7Al1g)
`linux_x86_64` at commit
[b63f139](https://github.com/frankbraun/codechain/tree/b63f13940558f136cf778965b3b7a19e9a7d42a0)

This second user creates a codechain key

`codechain keygen`

Then adds it to the codechain signing keys for this repo

`codechain addkey <pubkey> <signature> <comment>`

The values for pubkey signature and comment can be found easily by

`codechain keyfile -s </path/to/my/.config/codechain/secrets/keyname>`

This operation can be seen within the `.codechain/hashchain` file:

`cat .codechain/hashchain`

which should now show two lines (one for each key).

The first line operation is `cstart` for starting codechain.

The second line operation is `addkey` for adding the second key.

But when we run `codechain status` our second key is not part of the chain yet.

I tried to add the second key myself with `codechain sigctl -m 2` but the
second key can only 'stage' this change, not approve it. Approval requires
the first key.

Running `codechain status` shows these unsigned actions:

```
unsigned entries:
0 addkey 1 <second key details>
0 sigctl 2
```

## Approving the second signer

Presumably the first signer (who has all the rights to the repo) must approve
the second signer.

Now operating as the first key, I have pulled the second key changes using
`git pull`

It's worth checking whether any codechain stuff is pending before
getting into any git/code changes.

`codechain status`

Shows there are changes waiting.

The changes can be reviewed and commited using

`codechain review`

The changes are shown and can be approved with a y/n prompt.

It's wise to double check the key being added really does belong to the second
person, not exactly sure best practice here but seems like one of those
'very important things' gpg is always hammering on about (keyparties etc).

After approving these changes the status now shows 2-of-2 signatures required.

This is interesting because the second key proposed the change, even without
any authority to sign the change. Clear separation of proposal and signing.

Now we have two signers in place, and both are required before a valid release
can be created.

Note that `codechain status` still shows 'tree is dirty' (because `head` - of
codechain, not git - is not signed?)

## Signing code changes

TODO

* [ ] use git to make code changes, and codechain to sign them.
* [ ] single commit release using detached signatures and out-of-band comms
* [ ] consider if signing can be automated or aliased somehow
* [ ] consider possible git pull combined with codechain review
* [ ] get a third key from someone else and see what happens
