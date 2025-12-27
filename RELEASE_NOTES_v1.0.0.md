# Translation Worker v1.0.0

## Overview

The Translation Worker is a production-ready, distributed translation processing service that consumes translation jobs from RabbitMQ, translates content using multiple provider APIs, and writes results directly to the database. This initial stable release features multi-provider support, automatic retry logic, circuit breakers, health checks, metrics, and comprehensive error handling.

## Key Features

### Core Functionality
- **RabbitMQ Integration**: Consumes translation jobs from configurable RabbitMQ queues
- **Multi-Provider Translation**: Supports Google Translate, DeepL, and LibreTranslate APIs
- **Direct Database Writes**: Writes translated content directly to PostgreSQL (no round-trip to server)
- **Source Text Auto-Fetching**: Automatically fetches source text from database when not provided in job
- **Entity Type Support**: Handles multiple entity types (testimonial, project, certification, skill, etc.)
- **Language Support**: Comprehensive language code mapping for 12+ languages

### Production Features
- **Circuit Breakers**: Protects against cascading failures with configurable circuit breakers per provider
- **Health Check Endpoint**: HTTP endpoint at `/healthz` (port 8080) for Kubernetes liveness/readiness probes
- **Prometheus Metrics**: Comprehensive metrics exposed at `/metrics` endpoint
  - Job processing counters and durations
  - Failed job tracking by error type
  - Queue depth monitoring
  - Dead letter queue size tracking
- **Structured Logging**: JSON logging in production, text logging in development with trace ID support
- **Graceful Shutdown**: Handles SIGTERM/SIGINT signals for clean shutdown
- **Connection Retry Logic**: Automatic retry with exponential backoff for RabbitMQ connections

### Translation Providers
- **Google Translate**: Google Cloud Translation API v3 with project ID support
- **DeepL**: DeepL API (free and pro) with automatic endpoint detection
- **LibreTranslate**: LibreTranslate API (self-hosted or public instance) with optional API key
- **Automatic Fallback**: Falls back to LibreTranslate if primary provider is not configured

### Configuration
- **Environment-based Configuration**: All settings via environment variables
- **Database Configuration**: PostgreSQL connection via DATABASE_URL or individual components
- **RabbitMQ Configuration**: Flexible connection string or individual parameters
- **Queue Configuration**: Configurable queue names, exchanges, and routing keys
- **Translation Settings**: Configurable timeouts, retries, and retry delays
- **Prefetch Control**: Configurable message prefetch count for optimal throughput

### Code Quality
- **Comprehensive Testing**: Unit tests and integration tests included
- **Test Coverage**: Coverage reporting with threshold checks (70% minimum)
- **Clean Architecture**: Separation of concerns with internal and pkg packages
- **Go 1.23**: Built with latest Go version

## Architecture

```
┌─────────────┐
│   Server    │
└──────┬──────┘
       │
       │ Publish TranslationJob
       │
┌──────▼──────────┐
│   RabbitMQ      │
│   Queue         │
└──────┬──────────┘
       │
       │ TranslationJob Message
       │
┌──────▼─────────────────────────┐
│   Translation Worker           │
│                                │
│  ┌──────────────────┐          │
│  │  Queue Consumer  │          │
│  └────────┬─────────┘          │
│           │                    │
│  ┌────────▼─────────┐          │
│  │ Job Validator    │          │
│  └────────┬─────────┘          │
│           │                    │
│  ┌────────▼─────────┐          │
│  │ Source Text      │          │
│  │ Fetcher (DB)     │          │
│  └────────┬─────────┘          │
│           │                    │
│  ┌────────▼─────────┐          │
│  │ Translator       │          │
│  │ (Google/DeepL/   │          │
│  │  Libre)          │          │
│  └────────┬─────────┘          │
│           │                    │
│  ┌────────▼─────────┐          │
│  │ DB Writer        │          │
│  └──────────────────┘          │
└────────────────────────────────┘
```

## Dependencies

- **github.com/rabbitmq/amqp091-go**: RabbitMQ client library
- **github.com/prometheus/client_golang**: Prometheus metrics
- **github.com/sony/gobreaker**: Circuit breaker implementation
- **github.com/google/uuid**: UUID generation
- **gorm.io/gorm**: ORM for database operations
- **gorm.io/driver/postgres**: PostgreSQL driver
- **github.com/stretchr/testify**: Testing utilities

## Configuration Variables

### Database Configuration
- `DATABASE_URL`: PostgreSQL connection string (required)
  - Alternative: Use `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSLMODE`

### RabbitMQ Configuration
- `RABBITMQ_URL`: Full AMQP connection URL (overrides individual settings)
- `RABBITMQ_USER`: RabbitMQ username (default: "woragis")
- `RABBITMQ_PASSWORD`: RabbitMQ password (default: "woragis")
- `RABBITMQ_HOST`: RabbitMQ hostname (default: "rabbitmq")
- `RABBITMQ_PORT`: RabbitMQ port (default: "5672")
- `RABBITMQ_VHOST`: RabbitMQ virtual host (default: "/")

### Translation Provider Configuration
- `TRANSLATION_PROVIDER`: Provider to use - "google", "deepl", or "libre" (default: "google")

#### Google Translate
- `GOOGLE_TRANSLATE_API_KEY`: Google Cloud Translation API key (required for Google provider)
- `GOOGLE_CLOUD_PROJECT_ID`: Google Cloud project ID (optional)

#### DeepL
- `DEEPL_API_KEY`: DeepL API key (required for DeepL provider)

#### LibreTranslate
- `LIBRE_TRANSLATE_API_URL`: LibreTranslate API URL (default: "https://libretranslate.com/translate")
- `LIBRE_TRANSLATE_API_KEY`: LibreTranslate API key (optional for public instance)

### Translation API Settings
- `TRANSLATION_TIMEOUT`: Request timeout in seconds (default: 30)
- `TRANSLATION_MAX_RETRIES`: Maximum retry attempts (default: 3)
- `TRANSLATION_RETRY_DELAY`: Retry delay in milliseconds (default: 1000)

### Worker Configuration
- `TRANSLATION_QUEUE_NAME`: Queue name (default: "translations.queue")
- `TRANSLATION_EXCHANGE`: Exchange name (default: "woragis.tasks")
- `TRANSLATION_ROUTING_KEY`: Routing key (default: "translations.process")
- `TRANSLATION_PREFETCH_COUNT`: Prefetch count (default: 1)

### Environment
- `ENV`: Environment mode - "development" or "production" (affects logging)
- `LOG_TO_FILE`: Enable file logging in development (default: false)
- `LOG_DIR`: Directory for log files (default: "logs")

## Message Format

The worker expects `TranslationJob` messages in the following JSON format:

```json
{
  "id": "job-uuid",
  "entityType": "project",
  "entityId": "entity-uuid",
  "language": "pt-BR",
  "fields": ["name", "description"],
  "sourceText": {
    "name": "My Project",
    "description": "Project description"
  }
}
```

**Note:** If `sourceText` is empty or not provided, the worker will automatically fetch it from the database based on `entityType` and `entityId`.

## Supported Entity Types

- `testimonial`
- `project`
- `certification`
- `skill`
- (More can be added in `internal/database/database.go`)

## Supported Languages

The worker supports translation to/from the following languages with automatic code mapping:

- `en` - English
- `pt-BR` - Portuguese (Brazil)
- `pt` - Portuguese
- `es` - Spanish
- `fr` - French
- `de` - German
- `ru` - Russian
- `ja` - Japanese
- `ko` - Korean
- `zh-CN` - Chinese (Simplified)
- `el` - Greek
- `la` - Latin
- `sv` - Swedish

Language codes are automatically mapped to each provider's format (e.g., "pt-BR" → "PT-BR" for DeepL, "pt" for Google).

## Circuit Breakers

Each translation provider is protected by a circuit breaker that:

- **Opens** after 5 consecutive failures
- **Half-Open** after 30 seconds timeout
- **Allows** up to 3 requests in half-open state
- **Resets** counters every 60 seconds in closed state

Circuit breaker state changes are logged for monitoring and debugging.

## Health Check

The service exposes a health check endpoint at `GET /healthz`:

- **Healthy Response (200 OK)**: All checks pass
- **Unhealthy Response (503)**: RabbitMQ connection is down

The health check includes:
- RabbitMQ connection status check (2s timeout)
- Result caching (5s TTL) for performance

## Metrics

Prometheus metrics are exposed at `GET /metrics`:

- `worker_jobs_processed_total{worker, status}`: Total jobs processed
- `worker_job_duration_seconds{worker}`: Job processing duration histogram
- `worker_jobs_failed_total{worker, error_type}`: Failed job counter (by error type)
- `worker_jobs_retried_total{worker}`: Retried job counter
- `queue_depth{queue_name}`: Current queue depth
- `queue_dlq_size{queue_name}`: Dead letter queue size

## Deployment

### Docker
The service can be containerized and deployed as a standalone worker or in Kubernetes.

### Kubernetes
- **Liveness Probe**: `GET /healthz` every 10s, timeout 5s
- **Readiness Probe**: `GET /healthz` every 10s, timeout 5s
- **Port**: 8080 (health and metrics)

### Scaling
Multiple worker instances can consume from the same queue for horizontal scaling. Prefetch count can be adjusted based on translation API rate limits and processing time.

## Development

### Testing
```bash
# Run all tests
make test

# Run unit tests only
make test-unit

# Run with coverage
make test-cov

# Check coverage threshold (70%)
make test-cov-check

# Run tests in Docker
make test-docker
```

### Building
```bash
# Install dependencies
make install

# Build binary
make build

# Build Docker image
make docker-build
```

### Running Locally
```bash
# Set environment variables
export DATABASE_URL="host=localhost port=5432 user=woragis password=woragis dbname=woragis sslmode=disable"
export RABBITMQ_URL="amqp://woragis:woragis@localhost:5672/woragis"
export TRANSLATION_PROVIDER=google
export GOOGLE_TRANSLATE_API_KEY="your-api-key"

# Run worker
./bin/translation-worker
```

## Error Handling

- **Invalid Messages**: Rejected without requeue (sent to DLQ)
- **Translation Failures**: Retried up to max retries, then sent to DLQ
- **Database Errors**: Retried up to max retries, then sent to DLQ
- **API Errors**: 
  - Client errors (4xx): Not retried, sent to DLQ immediately
  - Server errors (5xx): Retried with exponential backoff
  - Circuit breaker open: Fails fast, sent to DLQ
- **Validation Errors**: Rejected without requeue

## Integration with Server

The server should publish translation jobs to the RabbitMQ queue. The worker handles all processing and writes results directly to the database, so the server only needs to:

1. Create a `TranslationJob` with entity information
2. Publish it to RabbitMQ
3. Optionally poll the database for completion status

## Performance Considerations

- **Concurrency**: Set `TRANSLATION_PREFETCH_COUNT` to control concurrent job processing
- **Retries**: Configure `TRANSLATION_MAX_RETRIES` and `TRANSLATION_RETRY_DELAY` based on API reliability
- **Timeout**: Adjust `TRANSLATION_TIMEOUT` based on API response times
- **Scaling**: Run multiple worker instances for horizontal scaling
- **Circuit Breakers**: Protect against API failures and rate limits
- **Database Connections**: GORM manages connection pooling automatically

## Documentation

- **README.md**: Comprehensive service documentation and usage examples
- **HEALTH_CHECK.md**: Detailed health check documentation
- **LOGGING.md**: Structured logging guidelines and usage
- **TEST_RESULTS.md**: Test results and coverage information
- **tests/README.md**: Testing documentation
- **internal/integration/README.md**: Integration test documentation

## Breaking Changes

None - This is the initial release.

## Future Enhancements

Potential improvements for future versions:
- Batch translation support
- Translation caching layer
- Additional translation providers (Azure Translator, AWS Translate)
- Translation quality scoring
- Progress tracking for long-running translations
- Webhook notifications on completion
- Translation memory integration
- Custom language mapping configuration
- Rate limiting per provider
- Cost tracking and analytics

## Contributors

Initial release by the Woragis team.

## License

Part of the Woragis backend infrastructure.

