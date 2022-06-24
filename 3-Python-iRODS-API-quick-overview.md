**Authors**
- Arthur Newton 
- Christine Staiger 
- Claudia Behnke (SURF)
- Claudio Cacciari (SURF)
- Maithili Kalamkar Stam (SURF)

**License**
Copyright 2022 SURF BV

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Goal
You will learn how to interact with iRODS via the python API. In this module we will explore the API in interactive mode. You will:

- Explore the python API to iRODS in ipython
- Upload and download data and collections
- List metadata

## 1. Explore the python API to iRODS in ipython

### 1.1 Set up

If you have followed the earlier modules in this training, you should have performed the following steps already 

- Logged in the Lisa compute cluster
- Cloned the github repository on the Lisa compute cluster (https://github.com/ccacciari/iRODS-RDM-HPC-course.git)

On the login node of the Lisa compute cluster, you have access to *ipython* interpreter, the python package *irods* and you will install [python-irodsclient](https://github.com/irods/python-irodsclient). The login node behaves like any compute node in a cluster, i.e. when your code works here, it will also work on the compute nodes. Later on you will create a script that calls the workflow and runs it on several remote worker nodes without any direct interaction of us.


Install the *irods-pythonclient*:

```
module load 2020
module load iRODS-iCommands/4.3.0
pip3 install python-irodsclient
```

Start an ipython session (first time doing this could take some time):

```sh
ipython3
```

## 2. Upload and download data object and collections

### 2.1 Connect to iRODS
You authenticate yourself with an iRODS username and password (this is different from your Lisa credentials). The module *getpass* asks for passwords without printing the input on screen. With en encoding function we prevent that the variable contains the plain password when we pass it to the iRODS server.

```py
import getpass
pw = getpass.getpass().encode()
```
Now we can create an iRODS session:
```
from irods.session import iRODSSession
session = iRODSSession(host='rsc-test1.irods.surfsara.nl', port=1247, user='irods-user1', password=pw.decode(), zone='surfZone1')
```

Note that you will need to change the username to the one you are given. Throughout this course we will assume you are `irods-user1`. 

You can test whether we have done everything correctly and have access:

```py
coll = session.collections.get('/surfZone1/home/irods-user1')
print(coll.path)
print(coll.data_objects)
print(coll.subcollections)
```

We will need our home collection more often as reference point, so let us store the collection path:

```py
iHome = coll.path
```
 
### 2.2 Upload a data object
The preferred way to upload data to iRODS is a data object *put*. Now we create the logical path and Alice in wonderland file (alice.txt)to iRODS (similar to previous exercise:

```py
iPath = iHome+'/alice.txt'
session.data_objects.put('alice.txt', iPath)
```

Run the command again:

```py
session.data_objects.put('alice.txt', iPath)
```
Now try editing the alice.txt file and run the command again. You can login from another terminal and edit the file e.g., with a vi editor


```py
session.data_objects.put('alice.txt', iPath)
```
Do you know why you do not get an error now?

> **_Food for brain:_**
>
> * Does iRODS allow overwrites? Why do you not get any error? (tip: https://github.com/irods/python-irodsclient/issues/322)
> * Try to put data to an undefined path or your neighbours home collection (spelling mistake ...)


The object carries some vital system information, otherwise it is empty. 

```
obj = session.data_objects.get(iPath)
print("Name: ", obj.name)
print("Owner: ", obj.owner_name)
print("Size: ", obj.size)
print("Checksum:", obj.checksum)
print("Create: ", obj.create_time)
print("Modify: ", obj.modify_time)
print("Metadata: ", obj.metadata.items())
```

Less code to write to display the full object:
```
vars(obj)
```

**Remark**: iRODS stores times in UNIX [epoch](https://en.wikipedia.org/wiki/Unix_time) time, but the python client always returns times in UTC (2 hours behind our local time).


### 2.4 Download a data object
We can download a data object as follows (note that we use the environment variable 'HOME' that is defined to be your Lisa homefolder):

```py
import os
localpath = os.environ['HOME']+'/'+os.path.basename(obj.path)
obj = session.data_objects.get(obj.path,localpath)
os.path.exists(localpath)
```

Now try running the command again.


```py
obj = session.data_objects.get(obj.path,localpath)
```
Do you know why do you get an error now?

**Exercise** Calculate the MD5 checksum for the downloaded data and compare with the data object's checksum in iRODS. (hint: `import hashlib; hashlib.md5(open(<filename>, 'rb').read()).hexdigest()`
