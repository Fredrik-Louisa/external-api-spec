# Fredrik & Louisa External API Specification

This repository contains the Protocol Buffer service definitions for the
Fredrik & Louisa external API. It is the canonical API contract between
Fredrik & Louisa and its integration partners.

This repository is **not** a client library and does not contain runnable code.
Consumers generate their own client code from the `.proto` files included here
(see [Code generation](#code-generation)).

## gRPC and Protocol Buffers

The API uses [gRPC](https://grpc.io/) with
[Protocol Buffers](https://protobuf.dev/) as the interface definition language.
If you are new to gRPC, see the
[Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/).

### Code generation

Clone this repository (or download a specific
[release](https://github.com/Fredrik-Louisa/external-api-spec/releases)) to get
the `.proto` files. Then use a code generation tool such as
[`protoc`](https://grpc.io/docs/protoc-installation/) or
[`buf`](https://buf.build/) to generate client stubs in your language. The
generated code gives you typed request/response objects and a client that
handles serialization and transport.

See the language-specific
[gRPC quick start guides](https://grpc.io/docs/languages/) for tooling and
plugin setup.

## Authentication

Authentication uses the OAuth 2.0 client credentials flow. An authentication
request looks like this:

```bash
curl --request POST \
  --url https://auth.fl-intern.no/oauth/token \
  --header 'content-type: application/json' \
  --data '{
    "client_id": "<your client ID>",
    "client_secret": "<your client secret>",
    "audience": "https://fl-intern.no/",
    "grant_type": "client_credentials"
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

Attach the token to every request as a bearer token in the `authorization`
header:

```
authorization: Bearer <JWT token>
```

### Token lifetime and caching

Tokens are valid for **900 seconds (15 minutes)**. Cache the token and refresh
it before it expires to avoid unnecessary token requests.

## Environments

### Staging

|                | URL                                             |
|----------------|-------------------------------------------------|
| Token endpoint | `https://auth.staging.fl-intern.no/oauth/token` |
| Audience       | `https://staging.fl-intern.no/`                 |
| API base URL   | `https://api.staging.fl-intern.no`              |

### Production

|                | URL                                     |
|----------------|-----------------------------------------|
| Token endpoint | `https://auth.fl-intern.no/oauth/token` |
| Audience       | `https://fl-intern.no/`                 |
| API base URL   | `https://api.fl-intern.no`              |

## Example

[grpcurl](https://github.com/fullstorydev/grpcurl) is useful for quickly
verifying connectivity:

```bash
grpcurl -v -H 'authorization: Bearer <your token>' \
  -proto fredrik_og_louisa/product/v1alpha/product.proto \
  api.staging.fl-intern.no:443 \
  fredrik_og_louisa.product.v1alpha.ProductService.ListProducts
```

## Versioning and compatibility

The API is currently in alpha and makes no stability guarantees — breaking
changes can happen at any time without a deprecation period.

This section describes how the **spec distribution** — this repository's
packaged `.proto` files — is versioned and released. That is separate from the
version of the API itself:

- **API version** — each service is versioned independently. The version is the
  segment in its Protocol Buffers package and directory path, e.g. the `v1alpha`
  in `fredrik_og_louisa.product.v1alpha`, so different services may sit at
  different versions. See [Graduation plan](#graduation-plan) for how `v1alpha`
  will advance.
- **Spec distribution version** — the semantic versioning, releases, and tags
  below apply to snapshots of this repository, not to the API surface. A
  release captures the state of all `.proto` files at a given commit.

### Semantic versioning

This repository follows [semantic versioning](https://semver.org/). Releases
are cut manually; each one is published as a `vMAJOR.MINOR.PATCH` Git tag (see
[Tags](#tags)) with a matching entry on the
[releases page](https://github.com/Fredrik-Louisa/external-api-spec/releases),
which serves as the changelog.

### Tags

Two kinds of tags are used in this repository:

- **Release tags** (`vMAJOR.MINOR.PATCH`) mark published releases. They are
  permanent: a release tag always points to the same commit and never moves.
  Pin to one of these when you need a stable, reproducible version.
- **Environment tags** (`staging` and `production`) track which version of the
  spec each [API environment](#environments) is currently serving. Each one
  points to the commit deployed to that environment and is moved whenever a new
  version is deployed there. Check out `staging` or `production` to generate
  clients that match what that environment is running today.

Environment tags are mutable, so do not rely on them as a fixed version — use a
release tag for that. Because they move, run `git fetch --tags --force` to
update their position if you have fetched them before.

### Pinning a version

To pin your integration to a specific version of the spec, check out the
relevant Git tag:

```bash
git clone --branch <tag> https://github.com/Fredrik-Louisa/external-api-spec.git
```

Or reference a specific tag in your build tooling (e.g. a Buf module pin or Git
submodule revision). To track the latest changes, use the `main` branch.

### Breaking changes

Breaking changes are communicated through

- **GitHub releases** on this repository
- **Direct outreach** to known consumers

### Graduation plan

When the API stabilizes, v1alpha will graduate directly to **v1**. There is no
planned beta phase. No timeline has been set for this transition.

### Release notifications

Watch this repository on GitHub to be notified of new releases.

## Support

For questions or
issues, [open an issue](https://github.com/Fredrik-Louisa/external-api-spec/issues).
