# Kinde Elixir SDK

The Kinde Elixir SDK allows developers to use this library for the Kinde Business.

## Register for Kinde

If you haven’t already got a Kinde account, [register for free here](http://app.kinde.com/register) (no credit card required).

You need a Kinde domain to get started, e.g. `yourapp.kinde.com`.

## Installation

Add the following dependency in your project:

```elixir
{:kinde_sdk, path: "/path/to/kinde-elixir-sdk"}
```
and add to your extra applications:
```elixir
 def application do
    [
      extra_applications: [:logger, :kinde_sdk]
    ]
  end
```

## Configuration

### API Keys

You can set your keys in your application configuration. Use `config/config.exs`. For example:

```elixir
config :kinde_sdk,
  backend_client_id: "test_x1y2z3a1",
  frontend_client_id: "test_a1b2c3d4",
  client_secret: "test_112233",
  redirect_url: "http://text.com/callback",
  domain: "https://test.kinde.com",
  logout_redirect_url: "http://text.com/logout"
```

Optionally, you can also set `scope` as well.
```elixir
config :kinde_sdk,
  scope: "email"
```

You can also use `System.get_env/1` to retrieve the API key from an environment variables. For example:
```elixir
config :kinde_sdk,
  backend_client_id: System.get_env("KINDE_BACKEND_CLIENT_ID")
```

## Usage

Initialize your client like this:

```elixir
{conn, client} =
  KindeClientSDK.init(
    conn,
    Application.get_env(:kinde_sdk, :domain),
    Application.get_env(:kinde_sdk, :redirect_url),
    Application.get_env(:kinde_sdk, :backend_client_id),
    Application.get_env(:kinde_sdk, :client_secret),
    :client_credentials,
    Application.get_env(:kinde_sdk, :logout_redirect_url)
  )
```

### OAuth Flows (Grant Types)
KindeClientSDK implements three OAuth flows: Client Credentials flow, Authorisation Code flow and Authorisation Code with PKCE flow. Each flow can be used with their corresponding grant type when initializing a client.

| OAuth Flow | Grant Type | Type |
| ---------- | ---------- | ---- |
| Client Credentials | :client_credentials | atom |
| Authorisation Code | :authorization_code | atom |
| Authorisation Code with PKCE | :authorization_code_flow_pkce | atom |

### ETS Cache

KindeClientSDK implements persistant ETS cache for storing the client data and authenticating variables.

You may call your created client like this:
```elixir
client = KindeClientSDK.get_kinde_client(conn)
```

### Login
```elixir
conn = KindeClientSDK.login(conn, client)
```

### Register
```elixir
conn = KindeClientSDK.register(conn, client)
```

## Callbacks

When the user is redirected back to your site from Kinde, this will call your callback URL defined in the `redirect_url` config. You will need to route `/callback` to call a function to handle this.

```elixir
def callback(conn, _params) do
    {conn, client} = KindeClientSDK.get_token(conn)

    data = KindeClientSDK.get_all_data(conn)
    IO.inspect(data.access_token, label: "kinde_access_token")
  end
```

## Tokens

We can use Kinde helper function to get the tokens generated by `login` and `get_token` functions.

```elixir
data = KindeClientSDK.get_all_data(conn)
IO.inspect(data.login_time_stamp, label: "Login Time Stamp")
```

Or first calling the `get_token` function:
```elixir
{conn, client} = KindeClientSDK.get_token(conn)
```

### User details
You need to have already authenticated before you call the API, otherwise an error will occur.
```elixir
KindeClientSDK.get_user_detail(conn)
```

### Create an organization
To have a new organization created within your application
```elixir
conn = KindeClientSDK.create_org(conn, client)
conn = KindeClientSDK.create_org(conn, client)
```

### Logout
The Kinde SDK client comes with a logout method.
```elixir
conn = KindeClientSDK.logout(conn)
```

### Authenticated
Returns whether if a user is logged in.
```elixir
KindeClientSDK.authenticated?(conn)
```

### Claims
We have provided a helper to grab any claim from your id or access tokens. The helper defaults to access tokens:
```elixir
KindeClientSDK.get_claims(conn)

KindeClientSDK.get_claim(conn, "jti", :id_token)
```

### Permissions
We provide helper functions to more easily access permissions:
```elixir
KindeClientSDK.get_permissions(conn)

KindeClientSDK.get_permission(conn, "create:users")
```

See [Define user permissions](https://kinde.com/docs/user-management/user-permissions).

## Audience
An `audience` is the intended recipient of an access token - for example the API for your application. The audience argument can be passed to the Kinde client to request an audience be added to the provided token.

```elixir
additional_params = %{
      audience: "api.yourapp.com"
    }

KindeClientSDK.init(
  conn,
  Application.get_env(:kinde_sdk, :domain),
  Application.get_env(:kinde_sdk, :redirect_url),
  Application.get_env(:kinde_sdk, :backend_client_id),
  Application.get_env(:kinde_sdk, :client_secret),
  :authorization_code_flow_pkce,
  Application.get_env(:kinde_sdk, :logout_redirect_url),
  "openid profile email offline",
  additional_params
  )
```

For details on how to connect, see [Register an API](https://kinde.com/docs/developer-tools/register-an-api/)

## Overriding scope

By default the KindeSDK requests the following scopes:

- profile
- email
- offline
- openid

You can override this by passing scope into the KindeSDK.

## kinde SDK Reference

| Property | Type | Is required | Default | Description |
| -------- | ---- | ----------- | ------- | ----------- |
| domain | string | Yes | | Either your Kinde instance url or your custom domain. e.g: https://yourapp.kinde.com |
| redirect_url | string | Yes |  | The url that the user will be returned to after authentication |
| backend_client_id    | string | Yes |  | The id of your backend application |
| frontend_client_id    | string | Yes |  | The id of your frontend application |
| client_secret    | string | Yes |  | The id secret of your application - get this from the Kinde admin area |
| logout_redirect_url  | string | Yes |  | Where your user will be redirected upon logout  |
| scope  | string | No  | openid profile email offline | The scopes to be requested from Kinde  |
| additional_parameters    | map  | No  | %{} | Additional parameters that will be passed in the authorization request |
| additional_parameters - audience | string | No  |  | The audience claim for the JWT |
| additional_parameters - org_name | string | No  |  | The org claim for the JWT |
| additional_parameters - org_code | string | No  |  | The org claim for the JWT |


## SDK Functions

| Function | Description | Arguments | Usage |
| -------- | ---- | ----------- | ------- |
| login   | Constructs redirect url and sends user to Kinde to sign in    | conn, client  | ```KindeClientSDK.login(conn, client)```   |
| register     | Constructs redirect url and sends user to Kinde to sign up    | conn, client  | ```KindeClientSDK.register(conn, client)```   |
| logout  | Logs the user out of Kinde    | conn | ```KindeClientSDK.logout(conn)```  |
| get_token     | Returns the raw access token from URL after logged from Kinde    | conn | ```KindeClientSDK.get_token(conn)```   | 
| create_org    | Constructs redirect url and sends user to Kinde to sign up and create a new org for your business | conn, client  | ```KindeClientSDK.create_org(conn, client)``` |
| get_claims     | Gets all claims from an access or id token   | conn, atom | ```KindeClientSDK.get_claims(conn)``` or ```KindeClientSDK.get_claims(conn, :id_token)```  |
| get_claim     | Gets a claim from an access or id token   | conn, string, atom | ```KindeClientSDK.get_claim(conn, "jti")``` or ```KindeClientSDK.get_claim(conn, "jti", :id_token)```  |
| get_permissions   | Returns the state of a all permissions   | conn, atom  | ```KindeClientSDK.get_permissions(conn, :id_token)```   |
| get_permission   | Returns the state of a given permission   | conn, string  | ```KindeClientSDK.get_permission(conn, "create:users")```   |
| get_organization | Get details for the organization your user is logged into     | conn | ```KindeClientSDK.get_user_organization(conn)```     |
| get_user_detail  | Returns the profile for the current user  | conn | ```KindeClientSDK.get_user_detail(conn)``` |
| get_user_organizations  | Returns the org code from the user token  | conn | ```KindeClientSDK.get_user_organizations(conn)``` |
| get_cache_pid | Returns the Kinde cache PID from the `conn` | conn | ```KindeClientSDK.get_cache_pid(conn)``` |
| save_kinde_client | Saves the Kinde client created into the `conn` | conn | ```KindeClientSDK.save_kinde_client(conn)``` |
| get_kinde_client | Returns the Kinde client created from the `conn` | conn | ```KindeClientSDK.get_kinde_client(conn)``` |
| get_all_data | Returns all the Kinde data (tokens) returned | conn | ```KindeClientSDK.get_all_data(conn)``` |
