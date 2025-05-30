---
id: provider
title: Bitbucket Server Authentication Provider
sidebar_label: Bitbucket Server
description: Adding Bitbucket Server OAuth as an authentication provider in Backstage
---

:::info
This documentation is written for [the new backend system](../../backend-system/index.md) which is the default since Backstage
[version 1.24](../../releases/v1.24.0.md). If you are still on the old backend
system, you may want to read [its own article](https://github.com/backstage/backstage/blob/v1.37.0/docs/auth/bitbucketServer/provider--old.md)
instead, and [consider migrating](../../backend-system/building-backends/08-migrating.md)!
:::

The Backstage `core-plugin-api` package comes with a Bitbucket Server authentication provider that can authenticate
users using Bitbucket Server. This does **NOT** work with Bitbucket Cloud.

## Create an Application Link in Bitbucket Server

To add Bitbucket Server authentication, you must create an incoming application link. Follow the steps described in
the [Bitbucket Server documentation](https://confluence.atlassian.com/bitbucketserver/configure-an-incoming-link-1108483657.html)
to create one.

## Configuration

The provider configuration can then be added to your `app-config.yaml` under the root `auth` configuration:

```yaml
auth:
  environment: development
  providers:
    bitbucketServer:
      development:
        host: bitbucket.example.org
        clientId: ${AUTH_BITBUCKET_SERVER_CLIENT_ID}
        clientSecret: ${AUTH_BITBUCKET_SERVER_CLIENT_SECRET}
        ## uncomment to set lifespan of user session
        # sessionDuration: { hours: 24 } # supports `ms` library format (e.g. '24h', '2 days'), ISO duration, "human duration" as used in code
```

The Bitbucket Server provider is a structure with two configuration keys:

- `clientId`: The client ID that was generated by Bitbucket, e.g. `b0f868455c15dcdff5c5fb5d173ae684`.
- `clientSecret`: The client secret tied to the generated client ID.

### Optional

- `sessionDuration`: Lifespan of the user session.

### Resolvers

This provider includes several resolvers out of the box that you can use:

- `emailMatchingUserEntityProfileEmail`: Matches the email address from the auth provider with the User entity that has a matching `spec.profile.email`. If no match is found, it will throw a `NotFoundError`.
- `emailLocalPartMatchingUserEntityName`: Matches the [local part](https://en.wikipedia.org/wiki/Email_address#Local-part) of the email address from the auth provider with the User entity that has a matching `name`. If no match is found, it will throw a `NotFoundError`.

:::note Note

The resolvers will be tried in order but will only be skipped if they throw a `NotFoundError`.

:::

If these resolvers do not fit your needs, you can build a custom resolver; this is covered in the [Building Custom Resolvers](../identity-resolver.md#building-custom-resolvers) section of the Sign-in Identities and Resolvers documentation.

## Backend Installation

To add the provider to the backend, we will first need to install the package by running this command:

```bash title="from your Backstage root directory"
yarn --cwd packages/backend add @backstage/plugin-auth-backend-module-bitbucket-server-provider
```

Then we will need to add this line:

```ts title="packages/backend/src/index.ts"
//...
backend.add(import('@backstage/plugin-auth-backend'));
// highlight-add-start
backend.add(
  import('@backstage/plugin-auth-backend-module-bitbucket-server-provider'),
);
// highlight-add-end
//...
```

## Frontend Configuration

### Sign-in

To add the provider to the frontend, add the `bitbucketServerAuthApiRef` reference and
`SignInPage` component as shown in
[Adding the provider to the sign-in page](../index.md#sign-in-configuration).

### ScmAuth

For Backstage to be able to use the OAuth token of the logged-in user to access the Bitbucket Server API, you need to add it to the list of ScmAuth providers as shown in [Custom ScmAuthApi Implementation](../index.md#custom-scmauthapi-implementation) using the `ScmAuth.forBitbucketServer` method.
