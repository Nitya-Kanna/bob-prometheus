# Bob Prometheus Setup

A Prometheus monitoring setup with MCP (Model Context Protocol) integration for AI-powered metrics querying.

## Features

- Prometheus Server for metrics collection
- Node Exporter for system-level metrics (CPU, memory, disk, network)
- Prometheus MCP Server for AI assistant integration
- Docker-based deployment

## Prerequisites

- Docker (or Colima on macOS)
- Python 3.10+
- uv package manager

## Setup

### 1. Clone and Setup Virtual Environment

```bash
git clone https://github.com/Nitya-Kanna/bob-prometheus.git
cd bob-prometheus
python3 -m venv venv
source venv/bin/activate
```

### 2. Configure Environment Variables

Copy the example environment file and add your tokens:

```bash
cp .env.example .env
```

Edit `.env` and add your actual values:
- `GRAFANA_SERVICE_ACCOUNT_TOKEN` - Your Grafana service account token
- `GRAFANA_URL` - Grafana server URL (default: http://localhost:3000)
- `PROMETHEUS_URL` - Prometheus server URL (default: http://localhost:9090)

**Important:** Never commit the `.env` file to version control. It contains sensitive tokens.

### 3. Install Prometheus MCP Server

```bash
cd prometheus-mcp
uv pip install -e .
cd ..
```

### 4. Start Services

```bash
# Start Docker (or Colima on macOS)
colima start  # macOS only

# Start Prometheus
docker run -d --name prometheus \
  -p 9090:9090 \
  -v "$(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml" \
  prom/prometheus:latest

# Start Node Exporter
docker run -d --name node-exporter \
  -p 9100:9100 \
  prom/node-exporter:latest
```

### 5. Configure MCP Servers

Add to your MCP settings file (e.g., `.bob/settings/mcp_settings.json`):

```json
{
  "mcpServers": {
    "prometheus": {
      "command": "/path/to/bob-prometheus/venv/bin/python",
      "args": ["-m", "prometheus_mcp_server.main"],
      "env": {
        "PROMETHEUS_URL": "http://localhost:9090"
      }
    },
    "grafana": {
      "command": "uvx",
      "args": ["mcp-grafana"],
      "env": {
        "GRAFANA_URL": "http://localhost:3000",
        "GRAFANA_SERVICE_ACCOUNT_TOKEN": "your_token_here"
      }
    }
  }
}
```

**Security Note:** For production use, consider loading tokens from environment variables in your MCP settings instead of hardcoding them.

### 6. Start Grafana (Optional)

If you want to use Grafana for visualization:

```bash
docker run -d --name grafana \
  -p 3000:3000 \
  grafana/grafana:latest
```

Access Grafana at http://localhost:3000 (default credentials: admin/admin)

**Create a Service Account Token:**
1. Go to Administration → Service Accounts
2. Create a new service account with Admin role
3. Generate a token and save it to your `.env` file

## Available Metrics

### System Metrics (Node Exporter)
- CPU usage per core and by mode
- Memory usage (total, available, used)
- Disk I/O operations and throughput
- Network traffic and errors
- System load and uptime

### Prometheus Metrics
- Internal server metrics
- Scrape performance
- Query statistics

## Example Queries

**CPU Usage:**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

**Memory Usage:**
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

**Disk I/O Rate:**
```promql
rate(node_disk_read_bytes_total[5m])
```

## MCP Tools

The MCP server provides these tools for AI assistants:

- `health_check` - Check server status
- `execute_query` - Run instant PromQL queries
- `execute_range_query` - Run time-range queries
- `list_metrics` - Browse available metrics
- `get_metric_metadata` - Get metric descriptions
- `get_targets` - View scrape targets

## Web Interfaces

- Prometheus UI: http://localhost:9090
- Node Exporter: http://localhost:9100/metrics

## Project Structure

```
bob-prometheus/
├── README.md
├── .gitignore
├── prometheus.yml
├── venv/
└── prometheus-mcp/
```

## Stopping Services

```bash
docker stop prometheus node-exporter
docker rm prometheus node-exporter
colima stop  # macOS only
```

## Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Prometheus MCP Server](https://github.com/pab1it0/prometheus-mcp-server)
- [Model Context Protocol](https://modelcontextprotocol.io)

## License

MIT