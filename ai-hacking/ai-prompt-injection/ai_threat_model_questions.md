# ai\_threat\_model\_questions

## System Inputs & Entry Points

{% stepper %}
{% step %}
### What are all the interfaces through which users can submit prompts to the LLM?

(Enumerate web UI, API endpoints, SDKs, chatbots, CLI, integrations, webhooks, mobile apps, email ingestion, etc.)
{% endstep %}

{% step %}
### Are there any indirect input vectors (file uploads, document processing, etc.)?

(Include file attachments, pasted documents, scraped content, connectors to cloud storage, ingestion pipelines, OCR, data brokers, third‑party content fetchers, and other non-interactive sources.)
{% endstep %}

{% step %}
### How is user authentication handled for different input channels?

(Describe auth mechanisms per channel: OAuth, API keys, JWTs, session cookies, SSO, role mapping, anonymous access controls, and any channel‑specific constraints.)
{% endstep %}

{% step %}
### What input validation exists at each entry point?

(Detail syntactic/semantic validation, rate limits, size/type checks, file type scanning, content sanitization, encoding normalization, schema validation, and upstream filtering.)
{% endstep %}
{% endstepper %}

## Ecosystem Vulnerabilities

{% stepper %}
{% step %}
### What third-party components make up the LLM system's ecosystem?

(List model providers, inference services, orchestration layers, SDKs, data stores, vector DBs, plugins, observability tools, cloud services, and external connectors.)
{% endstep %}

{% step %}
### How are dependencies and libraries secured and updated?

(Describe dependency management, vulnerability scanning, signed packages, pinned versions, SBOMs, patching cadence, and supply chain controls.)
{% endstep %}

{% step %}
### Are there vulnerabilities in the hosting infrastructure?

(Include virtualization/container escape risks, misconfigurations, IAM issues, insecure storage, exposed management planes, and host OS patching.)
{% endstep %}

{% step %}
### What network attack surfaces exist in the system's ecosystem?

(List public endpoints, inter-service APIs, management consoles, exposed ports, ingress/egress flows, misconfigured firewalls, and third‑party connection points.)
{% endstep %}
{% endstepper %}

## Model Security

{% stepper %}
{% step %}
### Is this a proprietary, open-source, or third-party provided LLM?

(Identify ownership, licensing, hosting model—self‑hosted vs managed—and any hybrid arrangements.)
{% endstep %}

{% step %}
### What known model vulnerabilities or weaknesses exist?

(Consider hallucinations, prompt sensitivity, training data biases, memorization of sensitive data, and documented CVEs or advisories.)
{% endstep %}

{% step %}
### Is the model susceptible to adversarial attacks or jailbreaking techniques?

(Assess susceptibility to prompt injection, adversarial token sequences, context manipulation, or crafted inputs aimed at bypassing safety controls.)
{% endstep %}

{% step %}
### How is the model protected against inference manipulation?

(Describe rate limiting, input normalization, model supervision layers, output filters, safety classifiers, and ensemble or voting defenses.)
{% endstep %}
{% endstepper %}

## Prompt Engineering Security

{% stepper %}
{% step %}
### How are system prompts and instructions secured?

(Explain storage/access controls for system prompts, secrets in prompts, CI/CD handling, revision history, and environment separation.)
{% endstep %}

{% step %}
### What measures prevent prompt injection attacks?

(Describe sandboxing, input/output whitelisting, prompt templates with strict placeholders, escape/hardening techniques, and context trimming policies.)
{% endstep %}

{% step %}
### Are there filtering mechanisms for malicious instruction attempts?

(Detail content classifiers, blocklists, safety models, regex heuristics, and escalation flows for suspicious inputs.)
{% endstep %}

{% step %}
### Could prompt leakage expose sensitive system configurations?

(Consider where prompts are logged, debug output, telemetry, embeddings, and access controls that could leak system prompts or secrets.)
{% endstep %}
{% endstepper %}

## Data Security

{% stepper %}
{% step %}
### What sensitive data might be processed by the LLM?

(Enumerate PII, PHI, credentials, API keys, proprietary IP, financial data, customer data, and any regulated information.)
{% endstep %}

{% step %}
### How is training, fine-tuning, and user data secured?

(Describe encryption at rest/in transit, access controls, isolation of training corpora, differential privacy, and secure retraining processes.)
{% endstep %}

{% step %}
### Are vector databases or embeddings protected against leakage?

(Include access controls, encryption, query auditing, anonymization, rate limiting, and vector similarity probing mitigations.)
{% endstep %}

{% step %}
### What data retention and deletion policies are in place?

(Explain retention durations, deletion workflows, backups/snapshots handling, GDPR/CCPA compliance, and verification of deletion.)
{% endstep %}
{% endstepper %}

## Application Security

{% stepper %}
{% step %}
### How is the application layer (frontend, API) secured?

(Describe secure coding practices, input sanitization, CORS policies, CSP, TLS usage, secrets management, and API gateway protections.)
{% endstep %}

{% step %}
### What authentication and authorization controls exist?

(Detail user roles, RBAC/ABAC, MFA, session management, token lifetimes, least privilege, and service identity controls.)
{% endstep %}

{% step %}
### Are there rate limits and abuse prevention mechanisms?

(Include per-user/IP rate limits, throttling, CAPTCHA, behavioral analytics, per‑tenant quotas, and automated throttling backoffs.)
{% endstep %}

{% step %}
### How is the application monitored for unusual behavior?

(Describe telemetry, anomaly detection, logging, tracing, SIEM integration, alerting thresholds, and periodic reviews.)
{% endstep %}
{% endstepper %}

## Pivoting Potential

{% stepper %}
{% step %}
### Could the LLM be used to pivot to other systems?

(Assess whether outputs or access can reveal credentials, internal endpoints, or enable social engineering to reach other systems.)
{% endstep %}

{% step %}
### What lateral movement paths exist if one component is compromised?

(Map privileged network connections, shared credentials, service accounts, and cross‑tenant data access that could enable lateral movement.)
{% endstep %}

{% step %}
### Does the LLM have access or connections to sensitive internal systems?

(Identify integrations with databases, CRMs, internal APIs, file stores, orchestration tools, or admin consoles.)
{% endstep %}

{% step %}
### What is the blast radius if a compromise occurs?

(Estimate affected tenants, data types, system components, and business impact—consider both confidentiality and integrity impacts.)
{% endstep %}
{% endstepper %}

## Monitoring & Response

{% stepper %}
{% step %}
### How are attacks against each vector detected and alerted?

(Describe detection rules per vector, telemetry sources, correlation logic, and alerting channels.)
{% endstep %}

{% step %}
### Is there a specific incident response plan for LLM-related security events?

(Include roles, containment procedures, forensic steps, communication plans, and playbooks tailored to model compromises or data leakage.)
{% endstep %}

{% step %}
### How are security logs collected and analyzed?

(Detail centralized logging, retention, log integrity, parsing for model-specific events, and SOC workflows.)
{% endstep %}

{% step %}
### What is the process for addressing new attack techniques?

(Explain threat intelligence ingestion, red-team/blue-team exercises, patching/updating cycles, and updates to detection/mitigation controls.)
{% endstep %}
{% endstepper %}
