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

### 2. Install Prometheus MCP Server

```bash
cd prometheus-mcp
uv pip install -e .
cd ..
```

### 3. Start Services

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

### 4. Configure MCP Server

Add to your MCP settings file:

```json
{
  "mcpServers": {
    "prometheus": {
      "command": "/path/to/bob-prometheus/venv/bin/python",
      "args": ["-m", "prometheus_mcp_server.main"],
      "env": {
        "PROMETHEUS_URL": "http://localhost:9090"
      }
    }
  }
}
```

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