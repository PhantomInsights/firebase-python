# Firebase Storage

Firebase Storage is based on Google Cloud Storage, a very easy and flexible solution for storing all kinds of files.

Files are stored the same way as in your personal computer, using a tree hierarchy. This means there's a root folder which can contain more folders and those folders can contain additional folders and files.

It is strongly recommended to avoid the use of special characters when naming files and folders.

You will need special care for the slash character `(/)`. You need to always replace it with `%2F` or else it will throw an error indicating that the path is incorrect.

All these examples only require the `urllib` and `json` modules to run.

In the context of this guide a `bucket` is a synonymous to your Firebase project.

## Firebase Rules

The Firebase Rules are a flexible way to set permissions on who can access certain files and data.

By default all the data is private and can only be accessed by Authenticated users.

To modify the Rules follow these steps:

1. Open the [Firebase console](https://firebase.google.com)
2. Select your project.
3. Click on the Storage option from the left side menu.
4. Click on `RULES` from the top menu.

## Default Rules

```
service firebase.storage {
  match /b/<YOUR-PROJECT-ID>.appspot.com/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

These rules are very similar to the `Auth` default rules. They mean that any authenticated user can upload, delete and modify all files from your bucket.

## Public Reading and Writing

The following rules allows any user to upload, delete and modify any files from your entire bucket. Use this only while developing and testing.

```
service firebase.storage {
  match /b/<YOUR-PROJECT-ID>.appspot.com/o {
    match /{allPaths=**} {
      allow read, write;
    }
  }
}
```

## Public Reading

Use the following rules if you need to host some files that anyone on the Internet can download, such as images, documents, audio and video.

```
service firebase.storage {
  match /b/<YOUR-PROJECT-ID>.appspot.com/o {
    match /{allPaths=**} {
      allow read;
    }
  }
}
```

The following rules will allow anyone to read but not to write the contents of a folder named `public`.

```
service firebase.storage {
  match /b/<YOUR-PROJECT-ID>.appspot.com/o {
    match /public/} {
      allow read;
    }
  }
}
```

## Uploading a File

To upload a file you require to send it as `bytes`. The following examples show the most common scenarios.
If you upload the same file to the same location, it will be replaced with new metadata.

In this example we are uploading a file from a predefined location. A common example is syncing a save game after a game session:

```python
def upload_file():

    my_file = open("savegame.data", "rb")
    my_bytes = my_file.read()
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Content-Type": "text/plain"}

    my_request = urllib.request.Request(my_url, data=my_bytes, headers=my_headers, method="POST")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "name": "savegames/savegame.data",
    "bucket": "<YOUR-PROJECT-ID>.appspot.com",
    "generation": "1473948546121000",
    "metageneration": "1",
    "contentType": "text/plain",
    "timeCreated": "2016-09-15T14:09:06.053Z",
    "updated": "2016-09-15T14:09:06.053Z",
    "storageClass": "STANDARD",
    "size": "10450",
    "md5Hash": "7aIjAPS+Sd0DaF5SmGTUYw==",
    "contentEncoding": "identity",
    "crc32c": "DObTDw==",
    "etag": "CKj6iJzGkc8CEAE=",
    "downloadTokens": "7232aa46-f2e1-4df5-9698-d9c77b88ad5f"
}
```

Your new file and a `save_games` folder will instantly appear in the Storage section from the Firebase console.

The `Content-Type` header doesn't need to be accurate, but it is recommended to set it properly. You can use the `mimetypes` module to help setting it.

```python
my_type = mimetypes.guess_type("cat.jpg")
my_headers = {"Content-Type": my_type[0]}
```

## Uploading a File with Auth

Authorizing requests for Firebase Storage is a bit different than in Firebase Database. Instead of adding an `auth` parameter in the URL with the `authToken`, we add it into a `header`.

```python
def upload_file(auth_token):

    my_file = open("savegame.data", "rb")
    my_bytes = my_file.read()
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Authorization": "Bearer "+auth_token, "Content-Type": "text/plain"}

    my_request = urllib.request.Request(my_url, data=my_bytes, headers=my_headers, method="POST")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "name": "save_games/savegame.data",
    "bucket": "<YOUR-PROJECT-ID>.appspot.com",
    "generation": "1473948546121000",
    "metageneration": "1",
    "contentType": "text/plain",
    "timeCreated": "2016-09-15T14:09:06.053Z",
    "updated": "2016-09-15T14:09:06.053Z",
    "storageClass": "STANDARD",
    "size": "10450",
    "md5Hash": "7aIjAPS+Sd0DaF5SmGTUYw==",
    "contentEncoding": "identity",
    "crc32c": "DObTDw==",
    "etag": "CKj6iJzGkc8CEAE=",
    "downloadTokens": "7232aa46-f2e1-4df5-9698-d9c77b88ad5f"
}
```

## Deleting a File

Deleting a file is very simple, you only need to send a `DELETE` request with the file path syou want to delete.

```python
def delete_file():
	
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_request = urllib.request.Request(my_url, method="DELETE")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will return an [empty String](https://cloud.google.com/storage/docs/json_api/v1/objects/delete).

## Deleting a File with Auth

To delete a file with authentication you only need to provide an `auth_token` in the `Authorization` header and the file path in a `DELETE` request.

```python
def delete_file(auth_token):
	
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Authorization": "Bearer "+auth_token}

    my_request = urllib.request.Request(my_url, headers=my_headers, method="DELETE")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

## Update Metadata

To modify the metadata generated after your upload a file you will only require to `JSON` encode which fields do you need to update and send them in a `PATCH` request. This is very similar as updating the Firebase Database data.

Click [here](https://firebase.google.com/docs/storage/web/file-metadata) for a list of all the fields that can be modified. In the following example we are going to change the `contentType`.

```python
def update_metadata():

    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Content-Type": "application/json"}
    my_data = {"contentType": "application/binary"}
    
    json_data = json.dumps(my_data).encode()

    my_request = urllib.request.Request(my_url, data=json_data, headers=my_headers, method="PATCH")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure. You can notice the `contentType` has been updated to `"application/binary"`.

```json
{
    "name": "save_games/savegame.data",
    "bucket": "<YOUR-PROJECT-ID>.appspot.com",
    "generation": "1473948546121000",
    "metageneration": "2",
    "contentType": "application/binary",
    "timeCreated": "2016-09-15T14:09:06.053Z",
    "updated": "2016-09-16T02:46:44.439Z",
    "storageClass": "STANDARD",
    "size": "10450",
    "md5Hash": "7aIjAPS+Sd0DaF5SmGTUYw==",
    "contentEncoding": "identity",
    "crc32c": "DObTDw==",
    "etag": "CKj6iJzGkc8CEAE=",
    "downloadTokens": "7232aa46-f2e1-4df5-9698-d9c77b88ad5f"
}
```

## Update Metadata with Auth

To update metadata with authentication you need to provide an `auth_token` in the `Authorization` header.

You will also require to `JSON` encode which fields do you need to update and send them in a `PATCH` request.

```python
def update_privatemetadata(auth_token):

    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Authorization": "Bearer "+auth_token, "Content-Type": "application/json"}
    my_data = {"contentType": "application/binary"}
    
    json_data = json.dumps(my_data).encode()

    my_request = urllib.request.Request(my_url, data=json_data, headers=my_headers, method="PATCH")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

## Downloading a File

To download files from your Firebase Storage bucket you only require to send a `GET` request with the full path of the file and the parameter `alt=media`.
You will also require the followinv values from the `JSON` structure.

Name | Description
---|---
`name` | The path of the file including its name.
`bucket` | Your Firebase Project ID plus the `appspot.com` domain.
`downloadTokens` | A String used for downloading private files.

```json
{
    "name": "save_games/savegame.data",
    "bucket": "<YOUR-PROJECT-ID>.appspot.com",
    "generation": "1473948546121000",
    "metageneration": "1",
    "contentType": "text/plain",
    "timeCreated": "2016-09-15T14:09:06.053Z",
    "updated": "2016-09-15T14:09:06.053Z",
    "storageClass": "STANDARD",
    "size": "10450",
    "md5Hash": "7aIjAPS+Sd0DaF5SmGTUYw==",
    "contentEncoding": "identity",
    "crc32c": "DObTDw==",
    "etag": "CKj6iJzGkc8CEAE=",
    "downloadTokens": "7232aa46-f2e1-4df5-9698-d9c77b88ad5f"
}
```

To download a file and save it to disk we are going to use `urlretrieve` from the `urllib` module.

The following example downloads a `public` file:

```python
def download_file():
    
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data?alt=media"

    try:
        loader = urllib.request.urlretrieve(my_url, "newdata.data")
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader)
```

## Downloading a Private File

Downloading private files doesn't require the `Authorization` header. You only require to provide the `token` parameter and the file path.

The `token` parameter is the `downloadTokens` value from the `JSON` response when you upload a file.

```python
def download_privatefile(download_tokens):
    
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data?alt=media&token="+download_tokens

    try:
        loader = urllib.request.urlretrieve(my_url, "newdata.data")
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader)
```

## Download Metadata

You can download the information of any file in `JSON` format without downloading the file itself.

To download the metadata of a `public` file you only require to send a `GET` request with the full file path.

```python
def download_metadata():

    try:
        loader = urllib.request.urlopen("https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data")
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

## Dowload Private Metadata

To download metadata from private files you require to provide an `auth_token` in the `Authorization` header.

```python
def download_privatemetadata(auth_token):

    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2Fsavegame.data"
    my_headers = {"Authorization": "Bearer "+auth_token}
    my_request = urllib.request.Request(my_url, headers=my_headers)

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

## User Specific Files

So far we have worked with the same file (`savegame.data`) in the same location (`save_games` folder),
now we are going to step it up and make it so every registered user can have their own folder with their respective `savegame.data` file.

The following rules specify that only authenticated users can read and write the file `savegame.data` that will be located inside a folder named the same as their `uid` (`localId`):

```
service firebase.storage {
    match /b/<YOUR-PROJECT-ID>.appspot.com/o {
        match /savegames/{userId}/savegame.data {
            allow read, write: if request.auth.uid == userId;
        }
    }
}
```

We can use the following rules if we want users to have control over their complete folder:

```
service firebase.storage {
    match /b/<YOUR-PROJECT-ID>.appspot.com/o {
        match /savegames/{userId}/{allPaths=**} {
            allow read, write: if request.auth.uid == userId;
        }
    }
}
```

The following snippet requires that you already have a valid `auth_Token` and a `localId`.

The `localId` can be obtained after a successful `Sign In`, `Sign Up` or `Get Account Info` request.

The `auth` value can be obtained after a successful `Refresh Token` request.

For more information on these values you can read the [Firebase Auth guide](./../auth/).

```python
def upload_personalfile(auth_token, local_id):

    my_file = open("savegame.data", "rb")
    my_bytes = my_file.read()
    my_url = "https://firebasestorage.googleapis.com/v0/b/<YOUR-PROJECT-ID>.appspot.com/o/save_games%2F"+local_id+"%2F"+"savegame.data"
    my_headers = {"Authorization": "Bearer "+auth_token, "Content-Type": "text/plain"}

    my_request = urllib.request.Request(my_url, data=my_bytes, headers=my_headers, method="POST")

    try:
        loader = urllib.request.urlopen(my_request)
    except urllib.error.URLError as e:
        message = json.loads(e.read())
        print(message["error"]["message"])
    else:
        print(loader.read())
```

A successful response will look like the following JSON structure:

```json
{
    "name": "save_games/ktfSpKHar2fW1fcZePigI0Zr0bP2/savegame.data",
    "bucket": "<YOUR-PROJECT-ID>.appspot.com",
    "generation": "1473948546121000",
    "metageneration": "1",
    "contentType": "text/plain",
    "timeCreated": "2016-09-15T14:09:06.053Z",
    "updated": "2016-09-16T02:46:44.439Z",
    "storageClass": "STANDARD",
    "size": "10450",
    "md5Hash": "7aIjAPS+Sd0DaF5SmGTUYw==",
    "contentEncoding": "identity",
    "crc32c": "DObTDw==",
    "etag": "CKj6iJzGkc8CEAE=",
    "downloadTokens": "7232aa46-f2e1-4df5-9698-d9c77b88ad5f"
}
```

You will notice that the `localId` value has been added to the name.