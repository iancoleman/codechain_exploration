# Disclaimer

This is an experimental document. It's a recording of my process,
not necessarily statements of facts.

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

One of the signers can create a new release.

The first thing is commit all changes to git as usual.

Then run `codechain publish`

A review of the changes will be shown.

You may need to press 'q' to exit the preview.

A y/n prompt will be shown to confirm the changes.

Then an option to describe this patch (set of changes).

This will create a patch, and a new codechain tree and hashchain.

Commit the new codechain metadata using `git add` and `git commit`

Then push these changes for the second signer to confirm.

Note at this partially signed stage, `codechain status` shows one line in the
unsigned entries, and still says 'tree is dirty'

## Adding a second signature

The second signer can now `git pull` the changes to codechain from the first
signer.

Then the second signer calls `codechain review` which will prompt them to
'review patch (no aborts)? y/n'

Selecting y will show the patch diff.

If it's a big diff the reviewer will need to press 'q' to exit the review
and choose to sign or not sign the patch.

Once reviewed the prompt is shown 'sign patch? y/n'

When the patch is signed the signer can call `codechain status`

But for me it isn't showing a signed release.

Maybe because there was a git commit *after* the codechain signature? And now
when I call `codechain review` it's applying the review to a later git commit?
Are codechain reviews like full-stops, requiring all signatures before further
code is added?

I'm a bit confused here.

I'll `git commit` and `git push` the second signature.

## Finalizing the release

When I did `git pull` of the second signature back onto the laptop (where the
first signature was generated) I followed up with `codechain review`. This
combo of `git pull && codechain review` seems very common.

After the review was made and signed `codechain status` showed a signed
release. It still aslo says 'tree is dirty' so there's something in that
beyond just release status I suppose.

So it looks like I've managed to sign a release. A couple of surprising
observations:

* I ended up doing three signatures to make a release (laptop, then desktop,
then laptop again). I wonder if that could have been less or if I messed it up
somehow? I would have thought just laptop then desktop would be enough.

* git commits and codechain patches are not tied to each other. I'm still
conceptualizing the most appropriate way for these to work together. It's
probably going to be a git feature-branch type of workflow that suits this
best.

Next step is to try to avoid having N git commits for N codechain signatures.

This should be achieved by using detached signatures and non-git communication
of those patches to a 'leader' of the patch, who can commit all the signatures
in one single git commit.

## Open questions

Is there a clear and simple correlation between codechain patches and git
commits? I think not, codechain patches are *completely* independent of the
git history, and it's a mistake to believe that a git commit containing a
codechain patch must include all prior commits.

## TODO

* [ ] use git to make code changes, and codechain to sign them.
* [ ] single commit release using detached signatures and out-of-band comms
* [ ] consider if signing can be automated or aliased somehow
* [ ] consider possible git pull combined with codechain review
* [ ] get a third key from someone else and see what happens
