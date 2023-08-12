# Guide

> It's not official guide. It's just collection of hints and possible solutions

## Copy project:

`git clone https://github.com/0xPolygonID/issuer-node.git`

## Install docker (Ubuntu)
See: `https://docs.docker.com/engine/install/ubuntu/`

## Makefile toolchain
check if exists: `make -version`

install: `sudo apt install make`

## Go 1.20

`https://www.linuxtechi.com/install-go-golang-on-ubuntu-linux/`


download Go:

`wget https://go.dev/dl/go1.20.7.linux-amd64.tar.gz`

unzip: 

`sudo tar -C . -xzf go1.20.7.linux-amd64.tar.gz `

Official documentation:

Now you need to setup Go language environment variables for your project. Commonly you need to set 3 environment variables as GOROOT, GOPATH and PATH.

GOROOT is the location where Go package is installed on your system.

`export GOROOT=/usr/local/go` 

GOPATH is the location of your work directory. For example my project directory is `~/Projects/Proj1` .

`export GOPATH=$HOME/Projects/Proj1 `

Now set the PATH variable to access go binary system wide.

`export PATH=$GOPATH/bin:$GOROOT/bin:$PATH `

All the above environment will be set for your current session only. To make it permanent add above commands in `~/.profile` file

You can chekc Go version: `go version`

# Postgres

`sudo apt update`

`sudo apt install postgresql postgresql-contrib`

Look at this tutorial: `https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-22-04-quickstart`

`sudo -u postgres psql`

`create user polygonid with password 'superstrongpassword';`

`create database platformid owner polygonid;`

# Redis

Look at this tutorial: `https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-22-04`

1. `sudo apt update`
2. `sudo apt install redis-server`
3. `sudo nano /etc/redis/redis.conf`
    then change `supervised no` into `supervised systemd`
4. `sudo systemctl restart redis.service`
5. Binding to localhost: `sudo nano /etc/redis/redis.conf`, then uncomment: `bind 127.0.0.1 ::1`
6. `sudo systemctl restart redis`
7. `sudo netstat -lnp | grep redis` - install `sudo apt install net-tools` if needed

Configuring a Redis Password
- `sudo nano /etc/redis/redis.conf`
- uncomment: `# requirepass foobared`
- change `foobared` into your strong password

Restart redis

# Hashicorp Vault

Official Packaging Guide:

`https://www.hashicorp.com/official-packaging-guide`

 > Remember to change `sudo apt install vault` instead of `sudo apt install consul`


 Copy `.env-api.sample` as `.env-api` and `.env-issuer.sample` as `.env-issuer`. Most important keys to set:

 `.env-api`:

 ```
ISSUER_API_UI_SERVER_URL=http://localhost:3002 <-- to change in prod env
ISSUER_API_UI_SERVER_PORT=3002
ISSUER_API_UI_AUTH_USER=user-api <-- change
ISSUER_API_UI_AUTH_PASSWORD=password-api <-- change

ISSUER_API_UI_ISSUER_DID=<YOUR DID> - after generating DID

 ```

`.env-issuer`:

 ```
ISSUER_SERVER_URL=http://localhost:3001 <-- to change in prod env
ISSUER_SERVER_PORT=3001
ISSUER_DATABASE_URL=postgres://polygonid:polygonid@postgres:5432/platformid?sslmode=disable
ISSUER_API_AUTH_USER=user-issuer <-- change
ISSUER_API_AUTH_PASSWORD=password-issuer <-- change
ISSUER_REVERSE_HASH_SERVICE_URL=http://localhost:3001
ISSUER_ETHEREUM_URL=https://polygon-mumbai.g.alchemy.com/v2/_NmBV73LfmMpwRnRnkWL1_QlPr_6PxTd
ISSUER_ETHEREUM_RESOLVER_PREFIX=polygon:mumbai
ISSUER_PROVER_SERVER_URL=http://localhost:8002
ISSUER_PROVER_TIMEOUT=600s
ISSUER_CIRCUIT_PATH=./pkg/credentials/circuits
ISSUER_REDIS_URL=redis://@redis:6379/1
ISSUER_SCHEMA_CACHE=false
ISSUER_KEY_STORE_TOKEN=<TOKEN> <-- add after unseal Vault

 ```

 
 # Standalone Mode Setup

 On step 2: Run make `build-local`. Sometimes it produces error, and possibly solution is described here:

`https://github.com/0xPolygonID/issuer-node/issues/471`

But in my case, install `sudo apt-get install build-essential` was enough.

------

Before Step 3 you have to run Vault and unseal

`https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-deploy`

Set on: `sudo nano /etc/vault.d/vault.hcl`

```
ui = true
api_addr = "http://localhost:8200"
cluster_addr = "https://localhost:8201"

# HTTP listener
listener "tcp" {
  address = "127.0.0.1:8200"
  tls_disable = 1
}
```

 - `export VAULT_ADDR='http://127.0.0.1:8200'`
 - `vault operator init` (sometimes it helps: `vault operator init -address=http://127.0.0.1:8200`)


 - start Vault: `vault server -config=/etc/vault.d/vault.hcl`

 Example output:

 ```
 ==> Vault server configuration:

Administrative Namespace: 
             Api Address: http://localhost:8200
                     Cgo: disabled
         Cluster Address: https://localhost:8201
   Environment Variables: DBUS_SESSION_BUS_ADDRESS, GODEBUG, GOTRACEBACK, HOME, LANG, LESSCLOSE, LESSOPEN, LOGNAME, LS_COLORS, MOTD_SHOWN, PATH, PWD, SHELL, SHLVL, SSH_CLIENT, SSH_CONNECTION, SSH_TTY, TERM, USER, XDG_DATA_DIRS, XDG_RUNTIME_DIR, XDG_SESSION_CLASS, XDG_SESSION_ID, XDG_SESSION_TYPE, _
              Go Version: go1.20.5
              Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: 
                   Mlock: supported: true, enabled: true
           Recovery Mode: false
                 Storage: file
                 Version: Vault v1.14.1, built 2023-07-21T10:15:14Z
             Version Sha: bf23fe8636b04d554c0fa35a756c75c2f59026c0
 ```

You have to init Vault: `vault operator init`

Possible error with permissions to `/opt/vault/data` - change permissions (for example for user `ubuntu`):

```
sudo chown -R ubuntu:ubuntu /opt/vault/data
sudo chmod -R 750 /opt/vault/data

```

 It's generate data. Save generated data in some txt file. Then you have to unseal vault. Example output:

 ```
Unseal Key 1: T+dNKwAfJz9y8fxB9Jyiw3lEOWJL33nmqxFg46B+rvOA
Unseal Key 2: b6mJREXsNlw4bzMt3u+qHl6hIuyojSD/81AXfjrxGhzJ
Unseal Key 3: jXiy7vOQUAKpi8Jvbtz+Z4uGXdy6+u8KZcL39x/xwRw2
Unseal Key 4: xu+fF+hASSMAF4uy1pkikik+5OfDYXJ3VtFm8f35wwbI
Unseal Key 5: rOzqz9SnI35Vp7soDMBiCdG/VA7sRlxRAljyNyTQA7wn

Initial Root Token: hvs.8DSonh4qzHnDKG5exKp0U68l

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
 ```

 then you can login: `vault login hvs.8DSonh4qzHnDKG5exKp0U68l`

------

# If you wan to start Vault as service:

## Enable and start the Vault service
Enable the systemd vault.service unit to allow the Vault service to start.

`sudo systemctl enable vault.service`

Start the Vault service.

`sudo systemctl start vault.service`

Check the status of the Vault service to ensure it is running.

`sudo systemctl status vault.service`

 --------

To create DID, you have to type:

`make generate-issuer-did`

It downloads docker to create DID. To write DID, maybe you'll have to change `docker-compose.yml` in `issuer-node/infrastructure` folder:

```
initializer:
    build:
      context: ../../
      dockerfile: ${DOCKER_FILE}
    env_file:
      - ../../.env-api
      - ../../.env-issuer
    volumes:
      - ../../did/:/did/
    command: sh -c "sleep 4s && ./migrate && ./issuer_initializer"
    network_mode: "host"
```

Create folder `did` inside `issuer-node`

----
Default port for issuer is: 3002, issuer user API: 3002. You can change this in: `api` and `api_ui` folder by editing `api.yaml` file.

