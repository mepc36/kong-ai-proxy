# ACCESS OLLAMA VIA KONG:

## SOURCE:

https://docs.konghq.com/hub/kong-inc/ai-proxy/how-to/llm-provider-integration-guides/llama2/

## OLLAMA DOCKER START COMMAND:

1. You need a service to contain the route for the LLM provider. Create a service first:

```
curl -X POST http://localhost:8001/services \
  --data "name=ai-proxy2" \
  --data "url=http://localhost:32000"
```

2. After installing and starting your Llama2 instance, you can then create an AI Proxy route and plugin configuration. Create the route:

```
curl -X POST http://localhost:8001/services/ai-proxy2/routes \
  --data "name=llama2-chat2" \
  --data "paths[]=~/llama2-chat2$"
```

3. Enable and configure the AI Proxy plugin for Llama2:

```
curl -X POST http://localhost:8001/routes/llama2-chat2/plugins \
 --data "name=ai-proxy" \
 --data "config.route_type=llm/v1/chat" \
 --data "config.model.provider=llama2" \
 --data "config.model.name=llama2" \
 --data "config.model.options.llama2_format=ollama" \
 --data "config.model.options.upstream_url=http://host.docker.internal:11434/api/chat"
```

4. Make an llm/v1/chat type request to test your new endpoint:

```
curl --location 'http://localhost:8000/llama2-chat2' \
--header 'Content-Type: application/json' \
--data '{ "messages": [ { "role": "system", "content": "You are a mathematician" }, { "role": "user", "content": "What is 1+1?"} ] }'
```