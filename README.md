# Blue/Green Deployment with Nginx Auto-Failover

This project implements a Blue/Green deployment strategy using Nginx with automatic failover capabilities. When the primary service fails, Nginx automatically routes traffic to the backup service with zero client-visible downtime.

## Prerequisites

- Docker
- Docker Compose

## Quick Start

1. **Clone the repository**
```bash
git clone https://github.com/EmmyAnieDev/hng13-stage2-devops.git
cd hng13-stage2-devops
```

2. **Configure environment variables**

Copy `.env.example` to `.env` and modify as needed.:
```bash
cp .env.example .env
```

3. **Start services**
```bash
docker-compose up -d
```

4. **Verify deployment**
```bash
curl http://localhost:8080/version
```

## Service Endpoints

- **Nginx (Public)**: `http://localhost:8080`
- **Blue (Direct)**: `http://localhost:8081`
- **Green (Direct)**: `http://localhost:8082`

## Testing Failover

1. **Check current active pool**
```bash
curl -i http://localhost:8080/version
# Returns: X-App-Pool: blue
```

2. **Trigger chaos on Blue**
```bash
curl -X POST "http://localhost:8081/chaos/start?mode=error"
```

3. **Verify automatic failover to Green**
```bash
curl -i http://localhost:8080/version
# Returns: X-App-Pool: green
```

4. **Stop chaos and verify failback**
```bash
curl -X POST "http://localhost:8081/chaos/stop"
curl -i http://localhost:8080/version
# Returns: X-App-Pool: blue
```

## Architecture

- **Blue Service**: Primary application instance
- **Green Service**: Backup application instance
- **Nginx**: Load balancer with upstream backup configuration
  - Primary: Blue (default)
  - Backup: Green (auto-activates on Blue failure)
  - Fast failure detection (2s timeouts)
  - Automatic retry on errors (500, 502, 503, 504, timeout)

## Configuration Files

- `.env` - Environment variables
- `docker-compose.yml` - Service orchestration
- `nginx/conf.d/default.conf.template` - Nginx configuration template

## Stopping Services
```bash
docker-compose down
```

## Notes

- All client requests return 200 OK during failover
- Headers `X-App-Pool` and `X-Release-Id` identify the active service
- Failover happens automatically within 2-5 seconds
- No manual intervention required for service recovery