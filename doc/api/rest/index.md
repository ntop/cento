---
title: "RESTful API"
description: "Cento implements a RESTful API, using an embedded HTTP server, for dynamically reconfiguring the forwarding engine."
outline: [2, 3]
---
# RESTful API {#restful-api}

Cento implements a RESTful API, using an embedded HTTP server, for dynamically reconfiguring the forwarding engine.

The RESTful API can be enabled by adding the `[--rest-address|-Y] <address>`
option and accessed via HTTP (on port 8880 by default) or via HTTPS (on port 4443 by default).

HTTP and HTTPs ports can be changes using the `[--rest-http-port|-T] <port>`
and `[--rest-https-port|-S] <port>` options.

In order to use HTTP, a certificate is required. Create it with:

```console
mkdir -p /usr/local/share/cento/httpdocs/ssl
openssl req -new -x509 -sha1 -extensions v3_ca -nodes -days 365 -out cert.pem
cat privkey.pem cert.pem > /usr/local/share/cento/httpdocs/ssl/ssl-cert.pem
```

Sample command line using the --rest-address option for enabling the REST API:

```console
cento-ids -i zc:eth1 --aggregated-egress-queue --egress-conf egress.example --dpi-level 2 -v 4 --rest-address 127.0.0.1
```

Defaults settings:

> - HTTP port: `8880`
> - HTTPs port: `4443`

There are no default credentials: a password must be configured in Redis for a user before
the REST API will accept authentication as that user. See the *Authentication* section of the
REST API specification for how to set it.

Please check the REST API specifications for information about the available REST endpoints.

This is available on *cento-ids* and *cento-bridge* only as it is mainly used
to change the egress configuaration at runtime.

- [1. RESTful API Specification](/api/rest/api.md)
  - [1.1. Authentication](/api/rest/api.md#authentication)
  - [1.2. API](/api/rest/api.md#api)
- [2. Examples](/api/rest/examples.md)
  - [2.1. Identifying the base REST Endpoint](/api/rest/examples.md#identifying-the-base-rest-endpoint)
  - [2.2. Configuring Queue-Level Rules](/api/rest/examples.md#configuring-queue-level-rules)
  - [2.3. Configuring Subnet-Level Rules](/api/rest/examples.md#configuring-subnet-level-rules)
  - [2.4. Configuring Protocol-Level Rules](/api/rest/examples.md#configuring-protocol-level-rules)

