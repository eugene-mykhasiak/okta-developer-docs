### Direct authentication OOB flow

<div class="three-quarter">

![Sequence diagram that displays the back and forth between the resource owner, client app, and authorization server for OOB flow"](/img/authorization/oauth-oob-grant-flow.png)

</div>

<!-- Source for image. Generated using http://www.plantuml.com/plantuml/uml/
@startuml
skinparam monochrome true
actor "User" as user
participant "Client App (Your app)" as client
participant "Authorization Server (Okta) " as okta

autonumber "<b>#."
client -> user: Prompts user for username
user -> client: Enters username
client -> okta: Sends OOB authentication request to `/oob-authenticate`
okta -> client: Sends `oob_code`, `interval`, other parameters required by authenticator
okta -> user: Sends push notification
user <-> client: Per configured authenticator options, more interaction may occur
client -> okta: Polls `/token` at set `interval` with `oob_code`, `grant_type`
okta -> client: Responds with HTTP 400 `authorization_pending`
user -> okta: Opens the Okta Verify app and taps **Yes it's me**
user <-> okta: Per configured authenticator options, more interaction may occur
client -> okta: Polls `/token` endpoint
okta -> client: Responds with access token (optionally refresh token)
@enduml
-->

At a high level, this flow has the following steps:

1. Your client app prompts the user for their username in the app interface.
1. The user enters their username.
1. Your app sends the username as a `login_hint` and `channel_hint=push` to the Okta authorization server `/oob-authenticate` endpoint.

    Register your app so that Okta can accept the authorization request. See [Set up your app](#set-up-your-app) to register and configure your app with Okta. After registration, your app can make an authorization request to Okta. See [Request for tokens](#request-for-tokens).

1. Okta responds with the `oob_code`, `interval` in seconds to poll (default is `5`), and any other parameters required by the configured authenticator.

   >**Note:** For Okta Verify prompts with [number challenge](https://help.okta.com/okta_help.htm?type=oie&id=ext-config-okta-verify-options), `binding_method=transfer` and `binding_code` are also returned. See the [OOB response example](#oob-response-example).

1. Okta sends a push notification to the user.
1. Per configured authenticator options, more interaction may occur.

   >**Note:** For Okta Verify prompts with number challenge, the client displays the `binding_code` to the user.

1. Your app polls the Okta `/token` endpoint at the set `interval`. The `oob_code` and the [OOB grant type](https://developer.okta.com/docs/api/openapi/okta-oauth/oauth/tag/OrgAS/#tag/OrgAS/operation/token) (`grant_type=urn:okta:params:oauth:grant-type:oob`) are included in the requests.
1. Okta responds with an HTTP 400 `authorization_pending` error.
1. The user opens the Okta Verify app and taps **Yes it's me**.
1. Per configured authenticator options, more interaction may occur.

   >**Note:** For Okta Verify prompts with number challenge, three number options display in Okta Verify. The user then taps the number that matches the binding code sent by the client in step 6.

1. Your app polls the `/token` endpoint again.
1. Okta returns the requested tokens.
