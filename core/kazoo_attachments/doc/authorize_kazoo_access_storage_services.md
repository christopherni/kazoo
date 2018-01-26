# Steps to authorize kazoo to access dropbox/google_drive/one_drive/etc applications

1. Create the app in the wanted storage application, e.g: dropbox:
    - After you are logged in go to https://www.dropbox.com/developers
    - On the left sidebar click `My apps`
    - Click `Create app` button
    - Choose any of the APIs (Dropbox or Dropbox Business)
    - Choose the access you need (App folder or Full Dropbox)
    - Name your app
    - Click `Create app` button
    - Go to the `OAuth 2` section in your app's settings and set redirect URIs to:
        - *ht<span>tp://</span>localhost:3000* if you are running Monster UI locally using `gulp`
        - *ht<span>tps://</span>yourdomain.com:yourport* if you have already deployed Monster UI in a production environment
        **NOTE**: Only `localhost` can be use with `HTTP` protocol, if you use a different domain you must use `HTTPS` protocol
    - Take note of *app_key* and *app_secret* values

2. Create the *whitelabel* document within the *accountid* database:
```
$ cat whitelabel.json
{
   "_id": "whitelabel",
   "company_name": "Company Name",
   "domain": "{DOMAIN}",
   "hide_powered": false,
   "custom_welcome": false,
   "ui_metadata": {
       "version": "master",
       "ui": "monster-ui",
       "origin": "branding"
   },
   "hide_registration": false,
   "hide_credits": false,
   "nav": {
       "learn_more": "",
       "help": ""
   },
   "port": {
       "terms": ""
   },
   "sso_providers": [
       {
           "url": "https://www.dropbox.com/oauth2/authorize",
           "name": "dropbox",
           "friendly_name": "DropBox",
           "params": {
               "client_id": "{DROPBOX_API_KEY}",
               "scopes": [
               ],
               "response_type": "code"
           },
           "authenticate_photoUrl": false
       }
   ],
   "realm_suffix": "Domain suffix",
   "applicationTitle": "",
   "custom_welcome_message": "",
   "companyName": "",
   "hidePasswordRecovery": false
}


$ curl -X PUT http://monster-ui.url/accounts/{account_id}/whitelabel
    -H "Content-Type: application/json"
    -H "X-Auth-Token: your_token_must_go_here"
    -d @whitelabel.json
```

3. Register the auth app by running `sup kazoo_auth_maintenance register_auth_app account_id app_key app_secret provider_id` (check example below):
    - example: *sup kazoo_auth_maintenance register_auth_app my_kazoo_account_id my_dropbox_app_key my_dropbox_app_secret dropbox*

4. Create provider document within system_auth database by running `sup kazoo_data_maintenance load_doc_from_file db_name full/path/to/file.json` (check example below):
    - example (let's say you have your file in your home dir and it is called *provider.json*):
        ```
        $ cat provider.json
        {
          "_id": "dropbox",
          "oauth_url": "https://api.dropboxapi.com/oauth2/token",
          "authorize_url": "https://www.dropbox.com/oauth2/authorize",
          "auth_url": "https://api.dropboxapi.com/oauth2/token",
          "pvt_type": "provider",
          "auth_module": "oauth",
          "access_code_url": "https://api.dropboxapi.com/oauth2/token",
          "profile_url": "https://api.dropboxapi.com/2/users/get_current_account",
          "profile_access_verb": "post",
          "profile_identity_field": "account_id",
          "profile_displayName_field": [
              "name",
              "display_name"
          ],
          "static_tokens": true,
          "fix_token_type_caps": true,
          "static_token_type": "Bearer"
        }
        ```
        `sup kazoo_data_maintenance load_doc_from_file system_auth /home/my_user/provider.json`

5. Now go to Monster UI and refresh the page so you can see the Dropbox (provider) button in the login form, then click it.

6. Authorize Kazoo to access Dropbox's preconfigured application by clicking the Authorize button in the popup.

7. If this is the first time that the user is authenticating using this SSO provider, Kazoo
returns and empty response indicating that user should first login using it's own Kazoo
credentials first to link the SSO application with its user. After login the user is linked
with the application and it no need to manual login again.
