# Requirements Document

## Introduction

This document defines the requirements for building a Proof of Concept (POC) environment on ESXi VMware that replicates HDB Bank's production banking architecture. The POC environment will simulate the full transaction flow from consumer channels through the middleware stack to the core banking system and database layer. Dynatrace will be deployed across all components to demonstrate end-to-end observability, distributed tracing, and infrastructure monitoring. The final deliverable is professional documentation suitable for presentation to HDB Bank stakeholders.

## Glossary

- **ESXi_Host**: The VMware ESXi hypervisor server with root access where all virtual machines are provisioned
- **POC_Environment**: The complete set of virtual machines and network configurations that replicate HDB Bank's banking architecture
- **VM**: A virtual machine running on the ESXi_Host
- **F5_VM**: A virtual machine simulating the F5 load balancer that distributes incoming traffic across backend services
- **DataPower_VM**: A virtual machine simulating the IBM DataPower API Gateway and Web Application Firewall
- **AppConnect_VM**: A virtual machine (2 nodes) simulating IBM App Connect Enterprise middleware with orchestration and business logic routing
- **IRIS_VM**: A virtual machine hosting REST API services that interface with the core banking system
- **TWS_VM**: A virtual machine hosting SOAP-based web services that interface with the core banking system
- **T24_VM**: A virtual machine simulating the Temenos T24 Core Banking System
- **Oracle_DB_VM**: A virtual machine running Oracle Database as the primary data store
- **CardMgmt_VM**: A virtual machine simulating the Card Management System connected to the middleware layer
- **Legacy_VM**: A virtual machine simulating the Legacy System connected to the middleware layer
- **MSSQL_VM**: A virtual machine running Microsoft SQL Server for logging and lookup operations
- **Channel_Simulator**: A component that generates synthetic traffic simulating consumer channels (Internet Banking, Mobile App, IBM BPM, IVR, Others)
- **Dynatrace_Server**: The Dynatrace monitoring platform (SaaS or Managed) that collects and visualizes telemetry data
- **OneAgent**: The Dynatrace agent deployed on each VM to collect metrics, traces, and logs
- **Dynatrace_ActiveGate**: An optional Dynatrace component that routes monitoring traffic from OneAgents to the Dynatrace_Server
- **POC_Documentation**: The professional documentation package describing the environment, architecture, monitoring setup, and results for HDB Bank presentation
- **Transaction_Flow**: The end-to-end path of a request from a consumer channel through the infrastructure stack to the database and back

## Requirements

### Requirement 1: ESXi Virtual Machine Provisioning
**User Story:** As a POC engineer, I want to provision virtual machines on the ESXi host for each component in the banking architecture, so that the environment accurately replicates HDB Bank's production topology.
#### Acceptance Criteria
1. THE ESXi_Host SHALL host a dedicated VM for each of the following components: F5_VM, DataPower_VM, AppConnect_VM (2 nodes), IRIS_VM, TWS_VM, T24_VM, Oracle_DB_VM, CardMgmt_VM, Legacy_VM, and MSSQL_VM
2. WHEN a VM is provisioned, THE ESXi_Host SHALL allocate CPU, memory, and disk resources according to the component sizing specifications defined in the POC resource plan
3. WHEN all VMs are provisioned, THE POC_Environment SHALL contain a minimum of 11 virtual machines representing the complete banking architecture
4. THE ESXi_Host SHALL use descriptive VM naming conventions that identify each component's role (e.g., HDB-POC-F5, HDB-POC-DataPower)

### Requirement 2: Network Topology and Segmentation
**User Story:** As a POC engineer, I want to configure network connectivity between VMs that mirrors the production network flow, so that traffic routing accurately represents the real banking environment.
#### Acceptance Criteria
1. THE POC_Environment SHALL configure virtual network segments that separate consumer-facing traffic, middleware traffic, backend service traffic, and database traffic
2. THE F5_VM SHALL be reachable from the Channel_Simulator and SHALL route traffic to the DataPower_VM
3. THE DataPower_VM SHALL route validated requests to the AppConnect_VM nodes
4. THE AppConnect_VM SHALL route requests to IRIS_VM, TWS_VM, CardMgmt_VM, Legacy_VM, and MSSQL_VM based on business logic routing rules
5. THE IRIS_VM and TWS_VM SHALL communicate with the T24_VM and Oracle_DB_VM
6. WHEN a network path between two connected components is tested, THE POC_Environment SHALL confirm bidirectional connectivity with latency below 5ms within the ESXi_Host

### Requirement 3: F5 Load Balancer Simulation
**User Story:** As a POC engineer, I want to simulate F5 load balancing behavior, so that incoming requests are distributed and the entry point of the architecture is accurately represented.
#### Acceptance Criteria
1. THE F5_VM SHALL accept incoming HTTP and HTTPS requests from the Channel_Simulator
2. WHEN a request arrives at the F5_VM, THE F5_VM SHALL forward the request to the DataPower_VM
3. THE F5_VM SHALL expose a health check endpoint that returns the operational status of the load balancer
4. WHILE the F5_VM is operational, THE F5_VM SHALL log all incoming requests with timestamp, source IP, and destination

### Requirement 4: IBM DataPower API Gateway Simulation
**User Story:** As a POC engineer, I want to simulate IBM DataPower as an API Gateway and WAF, so that API routing and security filtering behavior is represented in the POC.
#### Acceptance Criteria
1. WHEN a request is received from the F5_VM, THE DataPower_VM SHALL validate the request headers and payload structure
2. IF a request fails validation, THEN THE DataPower_VM SHALL return an HTTP 400 error response with a descriptive error message
3. WHEN a valid request is received, THE DataPower_VM SHALL route the request to the appropriate AppConnect_VM node
4. THE DataPower_VM SHALL log all requests with timestamp, request method, endpoint path, and validation result

### Requirement 5: IBM App Connect Middleware Simulation
**User Story:** As a POC engineer, I want to simulate IBM App Connect Enterprise middleware with two active nodes, so that orchestration and business logic routing behavior is demonstrated.
#### Acceptance Criteria
1. THE POC_Environment SHALL deploy two active AppConnect_VM nodes to simulate the production active-active middleware configuration
2. WHEN a request is received from the DataPower_VM, THE AppConnect_VM SHALL route the request to the appropriate backend service based on the request type
3. THE AppConnect_VM SHALL implement orchestration logic that coordinates calls to multiple backend services for composite transactions
4. WHEN a backend service is unreachable, THE AppConnect_VM SHALL return a structured error response indicating the failed service and error details
5. THE AppConnect_VM SHALL log all routing decisions with timestamp, source request, target service, and routing rule applied

### Requirement 6: IRIS REST API and TWS SOAP Service Simulation
**User Story:** As a POC engineer, I want to simulate IRIS REST APIs and TWS SOAP services, so that both API styles used in the banking architecture are represented.
#### Acceptance Criteria
1. THE IRIS_VM SHALL expose REST API endpoints that simulate core banking operations (account inquiry, fund transfer, balance check)
2. THE TWS_VM SHALL expose SOAP service endpoints that simulate legacy banking operations (statement retrieval, standing order management)
3. WHEN a REST request is received, THE IRIS_VM SHALL process the request and return a JSON response with simulated banking data
4. WHEN a SOAP request is received, THE TWS_VM SHALL process the request and return an XML response with simulated banking data
5. THE IRIS_VM and TWS_VM SHALL communicate with the T24_VM to retrieve and persist simulated transaction data

### Requirement 7: Temenos T24 Core Banking Simulation
**User Story:** As a POC engineer, I want to simulate the Temenos T24 core banking system, so that the central transaction processing layer is represented in the POC.
#### Acceptance Criteria
1. THE T24_VM SHALL expose service endpoints that accept transaction requests from IRIS_VM and TWS_VM
2. WHEN a transaction request is received, THE T24_VM SHALL validate the transaction, process the simulated business logic, and persist the result to the Oracle_DB_VM
3. THE T24_VM SHALL maintain simulated customer account data including account numbers, balances, and transaction history
4. IF a transaction request contains invalid data, THEN THE T24_VM SHALL return a structured error response with a specific error code and description

### Requirement 8: Oracle Database Provisioning
**User Story:** As a POC engineer, I want to deploy Oracle Database as the primary data store, so that the POC accurately represents the production database layer.
#### Acceptance Criteria
1. THE Oracle_DB_VM SHALL run an Oracle Database instance configured as the primary data store for the T24_VM
2. THE Oracle_DB_VM SHALL contain a schema with tables representing core banking entities (accounts, transactions, customers)
3. THE Oracle_DB_VM SHALL be pre-populated with sample data sufficient to demonstrate realistic transaction flows
4. WHEN a query is executed against the Oracle_DB_VM, THE Oracle_DB_VM SHALL return results within 100ms for standard lookup operations

### Requirement 9: Side Systems Simulation
**User Story:** As a POC engineer, I want to simulate the Card Management System, Legacy System, and MSSQL logging database, so that the full middleware integration landscape is represented.
#### Acceptance Criteria
1. THE CardMgmt_VM SHALL expose service endpoints that simulate card issuance, card status inquiry, and card blocking operations
2. THE Legacy_VM SHALL expose service endpoints that simulate legacy system interactions using fixed-format message protocols
3. THE MSSQL_VM SHALL run a Microsoft SQL Server instance configured for logging and lookup operations
4. WHEN the AppConnect_VM sends a request to the CardMgmt_VM, THE CardMgmt_VM SHALL return a simulated response within 200ms
5. WHEN the AppConnect_VM writes a log entry to the MSSQL_VM, THE MSSQL_VM SHALL persist the log record with timestamp, transaction ID, and log level

### Requirement 10: Consumer Channel Traffic Simulation
**User Story:** As a POC engineer, I want to simulate traffic from consumer channels (Internet Banking, Mobile App, IBM BPM, IVR), so that realistic end-to-end transaction flows can be demonstrated.
#### Acceptance Criteria
1. THE Channel_Simulator SHALL generate HTTP/HTTPS requests that simulate traffic from Internet Banking, Mobile App, IBM BPM, IVR, and other consumer channels
2. THE Channel_Simulator SHALL support configurable traffic volume to simulate low, medium, and high load scenarios
3. WHEN the Channel_Simulator generates a request, THE Channel_Simulator SHALL include headers identifying the originating channel type
4. THE Channel_Simulator SHALL generate a mix of transaction types including account inquiries, fund transfers, card operations, and statement retrievals

### Requirement 11: Dynatrace OneAgent Deployment
**User Story:** As a POC engineer, I want to deploy Dynatrace OneAgent on every VM in the POC environment, so that full-stack monitoring data is collected from all components.
#### Acceptance Criteria
1. THE POC_Environment SHALL have OneAgent installed and running on every VM
2. WHEN OneAgent is deployed on a VM, THE OneAgent SHALL automatically discover the running processes and services on that VM
3. WHEN OneAgent is running, THE OneAgent SHALL report host metrics (CPU, memory, disk, network) to the Dynatrace_Server at intervals no greater than 60 seconds
4. IF OneAgent loses connectivity to the Dynatrace_Server, THEN THE OneAgent SHALL buffer monitoring data locally and transmit the buffered data when connectivity is restored

### Requirement 12: Dynatrace Distributed Tracing
**User Story:** As a POC engineer, I want Dynatrace to capture distributed traces across the full transaction flow, so that end-to-end request paths through the banking architecture are visible.
#### Acceptance Criteria
1. WHEN a request traverses the full stack, THE Dynatrace_Server SHALL display the complete distributed trace as a single PurePath
2. THE Dynatrace_Server SHALL capture response times for each service call within a distributed trace
3. WHEN a transaction involves calls to side systems, THE Dynatrace_Server SHALL include those calls in the distributed trace
4. THE Dynatrace_Server SHALL automatically detect and map service dependencies between all monitored components

### Requirement 13: Dynatrace Dashboard and Alerting Configuration
**User Story:** As a POC engineer, I want to configure Dynatrace dashboards and alerting rules, so that the monitoring capabilities are clearly demonstrated to HDB Bank stakeholders.
#### Acceptance Criteria
1. THE Dynatrace_Server SHALL display a topology dashboard showing all POC_Environment components and their interconnections
2. THE Dynatrace_Server SHALL display a transaction flow dashboard showing end-to-end response times, throughput, and error rates
3. THE Dynatrace_Server SHALL display an infrastructure dashboard showing CPU, memory, disk, and network utilization for each VM
4. WHEN a service response time exceeds a configured threshold, THE Dynatrace_Server SHALL generate an alert within 120 seconds
5. WHEN a VM's CPU utilization exceeds 90% for more than 2 minutes, THE Dynatrace_Server SHALL generate an infrastructure alert

### Requirement 14: POC Documentation Package
**User Story:** As a POC engineer, I want to produce professional documentation covering the environment setup, architecture, monitoring configuration, and results, so that the POC can be formally presented to HDB Bank.
#### Acceptance Criteria
1. THE POC_Documentation SHALL include an architecture diagram showing all VMs, network segments, and traffic flows
2. THE POC_Documentation SHALL include a VM inventory table listing each VM's name, role, OS, IP address, CPU, memory, and disk allocation
3. THE POC_Documentation SHALL include a Dynatrace configuration section describing OneAgent deployment, dashboard setup, and alerting rules
4. THE POC_Documentation SHALL include screenshots or exports of Dynatrace dashboards
5. THE POC_Documentation SHALL include a findings and recommendations section
6. THE POC_Documentation SHALL be formatted in a professional presentation-ready style suitable for HDB Bank executive and technical stakeholders

### Requirement 15: End-to-End Transaction Flow Validation
**User Story:** As a POC engineer, I want to validate that a transaction flows correctly from the consumer channel through every layer to the database and back.
#### Acceptance Criteria
1. WHEN the Channel_Simulator sends an account inquiry request, THE POC_Environment SHALL process the request through the full stack and return a valid response
2. WHEN the Channel_Simulator sends a fund transfer request, THE POC_Environment SHALL process the request and return a response within 2 seconds
3. WHEN the Channel_Simulator sends a card operation request, THE POC_Environment SHALL route the request through AppConnect_VM to CardMgmt_VM and return a valid response
4. FOR EACH end-to-end transaction type, THE POC_Environment SHALL produce log entries at every component in the transaction path
