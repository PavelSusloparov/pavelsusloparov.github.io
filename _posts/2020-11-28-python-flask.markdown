---
layout: post
title: "Python3 - Flask - SQLite"
date:   2020-11-28 12:15:36 -0500
categories: python flask sqlite aws
---

# React app

I wrote a small sample on Python, which includes
* Backend Flask based component
* Files publishing to S3
* Deployment to AWS Elastic Beanstalk

Source code is here - [https://github.com/PavelSusloparov/warranty-service](https://github.com/PavelSusloparov/warranty-service)

The service provides API that will allow the creation and retrieval of warranties.

## In depths

### SqLite

The database is a C-language library. It is a lightweight, file base database, perfect for small projects.
It does not seem suitable for production-like applications due to limitations on simultaneous writes.
Remember, only one actor can modify a file at a time.

The full feature set of the SQLite - [link](https://www.sqlite.org/features.html)

### Server

I found it is much easier to use Flask/Python application comparing with Spring Boot/Java.
I like specifically the simplicity of the database migration part:

1. Database schema creates on the app bootstrap; no need for a separate script.
2. Initial data loads with a python script. Example is [here](https://github.com/PavelSusloparov/warranty-service/blob/master/scripts/init_db.py).

A pleasant finding for me was the Marshmallow library. It gives the ability to validate requests and check the data against the defined object during the serialization.
An example of a request object is [here](https://github.com/PavelSusloparov/warranty-service/blob/master/service/schema.py#L45)

The behavior of the validator is clear from the unit tests - [link](https://github.com/PavelSusloparov/warranty-service/blob/master/service/tests/test_routes_create_warranty.py#L53)

### S3

Files upload in S3 is extremely intuitive with the Boto3 library. The upload method - [link](https://github.com/PavelSusloparov/warranty-service/blob/master/service/routes.py#L64).
The upload is async base and does not block the response of the endpoint.

### AWS Elastic Beanstalk

The SaaS service makes easier the effort of creating a docker container.
With Elastic Beanstalk CLI installed locally, you need to deploy command from the project root and answer a set of questions.
However, it takes effort to install Elastic Beanstalk CLI locally.

I learned the hard way two things:

1. The tool works with Python 3.6 max(at least at the point of me doing the POC). So I had to downgrade.
2. I have to define my installed python binary as the interpreter for the Elasticbeanstalk CLI. Otherwise, it installs Python to `venv.`

The last bit, which gives me around an hour of head-scratching, is the application entry point.
By default, Elastic Beanstalk is looking for application.py in the root directory and crashes on bootstrap.
I had to configure the `aws:elasticbeanstalk:container:python` option to be `WSGIPath: service:application` since my project root locates in `./service.`
I used AWS console, Elasticbeanstalk->Application->Configure to set it up. Another option is to use `eb configure.`

Try to run the project on your local with your AWS credentials.

Thank you for reading! 
