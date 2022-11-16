# Sign Container

We are now ready to sign our container using our own sigstore infrastructure

But before we do that, we need to use our own TUF public key file, you might remember created this when deploying the certificate transparency server.

Have this file locally and set it as an environment variable:

```bash
export SIGSTORE_CT_LOG_PUBLIC_KEY_FILE="/path/to/ctfe_public.pem"
```

```bash
COSIGN_EXPERIMENTAL=1 cosign sign --oidc-issuer "https://oauth2.example.com/auth" --fulcio-url "https://fulcio.example.com" --rekor-url "https://rekor.example.com" ghcr.io/<github_user>/sigstore-thw:latest
```

> :notebook: `COSIGN_EXPERIMENTAL` does as it says, you're trying out an experimental feature here.

> 📝 If you receive an `UNAUTHORIZED: authentication required` error. You need
  to reauthenticate with your PAT in GitHub Container Registry again, refer to [Configure registry](08-configure-registry.md)

An example run:

```bash
COSIGN_EXPERIMENTAL=1 cosign sign -oidc-issuer https://oauth2.decodebytes.sh/auth -fulcio-url https://fulcio.decodebytes.sh --rekor-url https://rekor.decodebytes.sh ghcr.io/lukehinds/sigstore-thw:latest
Generating ephemeral keys...
Retrieving signed certificate...
Your browser will now be opened to:
https://oauth2.decodebytes.sh/auth/auth?access_type=online&client_id=sigstore&code_challenge=ZP91ElDffEaUAJxCTYpr_RfpvLHTx8a9WEuiDJiMQT0&code_challenge_method=S256&nonce=1vzuVUvfZ4caqLwqJlUsm0lJglb&redirect_uri=http%3A%2F%2Flocalhost%3A5556%2Fauth%2Fcallback&response_type=code&scope=openid+email&state=1vzuVUvXnKzS2hJnLzxkiDt0qOw
warning: uploading to the transparency log at https://rekor.decodebytes.sh for a private image, please confirm [Y/N]: Y
tlog entry created with index:  11
Pushing signature to: ghcr.io/lukehinds/sigstore-thw:latest:sha256-568999d4aedd444465c442617666359ddcd4dc117b22375983d2576c3847c9ba.sig
```

## Verifying the signing

We will now verify the signing, but before we do we need to tell cosign about our fulcio root.

Grab your `fulcio-root.pem` cerficate you generated on the fulcio server (and also copied to the certificate transparency server)

Set the following environment variable:

```bash
export SIGSTORE_ROOT_FILE="$HOME/fulcio-root.pem"
```

Download the Rekor public key:

```bash
wget -O publicKey.pem https://rekor.example.com/api/v1/log/publicKey
```

Set it in the appropriate environment variable:

```bash
export SIGSTORE_REKOR_PUBLIC_KEY="$PWD/publicKey.pem"
```

We can now verify:

```bash
COSIGN_EXPERIMENTAL=1 cosign verify --rekor-url https://rekor.example.com ghcr.io/<github_user>/sigstore-thw:latest
```

An example:

```bash
COSIGN_EXPERIMENTAL=1 cosign verify --rekor-url https://rekor.decodebytes.sh ghcr.io/lukehinds/sigstore-thw:latest

Verification for ghcr.io/lukehinds/sigstore-thw:latest --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The claims were present in the transparency log
  - The signatures were integrated into the transparency log when the certificate was valid
  - Any certificates were verified against the Fulcio roots.
Certificate subject:  [lhinds@redhat.com]
{"critical":{"identity":{"docker-reference":"ghcr.io/lukehinds/sigstore-thw"},"image":{"docker-manifest-digest":"sha256:568999d4aedd444465c442617666359ddcd4dc117b22375983d2576c3847c9ba"},"type":"cosign container image signature"},"optional":null}
```

## Congrats

If you got this far well done for completing the tutorial!

![glass](images/glass.gif)

## What Next

If you're not already part of the sigstore community, come and join us on our slack channel; [Invite link](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ)
and tell us abour your ideas!

If you want to improve this guide, please make an issue or better still a pull request!

Don't forget to delete your instances and not take on unwanted costs!

## Having issues, not working?

Raise an issue (best option, as others can learn) or message me on the [sigstore slack](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ), I'm always happy to help.
