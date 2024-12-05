# 🔓 Vault

{% hint style="info" %}
Vault is also used by Onyxia as the persistance layer for all saved configuration. If Vault is not configured, all user settings will be stored in the browser's local storage.
{% endhint %}

Onyxia-web use vault as a storage for two kinds of secrets:\
1\. secrets or information generate by Onyxia to store differents values (S3 sources configuration)\
2\. user secrets\

**Onyxia use the KV version 2 secret engine.**\
**Vault must be configured with JWT or OIDC authentification methods.**

As Vault needs to be initialized with a master key, it can't be directly configured with all parameters such as oidc or access policies and roles. So first step we create a vault with dev mode (do not use this in production and do your initialization with any of the recommanded configuration: Shamir, gcp, another vault).

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
 
DOMAIN=my-domain.net

cat << EOF > ./vault-values.yaml
server:
  dev:
    enabled: true
    # Set VAULT_DEV_ROOT_TOKEN_ID value
    devRootToken: "root"
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - host: "vault.lab.$DOMAIN"
    tls:
      - hosts:
          - vault.lab.$DOMAIN
EOF

helm install vault hashicorp/vault -f vault-values.yaml
```

#### Setting up JWT authentification for Vault

From Keycloak, create a client called "vault" (realm "datalab" as usually in this documentation)

1. _Root URL_: **https://vault.lab.my-domain.net/**
2. _Valid redirect URIs_: **https://vault.lab.my-domain.net/\*** and **https://datalab.my-domain.net/\***
3. _Web origins_: **\***

The expected value for the audience (aud) field of the JWT token by Vault is `vault`. You need to configure this in Keycloak.

1. Create a new Client scope: `vault`
2. Add Mapper by configuration
3. Choose Audience
   * Name: Audience for Vault
   * Included Client Audience: `vault`
   * Save

* Choose Clients: `vault`
  * Add Client Scope: `vault`

We will now configure Vault to enable `JWT` support, set policies for users permissions and initialize the secret engine.

You will need the Vault `CLI`. You can either download it [here](https://www.vaultproject.io/downloads) and configure `VAULT_ADDR=https://vault.lab.my-domain.net` and `VAULT_TOKEN=root` or exec into the vault pod `kubectl exec -it vault-0 -n vault -- /bin/sh` which will have vault `CLI` installed and pre-configured.

First, we start by creating a `JWT` endpoint in Vault, and writing information about Keycloak to the configuration. We use the same realm as usually in this documentation.

```
vault auth enable jwt
```

```
vault write auth/jwt/config \
    oidc_discovery_url="https://auth.lab.my-domain.net/auth/realms/datalab" \
    default_role="onyxia-user"
```

Onyxia uses only one single role for every user in Vault. This is in this tutorial `onyxia-user`\`. **To provide an authorization mechanism a policy is used that will depend on claims inside the JWT token.**

First you need to get the identifier (mount accessor) for the JWT authentification just created. You can use :

```
vault auth list -format=json | jq -r '.["jwt/"].accessor'
```

which should provide you something like `auth_jwt_xyz`. You will need it to **write a proper policy** by replacing the `auth_jwt_xyz` content with your own value.

#### Setting up a policy

Create locally a file named `onyxia-policy.hcl`.

You can notice that this policy is written for a KV version 2 secret engine mounted to the `onyxia-kv` path. The following policy is only working for personnal access because the entity name will be the preferred username in the JWT token.

{% code title="onyxia-policy.hcl" %}
```hcl
path "onyxia-kv/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["delete", "list", "read"]
}
```
{% endcode %}

{% hint style="info" %}
You can include access to any secret engine in the policy, which will be accessible within the services, though the Onyxia interface won’t utilize these permissions. If you have a use case where it would be beneficial for the Onyxia interface to access other secret engines, please let us know on Slack.
{% endhint %}

Allowing personal Vault tokens to access group storage in Vault is a bit more complex. We will map the group from the token into the entity’s metadata. The following policy maps the first 10 groups statically.

{% hint style="success" %}
If you have suggestions for a better authorization mechanism within Vault, please share them with us on Slack, as the current approach is not ideal.
{% endhint %}

{% code title="onyxia-policy.hcl" %}
```hcl
path "onyxia-kv/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/user-{{identity.entity.aliases.auth_jwt_xyz.name}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group0}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group0}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group0}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group1}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group1}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group1}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group2}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group2}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group2}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group3}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group3}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group3}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group4}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group4}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group4}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group5}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group5}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group5}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group6}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group6}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group6}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group7}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group7}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group7}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group8}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group8}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group8}}/*" {
  capabilities = ["delete", "list", "read"]
}

path "onyxia-kv/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group9}}/*" {
  capabilities = ["create","update","read","delete","list"]
}

path "onyxia-kv/data/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group9}}/*" {
  capabilities = ["create","update","read"]
}

path "onyxia-kv/metadata/projet-{{identity.entity.aliases.auth_jwt_xyz.metadata.group9}}/*" {
  capabilities = ["delete", "list", "read"]
}

```
{% endcode %}

Once the policy file is created, we can proceed with creating the policy.

```bash
vault policy write onyxia-policy onyxia-policy.hcl
```

We can go on with the role `onyxia-user`.

```bash
vault write auth/jwt/role/onyxia-user \
    role_type="jwt" \
    bound_audiences="vault" \
    user_claim="preferred_username" \
    claim_mappings="/groups/0=group0,/groups/1=group1,/groups/2=group2,/groups/3=group3,/groups/4=group4,/groups/5=group5,/groups/6=group6,/groups/7=group7,/groups/8=group8,/groups/9=group9" \
    token_policies="onyxia-policy"
```

We need to enable the secret engine.

```
vault secrets enable -path=onyxia-kv kv-v2
```

Then, you need to allow the URL https://datalab.my-domain.net in Vault's CORS settings.

```
vault write sys/config/cors allowed_origins="https://datalab.my-domain.net" enabled=true
```

You can finally modify your onyxia config file (in the helm values) :tada:

{% code title="onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
    regions:
      [
        {
          "id": "paris",
          ...
          "services": {...},
          "data": {...},
          "vault": {
              "URL": "https://vault.lab.my-domain.net",
              "kvEngine": "onyxia-kv",
              "role": "onyxia-user",
              "authPath": "jwt",
              "prefix": "user-",
              "groupPrefix" : "",
              "oidcConfiguration":
                {
                  "issuerURI": "https://auth.lab.my-domain.net/auth/realms/datalab",
                  "clientID": "vault",
                }
          }

    ]
```
{% endcode %}
