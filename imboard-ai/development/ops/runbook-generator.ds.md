---
authors:
- name: yuvaldim
checksum:
  algorithm: sha256
  hash: ca1d4e39ecd416d96aee93ae80f24ca785ef2fda138830a2d170031acde7f663
name: runbook-generator
objective: Generate service runbooks and ops checklists for oncall readiness
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: SwEk7lM8G/tDvlZRlYz3walItmBUGEzD0LTJu6svOJND+u6LVZNr4TDjhar94DImVyLY3NU2OeKUrLoqYRwYAA==
  signed_by: yuvaldim
  timestamp: '2025-12-14T15:32:03.319558+00:00'
status: draft
title: Runbook & Ops Checklist Generator
version: 1.0.0
---

# Runbook & Ops Checklist Generator

## Overview

This dossier defines the workflow for generating comprehensive service runbooks and operational checklists to improve oncall readiness, reduce Mean Time To Resolution (MTTR), and standardize service ownership practices.

## Metadata

- **Task ID**: D-010
- **Category**: Service Operations & Oncall Readiness
- **Trigger Events**: New service deployment, ownership handoff, post-incident review
- **Safety Level**: Read-only operations with assumption tracking

## Motivation

Service oncall and ownership transitions often suffer from incomplete documentation, missing operational knowledge, and unclear escalation paths. This leads to:
- Increased MTTR during incidents
- Knowledge silos and difficult ownership handoffs
- Inconsistent monitoring and alerting practices
- Unclear service dependencies and failure modes

This dossier standardizes the generation of runbooks and operational checklists to ensure every service has complete, actionable documentation for oncall engineers.

## Inputs and Context Requirements

### Required Inputs

1. **Service Identification**
   - Service name and description
   - Repository URL or local path
   - Primary programming language/framework

2. **Deployment Information**
   - Deployment manifests (Kubernetes YAML, Docker Compose, Terraform, etc.)
   - Infrastructure configuration files
   - Environment-specific configurations (dev, staging, production)

3. **Service Metadata**
   - Service owner(s) and team
   - Oncall rotation information (if exists)
   - Communication channels (Slack, PagerDuty, etc.)

### Optional Context (Enhances Output Quality)

- **Historical Incident Data**: Past incidents, postmortems, known issues
- **Monitoring Configuration**: Existing metrics, dashboards, alerts
- **Dependency Documentation**: Known upstream/downstream services
- **Architecture Diagrams**: Service architecture, data flow diagrams
- **API Documentation**: OpenAPI specs, GraphQL schemas, gRPC definitions

### Input Validation

Before proceeding, verify:
- [ ] Repository/service path is accessible
- [ ] Deployment manifests are present and parseable
- [ ] Service owner information is available
- [ ] At least one environment configuration exists

## Execution Workflow

### Phase 1: Service Discovery and Analysis

**Objective**: Extract factual information about the service from code and configuration.

1. **Analyze Repository Structure**
   - Identify service entry points (main.py, main.go, index.js, etc.)
   - Locate configuration files (config.yaml, .env.example, application.properties)
   - Find deployment manifests (k8s/, docker-compose.yml, Dockerfile)
   - Discover documentation (README.md, docs/, wiki links)

2. **Extract Service Facts**
   - **Network Configuration**:
     - Listening ports (HTTP, gRPC, metrics, health checks)
     - External endpoints and routes
     - Protocol details (REST, GraphQL, gRPC, WebSocket)

   - **Dependencies**:
     - Databases (type, connection details)
     - External services (APIs, message queues, cache systems)
     - Infrastructure dependencies (S3, blob storage, secrets managers)

   - **Configuration**:
     - Environment variables and their purposes
     - Feature flags and configuration management
     - Secrets and credential requirements (note locations, not values)

   - **Observability**:
     - Logging framework and log locations
     - Metrics exporters (Prometheus, StatsD, custom)
     - Tracing integration (Jaeger, Zipkin, OpenTelemetry)
     - Health check endpoints

3. **Analyze Deployment Manifests**
   - Resource requirements (CPU, memory, disk)
   - Replica count and scaling configuration
   - Deployment strategy (rolling, blue-green, canary)
   - Health check and readiness probe configuration
   - Volume mounts and persistent storage

### Phase 2: Runbook Generation

**Objective**: Create a structured runbook document with operational procedures.

Generate a markdown runbook with the following sections:

#### 1. Service Overview
- Service name, purpose, and business context
- Service owner and team contacts
- Links to repository, documentation, and dashboards

#### 2. Architecture and Dependencies
- High-level architecture description
- Dependency map (upstream and downstream services)
- Data stores and external integrations
- Network topology and exposure (internal/external)

#### 3. Service Configuration
- Environment variables reference table
- Configuration file locations and purposes
- Feature flag documentation
- Secret management approach

#### 4. Deployment and Operations
- Deployment process overview
- Environment-specific details (dev, staging, production)
- Rollback procedures
- Scaling guidelines

#### 5. Monitoring and Alerting
- Key metrics to monitor (with proposed thresholds if none exist)
- Dashboard links
- Existing alerts (or proposed alerts if none exist)
- Log query examples for common scenarios

#### 6. Troubleshooting Guide
- Common failure modes and symptoms
- Debugging procedures for typical issues
- Log analysis tips
- Performance troubleshooting steps

#### 7. Oncall Runbook
- Alert response procedures
- Escalation paths and contacts
- Critical operations (restart, rollback, scale up/down)
- Known issues and workarounds

#### 8. Emergency Procedures
- Service shutdown/startup procedures
- Data backup and recovery
- Incident response checklist
- Emergency contacts

### Phase 3: SLO and Alert Proposal

**Objective**: Propose Service Level Objectives and alert configurations.

1. **Identify Service Type and Criticality**
   - User-facing vs. internal service
   - Synchronous vs. asynchronous processing
   - Business impact of downtime

2. **Propose SLOs** (mark as "PROPOSED" if not already defined)
   - **Availability SLO**: Uptime target (e.g., 99.9%)
   - **Latency SLO**: Response time targets (p50, p95, p99)
   - **Error Rate SLO**: Acceptable error percentage
   - **Throughput expectations**: Request/transaction volumes

3. **Propose Alert Configurations**
   - **Critical Alerts**: Service down, error rate spike, dependency failure
   - **Warning Alerts**: High latency, elevated error rate, resource exhaustion
   - **Informational**: Deployment events, configuration changes

   For each alert, specify:
   - Alert condition (metric, threshold, duration)
   - Severity level
   - Notification channel
   - Runbook link for response

### Phase 4: Operational Checklist Generation

**Objective**: Create actionable checklists for service ownership and readiness.

#### Ops Readiness Checklist

Generate a checklist covering:

- [ ] **Monitoring**
  - [ ] Service health check endpoint exists and is monitored
  - [ ] Key metrics are instrumented and collected
  - [ ] Dashboards exist for service overview
  - [ ] Log aggregation is configured
  - [ ] Distributed tracing is enabled (if applicable)

- [ ] **Alerting**
  - [ ] Critical alerts are configured and tested
  - [ ] Alert notifications route to oncall
  - [ ] Alert runbooks are documented
  - [ ] False positive rate is acceptable

- [ ] **SLOs and SLIs**
  - [ ] Service SLOs are defined
  - [ ] SLI metrics are tracked
  - [ ] Error budget policy exists

- [ ] **Documentation**
  - [ ] README with service overview exists
  - [ ] Runbook is complete and accessible
  - [ ] Architecture diagram is current
  - [ ] API documentation is available

- [ ] **Deployment and Recovery**
  - [ ] Deployment process is documented
  - [ ] Rollback procedure is tested
  - [ ] Disaster recovery plan exists
  - [ ] Backup and restore procedures are documented

- [ ] **Security and Compliance**
  - [ ] Secrets are managed securely (no hardcoded credentials)
  - [ ] Access control is configured
  - Security scanning is enabled
  - [ ] Compliance requirements are documented

#### Handoff Checklist

For ownership transitions:

- [ ] **Knowledge Transfer**
  - [ ] Runbook review completed
  - [ ] Architecture walkthrough conducted
  - [ ] Recent incidents reviewed
  - [ ] Oncall rotation updated

- [ ] **Access and Permissions**
  - [ ] Repository access granted
  - [ ] Infrastructure access configured
  - [ ] Oncall rotation added
  - [ ] Monitoring/alerting access verified

- [ ] **Testing and Validation**
  - [ ] Service deployed in test environment
  - [ ] Common operations performed (deploy, rollback, scale)
  - [ ] Alert response practiced
  - [ ] Troubleshooting procedures validated

## Output Artifacts

The workflow produces three primary artifacts:

### 1. Service Runbook (`<service-name>-runbook.md`)

A comprehensive markdown document containing:
- Service overview and architecture
- Configuration reference
- Deployment procedures
- Monitoring and alerting details
- Troubleshooting guides
- Emergency procedures
- Contact information

### 2. Ops Readiness Checklist (`<service-name>-ops-checklist.md`)

A markdown checklist with:
- Monitoring and alerting requirements
- SLO/SLI definitions (proposed or existing)
- Documentation completeness checks
- Security and compliance items
- Deployment and recovery readiness

### 3. Handoff Checklist (`<service-name>-handoff-checklist.md`)

A markdown checklist for ownership transitions:
- Knowledge transfer items
- Access and permission requirements
- Testing and validation steps
- Communication and rotation updates

## Safety Constraints and Boundaries

### Read-Only Operations

- **NO modifications** to service code or configuration
- **NO deployment** or infrastructure changes
- **NO access** to production secrets or credentials
- **NO execution** of database queries or service commands

### Assumption Tracking

When information is incomplete or unavailable:

1. **Mark assumptions explicitly** using this format:
   ```
   > ASSUMPTION: [Description of assumption]
   > CONFIDENCE: [High/Medium/Low]
   > VALIDATION: [How to verify this assumption]
   ```

2. **Common scenarios requiring assumptions**:
   - Missing deployment manifests: Assume standard patterns for the framework
   - No existing monitoring: Propose standard metrics based on service type
   - Unclear dependencies: List potential dependencies marked as "unverified"
   - Missing SLOs: Propose industry-standard targets for service category

3. **Never assume**:
   - Specific production URLs or IP addresses (use placeholders)
   - Secret values or credentials
   - Exact infrastructure capacity or scaling limits
   - Specific incident details not documented

### Sensitive Information Handling

- **Redact** any discovered credentials, API keys, or tokens
- **Generalize** production infrastructure details (use "production-db-host" not actual hostnames)
- **Placeholder** oncall contacts if not publicly documented (use "ONCALL_TEAM@example.com")
- **Link** to secret management systems rather than embedding secret locations

### Quality Standards

- All facts must be traceable to source (code, config, or documentation)
- Procedures must be specific and actionable
- Checklists must be binary (clear yes/no completion criteria)
- Proposals must be clearly marked as "PROPOSED" vs. "EXISTING"

## Example Usage Scenarios

### Scenario 1: New Service Onboarding

**Context**: A new microservice has been developed and is ready for production deployment.

**Inputs**:
- Repository: github.com/company/payment-service
- Deployment: Kubernetes manifests in k8s/
- No existing runbook or monitoring

**Execution**:
1. Analyze repository structure and extract service facts
2. Generate runbook based on code and deployment configs
3. Propose SLOs based on service type (payment processing = high availability)
4. Create ops readiness checklist highlighting missing monitoring
5. Include handoff checklist for oncall team onboarding

**Output**: Complete runbook with proposed monitoring and SLOs marked for implementation.

### Scenario 2: Post-Incident Documentation

**Context**: A production incident occurred due to missing runbook procedures.

**Inputs**:
- Existing service with partial documentation
- Incident postmortem document
- Request to update runbook with incident learnings

**Execution**:
1. Review existing runbook and identify gaps
2. Extract incident details (failure mode, resolution steps)
3. Add troubleshooting section for this failure mode
4. Update emergency procedures if needed
5. Propose new alerts to prevent recurrence

**Output**: Updated runbook with incident-specific troubleshooting and alert proposals.

### Scenario 3: Ownership Handoff

**Context**: Service ownership is transferring from Team A to Team B.

**Inputs**:
- Existing service with outdated documentation
- New team contact information
- Request for handoff preparation

**Execution**:
1. Audit and update existing runbook
2. Generate comprehensive handoff checklist
3. Verify all access requirements are documented
4. Create knowledge transfer guide highlighting critical areas
5. Update oncall contacts and escalation paths

**Output**: Updated runbook + handoff checklist + access requirements document.

## Success Criteria

A successful runbook generation meets these criteria:

- [ ] All extracted facts are verifiable from source
- [ ] Assumptions are clearly marked with confidence levels
- [ ] Runbook sections are complete and actionable
- [ ] Checklists have clear completion criteria
- [ ] No sensitive information is exposed
- [ ] Proposed SLOs are appropriate for service type
- [ ] Alert proposals cover critical failure modes
- [ ] Troubleshooting steps are specific and testable
- [ ] Emergency procedures include rollback steps
- [ ] Contact information is current or marked as TBD

## Maintenance and Updates

Runbooks should be updated when:
- Service architecture changes significantly
- New dependencies are added
- Incidents reveal documentation gaps
- Ownership or oncall rotation changes
- Monitoring or alerting configuration changes
- SLOs are formally defined or modified

## References and Resources

- Site Reliability Engineering (SRE) Book: Chapter on Practical Alerting
- Google SRE Workbook: Runbook sections and templates
- ITIL Service Operation: Incident management practices
- Kubernetes Best Practices: Health checks and deployment strategies
- The Twelve-Factor App: Configuration and service design principles