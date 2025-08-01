# Deployment Plan: MCP Server on Railway

The most straightforward approach is to leverage Railway's native support for `docker-compose.yml`. This will allow you to provision both the `neo4j` database and the `graphiti-mcp` application as distinct services within a single Railway project, preserving the dependencies and configurations you've already defined.

**Here are the detailed steps:**

**Step 1: Fork the Repository**

1.  **Fork the `getzep/graphiti` repository** on GitHub. This will create a copy of the project under your own GitHub account.

**Step 2: Project Setup on Railway**

1.  **Create a New Project**: Start by creating a new, empty project on Railway.
2.  **Connect to GitHub**: Link the Railway project to **your forked repository**. When you authorize Railway, it will detect the `docker-compose.yml` file in the root of your repository.
3.  **Import Services**: Railway will prompt you to import the services from your `docker-compose.yml` file. You should see both `neo4j` and `graphiti-mcp` listed. Confirm the import.

**Step 3: Service Configuration**

Railway will automatically configure the services based on your `docker-compose.yml`, but you'll need to verify and set the environment variables.

*   **For the `neo4j` service:**
    *   Railway will use the `neo4j:5.26.0` image.
    *   The environment variables for memory (`NEO4J_server_memory_heap_initial__size`, `NEO4J_server_memory_heap_max__size`, `NEO4J_server_memory_pagecache_size`) and authentication (`NEO4J_AUTH`) will be imported.
    *   Persistent volumes for `/data` and `/logs` will be created automatically.

*   **For the `graphiti-mcp` service:**
    *   Railway will build the service from the `Dockerfile` in your repository.
    *   The `depends_on` condition ensures that this service will only start after the `neo4j` service is healthy.
    *   **Crucially, you will need to add the following environment variables in the Railway service settings for `graphiti-mcp`:**
        *   `NEO4J_URI`: This should be updated to point to the internal Railway address for the `neo4j` service. Railway typically provides service names as hostnames. So, it will likely be `bolt://neo4j:7687`.
        *   `NEO4J_USER`: `neo4j`
        *   `NEO4J_PASSWORD`: `demodemo`
        *   `OPENAI_API_KEY`: **You must provide your OpenAI API key here.**
        *   `MODEL_NAME`: `gpt-4.1-mini` (or your preferred model)
        *   `SEMAPHORE_LIMIT`: `10` (or your preferred limit)

**Step 3: Deployment and Networking**

1.  **Deploy**: Once the services are configured and the environment variables are set, you can trigger a deployment. Railway will build the `graphiti-mcp` image and start both services.
2.  **Expose the Service**: After the deployment is successful, navigate to the `graphiti-mcp` service settings in Railway. In the "Networking" section, generate a public URL for port `8000`. This will make your MCP server accessible over the internet.
