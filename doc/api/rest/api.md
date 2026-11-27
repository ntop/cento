---
title: "RESTful API Specification"
description: "There are no default credentials. Authentication is disabled for every user until a"
outline: [2, 3]
---
# RESTful API Specification {#restful-api-specification}

## Authentication {#authentication}

There are no default credentials. Authentication is disabled for every user until a
password is explicitly configured for that user in Redis, as described below.

Please note that the HTTP basic access authentication should be used for authentication,
for example with `curl` it is possible to specify username and password with
`-u <user>:<password>` as in the command below:

```bash
curl -u <user>:<password> "https://localhost:4443/egress/aggregated/default?action=forward"
```

Please check the *Examples* section for more examples.

### Setting the REST API Password {#setting-the-rest-api-password}

Credentials are stored in Redis (configured via `--redis`). Passwords are kept as
salted SHA-256 hashes under the key `cento.user.<username>.password`, in the format
`sha256:<salt>:<hash>`, where `<salt>` is a random 16-byte hex string and
`<hash>` is the SHA-256 digest of `<salt>` concatenated with the plaintext
password.

To set or change the password for a user, run the command below by replacing
`admin` with the actual username and `newpassword` with the desired password.

```bash
SALT=$(openssl rand -hex 16); redis-cli SET cento.user.admin.password "sha256:${SALT}:$(printf '%s%s' "$SALT" 'newpassword' | openssl dgst -sha256 -r | awk '{print $1}')"
```

If the Redis instance listens on a non-default host or port, supply the connection details
with `-h host` and `-p port`.

## API {#api}

<a id="Aggregated Egress Queue"></a>

## Aggregated Egress Queue

### GET /egress/aggregated/[&lt;Queue Index>/]default

**Set the default policy for the aggregated egress queue**

**Description:** Set the policy to be applied by default to the traffic directed to the aggregated egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Default action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/aggregated/[&lt;Queue Index>/]ip

**Set a subnet-based policy for the aggregated egress queue**

**Description:** Set a policy to be applied to all traffic matching the provided subnet and directed to the aggregated egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `subnet` | query | Subnet to match in CIDR format | string |
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/aggregated/[&lt;Queue Index>/]protocol/&lt;Application Protocol>

**Set a layer-7 protocol-based policy for the aggregated egress queue**

**Description:** Set a policy to be applied to all traffic matching the provided layer-7 application protocol (nDPI) and directed to the aggregated egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

<a id="Balanced Egress Queue"></a>

## Balanced Egress Queue

### GET /egress/balanced/default

**Set the default policy for the balanced egress queue**

**Description:** Set the policy to be applied by default to the traffic directed to the balanced egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Default action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/balanced/ip

**Set a subnet-based policy for the balanced egress queue**

**Description:** Set a policy to be applied to all traffic matching the provided subnet and directed to the balanced egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `subnet` | query | Subnet to match in CIDR format | string |
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/balanced/protocol/&lt;Application Protocol>

**Set a layer-7 protocol-based policy for the balanced egress queue**

**Description:** Set a policy to be applied to all traffic matching the provided layer-7 application protocol (nDPI) and directed to the balanced egress queue or interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

<a id="Bridge"></a>

## Bridge

### GET /egress/bridge/default

**Set the default policy for the bridge**

**Description:** Set the policy to be applied by default to the traffic directed to the bridged interfaces.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Default action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/bridge/ip

**Set a subnet-based policy for the bridge**

**Description:** Set a policy to be applied to all traffic matching the provided subnet and directed to the bridged interfaces.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `subnet` | query | Subnet to match in CIDR format | string |
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.

### GET /egress/bridge/protocol/&lt;Application Protocol>

**Set a layer-7 protocol-based policy for the bridge**

**Description:** Set a policy to be applied to all traffic matching the provided layer-7 application protocol (nDPI) and directed to the bridged interface.

**Produces:** `application/json`

**Parameters**

| Name | Position | Description | Type |
|---|---|---|---|
| `action` | query | Action (forward, discard, shunt, slice-l4, slice-l3) | string |

**Responses**

`200` Successful operation, the returned object contains the actual status.; `400` Invalid status value.
