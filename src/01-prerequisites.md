# Prerequisites

## Google Cloud Platform

### Install the Google Cloud SDK

Follow the Google Cloud SDK [documentation](https://cloud.google.com/sdk/) to install and configure the `gcloud` command line utility.

Verify the Google Cloud SDK version is 338.0.0 or higher:

```bash
gcloud version
```

### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

If you are using the `gcloud` command-line tool for the first time `init` is the easiest way to do this:

```bash
gcloud init
```

`gcloud init` will give you an opportunity to create a new project and set a zone. If you're going to copy and paste CLI commands then for ease of use name it `sigstore-the-hard-way-proj`

Be sure to authorize gcloud to access the Cloud Platform with your Google user credentials:

```bash
gcloud auth login
```

Next set a default compute region and compute zone:

```bash
gcloud config set compute/region europe-west1
```

Set a default compute zone:

```bash
gcloud config set compute/zone europe-west1-b
```

> 📝 Use the `gcloud compute zones list` command to view additional regions and zones.
