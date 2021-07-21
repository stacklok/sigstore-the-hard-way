# Provisioning Compute / Network Resources

We next need to create a network for our compute resources

`gcloud compute networks create sigstore-the-hard-way-proj --subnet-mode custom`

> 📝 if you recieve an  `reason: UREQ_PROJECT_BILLING_NOT_FOUND` error. You need
  to [enable billing on the API](https://support.google.com/googleapi/answer/6158867?hl=en)


We can now create a subnet with an internal range

```
gcloud compute networks subnets create sigstore \
  --network sigstore-the-hard-way-proj \
  --range 10.240.0.0/24
```

Create some firewall rules to allow tcp, udp and icmp protocols

```
gcloud compute firewall-rules create sigstore-the-hard-way-proj-allow-internal \
  --allow tcp,udp,icmp \
  --network sigstore-the-hard-way-proj \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
````

```
gcloud compute firewall-rules create sigstore-the-hard-way-allow-external \
  --allow tcp:22,tcp:80,tcp:443,icmp \
  --network sigstore-the-hard-way-proj \
  --source-ranges 0.0.0.0/0
```

Verify the rules were created

```
gcloud compute firewall-rules list --filter="network:sigstore-the-hardway-proj"
```

```
gcloud compute firewall-rules list --filter="network:sigstore-the-hardway-proj"

NAME                                  NETWORK                DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
default-allow-icmp                    default                INGRESS    65534     icmp                                False
default-allow-internal                default                INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
default-allow-rdp                     default                INGRESS    65534     tcp:3389                            False
default-allow-ssh                     default                INGRESS    65534     tcp:22                              False
sigstore-the-hard-way-proj-allow-internal  sigstore-the-hard-way-proj  INGRESS    1000      tcp,udp,icmp                        False
```

Create an external IP range

```
gcloud compute addresses create sigstore-the-hard-way-proj \
  --region $(gcloud config get-value compute/region)
```

Verify the external IP range

```
gcloud compute addresses list --filter="name=('sigstore-the-hardway-proj')"
```

e.g.

```
gcloud compute addresses list --filter="name=('sigstore-the-hardway-proj')"

NAME                        ADDRESS/RANGE  TYPE      PURPOSE  NETWORK  REGION        SUBNET  STATUS
sigstore-the-hard-way-proj  34.79.121.255  EXTERNAL                    europe-west1          RESERVED
```

Now we need to create four compute nodes for each service.

```
gcloud compute instances create sigstore-rekor \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.10 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-rekor
```

```
gcloud compute instances create sigstore-fulcio \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.11 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-fuclio
```

```
gcloud compute instances create sigstore-oauth2 \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.12 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-oauth2
```

```
gcloud compute instances create sigstore-ctl \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.13 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-ctl
```

Verify all compute instances are in a `RUNNING` state.

```
gcloud compute instances list --filter="tags.items=sigstore-the-hard-way-proj"

NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
sigstore-ctl     us-west1-c  e2-standard-2               10.240.0.13  34.145.19.46   RUNNING
sigstore-dex     us-west1-c  e2-standard-2               10.240.0.12  34.145.78.116  RUNNING
sigstore-fulcio  us-west1-c  e2-standard-2               10.240.0.11  34.83.166.248  RUNNING
sigstore-rekor   us-west1-c  e2-standard-2               10.240.0.10  34.83.40.13    RUNNING
```

Next: [Domain Configuration](03-domain-configuration.md)
