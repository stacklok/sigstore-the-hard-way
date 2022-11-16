# Configure Container Registry

To switch things up, we will use Github Container registry (ghcr.io)
to push an image and a signature with cosign. You can however
using an OCI registry, [see here](https://github.com/sigstore/cosign#registry-support)
for a list of those currently supported by cosign.

First, let's create an image. You can use the following `Dockerfile` or any existing image
you already have locally:

```bash
cat > Dockerfile <<EOF
FROM alpine
CMD ["echo", "Hello Sigstore!"]
EOF
```

```bash
docker build -t sigstore-thw:latest .
```

## gchr PAT code

Create a PAT (Personal Access Token) for your account, by following
[the relevant GitHub page](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)

Once you have your PAT code, login to ghcr:

```bash
export CR_PAT="YOUR_TOKEN" ; echo -n "$CR_PAT" | docker login ghcr.io -u <github_user> --password-stdin
```

## Tag and push an image

Now we can tag and push our image:

```bash
docker tag SOURCE_IMAGE_NAME:VERSION ghcr.io/TARGET_OWNER/TARGET_IMAGE_NAME:VERSION
```

Push re-tagged imaged to the container registry:

```bash
docker push ghcr.io/OWNER/IMAGE_NAME:VERSION
```

Example:

```bash
docker tag sigstore-thw:latest ghcr.io/lukehinds/sigstore-thw:latest
docker push ghcr.io/lukehinds/sigstore-thw:latest
The push refers to repository [ghcr.io/lukehinds/sigstore-thw]
cb381a32b229: Pushed
latest: digest: sha256:568999d4aedd444465c442617666359ddcd4dc117b22375983d2576c3847c9ba size: 528
```
