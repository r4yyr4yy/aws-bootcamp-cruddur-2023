# Week 2 — Distributed Tracing

## Honeycomb Setup

We are usig Honeycomb to help quickly make sense of thosand of rows of data in our project needed to fully represent the user experience in your complex, unpredictable systems.

Create a new environment and get the API key under the API key tab "xFOH6frzRh7PShOBCC5BxA"

INSERT PIC

Set environment variables:

```
export HONEYCOMB_API_KEY="xFOH6frzRh7PShOBCC5BxA"
gp env HONEYCOMB_API_KEY="xFOH6frzRh7PShOBCC5BxA"
```
```
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

Setting up Honeycomb for the backend

add the following code in the docker compose file

```
OTEL_SERVICE_NAME: "backend-flask"
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

