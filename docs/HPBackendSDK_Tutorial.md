- - - -
# Configuring the Apple CareKit application with a Hyper Protect Virtual Server and a Hyper Protect DBaaS MongoDB Backend.
Follow the listed steps below in order to successfully build the Apple CareKit application on an HPVS instance, while leveraging the Hyper Protect DBaaS for mongodb service as a viable backend database. 
- - - -

<br/>

# Prerequisites
	
* Hyper Protect Virtual Server instance
	* Follow steps here for assistance provisioning a Virtual Server https://cloud.ibm.com/docs/services/hp-virtual-servers?topic=hp-virtual-servers-connect_vs
	* Also: https://github.com/e-desouza/carekit-hyperprotect-lab/blob/master/docs/3_deploy_hyper_protect_services.md

* Hyper Protect DBaaS for MongoDB instance
	* Follow steps located https://github.com/e-desouza/carekit-hyperprotect-lab/blob/master/docs/3_deploy_hyper_protect_services.md for assistance provisioning a Hyper Protect MongoDB
	* Additional Information: https://cloud.ibm.com/docs/services/hyper-protect-dbaas-for-mongodb?topic=hyper-protect-dbaas-for-mongodb-gettingstarted

* Ubuntu package 'git' must be installed on the Virtual Server for this lab. Please access the Virtual Server via ssh protocol, and run the following command: _sudo apt-get install -y git_
	* After the package has finished installing, verify the installation by executing command _git --version_

* Install 'npm' on Virtual Server using  _sudo apt-get -y install npm_
	* The _npm_ package manager is required for the NodeJS platform
	* To verify the installation of _npm_ type in command _npm -v_

* Install 'docker' on the Virtual Server using command _sudo apt-get -y install docker.io
	
	* verify that docker was installed correctly using _docker --version_
		* If required, type in _systemctl |grep docker_ and the output should tell you if the Docker service was installed. If the service is present yet disabled, start the service using  _sudo systemctl start docker_
		
* In order to properly run the application using port 3000 for 'express', _ufw_ will need to be installed on the HPVS instance
	* Command to run on Virtual Server: _sudo apt-get -y install ufw_
	* Enable traffic to port 3000 in order to leverage POST calls to upload information to MongoDB and the backend application
		* ufw allow 3000/tcp
		* Verify port translation with command _ufw status verbose_ 

```
root@b4e8f18c497b:~/HyperProtectBackendSDK-test# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
3000/tcp                   ALLOW IN    Anywhere                  
3000/tcp (v6)              ALLOW IN    Anywhere (v6) 
```
	


<br/>

# Configuration Steps

1. Access the Hyper Protect Virtual Server via ssh
	* Instructions on how to access the VS can be found in the Prerequisite section above.
	* Make certain that all required prerequisites have been installed on the Virtual Server after success accessing the instance. 
		* Example: _ufw_ installation, _git_ installation, etc. Please see above section for a full list and instructions.
![Access-HPVS-Container](screenshots/logIntoHPVS.png)

<br/>


2. Clone the 'carekit-apple' Github repository on the provisioned HP Virtual Server using the following command: _git clone https://github.com/carekit-apple/CareKit.git_
	* See the afforementioned prerequisite section for assistance installing _git_
	* Note: This repository contains the CareKit's frontend (UI)

<br/>

3. One additional repo is required for this configuration, run this command: _git clone https://github.com/carekit-apple/HyperProtectBackendSDK.git_
	* This particular repository contains the application backend written in typescript, utilizing TypeORM.

<br/>

4. Within the Virtual Server, change the current directory to the recently cloned backend repository.
	* Changing directories can be achieved by using _cd HyperProtectBackendSDK-master_

<br/>

5. An environmental variable file (.env) has been created in the root directory. It is imperative that the necessary MongoDB cluster URI is added as the environmental variable's value, see below for an example.
	* Open up the '.env' file, and set the following values based on the information that the MongoDB DBaaS instance was created with, in the previous tutorial. 
		* MONGO_DB={mongodb://UserID:Password@dbaasXX.hyperp-dbaas.cloud.ibm.com:XXXX...}
			* Ensure to add the admin UserID and Password in the proper place and format within the DB string
			* Fill in full cluster URI, which should consist of 3 total DBaaS replica endpoints

**Initial .env file contents**:
![Contents-Of-ENV-File](screenshots/envContents.png)

<br/>

**Example of .env file with required information added**
![Example-Of-Modifying-ENV-File](screenshots/envVarExample.png)

<br/>


6. Run the 'npm' installation by using command _npm install_ from the root directory of the Github repo {enter name here once finalized}
	* The _npm install_ process will install all of the required packages and dependencies needed to run the CareKit application
![npm-Install](screenshots/npmInstall.png)

<br/>

7. Finally, start the application backend by executing the _npm start_ command. This particular command will initialize the application, and bring the application online leveraging port 3000. 
	* If the _npm start_ execution was successful, the message stating that 'Server started on port 3000' should propagate.
	* Ensure that the ts-node version is **greater** than 3.3.0, as several compiler errors will occur during the next step if an older version is used. Notice in the screenshot linked below that 'Using ts-node version 8.9.1' is delcared. 
![npm-Install](screenshots/npmStart.png)

<br/>

# Validation Test
To validate that the app is running properly, and listening on port 3000, a simple curl command can be issued to for verification. Please make certain that the IP address is changed in the http address after the POST declaration, as the goal is to hit the running application using the pulic IP addresss of the Virtual Server.
	
* Copy the entire curl command below, after replacing the HPVS Public IP address
* Run command from local machine, this will verify that the application is running on the Virtual Server, and is accessible. 
* If the test is successful, a returned output of 'RevisionRecord stored' will populate after the curl command. 
	* Also, on the Virtual Server running the application, the POST call will come through, and a _201_ code will be returned. 

**Curl Command**
```
curl --location --request POST 'http://{HPVS_Public_IP_Address}:3000/revisionRecord' \
--header 'Content-Type: application/json' \
--data-raw '{
    "entities": [
        {
            "type": "task",
            "object": {
                "schemaVersion": {
                    "majorVersion": 2,
                    "minorVersion": 0,
                    "patchNumber": 4
                },
                "id": "nausea",
                "uuid": "75EE244A-7303-43CF-9AA5-6CC3BB81210A",
                "createdDate": 609212115.685683,
                "updatedDate": 609212115.685702,
                "title": "Track your nausea",
                "notes": [],
                "timezone": {
                    "identifier": "America/Sao_Paulo"
                },
                "instructions": "Tap the button below anytime you experience nausea.",
                "impactsAdherence": false,
                "effectiveDate": 608785200,
                "schedule": {
                    "elements": [
                        {
                            "text": "Anytime throughout the day",
                            "duration": {
                                "isAllDay": true
                            },
                            "interval": {
                                "minute": 0,
                                "hour": 0,
                                "second": 0,
                                "day": 1,
                                "month": 0,
                                "year": 0,
                                "weekOfYear": 0
                            },
                            "targetValues": [],
                            "start": 608785200
                        }
                    ]
                }
            }
        },
        {
            "type": "task",
            "object": {
                "schemaVersion": {
                    "majorVersion": 2,
                    "minorVersion": 0,
                    "patchNumber": 4
                },
                "id": "doxylamine",
                "uuid": "C0861A29-C726-4B58-B3AB-89CF3E3294F6",
                "createdDate": 609212115.696223,
                "updatedDate": 609212115.696224,
                "title": "Take Doxylamine",
                "notes": [],
                "timezone": {
                    "identifier": "America/Sao_Paulo"
                },
                "instructions": "Take 25mg of doxylamine when you experience nausea.",
                "impactsAdherence": true,
                "effectiveDate": 608814000,
                "schedule": {
                    "elements": [
                        {
                            "duration": {
                                "seconds": 0,
                                "isAllDay": false
                            },
                            "interval": {
                                "minute": 0,
                                "hour": 0,
                                "second": 0,
                                "day": 1,
                                "month": 0,
                                "year": 0,
                                "weekOfYear": 0
                            },
                            "targetValues": [],
                            "start": 608814000
                        },
                        {
                            "duration": {
                                "seconds": 0,
                                "isAllDay": false
                            },
                            "interval": {
                                "minute": 0,
                                "hour": 0,
                                "second": 0,
                                "day": 2,
                                "month": 0,
                                "year": 0,
                                "weekOfYear": 0
                            },
                            "targetValues": [],
                            "start": 608835600
                        }
                    ]
                }
            }
        },
        {
            "type": "task",
            "object": {
                "schemaVersion": {
                    "majorVersion": 2,
                    "minorVersion": 0,
                    "patchNumber": 4
                },
                "id": "kegels",
                "uuid": "1B6AA55A-E5A1-4124-8B9E-59DE3EEF9DE5",
                "createdDate": 609212115.697711,
                "updatedDate": 609212115.697713,
                "title": "Kegel Exercises",
                "notes": [],
                "timezone": {
                    "identifier": "America/Sao_Paulo"
                },
                "instructions": "Perform kegel exercies",
                "impactsAdherence": true,
                "effectiveDate": 608814000,
                "schedule": {
                    "elements": [
                        {
                            "duration": {
                                "seconds": 0,
                                "isAllDay": false
                            },
                            "interval": {
                                "minute": 0,
                                "hour": 0,
                                "second": 0,
                                "day": 2,
                                "month": 0,
                                "year": 0,
                                "weekOfYear": 0
                            },
                            "targetValues": [],
                            "start": 608814000
                        }
                    ]
                }
            }
        }
    ],
    "knowledgeVector": {
        "processes": [
            { "id" : "1C43F648-D41A-4A5A-8708-15737425FA7C", "clock" : 10},
            { "id" : "2B43F648-D41A-4A5A-8708-15737425FA7C", "clock" : 4}
        ]
    }
}'
```

After the curl command has been issued, if successful the response will look similar to this screenshot:
![Confirmation](screenshots/Confirmation.png)
