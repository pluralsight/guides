## Guide to using the Tierion Hash API with Python.



Tierion makes it easy to create applications that record any data or business process in the blockchain. Tierion’s Hash API lets you anchor data to the blockchain while keeping your data private. Use of the Hash API is free up to 3 records per second or 1,000 records per hour.

https://tierion.com/docs/hashapi

In this guide we will be using Python 2.7

### Import Libaries

```
import requests
import hashlib
```


## Authentication

#### To obtain your access token, submit your Tierion credentials to the token endpoint.

https://hashapi.tierion.com/v1/auth/token

```
apiStem = 'https://hashapi.tierion.com/v1'

login_creds = {
    'username': 'youremail@here.com',
    'password': 'yourpassword',
}

reqToken = requests.post(apiPath + '/auth/token', json=login_creds)
reqToken.json()

token = reqToken.json()['access_token']
auth1 = {'Authorization': 'Bearer ' + token}
```

### Submit a string to Hash

https://hashapi.tierion.com/v1/hashitems

```
hashText = hashlib.sha256("Matthew Sedaghatfar Using the BlockChain").hexdigest()
hashItem = '/hashitems'
myHash = {'hash': hashText}

respHash = requests.post(apiStem + hashItem, json= myHash, headers = auth1)
receipt = respHash.json()['receiptId']

respHash.json()
```


#Get
getReceipts = '/receipts/'

getRec = requests.get(apiStem + getReceipts + receipt,headers = auth1)
getRe.json()

