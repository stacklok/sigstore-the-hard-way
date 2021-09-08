# certificate transparency log

We will now install the certificate transparency log (CTL).

CTL requires running instances of trillian's log server and signer

Let's start by logging in

```
gcloud compute ssh sigstore-ctl
```

## Dependencies

```
sudo apt-get update -y
```

```
sudo apt-get install mariadb-server git haproxy wget certbot -y
```

## Install Go 1.16

```
curl -O https://storage.googleapis.com/golang/getgo/installer_linux
```

```
chmod +x installer_linux
```

```
./installer_linux
```

## Database

Trillian requires a databbase, let's first run `mysql_secure_installation`

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

We can now import the database as we used for rekor

```
wget https://raw.githubusercontent.com/sigstore/rekor/main/scripts/createdb.sh
```

```
wget https://raw.githubusercontent.com/sigstore/rekor/main/scripts/storage.sql
```

```
chmod +x createdb.sh
```

```
sudo ./createdb.sh
```

E.g 

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

```
go install github.com/google/trillian/cmd/createtree@v1.3.14-0.20210713114448-df474653733c
```

## Run trillian

The following is best run in two terminals which are then left open (this helps for debugging)

```
trillian_log_server -http_endpoint=localhost:8090 -rpc_endpoint=localhost:8091 --logtostderr ...
```

```
trillian_log_signer --logtostderr --force_master --http_endpoint=localhost:8190 -rpc_endpoint=localhost:8191  --batch_size=1000 --sequencer_guard_window=0 --sequencer_interval=200ms
```

## Install CTFE server

```
go get -u github.com/google/certificate-transparency-go/trillian/ctfe/ct_server
```

## Create a private key

Note: The private key needs a passphrase amd remember it as you will need it for `your_passphrase`
```
openssl ecparam -genkey -name prime256v1 -noout -out unenc.key
openssl ec -in unenc.key -out privkey.pem -des
rm unenc.key
```

## Create a Tree ID

Note: `trillian_log_server` needs to be running for this command to execute
```
LOG_ID=`createtree --admin_server localhost:8091`
```

## Set up the config file


```
cat > ct.cfg <<EOF
config {
  log_id: ${LOG_ID}
  prefix: "sigstore"
  roots_pem_file: "${HOME}/fulcio-root.pem"
  private_key: {
    [type.googleapis.com/keyspb.PEMKeyFile] {
       path: "${HOME}/privkey.pem"
       password: "<your_passphrase>"
    }
  }
}
EOF
```

Afterwards, open the file again and change `<your_passphrase>` to the one you used
when generating the private key.


`/root/fuclio-root.pem` is the root ID, we created in [04-fulcio](04-fulcio.md).

## Start the CT log

```
ct_server -logtostderr -log_config ct.cfg -log_rpc_server localhost:8091 -http_endpoint 0.0.0.0:6105
```

> 📝 The `-http_endpoint` flag uses the internal private IP. We don't need this facing externally
  (for this tutorial at least)

Next: [Configure Registry](08-configure-registry.md)
