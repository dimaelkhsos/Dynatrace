# Implementation Plan: ESXi Dynatrace POC

## Overview

This plan implements the HDB Bank POC environment on ESXi VMware with Python Flask simulators for all banking components, Oracle/MSSQL databases, Dynatrace OneAgent deployment, and a professional documentation package. Tasks are ordered to build infrastructure first, then simulators bottom-up (database -> core banking -> middleware -> entry point), followed by Dynatrace integration, traffic simulation, and documentation.

## Tasks

### 1. Create project structure and shared utilities
- 1.1 Create directory structure and Python project scaffolding
- 1.2 Implement shared request/response models and utilities

### 2. Implement database schemas and seed data
- 2.1 Create Oracle Database schema and seed script
- 2.2 Create MSSQL logging schema script
- 2.3 (Optional) Write property test for Account Data Completeness

### 3. Implement T24 Core Banking Simulator
- 3.1 Implement T24 Flask application (POST /t24/transaction, GET /t24/account/<accountNumber>)
- 3.2 (Optional) Write property test for T24 Transaction Persistence Round-Trip
- 3.3 (Optional) Write unit tests for T24 business logic

### 4. Implement IRIS REST API and TWS SOAP Service Simulators
- 4.1 Implement IRIS REST API Flask application (port 8081)
- 4.2 (Optional) Write property test for REST Response Format Validity
- 4.3 Implement TWS SOAP Service Flask application (port 8082)
- 4.4 (Optional) Write property test for SOAP Response Format Validity

### 5. Implement Side Systems Simulators
- 5.1 Implement CardMgmt Flask application (port 8083)
- 5.2 Implement Legacy System Flask application (port 8084)
- 5.3 Implement MSSQL logging client utility
- 5.4 (Optional) Write property test for MSSQL Log Persistence Round-Trip

### 6. Checkpoint - Ensure all backend simulators pass tests

### 7. Implement AppConnect Middleware Simulator
- 7.1 Implement AppConnect Flask application (port 8080, shared codebase for both nodes)
- 7.2-7.5 (Optional) Property tests for routing, orchestration, error responses, logging

### 8. Implement DataPower API Gateway Simulator
- 8.1 Implement DataPower Flask application (port 8443)
- 8.2-8.3 (Optional) Property tests for validation and forwarding

### 9. Implement F5 Load Balancer Simulator
- 9.1 Implement F5 Flask application (ports 80/443)
- 9.2 (Optional) Write property test for F5 Request Forwarding

### 10. Implement Channel Simulator
- 10.1 Implement Channel Simulator script with configurable traffic generator
- 10.2 Create YAML traffic profile configurations (low/medium/high)
- 10.3-10.4 (Optional) Property tests for request validity and traffic distribution

### 11. Checkpoint - Ensure all simulators and property tests pass

### 12. Create ESXi provisioning automation scripts
- 12.1 Create PowerCLI VM provisioning script
- 12.2 Create network configuration script
- 12.3 Create service deployment script
- 12.4-12.5 (Optional) Property tests for VM provisioning and network latency

### 13. Create Dynatrace OneAgent deployment and configuration
- 13.1 Create OneAgent installation script
- 13.2 Create Dynatrace dashboard configuration files
- 13.3 Create Dynatrace alerting rule configurations

### 14. Create environment validation script
- 14.1 Write validation and health check script
- 14.2 (Optional) Write property test for Service Response Time Compliance

### 15. Implement end-to-end integration tests
- 15.1 Write end-to-end transaction flow integration tests
- 15.2 (Optional) Write property test for End-to-End Log Trail

### 16. Checkpoint - Ensure all integration tests pass

### 17. Create traffic simulation runner
- 17.1 Write traffic simulation orchestration script

### 18. Create POC documentation package
- 18.1 Create architecture documentation
- 18.2 Create Dynatrace configuration documentation
- 18.3 Create findings and recommendations template

### 19. Final checkpoint - Full environment validation

## Notes

- Tasks marked (Optional) can be skipped for faster MVP
- All simulators use Python Flask for consistency and Dynatrace OneAgent auto-instrumentation compatibility
- Provisioning scripts are designed to be idempotent and re-runnable
- See full task details in the local specs directory
