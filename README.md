# Apache Airflow - Hands on Guide

To reinforce my Airflow knowledge which I'm currently developing as part of a Data Engineering bootcamp, I have decided to work through a [Udemy course](https://www.udemy.com/course/the-ultimate-hands-on-course-to-master-apache-airflow). I'll add notes and progress here. Maybe it'll come in handy for some people!

## Airflow Basics 

### Why Airflow?

Let's say you want to run your pipeline at a specific time each day. The pipelines has multiple steps, with each step following on from the last, and with each step depending on the success of the last. With Airflow, we can manage and schedule our data pipelines.

### What is Airflow?

Airflow is an open source platform to programmatically **author**, **schedule** and **monitor** workflows. It is an orchestration tool. 

* Dynamic - everything is coded in Python. Everything we can do in Python, we can do in Airflow.
* Scalable - can execute as many tasks as we want when we want
* UI - a nice user interface to monitor our data pipelines
* Extensibility - can extend functionalities of airflow

Airflow is composed of:

1. Webserver - Flask server, serving the UI
1. Scheduler - In charge of scheduling workflows
1. Metastore - Database where metadata is stored
1. Executor - Class defining how your tasks should be executed on which system
1. Worker - Process or Sub Process executing your task

### What is a DAG?

A DAG stands for Directed Acyclic Graph. These are basically data pipelines.

There are directed (edges). DAGs are acyclic, meaning an earlier task can't depend on a later task running.

### What is an Operator?

An operator is a task in our DAG.

1. Action Operators - In charge of executing something (e.g. the python operator, or bash operator)
1. Transfer Operators - Transferring data from source to destination
1. Sensor Operators - Waits for something to happen, before moving forward

### What is a Task Instance?

When an operator runs in our DAG, this is a task instance

### What Airflow is not?

Not a data streaming solution, neither a data processing framework.

### How Airflow Works?

1. One Node Architecture

    Here we just have one node, with a web server, scheduler, metastore, and executor included.

    1. Our `web server` fetches data from the `metastore` to display information corresponding to our DAGs.
    2. The `scheduler` works with the `metastore` and `executor` to execute tasks. 

    The executor has an internal queue. This is how our tasks are executed in the right order. 

2. Multi Nodes Architecture using Celery

    Celery is a way to run tasks on multiple machines.

    Here we have a multiple nodes. One node with just scheduler, executor and web server. The other with just the metastore and queue. Redis is used to spread tasks across multiple machines.

    The main difference here is that the executor pushes task to the queue. Then workers on different machines fetches tasks from the queue using Redis.

Once DAG is in the folder `dags`. The webserver and scheduler parse that DAG. The scheduler will verify that it can be run, and create a DagRun Object which is stored in the metastore of airflow with status `running`.

If there is a task ready to be triggered in our DAG, the scheduler creates a task instance with the status `scheduled` in the metastore. This task instance is sent to the Executor. 

Once the Executor is ready to run the task, the task instance object has status `running` and executor updates status of task in the metastore. The scheduler then verified if it is done. If so, the object has the status `complete`.

The scheduler also updates the web server. 

REMEMBER: Different status are given to our tasks, and web server and scheduler both parse our DAG.

### Installing Airflow 2.0 Manually

We'll first install it with manually with Docker. This is like a little special virtual machine running inside our machine. Here's the gist:

#### Create our Docker Container based on Python 3.8, then enter a shell session for it

```bash
docker run -it --rm -p 8080:8080 python:3.8-slim /bin/bash
```

#### Create an environmental variable for airflow home which will be used to store dags, logs and configuration file

```bash
export AIRFLOW_HOME=/usr/local/airflow
```

#### Install tools and dependencies required for Airflow

```bash
apt-get update -y && apt-get install -y wget libczmq-dev curl libssl-dev git inetutils-telnet bind9utils freetds-dev libkrb5-dev libsasl2-dev libffi-dev libpq-dev freetds-bin build-essential default-libmysqlclient-dev apt-utils rsync zip unzip gcc && apt-get clean
```

#### Create airflow user

```bash
useradd -ms /bin/bash -d ${AIRFLOW_HOME} airflow
```

#### Upgrade Pip

```bash
pip install --upgrade pip
```

#### login to Airflow

```bash
su - airflow
```

#### Create a Python Virtual Environment

```bash
python -m venv .sandbox
```

#### Activate Virtual Environment

```bash
source .sandbox/bin/activate
```

#### Download a requirements file to install right version of Airflow's dependencies

```bash
wget https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.8.txt
```

#### Install Apache Airflow, with sub-dependencies defined between square brackets

```bash
pip install "apache-airflow[crypto,celery,postgres,cncf.kubernetes,docker]"==2.0.2 --constraint ./constraints-3.8.txt
```

#### Initialise Airflow Metadatabase

```bash
airflow db init
```

#### Start Airflow Scheduler

```bash
airflow scheduler &
```

#### Create User

```bash
# Show us how to create a user
airflow users create -h
```
```bash
# Create user
airflow users create -u admin -f admin -l admin -r Admin -e admin@airflow.com -p admin
```

#### Start Webserver

```bash
airflow webserver &
```

Once you've set everything, you can access the webserver by navigating to http://localhost:8080

But.... there's an easier way to do this. We can use Docker again, but this time we don't need to do a load of manual installation steps.

### Installing Airflow 2.0 with Docker

I won't run through the steps here. You can find details on Airflow's website, or sign up to the course I'm doing to see the Dockerfile being used. 

But essentially, we're just creating a Docker image which contains all the steps we just did. We can therefore skip having to go through, each 
part manually.

It's worth doing a Docker tutorial to understand all of this better. I actually took some notes from a YouTube tutorial I worked through [here](https://github.com/ABZ-Aaron/DockerOverview).

Once the image is built, this command will need to be run:

```bash
docker run -p 8080:8080 airflow-basic
```

## Airflow UI

<img src="https://github.com/ABZ-Aaron/AirflowOverview/blob/b838e9f801b4d783bb5993e881f45542b1bc9115/images/Screenshot%202022-02-03%20at%2015.57.56.png" width=60% height=60%>

Here we can see a list of example DAGs. Each DAG has tags, which we can filter by.

We also can see an `Owner` field which is used for auditing purposes. 

Under `Runs` we can see the status of all previous DAG runs.

Under `Schedule` we can see the schedule for the DAG.

The rest is fairly self-explanatory. Hover over things to see descriptions. Play around with it, see if you can find where the logs are for each task, etc.

## Airflow CLI

For some commands, you need to use the command line interface. 

To run these Airflow commands, you will need to be in your Airflow container:

```bash
docker exec -it <CONTAINER ID> /bin/bash 
```

To see commands:

```bash
airflow -h
```

Notice that commands are in groups. If you want to interact with DAGs, you'll need to type `airflow.dags <SOMETHING>` for example.

#### Print Current Working Directory

```bash
pwd
```
#### Initialise Metadatabase

```bash
airflow db init
```
#### Drop Everything (probably won't use this in production)

```bash
airflow db reset
```
#### Upgrade schemas that in Metadatabase (latest schemas, values, etc)

```bash
airflow db upgrade
```
#### Start Webserver

```bash
Start Airflow’s webserver
```
#### Start Scheduler
``airflow scheduler``
* Start Airflow’s scheduler

#### Start a Celery Worker (useful in distributed mode to spread tasks among nodes)

```bash
airflow celery worker
```

#### Give the list of known DAGS

```bash
airflow dags list
```

#### List Working Directory

```bash
ls
```

#### Trigger DAG with date in the past execution date (leave date out if you want it to run straight away)

```bash
airflow dags trigger example_python_operator -e 2021-01-01
```
#### Trigger DAG with data in the future

```bash
airflow dags trigger example_python_operator -e '2021-01-01 19:04:00+00:00'
```

#### Display history of DAG runs

```bash
airflow dags list-runs -d example_python_operator
```

#### List tasks contained in a DAG

```bash
airflow tasks list example_python_operator
```

#### Test A DAG

```bash
airflow tasks test example_python_operator print_the_context 2021-01-01
```

This last one allows us to test our DAG, and is useful to use right after we add a task. the `example_python_operator` is the DAG ID and the `print_the_context` is the Task ID.

