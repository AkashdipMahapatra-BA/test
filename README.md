<img width="1358" height="11740" alt="screencapture-britishairways-atlassian-net-wiki-spaces-IO-pages-1021838033-Automate-Validation-Checklist-2026-06-03-14_57_14" src="https://github.com/user-attachments/assets/c2939594-2ed3-4aec-bafe-c680ef721063" />
<img width="1358" height="2355" alt="screencapture-britishairways-datadoghq-eu-dashboard-npg-44j-ka4-2026-06-03-14_58_41" src="https://github.com/user-attachments/assets/b69ca958-fd71-418d-a824-64cd1152e865" />
<img width="1358" height="10628" alt="screencapture-britishairways-datadoghq-eu-notebook-311211-enterprise-deployment-validation-architecture-2026-06-03-14_57_54" src="https://github.com/user-attachments/assets/c4018b22-f9ea-4039-8cc1-bf56a26e1a64" />
<img width="1166" height="7128" alt="screencapture-britishairways-datadoghq-eu-notebook-311166-odie-api-gateway-dashboard-documentation-guide-2026-06-03-15_06_12" src="https://github.com/user-attachments/assets/e3b62312-5def-49e4-a4b3-18b6528f4928" />

```
Skip to:
Top Bar
Sidebar
Main Content


Search Confluence, Jira, Microsoft SharePoint and other apps

Create

9+



Back to top
Retrospectives (2)
Search by title
Updated May 21
Akashdip Mahapatra
Edit

Share


Automate Validation Checklist



By Akashdip Mahapatra

8 min

13

Add a reaction
Add Workflow
Add Workflow
Centralized Deployment Validation (CDV) Framework

Using - Python, Boto3, GitHub Actions, AWS CloudWatch, AWS Lambda, AWS SNS, AWS ECR, AWS API Gateway, Pandas, OpenPyXL, CI/CD Automation

diagram-export-26-04-2026-17_45_31.png

Expand to see the Flow Diagram
image-20260418-073513.png
Screenshot 2026-04-27 101114.png
pic: sample TEST repost after our Migration - PR 03 (18/04/2026) 

Video explanation : link : How to Use ?
The Work Flow link : Click and Use

 

1. Purpose
This document defines the end-to-end process for executing, validating, and auditing post-deployment health checks across our containerized and serverless data platform workloads.

Historically, post-deployment validation required manual querying of AWS CloudWatch, which was time-intensive and prone to human error. The Centralized Deployment Validation (CDV) framework is a zero-touch, Python-driven automation tool designed to interact directly with AWS APIs (Boto3).

The objective is to:

Reduce post-deployment manual validation time from 30+ minutes to under 2 minutes.

Automatically detect and extract critical application failures (Exceptions, DLQ events, Timeouts).

Generate an immutable, auditable Excel report for Change Request (CR) compliance.

Maintain a secure, zero-permissions footprint via GitHub Actions OIDC (OpenID Connect).

2. Source of Truth
Unlike previous iterations that relied on GitHub repository code structures, the CDV framework relies exclusively on live AWS Infrastructure.

Lambda Auto-Discovery: AWS Boto3 list_functions API.

Deployment Timestamps: Extracted dynamically via Lambda Configuration and ECR imagePushedAt metadata.

Telemetry Data: AWS CloudWatch Metrics and CloudWatch Logs Insights.

3. User Guide: Execution Workflow
This tool is designed to be utilized by any team member (DevOps, QA, or Data Engineers) immediately following a successful deployment. No coding or AWS console access is required to run this validation.

Step-by-step Execution:

Navigate to the ops-odie-datapipeline-gse-event-details repository in GitHub. Why I use a Data-Product repo ? - learn-more

Open the Actions tab. 

image-20260411-070452.png
Select Centralized Deployment Validation from the left-hand menu.

Click the Run workflow dropdown button located on the right side of the interface. 

Configure the Parameters:

Target Data Product: Enter the exact string name of the data product (e.g., gse-event-details, gse-tracking, flight-status).

N.B. For established products with known AWS abbreviations (like mnt-revisions or aircraft-restrictions-and-curfews), simply enter the full GitHub name here. The Python engine will automatically map it to the correct AWS name (ews-revs / ac-restrictions-curfews) behind the scenes. However, if this auto-mapping fails (indicated by a [FAIL - No Lambdas found...] message in the output Excel), please re-run the workflow and manually input the correct AWS abbreviation into the AWS Name Override (Optional) field.

Target Environment: Select the target environment from the dropdown menu (dev, uat, or prod).

Associated JIRA Ticket (Optional): Input the release or bug ticket number for audit traceability.

AWS Name Override (Optional): Use this only if you are deploying a brand new Data Product where the GitHub name differs from the AWS abbreviation, AND it hasn't been added to the config.json alias list yet. If left blank, the tool seamlessly defaults to the Target Data Product name.

image-20260416-080225.png
new update (PR 02) - add new button 16/04/2026 - read the NB
Click Run workflow to initiate the sequence.

Retrieve the Artifact: Once the pipeline completes (approx. 2 minutes), scroll to the bottom of the workflow run summary page. Download the generated .zip artifact. 


image-20260411-070628.png
4. What the Automation Performs
Once triggered, the workflow executes a strict, AWS-native validation sequence:

Step 1: Dynamic Discovery & Version Consolidation: Scans the target AWS account for all Lambdas, SNS Topics, and API Gateways matching the specified Data Product and Environment. Note: If multiple versions of the same resource exist due to migrations (e.g., a -v1 and a -v2), the engine uses regex filtering to automatically drop the legacy infrastructure and only validate the highest active version.


image-20260416-081521.png
⚙ Problem img - showing V1, V2 … all ECR repos

image-20260416-081546.png
⚙ Fix - all filter to show the latest version only. 16/04/2026 - PR 02

Step 2: Precision Timeframe Isolation: Identifies the exact millisecond the new code was deployed to AWS to prevent old errors from failing a new deployment.

Step 3: Traffic & Error Analysis: Compares pre-deployment traffic to post-deployment traffic to ensure data is flowing, while enforcing a zero-tolerance policy for AWS execution errors and throttles.

Step 4: Log Extraction: Performs server-side filtering on CloudWatch logs to hunt for specific exception patterns (ERROR, DLQ, Timeout).

Step 5: Proactive Endpoint Checks: Verifies SNS topic subscription health (flagging PendingConfirmation anomalies) and pings API Gateways for HTTP 200 responses.

5. Output Artifacts & Interpretation
The execution generates an immutable .zip artifact retained for 30 days. It contains two primary data classifications:

5.1 The Deployment Checklist (Excel)

A summarized matrix detailing the operational health of the deployment window. It provides a simple [PASS] or [FAIL] for every discovered resource, alongside the exact method used to test it.

5.2 Critical Error Logs (CSV)

If the Python engine detects application failures within the CloudWatch streams, it bypasses the standard UI and exports the raw log dictionaries directly to discrete CSV files. Note: The script automatically filters out benign infrastructure noise (e.g., standard Kafka broker reconnection attempts) to ensure only genuine data-loss or application failures are reported.

image-20260411-070653.png
image-20260411-070711.png
5.3 Graceful Zero-Discovery (The "Not Found" Fail-Safe) If a user intentionally inputs a random, non-existent data product name (e.g., "abc"), or if there is a severe naming mismatch between GitHub and AWS, the script will not fail silently or generate a false "PASS" report.

The workflow will safely upload the artifact and then fail with a Red Cross.

The Excel report will explicitly inject a failure row stating: [FAIL - No Lambdas found. The data product name may differ from the AWS resource name. Please check and provide the correct AWS name using the aws_name_override input, or ask the team to add an alias in config.json.]

This ensures absolute audit transparency and guides the engineer on how to fix the mismatch.

image-20260416-082231.png
New update - PR 02
6. Security & Architectural Controls (Value Addition)
To ensure enterprise-grade security and prevent unauthorized access, the CDV framework implements several strict architectural controls.

6.1 Input Sanitization (Shell Injection Prevention)

Because the Data Product name is a free-text input, it is vulnerable to malicious shell injection. The workflow utilizes a strict Regex validation gate before any processing begins.



# Validates the free-text inputs to prevent Shell Injection attacks
DATA_PRODUCT="${{ inputs.data_product }}"
AWS_OVERRIDE="${{ inputs.aws_name_override }}"
if [[ ! "$DATA_PRODUCT" =~ ^[a-zA-Z0-9_-]+$ ]]; then
  echo "[ERROR] Invalid data_product input. Only alphanumeric, dashes, and underscores allowed."
  exit 1
fi
if [[ -n "$AWS_OVERRIDE" && ! "$AWS_OVERRIDE" =~ ^[a-zA-Z0-9_-]+$ ]]; then
  echo "[ERROR] Invalid aws_name_override input."
  exit 1
fi
new update - PR 02 

6.2 Precision ECR Extraction

To calculate the precise deployment timeframe without requiring cross-repository GitHub permissions, the Python engine reverse-engineers the ECR container registry directly from the live Lambda's ImageUri configuration.



# Extract the ECR repo directly from the Lambda's ImageUri!
code_info = lambda_info.get('Code', {})
image_uri = code_info.get('ImageUri')
if image_uri:
    # Slices off the AWS URL prefix and the ':latest' tag to find the exact repo
    uri_without_host = image_uri.split('/', 1)[-1]
    ecr_repo = uri_without_host.rsplit(':', 1)[0]
 

6.3 OIDC Authentication

The workflow utilizes OpenID Connect (OIDC) to authenticate with AWS. This eliminates the need for long-lived IAM user credentials or GitHub Personal Access Tokens (PATs), mapping directly to the terraform-odin-datapipeline-deploy-role.

6.4 Configuration Alias Mapping ("Tribal Knowledge" Prevention) To prevent engineers from needing to memorize AWS naming quirks, the framework utilizes a config.json dictionary mapping system. Daily-use data products with known naming deviations (e.g., aircraft-restrictions-and-curfews mapped to ac-restrictions-curfews) are hardcoded as aliases. When a user inputs the standard GitHub name, the Python engine invisibly translates it to the AWS alias, completely removing the need for manual UI overrides for established products.

image-20260416-082731.png
New update - PR 02

7. Post-Deployment Validation Checklist Mapping
This framework directly automates the mandatory Post-PROD validation steps required by the release management team.

Validation Step

Status

Automated via CDV Framework

No Critical/High Vulnerabilities

Y/N

Manual (via Amazon Inspector Process)

All Lambda Invocations Verified

Y/N

Yes (Row 1 of Excel Output)

All Lambda ERRORs Verified

Y/N

Yes (Row 2 of Excel Output)

CloudWatch Logs Scanned for DLQ/Errors

Y/N

Yes (Row 3 of Excel Output & CSV Exports)

Endpoint Services (API/SNS) Health

Y/N

Yes (Row 4 & 5 of Excel Output)

DataDog Alerts in CR Window

Y/N

Pending Integration

image-20260411-071439.png
8. Expected Outcome & Future Enhancements
Expected Outcome:

By implementing this centralized automation, we achieve a predictable, auditable, and highly secure post-release validation cycle. It drastically reduces engineering toil and catches edge-case application exceptions that manual queries often miss.

Test 01
Automated check

image-20260411-071515.png
image-20260411-071303.png
Manual check

image-20260411-071626.png
image-20260411-071640.png
Test 02
Automated check

image-20260413-053316.png
image-20260413-053344.png
image-20260413-053354.png
Manual check

image-20260413-053415.png
🔎 logs : for Test 01 
1 Manual loges from Cloud Watch - logs-insights-results from aws CloudWatch.csv 

2 Automated Loges (auto Removed the "infrastructure Noise" (82 Logs)) - lmb-odin-uat-euwe1-flight-status-raw-sequencer-v2_ERRORS_20260410_191818.csv lmb-odin-uat-euwe1-flight-status-processed-to-curated-v2_ERRORS_20260410_191803.csv 

image-20260411-074613.png
Future Enhancements:

Integration with DataDog APIs to automatically pull triggered monitoring alerts directly into the final Excel artifact.

Expanding endpoint validation to include SQS Queue Depth and DynamoDB read/write capacity metrics.

Integration with ServiceNow/JIRA to automatically generate incident tickets if the CDV framework detects [FAIL] states in the Production environment.

 

ID

Improvement / Issue

Description

Owner

Priority

Status

Target Completion

Notes / Blockers

1

Improvement

Automate Validation Checklist

Akashdip Mahapatra

High

Dev Complete

Apr 10, 2026

Improvement stage

links:
RunBook - britishairways.datadoghq.eu/notebook/311211/enterprise-deployment-validation-architecture

PR 01 - Pull Request

Jira - Design & Architecture

Jira - Implementation & Testing

Jira - After deploy - Bug fix

Jira - implement DataDog

PR 02 - Pull Request 

PR 03 - Pull Request

Workflow

Documentation Repo 

Deployment Validation Pipeline — Eraser

Video explanation - Deployment Validation.mp4 



 

Related content

After Deployment Validation Checklist Templates
Vikas Panwar
Automation Testing
Sameer Penumutchu
Automation Testing
Shrinath Lakshmipathy
Test Automation - Requirement
Dhinesh Kumar Kamalanathan
Daily Checks
Senthil Nayagan
DevSecOps-Pipeline-Flow-Backend
RAJESH NAYAK




Add a comment

Add labels

Add a reaction
© 2025 British Airways. ALL RIGHTS RESERVED.



---

Related Resources:

Dashboard: ODIE - API Gateway Health Overview

Dashboard Documentation: Dashboard Guide - Notebook

Repository: ops-odie-datapipeline-gse-event-details

Tech Stack: Python, Boto3, GitHub Actions, AWS CloudWatch, AWS Lambda, AWS SNS, AWS ECR, AWS API Gateway, Pandas, OpenPyXL, CI/CD Automation

Table of Contents

System Overview

Environment Triggers & CI/CD Workflows

Security Model

The DevOps Engine: AWS-Native Dynamic Discovery

The Hybrid Configuration Pattern

The Python Validation Engine (Module Architecture)

6.0 Module Structure 

6.1 Alias Resolution & AWS Name Override

6.2 Precision Timeframe Isolation

6.3 Version Filtering (Highest Version Deduplication)

6.4 Zero-Discovery Guard

Row 1: Lambda Invocations (Hybrid Validation)

Row 2: Lambda Errors & Throttles (Zero-Tolerance)

Row 3: CloudWatch Log Analysis (Server-Side Filtering)

Row 4 & 5: Proactive Endpoint Health & Logs

Row 6: Production Version Mapping (Prod Only)

Bugs Found & Fixed

Changelog

API Gateway Validation (CloudWatch + Datadog Integration)

Known Limitations

Future Enhancements

1. System Overview

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/q4i-78n-6pn

Automated post-deployment validation system for enterprise data products. This workflow validates deployments by analyzing CloudWatch metrics, log streams, and infrastructure health, generating comprehensive Excel reports with intelligent status detection and evidence collection.

The system replaces manual validation processes that previously took 20-30 minutes with an automated 2-3 minute workflow, while providing detailed evidence for audit trails and upstream team accountability.

Note: This feature is currently hosted in ops-odie-datapipeline-gse-event-details for testing, as this repo already has the required OIDC roles, GitHub Environments, and AWS access configured. Once validated, it will be migrated to a dedicated ops-odie-actions tooling repository.

Repository Layout:

ops-odie-datapipeline-gse-event-details/
├── .github/workflows/
│   └── deployment-validation.yaml
└── tests/deployment_validation/
    ├── deployment_validator.py
    ├── api_sns_validator.py
    ├── current_status_checker.py
    ├── report_builder.py
    ├── report_formatter.py
    ├── config.json
    ├── config-example.json
    ├── .gitignore
    └── requirements.txt

The workflow checks out only this repo. All resource discovery is performed entirely via AWS Boto3 APIs using the user-provided inputs (data_product, environment, and optional aws_name_override).

2. Environment Triggers & CI/CD Workflows

The workflow uses workflow_dispatch, allowing engineers to run validation on-demand from the GitHub Actions UI.

Input

Type

Description

data_product

string (required)

Target data product name (e.g., flight-status)

environment

choice (required)

dev, uat, or prod

aws_name_override

string (optional)

Override when AWS name differs from repo name

jira_ticket

string (optional)

Associated JIRA ticket for audit trail

documentation_link

string (optional)

Read-only reference link to Confluence

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/4dw-yep-pjv

Fallback Behaviour: Environment-level fallback variables use || so the workflow executes even if inputs are empty:

env:
  FALLBACK_DATA_PRODUCT: 'flight-status'
  FALLBACK_ENVIRONMENT: 'dev'
  FALLBACK_REGION: 'eu-west-1'

Approval Gate (Currently Disabled): A validation-approval job gated behind environment: approval-nlv is defined but commented out.

3. Security Model

Input Sanitization (Shell Injection Prevention):

- name: Sanitize User Inputs
  run: |
    DATA_PRODUCT="${{ inputs.data_product || env.FALLBACK_DATA_PRODUCT }}"
    if [[ ! "$DATA_PRODUCT" =~ ^[a-zA-Z0-9_-]+$ ]]; then
      echo "[ERROR] Invalid data_product input."
      exit 1
    fi

Key Security Controls:

No cross-repository authentication required (all via AWS Boto3 + OIDC)

persist-credentials: false on checkout

Hard-fail on AWS credential errors (no continue-on-error: true)

Quoted Python arguments to prevent shell injection

4. The DevOps Engine: AWS-Native Dynamic Discovery

Lambda functions are discovered directly from AWS using the list_functions API. ECR repository names are resolved from each Lambda's Code.ImageUri field.

Lambda Discovery:

def _discover_lambdas_from_aws(self) -> list:
    search_string = f"lmb-odin-{self.environment}-euwe1-{self.search_product}"
    paginator = self.lambda_client.get_paginator('list_functions')
    for page in paginator.paginate():
        for fn in page['Functions']:
            if search_string in fn['FunctionName']:
                discovered.append({"name": fn['FunctionName'], "log_group": f"/aws/lambda/{fn['FunctionName']}"})

ECR Resolution via Lambda Image URI:

def _get_ecr_repo_from_lambda(self, func_name: str) -> str | None:
    info = self.lambda_client.get_function(FunctionName=func_name)
    image_uri = info.get('Code', {}).get('ImageUri', '')
    if image_uri:
        repo_with_tag = image_uri.split('/')[-1]
        return repo_with_tag.split(':')[0]
    return None

5. The Hybrid Configuration Pattern

Auto-discovery + manual config merged. Developers define anomalies in config.json, combined with discovered resources.

Scenario

Product Example

Purpose

1. Basic Alias

mnt-revisions -> ews-revs

Translates GitHub name to AWS abbreviation

2. Alias + Custom ECR

aircraft-restrictions-and-curfews

Forces specific ECR repository

3. Rogue Lambda

gse-tracking

Manually adds poorly-named Lambda

4. Isolated Endpoints

flight-status

Explicitly tests custom SNS topics and APIs

5. Fully Explicit

ops-odie-services

All Lambda names differ from repo name

All string values support {env} placeholder, resolved at runtime.

Merge Logic: Uses set() for O(1) deduplication on (name, log_group) tuples.

6. The Python Validation Engine (Module Architecture)

6.0 Module Structure

Module

Lines

Responsibility

deployment_validator.py

~574

Core orchestration: AWS discovery, Lambda checks, ECR validation

api_sns_validator.py

~201

API Gateway + SNS validation: CloudWatch health, subscriptions

current_status_checker.py

~80

Re-checks FAILED services (last 15 min) for self-resolved issues

report_builder.py

~186

Constructs checklist DataFrame from results dict

report_formatter.py

~116

Excel styling: color-coded rows, borders, summary

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/zsu-64k-2e7

Excel Row Color Legend:

Color

Status

Meaning

Green #C6EFCE  🟩

[PASS] Yes

All services PASS at deployment time

Yellow #FFEB9C 🟨

[RESOLVED] No (was FAIL)

FAIL detected but NOT occurring now (last 15 min clean)

Red #FFC7CE 🟥

[FAIL] No

FAIL persists from deployment time until now

Light Blue #ADD8E6 🟦

N/A

Check not applicable

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/8zs-yfj-d7g



6.1 Alias Resolution

Three-tier priority: CLI --aws-name-override > config aws_name_alias > raw data_product name.

6.2 Precision Timeframe Isolation

Primary: ECR image push time. Fallback: Lambda LastModified. 51-day cap for CloudWatch API limits. Timezone display: Europe/London (BST/GMT auto-switch).

6.3 Version Filtering

Regex -v(\d+)$ matches suffixes. Only highest version per base name retained.

6.4 Zero-Discovery Guard

If no resources found, logs critical error with hints and injects synthetic FAIL into report.

Row 1: Lambda Invocations (Hybrid Validation)

CI/CD passes if traffic > 0. Calculates mirrored temporal window for context: [PASS] (Prev: 129610 -> Curr: 109197)

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/kab-hqm-cbv



Row 2: Lambda Errors & Throttles (Zero-Tolerance)

Single error or throttle = FAIL. Displays historical context: [FAIL] (Prev: PASS -> Curr: FAIL) or [PASS] (Prev: FAIL -> Curr: PASS)

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/utp-ggz-6b7



Row 3: CloudWatch Log Analysis (Server-Side Filtering)

AWS servers execute the search. Error patterns from config.json. Critical errors exported to CSV for JIRA evidence.

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/g86-ktr-r4m



Row 4: Endpoint Health & SNS Broadcast Validation

Validates: 1) SNS topic exists + active subscriptions, 2) Data was broadcasted (root log traffic), 3) Delivery failures (/Failure log group).

Status

Meaning

[Broadcast: YES][Delivery Failures: 0]

Data sent, all subscribers received

[Broadcast: YES][Delivery Failures: 234]

Data sent, consumer-side failures

[Broadcast: NO TRAFFIC]

No delivery logs — investigate

Row 5: Prod Version Mapping

5-step closed-loop: resolve live alias -> extract ECR tag -> query latest ECR tag -> compare -> PASS/FAIL with direct repo link for remediation.

7. Bugs Found & Fixed

7.1 ECR "Random Image" Bug (maxResults=1)

Severity: Critical — silently anchors validation to wrong deployment date.

Root Cause: describe_images(maxResults=1) returns whichever image appears first in AWS's internal SHA256 index — NOT the most recently pushed.

Why invisible for months: The bug is deterministic but fragile. gse-event-details (prod) worked by luck (latest digest sorted first). aircraft-towing (dev/uat) consistently returned April 10/13 instead of April 22+.

Fix: Paginate ALL tagged images, explicitly sort by imagePushedAt descending:

imgs = []
for page in self.ecr_client.get_paginator('describe_images').paginate(
    repositoryName=ecr_repo, filter={"tagStatus": "TAGGED"}):
    imgs.extend(page.get("imageDetails", []))
latest_push = sorted(imgs, key=lambda x: x["imagePushedAt"], reverse=True)[0]["imagePushedAt"]

Why not query by Lambda's current tag instead? Because _get_ecr_push_time detects when the latest code was deployed to ECR, not when the currently running code was deployed. A new image pushed but not yet applied to Lambda would be missed.

8. Changelog (Key Changes)

ECR Multi-Tag False Failure Fix: _get_latest_ecr_tag now returns list (all tags on latest image). _evaluate_ecr_sync changed from == to in.

Environment Placeholder Support ({env}): _resolve_env_placeholders() replaces {env} in all explicit resource dicts. One config entry works across dev/uat/prod.

Production ECR Version Sync Check: 5-step closed-loop validation comparing deployed ECR tag against latest pushed tag. Mismatches produce FAIL with direct repo link.

Version Consolidation Filter: _filter_highest_versions drops older versions (e.g., keeps -v2, drops -v1).

Product Mappings Restructure: Flat product_config -> product_mappings dict keyed by product name. Three-tier alias resolution.

Deployment Time Priority Fix: ECR imagePushedAt checked first (actual code), Lambda LastModified as fallback (Terraform).

Zero-Discovery Guard: Injects synthetic FAIL with actionable hints when no resources found.

Dynamic Excel Detail Headers: [FAIL]Reason - now changes to [WARNING/INFO] Details - when row actually passed.

Module Refactor: Monolithic 755-line file split into 5 modules for CodeScene compliance (<600 lines).

Historical Context for Errors: Error row now shows [FAIL] (Prev: PASS -> Curr: FAIL) format.

Current Status Recheck (Yellow Row Logic): Re-checks FAILED services only. Lambda: last 15 min. API: last 5 min. Zero errors in recent window = yellow [RESOLVED].

9. API Gateway Validation (CloudWatch + Datadog Integration)

9.1 Why We Updated

Original _ping_api HTTP GET failed because ODIE APIs are Private REST APIs (VPC-only, auth required, query params needed). Solution: validate indirectly using CloudWatch metrics.

9.2 How It Works

Queries AWS/ApiGateway CloudWatch namespace using ApiName dimension. Time window: deployment_time to current_time.

9.3 Environment-Aware Traffic

Environment

Zero Traffic Result

Color

prod

FAIL - "CRITICAL: No consumer traffic"

Red

uat

WARNING - "UAT has reduced consumer traffic"

Yellow

dev/staging

WARNING - "No consumer traffic expected"

Yellow

9.4 Three Sub-Checks

Sub-Check

Metric

Pass Condition

Consumer Traffic

Count

> 0 requests

Server Errors

5XXError

<= 1

Client Errors

4XXError

<= 1

9.5 Excel Output Example

1) lmg-odin-prod-euwe1-persistence-layer-api [PASS]
   [PASS] Consumer Traffic: 6338 requests since deployment
   [PASS] Server Errors (5xx): 0 (threshold: <=1)
   [PASS] Client Errors (4xx): 0 (threshold: <=1)

https://britishairways.datadoghq.eu/s/e4f8bb8c-1d03-11ef-9b95-da7ad0900005/e6c-6a9-fpg



9.6 Datadog Dashboard

Dashboard: ODIE - API Gateway Health Overview (All Environments)

The live Datadog dashboard shows Consumer Traffic, 5xx/4xx Errors, and Latency across all environments with template variables $environment and $apiname.

Documentation: Dashboard Guide - Notebook

9.7 Datadog Monitor Reference

Monitor

ID

Priority

Metric

Threshold

prod-mon-apigw-1-gateway-5xx-error-count-v2

107107644

P2

aws.apigateway.5xxerror

avg > 1 in 5m

prod-mon-apigw-2-gateway-4xx-error-count-v2

107107653

P2

aws.apigateway.4xxerror

avg > 1 in 5m

uat-mon-apigw-1-gateway-5xx-error-count-v2

106529892

-

aws.apigateway.5xxerror

avg > 1 in 5m

uat-mon-apigw-2-gateway-4xx-error-count-v2

106529872

-

aws.apigateway.4xxerror

avg > 1 in 5m

dev-mon-apigw-1-gateway-5xx-error-count-v2

106492331

-

aws.apigateway.5xxerror

avg > 1 in 5m

dev-mon-apigw-2-gateway-4xx-error-count-v2

106492324

-

aws.apigateway.4xxerror

avg > 1 in 5m

staging-mon-apigw-1-gateway-5xx-error-count-v2

107055758

-

aws.apigateway.5xxerror

avg > 1 in 5m

staging-mon-apigw-2-gateway-4xx-error-count-v2

107055776

-

aws.apigateway.4xxerror

avg > 1 in 5m

9.8 Which APIs Are Validated Per Product

--data-product

APIs Validated

Discovery Method

ops-odie-services

lmg-odin-{env}-euwe1-api + persistence-layer-api

Explicit config

gse-event-details

None (only SNS topics)

Auto-discovery

flight-status

None (only SNS topics)

Auto-discovery

10. Known Limitations

Lambda naming convention dependency — Discovery relies on lmb-odin-{env}-euwe1-{search_product}. Deviations need config.json.

Container image Lambdas only — ECR time detection only for container images. Zip-based fall back to LastModified.

51-day cap — CloudWatch API retention limits.

5-minute granularity — CloudWatch metrics use 5-min periods for windows under 4 days.

Artifact retention — GitHub Actions artifacts deleted after 30 days.

Endpoint services non-blocking — SNS/API warnings never fail CI/CD pipeline.

Private API Gateways — Cannot HTTP-ping from GitHub runners. Uses CloudWatch metrics instead.

Code complexity constraints — Functions limited to 4 args and cyclomatic complexity <=9 for CodeScene.

Dev/QA API traffic — Zero consumer traffic in dev/QA is expected. API Count always returns WARNING (yellow).

11. Future Enhancements

DataDog API integration DONE — Dashboard link embedded in Excel report (Section 9)

CloudWatch Metrics for SNS publish success/failure rates

API Gateway 4xx/5xx thresholds DONE — CloudWatch-based validation matching Datadog P2 thresholds

Automated remediation suggestions

Slack/Teams notifications for failures

Historical trend analysis

Custom metric thresholds per Lambda

Support for Step Functions validation

DynamoDB table metrics validation

SQS queue depth monitoring

Custom CloudWatch dashboard DONE — Datadog dashboard npg-44j-ka4

Links & References

Resource

Link

Workflow

Run Validation

Datadog Dashboard

ODIE - API Gateway Health Overview

Dashboard Docs

Dashboard Guide Notebook

Technical Runbook (detailed)

Architecture Notebook

Confluence pg (with Video Guide)

https://britishairways.atlassian.net/wiki/x/0QLoP 

Runbook Author: akashdip.mahapatra@ba.com | Last Updated: 2026-05-21 | Dashboard | Dashboard Docs

---
Skip to notebook

Search Datadog
ctrl
k

Ask Bits


Bits AI
Dashboards
Monitoring
Developer Portal
Incident Response
Automation
Infrastructure
Cloud Cost
APM
Digital Experience
Software Delivery
Security
Data Observability
AI Observability
Errors
Metrics
Logs

Integrations

akashdip.mahapatra@ba.com
akashdip.mahapatra@ba.com
British Airways

Support

Past 1 Week






Share







Favorite

Documentation

ODIE API Gateway Dashboard - Documentation Guide



Akashdip Mahapatra
Updated 13 days ago

Unrestricted access
This is a comprehensive guide to the ODIE - API Gateway Health Overview (All Environments) dashboard.




Dashboard Purpose
This dashboard was created to:

Replace manual curl commands — no more manually hitting endpoints to check if consumers are calling your APIs.

Centralise all API Gateway health into a single view across all environments (prod, uat, dev, staging).

Quickly distinguish between problems on your side (5xx) vs consumer side (4xx).

Validate API health after deployments — linked from the deployment validation Excel report.

Template Variables (Filters at the Top)
Variable

Purpose

Values

$environment

Filter the entire dashboard to a specific environment

prod, uat, dev, staging

$apiname

Filter to a specific API/data product

* (all) or a specific API name

How to use: Select a value from the dropdown at the top of the dashboard. All widgets update automatically.

Section 1: Consumer Traffic — Request Count by Environment
What It Shows
The total number of API requests made by consumers (external callers) to your ODIE APIs in the last 24 hours, broken down by environment.

The Number Tiles
Tile

Metric

Meaning

PROD - Total Requests (24h)

sum:aws.apigateway.count{project:odie,environment:prod}

Total requests hitting your Production APIs in 24h. Example: 121k ops = 121,000 API calls.

UAT - Total Requests (24h)

sum:aws.apigateway.count{project:odie,environment:uat}

Total requests hitting your UAT APIs in 24h. Example: 79k ops = 79,000 API calls.

DEV - Total Requests (24h)

sum:aws.apigateway.count{project:odie,environment:dev}

Total requests hitting your Dev APIs. Shows (No data) if no traffic.

How to Read It
✅ Numbers visible = Consumers are actively calling your APIs. Everything is working.

⚠️ "No data" = No traffic in that environment. Could be normal (e.g., dev/staging may not always have traffic) or a problem (if prod shows no data, something is seriously wrong).

📉 Sudden drop in numbers = Consumers may have stopped calling, or something is blocking traffic (network, auth, deployment issue).

Possible Scenarios
Scenario

What It Means

Action

PROD = 121k, UAT = 79k

Normal — consumers are active in both environments

No action needed

PROD = 0 or No data

🚨 Critical — no consumer traffic in production

Check API Gateway, Lambda, network, DNS immediately

PROD drops from 121k to 20k

Significant traffic drop — some consumers may have stopped

Investigate if a deployment broke something

UAT = No data

May be normal if no testing is happening

Check if UAT testing is expected

The Timeseries Chart: "Request Count - All Environments by API"
Query: sum:aws.apigateway.count{project:odie,$environment,$apiname} by {environment,apiname}.as_count()

What the Bars Mean
Each colour represents a different API name (data product) within an environment.

The height of the bar shows how many requests that specific API received at that point in time.

Bars are stacked — the total height shows the combined traffic across all APIs.

How to Identify the Exact Data Product (API Name)
Hover over any bar — a tooltip appears showing the exact apiname and environment with its request count.

Look at the legend at the bottom of the chart — each colour is labelled with the format: apiname:<name>, environment:<env>.

Example from the current view:

🟪 Purple bars = apiname:lmg-odin-uat-euwe1-api (the main API gateway)

🟦 Blue bars = apiname:lmg-odin-uat-euwe1-persistence-lay... (persistence layer API)

What the Patterns Tell You
Pattern

Meaning

Regular daily peaks and valleys

Normal — traffic follows business hours/usage patterns

Sudden spike in one colour

One specific API is getting unusual traffic

A colour disappears

That specific API stopped receiving traffic

All bars drop to zero

All APIs are down — major outage

New colour appears

A new API was deployed and receiving traffic

Section 2: 5xx Server Errors — YOUR Side Problems 🔴
Group Title: 5xx Server Errors (Threshold: avg > 1 in 5m)

What 5xx Errors Mean
5xx errors are SERVER-SIDE errors — these are YOUR team's problem. The API Gateway received the consumer's request, but something went wrong on your backend (Lambda crash, timeout, internal error, bad gateway, etc.).

Common 5xx Error Codes
Code

Name

Typical Cause

500

Internal Server Error

Lambda function crashed, unhandled exception

502

Bad Gateway

Lambda returned invalid response, integration error

503

Service Unavailable

Lambda throttled, service overloaded

504

Gateway Timeout

Lambda took too long to respond (>29s for API GW)

The Number Tiles
Tile

Metric

Current Value

Meaning

PROD 5xx Errors (5m avg)

avg:aws.apigateway.5xxerror{environment:prod}

0 ops ✅

No server errors in Production — healthy

UAT 5xx Errors (5m avg)

avg:aws.apigateway.5xxerror{environment:uat}

0.13 ops ⚠️

Very low-level errors in UAT — worth monitoring

DEV / STAGING

Same metric, different env

(No data)

No traffic or no errors in these environments

The Timeseries Chart: "5xx Errors Over Time by Environment and API"
Query: avg:aws.apigateway.5xxerror{project:odie,$environment,$apiname} by {environment,apiname}

The Red Line (Alert Threshold)
The red dashed horizontal line labelled "Alert Threshold" at value 1 means:

If the 5-minute average of 5xx errors crosses above 1, the linked monitor will trigger an alert.

Below the line = ✅ healthy.

Above the line = 🚨 alert fires.

How to Find Which Data Product Has 5xx Errors
Hover over the chart — the tooltip shows apiname and environment.

Use the $apiname filter at the top — select a specific API to isolate its errors.

Check the "Top APIs by 5xx Errors" toplist at the bottom of the dashboard — it ranks APIs by error count.

Possible Scenarios
Scenario

Severity

Likely Cause

Action

5xx = 0 across all envs

✅ Healthy

No server errors

No action

5xx spikes in one API only

⚠️ Medium

That specific Lambda/backend is failing

Check Lambda logs for that API

5xx spikes across all APIs

🚨 Critical

Shared infrastructure issue (VPC, IAM, DB)

Check shared dependencies

UAT 5xx = 0.13

ℹ️ Low

Occasional transient errors during testing

Monitor but likely acceptable

Section 3: 4xx Client Errors — CONSUMER Side Problems 🟡
Group Title: 4xx Client Errors (Threshold: avg > 1 in 5m)

What 4xx Errors Mean
4xx errors are CLIENT-SIDE errors — these are the CONSUMER's problem. The consumer sent a request that was incorrect, unauthorized, or malformed.

Common 4xx Error Codes
Code

Name

Typical Cause

400

Bad Request

Consumer sent invalid JSON, wrong parameters

401

Unauthorized

Missing or invalid API key/token

403

Forbidden

Consumer doesn't have permission for this resource

404

Not Found

Consumer called a wrong URL/endpoint

429

Too Many Requests

Consumer exceeded rate limits

The Number Tiles
Tile

Current Value

Meaning

PROD 4xx Errors (5m avg)

0 ops ✅

No client errors — consumers are calling correctly

UAT 4xx Errors (5m avg)

0 ops ✅

No client errors in UAT

DEV / STAGING

(No data)

No traffic or no errors

The Timeseries Chart: "4xx Errors Over Time by Environment and API"
Same structure as the 5xx chart. The red Alert Threshold line at 1 means the same — if 4xx avg crosses above 1 in 5 minutes, the monitor alerts.

How to Find Which Data Product Has 4xx Errors
Same approach as 5xx — hover the chart, use the $apiname filter, or check the toplists.

When to Care About 4xx Errors
Scenario

Meaning

Action

4xx = 0

✅ Consumers are calling correctly

No action

Sudden spike in 4xx on one API

Consumer may have changed their integration

Communicate with the consumer team

4xx spike after YOUR deployment

⚠️ You may have changed the API contract

Verify your API didn't break backward compatibility

Steady low-level 4xx

Some consumers may be misconfigured

Share correct API documentation

Section 4: API Latency (Response Time) ⏱️
What It Shows
The average time your APIs take to respond to consumer requests.

The Number Tiles
Tile

Metric

Current Value

Meaning

PROD Avg Latency (ms)

avg:aws.apigateway.latency{environment:prod}

6.6s (6,600ms)

Average response time in Production

UAT Avg Latency (ms)

avg:aws.apigateway.latency{environment:uat}

10.2s (10,200ms)

Average response time in UAT

STAGING / DEV

Same metric

(No data)

No traffic

How to Read Latency
Latency

Assessment

Notes

< 1 second

✅ Excellent

Fast response

1–5 seconds

⚠️ Acceptable

Depends on the API's purpose

5–10 seconds

🟡 Slow

May need optimization

> 10 seconds

🔴 Very Slow

Investigate Lambda cold starts, DB queries, downstream calls

> 29 seconds

🚨 Critical

API Gateway has a 29s timeout — requests will start failing with 504

The Timeseries Chart: "API Latency Over Time by Environment and API"
Query: avg:aws.apigateway.latency{project:odie,$environment,$apiname} by {environment,apiname}

The Pink/Magenta Spike
In the current view, there is a pink/magenta spike visible around Wed 20 – Thu 21. This indicates a sudden increase in latency for a specific API. To identify which API:

Hover over the spike — tooltip shows the exact apiname and latency value.

Use $apiname filter to isolate individual APIs.

What Causes Latency Spikes
Cause

Description

Lambda cold starts

First invocation after idle period takes longer

Database slowness

Backend DB queries taking longer than usual

Downstream service issues

Your API calls another service that's slow

High traffic load

Too many concurrent requests saturating resources

Large payload processing

Consumer sending/receiving large data sets

Section 5: Environment Comparison — Traffic Ranking 📊
Top APIs by Request Count (All Envs)
A toplist ranking all APIs by total request count. This tells you which data products are the most used.

Current view (UAT filter active):

Rank

API Name

Requests

Meaning

1

lmg-odin-uat-euwe1-api

69.2k

Main API gateway — highest traffic

2

lmg-odin-uat-eu...nce-layer-api

9.6k

Persistence layer API — moderate traffic

Top APIs by 5xx Errors (All Envs)
A toplist ranking APIs by total 5xx (server) errors. This is your quick answer to "which data product has the most problems on our side?"

Current view:

Rank

API Name

5xx Errors

Meaning

1

lmg-odin-uat-euw...ence-layer-api

59

Persistence layer has the most server errors

2

lmg-odin-uat-euwe1-api

3

Main API has very few server errors

Key Insight: The persistence layer API (lmg-odin-uat-euwe1-persistence-layer-api) has significantly more 5xx errors than the main gateway. This suggests the backend/database layer behind this API may need investigation.

Section 6: Monitor Links (Left Panel Note)
The dashboard includes a note widget on the left with direct links to related Datadog monitors. These monitors alert automatically when thresholds are breached.

Monitor

What It Watches

PROD 5xx Monitor (#107107644)

Alerts when PROD 5xx errors avg > 1 in 5m

PROD 4xx Monitor (#107107653)

Alerts when PROD 4xx errors avg > 1 in 5m

UAT 5xx Monitor (#106529892)

Alerts when UAT 5xx errors avg > 1 in 5m

UAT 4xx Monitor (#106529872)

Alerts when UAT 4xx errors avg > 1 in 5m

DEV 5xx Monitor (#106492331)

Alerts when DEV 5xx errors avg > 1 in 5m

DEV 4xx Monitor (#106492324)

Alerts when DEV 4xx errors avg > 1 in 5m

Quick Troubleshooting Guide
"I see a problem — how do I find the exact data product?"
Look at the timeseries charts — each colour = a different API. Hover to see the name.

Check the toplists at the bottom — they rank APIs by traffic and errors.

Use the $apiname dropdown at the top — filter the entire dashboard to one specific API.

The API name IS the data product — e.g., lmg-odin-uat-euwe1-persistence-layer-api = the Persistence Layer data product in UAT.

Decision Tree: Is It Our Problem or the Consumer's?
Problem detected
├── 5xx errors increasing? → YOUR SIDE 🔴
│   ├── One API only? → Check that API's Lambda logs
│   └── All APIs? → Check shared infra (VPC, IAM, DB)
├── 4xx errors increasing? → CONSUMER SIDE 🟡
│   ├── After YOUR deployment? → You may have broken the API contract
│   └── No deployment? → Consumer misconfiguration — contact them
├── Latency increasing? → COULD BE EITHER
│   ├── + 5xx errors? → YOUR SIDE — backend overloaded/failing
│   └── No errors? → Possible DB slowness or cold starts
└── Traffic dropping? → CHECK BOTH SIDES
    ├── Consumer stopped calling? → Contact consumer
    └── API unreachable? → Check DNS, WAF, network
Dashboard: ODIE - API Gateway Health Overview | Author: akashdip.mahapatra@ba.com






Add Comment
Copyright Datadog, Inc. 2026
Version:35.116738157
Master Subscription Agreement
Privacy Policy
Cookie Policy
Datadog Status →: All Systems Operational
 
All Systems Operational
Ask AI
```
