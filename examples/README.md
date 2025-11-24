# Enterprise GenAI Toolbox - Example Configurations

This directory contains example YAML configurations for different use cases.

## Quick Reference

| File | Use Case | Description |
|------|----------|-------------|
| `quick-start.yaml` | Getting Started | Simple PostgreSQL, MySQL, and Redis |
| `local-development.yaml` | Local Testing | Docker-based local databases |
| `aws-complete.yaml` | AWS Production | All AWS database services |
| `enterprise-observability.yaml` | Monitoring | Honeycomb, Splunk, Tableau |
| `multi-cloud.yaml` | Hybrid Cloud | AWS + GCP + On-premises |
| `data-warehouse.yaml` | Analytics | Data warehouse and BI platforms |

## Usage

1. **Copy an example file:**
   ```bash
   cp examples/quick-start.yaml my-config.yaml
   ```

2. **Edit the configuration:**
   - Replace placeholder values (hosts, passwords, etc.)
   - Use environment variables for secrets: `${VAR_NAME}`
   - Remove sources you don't need

3. **Run the toolbox:**
   ```bash
   genai-toolbox --tools-file my-config.yaml
   ```

## Environment Variables

For security, use environment variables for sensitive data:

```bash
export REDSHIFT_PASSWORD="your-password"
export HONEYCOMB_API_KEY="hcxik_your_key"
export SPLUNK_TOKEN="your-token"
export TABLEAU_PAT_SECRET="your-secret"

genai-toolbox --tools-file examples/aws-complete.yaml
```

## Docker Compose for Local Development

Run local databases for testing:

```bash
# PostgreSQL
docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=dev postgres

# MySQL
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=dev mysql

# MongoDB
docker run -d --name mongodb -p 27017:27017 mongo

# Redis
docker run -d --name redis -p 6379:6379 redis

# DynamoDB Local
docker run -d --name dynamodb -p 8000:8000 amazon/dynamodb-local

# Then use local-development.yaml
genai-toolbox --tools-file examples/local-development.yaml
```

## AWS Authentication

AWS sources support multiple authentication methods:

1. **IAM Roles** (recommended for EC2/ECS/Lambda)
   - No configuration needed
   - Automatically uses instance/task role

2. **Environment Variables:**
   ```bash
   export AWS_ACCESS_KEY_ID="your-key"
   export AWS_SECRET_ACCESS_KEY="your-secret"
   export AWS_REGION="us-east-1"
   ```

3. **AWS CLI Profiles:**
   ```bash
   aws configure  # Sets up ~/.aws/credentials
   ```

4. **Explicit in YAML** (not recommended for production):
   ```yaml
   sources:
     my-dynamodb:
       kind: dynamodb
       region: us-east-1
       accessKeyId: your-key
       secretAccessKey: your-secret
   ```

## GCP Authentication

GCP sources use Application Default Credentials:

```bash
# Login with gcloud
gcloud auth application-default login

# Or set service account key
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"
```

## Common Patterns

### Production Secrets Management

**Option 1: Environment Variables**
```yaml
sources:
  prod-db:
    kind: postgres
    host: db.example.com
    password: ${DB_PASSWORD}  # Loaded from environment
```

**Option 2: AWS Secrets Manager** (retrieve before starting)
```bash
export DB_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id prod/db/password --query SecretString --output text)

genai-toolbox --tools-file config.yaml
```

### Connection Pooling

Adjust for your workload:

```yaml
sources:
  high-traffic-db:
    kind: postgres
    host: db.example.com
    # ... other config ...
    # For high traffic:
    maxOpenConns: 100
    maxIdleConns: 25
```

### Multi-Region Setup

```yaml
sources:
  us-east-dynamodb:
    kind: dynamodb
    region: us-east-1

  eu-west-dynamodb:
    kind: dynamodb
    region: eu-west-1
```

## Troubleshooting

### Connection Timeouts

Increase timeout values:

```yaml
sources:
  slow-db:
    kind: postgres
    host: remote-db.example.com
    connectTimeout: 30  # seconds
```

### TLS/SSL Issues

For self-signed certificates:

```yaml
sources:
  internal-db:
    kind: postgres
    host: internal.example.com
    sslMode: require
    sslRootCert: /path/to/ca.pem
```

### VPC/Firewall Issues

Ensure:
- Security groups allow inbound traffic
- Network ACLs permit connections
- VPC endpoints configured for AWS services
- Private DNS resolution working

## Next Steps

1. Review [AWS_INTEGRATIONS.md](../docs/guides/AWS_INTEGRATIONS.md) for detailed AWS setup
2. Check [PRODUCTION_DEPLOYMENT_GUIDE.md](../docs/guides/PRODUCTION_DEPLOYMENT_GUIDE.md)
3. See [VALIDATION_GUIDE.md](../docs/guides/VALIDATION_GUIDE.md) for testing

## Need Help?

- **GitHub Issues:** https://github.com/sethdford/genai-toolbox-enterprise/issues
- **Documentation:** https://github.com/sethdford/genai-toolbox-enterprise#readme
