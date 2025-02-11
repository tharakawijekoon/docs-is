# Pass OIDC Authentication Parameters as a Request Object

This page guides you through passing OpenID Connect authentication request parameters in a self contained JWT, instead of passing plain request parameters using a sample application. A JWT that contains a set of request parameters as its claims is known as a request object.

{!./includes/oauth-playground.md!}

----

{!./includes/encrypt-id-tokens.md!}

----

{!./includes/oidc-custom-claims.md!}

-----

## Create request object

1. Create a user called "Tom" with login permission.

    For instructions, see [Add a User]({{base_path}}/guides/identity-lifecycles/admin-creation-workflow/) and [Add a Role]({{base_path}}/guides/identity-lifecycles/add-user-roles/).

2. Edit Tom's user profile and enter values for **email**, **country**, **challenge Question1**, and **challenge Question 2**. 
    For instructions, see [Edit User Profile]({{base_path}}/guides/identity-lifecycles/update-profile/).

3. Create a JWT with the following payload and sign(RSA256) it with the private key of the keystore you created above.

    ```
    {
      "client_id": "<client-id>",
      "sub": "<client-id>",
      "aud": [
        "https://localhost:9443/oauth2/token"
      ],
      "claims": {
        "userinfo": {
          "given_name": {
            "essential": true
          },
          "nickname": null,
          "email": {
            "essential": true
          },
          "customClaim2": {
            "essential": true
          }
        },
        "id_token": {
          "gender": null,
          "birthdate": {
            "essential": true
          },
          "customClaim1": {
            "essential": true
          }
        }
      },
      "iss": "<client-id>",
      "exp": 1516786878,
      "iat": 1516783278,
      "jti": "1003"
    }
    ```

    This creates a signed request object.

----

## Try it out

Try out both of the following flows and observe the responses.

1. First, test the flow without a signed request object:
    
    Use the `authorization_code` grant type for the user, and use the OIDC scope from the playground application to obtain an `id_token`. Then, retrieve user information using the access token.

2. Next, test the flow with a signed request object:

    Use the `authorization_code` grant for the user, and specify the authentication endpoint as `https://localhost:9443/oauth2/authorize?request=<JWT>`. Next, obtain the `id_token` and retrieve user information.

    !!! tip
        The JWT used here is the signed JWT created in the previous section of this guide.

When you analyze the responses of the two tests, you will observe that together with `customClaim2` retrieved in the userinfo response, an additional claim `customClaim1` is retrieved via the `id_token` when you configure the authorization code flow with a signed request object.   

-----

## Configure signature validation for request objects

Now that you understand how to pass OIDC authentication request parameters in a signed request object via WSO2 IS, you can configure a service provider to only accept signed request objects.

Request objects can either be signed or unsigned. Therefore, if you want to only accept signed request objects in an authorization request, you need to enable request object signature validation in the OAuth/OIDC configuration of the service provider.

1. Click **Service Providers > List** and **Edit** the service provider you created for the application.

2. Expand **Inbound Authentication Configuration**, and then expand **OAuth2/OpenID Connect Configuration**.

3. Select **Enable Request Object Signature Validation** to enforce signature validation for request object.

4. To verify that signature validation has been configured successfully, send a plain JWT instead of a signed one in the authorization code grant request.

    If signature validation is successfully enforced, the request should get rejected and you should see an error page.

    ![Signature validation successful]({{base_path}}/assets/img/samples/signature-validation-successful.png)

---- 

!!! info "Related topics"
    - [Use Advanced Parameters in Authentication Requests ]({{base_path}}/guides/login/oidc-parameters-in-auth-request/)