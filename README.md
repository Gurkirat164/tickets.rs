# Tickets.rs 🎫

A high-performance, scalable Discord bot infrastructure written in Rust for managing support tickets and bot services. This project provides a complete microservices architecture for running a Discord bot with advanced features including ticket management, Patreon integration, voting systems, and whitelabel support.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Services](#services)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Environment Variables](#environment-variables)
- [Development](#development)
- [Building](#building)
- [Docker Deployment](#docker-deployment)
- [Database Setup](#database-setup)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🔍 Overview

Tickets.rs is a monorepo containing multiple Rust-based microservices that work together to provide a comprehensive Discord bot infrastructure. The project is designed for high availability, scalability, and performance, utilizing modern technologies like:

- **Rust** for high-performance, memory-safe code
- **Tokio** for async runtime
- **PostgreSQL** for persistent storage
- **Redis** for caching and session management
- **Kafka** for event streaming
- **Docker** for containerization

## 🏗️ Architecture

The project follows a microservices architecture with the following components:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Discord API   │────▶│     Sharder     │────▶│   Event Stream  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │                          │
                               ▼                          ▼
                        ┌─────────────┐          ┌──────────────┐
                        │    Cache    │          │ HTTP Gateway │
                        └─────────────┘          └──────────────┘
                               │                          │
                               ▼                          ▼
                        ┌─────────────┐          ┌──────────────┐
                        │  PostgreSQL │          │  Worker SVC  │
                        └─────────────┘          └──────────────┘
```

## 🧩 Services

### Core Services

#### 1. **Sharder**
- Manages Discord gateway connections and sharding
- Handles WebSocket connections to Discord
- Supports both public bot and whitelabel instances
- Location: `sharder/`

#### 2. **Cache**
- High-performance caching layer
- Supports PostgreSQL and in-memory storage
- Caches Discord entities (guilds, channels, users)
- Location: `cache/`

#### 3. **HTTP Gateway**
- Handles Discord interactions via HTTP
- Processes slash commands and interactions
- Validates Discord signatures
- Location: `http-gateway/`

#### 4. **Event Stream**
- Kafka-based event streaming
- Publishes and consumes Discord events
- Enables microservice communication
- Location: `event-stream/`

### Supporting Services

#### 5. **Cache Sync Service**
- Synchronizes cache across instances
- Maintains cache consistency
- Location: `cache-sync-service/`

#### 6. **Database**
- Database abstraction layer
- Handles whitelabel bot management
- Manages guild configurations
- Location: `database/`

#### 7. **Vote Listener**
- Receives voting webhooks from bot lists
- Processes user votes
- Updates vote statistics
- Location: `vote_listener/`

#### 8. **Bot List Updater**
- Updates bot statistics on listing sites
- Supports Discord Bot List, Discord Boats
- Scheduled updates
- Location: `bot-list-updater/`

#### 9. **Server Counter**
- Provides server count metrics
- Exposes Prometheus metrics
- REST API for statistics
- Location: `server-counter/`

#### 10. **Stats Channel Updater**
- Updates Discord channel with bot statistics
- Shows total server count in channel name
- Location: `stats-channel-updater/`

#### 11. **Patreon Proxy**
- Handles Patreon OAuth integration
- Manages patron benefits
- Syncs patron data
- Location: `patreon-proxy/`

#### 12. **Image Proxy**
- Proxies and caches Discord images
- Optimizes image delivery
- Location: `image-proxy/`

#### 13. **Secure Proxy**
- Authenticated reverse proxy
- Secures internal service communication
- Location: `secure-proxy/`

#### 14. **Global Resolver**
- Resolves global rate limits
- Coordinates API calls across shards
- Location: `global-resolver/`

### Libraries

#### 15. **Model**
- Discord data models
- Shared type definitions
- Location: `model/`

#### 16. **Common**
- Shared utilities and helpers
- Prometheus server
- Event forwarding utilities
- Location: `common/`

## 📦 Prerequisites

### System Requirements

- **Operating System**: Linux (recommended), macOS, or Windows
- **RAM**: Minimum 2GB, recommended 4GB+
- **CPU**: Multi-core processor recommended
- **Storage**: At least 10GB free space

### Required Software

1. **Rust** (latest stable version)
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. **PostgreSQL** (version 12+)
   - Download from: https://www.postgresql.org/download/

3. **Redis** (version 6+)
   - Download from: https://redis.io/download

4. **Apache Kafka** (version 2.8+)
   - Download from: https://kafka.apache.org/downloads

5. **Docker** (optional, for containerized deployment)
   - Download from: https://www.docker.com/get-started

6. **Docker Compose** (optional)
   - Usually included with Docker Desktop

### Discord Requirements

- Discord Bot Application
- Bot Token from Discord Developer Portal
- Public Key for interaction verification
- Bot invited to at least one server

### External Services (Optional)

- **Sentry**: For error tracking and monitoring
- **Patreon**: For patron integration
- **Discord Bot List**: For bot listing
- **Discord Boats**: For bot listing

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/tickets.rs.git
cd tickets.rs
```

### 2. Install Rust Dependencies

```bash
# Update Rust to the latest version
rustup update

# Install cargo-watch for development (optional)
cargo install cargo-watch
```

### 3. Install System Dependencies

#### Ubuntu/Debian
```bash
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libssl-dev cmake
```

#### macOS
```bash
brew install cmake openssl
```

#### Windows
- Install Visual Studio Build Tools
- Install OpenSSL from: https://slproweb.com/products/Win32OpenSSL.html

### 4. Set Up PostgreSQL

```bash
# Start PostgreSQL service
sudo systemctl start postgresql

# Create database
sudo -u postgres psql
CREATE DATABASE tickets;
CREATE USER tickets_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE tickets TO tickets_user;
\q
```

### 5. Set Up Redis

```bash
# Start Redis service
sudo systemctl start redis

# Or using Docker
docker run -d -p 6379:6379 redis:latest
```

### 6. Set Up Kafka

```bash
# Using Docker Compose is easiest
docker-compose up -d zookeeper kafka
```

Or manually:
```bash
# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka
bin/kafka-server-start.sh config/server.properties

# Create topics
bin/kafka-topics.sh --create --topic tickets-events --bootstrap-server localhost:9092
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the root directory or set environment variables for each service:

#### Sharder (Public Bot)

```env
# Required
SHARDER_ID=0
SHARDER_TOTAL=1
REDIS_ADDR=localhost:6379
REDIS_PASSWORD=
REDIS_THREADS=4
WORKER_SVC_URI=http://localhost:8080
SENTRY_DSN=your_sentry_dsn
KAFKA_BROKERS=localhost:9092
KAFKA_TOPIC=tickets-events

# Public Bot Specific
SHARDER_TOKEN=your_discord_bot_token
SHARDER_CLUSTER_SIZE=1
BOT_ID=your_bot_id
LARGE_SHARDING_BUCKETS=16

# Metrics (optional)
METRICS_ADDR=0.0.0.0:9091
```

#### Sharder (Whitelabel)

```env
# Required (same as above)
SHARDER_ID=0
SHARDER_TOTAL=1
REDIS_ADDR=localhost:6379
REDIS_PASSWORD=
REDIS_THREADS=4
WORKER_SVC_URI=http://localhost:8080
SENTRY_DSN=your_sentry_dsn
KAFKA_BROKERS=localhost:9092
KAFKA_TOPIC=tickets-events

# Whitelabel Specific
DATABASE_URI=postgresql://tickets_user:your_password@localhost/tickets
DATABASE_THREADS=4
```

#### HTTP Gateway

```env
SERVER_ADDR=0.0.0.0:8080
PUBLIC_BOT_ID=your_bot_id
PUBLIC_TOKEN=your_discord_bot_token
PUBLIC_PUBLIC_KEY=your_public_key_hex

DATABASE_URI=postgresql://tickets_user:your_password@localhost/tickets
DATABASE_THREADS=4

CACHE_URI=http://localhost:7878
CACHE_THREADS=4

WORKER_SVC_URI=worker-service:8081
SHARD_COUNT=1
```

#### Cache Service

```env
SERVER_ADDR=0.0.0.0:7878
DATABASE_URI=postgresql://tickets_user:your_password@localhost/tickets
DATABASE_THREADS=4
```

#### Vote Listener

```env
SERVER_ADDR=0.0.0.0:8082
DATABASE_URI=postgresql://tickets_user:your_password@localhost/tickets
DBL_TOKEN=your_discord_bot_list_token
VOTE_URL=https://example.com/vote
RUST_LOG=info
```

#### Bot List Updater

```env
DELAY=300  # seconds
BOT_ID=your_bot_id
DBL_TOKEN=your_discord_bot_list_token
DBOATS_TOKEN=your_discord_boats_token
```

#### Server Counter

```env
SERVER_ADDR=0.0.0.0:8083
CACHE_URI=http://localhost:7878
```

#### Stats Channel Updater

```env
SERVER_COUNTER_URL=http://localhost:8083/total
DISCORD_TOKEN=your_discord_bot_token
CHANNEL_ID=your_channel_id
```

#### Patreon Proxy

```env
PATREON_CAMPAIGN_ID=your_campaign_id
PATREON_CLIENT_ID=your_client_id
PATREON_CLIENT_SECRET=your_client_secret
PATREON_REDIRECT_URI=https://example.com/callback
SERVER_ADDR=0.0.0.0:8084
DATABASE_URI=postgresql://tickets_user:your_password@localhost/tickets
```

#### Secure Proxy

```env
SERVER_ADDR=0.0.0.0:8085
WORKER_URL=http://worker:8081
AUTH_HEADER_NAME=X-API-Key
AUTH_HEADER_VALUE=your_secret_key
```

## 🛠️ Development

### Running Individual Services

```bash
# Run the sharder (public bot)
cargo run --bin public

# Run the sharder (whitelabel)
cargo run --bin whitelabel --features whitelabel

# Run the HTTP gateway
cd http-gateway && cargo run

# Run the cache service
cd cache && cargo run --bin cache

# Run with auto-reload (requires cargo-watch)
cargo watch -x 'run --bin public'
```

### Running Tests

```bash
# Run all tests
cargo test --workspace

# Run tests for specific service
cd sharder && cargo test

# Run with output
cargo test -- --nocapture
```

### Database Migrations

```bash
# Install sqlx-cli
cargo install sqlx-cli --no-default-features --features postgres

# Run migrations
sqlx migrate run --database-url postgresql://tickets_user:your_password@localhost/tickets
```

## 🔨 Building

### Build All Services

```bash
# Debug build
cargo build --workspace

# Release build (optimized)
cargo build --workspace --release
```

### Build Specific Service

```bash
# Build sharder
cargo build --release --bin public

# Build HTTP gateway
cd http-gateway && cargo build --release

# Build with features
cargo build --release --bin whitelabel --features whitelabel
```

### Build Output

Binaries will be located in:
- Debug: `target/debug/`
- Release: `target/release/`

## 🐳 Docker Deployment

### Using Docker Compose (Recommended)

Create a `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: tickets
      POSTGRES_USER: tickets_user
      POSTGRES_PASSWORD: your_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --requirepass your_redis_password

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    ports:
      - "9092:9092"

  cache:
    build:
      context: .
      dockerfile: cache/Dockerfile
    environment:
      SERVER_ADDR: 0.0.0.0:7878
      DATABASE_URI: postgresql://tickets_user:your_password@postgres/tickets
      DATABASE_THREADS: 4
    depends_on:
      - postgres
    ports:
      - "7878:7878"

  sharder:
    build:
      context: .
      dockerfile: sharder/main.Dockerfile
    environment:
      SHARDER_ID: 0
      SHARDER_TOTAL: 1
      SHARDER_TOKEN: ${DISCORD_TOKEN}
      SHARDER_CLUSTER_SIZE: 1
      BOT_ID: ${BOT_ID}
      REDIS_ADDR: redis:6379
      REDIS_PASSWORD: your_redis_password
      REDIS_THREADS: 4
      WORKER_SVC_URI: http://worker:8081
      SENTRY_DSN: ${SENTRY_DSN}
      KAFKA_BROKERS: kafka:9092
      KAFKA_TOPIC: tickets-events
      LARGE_SHARDING_BUCKETS: 16
    depends_on:
      - redis
      - kafka
    restart: unless-stopped

  http-gateway:
    build:
      context: .
      dockerfile: http-gateway/Dockerfile
    environment:
      SERVER_ADDR: 0.0.0.0:8080
      PUBLIC_BOT_ID: ${BOT_ID}
      PUBLIC_TOKEN: ${DISCORD_TOKEN}
      PUBLIC_PUBLIC_KEY: ${PUBLIC_KEY}
      DATABASE_URI: postgresql://tickets_user:your_password@postgres/tickets
      DATABASE_THREADS: 4
      CACHE_URI: http://cache:7878
      CACHE_THREADS: 4
      WORKER_SVC_URI: worker:8081
      SHARD_COUNT: 1
    depends_on:
      - postgres
      - cache
    ports:
      - "8080:8080"

  vote-listener:
    build:
      context: .
      dockerfile: vote_listener/Dockerfile
    environment:
      SERVER_ADDR: 0.0.0.0:8082
      DATABASE_URI: postgresql://tickets_user:your_password@postgres/tickets
      DBL_TOKEN: ${DBL_TOKEN}
      VOTE_URL: https://yoursite.com/vote
      RUST_LOG: info
    depends_on:
      - postgres
    ports:
      - "8082:8082"

  bot-list-updater:
    build:
      context: .
      dockerfile: bot-list-updater/Dockerfile
    environment:
      DELAY: 300
      BOT_ID: ${BOT_ID}
      DBL_TOKEN: ${DBL_TOKEN}
      DBOATS_TOKEN: ${DBOATS_TOKEN}
    depends_on:
      - cache

  server-counter:
    build:
      context: .
      dockerfile: server-counter/Dockerfile
    environment:
      SERVER_ADDR: 0.0.0.0:8083
      CACHE_URI: http://cache:7878
    depends_on:
      - cache
    ports:
      - "8083:8083"

  patreon-proxy:
    build:
      context: .
      dockerfile: patreon-proxy/Dockerfile
    environment:
      PATREON_CAMPAIGN_ID: ${PATREON_CAMPAIGN_ID}
      PATREON_CLIENT_ID: ${PATREON_CLIENT_ID}
      PATREON_CLIENT_SECRET: ${PATREON_CLIENT_SECRET}
      PATREON_REDIRECT_URI: ${PATREON_REDIRECT_URI}
      SERVER_ADDR: 0.0.0.0:8084
      DATABASE_URI: postgresql://tickets_user:your_password@postgres/tickets
    depends_on:
      - postgres
    ports:
      - "8084:8084"

volumes:
  postgres_data:
```

### Deploy with Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build
```

### Building Individual Docker Images

```bash
# Build sharder
docker build -f sharder/main.Dockerfile -t tickets-sharder .

# Build cache
docker build -f cache/Dockerfile -t tickets-cache .

# Build HTTP gateway
docker build -f http-gateway/Dockerfile -t tickets-http-gateway .

# Run individual container
docker run -d --name sharder --env-file .env tickets-sharder
```

## 💾 Database Setup

### Initial Setup

```sql
-- Create database
CREATE DATABASE tickets;

-- Create user
CREATE USER tickets_user WITH PASSWORD 'your_secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE tickets TO tickets_user;

-- Connect to database
\c tickets

-- Create whitelabel tables (example)
CREATE TABLE whitelabel_bots (
    bot_id BIGINT PRIMARY KEY,
    token TEXT NOT NULL,
    public_key TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE whitelabel_status (
    bot_id BIGINT PRIMARY KEY,
    status TEXT NOT NULL,
    status_type INTEGER NOT NULL,
    FOREIGN KEY (bot_id) REFERENCES whitelabel_bots(bot_id)
);

CREATE TABLE whitelabel_guilds (
    bot_id BIGINT NOT NULL,
    guild_id BIGINT NOT NULL,
    PRIMARY KEY (bot_id, guild_id),
    FOREIGN KEY (bot_id) REFERENCES whitelabel_bots(bot_id)
);
```

### Run Migrations

If using sqlx migrations:

```bash
# Create migration
sqlx migrate add initial_schema

# Run migrations
sqlx migrate run --database-url postgresql://tickets_user:your_password@localhost/tickets

# Revert last migration
sqlx migrate revert --database-url postgresql://tickets_user:your_password@localhost/tickets
```

## 🚀 Deployment

### Production Deployment Checklist

- [ ] Set strong passwords for all services
- [ ] Enable SSL/TLS for PostgreSQL and Redis
- [ ] Configure firewall rules
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Configure Sentry for error tracking
- [ ] Set up log aggregation
- [ ] Configure automated backups
- [ ] Set up health checks
- [ ] Configure auto-restart policies
- [ ] Use environment-specific configurations
- [ ] Secure API keys and tokens
- [ ] Set up rate limiting
- [ ] Configure CORS properly
- [ ] Enable production logging levels

### Hosting Options

#### 1. **Self-Hosted (VPS/Dedicated Server)**
- Providers: DigitalOcean, Linode, Vultr, Hetzner
- Minimum specs: 4GB RAM, 2 CPU cores, 50GB SSD
- Install Docker and Docker Compose
- Deploy using docker-compose

#### 2. **Cloud Platforms**

**AWS**
- Use ECS for container orchestration
- RDS for PostgreSQL
- ElastiCache for Redis
- MSK for Kafka

**Google Cloud Platform**
- Use GKE for Kubernetes deployment
- Cloud SQL for PostgreSQL
- Memorystore for Redis
- Pub/Sub (alternative to Kafka)

**Azure**
- Use AKS for Kubernetes
- Azure Database for PostgreSQL
- Azure Cache for Redis
- Event Hubs (alternative to Kafka)

#### 3. **Kubernetes Deployment**

Create Kubernetes manifests in `k8s/` directory and deploy:

```bash
kubectl apply -f k8s/
```

### Reverse Proxy Setup (Nginx)

```nginx
# /etc/nginx/sites-available/tickets-bot

server {
    listen 80;
    server_name yourdomain.com;

    location /interactions {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /vote {
        proxy_pass http://localhost:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /patreon {
        proxy_pass http://localhost:8084;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Systemd Service Files

Create service files for each component:

```ini
# /etc/systemd/system/tickets-sharder.service

[Unit]
Description=Tickets.rs Sharder
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=tickets
WorkingDirectory=/opt/tickets.rs
EnvironmentFile=/opt/tickets.rs/.env
ExecStart=/opt/tickets.rs/target/release/public
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable tickets-sharder
sudo systemctl start tickets-sharder
sudo systemctl status tickets-sharder
```

## 🐛 Troubleshooting

### Common Issues

#### 1. Connection Refused Errors

```
Error: Connection refused (os error 111)
```

**Solution**: Ensure all dependent services (PostgreSQL, Redis, Kafka) are running:

```bash
# Check PostgreSQL
sudo systemctl status postgresql

# Check Redis
redis-cli ping

# Check Kafka
docker ps | grep kafka
```

#### 2. Discord Authentication Failed

```
Error: AuthenticationError
```

**Solution**: 
- Verify your bot token is correct
- Ensure bot has proper intents enabled in Discord Developer Portal
- Check if token hasn't been regenerated

#### 3. Database Connection Errors

```
Error: password authentication failed for user
```

**Solution**:
- Check DATABASE_URI format
- Verify PostgreSQL user credentials
- Ensure database exists
- Check pg_hba.conf for connection permissions

#### 4. Out of Memory

**Solution**:
- Increase system RAM
- Reduce REDIS_THREADS and DATABASE_THREADS
- Enable swap space
- Optimize query performance

#### 5. High CPU Usage

**Solution**:
- Check for infinite loops in logs
- Reduce number of shards if possible
- Optimize event processing
- Enable caching properly

### Logs

View service logs:

```bash
# Docker Compose
docker-compose logs -f service_name

# Systemd
sudo journalctl -u tickets-sharder -f

# Direct binary
RUST_LOG=debug ./target/release/public
```

### Debug Mode

Enable debug logging:

```env
RUST_LOG=debug
```

Or for specific modules:

```env
RUST_LOG=tickets_sharder=debug,tickets_cache=info
```

## 📊 Monitoring

### Prometheus Metrics

Most services expose Prometheus metrics:

- Sharder: `http://localhost:9091/metrics`
- Server Counter: `http://localhost:8083/metrics`

### Health Checks

Implement health check endpoints:

```bash
# Check if service is responding
curl http://localhost:8080/health
```

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Style

- Follow Rust standard formatting (`cargo fmt`)
- Run clippy before committing (`cargo clippy`)
- Write tests for new features
- Update documentation

## 📝 License

This project is licensed under the terms specified in the repository.

## 🔗 Links

- Discord Developer Portal: https://discord.com/developers/applications
- Rust Documentation: https://doc.rust-lang.org/
- Tokio Documentation: https://tokio.rs/
- Discord API Documentation: https://discord.com/developers/docs

## 💬 Support

For issues and questions:
- Open an issue on GitHub
- Check existing documentation
- Review Discord API documentation

---

**Note**: This is a complex distributed system. Take time to understand each component before deploying to production. Always test in a development environment first.
