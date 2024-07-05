# Design docs

> [!WARNING]  
> This is a very unfinished document where I brainstorm ideas and concepts for the new version of this project. Treat it as a very early draft.

## Authentication

## Authorization

### Scopes

Each scope is expressed as a Uniform Resource Name (URN). URNs are a way to uniquely identify resources and follow the format `urn:<namespace>:<resource>`. In our case, the format is `urn:<app name>:<auth>:<resource>:<access>`:

- `urn` is the prefix for all URNs
- Namespace is a combination of `<app name>` and `<auth>`
  - `<app name>` is the name of the app; we use `staart` in the examples below
  - Scopes can be for an organization or a user, so `<auth>` always starts with `org_` or `usr_`
- `<resource>` is the resource the scope is for
- Each scope must always end with the level of `<access>` which can be either `read` or `write`

For example:

- `urn:staart:org_1abc9c:*:read` -> read access to the organization with the ID `org_1abc9c`
- `urn:staart:usr_1abc9c:*:write` -> write access to the user with the ID `usr_1abc9c`

You can also have scopes that are limited to a specific resource, for example:

- `urn:staart:org_1abc9c:membership_16a085:read` -> read access to the membership with the ID `membership_16a085` in the organization with the ID `org_1abc9c`
- `urn:staart:usr_1abc9c:email:write` -> write access to the email of the user with the ID `usr_1abc9c`
- `urn:staart:org_1abc9c:membership_16a085:user:read` -> read access to the user in the membership with the ID `membership_16a085` in the organization with the ID `org_1abc9c`

Scope matching uses glob patterns, so you can also have wildcard scopes:

- `urn:staart:org_1abc9c:membership_*:read` -> read access to all memberships in the organization with the ID `org_1abc9c`
- `urn:staart:usr_*:write` -> write access to all users
- `urn:staart:org_*:membership_16a085:read` -> read access to the membership with the ID `membership_16a085` regardless of the organization

Other implementation notes:

- Each scope must have at least 4 `:` characters (`urn`, namespace, organization or user, resource, `read` or `write`)
  - `urn:staart:org_1abc9c:read` is not valid, instead use `urn:staart:org_1abc9c:*:read`
  - `urn:staart:usr_1abc9c:resource:subresource:subsubresource:read` is valid and can be used to create complex scopes
- `read` is a strict subset of `write`, so if a user has `write` access to a resource, they also have `read` access to it; therefore:
  - A token cannot have both `read` and `write` access to the same resource
- Superadmins by definition have all rights, so they have the scope `urn:staart:*:*:write`

## Unstructured ideas

- Use [`p-min-delay`](https://github.com/sindresorhus/p-min-delay) to delay responding to auth requests to at least 1 second to prevent timing attacks. For example, making a request to `/auth/login` will always take 1 second (even though it should usually be much shorter) so that it's impossible to know whether a user exists or not (since throwing a 404 is faster than when a user exists and password comparisons take place) and also useful for hash comparisons.
