# HDB Bank ESXi Dynatrace POC

Proof of Concept environment on VMware ESXi replicating HDB Bank's production banking architecture with Dynatrace full-stack observability.

## Architecture

```
Consumer Channels → F5 Load Balancer → IBM DataPower (API Gateway/WAF)
    → IBM App Connect (2 nodes, Orchestration/Routing)
        → IRIS (REST APIs) / TWS (SOAP Services) → Temenos T24 → Oracle DB
        → Card Management System
        → Legacy System
        → MSSQL (Logging/Lookups)
```

## Components

| VM Name | Role | Port |
|---------|------|------|
| HDB-POC-F5 | F5 Load Balancer | 80/443 |
| HDB-POC-DataPower | IBM DataPower API Gateway | 8443 |
| HDB-POC-AppConnect-01/02 | App Connect Middleware (2 nodes) | 8080 |
| HDB-POC-IRIS | IRIS REST APIs | 8081 |
| HDB-POC-TWS | TWS SOAP Services | 8082 |
| HDB-POC-CardMgmt | Card Management | 8083 |
| HDB-POC-Legacy | Legacy System | 8084 |
| HDB-POC-T24 | Temenos T24 Core Banking | 8090 |
| HDB-POC-OracleDB | Oracle Database | 1521 |
| HDB-POC-MSSQL | MSSQL Logging/Lookup | 1433 |

## Network Topology

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 10 | DMZ | 10.0.10.0/24 | Consumer-facing entry point |
| 20 | Gateway | 10.0.20.0/24 | API gateway / WAF |
| 30 | Middleware | 10.0.30.0/24 | Orchestration / routing |
| 40 | Backend | 10.0.40.0/24 | Application services |
| 50 | Core Banking | 10.0.50.0/24 | Core banking processing |
| 60 | Database | 10.0.60.0/24 | Data persistence / logging |
| Mgmt | Management | 10.0.100.0/24 | Dynatrace OneAgent traffic |

## Project Structure

```
simulators/          # Python Flask simulators for each component
provisioning/        # PowerCLI, shell scripts for VM provisioning
tests/               # Unit, property-based, and integration tests
config/              # VM inventory, traffic profiles, Dynatrace configs
docs/                # POC documentation and presentation materials
specs/               # Requirements, design, and implementation plan
```

## Dynatrace Integration

- OneAgent deployed on all 11+ VMs
- Distributed tracing (PurePath) across full transaction flow
- Topology, transaction flow, and infrastructure dashboards
- Alerting on response time thresholds and CPU utilization

## Quick Start

1. Run `provisioning/provision-vms.ps1` to create VMs on ESXi
2. Run `provisioning/configure-network.sh` to set up VLANs
3. Run `provisioning/deploy-services.sh` to deploy simulators
4. Run `provisioning/install-oneagent.sh` to deploy Dynatrace
5. Run `provisioning/validate-environment.sh` to verify setup
6. Run `provisioning/run-traffic.sh medium` to start traffic simulation
