# GitHub Copilot Integration Guide

Connect Enterprise GenAI Toolbox to GitHub Copilot for AI-powered database queries and infrastructure management directly in VS Code and other supported editors.

## What is GitHub Copilot + MCP?

**GitHub Copilot** is GitHub's AI pair programmer that helps you write code faster. With MCP (Model Context Protocol) support, Copilot can connect to external data sources and tools.

**Enterprise GenAI Toolbox** provides Copilot with direct access to your AWS databases, analytics platforms, and observability tools.

## Quick Setup (5 Minutes)

### Prerequisites

1. **GitHub Copilot** subscription ([sign up](https://github.com/features/copilot))
2. **VS Code** with GitHub Copilot extension installed
3. **Enterprise GenAI Toolbox** installed ([see INSTALL.md](../../INSTALL.md))
4. Your database credentials configured

### Step 1: Install GitHub Copilot Extension

In VS Code:
1. Open Extensions (Ctrl+Shift+X / Cmd+Shift+X)
2. Search for "GitHub Copilot"
3. Install "GitHub Copilot" and "GitHub Copilot Chat"
4. Sign in with your GitHub account

### Step 2: Start the MCP Server

The toolbox runs as an HTTP server that Copilot can connect to:

```bash
genai-toolbox --tools-file tools.yaml --port 5000
```

**Run in background:**
```bash
# macOS/Linux
nohup genai-toolbox --tools-file tools.yaml --port 5000 > toolbox.log 2>&1 &

# Or use systemd, Docker, etc.
```

### Step 3: Configure VS Code Settings

Add to your VS Code settings (`.vscode/settings.json` or User Settings):

```json
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "enterprise-database-toolbox",
        "url": "http://localhost:5000",
        "description": "Access to AWS databases and observability platforms"
      }
    ]
  }
}
```

**For team-wide configuration, add to workspace settings:**
```json
// .vscode/settings.json (committed to repo)
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "database-toolbox-dev",
        "url": "http://localhost:5000",
        "description": "Development databases"
      },
      {
        "name": "database-toolbox-prod",
        "url": "http://localhost:5001",
        "description": "Production databases (read-only)"
      }
    ]
  }
}
```

### Step 4: Verify Connection

In VS Code, open Copilot Chat (Ctrl+Shift+I / Cmd+Shift+I) and ask:
```
What database tools are available?
```

Copilot should show your configured database sources!

---

## Example Use Cases

### 1. Query DynamoDB While Coding

**Your tools.yaml:**
```yaml
sources:
  - name: app-dynamodb
    kind: dynamodb
    region: us-east-1

tools:
  scan-users:
    kind: dynamodb-scan
    source: app-dynamodb
    description: Scan users table
    parameters:
      - name: table_name
        type: string
        description: Table name
```

**In Copilot Chat:**
```
@workspace How many users do we have in the users table?
```

**In Code:**
```python
# You're writing code and need to know the table structure
# Ask Copilot: "What's the schema of the users table?"

class User:
    # Copilot suggests fields based on actual DynamoDB schema
    def __init__(self, user_id: str, email: str, created_at: str):
        self.user_id = user_id
        self.email = email
        self.created_at = created_at
```

### 2. Analytics Code Generation

**Your tools.yaml:**
```yaml
sources:
  - name: analytics
    kind: redshift
    host: analytics.us-west-2.redshift.amazonaws.com
    port: 5439
    user: ${REDSHIFT_USER}
    password: ${REDSHIFT_PASSWORD}
    database: analytics

tools:
  query-analytics:
    kind: postgres-sql
    source: analytics
    description: Query analytics warehouse
```

**In Copilot Chat:**
```
Generate a Python function to fetch daily active users from Redshift
```

**Copilot generates:**
```python
import psycopg2
from datetime import datetime, timedelta

def get_daily_active_users(days: int = 30) -> list:
    """
    Fetch daily active users for the last N days from Redshift.
    Uses the schema from your analytics warehouse.
    """
    # Copilot knows your actual schema from the toolbox
    query = """
        SELECT
            date_trunc('day', activity_timestamp) as date,
            COUNT(DISTINCT user_id) as active_users
        FROM user_activities
        WHERE activity_timestamp >= CURRENT_DATE - INTERVAL '%s days'
        GROUP BY date
        ORDER BY date DESC
    """ % days

    # ... rest of implementation
```

### 3. Debugging with Logs

**Your tools.yaml:**
```yaml
sources:
  - name: app-logs
    kind: cloudwatch
    region: us-east-1
    logGroup: /aws/lambda/my-app

tools:
  search-logs:
    kind: cloudwatch-filter
    source: app-logs
    description: Search application logs
```

**While debugging in VS Code:**
```
// Error in your code
function processPayment(order) {
    // throws error sometimes
}

// Ask Copilot: "Find recent errors related to processPayment in CloudWatch"
// Copilot queries logs and suggests the fix based on actual error messages
```

---

## Advanced Configuration

### Multiple Environments

Run separate toolbox instances for different environments:

**Development (port 5000):**
```bash
genai-toolbox --tools-file dev-tools.yaml --port 5000
```

**Staging (port 5001):**
```bash
genai-toolbox --tools-file staging-tools.yaml --port 5001
```

**Production (port 5002, read-only):**
```bash
genai-toolbox --tools-file prod-tools.yaml --port 5002
```

**VS Code settings:**
```json
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "dev-databases",
        "url": "http://localhost:5000",
        "description": "Development environment"
      },
      {
        "name": "staging-databases",
        "url": "http://localhost:5001",
        "description": "Staging environment"
      },
      {
        "name": "prod-databases-readonly",
        "url": "http://localhost:5002",
        "description": "Production (read-only)"
      }
    ]
  }
}
```

### Remote Server Configuration

For team setups, run toolbox on a shared server:

```json
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "team-databases",
        "url": "https://toolbox.company.internal:5000",
        "description": "Shared team database access",
        "headers": {
          "Authorization": "Bearer ${TOOLBOX_API_TOKEN}"
        }
      }
    ]
  }
}
```

**Server setup:**
```bash
# Run with TLS for production
genai-toolbox \
  --tools-file tools.yaml \
  --port 5000 \
  --address 0.0.0.0 \
  --telemetry-otlp http://telemetry:4318
```

### Authentication & Security

**API Token authentication:**
```bash
# Set environment variable
export TOOLBOX_API_TOKEN="your-secure-token"

# VS Code settings
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "secure-toolbox",
        "url": "https://toolbox.company.com",
        "headers": {
          "Authorization": "Bearer ${TOOLBOX_API_TOKEN}"
        }
      }
    ]
  }
}
```

---

## Workflow Examples

### 1. Data-Driven Development

**Scenario:** Building a new feature that needs to query existing data

```python
# In VS Code, ask Copilot:
# "Show me the most common user signup sources from the last 30 days"

# Copilot queries your database and suggests:
def analyze_signup_sources():
    """
    Based on your actual data:
    - Organic: 45%
    - Google Ads: 30%
    - Social Media: 15%
    - Referral: 10%
    """
    # Generates code with actual patterns from your database
    pass

# Then ask: "Generate a signup attribution dashboard using this data"
# Copilot creates the full implementation with real queries
```

### 2. Schema-Aware Code Generation

**Scenario:** Creating a new API endpoint

```typescript
// Ask Copilot: "Create a REST API endpoint to fetch user profile with orders"

// Copilot generates code using your ACTUAL database schema:
app.get('/api/users/:userId/profile', async (req, res) => {
    const { userId } = req.params;

    // Copilot knows your exact table structure
    const user = await db.query(`
        SELECT
            u.user_id,
            u.email,
            u.first_name,
            u.last_name,
            u.created_at,
            COUNT(o.order_id) as total_orders,
            SUM(o.total_amount) as lifetime_value
        FROM users u
        LEFT JOIN orders o ON u.user_id = o.user_id
        WHERE u.user_id = $1
        GROUP BY u.user_id
    `, [userId]);

    res.json(user);
});
```

### 3. Debugging with Context

**Scenario:** Application error in production

```javascript
// Error in logs: "Payment processing failed for order #12345"

// Ask Copilot: "What happened with order 12345?"

// Copilot queries:
// 1. DynamoDB for order details
// 2. CloudWatch for related logs
// 3. Redshift for payment transaction history

// Then suggests:
/*
 * Issue found: Payment gateway timeout after 30s
 *
 * Recommendation:
 * 1. Increase timeout to 45s
 * 2. Add retry logic with exponential backoff
 * 3. Implement better error handling
 *
 * Here's the fixed code:
 */

async function processPayment(order) {
    const timeout = 45000; // Increased from 30s
    const maxRetries = 3;

    for (let i = 0; i < maxRetries; i++) {
        try {
            return await paymentGateway.charge(order, { timeout });
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            await sleep(Math.pow(2, i) * 1000); // Exponential backoff
        }
    }
}
```

### 4. Performance Optimization

```sql
-- Copilot detects slow query in your code
-- Ask: "Optimize this query for better performance"

-- Before (slow):
SELECT * FROM orders
WHERE user_id IN (SELECT user_id FROM users WHERE created_at > '2024-01-01')

-- Copilot suggests (based on your actual indexes):
SELECT o.*
FROM orders o
INNER JOIN users u ON o.user_id = u.user_id
WHERE u.created_at > '2024-01-01'
AND o.status = 'completed'  -- Added based on typical usage patterns

-- Copilot explains: "Using INNER JOIN instead of IN subquery improves
-- performance by 10x. Also added status filter based on 95% of queries."
```

---

## Troubleshooting

### Copilot can't see the toolbox

**1. Verify toolbox is running:**
```bash
curl http://localhost:5000/v1/toolsets
```

**2. Check VS Code settings:**
```bash
# Open settings.json and verify externalTools configuration
code ~/.vscode/settings.json
```

**3. Restart VS Code:**
Close and reopen VS Code after changing settings

**4. Check Copilot logs:**
- Open Output panel (View > Output)
- Select "GitHub Copilot" from dropdown
- Look for connection errors

### Connection refused

**1. Confirm port is correct:**
```bash
netstat -an | grep 5000
# or
lsof -i :5000
```

**2. Check firewall:**
```bash
# macOS
sudo pfctl -sr | grep 5000

# Linux
sudo iptables -L | grep 5000
```

**3. Test with curl:**
```bash
curl -v http://localhost:5000/v1/toolsets
```

### Copilot doesn't use the tools

**1. Be explicit in requests:**
```
# Instead of: "How many users?"
# Try: "Query the database to count total users"
```

**2. Check tool descriptions:**
Make sure your tools.yaml has clear descriptions that Copilot can understand

**3. Verify toolset configuration:**
```yaml
toolsets:
  default:  # Copilot uses the default toolset
    - list-tables
    - query-users
    - search-logs
```

---

## Best Practices

### 1. Descriptive Tool Names

```yaml
# Good: Clear, specific names
tools:
  get-active-users-last-30-days:
    description: Retrieve count of active users in the last 30 days

  search-error-logs-by-severity:
    description: Search CloudWatch logs for errors by severity level
```

### 2. Use Toolsets for Context

```yaml
# Organize tools by feature/module
toolsets:
  user-management:
    - create-user
    - get-user-profile
    - list-users

  analytics:
    - daily-active-users
    - revenue-by-product
    - user-retention-cohorts

  monitoring:
    - search-error-logs
    - get-api-metrics
    - list-recent-deployments
```

### 3. Limit Scope

```yaml
# Read-only for safety
tools:
  query-production:
    kind: postgres-sql
    source: prod-database
    description: READONLY query production database
    statement: ${sql}  # Copilot knows this is read-only
```

### 4. Team Shared Configuration

Commit VS Code settings to your repository:

```json
// .vscode/settings.json (committed)
{
  "github.copilot.advanced": {
    "externalTools": [
      {
        "name": "project-databases",
        "url": "http://localhost:5000",
        "description": "Project database access for the team"
      }
    ]
  },
  "recommendations": [
    "github.copilot",
    "github.copilot-chat"
  ]
}
```

---

## Integration with Development Workflow

### 1. Code Reviews

Use Copilot with database context for better reviews:

```typescript
// Reviewer asks: "Is this query efficient?"
const users = await db.query(`
    SELECT * FROM users WHERE email LIKE '%@gmail.com'
`);

// Copilot (with database access) suggests:
/*
 * This query will be slow. Based on your users table (5M rows):
 *
 * Issues:
 * 1. No index on email
 * 2. Leading wildcard prevents index usage
 * 3. SELECT * fetches unnecessary columns
 *
 * Recommended:
 */
const users = await db.query(`
    SELECT user_id, email, name
    FROM users
    WHERE email LIKE '%%gmail.com'  -- Still slow but better
`);
// Better: Add index on email or use full-text search
```

### 2. Documentation Generation

```python
# Ask Copilot: "Document this database schema"

class UserModel:
    """
    User model representing the users table in PostgreSQL.

    Schema (from production):
    - user_id: UUID primary key
    - email: VARCHAR(255) unique
    - created_at: TIMESTAMP
    - last_login: TIMESTAMP
    - status: ENUM('active', 'inactive', 'suspended')

    Indexes:
    - idx_users_email (email)
    - idx_users_created_at (created_at)

    Relationships:
    - One-to-many with orders table
    - One-to-many with user_sessions table

    Average row count: 2.5M users
    Growth rate: ~1000 users/day
    """
    pass
```

### 3. Test Data Generation

```python
# Ask Copilot: "Generate test data matching our production schema"

def generate_test_users(count=100):
    """
    Generate test users matching production user patterns.
    Based on analysis of actual production data.
    """
    # Copilot generates realistic test data based on:
    # - Actual email patterns
    # - Common user names
    # - Typical signup dates
    # - Real status distributions
    pass
```

---

## Next Steps

- See [Claude Code Integration](CLAUDE_CODE_INTEGRATION.md) for Claude Desktop setup
- See [AWS Integration Guide](AWS_INTEGRATIONS.md) for AWS configuration
- See [Production Deployment](PRODUCTION_DEPLOYMENT_GUIDE.md) for production setup

---

## Resources

- [GitHub Copilot](https://github.com/features/copilot)
- [VS Code Copilot Extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
- [MCP Documentation](https://modelcontextprotocol.io/)
- [Example Configurations](../../examples/)

---

**🤖 Powered by GitHub Copilot and Enterprise GenAI Toolbox**
