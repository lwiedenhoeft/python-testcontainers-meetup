# testcontainers-python Meetup

![alt text](https://d33wubrfki0l68.cloudfront.net/a661dbbe55be3e9cb77889f24835a44c6daf53c2/ce0aa/logo.png "Testcontainer")



We try to demonstrate the following points:

1. Integration of testcontainers-python with [behave](https://behave.readthedocs.io/en/latest/) (or similar python 
Cucumber port).
2. The spinning up of a generic container - running RabbitMQ in this case.
3. Waiting for the application running in the container to finish initializing.
4. Interacting with the application running inside the test container.

## Start

To run the code and the integration tests, initialize a virtual environment.

    $ python -m venv ./.venv && source .venv/bin/activate && pip install -r requirements.txt
    
It is assumed that docker will have been installed and is running on the machine being used.
    
### run.py

`run.py` is the simple application that is being tested. It is a very simple web server that returns
'Hello, world!' in response to `GET` requests and adds the contents of the request body to RabbitMQ when a `POST` request
is received. 

To manually test the application, spin up a docker container running RabbitMQ:

    docker run -d --hostname my-rabbit -p 15672:15672 -p 5672:5672 --name some-rabbit rabbitmq:3.8.5-management
    343dde1b292fe975c3b1b5ef8d318922c0da12cecdc9f31055e159592bc1248b
    
Run the simple web server specifying the http port to listen for requests on and the rabbit port to send messages on:

    $ source .venv/bin/activate
    $ python run.py 8082 5672
    
Interact with the web server using cURL:

    $ curl http://localhost:8082
      Hello, world!
    
    $ curl -v http://localhost:8082 -d'[{"hello":"world"}]'
      *   Trying ::1...
      * TCP_NODELAY set
      * Connection failed
      * connect to ::1 port 8082 failed: Connection refused
      *   Trying 127.0.0.1...
      * TCP_NODELAY set
      * Connected to localhost (127.0.0.1) port 8082 (#0)
      > POST / HTTP/1.1
      > Host: localhost:8082
      > User-Agent: curl/7.64.1
      > Accept: */*
      > Content-Length: 19
      > Content-Type: application/x-www-form-urlencoded
      >
      * upload completely sent off: 19 out of 19 bytes
      * HTTP 1.0, assume close after body
      < HTTP/1.0 201 Created
      < Server: BaseHTTP/0.6 Python/3.8.1
      < Date: Mon, 06 Jul 2020 22:12:57 GMT
      <
      * Closing connection 0
    
Browse to http://localhost:15672/#/queues/%2F/test_queue using the credentials guest:guest to view the queue and see 
the 'hello practice' message added to the queue.

### Running the integration tests

To run the integration tests making use of testcontainers-python, activate the virtual environment and run `behave`:

    ~/W/c/testcontainers-stammtisch master !1 ?1 ❯ behave                                       testcontainers-stammtisch 00:05:52

    Pulling image rabbitmq:3.8.5-management
    ⠹
    Container started:  c2403dfa78
    Feature: Testcontainers POC: demonstrates use of Testcontainers for integration testing # features/everything.feature:1

      Scenario: When a GET request is made to the simple webserver, a successful 'Hello World' response is returned  # features/everything.feature:3
        When a GET request is made to the simple webserver                                                           # features/steps/steps.py:12 0.007s
        Then the webserver will respond with a HTTP status of "200"                                                  # features/steps/steps.py:27 0.000s
        And the response body will contain "Hello, world!"                                                           # features/steps/steps.py:32 0.000s

      Scenario: When a POST request is made to the simple webserve, the request body is added to the RabbitMQ queue  # features/everything.feature:8
        When a POST request is made to the simple webserver                                                          # features/steps/steps.py:17 0.024s
        Then the webserver will respond with a HTTP status of "201"                                                  # features/steps/steps.py:27 0.000s
        And the response body will be empty                                                                          # features/steps/steps.py:37 0.000s
        And the request body is added to the RabbitMQ queue                                                          # features/steps/steps.py:42 0.022s

    1 feature passed, 0 failed, 0 skipped
    2 scenarios passed, 0 failed, 0 skipped
    7 steps passed, 0 failed, 0 skipped, 0 undefined
    Took 0m0.055s

