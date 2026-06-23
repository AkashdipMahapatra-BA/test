Great work! I can see you've already updated the **Section 2 Confluence links** in both runbooks and the **MSK brokers in Section 5** of Flight Status. Here's the proof table and remaining items.

---

## ✅ What You've Already Updated (Old → New)

### Flight Status (FS) - Confluence Links (Section 2)
| Link | Old (iagtech) | New (britishairways) |
|---|---|---|
| Architecture | `iagtech.atlassian.net/.../1668912764` | `britishairways.atlassian.net/.../104320861` ✅ |
| Data Dictionary | `iagtech.atlassian.net/.../1407358052` | `britishairways.atlassian.net/.../104248596` ✅ |
| Monitoring | `iagtech.atlassian.net/.../1671866838` | `britishairways.atlassian.net/.../104321355` ✅ |
| SNS Link | `iagtech.atlassian.net/.../1653245704` | `britishairways.atlassian.net/.../104137165` ✅ |
| Data Link | `iagtech.atlassian.net/.../883064985` | `britishairways.atlassian.net/.../104248596` ✅ |
| LLD | `iagtech.atlassian.net/.../1706917890` | `britishairways.atlassian.net/.../104137685` ✅ |

### Flight Status (FS) - MSK Brokers (Section 5)
| Broker | Old | New |
|---|---|---|
| b-1 | `...91cqxg.c3.kafka...` | `...rxzqlq.c3.kafka...:9096` ✅ |
| b-2 | `...91cqxg.c3.kafka...` | `...rxzqlq.c3.kafka...:9096` ✅ |
| b-3 | `...91cqxg.c3.kafka...` | `...rxzqlq.c3.kafka...:9096` ✅ |

### Aircraft Towing - Confluence Links (Section 2)
| Link | Old (iagtech) | New (britishairways) |
|---|---|---|
| Architecture | `iagtech.atlassian.net/.../1838219339` | `britishairways.atlassian.net/.../104357944` ✅ |
| Towing | `iagtech.atlassian.net/.../1789559808` | `britishairways.atlassian.net/.../104137112` ✅ |
| Data Product | `iagtech.atlassian.net/.../1818302015` | `britishairways.atlassian.net/.../104357944` ✅ |
| Data Mapping | `iagtech.atlassian.net/.../1503002767` | `britishairways.atlassian.net/.../104137346` ✅ |
| SNS Contract | `iagtech.atlassian.net/.../1789200146` | `britishairways.atlassian.net/.../104343321` ✅ |
| Requirement | `iagtech.atlassian.net/.../1777041413` | `britishairways.atlassian.net/.../104137327` ✅ |
| DoR | `iagtech.atlassian.net/.../1759150107` | `britishairways.atlassian.net/.../104339759` ✅ |
| Monitoring | `iagtech.atlassian.net/.../1898938590` | `britishairways.atlassian.net/.../104137930` ✅ |

---

## 🔴 Still Needs Updating

### Flight Status (FS)

| # | Section | Issue | Current (Broken) | What To Do |
|---|---|---|---|---|
| 1 | 6.1 kafka-topics.sh commands | Old cluster ID in commands | `b-1.mskodin<env>euwe101.91cqxg...` (x3 commands) | Replace `91cqxg` → `rxzqlq` |
| 2 | 4.2 Kafka Connectors | UAT ECS cluster name | `kc-ecs-odin-uat-euwe1-odin-kafka-connect-ecs-01` | Get prod ECS cluster name |
| 3 | 4.2.3 | UAT ECS service name | `kc-ecs--flightstatus--archive-odin-uat-euwe1-odin-kafka-connect-ecs-01` | Get prod service name |
| 4 | 6.2 Troubleshooting | Repeats UAT ECS names (steps ii, iii, iv + Note section) | Same UAT names x4 | Same prod names |
| 5 | 7. SNS Config | iagtech Confluence link still there | `iagtech.atlassian.net/.../1525450676` | Find BA Confluence equivalent |
| 6 | 6.1 | Stale Datadog MSK log link | Hardcoded `from_ts`/`to_ts` | Use `live=true` without timestamps |
| 7 | 6.2.3 | Stale Datadog Lambda log link | Hardcoded timestamps, no service filter | Add `functionname:*flight*status*` filter |

### Aircraft Towing

| # | Section | Issue | Current (Broken) | What To Do |
|---|---|---|---|---|
| 1 | 4.1 Source System | iagtech link still there | `iagtech.atlassian.net/.../1838219339` | Use `britishairways.atlassian.net/.../104357944` (same as your Sec 2) |
| 2 | 9. SNS Config | iagtech Confluence link | `iagtech.atlassian.net/.../1525450676` | Find BA Confluence equivalent |
| 3 | 7. QA Testing | iagtech Jira link | `iagtech.atlassian.net/projects/IIDIP?...` | Update to `britishairways.atlassian.net/projects/IIDIP?...` (ref: GTA uses `britishairways.atlassian.net`) |
| 4 | 5. MSK brokers | Missing `:9096` port | `b-X.mskodinprodeuwe101.rxzqlq...:` **no port** | Append `:9096` to all 3 brokers |
| 5 | 4.2 DMS section | Says "In UAT" with UAT details | UAT endpoint names, UAT account `316022513217` | Update to PROD like GTA did (ref: GTA uses `dms-odin-prod-euwe1-...`, account `958211232326`) |
| 6 | 4.5 S3 table DMS URLs | Point to UAT DMS tasks | `dms-odin-uat-euwe1-...` in AWS console links | Change `uat` → `prod` in the URLs |
| 7 | 5. Datadog Lambda | Stale log link | Hardcoded timestamps | Use `live=true` and add `towing` service filter |
| 8 | 6.1.1 | Stale Datadog MSK log link | Hardcoded timestamps | Same fix as FS |
| 9 | 6.3.1 | Stale Datadog Lambda log link | Hardcoded timestamps, no service filter | Add proper `functionname:*towing*` filter |

---

## 📊 Progress Summary

| Runbook | Total Items | ✅ Done | 🔴 Remaining |
|---|---|---|---|
| **Flight Status (FS)** | 16 | 9 (Confluence + MSK brokers) | **7** |
| **Aircraft Towing** | 19 | 8 (Section 2 Confluence) | **9** (incl. 2 easy ones) |

The quickest wins are items **#1 (Towing Sec 4.1)** — you already have the correct URL from your Section 2 update — and **#4 (Towing port)** — just append `:9096`. For the DMS section (#5), use the GTA runbook as your template since it shows the PROD pattern clearly.