# Claude Code Integration Guide

Connect Enterprise GenAI Toolbox to Claude Code (Claude Desktop) via MCP (Model Context Protocol) for seamless database access from your AI assistant.

## What is Claude Code + MCP?

**Claude Code** (part of Claude Desktop) is an AI coding assistant that can connect to MCP servers to access external data sources and tools. This integration lets Claude directly query your databases, run analytics, and work with your data infrastructure.

**Model Context Protocol (MCP)** is an open protocol that enables AI assistants to securely connect to external systems.

## Quick Setup (5 Minutes)

### Prerequisites

1. **Claude Desktop** installed ([download](https://claude.ai/download))
2. **Enterprise GenAI Toolbox** installed ([see INSTALL.md](../../INSTALL.md))
3. Your database credentials configured

### Step 1: Start the MCP Server

The toolbox can run as an MCP server via stdio or HTTP:

**Option A: STDIO (Recommended for Claude Desktop)**
```bash
genai-toolbox --tools-file tools.yaml --stdio
```

**Option B: HTTP Server**
```bash
genai-toolbox --tools-file tools.yaml --port 5000
```

### Step 2: Configure Claude Desktop

Add the MCP server to your Claude Desktop configuration:

**Location:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Configuration:**

```json
{
  "mcpServers": {
    "enterprise-database-toolbox": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "/path/to/your/tools.yaml", "--stdio"],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_PROFILE": "default"
      }
    }
  }
}
```

**Multiple Environments:**
```json
{
  "mcpServers": {
    "toolbox-production": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "/path/to/prod-tools.yaml", "--stdio"]
    },
    "toolbox-staging": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "/path/to/staging-tools.yaml", "--stdio"]
    }
  }
}
```

### Step 3: Restart Claude Desktop

Close and reopen Claude Desktop to load the MCP server configuration.

### Step 4: Verify Connection

In Claude Desktop, type:
```
Can you list the available database tools?
```

Claude should now be able to see and use your configured database tools!

---

## Example Use Cases

### 1. Query DynamoDB from Claude

**Your tools.yaml:**
```yaml
sources:
  - name: my-dynamodb
    kind: dynamodb
    region: us-east-1

tools:
  list-tables:
    kind: dynamodb-list-tables
    source: my-dynamodb
    description: List all DynamoDB tables

toolsets:
  aws:
    - list-tables
```

**In Claude Desktop:**
```
Can you list all my DynamoDB tables?
```

Claude will automatically use the `list-tables` tool and show you the results!

### 2. Analytics with Redshift

**Your tools.yaml:**
```yaml
sources:
  - name: analytics-warehouse
    kind: redshift
    host: my-cluster.us-west-2.redshift.amazonaws.com
    port: 5439
    user: ${REDSHIFT_USER}
    password: ${REDSHIFT_PASSWORD}
    database: analytics

tools:
  query-sales:
    kind: postgres-sql
    source: analytics-warehouse
    description: Query sales data
    parameters:
      - name: sql
        type: string
        description: SQL query to execute
    statement: ${sql}
```

**In Claude Desktop:**
```
Show me total sales by region for the last 30 days
```

Claude will write and execute the SQL query using your Redshift connection!

### 3. Monitor with CloudWatch

**Your tools.yaml:**
```yaml
sources:
  - name: app-logs
    kind: cloudwatch
    region: us-east-1
    logGroup: /aws/lambda/my-app

tools:
  search-errors:
    kind: cloudwatch-filter
    source: app-logs
    description: Search application logs for errors
    parameters:
      - name: pattern
        type: string
        description: Filter pattern
```

**In Claude Desktop:**
```
Find all errors in the application logs from the last hour
```

---

## Advanced Configuration

### Secure Credential Management

**Using Environment Variables:**
```json
{
  "mcpServers": {
    "toolbox": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "/path/to/tools.yaml", "--stdio"],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_PROFILE": "production",
        "REDSHIFT_USER": "admin",
        "REDSHIFT_PASSWORD": "${REDSHIFT_PASSWORD}",
        "HONEYCOMB_API_KEY": "${HONEYCOMB_API_KEY}"
      }
    }
  }
}
```

**Using AWS Credential Chain:**
The toolbox automatically uses AWS credential chain:
1. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
2. AWS credentials file (`~/.aws/credentials`)
3. IAM roles (when running on EC2/ECS/EKS)

### Multiple Tool Configurations

**Separate by environment:**
```json
{
  "mcpServers": {
    "dev-databases": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "~/.config/toolbox/dev-tools.yaml", "--stdio"]
    },
    "prod-databases": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "~/.config/toolbox/prod-tools.yaml", "--stdio"]
    },
    "observability": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": ["--tools-file", "~/.config/toolbox/observability.yaml", "--stdio"]
    }
  }
}
```

### Custom Logging

**Enable debug logging:**
```json
{
  "mcpServers": {
    "toolbox": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": [
        "--tools-file", "/path/to/tools.yaml",
        "--stdio",
        "--log-level", "DEBUG"
      ]
    }
  }
}
```

**JSON logging for structured logs:**
```json
{
  "mcpServers": {
    "toolbox": {
      "command": "/usr/local/bin/genai-toolbox",
      "args": [
        "--tools-file", "/path/to/tools.yaml",
        "--stdio",
        "--logging-format", "JSON"
      ]
    }
  }
}
```

---

## Troubleshooting

### Claude Desktop doesn't see the MCP server

**1. Check configuration file location:**
```bash
# macOS
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Linux
cat ~/.config/Claude/claude_desktop_config.json
```

**2. Verify binary path:**
```bash
which genai-toolbox
# Use the full path in your config
```

**3. Test the command manually:**
```bash
/usr/local/bin/genai-toolbox --tools-file /path/to/tools.yaml --stdio
```

**4. Check Claude Desktop logs:**
- macOS: `~/Library/Logs/Claude/`
- Windows: `%APPDATA%\Claude\logs\`
- Linux: `~/.config/Claude/logs/`

### "Connection refused" or "Cannot connect to server"

**1. Verify the toolbox starts:**
```bash
genai-toolbox --tools-file tools.yaml --stdio
# Should wait for input (press Ctrl+C to exit)
```

**2. Check your tools.yaml syntax:**
```bash
python3 -c "import yaml; yaml.safe_load(open('tools.yaml'))"
```

**3. Verify database credentials:**
```bash
# Test AWS credentials
aws sts get-caller-identity

# Test with a simple tool
genai-toolbox --tools-file tools.yaml --port 5000
# Then in another terminal:
curl http://localhost:5000/v1/toolsets
```

### "Tool execution failed"

**1. Check database connectivity:**
```bash
# For AWS services
aws dynamodb list-tables --region us-east-1

# For PostgreSQL/Redshift
psql -h hostname -U user -d database -c "SELECT 1"
```

**2. Verify IAM permissions:**
- DynamoDB: `dynamodb:ListTables`, `dynamodb:Scan`, `dynamodb:Query`
- S3: `s3:ListBucket`, `s3:GetObject`
- CloudWatch: `logs:FilterLogEvents`, `logs:DescribeLogGroups`

**3. Enable debug logging:**
Add `--log-level DEBUG` to your config and check Claude Desktop logs.

---

## Best Practices

### 1. Principle of Least Privilege
Only grant Claude access to necessary tools:

```yaml
# Good: Specific, limited access
tools:
  list-production-errors:
    kind: cloudwatch-filter
    source: prod-logs
    description: List errors in production logs (last 24h only)
    parameters:
      - name: hours
        type: integer
        description: Hours to look back (max 24)
```

### 2. Use Toolsets for Organization
```yaml
toolsets:
  # Safe read-only operations
  readonly:
    - list-tables
    - query-sales-dashboard
    - view-errors

  # Administrative operations (use with care)
  admin:
    - create-table
    - delete-records
    - update-config
```

### 3. Secure Sensitive Data
```yaml
# Use environment variables for credentials
sources:
  - name: production-db
    kind: postgres
    host: prod.example.com
    user: ${DB_USER}
    password: ${DB_PASSWORD}  # Never commit passwords!
    database: main
```

### 4. Monitor Usage
```yaml
# Enable telemetry for monitoring
# Run with: --telemetry-otlp http://localhost:4318
```

---

## Example Workflows

### Development Workflow
```
Claude: "Can you show me the schema for the users table?"
→ Queries PostgreSQL information_schema

Claude: "Now show me all active users from the last 7 days"
→ Executes SELECT with date filter

Claude: "Export this data and create a CSV analysis report"
→ Formats results and creates report
```

### Debugging Workflow
```
Claude: "Find all error logs for API gateway in the last hour"
→ Queries CloudWatch logs

Claude: "What's the most common error?"
→ Analyzes log patterns

Claude: "Show me the DynamoDB queries that are timing out"
→ Queries CloudWatch metrics

Claude: "Create a summary report with recommendations"
→ Generates analysis report
```

### Analytics Workflow
```
Claude: "What's our daily active user count for the last 30 days?"
→ Queries Redshift analytics warehouse

Claude: "Show me user retention by cohort"
→ Executes complex analytics SQL

Claude: "Create visualizations for these metrics"
→ Generates data visualizations

Claude: "Compare this to last quarter"
→ Runs comparative analysis
```

---

## Next Steps

- See [AWS Integration Guide](AWS_INTEGRATIONS.md) for AWS-specific configuration
- See [Production Deployment Guide](PRODUCTION_DEPLOYMENT_GUIDE.md) for production setup
- See [Validation Guide](VALIDATION_GUIDE.md) for testing your configuration

---

## Resources

- [Claude Desktop Download](https://claude.ai/download)
- [MCP Documentation](https://modelcontextprotocol.io/)
- [Example Configurations](../../examples/)
- [Enterprise Installation Guide](../../INSTALL.md)

---

**🤖 Powered by Claude Code and Enterprise GenAI Toolbox**
