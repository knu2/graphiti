
# Deployment Plan: MCP Server on Railway

This deployment plan outlines the process for deploying the `graphiti-mcp` server on Railway, using a custom `Dockerfile` and integrating with Neo4j AuraDB for the database component. This approach avoids Railway's limited `docker-compose.yml` support and leverages a managed Neo4j instance for simplicity and scalability.

**Here are the detailed steps:**

## Step 1: Fork the Repository

1. **Fork the `getzep/graphiti` repository** on GitHub. This creates a copy of the project under your own GitHub account.

## Step 2: Project Setup on Railway

1. **Create a New Project**: Start by creating a new, empty project on Railway.
2. **Connect to GitHub**: Link the Railway project to your forked repository. Railway will detect the `Dockerfile` in the `mcp_server/` directory when configured properly.
3. **Add Service**: Create a new service named `graphiti-mcp` and connect it to your GitHub repository.

## Step 3: Configure the Service

Railway will build the `graphiti-mcp` service from the `mcp_server/Dockerfile`. You’ll need to configure the build and deployment settings.

1. **Set Build Configuration**:
   - In the Railway dashboard, go to the `graphiti-mcp` service’s **Settings** tab.
   - Set the **Dockerfile Path** to `mcp_server/Dockerfile`.
   - Ensure the **Root Directory** is set to `/` (the repository root).

2. **Update `railway.toml`**:
   - Create or update `railway.toml` in the root directory of your repository with the following content:
     ```toml
     [build]
     builder = "dockerfile"
     dockerfilePath = "mcp_server/Dockerfile"

     [deploy]
     startCommand = "uv run graphiti_mcp_server.py --transport sse"
     port = 8000
     ```
   - Commit and push this file to your repository:
     ```bash
     git add railway.toml
     git commit -m "Add railway.toml for graphiti-mcp service"
     git push origin main
     ```

3. **Set Environment Variables**:
   - In the Railway dashboard, go to the `graphiti-mcp` service’s **Variables** tab and add or verify the following:
     - `NEO4J_URI`: `bolt://<aura-instance>.databases.neo4j.io:7687` (replace `<aura-instance>` with your Neo4j AuraDB instance name from the AuraDB dashboard).
     - `NEO4J_USER`: `neo4j` (or your AuraDB username).
     - `NEO4J_PASSWORD`: Your AuraDB password (e.g., `demodemo`).
     - `OPENAI_API_KEY`: Your OpenAI API key (obtain from [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)).
     - `MODEL_NAME`: `gpt-4.1-mini` (or your preferred model).
     - `SMALL_MODEL_NAME`: `gpt-4.1-mini` (optional, for smaller LLM operations).
     - `LLM_TEMPERATURE`: `0.1` (optional).
     - `SEMAPHORE_LIMIT`: `10`.
     - `PORT`: `8000`.
     - `MCP_SERVER_HOST`: `0.0.0.0`.

## Step 4: Deploy the Service

1. **Trigger Deployment**:
   - In the Railway dashboard, go to the `graphiti-mcp` service’s **Deployments** tab and click **Redeploy**.
   - Alternatively, use the Railway CLI:
     ```bash
     railway up
     ```

2. **Verify Build and Deploy Logs**:
   - Check the **Build Logs** to ensure the `mcp_server/Dockerfile` builds successfully without errors (e.g., missing files or permission issues).
   - Check the **Deploy Logs** to confirm the service starts with `uv run graphiti_mcp_server.py --transport sse`.
   - Address any Neo4j or OpenAI-related errors by verifying environment variables.

3. **Expose the Service**:
   - After successful deployment, go to the `graphiti-mcp` service’s **Settings** tab.
   - In the "Networking" section, generate a public URL for port `8000`. This will make your MCP server accessible (e.g., `https://graphiti-mcp-production.up.railway.app:8000`).

## Step 5: Local Testing and Development

To test and improve the MCP server locally before deploying changes:

1. **Build Locally**:
   - Navigate to the root of your repository:
     ```bash
     cd /path/to/your/repository
     ```
   - Build the Docker image:
     ```bash
     docker build -f mcp_server/Dockerfile -t graphiti-mcp .
     ```

2. **Run Locally with Docker**:
   - Run the container with environment variables:
     ```bash
     docker run -p 8000:8000 \
       -e NEO4J_URI=bolt://<aura-instance>.databases.neo4j.io:7687 \
       -e NEO4J_USER=neo4j \
       -e NEO4J_PASSWORD=<your-aura-password> \
       -e OPENAI_API_KEY=<your-openai-api-key> \
       -e MODEL_NAME=gpt-4.1-mini \
       -e SMALL_MODEL_NAME=gpt-4.1-mini \
       -e LLM_TEMPERATURE=0.1 \
       -e SEMAPHORE_LIMIT=10 \
       -e PORT=8000 \
       -e MCP_SERVER_HOST=0.0.0.0 \
       graphiti-mcp
     ```
   - Test at `http://localhost:8000` or with an MCP client.

3. **Run Locally Without Docker**:
   - Navigate to `mcp_server/`:
     ```bash
     cd mcp_server
     ```
   - Install dependencies:
     ```bash
     pip install uv
     uv sync --frozen --no-dev
     ```
   - Set environment variables:
     ```bash
     export NEO4J_URI=bolt://<aura-instance>.databases.neo4j.io:7687
     export NEO4J_USER=neo4j
     export NEO4J_PASSWORD=<your-aura-password>
     export OPENAI_API_KEY=<your-openai-api-key>
     export MODEL_NAME=gpt-4.1-mini
     export SMALL_MODEL_NAME=gpt-4.1-mini
     export LLM_TEMPERATURE=0.1
     export SEMAPHORE_LIMIT=10
     export PORT=8000
     export MCP_SERVER_HOST=0.0.0.0
     ```
   - Run the server:
     ```bash
     uv run graphiti_mcp_server.py --transport sse
     ```
   - Test at `http://localhost:8000`.

4. **Develop Improvements**:
   - Edit `mcp_server/graphiti_mcp_server.py`, rebuild/run, and test locally.
   - Commit changes and redeploy with `railway up`.

## Step 6: Troubleshooting

- **Build Failures**: Check for missing files (e.g., `mcp_server/pyproject.toml`, `mcp_server/uv.lock`, `mcp_server/graphiti_mcp_server.py`) or permission errors. Ensure files are committed to GitHub.
- **Deployment Errors**: Verify Neo4j AuraDB credentials and OpenAI API key in the **Variables** tab. Check logs for connection issues.
- **Local Issues**: Test Neo4j with `cypher-shell -a $NEO4J_URI -u $NEO4J_USER -p $NEO4J_PASSWORD` and OpenAI with a Python script.

## Notes
- **Neo4j AuraDB**: Replaces the local Neo4j service from `docker-compose.yml`. Sign up at [https://neo4j.com/cloud/aura/](https://neo4j.com/cloud/aura/) for a managed instance.
- **Repository Structure**:
  ```
  ├── railway.toml
  ├── Dockerfile
  ├── pyproject.toml
  ├── uv.lock
  ├── README.md
  ├── graphiti_core/
  ├── mcp_server/
  │   ├── Dockerfile
  │   ├── pyproject.toml
  │   ├── uv.lock
  │   ├── graphiti_mcp_server.py
  ├── server/
  │   ├── pyproject.toml
  │   ├── uv.lock
  │   ├── README.md
  │   ├── graph_service/
  ```
- **Cache Mount**: Optionally reintroduce in `mcp_server/Dockerfile` for faster builds after testing:
  ```dockerfile
  RUN mkdir -p /app/.cache/uv && chmod -R u+rw /app/.cache
  RUN --mount=type=cache,id=s/9a6e0da9-4875-4d16-99b9-338347609f5f-/app/.cache/uv,target=/app/.cache/uv \
      uv sync --frozen --no-dev
  ```
```

