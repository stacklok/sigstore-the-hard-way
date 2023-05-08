# rekor

Rekor is sigstores signature transparency log.

Rekor requires running instances of trillian's log server and signer, with a database backend. A few different databases
can be used by trillian, for this example we will use mariadb.

Let's start by logging in:

```bash
gcloud compute ssh sigstore-rekor
```

## Dependencies

We need a few dependencies installed.

Update your system:

```bash
sudo apt-get update -y
```

If you want to save up some time, remove man-db first:

```bash
sudo apt-get remove -y --purge man-db
```

Grab the following packages:

```bash
sudo apt-get install mariadb-server git redis-server haproxy certbot -y
```

> 📝 redis-server is optional, but useful for a quick indexed search should you decide you need it. If you don't install it,
  you need to start rekor with `--enable_retrieve_api=false`

### Install latest golang compiler

Download and run the golang installer (system package are often older than what rekor requires):

```bash
curl -O https://storage.googleapis.com/golang/getgo/installer_linux
```

```bash
chmod +x installer_linux
```

```bash
./installer_linux
```

e.g.

```bash
Welcome to the Go installer!
Downloading Go version go1.20.4 to /home/luke/.go
This may take a bit of time...
Downloaded!
Setting up GOPATH
GOPATH has been set up!

One more thing! Run `source /home/$USER/.bash_profile` to persist the
new environment variables to your current session, or open a
new shell prompt.
```

As suggested run:

```bash
source /home/$USER/.bash_profile

go version
go version go1.20.4 linux/amd64
```

### Install rekor

We will work with the rekor repo (we grab the whole repo as we will need a some scripts):

```bash
mkdir -p ~/go/src/github.com/sigstore && cd "$_"
```

```bash
git clone https://github.com/sigstore/rekor.git && cd rekor/
```

And let's install both the server and the CLI:

```bash
go build -o rekor-cli ./cmd/rekor-cli
```

```bash
sudo cp rekor-cli /usr/local/bin/
```

```bash
go build -o rekor-server ./cmd/rekor-server
```

```bash
sudo cp rekor-server /usr/local/bin/
```

### Database

Trillian requires a database, let's first run `mysql_secure_installation` to
remove test accounts etc:

```bash
sudo mysql_secure_installation
```

The script is interactive. The following snippet captures the answers to
the script's prompts:
```bash
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorization.

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

We can now build the database.

Within the rekor repository is a `scripts/createdb.sh` script.

Edit this script and populate the root password `ROOTPASS` you set for the system
and then run the script (leave blank if not):

```bash
cd scripts/
sudo ./createdb.sh
Creating test database and test user account
Loading table data..
```

### Install trillian components

```bash
go install github.com/google/trillian/cmd/trillian_log_server@v1.3.14-0.20210713114448-df474653733c
```

```bash
sudo cp ~/go/bin/trillian_log_server /usr/local/bin/
```

```bash
go install github.com/google/trillian/cmd/trillian_log_signer@v1.3.14-0.20210713114448-df474653733c
```

```bash
sudo cp ~/go/bin/trillian_log_signer /usr/local/bin/
```

### Run trillian

The following are best run in two terminals, which are then left open (this
  helps for debugging):

```bash
trillian_log_server -http_endpoint=localhost:8090 -rpc_endpoint=localhost:8091 --logtostderr ...
```

```bash
trillian_log_signer --logtostderr --force_master --http_endpoint=localhost:8190 -rpc_endpoint=localhost:8191  --batch_size=1000 --sequencer_guard_window=0 --sequencer_interval=200ms
```

Alternatively, create bare minimal systemd services:

```bash
sudo bash -c 'cat << EOF > /etc/systemd/system/trillian_log_server.service
[Unit]
Description=trillian_log_server
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
ExecStart=/usr/local/bin/trillian_log_server -http_endpoint=localhost:8090 -rpc_endpoint=localhost:8091 --logtostderr ...
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF'
```

```bash
sudo bash -c 'cat << EOF > /etc/systemd/system/trillian_log_signer.service
[Unit]
Description=trillian_log_signer
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
ExecStart=/usr/local/bin/trillian_log_signer --logtostderr --force_master --http_endpoint=localhost:8190 -rpc_endpoint=localhost:8191  --batch_size=1000 --sequencer_guard_window=0 --sequencer_interval=200ms
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF'
```

Enable systemd services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable trillian_log_server.service
sudo systemctl enable trillian_log_signer.service
sudo systemctl start trillian_log_server.service
sudo systemctl start trillian_log_signer.service
sudo systemctl status trillian_log_server.service
sudo systemctl status trillian_log_signer.service
```

After the systemd services have been enabled, the output from the last command should be similar to:
```bash
● trillian_log_server.service - trillian_log_server
   Loaded: loaded (/etc/systemd/system/trillian_log_server.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-09-30 17:41:49 UTC; 8s ago
● trillian_log_signer.service - trillian_log_signer
   Loaded: loaded (/etc/systemd/system/trillian_log_signer.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-09-30 17:42:05 UTC; 12s ago
```

### Start rekor

Start rekor:

```bash
rekor-server serve --rekor_server.address=0.0.0.0 --trillian_log_server.port=8091
```

Note: Rekor runs on port 3000 on all interfaces by default.

Alternatively, you may create a bare minimal systemd service similar to trillian above:

```bash
sudo bash -c 'cat << EOF > /etc/systemd/system/rekor.service
[Unit]
Description=rekor
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
ExecStart=/usr/local/bin/rekor-server serve --rekor_server.address=0.0.0.0 --trillian_log_server.port=8091
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF'
```

Enable systemd services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable rekor.service
sudo systemctl start rekor.service
sudo systemctl status rekor.service
```

The last command should print:
```bash
 rekor.service - rekor
     Loaded: loaded (/etc/systemd/system/rekor.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-05-08 11:09:20 UTC; 2h 0min ago
   Main PID: 21612 (rekor-server)
      Tasks: 8 (limit: 2353)
     Memory: 21.8M
        CPU: 649ms
     CGroup: /system.slice/rekor.service
             └─21612 /usr/local/bin/rekor-server serve --rekor_server.address=0.0.0.0 --trillian_log_server.port=8091
```

### Let's encrypt (TLS) & HA Proxy config

Let's create a HAProxy config, set `DOMAIN` to your registered domain and your
private IP address:

```bash
DOMAIN="rekor.example.com"
IP="10.240.0.10"
```

Let's now run certbot to obtain our TLS certs:

```bash
sudo certbot certonly --standalone --preferred-challenges http \
      --http-01-address ${IP} --http-01-port 80 -d ${DOMAIN} \
      --non-interactive --agree-tos --email youremail@domain.com
```

Move the PEM chain into place:

```bash
sudo cat "/etc/letsencrypt/live/${DOMAIN}/fullchain.pem" \
    "/etc/letsencrypt/live/${DOMAIN}/privkey.pem" \
    | sudo tee "/etc/ssl/private/${DOMAIN}.pem" > /dev/null
```

Now we need to change certbot configuration for automatic renewal.

Prepare post renewal script:

```bash
cat /etc/letsencrypt/renewal-hooks/post/haproxy-ssl-renew.sh
#!/bin/bash

DOMAIN="rekor.example.com"

cat "/etc/letsencrypt/live/${DOMAIN}/fullchain.pem" \
    "/etc/letsencrypt/live/${DOMAIN}/privkey.pem" \
    > "/etc/ssl/private/${DOMAIN}.pem"

systemctl reload haproxy.service
```

Make sure the script has executable flag set:

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/haproxy-ssl-renew.sh
```

Replace port and address in the certbot's renewal configuration file for the domain (pass ACME request through the haproxy to certbot):

```bash
ls -l /etc/letsencrypt/renewal/rekor.example.com.conf
```

```bash
http01_port = 9080
http01_address = 127.0.0.1
```

Append new line:

```bash
post_hook = /etc/letsencrypt/renewal-hooks/post/haproxy-ssl-renew.sh
```

Prepare haproxy configuration:

```bash
cat > haproxy.cfg <<EOF
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

    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if letsencrypt-acl

    default_backend sigstore_rekor

backend sigstore_rekor
    server sigstore_rekor_internal ${IP}:3000

backend letsencrypt-backend
    server certbot_internal 127.0.0.1:9080
EOF
```

Inspect the resulting `haproxy.cfg` and make sure everything looks correct.

If so, copy it into place:

```bash
sudo cp haproxy.cfg /etc/haproxy/
```

Check syntax:

```bash
sudo /usr/sbin/haproxy -c -V -f /etc/haproxy/haproxy.cfg
```

### Start HAProxy

Let's now start HAProxy:

```bash
sudo systemctl enable haproxy.service
sudo systemctl restart haproxy.service
sudo systemctl status haproxy.service
```

Should print:
```bash
Executing: /lib/systemd/systemd-sysv-install enable haproxy
Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
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

Test automatic renewal

```bash
sudo certbot renew --dry-run
```

### Test rekor

Now we will test the operation of rekor. From the [rekor repository](https://github.com/sigstore/rekor) (so we have
some test files) we can perform an inclusion by adding some signing materials

```bash
rekor-cli upload --artifact tests/test_file.txt --public-key tests/test_public_key.key --signature tests/test_file.sig --rekor_server http://127.0.0.1:3000
```

Example:

```bash
git clone https://github.com/sigstore/rekor.git && cd rekor

rekor-cli upload --artifact tests/test_file.txt --public-key tests/test_public_key.key --signature tests/test_file.sig --rekor_server http://127.0.0.1:3000
Created entry at index 0, available at: http://127.0.0.1:3000/api/v1/log/entries/b08416d417acdb0610d4a030d8f697f9d0a718024681a00fa0b9ba67072a38b5
```
