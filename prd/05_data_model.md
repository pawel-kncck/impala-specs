### 3.3. Database Schemas

#### 3.3.1. PostgreSQL Schema (Extended)

```sql
-- Core Tables
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    plan_type VARCHAR(50) NOT NULL DEFAULT 'free',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(255) UNIQUE NOT NULL,
    hashed_password TEXT,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    avatar_url TEXT,
    organization_id UUID REFERENCES organizations(id),
    role VARCHAR(50) NOT NULL DEFAULT 'member',
    settings JSONB DEFAULT '{}',
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    organization_id UUID REFERENCES organizations(id),
    owner_id UUID REFERENCES users(id),
    settings JSONB DEFAULT '{}',
    is_archived BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Collaboration Tables
CREATE TABLE project_members (
    project_id UUID REFERENCES projects(id),
    user_id UUID REFERENCES users(id),
    role VARCHAR(50) NOT NULL, -- 'viewer', 'editor', 'admin'
    permissions JSONB DEFAULT '{}',
    invited_by UUID REFERENCES users(id),
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (project_id, user_id)
);

-- Data Management Tables
CREATE TABLE data_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'file', 'database', 'api', 'streaming'
    connection_config JSONB NOT NULL,
    schema_info JSONB,
    metadata JSONB DEFAULT '{}',
    status VARCHAR(50) NOT NULL DEFAULT 'active',
    last_sync_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE data_pipelines (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    name VARCHAR(255) NOT NULL,
    source_id UUID REFERENCES data_sources(id),
    pipeline_config JSONB NOT NULL,
    schedule_cron VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    last_run_at TIMESTAMP,
    next_run_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Analytics Tables
CREATE TABLE saved_queries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    name VARCHAR(255) NOT NULL,
    query_text TEXT NOT NULL,
    query_type VARCHAR(50) NOT NULL, -- 'sql', 'python', 'natural_language'
    parameters JSONB DEFAULT '{}',
    created_by UUID REFERENCES users(id),
    is_public BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE query_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    query_id UUID REFERENCES saved_queries(id),
    executed_by UUID REFERENCES users(id),
    status VARCHAR(50) NOT NULL, -- 'pending', 'running', 'completed', 'failed'
    parameters JSONB DEFAULT '{}',
    result_location TEXT,
    error_message TEXT,
    execution_time_ms INTEGER,
    rows_returned INTEGER,
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP
);

-- AI Integration Tables
CREATE TABLE ai_interactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    project_id UUID REFERENCES projects(id),
    interaction_type VARCHAR(50) NOT NULL,
    request JSONB NOT NULL,
    response JSONB NOT NULL,
    provider VARCHAR(50) NOT NULL,
    model VARCHAR(100) NOT NULL,
    input_tokens INTEGER,
    output_tokens INTEGER,
    cost_usd DECIMAL(10, 6),
    latency_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Audit Tables
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    organization_id UUID REFERENCES organizations(id),
    action VARCHAR(255) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id UUID,
    changes JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_projects_org ON projects(organization_id);
CREATE INDEX idx_data_sources_project ON data_sources(project_id);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id, created_at DESC);
CREATE INDEX idx_ai_interactions_user ON ai_interactions(user_id, created_at DESC);
```

#### 3.3.2. MongoDB Schema (Canvas Structure)

```javascript
// Canvas Collection
{
  "_id": ObjectId("..."),
  "project_id": "uuid",
  "name": "Q4 Sales Analysis",
  "description": "Analysis of Q4 2024 sales performance",
  "owner_id": "uuid",
  "version": 1,
  "layout": {
    "type": "grid",
    "columns": 12,
    "rowHeight": 50,
    "breakpoints": {
      "lg": 1200,
      "md": 996,
      "sm": 768,
      "xs": 480
    }
  },
  "blocks": [
    {
      "id": "block_1",
      "type": "chart",
      "position": { "x": 0, "y": 0, "w": 6, "h": 4 },
      "content": {
        "chart_type": "bar",
        "title": "Sales by Region",
        "data_source": {
          "type": "query",
          "query_id": "uuid",
          "parameters": {}
        },
        "config": {
          "x_axis": "region",
          "y_axis": "total_sales",
          "color": "product_category",
          "aggregation": "sum"
        },
        "style": {
          "theme": "light",
          "colors": ["#1f77b4", "#ff7f0e", "#2ca02c"]
        }
      },
      "metadata": {
        "created_by": "uuid",
        "created_at": ISODate("2024-01-15T10:30:00Z"),
        "last_modified_by": "uuid",
        "last_modified_at": ISODate("2024-01-15T14:20:00Z")
      }
    },
    {
      "id": "block_2",
      "type": "narrative",
      "position": { "x": 6, "y": 0, "w": 6, "h": 4 },
      "content": {
        "text": "## Key Insights\n\nSales in Q4 increased by {{metric:25.3}}% compared to Q3...",
        "data_bindings": [
          {
            "key": "metric:25.3",
            "source": "block_1",
            "calculation": "percentage_change",
            "value": 25.3
          }
        ]
      }
    },
    {
      "id": "block_3",
      "type": "table",
      "position": { "x": 0, "y": 4, "w": 12, "h": 6 },
      "content": {
        "title": "Detailed Sales Breakdown",
        "data_source": {
          "type": "query",
          "query_id": "uuid"
        },
        "columns": [
          { "field": "product", "header": "Product", "type": "string" },
          { "field": "q4_sales", "header": "Q4 Sales", "type": "currency" },
          { "field": "growth", "header": "Growth %", "type": "percentage" }
        ],
        "pagination": { "enabled": true, "pageSize": 20 },
        "sorting": { "enabled": true, "defaultSort": "q4_sales" },
        "filtering": { "enabled": true }
      }
    }
  ],
  "sharing": {
    "visibility": "private", // 'private', 'team', 'public'
    "permissions": [
      {
        "user_id": "uuid",
        "role": "editor",
        "granted_at": ISODate("2024-01-10T09:00:00Z"),
        "granted_by": "uuid"
      }
    ],
    "public_link": null,
    "embed_settings": {
      "enabled": false,
      "allowed_domains": []
    }
  },
  "interactions": {
    "views": 145,
    "last_viewed_at": ISODate("2024-01-15T16:45:00Z"),
    "comments_count": 12,
    "favorites_count": 5
  },
  "metadata": {
    "tags": ["sales", "q4", "regional"],
    "created_at": ISODate("2024-01-01T10:00:00Z"),
    "updated_at": ISODate("2024-01-15T14:20:00Z"),
    "version_history": [
      {
        "version": 1,
        "changes": "Initial creation",
        "changed_by": "uuid",
        "changed_at": ISODate("2024-01-01T10:00:00Z")
      }
    ]
  }
}

// Canvas Templates Collection
{
  "_id": ObjectId("..."),
  "name": "Sales Dashboard Template",
  "description": "Standard template for sales analysis",
  "category": "sales",
  "thumbnail_url": "https://...",
  "is_public": true,
  "usage_count": 234,
  "blocks": [...], // Predefined blocks structure
  "required_fields": ["date", "sales_amount", "region"],
  "created_by": "system",
  "created_at": ISODate("2024-01-01T00:00:00Z")
}
```

#### 3.3.3. Redis Cache Schema

```yaml
# Cache Key Patterns

# User Sessions
session:{user_id}:
  data: { token, user_info, permissions }
  ttl: 24h

# Query Results
query:result:{query_hash}:
  data: { result_data, metadata, execution_time }
  ttl: 1h

# Real-time Collaboration
canvas:state:{canvas_id}:
  data: { current_state, active_users, locks }
  ttl: persistent with cleanup

canvas:presence:{canvas_id}:
  data: { user_id: { cursor, selection, last_seen } }
  ttl: 5m rolling

# Rate Limiting
rate_limit:{user_id}:{endpoint}:
  data: request_count
  ttl: 1m

# AI Response Cache
ai:response:{prompt_hash}:
  data: { response, model, timestamp }
  ttl: 24h

# Temporary Data Processing
processing:{job_id}:
  data: { status, progress, partial_results }
  ttl: 1h
```
