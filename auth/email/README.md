# Email and Password

Firebase provides 2 options when you require a way for your users to log in into your app without using Facebook, Twitter or Google.

* Email/Password Auth
* Anonymous Auth

Email and Anonymous Auth also uses the Google Identity Toolkit to achieve this.

## Getting Started

Follow these steps to enable Email/Password Auth:

1. Open the [Firebase console](https://firebase.google.com) and select your project.
2. Click the `Auth` option in the left side menu.
3. Click the `SIGN-IN METHOD` button in the top menu and then select `Email/Password` from the providers list.
4. Click the `Enable` toggle button and set it to `on` and then press the `Save` button.

You might also want to repeat these steps for the `Anonymous` provider only if you want to have Anonymous users.

## Implementation

You will require to import the following modules:

* `urllib.request`
* `urllib.error`
* `json`

All the requests must be sent using the `POST` method and with the following header: `"Content-Type": "application/json"`.

You must also JSON encode the request body. Python already includes a `json` module to achieve this.

It is strongly recommended to wrap your requests with a `try ... except` statement, Firebase returns useful error information from all its API calls.

```python
try:
    loader = urllib.request.urlopen(request)
except urllib.error.URLError as e:
    message = json.loads(e.read())
    print(message["error"]["message"])
else:
    print(loader.read())
```

## Registering a New User (Sign Up)

To register a new user you only require to provide the following parameters:

Name | Description
---|---
`email` | A valid formatted Email Address.
`password` | A non weak Password.
`returnSecureToken` | Set to: `True`

```python
def register(email, password):

    my_data = dict()
    my_data["email"] = email
    my_data["password"] = password
    my_data["returnSecureToken"] = True

    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/signupNewUser?key="+firebase_apikey, data=json_data, headers=headers)

    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```
A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#SignupNewUserResponse",
    "idToken": "<A long String>",
    "email": "someone@example.com",
    "refreshToken": "<A long String>",
    "expiresIn": "3600",
    "localId": "I7auXeJz2VgOYWmQajpAyjqYFr23"
}
```
The user will be automatically registered in the Auth section from your Firebase console.

For an Anonymous approach you don't need to specify anything in the request body. You will still get a response similar to the above just without an Email Address.

The `idToken` received from this response is used to perform further account management requests.
The `refreshToken` is used to get an `access_token` for Auth requests. For more information see the bottom of this page.

## Verifying Credentials (Sign In)

To sign in an user you only require to provide the following parameters:

Name | Description
---|---
`email` | The user's Email Address.
`password` | The user's Password.
`returnSecureToken` | Set to: `True`

```python
def signin(email, password):

    my_data = dict()
    my_data["email"] = email
    my_data["password"] = password
    my_data["returnSecureToken"] = True
				
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/verifyPassword?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#VerifyPasswordResponse",
    "localId": "I7auXeJz2VgOYWmQajpAyjqYFr23",
    "email": "someone@example.com",
    "displayName": "",
    "idToken": "<A long String>",
    "registered": true,
    "refreshToken": "<A long String>",
    "expiresIn": "3600"
}
```

Note that failing to enter the correct password 3 times in a row will block the IP for future login attempts for a while.

The `idToken` received from this response is used to perform further account management requests.
The `refreshToken` is used to get an `access_token` for Auth requests. For more information see the bottom of this page.

## Password Reset

To reset a password you only require to provide the following parameters:

Name | Description
---|---
`email` | The Email Address you want to send the Password recovery email.
`requestType` | Set to: `PASSWORD_RESET`

```python
def reset_password(email):

    my_data = dict()
    my_data["email"] = email
    my_data["requestType"] = "PASSWORD_RESET"
				
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/getOobConfirmationCode?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#GetOobConfirmationCodeResponse",
    "email": "someone@example.com"
}
```

An email with instructions will be sent to the desired email address. You can customize the template of emails in the Auth section from the Firebase console.

## Verify Email

When you require that Email Addresses are actually real you can prompt the user to confirm their Email Address by sending them an email with a confirmation link.

This is commonly used in message boards and ecommerce solutions. 

This method is similar to the Reset Password one, you need to provide the following parameters:

Name | Description
---|---
`email` | The Email Address you want to verify.
`requestType` | Set to: `EMAIL_VERIFY`
`idToken` | A long encoded String that contains user information. You can obtain this String from the response in the Sign Up and Sign In methods. 

```python
def verify_email(token, email):

    my_data = dict()
    my_data["email"] = email
    my_data["idToken"] = token
    my_data["requestType"] = "VERIFY_EMAIL"
				
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/getOobConfirmationCode?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#GetOobConfirmationCodeResponse",
    "email": "someone@example.com"
}
```

An email with instructions will be sent to the desired email address. You can customize the template of emails in the Auth section from the Firebase console.

## Get Account Info

This method is used for retrieving the logged in user information, very useful to check if a user has confirmed their Email Address.

This method only requires a valid Email Address and an `idToken`. You should call this method right after a Sign In or Sign Up request since those methods return a fresh `idToken`.

```python
def get_accountinfo(token, email):void

    my_data = dict()
    my_data["email"] = email;
    my_data["idToken"] = idToken;
			
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/getAccountInfo?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#GetAccountInfoResponse",
    "users": [
        {
            "localId": "I7auXeJz2VgOYWmQajpAyjqYFr23",
            "email": "someone@example.com",
            "emailVerified": true,
            "providerUserInfo": [
                {
                    "providerId": "password",
                    "federatedId": "someone@example.com",
                    "email": "someone@example.com",
                    "rawId": "someone@example.com"
                }
            ],
            "passwordHash": "UkVEQUNURUQ=",
            "passwordUpdatedAt": 1.473621716E12,
            "validSince": "1473621716",
            "lastLoginAt": "1473625365000",
            "createdAt": "1473621716000"
        }
    ]
}
```
## Set Account Info

To change the Email and or Password for an account you only require to specify which fields do you want to change and provide a valid `idToken`

```python
def set_accountinfo(token, email, password):

    my_data = dict()
    # You can comment the email or password values if you don't need to change them
    my_data["email"] = email
    my_data["password"] = password
    my_data["idToken"] = token

    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/setAccountInfo?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response from a Password change will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#SetAccountInfoResponse",
    "localId": "I7auXeJz2VgOYWmQajpAyjqYFr23",
    "email": "someone@example.com",
    "passwordHash": "UkXEHANURUR=",
    "providerUserInfo": [
        {
            "providerId": "password",
            "federatedId": "someone@example.com"
        }
    ]
}
```

A successful response from an Email change will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#SetAccountInfoResponse",
    "localId": "I7auXeJz2VgOYWmQajpAyjqYFr23",
    "email": "someone@example2.com",
    "passwordHash": "UkXEHANURUR=",
    "providerUserInfo": [
        {
            "providerId": "password",
            "federatedId": "someone@example2.com"
        }
    ],
    "idToken": "<A long String>"
}
```

The Email Address is updated to the new one but it needs to be confirmed or it will turn back to its previous state. An email containing a confirmation link is automatically sent to the original Email Address.

## Delete Account

To delete an account you only require to provide a valid `idToken`.

```python
def delete_account(id_token):
		
    my_data = {"idToken": id_token}
   			
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://www.googleapis.com/identitytoolkit/v3/relyingparty/deleteAccount?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "kind": "identitytoolkit#DeleteAccountResponse"
}
```

## Obtaining and Refreshing an Access Token

By default the `access_token` has an expiration time of 60 minutes, you can reset its expiration by requesting a fresh one.
To obtain or refresh an `access_token` you only need to provide the following parameters:

Name | Description
---|---
`refreshToken` | A long encoded String that contains user information. You can obtain it from a Sign In request.
`grant_type` | Set to: `refresh_token`

```python
def refresh_accesstoken(token):

    my_data = dict()
    my_data["grant_type"] = "refresh_token"
    my_data["refresh_token"] = token
				
    json_data = json.dumps(my_data).encode()
    headers = {"Content-Type": "application/json"}		
    request = urllib.request.Request("https://securetoken.googleapis.com/v1/token?key="+firebase_apikey, data=json_data, headers=headers)
  		
    try:
        loader = urllib.request.urlopen(request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
``` 

A successful response will look like the following JSON structure:

```json
{
    "access_token": "<A long String>",
    "expires_in": "3600",
    "token_type": "Bearer",
    "refresh_token": "<A long String>",
    "id_token": "<A long String>",
    "user_id": "ZJ7ud0CEpHYPF6QFWRGTe1U1Gvy2",
    "project_id": "545203846422"
}
```

Once you have got the `access_token` you are ready to perform secure operations against the Firebase Database and Firebase Storage services.

In this guide and examples, the `access_token` and `auth_token` represent the same value.
