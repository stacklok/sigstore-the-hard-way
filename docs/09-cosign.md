# Cosign

We will now install cosign. It is assumed from now, that cosign will
be run on a machine local to you (such as your laptop or PC), and outside of the sigstore infrastructure.

## Install cosign

Head the [releases page for cosign v1.0](https://github.com/sigstore/cosign/releases/tag/v1.0.0) 
and download a release specific to your hardware (MacOS, Linux, Windows)

Also download the cosign public key, signature for your architecture.

* `release-cosign.pub`
* `cosign-$OS-$ARCH.sig`

Verify the signing.signing

Here is an example for a linux binary

```
openssl dgst -sha256 -verify release-cosign.pub -signature <(cat cosign-linux-amd64.sig | base64 -d) cosign-linux-amd64
Verified OK
```

For a MacOS binary

```
openssl dgst -sha256 -verify release-cosign.pub -signature <(cat cosign-darwin-amd64.sig | base64 -D) cosign-darwin-amd64
Verified OK
```

Next: [Sign a container](10-sign-container.md)