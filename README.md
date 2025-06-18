# NV-Ingest Pipeline POC Setup Guide

This README will guide you through setting up and running the NV-Ingest RAG pipeline demonstration, which shows how to ingest documents, export the vector database, and run RAG queries on a simulated destination computer.

## Prerequisites

### Environment Requirements

- **Windows with WSL2**: Windows 10/11 with Windows Subsystem for Linux 2
- **WSL**: Version 2.5.9.0 with Kernel 6.6.87.2-1
- **Ubuntu**: 22.04.5 LTS (jammy)
- **Docker Desktop for Windows**: Version 28.2.2, build e6534b4
- **Docker Compose**: Version v2.36.2-desktop.1
- **Python**: Version 3.10.12
- **Jupyter Notebook**: Version 7.4.3 (with JupyterLab 4.4.3)
- **NVIDIA API Key**: Valid NGC API key for accessing NVIDIA endpoints

### Hardware Requirements

- **GPU**: NVIDIA GPU with CUDA support (recommended)
- **RAM**: Minimum 16GB (32GB recommended)
- **Storage**: At least 10GB free space for containers and data

## Setup Instructions

### Step 1: Initial Directory Setup

Start in the directory where you want to install the POC:

```bash
# Navigate to your desired installation directory
cd /path/to/your/projects
```

### Step 2: Clone the NV-Ingest Repository

```bash
# Clone the official NVIDIA NV-Ingest repository
git clone https://github.com/NVIDIA/nv-ingest
cd nv-ingest
```

### Step 3: NVIDIA API Key Configuration

Configure your NVIDIA credentials using NGC CLI:

```bash
# Set up NGC configuration (follow prompts)
ngc config set

# Login to NVIDIA Container Registry
docker login nvcr.io
```

For detailed instructions on obtaining and configuring your NVIDIA API key, visit the [NVIDIA NGC documentation](https://docs.nvidia.com/ngc/ngc-overview/index.html).

### Step 4: Environment File Setup

Ensure your `.env` file is properly configured with your NVIDIA API keys:

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file to include your API keys
nano .env  # or use your preferred editor
```

Make sure your `.env` file includes:
```
NVIDIA_BUILD_API_KEY=your_nvidia_api_key_here
NGC_API_KEY=your_nvidia_api_key_here
```

### Step 5: Start Jupyter Notebook from WSL

```bash
# Start Jupyter notebook server from WSL
python3 -m notebook --no-browser --port=8888
```

Access Jupyter at: `http://localhost:8888` in your browser.

### Step 6: Load the Jupyter Notebooks

In Jupyter, load both notebooks:

1. **First Notebook**: `NV-Ingest-Pipeline-FINAL 18June25.ipynb` (Ingest pipeline)
2. **Second Notebook**: `NV-Ingest-Pipeline-Retrieval-FINAL 18June25.ipynb` (Retrieval pipeline)

## Running the Pipeline

### Phase 1: Document Ingestion

1. **Start the main pipeline containers:**
   ```bash
   docker-compose -f docker-compose.ingest.yml --profile retrieval up -d
   ```

2. **Source the environment file:**
   ```bash
   source .env
   ```

3. **Run the first notebook** (`NV-Ingest-Pipeline-FINAL 18June25.ipynb`)
   - This will ingest the pharmacopia document
   - Create embeddings and store in Milvus
   - Export the data to `nv_ingest_export.json` (See comments in code for detailed explanation)

### Phase 2: Destination Transfer and Retrieval

1. **Start the destination containers:**
   ```bash
   docker-compose -f docker-compose.destination.yml up -d
   ```

2. **Run the second notebook** (`NV-Ingest-Pipeline-Retrieval-FINAL 18June25.ipynb`)
   - This will import data to the destination Milvus (port 19540)
   - Demonstrate RAG queries on the destination

## Pipeline Overview and POC Demonstration 

The demonstration shows:

1. **Ingestion Pipeline** (First Notebook):
   - Document processing with NV-Ingest
   - Vector embedding creation
   - Storage in Milvus vector database
   - Data export for transfer

2. **Destination Pipeline** (Second Notebook):
   - Import exported data to destination Milvus
   - Lightweight RAG system operation on destination device
   - Query processing without full NV-Ingest infrastructure

## Stopping the Containers

When finished, stop all containers:

```bash
# Stop main pipeline containers
docker-compose -f docker-compose.ingest.yml --profile retrieval down

# Stop destination containers
docker-compose -f docker-compose.destination.yml down --remove-orphans
```

## Troubleshooting Tips

### Check Container Status

```bash
# View all running containers
docker ps

# Check specific container health
docker-compose -f docker-compose.ingest.yml --profile retrieval ps
docker-compose -f docker-compose.destination.yml ps
```

### View Container Logs

```bash
# Check NV-Ingest runtime logs for errors
docker logs nv-ingest-nv-ingest-ms-runtime-1 | grep -i error

# View recent logs (last 50 lines)
docker logs nv-ingest-nv-ingest-ms-runtime-1 --tail 50

# Check Milvus logs
docker logs milvus-standalone --tail 50

# Check destination Milvus logs
docker logs milvus-destination --tail 50
```

### Verify Environment Configuration

```bash
# Check if docker-compose is loading .env variables correctly
docker-compose -f docker-compose.ingest.yml config

# Verify destination compose configuration
docker-compose -f docker-compose.destination.yml config
```

### Common Issues and Solutions

1. **Connection Refused Errors**:
   - Check that containers are running: `docker ps`
   - Wait for containers to be healthy (can take 2-3 minutes)
   - Verify ports are not in use by other applications

2. **API Key Issues**:
   - Verify `.env` file contains correct API keys
   - Check that environment is sourced: `source .env`
   - Confirm NGC login: `docker login nvcr.io`

3. **Memory Issues**:
   - Ensure Docker Desktop has sufficient memory allocated (8GB+)
   - Close unnecessary applications
   - Monitor system resources: `docker stats`

4. **Network Issues**:
   - Check Docker network configuration
   - Restart Docker Desktop if needed
   - Verify WSL2 networking is working

5. **Container Startup Failures**:
   - Check logs for specific error messages
   - Verify all required volumes directories exist
   - Ensure proper permissions on mounted directories

### Port Information

- **Main Milvus**: `http://localhost:19530`
- **Destination Milvus**: `http://localhost:19540`
- **Redis**: `localhost:6379`
- **MinIO**: `localhost:9000` (API), `localhost:9001` (Console)
- **Destination MinIO**: `localhost:9010` (API), `localhost:9011` (Console)
- **NV-Ingest Runtime**: `localhost:7670`, `localhost:7671`

## Support

For additional support:
- Check the [NVIDIA NV-Ingest GitHub repository](https://github.com/NVIDIA/nv-ingest)
- Review the [NVIDIA NGC documentation](https://docs.nvidia.com/ngc/)
- Please consult Docker Desktop documentation for container issues

## Notes

- The demonstration uses the customer-provided pharmacopia document as sample data
- The destination setup simulates what would run on a Jetson device
- Vector embeddings are created using `nvidia/llama-3.2-nv-embedqa-1b-v2`
- Text generation uses `nvidia/llama-3.1-nemotron-70b-instruct`