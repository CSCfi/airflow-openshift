# airflow-openshift
Run Apache Airflow on OpenShift

The purpose of this template is to run [Apache Airflow](https://airflow.apache.org) on Openshift. Acknowledgement: The template is a modified version of https://github.com/adyachok/incubator-airflow.git

##### Docker Image Used
https://github.com/CSCfi/docker-airflow

#### Useful Variables

The template can be imported via the Openshift web ui or added via the command line interface (oc)

The required variables which need to be present in order to run the template are:

1. APPLICATION_NAME: A unique name identifying the airflow deployment
2. AUTHENTICATION_USERNAME: Username for the Airflow web UI authentication
3. AUTHENTICATION_PASSWORD: Password for the Airflow web UI authentication
4. JUPYTER_PASSWORD: For accessing the Jupyter web interface

Rest of the variables are optional and have a value by default which can be changed, if needed.

Some of the useful variables to you as the user could be:

- WORKER_COUNT: The number of workers being deployed, by default 2 workers are deployed
- WORKER_CPU: CPU of each worker deployed
- WORKER_MEMORY: Memory of each worker deployed
- PIP_REQUIREMENTS: Python requirements

## How to Upload DAGs on the Airflow webserver

The current template deploys a Jupyter pod for writing the python code for DAGs. The password will be the one you set in JUPYTER_PASSWORD variable.
**Note** When accessing Jupyter, you need to click on **Upload** to upload an existing python code (.py extension) of the Airflow DAG or you could click on **New->Text File** and then write the python code in the text file itself, *but remember to save it with the .py extension*

## How to Install Custom Python Libraries

There are two ways of doing this as of now. 

1. Before Deployment: Using the *variable* **PIP_REQUIREMENTS** , where you can specify the name of the library separated by whitespace, for eg. pandas scipy

2. After Deployment: If you have a requirements.txt file, you could edit the *configmap* **requirments** and add your requirements there (Please keep in mind, *boto3* is a required library which is present by default, do not remove that otherwise there could trouble accessing the S3 Object store.) 
**NOTE** - when using this option, you need to redeploy the *webserver* and *worker* deployments for the changes to take place!

## How to create a connection to a custom S3 Object Store

This requires the presence of the boto3 library, which is added by default to the requirements configmap.

After that, you can create a connection via the Airflow web ui by clicking on *Admin->Connections* , then fill the following fields:

* Conn Id: use a unique id for the connection and then you need to pass this id in your DAG Python code when interacting with S3
* Conn Type: S3
* Extra: {"aws\_access\_key\_id":"your-access-key-id", "aws\_secret\_access\_key": "your-secret-access-key", "host": "the-s3-endpoint-url"}

