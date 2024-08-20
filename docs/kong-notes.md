# KONG COMMON COMMANDS:

## BASIC ARCHITECTURE:

1. Run Llama model inside a docker container that exposes a REST API locally.
2. Proxy request to that llama service via Kong.

## SOURCE:

https://docs.konghq.com/gateway/latest/get-started/services-and-routes/

## KONG START COMMAND (DOCKER):

```
KONG_DATABASE=postgres docker-compose --profile database up -d  
```

## OLLAMA START COMMAND (DOCKER):

```
docker run --network compose_kong-net -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## BASIC KONG DEV CYCLE:

1. Create service
2. Create & route with service created in step #1.

## SERVICES:

1. Create service:

```
curl -i -s -X POST http://localhost:8001/services \
  --data name=example_service \
  --data url='http://httpbin.org'
```

2. Get service config:

```
curl -X GET http://localhost:8001/services/example_service
```

3. Update service:

```
curl --request PATCH \
 --url localhost:8001/services/example_service \
 --data retries=6
```

4. List all services:

```
curl -X GET http://localhost:8001/services
```

## ROUTES:

1. Create route:

```
curl -i -X POST http://localhost:8001/services/example_service/routes \
 --data 'paths[]=/mock' \
 --data name=example_route
```

2. Get route config:

Like services, when you create a route, Kong Gateway assigns it a unique id as shown in the response above. The id field, or the name provided when creating the route, can be used to identify the route in subsequent requests. The route URL can take either of the following forms:

`/services/{service name or id}/routes/{route name or id}`
`/routes/{route name or id}`

To view the current state of the llama_route route, make a GET request to the route URL:

```
curl -X GET http://localhost:8001/services/example_service/routes/example_route
```

3. Update route:

```
curl --request PATCH \
 --url localhost:8001/services/example_service/routes/example_route \
 --data tags="tutorial"
```

4. List all routes:

```
curl http://localhost:8001/routes
```

5. Proxy request:

```
curl -X GET http://localhost:8000/mock/anything
```
