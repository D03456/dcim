# Planning and Concept

## 1. Service Name

**Data Center Asset & Infrastructure Manager**

Short name: **DCIM Manager**

## 2. Service Overview

DCIM Manager is a lightweight SaaS product for centrally managing data centers, buildings, floors, areas, rack rows, racks, physical devices, network devices, IP subnets, maintenance contracts, and related operational information.

The service starts from small-scale use and is designed for data center operators, infrastructure engineers, corporate IT teams, MSPs, and SIers who currently manage infrastructure assets with spreadsheets or fragmented internal documents.

The initial product focuses on replacing spreadsheet-based infrastructure ledgers with a structured, searchable, and maintainable system.

## 3. Background and Problems

### 3.1 Current Problems

- Data center, rack, device, IP, and maintenance contract information is often scattered across spreadsheets or individually managed files.
- Rack installation positions, official device names, aliases, common names, and maintenance expiration dates are not consistently organized.
- Maintenance contract expiration checks tend to depend on individual knowledge and manual work.
- IP address ledgers and device information are often managed separately, making it difficult to check IP usage and availability.
- On-premises devices and future cloud resources may be managed separately, making the overall infrastructure picture hard to understand.
- Once operational data grows, adding physical hierarchy such as buildings, floors, areas, and rack rows later becomes difficult and costly.

### 3.2 Goals

- Manage data centers, buildings, floors, areas, rack rows, racks, devices, IP subnets, and maintenance contracts hierarchically.
- Link devices and maintenance contracts so that users can identify devices without contracts and contracts approaching expiration.
- Improve searchability with official names, common names, aliases, and tags.
- Provide a trial-friendly plan that allows users to evaluate the service before moving to paid plans.
- Design the domain model so that cloud resource management can be added in the future without disrupting the core physical asset model.

## 4. Target Users

| Category | Target User | Main Purpose |
|---|---|---|
| Corporate IT | Internal infrastructure teams | Manage servers, network devices, IP subnets, and maintenance contracts |
| Data center operators | DC operations teams | Manage racks, installation positions, facility hierarchy, and contacts |
| MSP / SIer | Customer infrastructure managers | Manage multiple customers and multiple data centers |
| Small businesses | Startups and SMBs | Replace spreadsheet-based infrastructure ledgers |
| Security / audit teams | Asset management and audit staff | Confirm managed devices, maintenance status, and locations |

## 5. Value Proposition

### 5.1 Core Value

- Centralized data center asset management
- Hierarchical facility management from data center to rack row and rack
- Rack-based device placement management
- IP subnet management
- Maintenance contract expiration management
- Improved searchability across assets
- Visualization of devices without maintenance contracts and expiration risks
- Minimum audit trail for asset and contract changes

### 5.2 Differentiation

- Lightweight DCIM focused on replacing spreadsheet-based infrastructure ledgers.
- Trial-friendly Free plan with a limited trial period instead of overly restrictive resource limits.
- Data center, rack, device, IP subnet, and maintenance contract management in one operational workflow.
- Search by official name, common name, alias, and tags.
- Bidirectional visibility between devices and maintenance contracts.
- Physical hierarchy support from the initial release, including buildings, floors, areas, and rack rows.
- Future expandability toward AWS and other cloud resource management.

## 6. Service Scope

### 6.1 Initial Release Scope

The following items are included in the initial release.

- Tenant management
- User management
- Plan management
- Data center management
- Building management
- Floor management
- Area management
- Rack row management
- Rack management
- Server device management
- Network device management
- IP subnet management
- Maintenance contract management
- Tag management
- Common name and alias management
- Notification feature
- Search and list views
- Plan limit control
- Minimum audit trail
  - Created by
  - Created at
  - Updated by
  - Updated at

### 6.2 Future Extension Scope

The following items are future extensions and are not included in the initial release.

- AWS account management
- AWS region management
- EC2 management
- Container management
- EKS Pod management
- Import / export
- Full audit log history
- Public API
- Fine-grained permission roles
- Rack diagram visualization
- Device templates
- Manufacturer device information integration

## 7. Plan Concept

The Free plan should be useful enough for evaluation, but limited by trial period.

| Plan | Trial / Contract | Data Center Limit | Rack Limit | Device Limit | Subnet Limit | User Limit |
|---|---|---:|---:|---:|---:|---:|
| Free | 14-day trial | 1 | 3 | 20 | 3 | 1 |
| Starter | Monthly / Annual | 2 | 5 | 50 | 10 | 3 |
| Business | Monthly / Annual | 5 | 50 | 100 | 50 | 10 |
| Enterprise | Custom | 10 | 100 | 1000 | 200 | 30 |

### 7.1 Free Plan Policy

The Free plan is intended for trial use.

- Usage period: 14 days
- Purpose: allow users to evaluate whether the service can replace spreadsheet-based management
- Resource limits are relaxed enough to register a small but realistic environment
- After the trial period, users should upgrade to a paid plan to continue using the service

## 8. Option Concept

| Option | Unit | Description |
|---|---:|---|
| Subnet addition | 10 subnets | Add manageable IP subnet capacity |
| Device addition | 100 devices | Add manageable device capacity |

## 9. Development Policy

| Item | Policy |
|---|---|
| Development method | Waterfall |
| Design policy | Domain separation with DDD awareness |
| Backend | Java / Spring Boot |
| Security | Spring Security |
| UI | Vaadin |
| Database | MariaDB |
| Build | Maven |
| Support | Lombok |
| Development support | Use OpenClaw for instruction and review support |

## 10. Initial Release Priority

### Must-have: Minimum Practical Version

- Tenant management
- User management
- Data center / building / floor / area / rack row / rack management
- Device management
- IP subnet management
- Maintenance contract management
- Search and list views
- Plan limit control
- Minimum audit trail

### Should-have: Practical Operation Features

- Tag management
- Common name and alias management
- Maintenance expiration notification
- Search for devices without maintenance contracts
- Maintenance contract and device linkage

### Could-have: Advanced Features

- CSV import / export
- Rack templates
- Rack diagram visualization
- Cloud resource management
- API integration
- Full audit log history
- Manufacturer device information integration
- Advanced permission management

## 11. Recommended Initial Policy

The initial development should include the minimum practical version plus selected practical operation features.

Specifically, the initial release should include:

- Physical hierarchy from data center to rack row and rack
- Device management
- IP subnet management
- Tag management
- Common name and alias management
- Maintenance contract and device linkage
- Search for devices without maintenance contracts
- Maintenance expiration notification two months before expiration
- Plan limit control
- Minimum audit trail

Reasons:

- If buildings, floors, areas, and rack rows are added later, the data model and existing data migration become more difficult.
- A simple asset ledger alone is not differentiated enough.
- Maintenance contracts, notifications, and searchability provide practical value from the beginning.
- Managing IP capacity by subnet is more natural than managing it by individual IP address count.
- Cloud resource management should remain a future extension to avoid expanding the initial scope too much.

## 12. Success Criteria

- Users feel that the service is worth migrating to from spreadsheets.
- Small users can evaluate the service during the 14-day Free trial.
- Users can start registering initial data within 30 minutes.
- Users can register at least one realistic rack environment during the trial.
- Users can search by device name, alias, IP subnet, and tag.
- Users can list devices with maintenance contracts expiring within two months.
- Users can list devices without maintenance contracts.
- The domain structure can be extended to cloud resource management in the future.

## 13. Future Considerations

- Final pricing
- Contract and billing method
- Tenant separation method
- Data import method
- Permission role design
- Whether full audit logs should be added soon after initial release
- Priority of cloud resource management
- Priority of rack diagram visualization
