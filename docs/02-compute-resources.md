# Provisioning Compute / Network Resources

## Network Resources

We next need to create a network for our compute resources

```bash
$ gcloud compute networks create sigstore-the-hard-way-proj --subnet-mode custom
```

> 📝 if you recieve an `reason: UREQ_PROJECT_BILLING_NOT_FOUND` error. You need
  to [enable billing on the API](https://support.google.com/googleapi/answer/6158867?hl=en)

We can now create a subnet with an internal range

```bash
$ gcloud compute networks subnets create sigstore \
    --network sigstore-the-hard-way-proj \
    --range 10.240.0.0/24
```

Create some firewall rules to allow tcp, udp and icmp protocols

```bash
$ gcloud compute firewall-rules create sigstore-the-hard-way-proj-allow-internal \
    --allow tcp,udp,icmp \
    --network sigstore-the-hard-way-proj \
    --source-ranges 10.240.0.0/24,10.200.0.0/16
```

```bash
$ gcloud compute firewall-rules create sigstore-the-hard-way-allow-external \
    --allow tcp:22,tcp:80,tcp:443,icmp \
    --network sigstore-the-hard-way-proj \
    --source-ranges 0.0.0.0/0
```

Verify the rules were created

```bash
$ gcloud compute firewall-rules list --filter="network:sigstore-the-hard-way-proj"

NAME                                       NETWORK                     DIRECTION  PRIORITY  ALLOW                       DENY  DISABLED
sigstore-the-hard-way-allow-external       sigstore-the-hard-way-proj  INGRESS    1000      tcp:22,tcp:80,tcp:443,icmp        False
sigstore-the-hard-way-proj-allow-internal  sigstore-the-hard-way-proj  INGRESS    1000      tcp,udp,icmp                      False
```

## Compute Resources

Now we need to create four compute nodes for each service.

```bash
$ gcloud compute instances create sigstore-rekor \
    --async \
    --boot-disk-size 200GB \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.10 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-rekor
```

```bash
$ gcloud compute instances create sigstore-fulcio \
    --async \
    --boot-disk-size 200GB \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.11 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-fulcio
```

```bash
$ gcloud compute instances create sigstore-oauth2 \
    --async \
    --boot-disk-size 200GB \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.12 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-oauth2
```

```bash
$ gcloud compute instances create sigstore-ctl \
    --async \
    --boot-disk-size 200GB \
    --image-family debian-10 \
    --image-project debian-cloud \
    --machine-type e2-small \
    --private-network-ip 10.240.0.13 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet sigstore \
    --tags sigstore-the-hard-way-proj,sigstore-ctl
```

Verify all compute instances are in a `RUNNING` state.

```bash
$ gcloud compute instances list --filter="tags.items=sigstore-the-hard-way-proj"

NAME             ZONE            MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
sigstore-ctl     europe-west1-c  e2-small                   10.240.0.13  35.241.198.188  RUNNING
sigstore-fulcio  europe-west1-c  e2-small                   10.240.0.11  35.241.201.91   RUNNING
sigstore-oauth2  europe-west1-c  e2-small                   10.240.0.12  35.240.60.139   RUNNING
sigstore-rekor   europe-west1-c  e2-small                   10.240.0.10  35.233.82.12    RUNNING
```

Next: [Domain Configuration](03-domain-configuration.md)
