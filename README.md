# WordPress Docker

Docker setup for running WordPress behind a reverse proxy (like Nginx Proxy Manager).

## Requirements

- Docker
- Docker Compose

## Quick Start

1. Copy the example env file:
   ```
   cp .env-example .env
   ```

2. Edit `.env` and set your database passwords:
   ```
   MYSQL_ROOT_PASSWORD=your_secure_password
   MYSQL_PASSWORD=your_secure_password
   ```

3. Start the containers:
   ```
   docker-compose up -d --build
   ```

4. Open http://localhost:8699 in your browser and complete the WordPress setup.

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| APP_PORT | 8699 | Port to access WordPress |
| APP_HOST | 127.0.0.1 | Set to `0.0.0.0` to expose to network |
| MYSQL_ROOT_PASSWORD | - | MySQL root password |
| MYSQL_USER | - | MySQL user for WordPress |
| MYSQL_PASSWORD | - | MySQL password for WordPress user |
| MYSQL_DB_NAME | wordpress | Database name |
| MYSQL_PORT | 3306 | MySQL port |
| MYSQL_TZ | "Asia/Kolkata" | MySQL timezone |

### Network Access

By default, WordPress is only accessible from localhost. To expose it to your local network:

1. Set `APP_HOST=0.0.0.0` in `.env`
2. Restart: `docker-compose up -d`

Works over HTTP for local network access. When behind a reverse proxy with SSL, WordPress automatically detects HTTPS via the forwarded headers.

### Redis Object Cache

Redis is included for object caching. To enable:

1. Install the "Redis Object Cache" plugin in WordPress
2. Go to Settings > Redis
3. Click "Enable Object Cache"

The connection is pre-configured via environment variables.

### File Uploads

Default upload limits are set in `uploads.ini`:
- `upload_max_filesize = 50M`
- `post_max_size = 50M`
- `max_execution_time = 600`

Adjust as needed.

## Reverse Proxy Setup

This setup is designed to run behind Nginx Proxy Manager or similar:

1. Point your domain to the server running NPM
2. In NPM, create a proxy host:
   - Domain: yourdomain.com
   - Forward Hostname: your server IP (or host IP if NPM is in Docker)
   - Forward Port: 8699 (or your APP_PORT)
   - Enable SSL via Let's Encrypt

The nginx config already includes:
- Real IP forwarding from proxy
- Security headers
- Gzip compression
- Static file caching

## Directory Structure

```
.
├── docker-compose.yml
├── Dockerfile.wordpress    # WordPress + Redis extension
├── nginx-conf/
│   └── nginx.conf
├── uploads.ini             # PHP upload settings
├── data/
│   ├── dbdata/            # MySQL data
│   ├── redis/             # Redis data
│   └── wordpress/         # WordPress files
└── .env                   # Your configuration
```

## Common Commands

```
# Start
docker-compose up -d --build

# Stop
docker-compose down

# View logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f wordpress

# Restart a service
docker-compose restart webserver

# Rebuild WordPress image
docker-compose build wordpress
docker-compose up -d
```

## Backups

To backup WordPress files and database:

```
# Backup database
docker exec db mysqldump -u root -p${MYSQL_ROOT_PASSWORD} wordpress > backup.sql

# Backup files
tar -czf wordpress-files.tar.gz ./data/wordpress
```

## Troubleshooting

**Container won't start:**
- Check if `.env` file exists and has all required variables
- Check logs: `docker-compose logs`

**Can't connect to database:**
- Verify MYSQL_* variables in `.env`
- Wait a few seconds for MySQL to initialize

**Redis not working:**
- Make sure Redis container is running: `docker-compose ps`
- Check if PHP Redis extension is installed: `docker exec wordpress php -m | grep redis`

**Permission issues:**
- The WordPress files in `./data/wordpress` are owned by the container user
- You may need to adjust permissions if editing files directly on the host
