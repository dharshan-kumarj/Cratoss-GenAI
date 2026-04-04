# Cratoss Docker Setup Guide

This guide explains how to run the Cratoss application (Frontend + Backend RAG) using Docker and Docker Compose.

## Architecture

The application consists of three main services:

1. **Frontend** - React + Vite + TypeScript application (Port 80 in production, 5173 in dev)
2. **Backend** - Python FastAPI RAG service (Port 8000)
3. **Ollama** - Local LLM inference service (Port 11434)

## Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose 2.0 or higher
- (Optional) NVIDIA GPU + nvidia-docker for GPU acceleration

## Quick Start

### Production Deployment

1. **Build and start all services:**
   ```bash
   docker-compose up -d
   ```

2. **Pull Ollama model (first time only):**
   ```bash
   docker exec -it cratoss-ollama ollama pull llama3.2
   # or any other model you want to use
   ```

3. **Access the application:**
   - Frontend: http://localhost
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs
   - Ollama: http://localhost:11434

4. **View logs:**
   ```bash
   # All services
   docker-compose logs -f

   # Specific service
   docker-compose logs -f backend
   docker-compose logs -f frontend
   ```

5. **Stop services:**
   ```bash
   docker-compose down
   ```

### Development Mode

For development with hot reloading:

1. **Start development environment:**
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

2. **Access services:**
   - Frontend (Dev Server): http://localhost:5173
   - Backend (Hot Reload): http://localhost:8000
   - Ollama: http://localhost:11434

3. **Stop development environment:**
   ```bash
   docker-compose -f docker-compose.dev.yml down
   ```

## Service Details

### Frontend Service

**Production:**
- Built with multi-stage Dockerfile
- Served via Nginx
- Optimized static assets
- API proxy configured to backend

**Development:**
- Vite dev server with hot module replacement
- Source code mounted as volume
- Direct access to backend API

### Backend Service

**Configuration:**
- FastAPI application
- RAG pipeline with LangChain
- ChromaDB vector store
- Automatic model loading on startup

**Volumes:**
- `./Cratoss--RAG/data` - PDF documents for RAG
- `./Cratoss--RAG/vectorstore` - Vector database storage
- `backend_cache` - Python cache and models

### Ollama Service

**GPU Support:**
The docker-compose.yml includes GPU configuration for NVIDIA GPUs. If you don't have a GPU, modify the ollama service in docker-compose.yml:

```yaml
# Remove or comment out the deploy section:
# deploy:
#   resources:
#     reservations:
#       devices:
#         - driver: nvidia
#           count: all
#           capabilities: [gpu]
```

**Available Models:**
```bash
# List downloaded models
docker exec -it cratoss-ollama ollama list

# Pull a specific model
docker exec -it cratoss-ollama ollama pull llama3.2
docker exec -it cratoss-ollama ollama pull mistral

# Remove a model
docker exec -it cratoss-ollama ollama rm model-name
```

## Common Commands

### Build Services

```bash
# Build all services
docker-compose build

# Build specific service
docker-compose build backend
docker-compose build frontend

# Build without cache
docker-compose build --no-cache
```

### Container Management

```bash
# Start services
docker-compose up -d

# Restart a specific service
docker-compose restart backend

# Stop all services
docker-compose stop

# Remove all containers and networks
docker-compose down

# Remove all containers, networks, and volumes
docker-compose down -v
```

### Debugging

```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs -f backend

# Execute commands in running container
docker exec -it cratoss-backend bash
docker exec -it cratoss-frontend sh

# Inspect container
docker inspect cratoss-backend
```

## Environment Variables

Create a `.env` file in the project root to customize settings:

```env
# Backend
BACKEND_PORT=8000
OLLAMA_BASE_URL=http://ollama:11434

# Frontend
FRONTEND_PORT=80
VITE_API_URL=http://localhost:8000

# Ollama
OLLAMA_PORT=11434
```

## Data Persistence

The following data is persisted using Docker volumes:

- **Ollama models**: `cratoss-ollama-data`
- **Backend cache**: `cratoss-backend-cache`
- **PDF documents**: `./Cratoss--RAG/data` (bind mount)
- **Vector store**: `./Cratoss--RAG/vectorstore` (bind mount)

To backup volumes:
```bash
docker run --rm -v cratoss-ollama-data:/data -v $(pwd):/backup alpine tar czf /backup/ollama-backup.tar.gz -C /data .
```

## Troubleshooting

### Backend not starting
- Check if Ollama is healthy: `docker-compose ps`
- View backend logs: `docker-compose logs backend`
- Ensure the model is downloaded in Ollama

### Frontend can't reach backend
- Check if backend is running: `curl http://localhost:8000`
- Verify network connectivity: `docker-compose exec frontend ping backend`
- Check nginx configuration in `Cratoss_Frontend/nginx.conf`

### Out of memory
- Reduce Ollama model size (use smaller models)
- Increase Docker memory limits in Docker Desktop settings
- Use CPU-only mode if GPU memory is limited

### Rebuild from scratch
```bash
docker-compose down -v
docker system prune -a
docker-compose build --no-cache
docker-compose up -d
```

## Production Considerations

1. **Use environment variables** for sensitive configuration
2. **Set up reverse proxy** (nginx/traefik) with SSL/TLS
3. **Configure proper logging** and log rotation
4. **Monitor resource usage** (RAM, CPU, GPU)
5. **Regular backups** of volumes and data
6. **Security hardening** - update base images regularly
7. **Rate limiting** on API endpoints
8. **CORS configuration** for production domains

## File Structure

```
Cratoss/
├── docker-compose.yml           # Production orchestration
├── docker-compose.dev.yml       # Development orchestration
├── DOCKER.md                    # This file
├── Cratoss_Frontend/
│   ├── Dockerfile              # Production build
│   ├── Dockerfile.dev          # Development build
│   ├── nginx.conf              # Nginx configuration
│   └── .dockerignore           # Ignore patterns
└── Cratoss--RAG/
    ├── Dockerfile              # Backend image
    └── .dockerignore           # Ignore patterns
```

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Ollama Documentation](https://github.com/ollama/ollama)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Vite Documentation](https://vitejs.dev/)
