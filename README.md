# Apache Airflow - Hands on Guide

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