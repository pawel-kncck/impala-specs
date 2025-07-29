# Technical Architecture

## 1. System Architecture Overview

### 1.1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                       Client Layer                                       │
├─────────────────┬─────────────────┬─────────────────┬─────────────────┬────────────────┤
│ Web App (React/ │  Mobile (React  │   Desktop App   │   Public API    │  Embedded      │
│   TypeScript)   │Native/TypeScript)│   (Electron)    │   (REST/GraphQL)│  Widget        │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┴────────────────┘
                                              │
                                              │ HTTPS/WSS
                                              ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    API Gateway Layer                                     │
├─────────────────┬─────────────────┬─────────────────┬─────────────────┬────────────────┤
│   Load Balancer │  Rate Limiter   │  Auth Gateway   │   API Router    │  WebSocket     │
│   (Nginx/ALB)   │  (Redis)        │  (JWT/OAuth)    │   (Kong/Envoy)  │  Manager       │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┴────────────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────────┐
                    ▼                         ▼                         ▼
┌─────────────────────────────┐ ┌─────────────────────────┐ ┌─────────────────────────┐
│      Core Services          │ │   Analytics Services    │ │  Collaboration Services │
├─────────────────────────────┤ ├─────────────────────────┤ ├─────────────────────────┤
│ • User Service              │ │ • Query Engine          │ │ • Real-time Sync        │
│ • Project Service           │ │ • Visualization Engine  │ │ • Comment Service       │
│ • Auth Service              │ │ • AI Service            │ │ • Notification Service  │
│ • File Service              │ │ • Transform Service     │ │ • Sharing Service       │
│ • Context Service           │ │ • Pipeline Service      │ │ • Version Control       │
│ • Onboarding Service        │ │ • Search & Discovery    │ │ • Message Queue         │
└─────────────────────────────┘ └─────────────────────────┘ └─────────────────────────┘
                    │                         │                         │
                    └─────────────────────────┴─────────────────────────┘
                                              │
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    Data Layer                                            │
├──────────────┬──────────────┬──────────────┬──────────────┬─────────────┬─────────────┤
│  PostgreSQL  │   MongoDB    │   Redis      │  ClickHouse  │   S3/Blob   │   Vector    │
│  (Core Data) │  (Canvases)  │  (Cache)     │  (Analytics) │  (Files)    │   DB        │
└──────────────┴──────────────┴──────────────┴──────────────┴─────────────┴─────────────┘
```

### 1.2. Core Architecture Principles

1. **Microservices Architecture**: Each service is independently deployable and scalable
2. **Event-Driven Communication**: Services communicate through event streams for loose coupling
3. **API-First Design**: All functionality exposed through well-defined APIs
4. **Cloud-Native**: Designed for container orchestration and auto-scaling
5. **Security by Design**: Zero-trust architecture with encryption at all layers
6. **Real-time Capable**: WebSocket support for live collaboration and updates
7. **Type-Safe Development**: TypeScript enforced across all frontend applications for enhanced reliability

## 2. Service Architecture

### 2.1. Core Services

#### 2.1.1. User Service

```yaml
service: user-service
responsibility: User management and profiles
technology:
  language: Python
  framework: FastAPI
  database: PostgreSQL
  cache: Redis
endpoints:
  - POST /users/register
  - POST /users/login
  - GET /users/profile
  - PUT /users/profile
  - POST /users/invite
  - GET /users/teams
integrations:
  - auth-service: JWT validation
  - notification-service: Welcome emails
  - audit-service: Activity logging
```

#### 2.1.2. Project Service

```yaml
service: project-service
responsibility: Project and workspace management
technology:
  language: Python
  framework: FastAPI
  database: PostgreSQL
  cache: Redis
endpoints:
  - POST /projects
  - GET /projects
  - PUT /projects/{id}
  - DELETE /projects/{id}
  - POST /projects/{id}/members
  - GET /projects/{id}/activity
integrations:
  - file-service: Data source management
  - canvas-service: Canvas association
  - permission-service: Access control
```

#### 2.1.3. Authentication Service

```yaml
service: auth-service
responsibility: Authentication and authorization
technology:
  language: Go
  framework: Gin
  database: PostgreSQL
  cache: Redis
features:
  - JWT token generation/validation
  - OAuth 2.0 integration (Google, Microsoft, GitHub)
  - SSO/SAML support
  - MFA support
  - API key management
  - Session management
endpoints:
  - POST /auth/token
  - POST /auth/refresh
  - POST /auth/revoke
  - GET /auth/validate
  - POST /auth/oauth/{provider}
```

#### 2.1.4. Context Service

```yaml
service: context-service
responsibility: AI context management and business domain understanding
technology:
  language: Python
  framework: FastAPI
  database: PostgreSQL
  vector_db: Pinecone/Weaviate
  cache: Redis
features:
  - Business context storage and retrieval
  - Domain knowledge persistence
  - Context versioning and history
  - Cross-project context sharing
  - Semantic search capabilities
  - Context-aware AI training
architecture:
  components:
    - Context Storage: Versioned context data
    - Knowledge Graph: Business relationships
    - Semantic Engine: Context understanding
    - Context Builder: Automated context extraction
    - Sharing Manager: Cross-project context
endpoints:
  - POST /context/create
  - GET /context/{project_id}
  - PUT /context/{id}
  - POST /context/search
  - GET /context/history/{id}
integrations:
  - ai-service: Context-aware responses
  - project-service: Project context association
  - vector-db: Semantic similarity search
```

#### 2.1.5. Onboarding Service

```yaml
service: onboarding-service
responsibility: User onboarding and guided experiences
technology:
  language: TypeScript
  framework: Node.js/Express
  database: PostgreSQL
  cache: Redis
features:
  - 60-second onboarding flow
  - Progressive feature disclosure
  - Interactive tutorials
  - Personalized recommendations
  - Achievement tracking
  - Multi-step workflows
architecture:
  components:
    - Flow Engine: Dynamic workflow management
    - Progress Tracker: User journey analytics
    - Content Manager: Tutorial and guide content
    - Personalization Engine: Adaptive experiences
    - Analytics Collector: Onboarding metrics
endpoints:
  - POST /onboarding/start
  - GET /onboarding/progress/{user_id}
  - PUT /onboarding/step/{id}
  - POST /onboarding/complete
  - GET /onboarding/recommendations
integrations:
  - user-service: Profile-based personalization
  - analytics-service: Usage pattern analysis
  - notification-service: Progress notifications
```

### 2.2. Analytics Services

#### 2.2.1. Query Engine Service

```yaml
service: query-engine-service
responsibility: Execute data queries and transformations
technology:
  language: Python/Rust
  framework: FastAPI
  query_engine: DuckDB/Apache Arrow
  cache: Redis
features:
  - SQL query execution
  - Python code execution (sandboxed)
  - Query optimization
  - Result caching
  - Streaming results
architecture:
  - Worker Pool: Isolated query execution
  - Query Parser: SQL/Python validation
  - Optimizer: Query plan optimization
  - Cache Manager: Intelligent result caching
endpoints:
  - POST /query/execute
  - GET /query/status/{id}
  - GET /query/results/{id}
  - POST /query/explain
  - POST /query/validate
```

#### 2.2.2. Visualization Engine

```yaml
service: visualization-engine
responsibility: Generate and render data visualizations
technology:
  language: TypeScript/Python
  framework: Node.js/FastAPI
  rendering: D3.js, Plotly, Apache ECharts
  cache: Redis, CDN
features:
  - 50+ chart types
  - Custom visualization plugins
  - Server-side rendering
  - Interactive visualizations
  - Export capabilities (PNG, SVG, PDF)
architecture:
  components:
    - Chart Registry: Pluggable chart types
    - Render Farm: Parallel rendering
    - Style Engine: Theming and customization
    - Export Service: Multiple format support
endpoints:
  - POST /visualize/render
  - GET /visualize/types
  - POST /visualize/export
  - GET /visualize/preview/{id}
```

#### 2.2.3. AI Service

```yaml
service: ai-service
responsibility: AI-powered features and integrations
technology:
  language: Python
  framework: FastAPI
  ml_frameworks: LangChain, Transformers
  vector_db: Pinecone/Weaviate
  gpu: CUDA-enabled workers
features:
  - Natural language to query
  - Automated insights
  - Data cleaning suggestions
  - Chart recommendations
  - Narrative generation
architecture:
  components:
    - LLM Gateway: Multi-provider support
    - Prompt Manager: Template management
    - Context Builder: Efficient context creation
    - Response Validator: Output verification
    - Cost Tracker: Usage monitoring
endpoints:
  - POST /ai/analyze
  - POST /ai/generate-query
  - POST /ai/suggest-visualization
  - POST /ai/explain
  - GET /ai/usage
```

#### 2.2.4. Transform Service

```yaml
service: transform-service
responsibility: Data transformation and ETL pipelines
technology:
  language: Python
  framework: FastAPI
  processing: Apache Beam/Pandas
  scheduler: Airflow/Temporal
features:
  - Visual ETL builder
  - Scheduled pipelines
  - Data quality checks
  - Incremental processing
  - Error recovery
architecture:
  components:
    - Pipeline Engine: DAG execution
    - Transform Registry: Reusable transforms
    - Scheduler: Cron and event triggers
    - Monitor: Pipeline health tracking
endpoints:
  - POST /transform/create
  - POST /transform/execute/{id}
  - GET /transform/status/{id}
  - POST /transform/schedule
  - GET /transform/history/{id}
```

#### 2.2.5. Pipeline Service

```yaml
service: pipeline-service
responsibility: Automated ETL pipeline orchestration and management
technology:
  language: Python
  framework: FastAPI
  orchestrator: Apache Airflow
  queue: RabbitMQ/AWS SQS
  database: PostgreSQL
  cache: Redis
features:
  - Visual pipeline builder
  - Multi-step workflows
  - Event-triggered pipelines
  - Data quality validation
  - Automatic retry mechanisms
  - Pipeline versioning
  - Performance monitoring
  - Cost optimization
architecture:
  components:
    - Workflow Engine: DAG orchestration via Airflow
    - Pipeline Builder: Visual drag-and-drop interface
    - Scheduler: Time and event-based triggers
    - Quality Monitor: Data validation and alerts
    - Resource Manager: Compute resource allocation
    - Lineage Tracker: Data flow documentation
    - Cost Optimizer: Resource usage optimization
endpoints:
  - POST /pipelines/create
  - GET /pipelines
  - PUT /pipelines/{id}
  - POST /pipelines/{id}/run
  - GET /pipelines/{id}/status
  - GET /pipelines/{id}/logs
  - POST /pipelines/{id}/schedule
  - GET /pipelines/{id}/metrics
integrations:
  - transform-service: Data transformation logic
  - data-sources: Source system connections
  - notification-service: Pipeline alerts
  - audit-service: Compliance logging
```

#### 2.2.6. Search & Discovery Service

```yaml
service: search-discovery-service
responsibility: Full-text search and content discovery across platform
technology:
  language: Python
  framework: FastAPI
  search_engine: Elasticsearch/OpenSearch
  database: PostgreSQL
  cache: Redis
features:
  - Full-text search across all content
  - Semantic search capabilities
  - Data lineage discovery
  - Content recommendations
  - Auto-completion
  - Search analytics
  - Faceted search
  - Real-time indexing
architecture:
  components:
    - Search Engine: Elasticsearch cluster
    - Indexer: Real-time content indexing
    - Query Processor: Search query optimization
    - Recommendation Engine: ML-based suggestions
    - Analytics Collector: Search behavior tracking
    - Facet Manager: Dynamic filter generation
endpoints:
  - POST /search/query
  - GET /search/suggestions
  - POST /search/index
  - GET /search/lineage/{id}
  - GET /search/recommendations
  - GET /search/analytics
integrations:
  - all-services: Content indexing
  - ai-service: Semantic search enhancement
  - user-service: Personalized results
  - analytics-service: Search metrics
```

### 2.3. Collaboration Services

#### 2.3.1. Real-time Sync Service

```yaml
service: realtime-sync-service
responsibility: Real-time collaboration and synchronization
technology:
  language: Node.js/Go
  framework: Socket.io/WebSocket
  pubsub: Redis Pub/Sub
  state: Redis/MongoDB
features:
  - Collaborative editing
  - Presence indicators
  - Cursor tracking
  - Conflict resolution
  - Offline sync
architecture:
  components:
    - WebSocket Manager: Connection handling
    - State Synchronizer: CRDT/OT implementation
    - Presence Tracker: User activity
    - Event Broadcaster: Change propagation
protocols:
  - WebSocket for real-time
  - Server-Sent Events for fallback
  - Long polling for compatibility
```

#### 2.3.2. Comment Service

```yaml
service: comment-service
responsibility: Comments and discussions
technology:
  language: Python
  framework: FastAPI
  database: PostgreSQL
  cache: Redis
features:
  - Threaded discussions
  - @mentions
  - Rich text support
  - Attachments
  - Reactions
endpoints:
  - POST /comments
  - GET /comments/{canvas_id}
  - PUT /comments/{id}
  - DELETE /comments/{id}
  - POST /comments/{id}/reactions
```

#### 2.3.3. Notification Service

```yaml
service: notification-service
responsibility: Multi-channel notifications
technology:
  language: Python
  framework: FastAPI
  queue: RabbitMQ/AWS SQS
  database: PostgreSQL
features:
  - In-app notifications
  - Email notifications
  - Push notifications
  - SMS (optional)
  - Slack/Teams integration
architecture:
  components:
    - Notification Queue: Priority handling
    - Template Engine: Multi-language support
    - Delivery Manager: Channel routing
    - Preference Manager: User settings
endpoints:
  - POST /notifications/send
  - GET /notifications/user/{id}
  - PUT /notifications/read
  - GET /notifications/preferences
  - PUT /notifications/preferences
```

#### 2.3.4. Version Control Service

```yaml
service: version-control-service
responsibility: Advanced version control with branching and merging
technology:
  language: Go
  framework: Gin
  database: PostgreSQL
  blob_storage: S3/Azure Blob
  cache: Redis
features:
  - Git-like branching model
  - Merge conflict resolution
  - Advanced diff algorithms
  - Rollback orchestration
  - Version tagging and milestones
  - Audit trails
  - Binary diff for large files
  - Collaborative merge workflows
architecture:
  components:
    - Version Engine: Core versioning logic
    - Branch Manager: Branch lifecycle management
    - Merge Engine: Conflict resolution algorithms
    - Diff Processor: Content comparison
    - Blob Manager: Large file versioning
    - Timeline Builder: Version history visualization
    - Conflict Resolver: Interactive merge tools
endpoints:
  - POST /versions/create
  - GET /versions/{resource_id}
  - POST /versions/{id}/branches
  - POST /versions/merge
  - GET /versions/{id}/diff
  - POST /versions/{id}/rollback
  - GET /versions/{id}/conflicts
  - PUT /versions/{id}/resolve
integrations:
  - all-services: Resource version tracking
  - real-time-sync: Collaborative version control
  - notification-service: Version change alerts
  - audit-service: Change tracking
```

#### 2.3.5. Message Queue Service

```yaml
service: message-queue-service
responsibility: Distributed message queuing and event streaming
technology:
  language: Go
  message_broker: RabbitMQ/Apache Kafka
  database: PostgreSQL (metadata)
  monitoring: Prometheus
features:
  - Event-driven architecture
  - Message routing and filtering
  - Dead letter queues
  - Message persistence
  - Scalable consumers
  - Exactly-once delivery
  - Message replay capabilities
  - Cross-service communication
architecture:
  components:
    - Message Broker: RabbitMQ/Kafka clusters
    - Router: Intelligent message routing
    - Consumer Manager: Auto-scaling consumers
    - DLQ Handler: Failed message processing
    - Replay Engine: Message history replay
    - Schema Registry: Message format validation
endpoints:
  - POST /messages/publish
  - GET /messages/consume
  - POST /messages/subscribe
  - GET /messages/status
  - POST /messages/replay
  - GET /messages/metrics
integrations:
  - pipeline-service: ETL coordination
  - real-time-sync: Live collaboration events
  - notification-service: Async notifications
  - all-services: Inter-service communication
```

## 3. Data Architecture

### 3.1. Data Storage Strategy

```yaml
storage_tiers:
  hot_data:
    description: Frequently accessed data
    storage: Redis, In-memory cache
    ttl: 1-24 hours
    examples:
      - Active query results
      - User sessions
      - Real-time collaboration state

  warm_data:
    description: Recent operational data
    storage: PostgreSQL, MongoDB
    retention: 30-90 days
    examples:
      - Recent projects
      - Canvas content
      - User activity

  cold_data:
    description: Historical and archived data
    storage: S3/Blob Storage
    retention: 1+ years
    examples:
      - Archived projects
      - Historical analytics
      - Compliance logs

  analytical_data:
    description: Optimized for analytics
    storage: ClickHouse/BigQuery
    retention: 1 year
    examples:
      - Aggregated metrics
      - Usage analytics
      - Performance data
```

### 3.2. Data Flow Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Ingestion │────▶│  Processing │────▶│   Storage   │────▶│   Serving   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
      │                    │                    │                    │
      ▼                    ▼                    ▼                    ▼
 • File Upload      • Validation        • PostgreSQL        • Query API
 • API Import       • Transformation    • MongoDB           • Cache Layer
 • Direct Connect   • Enrichment        • Object Storage    • CDN
 • Streaming        • Aggregation       • Analytics DB      • Export API
```

### 3.3. Database Design

For detailed database schemas and data models, see:

- [Data Model Specification](./05_data_model.md)
- [API Specifications](./06_api_specifications.md)

## 4. Security Architecture

### 4.1. Security Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                          Edge Security                           │
│  • DDoS Protection  • WAF Rules  • SSL/TLS  • Bot Protection   │
└─────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────┐
│                      Application Security                        │
│  • JWT Authentication  • OAuth 2.0  • API Keys  • Rate Limiting │
└─────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────┐
│                         Service Security                         │
│  • mTLS  • Service Mesh  • Network Policies  • RBAC            │
└─────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────┐
│                          Data Security                           │
│  • Encryption at Rest  • Encryption in Transit  • Key Management│
└─────────────────────────────────────────────────────────────────┘
```

### 4.2. Authentication & Authorization Flow

```python
# Authentication Flow
class AuthenticationFlow:
    """
    1. User provides credentials (username/password, OAuth, SSO)
    2. Auth Service validates credentials
    3. Generate JWT with claims
    4. Return access_token and refresh_token
    5. Client includes token in Authorization header
    6. API Gateway validates token
    7. Service receives validated user context
    """

    async def authenticate(self, credentials: Credentials) -> TokenPair:
        # Validate credentials
        user = await self.validate_credentials(credentials)

        # Generate tokens
        access_token = self.generate_jwt(
            user_id=user.id,
            organization_id=user.organization_id,
            roles=user.roles,
            expires_in=timedelta(hours=1)
        )

        refresh_token = self.generate_refresh_token(
            user_id=user.id,
            expires_in=timedelta(days=30)
        )

        # Store refresh token
        await self.store_refresh_token(user.id, refresh_token)

        return TokenPair(
            access_token=access_token,
            refresh_token=refresh_token,
            expires_in=3600
        )

# Authorization Flow
class AuthorizationFlow:
    """
    1. Extract user context from validated JWT
    2. Load user permissions from cache/database
    3. Check resource ownership
    4. Apply RBAC rules
    5. Apply attribute-based access control (ABAC)
    6. Log access decision
    """

    async def authorize(
        self,
        user_context: UserContext,
        resource: Resource,
        action: Action
    ) -> bool:
        # Check organization membership
        if not await self.check_organization(user_context, resource):
            return False

        # Check resource permissions
        permissions = await self.get_permissions(user_context, resource)

        # Apply RBAC
        if not self.check_rbac(user_context.roles, action):
            return False

        # Apply ABAC
        if not self.check_abac(user_context, resource, action):
            return False

        # Log decision
        await self.audit_log(user_context, resource, action, "granted")

        return True
```

### 4.3. Data Protection

```yaml
encryption:
  at_rest:
    databases:
      algorithm: AES-256-GCM
      key_management: AWS KMS / Azure Key Vault
      rotation: 90 days

    file_storage:
      algorithm: AES-256-GCM
      client_side: Optional for sensitive data
      server_side: Always enabled

  in_transit:
    external:
      protocol: TLS 1.3
      cipher_suites: [TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256]
      certificate: Let's Encrypt / ACM

    internal:
      service_mesh: mTLS via Istio/Linkerd
      database: TLS with certificate validation

data_masking:
  pii_detection:
    - Email addresses
    - Phone numbers
    - SSN/Tax IDs
    - Credit card numbers

  masking_rules:
    display: Partial masking (show last 4 digits)
    export: Full masking or removal
    ai_processing: Anonymization

compliance:
  gdpr:
    - Right to be forgotten
    - Data portability
    - Consent management
    - Privacy by design

  soc2:
    - Access controls
    - Audit logging
    - Incident response
    - Business continuity
```

## 5. Infrastructure Architecture

### 5.1. Container Orchestration

```yaml
kubernetes:
  cluster_topology:
    production:
      regions: [us-east-1, eu-west-1, ap-southeast-1]
      nodes_per_region: 10-50 (auto-scaling)
      node_types:
        - compute_optimized: API services
        - memory_optimized: Query processing
        - gpu_enabled: AI workloads

  namespaces:
    - core-services
    - analytics-services
    - collaboration-services
    - monitoring
    - ingress

  resource_management:
    horizontal_pod_autoscaler:
      min_replicas: 2
      max_replicas: 100
      target_cpu: 70%
      target_memory: 80%

    vertical_pod_autoscaler:
      enabled: true
      update_mode: Auto

    pod_disruption_budget:
      min_available: 50%

  networking:
    service_mesh: Istio
    ingress: Nginx Ingress Controller
    network_policies: Calico
    load_balancing: Layer 7 with session affinity
```

### 5.2. CI/CD Pipeline

```yaml
pipeline:
  source_control:
    platform: GitHub/GitLab
    branching_strategy: GitFlow
    protection_rules:
      - main: Requires 2 approvals
      - develop: Requires 1 approval

  stages:
    - name: build
      steps:
        - lint_code
        - run_unit_tests
        - security_scan (SAST)
        - build_containers
        - scan_containers

    - name: test
      steps:
        - integration_tests
        - api_tests
        - performance_tests
        - security_tests (DAST)
        - chaos_engineering

    - name: deploy
      environments:
        staging:
          approval: automatic
          tests: [smoke_tests, integration_tests]
        production:
          approval: manual
          tests: [canary_deployment, full_regression]

  rollback_strategy:
    blue_green: true
    canary_percentage: 10
    automatic_rollback: true
    health_checks:
      - endpoint: /health
      - metrics: error_rate < 1%
      - duration: 5 minutes
```

## 6. Message Queue Infrastructure

### 6.1. Event-Driven Architecture

```yaml
event_architecture:
  message_broker:
    primary: Apache Kafka
    fallback: RabbitMQ
    cluster_size: 3 brokers minimum
    
  topics:
    user_events:
      partitions: 12
      replication_factor: 3
      retention: 7 days
      
    project_events:
      partitions: 24
      replication_factor: 3
      retention: 30 days
      
    canvas_events:
      partitions: 48
      replication_factor: 3
      retention: 7 days
      
    pipeline_events:
      partitions: 12
      replication_factor: 3
      retention: 90 days
      
    system_events:
      partitions: 6
      replication_factor: 3
      retention: 30 days

  consumers:
    real_time_sync:
      group_id: realtime-sync-group
      auto_offset_reset: latest
      max_poll_records: 100
      
    notification_processor:
      group_id: notification-group
      auto_offset_reset: earliest
      max_poll_records: 50
      
    analytics_collector:
      group_id: analytics-group
      auto_offset_reset: earliest
      max_poll_records: 1000
      
    pipeline_orchestrator:
      group_id: pipeline-group
      auto_offset_reset: earliest
      max_poll_records: 10

  schema_registry:
    url: http://schema-registry:8081
    compatibility: BACKWARD
    security: SSL/SASL
```

### 6.2. Event Patterns

```python
# Event Schema Examples
from typing import Dict, Any, Optional
from datetime import datetime
from enum import Enum

class EventType(Enum):
    USER_CREATED = "user.created"
    USER_UPDATED = "user.updated"
    PROJECT_CREATED = "project.created"
    PROJECT_SHARED = "project.shared"
    CANVAS_MODIFIED = "canvas.modified"
    PIPELINE_STARTED = "pipeline.started"
    PIPELINE_COMPLETED = "pipeline.completed"
    PIPELINE_FAILED = "pipeline.failed"

class BaseEvent:
    def __init__(
        self,
        event_type: EventType,
        source_service: str,
        entity_id: str,
        user_id: Optional[str] = None,
        organization_id: Optional[str] = None,
        metadata: Optional[Dict[str, Any]] = None
    ):
        self.event_id = str(uuid.uuid4())
        self.event_type = event_type
        self.source_service = source_service
        self.entity_id = entity_id
        self.user_id = user_id
        self.organization_id = organization_id
        self.timestamp = datetime.utcnow()
        self.metadata = metadata or {}

# Usage Examples
class PipelineEventProducer:
    async def pipeline_started(self, pipeline_id: str, user_id: str):
        event = BaseEvent(
            event_type=EventType.PIPELINE_STARTED,
            source_service="pipeline-service",
            entity_id=pipeline_id,
            user_id=user_id,
            metadata={
                "pipeline_name": "Daily Sales Sync",
                "estimated_duration": "5 minutes",
                "scheduled": True
            }
        )
        await self.publish_event(event)

    async def pipeline_completed(self, pipeline_id: str, metrics: Dict):
        event = BaseEvent(
            event_type=EventType.PIPELINE_COMPLETED,
            source_service="pipeline-service",
            entity_id=pipeline_id,
            metadata={
                "duration": metrics["duration"],
                "rows_processed": metrics["rows_processed"],
                "success": True
            }
        )
        await self.publish_event(event)
```

## 7. Enhanced Service Integration

### 7.1. Cross-Service Communication

```yaml
communication_patterns:
  synchronous:
    protocol: HTTP/gRPC
    timeout: 30 seconds
    retry_policy:
      max_attempts: 3
      backoff: exponential
      base_delay: 1s
    circuit_breaker:
      failure_threshold: 50%
      recovery_time: 30s
    
  asynchronous:
    protocol: Message Queue
    delivery_guarantee: at_least_once
    ordering: partition_key
    batch_size: 100
    
  streaming:
    protocol: WebSocket/Server-Sent Events
    heartbeat: 30s
    reconnect: automatic
    buffer_size: 1000

service_mesh:
  technology: Istio
  features:
    - mTLS between services
    - Traffic routing and load balancing
    - Circuit breaker patterns
    - Distributed tracing
    - Metrics collection
    - Security policies

api_gateway:
  rate_limiting:
    per_user: 1000 requests/minute
    per_api_key: 10000 requests/minute
    burst_capacity: 2x rate limit
    
  caching:
    strategy: Redis cluster
    ttl: 5 minutes (default)
    cache_keys: user_id, resource_id
    
  authentication:
    jwt_validation: true
    api_key_support: true
    oauth_integration: true
    
  monitoring:
    request_logging: enabled
    metrics_collection: enabled
    distributed_tracing: enabled
```

### 7.2. Data Flow Integration

```yaml
data_integration:
  real_time_pipeline:
    - source: User interactions
    - processing: Stream processing (Apache Kafka Streams)
    - destination: Real-time dashboards
    - latency: < 100ms
    
  batch_pipeline:
    - source: ETL pipelines
    - processing: Apache Spark
    - destination: Analytics database
    - schedule: Configurable (hourly, daily, etc.)
    
  context_pipeline:
    - source: All user interactions
    - processing: NLP and semantic analysis
    - destination: Context service
    - update_frequency: Real-time
    
  search_pipeline:
    - source: All content changes
    - processing: Elasticsearch indexing
    - destination: Search service
    - latency: < 5 seconds

project_lifecycle_integration:
  creation:
    - project-service: Create project record
    - context-service: Initialize project context
    - version-control-service: Create initial version
    - search-service: Index project metadata
    - notification-service: Notify team members
    
  collaboration:
    - real-time-sync: Coordinate live editing
    - version-control-service: Track changes
    - comment-service: Handle discussions
    - notification-service: Send updates
    
  automation:
    - pipeline-service: Execute ETL workflows
    - transform-service: Process data
    - message-queue: Coordinate tasks
    - monitoring-service: Track health
```

## 8. Frontend Architecture

### 8.1. Technology Stack

```yaml
frontend_stack:
  core_language:
    name: TypeScript
    version: "5.x"
    rationale:
      - Type safety for large-scale applications
      - Enhanced IDE support and autocompletion
      - Compile-time error detection
      - Better refactoring capabilities
      - Self-documenting code through types
      
  framework:
    name: React
    version: "18.x"
    with_typescript: true
    configuration:
      - Strict mode enabled
      - Function components with hooks
      - Concurrent features enabled
      
  build_tools:
    bundler: Vite
    type_checker: TypeScript compiler
    linter: ESLint with TypeScript parser
    formatter: Prettier
    
  ui_libraries:
    component_library: 
      name: Mantine UI
      version: "7.x"
      typescript_support: Full native TypeScript support
    styling:
      name: Tailwind CSS
      version: "3.x"
      with_typescript: TypeScript configuration for custom plugins
      
  state_management:
    primary: React Context + useReducer
    async_state: TanStack Query (React Query)
    form_state: React Hook Form with TypeScript
    
  testing:
    unit_tests: Jest with TypeScript
    component_tests: React Testing Library
    e2e_tests: Playwright with TypeScript
    type_tests: tsd (TypeScript type testing)
```

### 8.2. TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "allowSyntheticDefaultImports": true,
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@services/*": ["./src/services/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@types/*": ["./src/types/*"],
      "@utils/*": ["./src/utils/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### 8.3. Type System Architecture

```typescript
// Core type definitions structure
interface TypeArchitecture {
  // Domain models matching backend schemas
  models: {
    user: "src/types/models/user.ts";
    project: "src/types/models/project.ts";
    canvas: "src/types/models/canvas.ts";
    dataSource: "src/types/models/dataSource.ts";
  };
  
  // API types for request/response
  api: {
    requests: "src/types/api/requests.ts";
    responses: "src/types/api/responses.ts";
    errors: "src/types/api/errors.ts";
  };
  
  // UI component prop types
  components: {
    props: "src/types/components/props.ts";
    events: "src/types/components/events.ts";
    state: "src/types/components/state.ts";
  };
  
  // Utility types
  utilities: {
    helpers: "src/types/utils/helpers.ts";
    guards: "src/types/utils/guards.ts";
    branded: "src/types/utils/branded.ts";
  };
}

// Example of strict typing for API integration
interface APIClient {
  get<T>(url: string, config?: RequestConfig): Promise<APIResponse<T>>;
  post<T, D>(url: string, data: D, config?: RequestConfig): Promise<APIResponse<T>>;
  put<T, D>(url: string, data: D, config?: RequestConfig): Promise<APIResponse<T>>;
  delete<T>(url: string, config?: RequestConfig): Promise<APIResponse<T>>;
}

// Type-safe hooks
interface TypedHooks {
  useAuth(): AuthContext;
  useProject(id: ProjectId): ProjectData;
  useCanvas(id: CanvasId): CanvasState;
  useAIGeneration<T extends ChartType>(
    prompt: string,
    data: DataSource
  ): AIGenerationResult<T>;
}
```

### 8.4. Component Architecture with TypeScript

```typescript
// Strict component typing pattern
import { FC, ReactNode } from 'react';
import { MantineSize, MantineColor } from '@mantine/core';

// Base component props with common properties
interface BaseComponentProps {
  className?: string;
  children?: ReactNode;
  testId?: string;
}

// Example of typed component with Mantine integration
interface ButtonProps extends BaseComponentProps {
  variant?: 'filled' | 'outline' | 'ghost';
  size?: MantineSize;
  color?: MantineColor;
  loading?: boolean;
  disabled?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  type?: 'button' | 'submit' | 'reset';
}

export const Button: FC<ButtonProps> = ({
  variant = 'filled',
  size = 'md',
  color = 'primary',
  loading = false,
  disabled = false,
  onClick,
  type = 'button',
  className,
  children,
  testId,
}) => {
  // Implementation with full type safety
};

// Typed context pattern
interface AppContextValue {
  user: User | null;
  theme: Theme;
  preferences: UserPreferences;
  actions: {
    updateUser: (user: User) => void;
    updateTheme: (theme: Theme) => void;
    updatePreferences: (preferences: Partial<UserPreferences>) => void;
  };
}

const AppContext = createContext<AppContextValue | undefined>(undefined);

export const useAppContext = (): AppContextValue => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within AppProvider');
  }
  return context;
};
```

### 8.5. Type-Safe Data Flow

```yaml
data_flow_typing:
  api_layer:
    - Generated TypeScript types from OpenAPI spec
    - Runtime validation with Zod schemas
    - Type guards for response validation
    
  state_management:
    - Discriminated unions for state machines
    - Immutable state updates with Immer
    - Type-safe action creators
    
  component_props:
    - Strict prop types for all components
    - Generic components with type parameters
    - Event handler typing
    
  form_handling:
    - Typed form schemas with Zod
    - Type-safe form state with React Hook Form
    - Validation error typing
    
  routing:
    - Typed route parameters
    - Type-safe navigation hooks
    - Route guard typing
```

### 8.6. Code Quality Standards

```yaml
typescript_standards:
  strict_mode:
    - No implicit any
    - Strict null checks
    - Strict function types
    - No unused variables/parameters
    
  naming_conventions:
    - Interfaces: PascalCase with 'I' prefix optional
    - Types: PascalCase
    - Enums: PascalCase with singular names
    - Constants: UPPER_SNAKE_CASE
    
  file_organization:
    - One component per file
    - Co-locate types with components
    - Separate type definition files for shared types
    - Index files for clean exports
    
  documentation:
    - JSDoc comments for public APIs
    - Inline comments for complex logic
    - Type documentation for complex types
    - README files for major features
```

### 8.7. Development Workflow

```yaml
typescript_workflow:
  pre_commit:
    - TypeScript compilation check
    - ESLint with TypeScript rules
    - Prettier formatting
    - Unit test execution
    
  ci_pipeline:
    - Type checking across entire codebase
    - Strict build with no warnings
    - Bundle size analysis
    - Type coverage reporting
    
  ide_setup:
    - VS Code with TypeScript extensions
    - Real-time type checking
    - Auto-imports and refactoring
    - Integrated debugging
```
