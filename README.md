# Fredrik & Louisa External API Specification

This repository is the canonical contractual specification for the Fredrik &
Louisa Integration Platform's APIs intended to be used by external partners

## Authenticating

Authentication uses OAuth 2.0.

An authentication request looks like this:

```bash
curl --request POST \
  --url https://auth.fl-intern.no/oauth/token \
  --header 'content-type: application/json' \
  --data '{
    "client_id":"<your client ID>",
    "client_secret":"<your client secret>",
    "audience":"https://fl-intern.no/",
    "grant_type":"client_credentials"
  }'
```

A successful authentication response looks like this:

```json
{
  "access_token": "<JWT token>",
  "expires_in": 900,
  "token_type": "Bearer"
}
```

The access token should be attached to each request as a bearer token in the
`authorization` header:
`authorization: Bearer <JWT token>`

## Environment specific values

### Staging

* Token request URL: `https://auth.staging.fl-intern.no/oauth/token`
* Audience: `https://staging.fl-intern.no/`
* API base URL: `https://api.staging.fl-intern.no`

### Production

* Token request URL: `https://auth.fl-intern.no/oauth/token`
* Audience: `https://fl-intern.no/`
* API base URL: `https://api.fl-intern.no`

## Example

After you've authenticated you'll want to send a request to see that you're
set up properly:

### gRPCurl

```bash
grpcurl -v -H 'authorization: Bearer <your token>' \
  -proto fredrik_og_louisa/product/v1alpha/product.proto \
  api.staging.fl-intern.no:443 \
  fredrik_og_louisa.product.v1alpha.ProductService.ListProducts
```