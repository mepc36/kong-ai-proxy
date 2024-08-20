# Summary:

This repo is a minimal example. It shows how to call a Llama model via a [Kong API Gateway](https://github.com/Kong/kong). It manages the Llama model via [Ollama](https://github.com/ollama/ollama).

This repo is a WIP. Approach instructions accordingly.

## Sources:

This repo is the result of following these 2 tutorials. Please refer to them if you run into any snags.

1. Kong docs [here](https://docs.konghq.com/hub/kong-inc/ai-proxy/how-to/llm-provider-integration-guides/llama2/).
2. Ollama docs [here](https://github.com/ollama/ollama?tab=readme-ov-file#rest-api)

## Running this App:

To create our Kong API Gateway, we will:

1. Pull & run a Kong Docker image
2. Create an `ai-proxy` service on that Kong API Gateway
3. Expose a Llama model via a RESTful API using ollama
3. Create a route on our Kong service that proxies requests to that ollama RESTful API.

Here's how:

1. Change directory into `~/docker-kong/compose`:

```
cd ./docker-kong/compose
```

2. Start the Kong API Gateway as a daemonized Docker container:

```
KONG_DATABASE=postgres docker-compose --profile database up -d
```

3. Download the ollama CLI:

```
brew install ollama
```

4. Download the llama 2 model using the ollama CLI:

```
ollama pull llama2
```

5. Get the name of the Docker network that the Kong API Gateway is running on:

```
docker network ls
```

4. Pull & run an ollama RESTful API inside a Docker container on the same Docker network as your Kong API Gateway. For example, `compose_kong-net`:

```
docker run --network compose_kong-net -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

We run the services on the same Docker network so that `host.docker.internal` refers to the same IP address for both the `kong` and `ollama` services. See the configure-plugin step below for more.

5. Create the `ai-proxy` service:

```
curl -X POST http://localhost:8001/services \
  --data "name=ai-proxy" \
  --data "url=http://localhost:32000"
```

Please note that the created URL will be overwwritten by the AI Proxy plugin. More info [here](https://docs.konghq.com/hub/kong-inc/ai-proxy/how-to/llm-provider-integration-guides/llama2/#using-the-plugin-with-llama2).

6. Create the route for the AI Proxy service:

```
curl -X POST http://localhost:8001/services/ai-proxy/routes \
  --data "name=llama2-chat2" \
  --data "paths[]=~/llama2-chat2$"
```

7. Configure the AI Proxy Plugin for Llama 2:

```
curl -X POST http://localhost:8001/routes/llama2-chat2/plugins \
 --data "name=ai-proxy" \
 --data "config.route_type=llm/v1/chat" \
 --data "config.model.provider=llama2" \
 --data "config.model.name=llama2" \
 --data "config.model.options.llama2_format=ollama" \
 --data "config.model.options.upstream_url=http://host.docker.internal:11434/api/chat"
```

8. Make an `/llm/v1/chat` type request to test your new endpoint:

```
curl --location 'http://localhost:8000/llama2-chat2' \
--header 'Content-Type: application/json' \
--data '{ "messages": [ { "role": "system", "content": "You are a mathematician" }, { "role": "user", "content": "What is 1+1?"} ] }'
```

9. In case you need to delete a service, first get its service ID...

```
curl -X GET http://localhost:8001/services
```

...and then send a DELETE request:

```
curl -X DELETE http://localhost:8001/services/82febcf5-88ab-4ae6-a41c-2e6b90f0cec6
```

Please note that you cannot delete a service if it still has routes, so make sure you delete all of a service's rotues before attempting to delete the service itself.

First, list all routes...

```
curl http://localhost:8001/routes
```

...and then pass in the target route's ID to a DELETE call:

```
curl -X DELETE "http://localhost:8001/services/ai-proxy/routes/53cae54a-4197-4d9c-bfb0-41a72eea5229"
```

## Resources:

Kong has made a POSTman collection available with common operations [here](https://www.postman.com/api-evangelist/kong/request/2fvao6o/delete-route).

Docs on Kong's AI Proxy plugin can be found [here](https://docs.konghq.com/hub/kong-inc/ai-proxy/).

## TODO:

1. Can we remove `~/docker-kong` and build right from Docker Hub, just like we do with ollama?
2. Integrate with llama3, not llama2.
3. Add Gemma as LLM.
4. Move to cloud.
