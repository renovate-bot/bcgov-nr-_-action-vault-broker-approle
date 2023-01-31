<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-vault-broker-approle)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-vault-broker-approle)](/../../pulls)
[![MIT License](https://img.shields.io/github/license/bcgov-nr/action-vault-broker-approle.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

# Vault Approle Token extractor through Vault Broker API

This action acquires an approle token from vault through the Broker API. This allows the team to access and generate tokens through the github action pipeline.

This is useful in CI/CD pipelines where you need to access a secret, get a vault token or anything vault related.

This tool is currently based on the existing documentation provided by 1team.

# Usage

```yaml
- uses: bcgov-nr/action-vault-broker-approle@main
  with:
    ### Required
    
    # Broker JWT Token
    broker_jwt: The JWT to be used on the broker

    # Role ID for Provision
    provision_role_id: The id of the role to be used during provisioning
    
    ### Usually a bad idea / not recommended

    # Overrides the default branch to diff against
    # Defaults to the default branch, usually `main`
    diff_branch: ${{ github.event.repository.default_branch }}

    # Repository to clone and process
    # Useful for consuming other repos, like in testing
    # Defaults to the current one
    repository: ${{ github.repository }}

    # Broker server address
    # Useful when consuming from a test server or other environment
    broker_url: https://nr-broker.apps.silver.devops.gov.bc.ca
      
    # Vault server address
    # Useful when interacting with other instances of vault
    vault_addr: https://vault-iit.apps.silver.devops.gov.bc.ca


    
```

# Example, Reading secrets

Read a secret from the vault

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/pr-open.yml`

```yaml
name: Pull Request

on:
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  builds:
    permissions:
      packages: write
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: Broker
        id: broker
        uses: bcgov-nr/action-vault-broker-approle@main
        with:
          broker_jwt: ${{ secrets.BROKER_JWT }}
          provision_role_id: ${{ secrets.PROVISION_ROLE }}
      - name: Import Secrets
        id: secrets
        uses: hashicorp/vault-action@v2.5.0
        with:
          url: https://vault-iit.apps.silver.devops.gov.bc.ca
          token: ${{ steps.broker.outputs.vault_token }}
          exportEnv: 'false'
          secrets: |
            apps/data/dev/super_secrets username | SECRET_USER;
            apps/data/dev/super_secrets password | SECRET_PWD;

```

# Example, Matrix Token Reads

Read from multiple environments.

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/pr-open.yml`

```yaml
name: Pull Request

on:
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  builds:
    permissions:
      packages: write
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        env: [dev, test]
    steps:
      - uses: actions/checkout@v3
      - name: Broker
        id: broker
        uses: bcgov-nr/action-vault-broker-approle@main
        with:
          broker_jwt: ${{ secrets.BROKER_JWT }}
          provision_role_id: ${{ secrets.PROVISION_ROLE }}
      - name: Import Secrets
        id: secrets
        uses: hashicorp/vault-action@v2.5.0
        with:
          url: https://vault-iit.apps.silver.devops.gov.bc.ca
          token: ${{ steps.broker.outputs.vault_token }}
          exportEnv: 'false'
          secrets: |
            apps/data/${{ matrix.env }}/super_secrets username | SECRET_USER;
            apps/data/${{ matrix.env }}/super_secrets password | SECRET_PWD;

```

# Output

If a token is acquired this action will output the token value as the `vault_token`.
See examples above.


<!-- # Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia. -->
