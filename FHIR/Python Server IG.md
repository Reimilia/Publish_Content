#Overview

   An implementation guide to a Smart-on-fhir Genomic Sandbox, which includes a python server and the rule on how to create an app that can connect to this back-end server. If you only concern about the apps , you can jump to [How to use this sandbox server with your own web apps](#Chapter1.3) part. Here are some possible public server for you to test:

> http://ec2-52-43-253-18.us-west-2.compute.amazonaws.com:2048/ (STU3)
> 
> http://genomics-advisor.smartplatforms.org/ (DSTU2)

Source code of the server will be at:
https://github.com/LXander/FHIR-Genomics_v2/tree/wy

If you have any question about this guide, contact at:
<span id="Email"></span>

> panzer.wy@gmail.com
> 
> gil.alterovitz@gmail.com



 <span id = "Content"></span> 

##Contents

*	[Tutorial of Usage](#Chapter1)
	1. [Preliminary Environment Configuration](#Chapter1.1)
	2. [How to set up and run a server](#Chapter1.2)
	3. [How to use this sandbox server with your own web apps](#Chapter1.3)

		*    [How to communicate with server](#Chapter1.3.1)
		*    [How to create your own app](#Chapter1.3.2)
		*    [How to create and submit own data instances](#Chapter1.3.3)
		*    [RESTful API and data recevied from server](#Chapter1.3.4)
		
*	[Topics for developers](#Chapter2)
	1. Web developing with Flask
	2. How does this server match FHIR datatype
	3. How does this server check the correctness of the data
	4. How does this server organize RESTful API

*	[Bugs and Issues](#Chapter3)
	1. Potential bugs
	2. Developer's Progress Bar


----------
 <span id = "Chapter1"><h3>[Tutorial of Usage](#Content)</h3></span>
 <span id = "Chapter1.1"><h4>[Preliminary Environment Configuration](#Content)</h4></span> 

This server cannot run on Windows environment due to the pysam package. The following steps(including later part) are all tested on linux **Ubuntu** only.

First, you need a **python 2.7** environment, we recommended use the package **virtualenv**. Then, use pip to install the package dependency with the command in terminal:
```
pip install -r requirements.txt
```
This will install required package automatically. If there's some problem in installation, mostly the problem is lack of system package. To fix this issue ,run this command in teminal:
```bash
sudo apt-get install python-dev libpq-dev libxml2-dev libxslt1-dev
```

Then, it is crucial to setup a local **database** to store the data. Now the server can support two method:

* postgresql: need install **postgresql** manually and setup the database with the help of setup_db.py. Also see config.py to configure your settings.

* sqlite:  make everything default, then ok to go. But this is not recommended when the number of data is too large

Next, the database is set but it is empty now. To simply generate some test data, you can run load_example.py. This will create a super user and add some data to this user. See the python file itself to know more about that.

Then you are about to go. Below we will show you how to set up and deploy your server.

 <span id = "Chapter1.2"><h4>[How to set up and run a server](#Content)</h4></span> 

Now that you have the initial data, you are ready to run the server! Simply run the file server.py to enjoy the sandbox now: here are some ways to run it:

*	python server.py --debug ： <div>This will run the server with **debug** mode in package flask. At this mode, every single error will be traceback and reveal in either a website or in command lines.</div>

*	python server.py run :  <div> In this **deploy** mode, the server will run back-end, and it will keep running unless a kill command, shutdown of system or a critical error to crash the process down</div>

*	python server.py clear: <div> Clean the data base of this server, for whatever the reason is.</div>

Note that you need to check config.py, the variable HOST will tell the program where(i.e. the website address) the server will run. If you want to deploy this server to public, make sure you set the right HOST location and it has a public ip so everyone can have access to it (localhost:xxxx is only for local visit)

If everything works fine, you will see the page like that:


Congratulations. The server is running!

 <span id = "Chapter1.3"><h4>[How to use this sandbox server with your own web apps](#Content)</h4></span> 
Now you can try do something to this server. But wait, there are some things you need to know to take better advantage of this server. If there's no web apps, this server is boring :-(

Here is  a simple but  a live demo that will teach you how to code a simple apps that can commute data with the sandbox server. The program will be written with **python 2.7.**

 <span id = "Chapter1.3.1"><h5>[How to communicate with server](#Content)</h5></span> 

Basically, the sandbox is providing a technique that separate one's data with others called Oauth2.0. This technique can grant for your permission to operate the data in sandbox server with an existing registered app.
![Oauth2.0](https://drive.google.com/uc?id=0B1Y9kFL5yCVDNmFGRnJJMVhCOE0)

Note the sandbox combine Authorization Server and Resource Server altogether. So, if you want to connect to this server.  You need first **register an account**. Simply to the base location of the server (default: http://localhost:2048)
 ![Signup](https://drive.google.com/uc?id=0B1Y9kFL5yCVDUExLVm12bFdFWW8)
 
 Click "here" to sign up
![Signup2](https://drive.google.com/uc?id=0B1Y9kFL5yCVDY2JQdmZNbVF1MjA)

Simply use an email address to register and set up your password. And then you can see the main page.
![Hello](https://drive.google.com/uc?id=0B1Y9kFL5yCVDWmR0dk1zMUNxQUk)

1.	This is where you click to register an app
2.	This is navigate bar.

Click on "New App" and fill up the basic info you need to complete registration:
![NewApp](https://drive.google.com/uc?id=0B1Y9kFL5yCVDcmpkZVVsQVRoazA)

*	first line is you **app name**
*	second line is **redirect URL**, i.e. after authorization, the server will redirect to which address. (should be part in your app)
*	third line is  **launch URL**, i.e. where the app starts. 
*	the select box of **"Public" or "Confidential"** are two authorization strategies. Default is Public, though Confidential is safer.

When you fill up the form , simply click on 'Register'.  And then you will see your Client ID, which is quite like a id-card to ensure your app can connect to this server.
![ClientID](https://drive.google.com/uc?id=0B1Y9kFL5yCVDVXNSVHZkVXo3REU)

Remember this key, next we need use this key to set up connection

<span id = "Chapter1.3.2"><h5>[How to create your own app](#Content)</h5></span> 
**Caution: Read this part carefully!**

Since Oauth2.0 authorization, most RESTful API point to the server will be rejected due to safety concerns. However, with the **ClientID** get from a public App registration, you can possibly visit the data in this server.

If you are using python, simply create a file called , say 'config.py', and type these codes in the file:
```python
API_BASE = 'http://localhost:2048/api'
AUTH_BASE = 'http://localhost:2048/auth'
CLIENT_ID = '403c2a26-392d-4aea-b737-05fa64e4440d'
REDIRECT_URI = 'http://localhost:8000/recv_redirect'
SCOPES = ['user/Sequence.read','user/Sequence.write',
        'user/observationforgenetics.read',
        'user/reportforgenetics.read',
        'user/Patient.read',
        'user/Condition.read'
]
```

*	API_BASE :  This is where the RESTful API starts with  , like a base server parameter.
*	AUTH_BASE: This is where the Oauth2.0 Authorization starts with
*	CLIENT_ID: key to gain access to server
*	REDIRECT_URI: This is what you type when registered your app
*	SCOPES: which type of FHIR resource you want to visit, and for what purpose you want to do with resource. 
 
**Caution！：** When you want to visit public server, please change the API_BASE and AUTH_BASE accordingly! 
(E.g. API_BASE = 'http://ec2-52-43-253-18.us-west-2.compute.amazonaws.com:2048/api')

Next, we are moving on to the authorization connection. Here is the workflow of how app get permit on server:

*	Every time an API request is pointing to a server, the server will check **cookies** with this API request to see if you are permitted to get access.
*	If not , reject the request.

So we need to get **cookies** first, here is the workflow to get cookies:

*	every time the app sends an API request, if not have **cookies**, redirect to the authorization page on server (as a third party grant page), with the following parameters:
	*	scope:  SCOPES
	*	client_id: CLIENT_ID
	*	redirect_uri: REDIRECT_URI
	*	response_type: 'code'
*	Based on registration information, (if not logged in ,redirect and **login first**), you will be redirect to REDIRECT_URI you typed if your CLIENT_ID is approved. 
*	Then at REDIRECT_URI page, the app needs to get access_token, which is used as **cookies**. The cookie will remain for 30 minutes due to server's configuration.

This might be little tricky. Here is the sample code :
```python
#auth. py
from flask import redirect, request
from urllib import urlencode
from functools import wraps
import requests
from config import AUTH_BASE,CLIENT_ID, REDIRECT_URI, API_BASE, SCOPES


class OAuthError(Exception):
    pass


def api_call_for_test(api_endpoint):
    '''
    helper function that makes API call
    '''
    access_token = request.cookies['access_token']
    auth_header = {'Authorization': 'Bearer %s'% access_token}
    return requests.get('%s%s'% (API_BASE, api_endpoint), headers=auth_header)


def get_access_token(auth_code):
    '''
    exchange `code` with `access token`
    '''
    exchange_data = {
        'code': auth_code,
        'redirect_uri': REDIRECT_URI,
        'client_id': CLIENT_ID,
        'grant_type': 'authorization_code'
    }
    resp = requests.post(AUTH_BASE+'/token', data=exchange_data)
    if resp.status_code != 200:
        raise OAuthError
    else:
        return resp.json()['access_token']


def has_access():
    '''
    check if application has access to API
    '''
    if 'access_token' not in request.cookies:
        return False
    # we are being lazy here and don't keep a status of our access token,
    # so we just make a simple API call to see if it expires yet
    access_resource = SCOPES[0].split('/')[-1].split('.')[0]
    test_call = api_call_for_test('/%s?_count=1' % access_resource)
    return test_call.status_code != 403


def require_oauth(view):
    @wraps(view)
    def authorized_view(*args, **kwargs):
        # check is we have access to the api, if not, we redirect to the API's auth page
        if has_access():
            return view(*args, **kwargs)
        else:
            redirect_args = {
                'scope': ' '.join(SCOPES),
                'client_id': CLIENT_ID,
                'redirect_uri': REDIRECT_URI,
                'response_type': 'code'}
            return redirect('%s/authorize?%s'% (AUTH_BASE, urlencode(redirect_args)))

    return authorized_view
```

And below is the real app example that use function auth.py , you can see the complete tiny app at [github](https://github.com/Reimilia/Genetic-Report-Viewer).

```python
# part of app.py
from flask import Flask, render_template
from auth import *

# we use this to shorten a long resource reference when displaying it
MAX_LINK_LEN = 20
# we only care about genomic stuff here


app = Flask(__name__)


def api_call(api_endpoint):
    '''
    helper function that makes API call 
    '''
    access_token = request.cookies['access_token']
    auth_header = {'Accept': 'application/json', 'Authorization': 'Bearer %s' % access_token}
    resp = requests.get('%s%s' % (API_BASE, api_endpoint), headers=auth_header)
    return resp.json()


def to_internal_id(id):
    '''
    markup an internal resource id with anchor tag.
    '''
    return '<a href="/reports/%s">%s...</a>' % (id, 'Genetics Report')


def render_fhir(resource):
    '''
    render a "nice" view of a FHIR bundle
    '''
    for entry in resource['entry']:
        entry['resource']['id'] = to_internal_id(entry['resource'].get('id', ''))
    return render_template('bundle_view.html', **resource)


@app.route('/')
@require_oauth
def index():
    return redirect('/resources/Patient')


# if the user authorize the app, use code to exchange access_token to the server
@app.route('/recv_redirect')
def recv_code():
    code = request.args['code']
    access_token = get_access_token(code)
    resp = redirect('/')
    resp.set_cookie('access_token', access_token)
    return resp


@app.route('/resources/<path:forwarded_url>')
@require_oauth
def forward_api(forwarded_url):
    forward_args = request.args.to_dict(flat=False)

    forward_args['_format'] = 'json'
    api_url = '/%s?%s'% (forwarded_url, urlencode(forward_args, doseq=True))
    bundle = api_call(api_url)
    # not bundle but plain resource
    print bundle 
    if bundle.get('type') != 'searchset':
        resource = bundle
        bundle = {
            'resourceType': resource['resourceType'],
            'entry': [{
                'resource': resource,
                'id': forwarded_url
            }],
            'is_single_resource': True,
        }
    elif len(bundle.get('entry', [])) > 0:
        bundle['resourceType'] = bundle['entry'][0]['resource']['resourceType']

    return render_fhir(bundle)

if __name__ == '__main__':
    app.run(debug=True, port=8000)

```
The running result looks like these:
![result1](https://drive.google.com/uc?id=0B1Y9kFL5yCVDSTRIdjA3dmg2MlE)
![result2](https://drive.google.com/uc?id=0B1Y9kFL5yCVDREViY1Nic253TzA)

We suppose you know the format of file the server will return. If you don't know,  you can check it [here](#Chapter1.3.4-Data). You can also get the idea of how to use RESTful API in this tiny app. But if you want a formal document , see [this](http://hl7.org/fhir/2016Sep/http.html) for a STU3 API format.


<span id = "Chapter1.3.3"><h5>[How to create and submit own data instances](#Content)</h5></span> 

There are two ways to submit a data by yourself.

*	Submit by account. Just Click on 'Submit data' icon when you logged in.
*	Submit by POST and GET method. See [here](#Chapter1.3.4)
 
Remember you need to check the format of FHIR resource. The server does not allow illegal resource format to be posted. Also , for the present time, **submit by account is JSON data only**.

 <span id = "Chapter1.3.4"><h5>[RESTful API and data recevied from server](#Content)</h5></span>
  
####API Reference:

The SMART Genomics API is built on top of SMART on FHIR please see [here](http://hl7.org/fhir/2016Sep/http.html) for more information.

Note: The SMART Genomics API supports both XML and JSON formats. Append ?_format= xml|json in HTTP requests to differentiate between the two.

The following operations are defined:

> **read**: Read the current state of the resource
> 
> **search**: Search the resource type based on some filter criteria
> 
> **update**: Update an existing resource by its id (or create it if it is new)
> 
> **create**: Create a new resource with a server assigned id
> 
> **delete**: Delete a resource
> 
> **history**: Retrieve the update history for a particular resource
> 

The Service Root URL is the address where all of the resources defined by this interface are found. The Service Root URL takes the form of:

> http:// [server base] /api/resourceType

####Style:

```
Read
GET [base]/[type]/[id]
For example:
http://localhost:2048/api/Sequence/[id]

Search
GET [base]/[type]{?[parameters]}
For example:
http://localhost:2048/api/Sequence?variationID=[variationID]

Create
POST [base]/[type]{?_format=json|xml}
With data submitted

Update
PUT [base]/[type]{?[parameters]&_format=json|xml}
With data Submitted

History
GET [base]/[type]/[id]/_history{?[parameter]}
http://localhost:2048/api/Sequence/[id]/_history?variationID=[variationID]
```
####Sample codes:

All written in python 2.7
API_BASE = 'http://localhost:2048/api'

####to read data
```python
def read(request, url, id):
  access_token = request.COOKIES['genomic_access_token']
  resp = requests.get('%s/%s/%s?_format=json'%(API_BASE, url, id),
			  headers={'Accept': 'application/json','Authorization': 'Bearer %s'% access_token})
  return resp.json()
```
####to create data
```python
def create(request, url, data):
  access_token = request.COOKIES['genomic_access_token']
  resp = requests.post('%s/orderforgenetics?_format=json'% API_BASE,
  						data=json.dumps(data),headers={'Authorization': 'Bearer %s'% access_token})
  return resp.json()
```

####to update data
```python
def read(request, url, id, data):
  access_token = request.COOKIES['genomic_access_token']
  resp = requests.put('%s/%s/%s?_format=json'%(API_BASE, url, id),
  					 data=json.dumps(data),
			  headers={'Accept': 'application/json','Authorization': 'Bearer %s'% access_token})
  return resp.json()
```

####to search data
```python
def search(url, args={}):
  access_token = request.COOKIES['genomic_access_token']
  args['_format'] = 'json'
  resp = requests.get('%s%s?%s'% (API_BASE, url, urlencode(args)),
						headers={'Accept': 'application/json','Authorization': 'Bearer %s'% access_token})
  return resp.json()
```

<span id="Chapter1.3.4-Data"></span>
####Data from Server

Server will return either one specific data (if you point to a specific resource with [base]/[type]/[id] format or the bundled resource (if you are searching or want a list of specific resource type)

####Single instance
It will basically follow the data as FHIR structure, here is one example:
```json
{
    "patient": {
        "reference": "Patient/1712718f-3c5b-46e3-9ae1-6e8670276b3d"
    }, 
    "resourceType": "Condition", 
    "verificationStatus": "confirmed", 
    "code": {
        "text": "Breast cancer", 
        "coding": [
            {
                "code": "254837009", 
                "display": "Malignant tumor of breast", 
                "system": "http://snomed.org/sct"
            }
        ]
    }, 
    "meta": {
        "versionID": 1, 
        "lastUpdated": "2016-08-30T11:08:55.828815"
    }, 
    "id": "4f1ddc9e-fdde-44f3-a5fd-26499c206164", 
    "subject": "Patient"
}
```

####bundle result
It will follow the format like the following example:
```json
{
    "updated": "2016-09-09T16:59:41.181048",
    "resourceType": "Bundle",
    "link": [
        {
            "href": "http://localhost:2048/api/Patient?_format=json",
            "rel": "self"
        }
    ],
    "entry": [
	{"result1": "some resource"},
	{"result2": "some other resource"}
    ],
    "total": 21,
    "type": "searchset",
    "id": "http://localhost:2048/api/Patient?_format=json"
}
```

The format of each resource will follow the specification online. (E.g. [FHIR 1.6.0](http://hl7.org/fhir/2016Sep/sequence.html)) You can check at anytime. Note there exists subtle difference between the resource format from DSTU2 and STU3, and indeed it will bring some difficulties for developer. Indeed the little demo app only applies STU3(FHIR 1.6.0) version. For more infomation about DSTU2 version. Contact at the email [here](#Email). 

----------
 <span id = "Chapter2"><h3>[Topics for developers](#Content)</h3></span> 

To be continued

----------
 <span id = "Chapter3"><h3>[Bugs and Issues](#Content)</h3></span> 

To be continued