# Privacy Agent API

This document describes the Privacy Agent public API.

# Overview

The Privacy Agent is WebShield Software delivered as a Docker Container that runs in a Privacy Domain and instantiates Privacy Pipes that act as privacy preserving HTTP(s) forward and reverse proxy when moving data across Privacy Domains. A forward proxy takes as input data and optional metadata and outputs a Privacy Graph that is POST|etc to a destination over HTTP(s), and reverse proxy takes as input a Privacy Graph and produces data that is POST|etc to a destination over HTTP(s).

The Privacy Adapter is developed by external parties and typically sits in-front or behind a Privacy Agent. A Privacy Adapter is responsible for
  - optional transport, format, and schema translation
  - optionally creating any Custom Trust Criteria that needs to be included in the Privacy Graph.
  - optionally creating any Custom Provenance that needs to be included in the Privacy Graph

#Privacy Pipes

Privacy pipes act as privacy preserving HTTP(s) proxies allowing services running inside or outside of Privacy Domains to send and receive data with other services inside or outside of a Privacy Domains whilst meeting the required Trust Criteria.

A privacy pipe can act as either as
 - a privacy preserving forward proxy that accepts http verbs, headers and input data; and applies Privacy Algorithm that cryptographically obfuscates data and binds the trust criteria and provenance producing a privacy graph and then forwards the HTTP verb, headers, and graph onto the next service.
 - a privacy preserving reverse proxy that accepts http verbs, headers and a privacy graph, it verifies and unwraps the privacy graph and if ok then passes the http verb, headers and data onto the proxied service.

Privacy pipes are configured within a privacy agent running in a privacy domain and are manifested as a URL specified in the configuration.

   ```
   https://{domain}/{base_url}/pa/v1/pipe/{name}
   ```

Typically the pipe name reflects the service that is being proxied, for example AUTH-NET privacy domain provides an evaluate URL that accepts a HTTP(s) POST with a JSON Body. If ACME wanted to integrate its services to the AUTH-NET service they would configure a forward privacy pipe within their privacy domain with say the following URL

   'https://pd.acme.com/pa/v1/pipe/evaluate'

The ACME services would then POST to the privacy pipe URL using either of the following patterns

*Pattern One* If the body is already JSON or 'x-www-form-urlencoded' and the PN Trust Criteria and PN Provenance are good enough then the ACME services can just POST the evaluate JSON body to privacy pipe URL as if they were posting directly to the protected service and would receive a HTTP response as if it came directly from th protected service. The only difference is that they may receive an HTTP Bad Gateway response code if there is a internal problem with the privacy pipe or verification of a privacy graph fails.

```json
  // A POST to the AUTH-NET evaluate service has following request body that can be posted to the Privacy Agent as is
  // req.Header.Set("Content-Type", "application/json") - pass through pipe
	// req.Header.Set("X-Auth-Token", "Bearer "+bearerToken) - passed through pipe, the AUTH-NET service uses this to authenticate ACME can uses the service
  {
    "requestId":"1",
    "context": {
      "email": "rich@webshield.io",
      "tag0": "https://schema.webshield.io/medical_records",
      },
    "config": {"param1": "1"},
  }
```

`{"requestId":"1","context": {"email": "rich@webshield.io", "tag0": "https://schema.webshield.io/medical_records"}, "config": {"param1": "1"}}`

*Pattern Two* If the body is not JSON or Custom Trust Criteria and Custom PN Provenance needed to be added then the ACME services can POST the evaluate JSON body a custom Privacy Adaptor that pre-processes the data and then posts it onto the Privacy Agent as described in the previous point. In this case the information is passed to the Privacy Agent in a Privacy Pipe Request Object shown below to the Pipe URL. The Privacy Agent recognizes that payload is this object and processes accordingly. See Trust Criteria and Provenance for more data.

```json

// Pipe request POST to privacy agent
// A POST to the AUTH-NET evaluate service has following request body shown in Data
// req.Header.Set("Content-Type", "application/json") - pass through pipe
// req.Header.Set("X-Auth-Token", "Bearer "+bearerToken) - passed through pipe, the AUTH-NET service uses this to authenticate ACME can uses the service
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [
    {
      "requestId":"1",
      "context": {
        "email": "rich@webshield.io",
        "tag0": "https://schema.webshield.io/medical_records",
        },
      "config": {"param1": "1"},
    },
  ],
  "pn_datamodel": "<id of pn data model for this data>",
  "data_custom_trust_criteria": "<pn_custom_trust_criteria object>",
  "default_destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria object>",
  "data_custom_provenance": "<pn_custom_provenance object>",
}
```

# Securing the connection between the client and a Privacy Pipe.
The connection between the client and the Privacy pipe can be secured as follows
  - All communication between a client and a privacy pipe is over TLS
  - Optionally a web proxy such as NGINX can be placed in-front of a Privacy Pipe that can authenticate the client
  - Optionally the Privacy Pipes can accept JWTs from the client that provides integrity, and authentication of the client if the JWT cannot be verified.
  - Optionally the Privacy Pipes can accept a JWE containing a JWT from the client that is encrypted with the public key of the Privacy Agent

# Supporting APIs

Getting Privacy Agent status

  ```
  GET https://{domain}/{base_url}/pa/v1/status
  ```

Getting a Privacy Domain Public Signing Key to validate that a results is signed by a Privacy Domain.

  ```
  GET https://{domain}/{base_url}/pa/v1/privacy_domain/:name/sign/jwks.json
  ```

Getting a Privacy Domain Public Encryption Key to encrypt data that want to a Privacy Agent. Note this is used to send to the data owners Privacy Agent which will decrypt and then apply the necessary Privacy Algorithm to re-encrypt before sending out of the data owners Privacy Domain.

  ```
  GET https://{domain}/{base_url}/pa/v1/privacy_domain/:name/encrypt/jwks.json
  ```
