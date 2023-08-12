#### Standalone Mode Guide Requirements

- [Docker Engine](https://docs.docker.com/engine/) 1.27
- Makefile toolchain
- Unix-based operating system (e.g. Debian, Arch, Mac OS X)
- [Go](https://go.dev/) 1.20
- [Postgres](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [Hashicorp Vault](https://github.com/hashicorp/vault)

#### Standalone Mode Setup

1. Copy `.env-api.sample` as `.env-api` and `.env-issuer.sample` as `.env-issuer`. Please see the [configuration](#configuration) section for more details.
1. Run `make build-local`. This will generate a binary for each of the following commands:
   - `platform`
   - `platform_ui`
   - `migrate`
   - `pending_publisher`
   - `notifications`
   - `issuer_initializer`
1. Run `make db/migrate`. This checks the database structure and applies any changes to the database schema.
1. Run `./bin/platform` command to start the issuer.
1. Run `./bin/pending_publisher`. This checks that publishing transactions to the blockchain works.
1. [Add](#import-wallet-private-key-to-vault) your Ethereum private key to the Vault.
1. [Add](#add-vault-to-configuration-file) the Vault to the config.
1. [Create](#create-issuer-did) your issuer DID.
1. _(Optional)_ To set up the UI with its own API, first copy `.env-ui.sample` as `.env-ui`. Please see the [configuration](#configuration) section for more details.
1. _Documentation pending: standalone UI setup._
