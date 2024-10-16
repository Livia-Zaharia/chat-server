# Chat Server

This project demonstrates a Hugging Face chat UI integrated with a LiteLLM proxy deployment. By default, it uses OpenAI through a caching proxy.

## Quick Start Guide

### 1. Clone the Repository

```bash
git clone https://github.com/longevity-genie/chat-server.git
cd chat-server
```

### 2. Set Up the Environment

a. Install Docker and Docker Compose (if not already installed):
   - For Ubuntu users, you can use the provided script:
     ```bash
     ./install_docker_ubuntu.sh
     ```
   - For other operating systems, please refer to the official Docker documentation.

b. Create configuration files:
   ```bash
   cp .env.local.template .env.local
   cp .env.proxy.template .env.proxy
   cp .env.db.template .env.db
   ```

c. Edit the configuration files:

   This chat server uses several configuration files to set up the environment, API keys, and model settings. You'll need to edit these files to customize your setup.

   1. Edit `.env.local`:
      ```bash
      nano .env.local
      ```
      This file configures the chat UI settings. You'll need to set:
      - `OPENAI_API_BASE`: The base URL for your API (default is `http://127.0.0.1:14000/v1`) or it can be any OpenAI or any other REST API which support OpenAI standard.
      - Other UI-related settings like `PUBLIC_APP_NAME`, `PUBLIC_DEFAULT_SYSTEM_PROMPT`, etc.

   2. Edit `.env.proxy`:
      ```bash
      nano .env.proxy
      ```
      This file contains API keys for various services. Add your keys here:
      - `OPENAI_API_KEY=your_openai_api_key_here`
      - `GROQ_API_KEY=your_groq_api_key_here`
      - Add any other API keys required for models you plan to use

   3. Edit `proxy.yaml`:
      ```bash
      nano proxy.yaml
      ```
      This file configures the LiteLLM proxy settings and available models. Adjust as needed:
      - Modify the `model_list` section to include the models you want to use
      - Adjust `litellm_settings` if necessary (e.g., drop_params, add_function_to_prompt)


d. Configure custom REST API endpoint (if needed):

   If you're using a custom API endpoint (e.g., for an agentic chain), you'll need to configure it in two places:

   1. In `.env.local`, set the model baseURL:
      ```
      OPENAI_API_BASE=http://127.0.0.1:14000/v1
      ```

   2. In `docker-compose.yml`, set the baseURL for your agent:
      ```bash
      nano docker-compose.yml
      ```
      Look for the `proxy` service configuration and adjust the `command` section:
      - For containerized endpoints, use the `chat-server` network
      - For endpoints outside containers, use the Docker gateway IP (usually `172.17.0.1`)
      - For local endpoints, use `localhost` or `127.0.0.1` with port `14000`

These configuration steps set up the chat server environment, including:
- The chat UI settings and appearance
- API keys for accessing language models
- Available models and their configurations
- Database credentials for storing chat history
- Custom API endpoints for specialized use cases

By carefully configuring these files, you're preparing the chat server to connect to the right APIs, use the correct models, and store data securely. This setup allows the chat UI to communicate with the language models through the LiteLLM proxy, providing a seamless chat experience while maintaining flexibility and security.

### 3. Understanding Port Configuration

The `docker-compose.yml` file defines several services with specific port mappings. Here's an overview:

- huggingchat-ui: `0.0.0.0:13000:3000`
  This service is accessible from any IP on the host machine on port 13000.

- litellm-proxy: `127.0.0.1:14000:4000`
  This service is only accessible locally on the host machine on port 14000.

- llm-cache: `127.0.0.1:16379:6379`
  The Redis cache is accessible locally on port 16379.

- chat-mongo: `127.0.0.1:17017:27017`
  The MongoDB service is accessible locally on port 17017.

In these port mappings:
- The first number is the host port (what you access on your machine).
- The second number is the container port (what the service uses inside Docker).
- `0.0.0.0` means the service is accessible from any IP address.
- `127.0.0.1` means the service is only accessible locally on the host machine.

### 4. Launch the Application

a. Start the OpenAI API endpoint:
   ```bash
   python openai_api_endpoint.py
   ```
It can be openai rest API directly, it can be any LLM model wrapped by OpenAI compatible REST API or it can be some custom REST API that extends model with additional features (like RAG), for example (https://github.com/longevity-genie/longevity_gpts/blob/main/openai_api_endpoint.py) 


b. Run the Docker containers:
   ```bash
   docker-compose up
   ```

## Advanced Options

### Building chat-ui from source

By default, `docker-compose.yml` uses the pre-built `longevity-genie/chat-ui`. To build from source:

1. Update submodules:
   ```bash
   git submodule update --init --recursive
   git pull --recurse-submodules
   ```

2. Build and run:
   ```bash
   docker-compose up --build
   ```

## Troubleshooting

If you encounter any issues, please check the following:
1. Ensure all API keys are correctly set in the `.env` files
2. Verify that Docker and Docker Compose are properly installed
3. Check if all required ports are available on your system

For further assistance, please open an issue on the GitHub repository.

## Example

For running the chat UI with the longevity gpt you can follow the next steps:
1. in longevity_gpts clone repo folder you should run the `openai_endpoint_api.py`
2. in the same file notice the port it is forwarding to, in this case 8088, found in this part of the code
```
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("openai_api_endpoint:app", host="0.0.0.0", port=8088, workers=10)
```
3. be sure to have also cloned the repo for chat-ui, found separately here ('https://github.com/longevity-genie/chat-ui')
4. start editing all the env.local, .env according to instructions
4.1 for `.env.local`
   ```
   MONGODB_URL=mongodb://genie:super-secret-password@chat-mongo:27017
   ALLOW_INSECURE_COOKIES=true
   OPENAI_API_BASE=http://127.0.0.1:8088/v1
   ```
   also MODELS  has to be written according to what you intend to use, example
   ```
   MODELS=`[{
      "name": "gpt-4o-mini",
      "displayName": "proxified-gpt-4o",
      "description": "OpenAI gpt-4o-mini model served through cache-proxy",
      "parameters": {
        "temperature": 0.0,
        "max_new_tokens": 10000,
            "stop": ["[DONE]"]
      },
      "endpoints": [
        {
          "type": "openai",
          "baseURL": "http://litellm-proxy:4000/v1",
          "apiKey": "no_key_needed"
        }
      ],
      "promptExamples": [
            {
              "title": "What processes are improved in GHR knockout mice?",
              "prompt": "What processes are improved in GHR knockout mice?"
            },
            {
              "title": "What genes need to be downregulated in worms to extend their lifespan?",
              "prompt": "What genes need to be downregulated in worms to extend their lifespan?"
            },
      ]
   },
   ]`
   ```
4.2 for `.env` file found in chat-ui submodule
   ```
MONGODB_URL=mongodb://genie:super-secret-password@localhost:27017/
MONGODB_DB_NAME=chat-ui
MONGODB_DIRECT_CONNECTION=false
   ```

5. run ``` docker- compose up```

6. open browser and go to adress ``` 0.0.0.0:13000 ```
