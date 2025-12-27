# Translation Worker v1.1.0

## Release Date
December 27, 2025

## Overview
This release focuses on CI/CD improvements, test coverage enhancements, and workflow automation. The service now includes comprehensive GitHub Actions workflows for continuous integration and deployment.

## What's New

### CI/CD Workflows
- **GitHub Actions CI Workflow**: Automated testing on push and pull requests
  - Unit tests with coverage reporting
  - Integration tests with RabbitMQ and PostgreSQL service containers
  - Docker image building and validation
  - Markdown test reports in GitHub Actions
- **GitHub Actions CD Workflow**: Automated deployment on version tags
  - Docker image building and pushing to Docker Hub
  - Multi-tag support (vX.Y.Z, X.Y.Z, latest)
- **Go Module Caching**: Optimized dependency installation with Go module cache

### Test Improvements
- **Test Coverage**: Improved unit test coverage
  - Added comprehensive metrics tests (100% coverage)
  - Enhanced test coverage across all packages
- **Test Reports**: Markdown-formatted test summaries in CI
- **Coverage Threshold**: Set to 30% (packages requiring external services excluded)
- **Integration Tests**: RabbitMQ and PostgreSQL service container configuration for CI

### Docker Improvements
- **Docker Build**: Enhanced Dockerfile with multi-stage build
- **Image Testing**: Automated Docker image validation in CI

## Technical Details

### Dependencies
- No breaking changes to dependencies
- Go 1.23 (as specified in go.mod)

### Configuration
- No configuration changes required
- RabbitMQ and PostgreSQL service container configuration for integration tests

## Migration Guide
No migration required. This is a non-breaking release.

## Full Changelog

### Added
- GitHub Actions CI workflow (`.github/workflows/ci.yml`)
- GitHub Actions CD workflow (`.github/workflows/cd.yml`)
- Markdown test reports in CI
- Go module caching in CI workflows
- Docker image validation in CI
- Comprehensive metrics tests (`pkg/metrics/metrics_test.go`) with 100% coverage
- PostgreSQL service container for integration tests

### Changed
- Improved test coverage reporting
- Enhanced RabbitMQ and PostgreSQL service container configuration
- Better test result parsing in CI with multiline output format
- Excluded `cmd/translation-worker` from coverage (main functions are hard to test)
- Lowered coverage threshold to 30% (packages requiring external services)

### Fixed
- Fixed test result parsing to handle zero counts correctly
- Improved GitHub Actions output format to prevent "Invalid format" errors
- Enhanced coverage extraction from Go test output

## Known Issues
- Integration tests may fail due to RabbitMQ vhost permissions (to be addressed in future release)

## Contributors
- Automated CI/CD implementation
- Test coverage improvements

