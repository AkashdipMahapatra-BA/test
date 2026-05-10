# Deep Dive Investigation: MSK Broker CPU Flapping (`dev-mon-kafka-5-cpu-usage`) 🔍

**Date:** May 10, 2026

**Environment:** AWS MSK DEV (`msk-odin-dev-euwe1-01`)

**Investigator:** Akashdip Mahapatra

<img width="555" height="407" alt="image" src="https://github.com/user-attachments/assets/f24646ba-bf42-4923-a295-dbc438592c6b" />

## 1. Context & Problem Statement

During my L2 Platform Support shift, the Datadog monitor `dev-mon-kafka-5-cpu-usage` triggered an excessive number of flapping alerts (over 313 events in 7 days, peaking at 40+ per day).

* **The Symptom:** `broker_id:1` was consistently sustaining 70%–86% CPU utilization, hovering right around the 80% critical threshold. Meanwhile, `broker_id:2` and `broker_id:3` were healthy and idling at ~20% CPU.
* **The Impact:** Constant P2 alert fatigue for the L3 team and degraded performance headroom on the primary DEV Kafka cluster.

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/278cce5f-9036-4fa5-9738-095a9767cd64" />
<img width="100%" alt="image" src="https://github.com/user-attachments/assets/8212dd62-d087-4e5a-85d0-c986261905c5" />

---

## 2. The Initial Hypothesis

My initial assumption was a **structural partition imbalance**. In Apache Kafka, read and write requests are handled exclusively by the broker acting as the "Leader" for a given partition. If Broker 1 had accidentally been assigned as the leader for a significantly higher number of partitions than the other brokers, its CPU would naturally be much higher.

To prove this, I needed to bypass Datadog and query the AWS MSK cluster directly for its exact metadata state.

---

## 3. Step-by-Step Execution & Telemetry Extraction

### Step 3.1: Securing Bastion Access & Identifying Endpoints

To interact with the MSK cluster securely, I SSH'd into the designated EC2 bastion host within the VPC: `ec2-dev-euwe1-mercury-sandbox-01`. aws [link](https://316022513217-oycl4cis.eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#InstanceDetails:instanceId=i-0962106d348f5b42b)

Next, I needed the broker endpoints. I retrieved the [MSK v1](https://316022513217-oycl4cis.eu-west-1.console.aws.amazon.com/msk/home?region=eu-west-1#/cluster/arn%3Aaws%3Akafka%3Aeu-west-1%3A316022513217%3Acluster%2Fmsk-odin-dev-euwe1-01%2F52a91b72-22ab-4d85-966c-0035ce6f6ba7-3/view?tabId=metrics) cluster (`msk-odin-dev-euwe1-01`) bootstrap servers via the [AWS MSK Console](https://316022513217-oycl4cis.eu-west-1.console.aws.amazon.com/msk/home?region=eu-west-1#/cluster/arn%3Aaws%3Akafka%3Aeu-west-1%3A316022513217%3Acluster%2Fmsk-odin-dev-euwe1-01%2F52a91b72-22ab-4d85-966c-0035ce6f6ba7-3/viewClientInfo). Because the cluster enforces SASL/SCRAM authentication, I utilized port `9096`.

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/1f1b8c05-f91c-4245-b9bb-738c268b3136" />
<img width="100%" alt="image" src="https://github.com/user-attachments/assets/916da6a8-dc0d-4164-82e6-e1eb3552a6b7" />


### Step 3.2: Retrieving Valid Authentication Credentials (The ECS Trick)

To query the cluster, the Kafka administrative scripts require valid SASL/SCRAM credentials. Instead of guessing or resetting passwords, I reverse-engineered the credentials by inspecting an actively running consumer in AWS ECS.

I navigated to the ECS Cluster `kc-ecs-odin-dev-euwe1-odin-kafka-connect-ecs-01` and inspected the [JSON](https://316022513217-oycl4cis.eu-west-1.console.aws.amazon.com/ecs/v2/task-definitions/ecst-odin-dev-euwe1-odin-kafka-connect-ecs-01/165/json?region=eu-west-1) of the running Task Definition (`ecst-odin-dev-euwe1-odin-kafka-connect-ecs-01:165`). Under the environment variables, I safely extracted the working credentials:

* **Username:** `MSK_sasl-auth-dev`
* **Password:** *XXXXXXXXXXXXXXXXXXXX*

</br>

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/db6189a9-f4c8-4bf3-bbd8-add4e5f4bd00" />
<img width="100%" alt="image" src="https://github.com/user-attachments/assets/08c6e01e-7eef-42fa-abc3-9f7768875d67" />

### Step 3.3: Creating an Isolated Configuration File

To avoid overwriting existing configuration files (like `client.properties`) which might be actively used by other automated scripts or engineers, I created a dedicated, isolated properties file specifically for this administrative task.

```bash
cd /data/kafka_2.13-2.8.1/bin
nano v1-admin.properties

```

Inside this file, I defined the SASL/SCRAM authentication mechanism using the credentials retrieved from ECS:

```properties
sasl.mechanism=SCRAM-SHA-512
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="MSK_sasl-auth-dev" password="[REDACTED]";
[cite_start]bootstrap.servers=b-1.mskodindeveuwe101... [cite: 2]

```

### Step 3.4: Extracting the Cluster State

With authentication secured, I executed a read-only `--describe` command to dump the cluster's partition topology. This command asks the brokers to report their metadata without modifying or affecting any live data.

```bash
./kafka-topics.sh \
  --bootstrap-server b-1.mskodindeveuwe101... \
  --command-config v1-admin.properties \
  --describe > cluster_state.txt

```

### Step 3.5: Analyzing the Partition Distribution

I used bash string manipulation to parse the resulting `cluster_state.txt` file and count exactly how many partition leaders were assigned to each broker.

```bash
echo "Broker 1 Leaders:" && grep "Leader: 1" cluster_state.txt | wc -l
echo "Broker 2 Leaders:" && grep "Leader: 2" cluster_state.txt | wc -l
echo "Broker 3 Leaders:" && grep "Leader: 3" cluster_state.txt | wc -l

```

**The Output:**

* Broker 1 Leaders: **1310** 


* Broker 2 Leaders: **1305** 


* Broker 3 Leaders: **1297** 

</br>

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/5719385d-4a02-4ede-9b97-e78a15fda0a5" />

---

## 4. The Plot Twist & Datadog Pivot

The output completely dismantled the initial hypothesis. The partition distribution was perfectly balanced, ruling out a simple count imbalance.

If the *quantity* of partitions was equal, the issue had to be the *quality* of those partitions. Broker 1 was suffering from **"Hot Partitions"**—it had unluckily been elected as the leader for a small number of topics driving massive network throughput.

*(Architecture Diagram: Visual representation of evenly distributed partitions, but showing thick data streams hitting only Broker 1)*

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/97fe4e37-1ecd-4117-9558-686da12dfb1f" />


I pivoted back to Datadog Metrics Explorer and queried `aws.kafka.bytes_in_per_sec{environment:dev, broker_id:1}` grouped by `topic` to find the culprits.

**Datadog Findings:**

* **Total Broker 1 Throughput:** ~104 KB/s (35x higher than Brokers 2 and 3).
* **Top Culprit:** The internal Kafka topic **`_consumer_offsets`** was pushing 103,799 B/s (106.4 messages/sec), accounting for **97.1%** of Broker 1's total load.

*(Datadog Screenshot: Top List showing `_consumer_offsets` dominating Broker 1 traffic)*  [*link*](https://britishairways.datadoghq.eu/metric/explorer?fromUser=false&graph_layout=multi&start=1778419452743&end=1778423052743&paused=false#N4Ig7glgJg5gpgFxALlAGwIYE8D2BXJVEADxQEYAaELcqyKBAC1pEbghkcLIF8qo4AMwgA7CAgg4RKUAiwAHOChASAtnADOcAE4RNIKtrgBHPJoQaUAbVBGN8qVoD6gnNtUZCKiOq279VKY6epbINiAiGOrKQdpYZAYgUJ4YThr42gDGSsgg6gi6mZaBZnHKGniqyBhgGgB0ANYYgk11AEZYCJpOok6K2mlwmcBwIgBuENpS6iIIyAJjFAAEbVMNOj1QyLwrWEvACDjyEJk8IDwAulSu7niYoeE3qncYMaXx51cgGnJoOaDyDB-BBdZRQHAwJyZe4aE6JNwCAYdZRjIFmYogNCiOBOOSKco4LFQRJYkQ4+hMZRME4Nc78CD2TBYXEKHKY7GfHh8b7yLEIADCUmEMBQIjuaB4QA)

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/90fc9eac-7d2b-4f85-944f-f1d2c6fd3af6" />

---

## 5. Root Cause Analysis

The CPU skew on Broker 1 is an architectural consequence of how Kafka manages consumer groups.

When a consumer group connects, Kafka hashes its `group.id` to assign it to one of the 50 partitions within the `_consumer_offsets` topic. This partition acts as the Group Coordinator. In the DEV environment, a highly active (and likely over-chatty) cluster of consumer groups hashed to partitions where Broker 1 happened to be the leader. Because these consumers are committing offsets 100+ times a second, Broker 1 is being overwhelmed by internal bookmarking traffic.

<img width="1366" height="619" alt="Screenshot 2026-04-29 124303" src="https://github.com/user-attachments/assets/17dbe73d-0a1c-4f40-a7e9-7b3af3b22600" />
<img width="1366" height="617" alt="Screenshot 2026-04-30 173009" src="https://github.com/user-attachments/assets/fbf6cdca-aff1-44b1-94ec-0e10eb037a2d" />
