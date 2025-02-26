# Users + Authentication

Every 8base workspace is initialized with native support for signing up, managing and authorizing your application's Users. This eliminates the requirement of managing emails and passwords or social sign-on providers without compromising on access to your user data.

## Users
**Users** is defined as a *System Table* in 8base, meaning that it is required in every workspace. That said, it is fully customizable using the Data Builder and can be configured as required by your application.

By default, the Users table has the following fields defined.

```javascript
{
	id: ID
	_description: String
	createdAt: DateTime
	updatedAt: DateTime
	createdBy: User
	email: String
	is8base: Boolean
	firstName: String
	lastName: String
	cellPhone: String
	workPhone: String
	workPhoneExt: String
	gender: String
	birthday: String
	language: String
	timezone: String
	avatar: File
	sentInvitations: Array
	permissions: Array
	roles: Array
}
````

### Managing Users in the Console
In most applications, Users records will be created as a part of a sign-up flow. However, in situations where a User must be created, updated or deleted manually by an admin it is easy to do so using the Data Viewer pane when reviewing the Users table.

![Creating a user in the 8base Management Console](../.gitbook/assets/data-viewer-create-user.png)

## Authorization
Under the hood, 8base utilizes [Auth0](https://auth0.com/) to manage your users' identities and ensure the best security standards are being met. All user accounts are by default stored in an Auth0 account that's managed by 8base. For upgraded workspace plans, the option of connecting ones own Auth0 account or an OpenID provider is available.

### Your Own Auth0 Account
To set up your own Auth0 account on 8base, there are only a few steps required. First, navigate to the `Settings > Authentication` of your workspace and create a new *Authentication Profile*. In the form that appears, select *Your Auth0 Account*.

All required information can be found in the settings of your Auth0 account.

![Connecting your own Auth0 account](../.gitbook/assets/auth-own-auth0.png)

### OpenID Connect
The ability to setup an authentication provider that supports the OpenID specification is available for workspaces on a *Profession* or *Enterprise* plan. To use this feature, there is some light setup required in the Managment Console and a custom *resolver* function that should be deployed to your project's workspace.

### Sign-on Providers
Sign-on providers can easily be enable/disabled in the *8base Authentication Settings* section of the workspace's Authentication view. In order to use this feature, at least one authentication profile must be created with the type set to "8base Authentication".

![Creating an Authentication Profile](../.gitbook/assets/signon-provider-form.png)

Each sign-on provider requires a *Client ID* and *Client Secret* to be configured. These credentials can be collected from the sign-on provider(s) you wish to configure. Once collected, enter the credentials into the relevant sign-on provider form before clicking "Enable Sign-On Provider" and "Save".

![Enabling a Sign-on Provider](../.gitbook/assets/signon-provider-config.png)

##### Configuring the OpenID Settings
In the 8base Management Console you're able to configure one or more authentication providers under `Settings > Authentication`. Click the "+" button and fill out the provider form, selecting *OpenID* as the type and adding a OpenID Provider URL. Once completed, the record will be saved to your *Authentication Profiles*.

![Adding an OpenID Authentication Provider in 8base](../.gitbook/assets/openid-settings.png)

##### getToken Resolver
A custom *getToken* resolver mutation function should be deployed to the workspace. This can be done by installing the [8base CLI](../development-tools/cli/README.md).

In the provided *getToken* function, the relevant environment variables are being accessed - as they are set in the Management Console - to provide the required credentials and configurations. A request is then made to the authentication provider to query or create the authenticating user from the database and return the user's token.

{% code-tabs %}
{% code-tabs-item title="8base.yml" %}
```yaml
functions:
  getToken:
    handler:
      code: src/getToken.ts
    type: resolver
    schema: src/getToken.graphql
```
{% endcode-tabs-item %}

{% code-tabs-item title="handler.js" %}
```javascript
const { URLSearchParams } = require('url');
const fetch = require('node-fetch');
const gql = require('graphql-tag');
const jwtDecode = require('jwt-decode');

const APP_ID_CLIENT_ID = process.env.APP_ID_CLIENT_ID;
const APP_ID_TENANT_ID = process.env.APP_ID_TENANT_ID;
const APP_ID_SECRET = process.env.APP_ID_SECRET;
const APP_ID_URL = process.env.APP_ID_URL;
const TOKEN_PATH = '/token';

const CLIENT_REDIRECT_URI = process.env.CLIENT_REDIRECT_URI;

const CURRENT_USER_QUERY = gql`
  query CurrentUser {
    user {
      id
      email
    }
  }
`;

const USER_SIGN_UP_MUTATION = gql`
  mutation UserSignUp($user: UserCreateInput!, $authProfileId: ID) {
    userSignUpWithToken(user: $user, authProfileId: $authProfileId) {
      id
      email
    }
  }
`;

export default async (event: any, context: any) => {
  const body = new URLSearchParams();

  body.append('grant_type', 'authorization_code');
  body.append('code', event.data.code);
  body.append('client_id', APP_ID_CLIENT_ID);
  body.append('redirect_uri', CLIENT_REDIRECT_URI);

  let token;
  let email;

  try {
    let tokenResponse = await fetch(`${APP_ID_URL}${APP_ID_TENANT_ID}/${TOKEN_PATH}`, {
      body,
      headers: {
        'Authorization': 'Basic ' + Buffer.from(`${APP_ID_CLIENT_ID}:${APP_ID_SECRET}`).toString('base64'),
        'Content-Type': 'application/x-www-form-urlencoded'
        'Accept': 'application/json',
      },
      method: 'post',
    });

    ({ id_token: token } = await tokenResponse.json());

    try  {
      await context.api.gqlRequest(CURRENT_USER_QUERY, {}, {
        authorization: token,
      });
    } catch (e) {
      ({ email } = jwtDecode(token));

      await context.api.gqlRequest(USER_SIGN_UP_MUTATION, {
        user: {
          email,
        },
        authProfileId: event.data.authProfileId,
      }, {
        authorization: token,
      });
    }
  } catch (e) {
    console.log(e);
    throw Error('Authorization Error');
  }

  return {
    data: {
      token,
    },
  };
};
```
{% endcode-tabs-item %}

{% code-tabs-item title="schema.graphql" %}
```javascript
type TokenResult {
  token: String!
}

extend type Mutation {
  getToken(code: String!, authProfileId: ID!): TokenResult
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

##### Setting Environment Variables
To set environment variables that can be accessed from within custom functions, open up your workspace and navigate to `Settings > Environment Variables`. Here, any key-value pair may be securely stored and accessed from within your functions at `process.env.<ENV_VARIABLE_KEYNAME>`.

![Environment variables manager in the 8base Management Console](../.gitbook/assets/openid-env-variables.png)

##### Troubleshooting
If you're unable to get the authentication provider to work and are receiveing a "Not Authorized" error message, you may need to update the associated role and its API permissions. You can do this by first ensuring that the configured provider has an associated role, like *Guest*. Next, navigate to `Settings > Roles > [ROLE_NAME] > Data` and ensure that the role is enabled for the *Get Token* function call.
