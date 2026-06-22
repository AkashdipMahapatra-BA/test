I've reviewed both runbooks in detail. Here's a comprehensive breakdown of what needs updating in your two assigned runbooks (IIDIP-24855). The issues fall into **3 categories**: broken Confluence links (IAG Tech → BA migration), potentially outdated MSK/AWS components (MSK migration), and stale Datadog links.

---

## 📘 Runbook 1: [Runbook for Flight Status (FS)](/notebook/297692/runbook-for-flight-status-fs)

### 🔴 Category 1: Broken Confluence Links (iagtech.atlassian.net → BA Confluence)

| Section | Link Label | Current Broken URL | Action |
|---|---|---|---|
| 2. Architecture | Architecture Link | `iagtech.atlassian.net/wiki/spaces/IO/pages/1668912764` | Replace with BA Confluence equivalent |
| 2. Architecture | Data Dictionary | `iagtech.atlassian.net/wiki/spaces/IO/pages/1407358052` | Replace with BA Confluence equivalent |
| 2. Architecture | Monitoring Link | `iagtech.atlassian.net/wiki/spaces/IO/pages/1671866838` | Replace with BA Confluence equivalent |
| 2. Architecture | Flight Status SNS Link | `iagtech.atlassian.net/wiki/spaces/IO/pages/1653245704` | Replace with BA Confluence equivalent |
| 2. Architecture | Flight Status Data Link | `iagtech.atlassian.net/wiki/spaces/CHAT/pages/883064985` | Replace with BA Confluence equivalent |
| 2. Architecture | LLD | `iagtech.atlassian.net/wiki/spaces/IO/pages/1706917890` | Replace with BA Confluence equivalent |
| 7. SNS Config | SNS Forwarder Lambda Config | `iagtech.atlassian.net/wiki/spaces/IO/pages/1525450676` | Replace with BA Confluence equivalent |

### 🟡 Category 2: MSK Migration / AWS Component Updates

| Section | Component | Current Value | Action |
|---|---|---|---|
| 5. Env Config | MSK Broker b-1 | `b-1.mskodinprodeuwe101.91cqxg.c3.kafka.eu-west-1.amazonaws.com:9096` | Verify if MSK cluster ID changed post-migration |
| 5. Env Config | MSK Broker b-2 | `b-2.mskodinprodeuwe101.91cqxg.c3.kafka.eu-west-1.amazonaws.com:9096` | Same as above |
| 5. Env Config | MSK Broker b-3 | `b-3.mskodinprodeuwe101.91cqxg.c3.kafka.eu-west-1.amazonaws.com:9096` | Same as above |
| 4.2 Kafka Connectors | ECS Cluster name | `kc-ecs-odin-uat-euwe1-odin-kafka-connect-ecs-01` | ⚠️ Says **UAT** but used in prod context — likely needs prod cluster name |
| 4.2 Kafka Connectors | ECS Service name | `kc-ecs--flightstatus--archive-odin-uat-euwe1-odin-kafka-connect-ecs-01` | ⚠️ Same UAT/prod mismatch — update to prod name |
| 6.2 Data Ingestion | ECS Cluster (repeated) | Same UAT names used in troubleshooting steps | Update to match prod |
| 6.1 MSK commands | Bootstrap server in kafka-topics.sh commands | Uses `<env>` placeholder — verify MSK cluster ID | Confirm correct post-migration value |

### 🟠 Category 3: Stale Datadog Links

| Section | Link | Issue |
|---|---|---|
| 6.1 Data Storage | "Datadog MSK Log" | Has hardcoded timestamps (`from_ts`/`to_ts`) — will show old time window |
| 6.2.3 Data Filtration | "Link to Lambda Logs" | Has hardcoded timestamps + no service filter (shows all non-warn/info/ok logs, not FS-specific) |

---

## 📘 Runbook 2: [Runbook for Aircraft Towing](/notebook/297699/runbook-for-aircraft-towing)

### 🔴 Category 1: Broken Confluence Links (iagtech.atlassian.net → BA Confluence)

| Section | Link Label | Current Broken URL | Action |
|---|---|---|---|
| 2. Architecture | Architecture and Design | `iagtech.atlassian.net/wiki/spaces/IO/pages/1838219339` | Replace with BA Confluence equivalent |
| 2. Architecture | Aircraft Towing | `iagtech.atlassian.net/wiki/spaces/IO/pages/1789559808` | Replace with BA Confluence equivalent |
| 2. Architecture | Data Product | `iagtech.atlassian.net/wiki/spaces/IO/pages/1818302015` | Replace with BA Confluence equivalent |
| 2. Architecture | Data Mapping | `iagtech.atlassian.net/wiki/spaces/IO/pages/1503002767` | Replace with BA Confluence equivalent |
| 2. Architecture | Towing SNS Contract | `iagtech.atlassian.net/wiki/spaces/IO/pages/1789200146` | Replace with BA Confluence equivalent |
| 2. Architecture | Requirement Specification | `iagtech.atlassian.net/wiki/spaces/IO/pages/1777041413` | Replace with BA Confluence equivalent |
| 2. Architecture | Towing DoR | `iagtech.atlassian.net/wiki/spaces/IO/pages/1759150107` | Replace with BA Confluence equivalent |
| 2. Architecture | Monitoring Alerts | `iagtech.atlassian.net/wiki/spaces/IO/pages/1898938590` | Replace with BA Confluence equivalent |
| 4.1 Source System | Architecture (repeated) | `iagtech.atlassian.net/wiki/spaces/IO/pages/1838219339` | Same as above |
| 9. SNS Config | SNS Forwarder Lambda Config | `iagtech.atlassian.net/wiki/spaces/IO/pages/1525450676` | Replace with BA Confluence equivalent |
| 7. QA Testing | Jira Testing Board | `iagtech.atlassian.net/projects/IIDIP?...` | ⚠️ This is a Jira link (not Confluence) — verify if Jira also migrated |

### 🟡 Category 2: MSK Migration / AWS Component Updates

| Section | Component | Current Value | Action |
|---|---|---|---|
| 5. Env Config | MSK Broker b-1 | `b-1.mskodinprodeuwe101.rxzqlq.c3.kafka.eu-west-1.amazonaws.com` | Verify if MSK cluster ID changed post-migration |
| 5. Env Config | MSK Broker b-2 | `b-2.mskodinprodeuwe101.rxzqlq.c3.kafka.eu-west-1.amazonaws.com` | Same |
| 5. Env Config | MSK Broker b-3 | `b-3.mskodinprodeuwe101.rxzqlq.c3.kafka.eu-west-1.amazonaws.com` | Same |
| 6.1 MSK commands | Bootstrap server in kafka-topics.sh | Uses above broker addresses | Update if cluster ID changed |
| 4.2 DMS Connectors | DMS Source Endpoint | `dms-odin-uat-euwe1-aodb-towing-ora-source-endpoint-01` | ⚠️ Section header says "In UAT" — verify if prod details needed |
| 4.2 DMS Connectors | DMS Target Endpoint | `dms-odin-uat-euwe1-aodb-towing-s3-target-endpoint-01` | Same |
| 4.2 DMS Connectors | DMS Task URLs | Point to `dms-odin-uat-euwe1-...` in AWS console | Update to prod URLs if applicable |

### 🟠 Category 3: Stale Datadog Links

| Section | Link | Issue |
|---|---|---|
| Datadog Lambda Url | "Lambda" log link | Has hardcoded timestamps + uses generic query `towing` without service filter |
| 6.3.1 Data Filtration | "Link to Datadog Lambda logs" | Hardcoded timestamps + no service/function filter |
| 6.1.1 Data Storage | "Datadog MSK Log" | Hardcoded timestamps |

---

## 📋 Summary: Your Action Items

| Category | Flight Status (FS) | Aircraft Towing | Total |
|---|---|---|---|
| 🔴 Broken Confluence links | 7 links | 11 links | **18 links** |
| 🟡 MSK/AWS component updates | 6 items | 7 items | **13 items** |
| 🟠 Stale Datadog links | 2 links | 3 links | **5 links** |

### Recommended approach:
1. **Confluence links** — Search each page title in your BA Confluence (e.g., "Flight Status - Architecture Glossary") and get the new URL
2. **MSK brokers** — Check the current MSK cluster in AWS Console (prod account `958211232326`) for the correct bootstrap server addresses post-migration
3. **ECS/DMS names** — The UAT names referenced in prod context are likely a pre-existing issue exposed by this review — confirm the correct prod names with Vikas
4. **Datadog links** — Replace hardcoded timestamp URLs with `live=true` URLs (remove `from_ts`/`to_ts` params) and add proper service filters (e.g., `service:odie` or function-specific)

Want me to help generate corrected Datadog log links with proper filters for either runbook?
