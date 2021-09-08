# Domain configuration

Now that are instances are running, lets grab the external IP's and set up domains.

> 📝 A cheap temp domain can be grabbed from [Google Cloud Domains](https://console.cloud.google.com/net-services/domains/). Just type in random, nonsensical string
  and you should easily be able to get a domain
  for $1. There are also lots of other providers. Use whatever works for you.

## Configuration

### rekor.example.com

Grab your external / public IP

```
gcloud compute instances describe sigstore-rekor \
--format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

You now want to make an "A Record" to a subdomain or "rekor" and to your external IP from the above command

To create resource records on Google, 
1. Go to [Google Domains](https://domains.google.com/)
2. Click on your domain from the homepage
3. DNS > Manage Custom Records

|Type|Host| Value|
|---|---|---|
| A Record|rekor|x.x.x.x|

### fulcio.example.com

Now repeat the same for fulcio, and dex

```
gcloud compute instances describe sigstore-fulcio \
--format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

|Type|Host| Value|
|---|---|---|
| A Record|fulcio|x.x.x.x|

### oauth2.example.com

```
gcloud compute instances describe sigstore-dex \
--format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

|Type|Host| Value|
|---|---|---|
| A Record|dex|x.x.x.x|

> 📝 We do not need a domain for the certificate transparency log. This only
 communicate over a private network to Fulcio.

Next: [Rekor](04-rekor.md)
