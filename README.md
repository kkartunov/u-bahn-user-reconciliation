DEPRECATED 2/1/2024 https://topcoder.atlassian.net/browse/CORE-203

# UBahn - User Reconciliation Processor

Sync a v3 user  

## Dependencies

- Nodejs(v12+)
- Kafka

## Configuration

Configuration for the user reconciliation processor is at `config/default.js`.
The following parameters can be set in config files or in env variables:

- LOG_LEVEL: the log level; default value: 'debug'
- KAFKA_URL: comma separated Kafka hosts; default value: 'localhost:9092'
- KAFKA_CLIENT_CERT: Kafka connection certificate, optional; default value is undefined;
    if not provided, then SSL connection is not used, direct insecure connection is used;
    if provided, it can be either path to certificate file or certificate content
- KAFKA_CLIENT_CERT_KEY: Kafka connection private key, optional; default value is undefined;
    if not provided, then SSL connection is not used, direct insecure connection is used;
    if provided, it can be either path to private key file or private key content
- KAFKA_GROUP_ID: the Kafka group id, default value is 'user-reconciliation-processor'
- USER_RECONCILATION_TOPIC: Kafka topic, default value is 'backgroundjob.reconcile.user'
- V5_API_URL: The topcoder v5 api base url, default value: 'https://api.topcoder-dev.com/v5'
- AUTH0_URL: The auth0 url, default value: 'https://topcoder-dev.auth0.com/oauth/token'
- AUTH0_AUDIENCE: The auth0 audience for accessing ubahn api(s), default value: 'https://m2m.topcoder-dev.com/'
- AUTH0_CLIENT_ID: The auth0 client id
- AUTH0_CLIENT_SECRET: The auth0 client secret
- AUTH0_PROXY_SERVER_URL: The auth0 proxy server url
- PUBLISH_TOPIC: after processing publish to another kafka topic i.e. 'legacy.sync.user'

There is a `/health` endpoint that checks for the health of the app. This sets up an expressjs server and listens on the environment variable `PORT`. It's not part of the configuration file and needs to be passed as an environment variable

## Local Kafka setup

### Install bin

- `http://kafka.apache.org/quickstart` contains details to setup and manage Kafka server,
  below provides details to setup Kafka server in Linux/Mac, Windows will use bat commands in bin/windows instead

### Local install with Docker

- Navigate to the directory `docker-kafka`
- Run the command `docker-compose up -d`

## Local deployment

1. Make sure that Kafka is running.

2. From the project root directory, run the following command to install the dependencies

    ```bash
    npm install
    ```

3. To run linters if required

    ```bash
    npm run lint
    ```

    To fix possible lint errors:

    ```bash
    npm run lint:fix
    ```

4. Start the processor and health check dropin

    ```bash
    npm start
    ```

## Local Deployment with Docker

To run this processor using docker, follow the below steps

1. Navigate to the directory `docker`

2. Rename the file `sample.api.env` to `api.env`

3. Set the auth0 config in the file `api.env`

4. Once that is done, run the following command

    ```bash
    docker-compose up
    ```

5. When you are running the application for the first time, It will take some time initially to download the image and install the dependencies

## Verification

1. config `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`
2. start kafka-console-producer to write messages to `backgroundjob.reconcile.user`
  `docker exec -it user-reconciliation-processor_kafka /opt/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic backgroundjob.reconcile.user`
3. write message:
  `{ "topic": "backgroundjob.reconcile.user", "originator": "zapier", "timestamp": "2021-05-08T00:00:00.000Z", "mime-type": "application/json", "payload": {"handle":"billsedison"} }`
4. Watch the app console, It will show message successfully handled.
5. write non handle message:
  `{ "topic": "backgroundjob.reconcile.user", "originator": "backgroundjob.service", "timestamp": "2021-05-08T00:00:00.000Z", "mime-type": "application/json", "payload": {"id":"88774616","firstName":"Sachin1","lastName":"Kumar1"} }`
6. watch the app console, it will show the ignoring message 
