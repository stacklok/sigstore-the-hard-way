# rekor

Rekor is sigstores signature transparency log.

Rekor requires running instances of trillian's log server and signer, with a database backend. A few different databases
can be used by trillian, for this example we will use mariadb.

Let's start by logging in

```
gcloud compute ssh --zone "europe-west1-b" "sigstore-rekor"  --project "sigstore-the-hard-way-proj"
```

## Dependencies

We need a few dependencies installed

Update your system

```
sudo apt-get update -y
```

Grab the following packages

```
sudo apt-get install mariadb-server git redis-server haproxy certbot -y
```


> 📝 redis-server is optional, but useful for a quick indexed search should you decide you need it. If you don't install it,
  you need to start rekor with `--enable_retrieve_api=false`

### Install go 1.16

Download and run the golang installer (system package is not yet 1.16)

```
curl -O https://storage.googleapis.com/golang/getgo/installer_linux
```

```
chmod +x installer_linux
```

```
./installer_linux
```

e.g.

```
Welcome to the Go installer!
Downloading Go version go1.16.6 to /home/luke/.go

This may take a bit of time...
Downloaded!
Setting up GOPATH
GOPATH has been set up!

One more thing! Run `source /home/$USER/.bash_profile` to persist the
new environment variables to your current session, or open a
new shell prompt.

source /home/$USER/.bash_profile
luke@sigstore-rekor:~$ go version
go version go1.16.6 linux/amd64
```

## Install rekor

We will work with the rekor repo (we grab the whole repo as we will need a some scripts)

```
mkdir -p ~/go/src/github.com/sigstore && cd "$_"
```

```
git clone https://github.com/sigstore/rekor.git && cd rekor/
```

And let's install both the server and the CLI

```
go build -o rekor-cli ./cmd/rekor-cli
```

```
sudo mv rekor-cli /usr/local/bin/
```

```
go build -o rekor-server ./cmd/rekor-server
```

```
sudo mv rekor-server /usr/local/bin/
```

## Database

Trillian requires a database, let's first run `mysql_secure_installation` to
remove test accounts etc.

```
sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

We can now build the database

Within the rekor repository is a `scripts/createdb.sh` script.


Edit this script and populate the root password `ROOTPASS` you set for the system
and then run the script

```
sudo ./createdb.sh
Creating test database and test user account
Loading table data..
```

## Install trillian components

```
go install github.com/google/trillian/cmd/trillian_log_server@v1.3.14-0.20210713114448-df474653733c
```

```
go install github.com/google/trillian/cmd/trillian_log_signer@v1.3.14-0.20210713114448-df474653733c
```

## Run trillian

The following are best run in two terminals, which are then left open (this
  helps for debugging)

```
trillian_log_server -http_endpoint=localhost:8090 -rpc_endpoint=localhost:8091 --logtostderr ...
```

```
trillian_log_signer --logtostderr --force_master --http_endpoint=localhost:8190 -rpc_endpoint=localhost:8191  --batch_size=1000 --sequencer_guard_window=0 --sequencer_interval=200ms
```

## Start rekor

Start rekor..

```
rekor-server serve --rekor_server.address=0.0.0.0 --trillian_log_server.port=8091
```
Note: Rekor runs on port 3000 by default

## Let's encrypt (TLS) & HA Proxy config


Let's create a HAProxy config, set `DOMAIN` to your registered domain and your
private IP address.

```
DOMAIN=rekor.yourdomain.com
IP=10.240.0.10
```

```
cat > ~/haproxy.cfg <<EOF
defaults
    timeout connect 10s
    timeout client 30s
    timeout server 30s
    log global
    mode http
    option httplog
    maxconn 3000
    log 127.0.0.1 local0

frontend haproxy
    #public IP address
    bind ${IP}:80
    bind ${IP}:443 ssl crt /etc/ssl/private/${DOMAIN}.pem

    # HTTPS redirect
    redirect scheme https code 301 if !{ ssl_fc }

    default_backend sigstore_rekor

    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if letsencrypt-acl

backend sigstore_rekor
```

Inspect the resulting `haproxy.cfg` and make sure everything looks correct.

If so, move it into place

```
sudo mv haproxy.cfg /etc/haproxy/
```

Let's now run certbot to obtain our TLS certs.

```
sudo certbot certonly --standalone --preferred-challenges http \
      --http-01-address ${IP} --http-01-port 80 -d ${DOMAIN} \
      --non-interactive --agree-tos --email youremail@domain.com
```

```
sudo cat "/etc/letsencrypt/live/${DOMAIN}/fullchain.pem" \
     "/etc/letsencrypt/live/${DOMAIN}/privkey.pem" \
     > "./${DOMAIN}.pem"

```

Move the PEM chain into place
```
sudo cp ./${DOMAIN}.pem /etc/ssl/private/${DOMAIN}.pem
```

Let's now start HAProxy

```
sudo systemctl enable haproxy.service

Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable haproxy

sudo systemctl start haproxy.service

sudo systemctl status haproxy.service
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2021-07-18 10:12:28 UTC; 58min ago
     Docs: man:haproxy(1)
           file:/usr/share/doc/haproxy/configuration.txt.gz
 Main PID: 439 (haproxy)
    Tasks: 2 (limit: 2322)
   Memory: 4.1M
   CGroup: /system.slice/haproxy.service
           ├─439 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
           └─444 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid

Jul 18 10:12:27 sigstore-fulcio systemd[1]: Starting HAProxy Load Balancer...
Jul 18 10:12:28 sigstore-fulcio systemd[1]: Started HAProxy Load Balancer.
```


Now we will test the operation of rekor. From the rekor repository (so we have
some test files) we can perform an inclusion by adding some signing materials

```
rekor-cli upload --artifact tests/test_file.txt --public-key tests/test_public_key.key --signature tests/test_file.sig --rekor_server http://127.0.0.1:3000
```

Example:
```
rekor-cli upload --artifact tests/test_file.txt --public-key tests/test_public_key.key --signature tests/test_file.sig --rekor_server http://127.0.0.1:3000
Created entry at index 0, available at: http://127.0.0.1:3000/api/v1/log/entries/b08416d417acdb0610d4a030d8f697f9d0a718024681a00fa0b9ba67072a38b5
```

Next: [Dex](05-dex.md)
