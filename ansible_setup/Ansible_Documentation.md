# Ansible Lab Instructions
Please follow the listed steps below to clone this repository, and successfully run the ansible playbook that configures the newly provisioned HPVS instance.
- - - -

# Prerequisites
1. Install ansible via pip3 (or pip)
    * In order to run this playbook on the local machine _ansible_ is a required package.
    * Command for installation: _pip3 install ansible_
      * Please note that this command uses pip3, if you still have python2 as the default python interpreter, use 'pip' in lieu of pip3

# Initial setup prior to running playbook
1. Clone the repository on the local machine
    * git clone https://github.com/e-desouza/carekit-hyperprotect-lab.git
    * This github repository containing the ansible playbooks has been made public in order to clone this repository 
<br/>
   
2. After the Github repo has been cloned to the local machine, change the terminal directory to _carekit-hyperprotect-lab/ansible_setup_
    * cd Directory/whereRepository/wasCloned/carekit-hyperprotect-lab/ansible_setup
<br/>

3. There are two ansible playbooks that can be run for bringing up the CareKit Backend SDK application. The first uses a HyperProtect Virtual Server, and DBaaS MongoDB instance. While the second option will configure the Backend SDK app on the local machine, uses localhost. Choose the next set of instructions accordingly.


- - - 
# Instructions for the 'hpvs_setup.yml' playbook 
Requires HPVS and MongoDB IBM Cloud instances

1. Two crucial additions to the yml files must be made in order to successfully run the hpvs_setup.yml playbook. Add the following values accordingly:
    * ansible.cfg
       * Within the ansible.cfg file, the value for environmental variable _ansible_ssh_private_key_file_ is currently an empty string. Add the location and name of your public SSH key used to provision the Hyper Protect Virtual Server
       * Example: ansible_ssh_private_key_file = "~/.ssh/id_rsa.pub"
       ![Example-Of-CFG-File](screenshots/cfgFile.png)
       <br/>
       
    * inventory.yml
       * Underneath the commented line, add the public IP address of the Hyper Protect Virtual Server
       * Example:
       ![Example-Of-Inventory-File](screenshots/hostFile.png)
<br/>

2. One final preparation step is required prior to running the ansible playbook. The DBaaS MongoDB admin ID and password needs to be added to the mongo URI, as this value will be passed into the command line when invoking the playbook.
     * Ensure that the entire string including all 3 replica hosts are in the uri string, and also the Cluster name at the end.
     * Example:
     ```
     mongodb://admin:password@dbaas30.hyperp-dbaas.cloud.ibm.com:28162,dbaas29.hyperp-dbaas.cloud.ibm.com:28130,dbaas31.hyperp-dbaas.cloud.ibm.com:28222/admin?replicaSet=Wu-Tang_Clan
     ```
      * Replace 'admin' and 'password' with the proper admin ID and correlated password into the MongoDB URI string provided after provisioning the DBaaS instance.
<br/>

3. Now that the Public IP address has been added as a listed host, and the location of the public SSH key was specified, it is now time to run the ansible playbook for setup. Use the following command in order to run the playbook properly.
    * Notice that the initial portion of the extra variable being passed to the playbook starts with _db=_. This is required to specify which variable is being passed to the ansible playbook.
    * Command Example:
    ```
    ansible-playbook hpvs_setup.yml -e "db=mongodb://{admin_ID}:{Mongo_Password}@DBaaS_Mongo_URI:port.../admin?replicaSet=Cluster_Example"
     ```
     <br/>
     
 ![Example-Of-Ansible-Command](screenshots/Command.png)
 
 <br/>
 
4. Allow the playbook to run through it's designated tasks and configure the HPVS container.
![Example-Of-Playbook-Running-Tasks](screenshots/Playbook.png)
 
 <br/>
  
**The HPVS configuration should now be complete, follow the validation test section listed below to confirm the setup worked as intended.**

<br/>


- - -
# Instructions for the 'local_setup.yml' playbook 
Please note that while the local setup does _not_ require an IBM Cloud HPVS nor DBaaS instance, a few local 

1. Unlike the 'hpvs_setup.yml' playbook, the local setup already has the correct _locahost_ configurations written within the playbook itself, and does not require any other changes to the inventory file, nor ansible.cfg. Please use the listed command below to run the ansible script.
  * Command: _ansible-playbook local_setup.yml -K_
    * The _-K_ argument is required due to the playbook utilizing the "Become" parameter. Enter the local machine password (user profile credentials) when prompted.

![Run-Local-Setup](screenshots/local_setup.png)

<br/>

2. Allow the playbook to run and complete the predefined tasks. 

![Playbook-After-Completion](screenshots/finished_playbook.png)

<br/>

3. In order to ensure that the Backend SDK app was created properly, use the following docker commands to check.
 * _docker ps -a_
     * You should see two containers as a result, one labeled as 'hyperprotectbackendsdk', and the other container as 'mongo'.
   
 * Copy the Container ID serial number from the above output, and use the _docker logs <containerID>_ command to check the logs. 
     * If the app is running properly from the local machine, you will see output stating that the example app is running on port 3000.

![Example-Of-Docker-Commands](screenshots/docker_example.png)

<br/>





# Validation Test
To validate that the app is running properly, and listening on port 3000, a simple curl command can be issued to for verification. Please make certain that the IP address is changed in the http address after the POST declaration, as the goal is to hit the running application using the pulic IP addresss of the Virtual Server.
  
* Copy the entire curl command below, after replacing the _{HPVS_IP_or_locahost}_ parameter depending on which setup playbook was run.
  * If 'local_setup.yml' was used for configuration, change the parameter to _localhost_
  * If 'hpvs_setup.yml' was used instead, simply change the parameter to the public IP address of the HPVS instance
* Run the curl command from local machine, this will validate the Backend SDK application is reachable, and taking requests.
* If the test is successful, a returned output of 'RevisionRecord stored' will populate after the curl command. 
  * Also, within the docker container logs, the POST call will come through, and a _201_ code will be returned. 
  * Command to view docker logs: _docker logs <containerID>_
    * ContainerID can be found by running _docker ps -a_ and locating the HyperProtectBackendSDK container

**Curl Command**
```
curl --cacert rootCA.crt --location --request POST 'https://{HPVS_IP_or_locahost}:3000/revisionRecord' \
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
```
>     "knowledgeVector": {
>         "processes": [
>             { "id" : "1C43F648-D41A-4A5A-8708-15737425FA7C", "clock" : 10},
>             { "id" : "2B43F648-D41A-4A5A-8708-15737425FA7C", "clock" : 4}
>         ]
>     }
> }'
RevisionRecord stored
```

<br/>

Another verification check is to check the Docker container logs on the Virtual Server. The docker logs can be checked as such:

1. Check docker for running containers
> docker ps -a
  
* Locate the Docker container ID
  * Example: 
  ```
  root@b4e8f18c497b:~# docker ps -a
  CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS
  c876afbe4704        hyperprotectbackendsdktest_app   "docker-entrypoint.s…"   2 hours ago         Up 2 hours          
  ```
<br/>

2. Use container ID from previous step to check current log entries
> docker logs {Container_ID}
  
* View the Docker logs, traces of the curl command should be present as depicted in the screenshot below. 
  
 ![Docker-Logs-Example](screenshots/Results.png)
