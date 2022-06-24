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
You will learn how to interact with iRODS via WebDAV. You will:

- Connect to the iRODS WebDAV interface via Web browser
- Connect to the iRODS WebDAV interface through a WebDAV client
- Upload and download data and collections

## 1. Connect to the iRODS WebDAV interface via Web browser

### 1.1 Set up

Just copy the url of the test iRODS environment in your web browser: https://rsc-test1.irods.surfsara.nl/  
You should see a pop-up window, which asks your iRODS username and password. Enter them.  
You should now see the content of your iRODS home folder.

## 2. Connect to the iRODS WebDAV interface through a WebDAV client

### 2.1 Set up

Perform the following steps: 

- Log in the Lisa compute cluster
- Clone the github repository on the Lisa compute cluster (https://github.com/ccacciari/iRODS-RDM-HPC-course.git)

We will use a client called rclone (https://rclone.org). It is already available on Lisa.  
This client supports many different protocols and interfaces, included WebDAV.

```
cd iRODS-RDM-HPC-course
rclone ls :webdav: --webdav-url=https://rsc-test1.irods.surfsara.nl --webdav-vendor=other --webdav-user=irods-user1 --webdav-pass=xxxxxxx
```

Does it work? The rclone client requires an "obscurated" password:

```
rclone obscure mypassword
rclone ls :webdav: --webdav-url=https://rsc-test1.irods.surfsara.nl --webdav-vendor=other --webdav-user=irods-user1 --webdav-pass=myobscurepassword
```

## 3. Upload and download data object and collections

### 3.1 

 Now if you want to upload a file:

```
rclone move alice.txt :webdav: --webdav-url=https://rsc-test1.irods.surfsara.nl --webdav-vendor=other --webdav-user=irods-user1 --webdav-pass=myobscurepassword
rclone ls :webdav: --webdav-url=https://rsc-test1.irods.surfsara.nl --webdav-vendor=other --webdav-user=irods-user1 --webdav-pass=myobscurepassword
```

If you try to reach the webdav interface with your web browser now, you should see the same file.  
You can downaload it with your browser on your pc.  
Or download it to Lisa via rclone:

```
rclone move :webdav:alice.txt alice.txt --webdav-url=https://rsc-test1.irods.surfsara.nl --webdav-vendor=other --webdav-user=irods-user1 --webdav-pass=myobscurepassword
ls -l
```

You can also try the command ```copy``` and ```delete```.