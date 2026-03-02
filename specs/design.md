# Design Document: ESXi Dynatrace POC

## Overview

This design describes the architecture and implementation of a Proof of Concept environment on VMware ESXi that replicates HDB Bank's production banking architecture. The POC provisions 11+ virtual machines simulating the full transaction flow. Dynatrace OneAgent is deployed on every VM to demonstrate end-to-end distributed tracing (PurePath), infrastructure monitoring, dashboards, and alerting.

### Key Design Decisions

1. **Simulation over Emulation**: Each banking component is simulated using lightweight HTTP/SOAP services rather than deploying actual vendor products.
2. **Infrastructure as Code**: All VM provisioning, network configuration, and service deployment is automated via scripts (PowerCLI / shell).
3. **Dynatrace OneAgent Auto-Instrumentation**: OneAgent is deployed on every VM and relies on automatic process/service discovery.
4. **Single ESXi Host**: All VMs run on a single ESXi host. Network segmentation is achieved via vSwitches and port groups.
5. **Python/Node.js Simulators**: Component simulators are implemented as lightweight HTTP servers (Python Flask).

## Architecture

### Network Topology

| VLAN | Name | Subnet | Purpose | VMs |
|------|------|--------|---------|-----|
| 10 | DMZ | 10.0.10.0/24 | Consumer-facing entry point | F5_VM |
| 20 | Gateway | 10.0.20.0/24 | API gateway / WAF | DataPower_VM |
| 30 | Middleware | 10.0.30.0/24 | Orchestration / routing | AppConnect_VM x2 |
| 40 | Backend | 10.0.40.0/24 | Application services | IRIS_VM, TWS_VM, CardMgmt_VM, Legacy_VM |
| 50 | Core Banking | 10.0.50.0/24 | Core banking processing | T24_VM |
| 60 | Database | 10.0.60.0/24 | Data persistence / logging | Oracle_DB_VM, MSSQL_VM |
| Mgmt | Management | 10.0.100.0/24 | Dynatrace OneAgent to Server | All VMs (secondary NIC) |

### VM Inventory

| VM Name | Role | OS | vCPU | RAM (GB) | Disk (GB) | Primary IP | VLAN |
|---------|------|----|------|----------|-----------|------------|------|
| HDB-POC-F5 | F5 Load Balancer | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.10.10 | 10 |
| HDB-POC-DataPower | IBM DataPower API GW | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.20.10 | 20 |
| HDB-POC-AppConnect-01 | App Connect Node 1 | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.30.10 | 30 |
| HDB-POC-AppConnect-02 | App Connect Node 2 | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.30.11 | 30 |
| HDB-POC-IRIS | IRIS REST APIs | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.40.10 | 40 |
| HDB-POC-TWS | TWS SOAP Services | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.40.11 | 40 |
| HDB-POC-CardMgmt | Card Management | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.40.12 | 40 |
| HDB-POC-Legacy | Legacy System | Ubuntu 22.04 | 2 | 4 | 40 | 10.0.40.13 | 40 |
| HDB-POC-T24 | Temenos T24 Core Banking | Ubuntu 22.04 | 4 | 8 | 80 | 10.0.50.10 | 50 |
| HDB-POC-OracleDB | Oracle Database | Oracle Linux 8 | 4 | 8 | 120 | 10.0.60.10 | 60 |
| HDB-POC-MSSQL | MSSQL Logging/Lookup | Ubuntu 22.04 | 2 | 4 | 60 | 10.0.60.11 | 60 |

## Component Interfaces

### F5_VM - Load Balancer Simulator
- Technology: Python Flask or Nginx reverse proxy
- Port: 443 (HTTPS), 80 (HTTP)
- Endpoints: POST /api/v1/transaction, GET /health

### DataPower_VM - API Gateway Simulator
- Technology: Python Flask, Port: 8443
- Validation: Required headers (X-Channel-Type, Content-Type), Required body fields (transactionType, accountNumber)
- Routing: Round-robin to AppConnect nodes

### AppConnect_VM - Middleware Simulator
- Technology: Python Flask, Port: 8080
- Routing: accountInquiry/fundTransfer/balanceCheck -> IRIS, statementRetrieval/standingOrder -> TWS, card* -> CardMgmt, legacyLookup -> Legacy
- All requests logged to MSSQL

### IRIS_VM - REST API Simulator
- Technology: Python Flask, Port: 8081
- Endpoints: POST /api/v1/account/inquiry, /transfer, /balance

### TWS_VM - SOAP Service Simulator
- Technology: Python Flask + lxml, Port: 8082
- Endpoints: POST /ws/statement, /ws/standingorder

### T24_VM - Core Banking Simulator
- Technology: Python Flask, Port: 8090
- Endpoints: POST /t24/transaction, GET /t24/account/<accountNumber>

### CardMgmt_VM - Card Management Simulator
- Technology: Python Flask, Port: 8083
- Endpoints: POST /api/v1/card/issue, GET /api/v1/card/status/<cardId>, POST /api/v1/card/block

### Legacy_VM - Legacy System Simulator
- Technology: Python Flask, Port: 8084
- Endpoint: POST /legacy/message (fixed-format 80-char messages)

### Oracle_DB_VM - Database
- Technology: Oracle Database XE, Port: 1521
- Schema: HDB_POC with CUSTOMERS, ACCOUNTS, TRANSACTIONS tables

### MSSQL_VM - Logging Database
- Technology: Microsoft SQL Server 2019 Express, Port: 1433
- Schema: HDB_POC_LOG with TRANSACTION_LOG table

## Data Models

### Oracle Database Schema (HDB_POC)

```sql
CREATE TABLE CUSTOMERS (
    customer_id     VARCHAR2(20) PRIMARY KEY,
    name            VARCHAR2(100) NOT NULL,
    email           VARCHAR2(100),
    phone           VARCHAR2(20),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE ACCOUNTS (
    account_number  VARCHAR2(20) PRIMARY KEY,
    customer_id     VARCHAR2(20) REFERENCES CUSTOMERS(customer_id),
    account_type    VARCHAR2(20) CHECK (account_type IN ('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT')),
    balance         NUMBER(15,2) DEFAULT 0,
    currency        VARCHAR2(3) DEFAULT 'SGD',
    status          VARCHAR2(10) CHECK (status IN ('ACTIVE', 'DORMANT', 'CLOSED')),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE TRANSACTIONS (
    transaction_id  VARCHAR2(36) PRIMARY KEY,
    account_number  VARCHAR2(20) REFERENCES ACCOUNTS(account_number),
    transaction_type VARCHAR2(20),
    amount          NUMBER(15,2),
    target_account  VARCHAR2(20),
    status          VARCHAR2(10),
    channel         VARCHAR2(20),
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### MSSQL Logging Schema

```sql
CREATE TABLE TRANSACTION_LOG (
    id              INT IDENTITY(1,1) PRIMARY KEY,
    timestamp       DATETIME2 DEFAULT GETDATE(),
    transaction_id  NVARCHAR(36) NOT NULL,
    log_level       NVARCHAR(10),
    source_service  NVARCHAR(50) NOT NULL,
    target_service  NVARCHAR(50),
    routing_rule    NVARCHAR(100),
    message         NVARCHAR(500)
);
```

## Dynatrace Integration

### OneAgent Deployment
- Installer downloaded from Dynatrace Server API
- Deployed via SSH shell script on each VM
- Auto-discovers Python/Flask, Oracle, MSSQL processes
- Trace propagation via x-dynatrace and traceparent headers

### Dashboards
| Dashboard | Content | Purpose |
|-----------|---------|--------|
| Topology Map | Auto-discovered service map | Show interconnections |
| Transaction Flow | Response times, throughput, error rates | End-to-end performance |
| Infrastructure | CPU, memory, disk, network per VM | Resource utilization |

### Alerting Rules
| Alert | Condition | Threshold |
|-------|-----------|----------|
| Slow Service | Response time > threshold | Configurable (default 1s) |
| High CPU | CPU > 90% for > 2 min | 90% sustained |

## Provisioning Automation

1. provision-vms.ps1 - PowerCLI script to create/clone VMs
2. configure-network.sh - Configure vSwitches, port groups, VM NICs
3. deploy-services.sh - Deploy simulator services via SSH + systemd
4. install-oneagent.sh - Download and install Dynatrace OneAgent
5. validate-environment.sh - Run connectivity and health checks
6. run-traffic.sh - Start channel simulator with configurable profiles

## Error Handling

| Component | Error Condition | Response | HTTP Status |
|-----------|----------------|----------|-------------|
| F5_VM | DataPower unreachable | F5_BACKEND_DOWN | 502 |
| DataPower_VM | Missing required headers | VALIDATION_ERROR | 400 |
| DataPower_VM | AppConnect unreachable | DP_BACKEND_DOWN | 502 |
| AppConnect_VM | Unknown transaction type | ROUTING_ERROR | 400 |
| AppConnect_VM | Backend unreachable | SERVICE_UNAVAILABLE | 502 |
| IRIS_VM | T24 unreachable | IRIS_T24_DOWN | 502 |
| TWS_VM | T24 unreachable | TWS_T24_DOWN (SOAP Fault) | 500 |
| T24_VM | Invalid data | T24_VALIDATION_ERROR | 400 |
| T24_VM | Insufficient balance | T24_INSUFFICIENT_FUNDS | 422 |
| CardMgmt_VM | Invalid card ID | CARD_NOT_FOUND | 404 |

## Testing Strategy

- Unit tests (pytest) for each component
- Property-based tests (Hypothesis) for 18 correctness properties
- Integration tests for end-to-end transaction flows
- Infrastructure validation tests for VM/network/OneAgent verification

See full design document in the local specs directory for complete details including Mermaid diagrams, all 18 correctness properties, and detailed request/response models.
