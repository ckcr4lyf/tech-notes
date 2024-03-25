# PGP

PGP is great for signing and encryption. Signing is an especially valuable concept, since it has stronger guarantees on the author as opposed to any kind of e.g. social media profile, which by nature have backdoors accessible by staff.

## Keygen

We will use keys based on [Curve25519](https://en.wikipedia.org/wiki/Curve25519). For signing, this means [Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519), and for encryption [X25519](https://x25519.xargs.org/).


We will first generate a master ceritification key:

```
gpg --quick-generate-key 'Your Name <email@domain.com>' ed25519 cert never
```

You should get something like:

```
pub   ed25519 2024-03-25 [C]
      224245002A2A0003CD7CF693FFC081E4E8D5FA41
uid                      Your Name <email@domain.com>
```

The long hex string is the key fingerprint. We will save it in a temporary variable for generating the subkeys:

```
export KEYFP=224245002A2A0003CD7CF693FFC081E4E8D5FA41
```

Now we will generate the subkeys for signing and encryption. Here we will use a 3 year expiration, you can change it to something else if you'd like:

```
gpg --quick-add-key $KEYFP ed25519 sign 3y
gpg --quick-add-key $KEYFP cv25519 encr 3y
```

Our keyring should now look like:

```
$ gpg -K --with-keygrip

sec   ed25519 2024-03-25 [C]
      224245002A2A0003CD7CF693FFC081E4E8D5FA41
      Keygrip = FCC73C8DA31BEB61ECF3B2316131CBF9AFE02675
uid           [ultimate] Your Name <email@domain.com>
ssb   ed25519 2024-03-25 [S] [expires: 2027-03-25]
      Keygrip = 5C407E3653C2BC7C1E4F06BB1179A9A317A88F52
ssb   cv25519 2024-03-25 [E] [expires: 2027-03-25]
      Keygrip = EBA000BD362AE3CFC512730544B133E7BBA7E217
```

Now that we generated the subkeys for signing and encryption, we can delete the master key. But we need to first save it, for when we need to generate new keys (e.g. after the current ones expire).

Back up the master key via

```
$ gpg --armor --export-secret-keys $KEYFP > gpg_master_key.asc
```

Now, we will delete the master key from our keyring. Go to your private key directory (usually `~/.gnupg`), and delete the file matching the keygrip of the `sec` key. In our case it is `FCC73C8DA31BEB61ECF3B2316131CBF9AFE02675.key`.

Now when you view your keyring, GPG will print `sec#` indicating the secret key is not available:

```
sec#  ed25519 2024-03-25 [C]
      224245002A2A0003CD7CF693FFC081E4E8D5FA41
      Keygrip = FCC73C8DA31BEB61ECF3B2316131CBF9AFE02675
uid           [ultimate] Your Name <email@domain.com>
ssb   ed25519 2024-03-25 [S] [expires: 2027-03-25]
      Keygrip = 5C407E3653C2BC7C1E4F06BB1179A9A317A88F52
ssb   cv25519 2024-03-25 [E] [expires: 2027-03-25]
      Keygrip = EBA000BD362AE3CFC512730544B133E7BBA7E217
```

Make sure to save `gpg_master_key.asc` somewhere encrypted for when you need it!

## References

* https://github.com/drduh/YubiKey-Guide
* https://musigma.blog/2021/05/09/gpg-ssh-ed25519.html
