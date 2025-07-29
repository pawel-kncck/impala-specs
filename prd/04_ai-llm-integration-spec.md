# AI/LLM Integration Specification

## 1. Overview

The AI/LLM integration is central to Impala's value proposition, enabling natural language interactions for data analysis, automated insights generation, and intelligent data preparation. This specification defines how AI capabilities are integrated throughout the application while maintaining transparency, accuracy, and cost-effectiveness.

## 2. Architecture Overview

### 2.1. AI Service Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Frontend UI   │────▶│   Backend API    │────▶│   AI Service    │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                           │
                              ┌────────────────────────────┼────────────────────────────┐
                              │                            ▼                            │
                        ┌─────┴─────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┴────┐
                        │ Provider  │  │   Prompt     │  │   Response   │  │   Context    │
                        │ Adapter   │  │  Templates   │  │  Validator   │  │   Manager    │
                        └─────┬─────┘  └──────────────┘  └──────────────┘  └──────────────┘
                              │
                ┌─────────────┼─────────────┬─────────────┬─────────────┐
                ▼             ▼             ▼             ▼             ▼
          ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
          │  OpenAI  │  │Anthropic │  │  Google  │  │  Local   │  │  Custom  │
          │   API    │  │   API    │  │  Vertex  │  │  (Llama) │  │  Models  │
          └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

### 2.2. Core Components

1. **Provider Adapter Layer**: Abstracts different LLM providers behind a common interface
2. **Prompt Template Engine**: Manages and optimizes prompts for different use cases
3. **Response Validator**: Ensures AI outputs are accurate and properly formatted
4. **Context Manager**: Handles conversation state and data context efficiently
5. **Cost Optimizer**: Tracks and optimizes token usage across requests

## 3. Provider Integration

### 3.1. Multi-Provider Strategy

```python
# Provider configuration schema
{
    "providers": {
        "primary": {
            "name": "openai",
            "model": "gpt-4-turbo-preview",
            "api_key": "${OPENAI_API_KEY}",
            "max_tokens": 4096,
            "temperature": 0.1
        },
        "fallback": {
            "name": "anthropic",
            "model": "claude-3-sonnet",
            "api_key": "${ANTHROPIC_API_KEY}",
            "max_tokens": 4096
        },
        "specialized": {
            "code_generation": {
                "name": "openai",
                "model": "gpt-4-turbo-preview",
                "temperature": 0.0
            },
            "data_analysis": {
                "name": "openai",
                "model": "gpt-4-turbo-preview",
                "temperature": 0.2
            }
        }
    }
}
```

### 3.2. Provider Adapter Interface

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Optional, AsyncGenerator

class LLMProvider(ABC):
    @abstractmethod
    async def complete(
        self,
        messages: List[Dict[str, str]],
        max_tokens: int = 2048,
        temperature: float = 0.1,
        stream: bool = False
    ) -> Union[str, AsyncGenerator[str, None]]:
        """Generate completion from messages"""
        pass
    
    @abstractmethod
    async def embed(self, text: str) -> List[float]:
        """Generate embeddings for text"""
        pass
    
    @abstractmethod
    def count_tokens(self, text: str) -> int:
        """Count tokens in text"""
        pass
    
    @abstractmethod
    def get_cost(self, input_tokens: int, output_tokens: int) -> float:
        """Calculate cost for token usage"""
        pass
```

### 3.3. Fallback and Load Balancing

```python
class AIServiceManager:
    def __init__(self):
        self.providers = self._initialize_providers()
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            recovery_timeout=60
        )
    
    async def get_completion(self, request: CompletionRequest) -> CompletionResponse:
        for provider in self._get_provider_chain(request.use_case):
            try:
                if self.circuit_breaker.is_open(provider.name):
                    continue
                
                response = await provider.complete(
                    messages=request.messages,
                    max_tokens=request.max_tokens
                )
                
                self.circuit_breaker.record_success(provider.name)
                return response
                
            except ProviderError as e:
                self.circuit_breaker.record_failure(provider.name)
                logger.error(f"Provider {provider.name} failed: {e}")
                continue
        
        raise AllProvidersFailedError("All AI providers are unavailable")
```

## 4. Use Case Implementations

### 4.1. Data Profiling and Cleaning

```python
class DataProfilingAI:
    """AI-powered data profiling and cleaning suggestions"""
    
    async def analyze_dataset(self, df: pd.DataFrame) -> DataProfile:
        # Generate compact data summary
        summary = self._generate_summary(df)
        
        prompt = PromptTemplate.load("data_profiling")
        messages = prompt.format(
            data_summary=summary,
            sample_rows=df.head(10).to_dict(),
            column_info=self._get_column_info(df)
        )
        
        response = await self.ai_service.complete(
            messages=messages,
            response_format=DataProfileSchema
        )
        
        return DataProfile.parse(response)
    
    def _generate_summary(self, df: pd.DataFrame) -> Dict:
        return {
            "shape": df.shape,
            "columns": df.columns.tolist(),
            "dtypes": df.dtypes.to_dict(),
            "missing_values": df.isnull().sum().to_dict(),
            "unique_counts": {col: df[col].nunique() for col in df.columns}
        }
```

### 4.2. Natural Language to Query Translation

```python
class NLToQueryTranslator:
    """Translate natural language to SQL/Python queries"""
    
    async def translate(
        self,
        question: str,
        schema: DataSchema,
        context: Optional[AnalysisContext] = None
    ) -> QueryResult:
        # Build context with schema information
        schema_context = self._build_schema_context(schema)
        
        # Use specialized prompt for query generation
        prompt = PromptTemplate.load("nl_to_query")
        messages = prompt.format(
            question=question,
            schema=schema_context,
            context=context,
            examples=self._get_relevant_examples(question)
        )
        
        # Generate query with validation
        response = await self.ai_service.complete(
            messages=messages,
            temperature=0.0,  # Deterministic for code
            response_format=QueryResponseSchema
        )
        
        # Validate generated query
        validated_query = await self._validate_query(
            response.query,
            schema
        )
        
        return QueryResult(
            query=validated_query,
            explanation=response.explanation,
            confidence=response.confidence
        )
    
    async def _validate_query(self, query: str, schema: DataSchema) -> str:
        """Validate and sanitize generated queries"""
        # Parse query to AST
        # Check for dangerous operations
        # Verify column references
        # Return sanitized query
        pass
```

### 4.3. Chart Generation

```python
class ChartGenerationAI:
    """AI-powered chart recommendations and generation"""
    
    async def generate_chart(
        self,
        request: str,
        data: pd.DataFrame,
        context: CanvasContext
    ) -> ChartSpecification:
        # Analyze data characteristics
        data_profile = self._profile_data(data)
        
        # Generate chart specification
        prompt = PromptTemplate.load("chart_generation")
        messages = prompt.format(
            request=request,
            data_profile=data_profile,
            available_columns=data.columns.tolist(),
            data_types=data.dtypes.to_dict(),
            canvas_context=context
        )
        
        response = await self.ai_service.complete(
            messages=messages,
            response_format=ChartSpecSchema
        )
        
        # Validate and enhance specification
        chart_spec = self._validate_chart_spec(response, data)
        
        # Add intelligent defaults
        chart_spec = self._apply_smart_defaults(chart_spec, data)
        
        return chart_spec
```

### 4.4. Narrative Generation

```python
class NarrativeGenerationAI:
    """Generate insights and narratives from data"""
    
    async def generate_narrative(
        self,
        visualization: Visualization,
        data: pd.DataFrame,
        context: AnalysisContext
    ) -> Narrative:
        # Extract key statistics
        stats = self._extract_statistics(visualization, data)
        
        # Identify patterns and insights
        patterns = self._identify_patterns(data, visualization.type)
        
        prompt = PromptTemplate.load("narrative_generation")
        messages = prompt.format(
            visualization_type=visualization.type,
            statistics=stats,
            patterns=patterns,
            context=context,
            tone="professional"
        )
        
        response = await self.ai_service.complete(
            messages=messages,
            max_tokens=500
        )
        
        # Parse and link narrative elements
        narrative = self._parse_narrative(response)
        narrative = self._add_data_links(narrative, stats)
        
        return narrative
```

## 5. Prompt Engineering

### 5.1. Prompt Template Structure

```yaml
# prompt_templates/data_profiling.yaml
name: data_profiling
version: 1.2
model_requirements:
  min_context: 8192
  preferred_models: ["gpt-4", "claude-3"]

system_prompt: |
  You are a data analysis expert helping users understand their datasets.
  Analyze the provided data and suggest:
  1. Data quality issues and cleaning steps
  2. Potential data types for each column
  3. Interesting patterns or anomalies
  4. Recommended transformations

messages:
  - role: system
    content: "{{ system_prompt }}"
  - role: user
    content: |
      Dataset Summary:
      {{ data_summary | json }}
      
      Sample Rows:
      {{ sample_rows | format_table }}
      
      Please provide a comprehensive analysis.

response_format:
  type: json_schema
  schema:
    type: object
    properties:
      data_types:
        type: object
        additionalProperties:
          type: string
          enum: ["number", "string", "date", "boolean", "category"]
      quality_issues:
        type: array
        items:
          type: object
          properties:
            column: string
            issue: string
            severity: string
            suggestion: string
      transformations:
        type: array
        items:
          type: object
          properties:
            type: string
            description: string
            columns: array
```

### 5.2. Context Window Management

```python
class ContextWindowManager:
    """Manage context to fit within token limits"""
    
    def __init__(self, max_tokens: int = 8192):
        self.max_tokens = max_tokens
        self.token_counter = TokenCounter()
    
    def optimize_context(
        self,
        messages: List[Message],
        data_context: Dict,
        priority: str = "recent"
    ) -> List[Message]:
        # Calculate current token usage
        base_tokens = self.token_counter.count_messages(messages)
        
        # Prioritize context elements
        context_elements = self._prioritize_context(data_context, priority)
        
        # Add context within token budget
        optimized_messages = messages.copy()
        remaining_tokens = self.max_tokens - base_tokens - 1000  # Reserve
        
        for element in context_elements:
            element_tokens = self.token_counter.count(element.content)
            if element_tokens <= remaining_tokens:
                optimized_messages = self._inject_context(
                    optimized_messages,
                    element
                )
                remaining_tokens -= element_tokens
        
        return optimized_messages
```

## 6. Response Validation

### 6.1. Validation Framework

```python
class ResponseValidator:
    """Validate AI responses for accuracy and safety"""
    
    async def validate_code(self, code: str, schema: DataSchema) -> ValidationResult:
        # Static analysis
        static_issues = await self._static_analysis(code)
        if static_issues.critical:
            return ValidationResult(valid=False, issues=static_issues)
        
        # Sandbox execution
        sandbox_result = await self._sandbox_execute(code, schema)
        if not sandbox_result.success:
            return ValidationResult(valid=False, error=sandbox_result.error)
        
        # Security checks
        security_issues = self._security_scan(code)
        if security_issues:
            return ValidationResult(valid=False, security_issues=security_issues)
        
        return ValidationResult(valid=True, warnings=static_issues.warnings)
    
    def validate_chart_spec(self, spec: Dict, data: pd.DataFrame) -> ValidationResult:
        # Verify column references
        invalid_columns = self._check_column_references(spec, data.columns)
        if invalid_columns:
            return ValidationResult(
                valid=False,
                error=f"Invalid columns: {invalid_columns}"
            )
        
        # Validate aggregations
        if not self._validate_aggregations(spec, data):
            return ValidationResult(
                valid=False,
                error="Invalid aggregation specification"
            )
        
        # Check chart type compatibility
        if not self._check_chart_compatibility(spec, data):
            return ValidationResult(
                valid=False,
                error="Incompatible chart type for data"
            )
        
        return ValidationResult(valid=True)
```

### 6.2. Hallucination Detection

```python
class HallucinationDetector:
    """Detect and prevent AI hallucinations in data analysis"""
    
    def check_factual_accuracy(
        self,
        narrative: str,
        source_data: pd.DataFrame,
        statistics: Dict
    ) -> List[FactualIssue]:
        issues = []
        
        # Extract claims from narrative
        claims = self._extract_claims(narrative)
        
        for claim in claims:
            # Verify numerical claims
            if claim.type == "numerical":
                actual_value = self._calculate_value(claim.expression, source_data)
                if not self._values_match(claim.value, actual_value):
                    issues.append(FactualIssue(
                        claim=claim,
                        expected=claim.value,
                        actual=actual_value
                    ))
            
            # Verify statistical claims
            elif claim.type == "statistical":
                if not self._verify_statistical_claim(claim, statistics):
                    issues.append(FactualIssue(
                        claim=claim,
                        type="unverifiable_statistic"
                    ))
        
        return issues
```

## 7. Cost Management

### 7.1. Token Usage Tracking

```python
class TokenUsageTracker:
    """Track and optimize token usage across requests"""
    
    def __init__(self):
        self.usage_db = TokenUsageDB()
        self.cost_calculator = CostCalculator()
    
    async def track_request(
        self,
        request_id: str,
        provider: str,
        model: str,
        input_tokens: int,
        output_tokens: int,
        user_id: str,
        use_case: str
    ):
        cost = self.cost_calculator.calculate(
            provider=provider,
            model=model,
            input_tokens=input_tokens,
            output_tokens=output_tokens
        )
        
        await self.usage_db.record(
            request_id=request_id,
            user_id=user_id,
            provider=provider,
            model=model,
            input_tokens=input_tokens,
            output_tokens=output_tokens,
            cost=cost,
            use_case=use_case,
            timestamp=datetime.utcnow()
        )
        
        # Check usage limits
        await self._check_usage_limits(user_id)
```

### 7.2. Cost Optimization Strategies

```python
class CostOptimizer:
    """Optimize AI usage for cost efficiency"""
    
    def select_model(self, request: AIRequest) -> ModelSelection:
        # Simple requests use cheaper models
        if request.complexity == "simple":
            return ModelSelection(
                provider="openai",
                model="gpt-3.5-turbo",
                max_tokens=1000
            )
        
        # Complex analysis uses more capable models
        elif request.complexity == "complex":
            return ModelSelection(
                provider="openai",
                model="gpt-4-turbo-preview",
                max_tokens=4000
            )
        
        # Code generation requires high accuracy
        elif request.use_case == "code_generation":
            return ModelSelection(
                provider="openai",
                model="gpt-4-turbo-preview",
                temperature=0.0,
                max_tokens=2000
            )
    
    def optimize_prompt(self, prompt: str, target_reduction: float = 0.2) -> str:
        # Remove redundancy
        prompt = self._remove_redundancy(prompt)
        
        # Compress examples
        prompt = self._compress_examples(prompt)
        
        # Use references for repeated content
        prompt = self._use_references(prompt)
        
        return prompt
```

## 8. Caching Strategy

### 8.1. Response Caching

```python
class AIResponseCache:
    """Cache AI responses for common queries"""
    
    def __init__(self, ttl_seconds: int = 3600):
        self.cache = Redis()
        self.ttl = ttl_seconds
    
    async def get_or_generate(
        self,
        cache_key: str,
        generator_func: Callable,
        ttl_override: Optional[int] = None
    ) -> Any:
        # Check cache
        cached = await self.cache.get(cache_key)
        if cached:
            return json.loads(cached)
        
        # Generate new response
        response = await generator_func()
        
        # Cache response
        ttl = ttl_override or self.ttl
        await self.cache.setex(
            cache_key,
            ttl,
            json.dumps(response)
        )
        
        return response
    
    def generate_cache_key(
        self,
        use_case: str,
        params: Dict,
        data_hash: str
    ) -> str:
        # Create deterministic cache key
        param_str = json.dumps(params, sort_keys=True)
        return f"ai:{use_case}:{data_hash}:{hashlib.md5(param_str.encode()).hexdigest()}"
```

### 8.2. Embedding Cache

```python
class EmbeddingCache:
    """Cache embeddings for repeated content"""
    
    def __init__(self):
        self.vector_db = ChromaDB()
    
    async def get_or_create_embedding(
        self,
        text: str,
        model: str = "text-embedding-ada-002"
    ) -> List[float]:
        # Check if embedding exists
        text_hash = self._hash_text(text)
        existing = await self.vector_db.get(text_hash)
        
        if existing:
            return existing.embedding
        
        # Generate new embedding
        embedding = await self.ai_service.embed(text, model)
        
        # Store in vector DB
        await self.vector_db.upsert(
            id=text_hash,
            embedding=embedding,
            metadata={"text": text, "model": model}
        )
        
        return embedding
```

## 9. Error Handling

### 9.1. Graceful Degradation

```python
class AIErrorHandler:
    """Handle AI service errors gracefully"""
    
    async def handle_with_fallback(
        self,
        primary_func: Callable,
        fallback_func: Callable,
        error_message: str
    ) -> Any:
        try:
            return await primary_func()
        except AIServiceError as e:
            logger.warning(f"AI service error: {e}")
            
            # Try fallback
            try:
                return await fallback_func()
            except Exception as fallback_error:
                logger.error(f"Fallback also failed: {fallback_error}")
                
                # Return user-friendly error
                raise UserFacingError(
                    message=error_message,
                    suggestion="Try simplifying your request or try again later"
                )
```

### 9.2. Timeout Management

```python
class AITimeoutManager:
    """Manage timeouts for AI requests"""
    
    def __init__(self, default_timeout: int = 30):
        self.default_timeout = default_timeout
    
    async def execute_with_timeout(
        self,
        func: Callable,
        timeout: Optional[int] = None,
        stream: bool = False
    ) -> Any:
        timeout = timeout or self.default_timeout
        
        if stream:
            return self._stream_with_timeout(func, timeout)
        else:
            try:
                return await asyncio.wait_for(func(), timeout=timeout)
            except asyncio.TimeoutError:
                raise AITimeoutError(
                    f"AI request timed out after {timeout} seconds"
                )
```

## 10. Monitoring and Analytics

### 10.1. Performance Metrics

```python
@dataclass
class AIMetrics:
    request_id: str
    use_case: str
    provider: str
    model: str
    latency_ms: float
    input_tokens: int
    output_tokens: int
    cost: float
    success: bool
    error_type: Optional[str] = None
    
class AIMetricsCollector:
    """Collect and analyze AI performance metrics"""
    
    async def record_metric(self, metric: AIMetrics):
        # Store in time-series DB
        await self.influx_db.write(
            measurement="ai_requests",
            tags={
                "use_case": metric.use_case,
                "provider": metric.provider,
                "model": metric.model,
                "success": metric.success
            },
            fields={
                "latency_ms": metric.latency_ms,
                "input_tokens": metric.input_tokens,
                "output_tokens": metric.output_tokens,
                "cost": metric.cost
            }
        )
        
        # Update real-time dashboards
        await self._update_dashboards(metric)
```

### 10.2. Quality Monitoring

```python
class AIQualityMonitor:
    """Monitor AI response quality"""
    
    async def evaluate_response(
        self,
        request: AIRequest,
        response: AIResponse,
        user_feedback: Optional[UserFeedback] = None
    ) -> QualityScore:
        scores = {}
        
        # Automated quality checks
        scores['format_compliance'] = self._check_format(response)
        scores['factual_accuracy'] = await self._check_accuracy(response)
        scores['relevance'] = self._check_relevance(request, response)
        
        # User feedback integration
        if user_feedback:
            scores['user_satisfaction'] = user_feedback.rating
        
        # Calculate composite score
        quality_score = self._calculate_composite_score(scores)
        
        # Store for analysis
        await self._store_quality_metrics(
            request_id=request.id,
            scores=scores,
            composite_score=quality_score
        )
        
        return quality_score
```

## 11. Configuration Management

### 11.1. Environment Configuration

```yaml
# config/ai_config.yaml
development:
  providers:
    openai:
      api_key: ${OPENAI_API_KEY}
      organization: ${OPENAI_ORG}
      base_url: https://api.openai.com/v1
      timeout: 30
      retry_attempts: 3
      retry_delay: 1
  
  rate_limits:
    requests_per_minute: 60
    tokens_per_minute: 90000
    concurrent_requests: 10
  
  caching:
    enabled: true
    ttl_seconds: 3600
    max_cache_size_mb: 1000

production:
  providers:
    openai:
      api_key: ${OPENAI_API_KEY}
      organization: ${OPENAI_ORG}
      base_url: https://api.openai.com/v1
      timeout: 60
      retry_attempts: 5
      retry_delay: 2
    
    anthropic:
      api_key: ${ANTHROPIC_API_KEY}
      base_url: https://api.anthropic.com/v1
      timeout: 60
  
  rate_limits:
    requests_per_minute: 3000
    tokens_per_minute: 1000000
    concurrent_requests: 100
  
  caching:
    enabled: true
    ttl_seconds: 7200
    max_cache_size_mb: 10000
```

### 11.2. Feature Flags

```python
class AIFeatureFlags:
    """Manage AI feature rollout"""
    
    flags = {
        "enable_streaming": True,
        "enable_fallback_providers": True,
        "enable_response_caching": True,
        "enable_cost_optimization": True,
        "enable_quality_monitoring": False,  # Beta
        "enable_custom_models": False,       # Future
        "max_retries": 3,
        "timeout_seconds": 30
    }
    
    @classmethod
    def is_enabled(cls, feature: str, user_id: Optional[str] = None) -> bool:
        # Check base flag
        if feature not in cls.flags:
            return False
        
        base_enabled = cls.flags[feature]
        
        # Check user-specific overrides
        if user_id:
            override = cls._get_user_override(feature, user_id)
            if override is not None:
                return override
        
        return base_enabled
```

## 12. Testing Strategy

### 12.1. Unit Tests

```python
import pytest
from unittest.mock import Mock, patch

class TestAIService:
    @pytest.fixture
    def ai_service(self):
        return AIService(provider="mock")
    
    @pytest.mark.asyncio
    async def test_prompt_generation(self, ai_service):
        # Test that prompts are generated correctly
        prompt = await ai_service.generate_prompt(
            use_case="data_profiling",
            context={"columns": ["sales", "date", "region"]}
        )
        
        assert "data analysis expert" in prompt.system_message
        assert "sales" in prompt.user_message
    
    @pytest.mark.asyncio
    async def test_response_validation(self, ai_service):
        # Test response validation
        invalid_response = {"query": "DROP TABLE users;"}
        
        with pytest.raises(ValidationError):
            await ai_service.validate_response(
                invalid_response,
                ResponseType.SQL_QUERY
            )
    
    @pytest.mark.asyncio
    @patch('ai_service.providers.OpenAIProvider')
    async def test_fallback_behavior(self, mock_provider, ai_service):
        # Test fallback when primary provider fails
        mock_provider.complete.side_effect = ProviderError("Rate limit")
        
        response = await ai_service.get_completion(
            messages=[{"role": "user", "content": "test"}]
        )
        
        assert response.provider == "anthropic"  # Fallback provider
```

### 12.2. Integration Tests

```python
class TestAIIntegration:
    @pytest.mark.integration
    @pytest.mark.asyncio
    async def test_end_to_end_chart_generation(self):
        # Upload test data
        test_data = pd.DataFrame({
            'month': ['Jan', 'Feb', 'Mar'],
            'sales': [1000, 1500, 1200]
        })
        
        # Request chart generation
        request = ChartRequest(
            natural_language="Create a bar chart of sales by month",
            data_id="test_data_001"
        )
        
        response = await self.ai_service.generate_chart(request)
        
        # Validate response
        assert response.chart_type == "bar"
        assert response.x_axis == "month"
        assert response.y_axis == "sales"
        assert response.title == "Sales by Month"
```

## 13. Security Considerations

### 13.1. Prompt Injection Prevention

```python
class PromptSanitizer:
    """Prevent prompt injection attacks"""
    
    DANGEROUS_PATTERNS = [
        r"ignore previous instructions",
        r"system prompt",
        r"admin access",
        r"bypass security"
    ]
    
    def sanitize(self, user_input: str) -> str:
        # Check for dangerous patterns
        for pattern in self.DANGEROUS_PATTERNS:
            if re.search(pattern, user_input, re.IGNORECASE):
                raise SecurityError(f"Potentially malicious input detected")
        
        # Escape special characters
        sanitized = self._escape_special_chars(user_input)
        
        # Limit length
        if len(sanitized) > 10000:
            sanitized = sanitized[:10000]
        
        return sanitized
```

### 13.2. Data Privacy

```python
class DataPrivacyManager:
    """Ensure data privacy in AI interactions"""
    
    def prepare_data_for_ai(
        self,
        data: pd.DataFrame,
        privacy_level: str = "standard"
    ) -> pd.DataFrame:
        if privacy_level == "strict":
            # Remove all PII
            data = self._remove_pii(data)
            # Anonymize identifiers
            data = self._anonymize_ids(data)
        elif privacy_level == "standard":
            # Mask sensitive columns
            data = self._mask_sensitive_columns(data)
        
        return data
    
    def _remove_pii(self, data: pd.DataFrame) -> pd.DataFrame:
        pii_columns = self._detect_pii_columns(data)
        return data.drop(columns=pii_columns)
```