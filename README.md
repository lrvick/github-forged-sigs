# github-forged-sigs

## Threat model

When you press the merge button your threat model assumes:

1. Your browser is not compromised
2. Your browser has no compromised browser plugins
3. There are no exploited flaws on production Github servers or frontends
4. There are no compromised employees at Github with production access
5. There is no MITM on Github.com (not likely but BGP+TLS is *possible*)

If you wish to eliminate these risks from your threat model when reviewing
sensitive code, you could simply check out the code, verify signatures, and
review/merge on your own local system as Torvalds intended. Then you don't
actually need to trust GitHub at all, which is a major security win.

Further say you want to prove that signed commits and merge commits can't be
made via keys stolen by remote actors from a compromised endpoint or similar.

For even better security you can ensure everyone on your team maintains their
signing keys exclusively on hardware security modules like yubikeys, so even
if an endpoint is compromised, the key is never exposed to system memory, thus
preventing a remote actor from being able to obtain it and use it elsewhere.

## Implementation

Github helpfully shows commits you sign and push in the web interface with a
green "verified" tag to signify the commit is signed and verified by GitHub
against the PGP public key associated with that users account, and that it
has a matching email address in one of the UIDs, that also has a matching
verified email in the Github system.

This is a great external sanity check clearly inspired by the green lock
in URL bars for HTTPS.

A common workflow in Github is Pull Requests where peers can review code.

There is a green merge button in pull requests in order to merge code you
happen to review in the web UI.

Code merged via this method similarly gets the green "verified" tag.

## Problem

The big greem merge button in PRs works even when your HSM is not inserted.

If the key only exists in an HSM, then how is it possible for Github to sign
the merge commit?

Well it turns out, they can't. Instead they just forge a commit as you, and
sign with their own key.

Even if your password gets phished and someone gets access to your account, but
they don't have your HSM, they can merge code to master and show the commit
has a valid signature in the GitHub web interface as though you signed a merge
commit locally.

You can't turn this "feature" off.

## Example

My PGP key for this GitHub account is: 8E47A1EC35A1551D

GitHub exposes this publicly and you can use this interface
to verify my commits.

You can for example verify the root commit on this repo:

```
$ https://github.com/lrvick.gpg | gpg --import
$ gpg --list-keys 8E47A1EC35A1551D
$ git clone https://github.com/lrvick/git-forged-sigs
$ git log --show-signature ba89474
commit ba894740f484f0a4ff2832b415a2c6997aeff105
gpg: Signature made Fri 06 Nov 2020 01:35:17 AM PST
gpg:                using RSA key 67553FBDA46BB71ABD2E0B0B8E47A1EC35A1551D
gpg: Good signature from "Lance R. Vick (Personal) <lance@lrvick.net>" [ultimate]
gpg:                 aka "[jpeg image of size 6119]" [ultimate]
gpg:                 aka "Lance R. Vick (Work) <lance@bitgo.com>" [ultimate]
Author: Lance R. Vick <lance@lrvick.net>
Date:   Fri Nov 6 01:35:17 2020 -0800

    first commit
```

The merge commit that added this PR to master, will show as invalid in git
using the above steps, but perfectly valid in the GitHub web interface.

```
commit 7e5c26d2a05d02181b4cbc7957a56a4dffde5c3d (HEAD -> main, origin/main)
gpg: Signature made Fri 06 Nov 2020 02:41:56 AM PST
gpg:                using RSA key 4AEE18F83AFDEB23
gpg: requesting key 4AEE18F83AFDEB23 from hkp server keys.gnupg.net
gpg: key 4AEE18F83AFDEB23: 1 duplicate signature removed
gpg: key 4AEE18F83AFDEB23: public key "GitHub (web-flow commit signing) <noreply@github.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: Good signature from "GitHub (web-flow commit signing) <noreply@github.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 5DE3 E050 9C47 EA3C F04A  42D3 4AEE 18F8 3AFD EB23
Merge: ba89474 ad9647c
Author: Lance R. Vick <lance@lrvick.net>
Date:   Fri Nov 6 02:41:56 2020 -0800

    Merge pull request #1 from lrvick/feature-branch

    add initial draft
```

## Conclusion

Imagine if Google started displaying domains as having valid HTTPS with a green
lock in chrome even if they didn't have HTTPS at all, but just clicked a button
in a Google interface somewhere.

Design flaws like these demonstrates GitHub misunderstands the threat models
and assumptions around commit signing and verification and overloaded the
meaning of the green "verified" in the web interface breaking git and GPG
compatibility in the process.

It also means "only allow signed commits" in settings is not actually enforced.

Consider that GitHub implements no other measures to prevent repo or user
impersonation allowing fake commits like this one:

https://github.com/github/dmca/tree/25cdace60ac64788a4853d33d9fa48c5338f7249

Git commit signing is the only tool we have to avoid issues like this, and it
is broken.

Github however has been aware of these issues for years and still has
demonstrated no willingness to address them.
