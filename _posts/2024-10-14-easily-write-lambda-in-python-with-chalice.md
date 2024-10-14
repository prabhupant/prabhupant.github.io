---
layout: post
title: "How to create AWS Lamda functions in Python"
---

Serverless architecture is a software design pattern where applications are hosted by a third-party, hence eliminating the need to manage the server software and hardware by the developer. Similarly, serverless functions are single-purpose, programmatic functions that are hosted on managed infrastructure. 

Some of the advantages/characterstics of a serverless functions are -
1. They are infinitely scalable (this sclability only depends on the third-party service for hosting these functions)
2. They undertake only one single task
3. They are basically not ACID compliant
4. They provide improved latency and geolocation
5. They cost low as comparted to hosting your own server (both in time and money)

With the many advantages and comfort of serverless functions, there are some drawbacks - 
1. Dependence on third-party for hosting
2. Hard to debug when many serverless functions are used in cohorts
3. Not ACID compliant. Avoid making CRUD operations using serverless functions. If you have to do database operations, then you'll need to use some queuing service.

The fundamental principle to remember about using serverless function is that they should be used for one purpose ONLY. If you want a serverless function to parse some XML files, then that should be the only thing the serverless function should do. Some common use cases for serverless functions are - 
1. Sending bulk/single email
2. OTP authentication
3. Cleaning data
4. Triggers for certain events
5. Storing or fetching something from a storage service like AWS S3

So, serverless architecture follows the mantra "Focus on your application, not on infrastructure". Amazon offers [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html), a serverless computing platform part of the AWS suite and writing serverless functions has never been easier! Let's try to create our own AWS Lambda function.

AWS provides a Python module called [Chalice](https://github.com/aws/chalice) which considerably reduces hassle of writing Lambda functions. First of all, make sure you have an AWS account and aws-cli is installed and configured on your machine. Now let's get to writing a basic Lambda function to check if a string is palindrome or not.

1. Create a virtual environment.

```bash
$ python3 -m venv venv
$ source venv/bin/activate
```

2. Now install chalice

```bash
$ pip install chalice
```

3. Start the project

```bash
$ chalice new-project palindrome
```

This will create a directory with the name `palindrome`. `cd` into the directory and you will see two main files - `app.py` and `requirements.txt`. `app.py` is where your code will rest and `requirements.txt` is to list the requirements of the `palindrome` function.

Now open the `app.py` file and you will see something like - 

```python
from chalice import Chalice

app = Chalice(app_name='palindrome')


@app.route('/')
def index():
    return {'hello': 'world'}
```

If you have ever used Flask, then this code must be familiar to you. The `app` variable is where your Chalice app is created and `@app.route` decorator is used for routing and associating a particular function to a route. Let's modify this

```python
from chalice import Chalice

app = Chalice(app_name='palindrome')


@app.route('/', methods=['POST'])
def check_palindrome():
    body = app.current_request.json_body # To get the JSON data from POST request
    word = body['word']

    if word[::-1] == word:
        return True
    else:
        return False

```

In order to allow the `check_palindrome` function accept POST requests, I have added the `methods` argument to the decorator and passed `POST` as its value. This argument accepts a list, so if you want this function to accept both GET and POST, then the argument must look like this `methods=['GET', 'POST']`. Inside the function, we are receiving the JSON body in `body` variable and then extracting the `word` key which contains the word to be checked for palindrome.

4. Our function is now ready and we can deploy it! First, freeqe the requirements.

```bash
$ pip freeze > requirements.txt
```

There are two stages to deploy the function in - `dev` and `prod`. By default, Chalice deploys all functions to dev.

To deploy in dev, do

```bash
$ chalice deploy
```

Now this will return an AWS API URL. This is the URL where our function is deployed. Copy it and check it in Postman with this input

```json
{
  "word": "racecar"
}
```
and see it working.

5. When your Lambda function is working fine, you can deploy it in prod.

```bash
$ chalice deploy --stage prod
```

This will give you another URL where your API is hosted in prod environment. It is generally a good practice to test all your changes on the dev stage and when you are fully sure and the API is tested thoroughly, then deploy it on prod.
