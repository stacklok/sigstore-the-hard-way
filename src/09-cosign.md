# Cosign

We will now install cosign. It is assumed from now, that cosign will
be run on a machine local to you (such as your laptop or PC), and outside of the sigstore infrastructure.

## Install cosign

Head the [releases page for cosign v1.0](https://github.com/sigstore/cosign/releases/tag/v1.2.1)
and download a release specific to your hardware (MacOS, Linux, Windows)

Also download the cosign public key, signature for your architecture.

* `release-cosign.pub`
* `cosign-$OS-$ARCH.sig`

Verify the signing.

### Linux binary

Download required files

```bash
curl -fsSL --remote-name-all https://github.com/sigstore/cosign/releases/download/v1.2.1/{cosign-linux-amd64,release-cosign.pub,cosign-linux-amd64.sig}
```

Verify signature

```bash
openssl dgst -sha256 -verify release-cosign.pub -signature <(cat cosign-linux-amd64.sig | base64 -d) cosign-linux-amd64
Verified OK
```

Remove signature files

```bash
rm cosign-linux-amd64.sig release-cosign.pub
```

Install

```bash
chmod +x cosign-linux-amd64
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
```

### MacOS binary

Download required files

```bash
curl -fsSL --remote-name-all https://github.com/sigstore/cosign/releases/download/v1.2.1/{cosign-darwin-amd64,release-cosign.pub,cosign-darwin-amd64.sig}
```

Verify signature

```bash
openssl dgst -sha256 -verify release-cosign.pub -signature <(cat cosign-darwin-amd64.sig | base64 -D) cosign-darwin-amd64
Verified OK
```

Remove signature files

```bash
rm cosign-darwin-amd64.sig release-cosign.pub
```

Install

```bash
chmod +x cosign-darwin-amd64
sudo mv cosign-darwin-amd64 /usr/local/bin/cosign
```
