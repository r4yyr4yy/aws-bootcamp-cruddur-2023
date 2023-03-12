# Week 2 — Distributed Tracing

### Honeycomb Setup

We are usig Honeycomb to help quickly make sense of thosand of rows of data in our project needed to fully represent the user experience in your complex, unpredictable systems.

Create a new environment and get the API key under the API key tab "xFOH6frzRh7PShOBCC5BxA"

INSERT PIC

#### Set environment variables:

```
export HONEYCOMB_API_KEY="xFOH6frzRh7PShOBCC5BxA"
gp env HONEYCOMB_API_KEY="xFOH6frzRh7PShOBCC5BxA"
```
```
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

#### Setting up Honeycomb for the backend

add the following code in the docker compose file

```
OTEL_SERVICE_NAME: "backend-flask"
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

add the following python in the requirements.txt file in backend-flask folder to install these packages to instrument a Flask app with OpenTelemetry:

``` 
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

![]Ref: https://ui.honeycomb.io/cloudsec/environments/bootcamp/send-data#

Now, change directory to backend-flask and install the dependencies

```
pip install -r requirements.txt
```

#### Update the app.py file with the code below

```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.trace.export import ConsolepanExporter, SimpleSpanProcessor
```

Initialize tracing and an exporter that can send data to Honeycomb

provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)

##### Show logs within the backend-flask app

simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)

trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

##### Initialize automatic instrumentation with Flask

app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

Run the docker compose file

NB: Make sure ports 3000 and 4567 are open and click on the link

insert pic (port 3000 -- Frontend)

insert pic (port 4567 --- Backend)

 #### Add the code below to the backend-flask in the docker compose file
 
 OTEL_SERVICE_NAME: "backend-flask"
 OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
 OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"


Let's create a span in the backend-flask/services/home_activities.py file to .... by getting a tracer


from opentelemetry import trace

tracer = trace.get_tracer("tracer.name.here")

with tracer.start_as_current_span("http-handler"):

#### ------------------------ Final code ------------------------------------------

```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      OTEL_SERVICE_NAME: "backend-flask"
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
volumes:
  db:
    driver: local
```
### AWS X RAY Setup

#### Add the following code to gitpod.yml file to install npm when gitpod is launched

```
- name: react-js
    command:
      cd frontend-react-js
      npm i	  
```

#### Add SDK for Amazon XRay

add aws-xray-sdk library to backend-flask/requirements.txt file and run *pip install -r requirements.txt* in backend-flask

Add the following to the app.py in the backend-flask/services

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
 
```
lets make a json file to xray.json

```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}

```
### Install Cloudwatch

Install Watchtower to handle logs for AWS CloudWatch logs

add Watchtower to backend-flask/requirements.txt and run pip install -r requirements.txt

```
Watchtower
```

```
pip install -r requirements.txt
```

Add the following code to app.py in the backend-flask/services folder

```
import watchtower
import logging
from time import strftime
```
