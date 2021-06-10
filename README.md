# airflow-openshift
The purpose of this template is to easily deploy [Apache Airflow](https://airflow.apache.org) on Openshift.

## Airflow container image
The used Airflow container image is built using the official image source codes distributed by Airflow, built with the additional providers packages by Apache Spark, Papermill and Mongo. It should also be possible to build your own image and deploy the template using that image, as long as the image is built using the official sources.

## Useful variables

The template can be imported via the Openshift web UI or added via the command line interface (oc)

The required variables which need to be present in order to run the template are:

- APPLICATION_NAME: A unique name identifying the airflow deployment
- AUTHENTICATION_USERNAME: Username for the Airflow web UI authentication
- AUTHENTICATION_PASSWORD: Password for the Airflow web UI authentication
- JUPYTER_PASSWORD: For accessing the Jupyter web interface

Rest of the variables are optional and have a value by default which can be changed, if needed.

Some of the useful variables to you as the user could be:

- WORKER_COUNT: The number of workers being deployed, by default 2 workers are deployed
- WORKER_CPU: CPU of each worker deployed
- WORKER_MEMORY: Memory of each worker deployed
- PIP_REQUIREMENTS: Python requirements for your DAGs

## How to upload DAGs on the Airflow webserver

The current template deploys a Jupyter pod for writing the python code for DAGs. The password will be the one you set in JUPYTER_PASSWORD variable.
**Note** When accessing Jupyter, you need to click on **Upload** to upload an existing python code (.py extension) of the Airflow DAG or you could click on **New->Text File** and then write the python code in the text file itself, *but remember to save it with the .py extension*

It can take up to 5 minutes for the DAGs to show up in the web UI, so be patient!

## How to install custom Python libraries
- The most reliable way to include your DAG dependencies in the image is to build your own Airflow image using the official sources, and include the dependencies in the building phase of the image. More information about building the image can be found [here](https://airflow.apache.org/docs/docker-stack/build.html). After building the image, push it to some image registry, and set the image link to the AIRFLOW_IMAGE variable to deploy the template using that image.

- You might also want to consider using PythonVirtualenvOperator, that creates a virtual environment for a task with the required pip packages, and tears it down after task completion. For more information about the PythonVirtualenvOperator, see the official documentation [here](https://airflow.apache.org/docs/apache-airflow/stable/howto/operator/python.html?highlight=pythonvirtualenvoperator#pythonvirtualenvoperator) and [here](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/python/index.html?highlight=pythonvirtualenvoperator#airflow.operators.python.PythonVirtualenvOperator).

- There is also an easy and fast way to install any python libraries when deploying the template, but it is error-prone and only recommended for testing. To use the easy method, you can:

    - Before Deployment: Use the variable **PIP_REQUIREMENTS**, where you can specify the name of the libraries separated by whitespace, for example `pandas scipy==1.5.4`

    - After Deployment: Edit the *configmap* **pip-requirements** and add your requirements there, similarly separated by whitespace. **NOTE** - when using this option, you need to redeploy the *scheduler* and *worker* deployments for the changes to take place!

    However, this is a fragile and unrecommended way to install the dependencies, and may result in error. 

## Setting configuration variables
If you want to change the Airflow configuration, the best way to do it is to add new environment variables in the deployment configs of the pods. Be aware that some variables have to be set in the worker pods, while some have to be set in the webserver pod for the effect to take place! For more information about configuring Airflow with environment variables, check the official documentation [here](https://airflow.apache.org/docs/stable/howto/set-config.html). For a list of all available Airflow configurations, see [here](https://airflow.apache.org/docs/stable/configurations-ref.html).

## Configuring email host
Airflow can be configured to send emails: you can both send custom emails through Airflow as a task, or receive alert emails yourself if one of your DAG runs have failed. For the email system to work the following configuration variables have to be set in the deployment config of *worker*:
* AIRFLOW__SMTP__SMTP_HOST
* AIRFLOW__SMTP__SMTP_USER
* AIRFLOW__SMTP__SMTP_PORT
* AIRFLOW__SMTP__SMTP_MAIL_FROM

And, if the smtp host requires it:
* AIRFLOW__SMTP__SMTP_PASSWORD

If you need CSC specific configuration, contact servicedesk@csc.fi.

To use a Google Gmail account as the email host, you first have to create an App Password to your account. To set up an App Password, follow instructions in <https://support.google.com/accounts/answer/185833?hl=en>.

When you have the password, enter these as environment variables in the worker deployment config:
* AIRFLOW__SMTP__SMTP_HOST = smtp.gmail.com
* AIRFLOW__SMTP__SMTP_USER = \<gmail address you used>
* AIRFLOW__SMTP__SMTP_PASSWORD = \<the password you just generated>
* AIRFLOW__SMTP__SMTP_PORT = 587
* AIRFLOW__SMTP__SMTP_MAIL_FROM = \<gmail address you used>

## How to create a connection to a custom S3 object Store

Create a connection via the Airflow web UI by clicking on *Admin->Connections* , then fill the following fields:

* Conn Id: use a unique id for the connection. When interacting with S3, you need to pass this id in your DAG Python code 
* Conn Type: S3
* Extra: `{"aws_access_key_id":"your-access-key-id", "aws_secret_access\_key": "your-secret-access-key", "host": "the-s3-endpoint-url"}`