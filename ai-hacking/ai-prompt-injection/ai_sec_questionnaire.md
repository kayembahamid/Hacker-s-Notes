# ai\_sec\_questionnaire

## 1. General Information

* **AI System Name:**
* **Organization/Team Responsible:**
* **Primary Use Case of AI System:**
* **Deployment Status:**
  * [ ] Development
  * [ ] Testing
  * [ ] Production
  * [ ] Decommissioning

***

## 2. Model Security

{% stepper %}
{% step %}
#### What type of AI model is used?

(e.g., LLM, computer vision, reinforcement learning, etc.)
{% endstep %}

{% step %}
#### What system or developer prompt is embedded in the model?
{% endstep %}

{% step %}
#### Is the model proprietary, open-source, or third-party provided?
{% endstep %}

{% step %}
#### Does the model include retrieval-augmented generation (RAG)?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is the model fine-tuned or zero-shot?
{% endstep %}

{% step %}
#### Is the AI system multi-modal?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is the AI agentic (autonomously taking actions)?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### What tools or APIs does the AI interact with?
{% endstep %}
{% endstepper %}

***

## 3. Storage & Data Security

{% stepper %}
{% step %}
#### Where is AI-related data stored?

* [ ] On-premises
* [ ] Cloud
* [ ] Hybrid
{% endstep %}

{% step %}
#### Which databases are used for model inputs, outputs, or embeddings?
{% endstep %}

{% step %}
#### Does the system use vector databases for embeddings?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### How is data encrypted at rest and in transit?
{% endstep %}

{% step %}
#### Are data access controls and audit logs in place?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}

***

## 4. Interface Security

{% stepper %}
{% step %}
#### What type of interface is used to interact with the AI?

* [ ] Chatbot
* [ ] API
* [ ] Data upload portal
* [ ] Other: \_\_\_\_\_\_\_
{% endstep %}

{% step %}
#### How is input data sanitized to prevent prompt injection?
{% endstep %}

{% step %}
#### Does the AI system have rate limiting or authentication for external users?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is there an approval process for API integrations with external tools?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}

***

## 5. Network Architecture & Components

{% stepper %}
{% step %}
#### Is the AI system exposed to external networks?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Does it integrate with external applications?

* [ ] Yes
* [ ] No

If yes, which applications? (e.g., Salesforce, Slack, Microsoft Teams, etc.)
{% endstep %}

{% step %}
#### Does it integrate with internal applications?

* [ ] Yes
* [ ] No

If yes, does it have read/write permissions?

* [ ] Read
* [ ] Write
* [ ] Both
{% endstep %}

{% step %}
#### Does the AI system use open-source software (OSS)?

* [ ] Yes
* [ ] No

If yes, are dependencies monitored for vulnerabilities?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Does the system use internal APIs?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Does it use a headless browser (e.g., Puppeteer, Selenium)?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is there a human-in-the-loop (HITL) component for oversight?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}

***

## 6. Development Environment Security

{% stepper %}
{% step %}
#### Where is the AI training environment hosted?

* [ ] On-premises
* [ ] Cloud
* [ ] Hybrid
{% endstep %}

{% step %}
#### What platform is used for prompt engineering and fine-tuning?
{% endstep %}

{% step %}
#### What development and AI/ML tools are used? (Select all that apply)

* [ ] MLflow
* [ ] Kubeflow
* [ ] Apache Airflow
* [ ] H2O.ai
* [ ] TensorFlow
* [ ] PyTorch
* [ ] AI-as-a-Service (Amazon SageMaker, Azure ML, Google Vertex AI, etc.)
* [ ] Other: \_\_\_\_\_\_\_\_\_\_
{% endstep %}

{% step %}
#### How is identity and access management (IAM) handled for developers?
{% endstep %}

{% step %}
#### Are AI-related workloads isolated from general IT infrastructure?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}

***

## 7. AI Supply Chain Security

{% stepper %}
{% step %}
#### Does the AI system use third-party AI models or datasets?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Are security controls in place for model registries?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is there a process to detect backdoored or malicious AI models?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Are virtual machines, containers, and AI platforms hardened against exploits?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Has the AI system undergone a supply chain security assessment?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Are Docker images used, and if so, are they verified for security?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}

***

## 8. Bias, Safety, and Accuracy

{% stepper %}
{% step %}
#### Is bias a concern for this AI system?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Are fairness and ethical considerations documented?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Does the AI system make decisions with potential legal or financial impact?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Has the AI been tested for biases in race, gender, age, or other factors?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Can the AI system be manipulated into providing unfair advantages (e.g., forced discounts)?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### What safeguards are in place to prevent harmful outputs?
{% endstep %}
{% endstepper %}

***

## 9. Security Testing & Incident Response

{% stepper %}
{% step %}
#### Has the AI system undergone security penetration testing?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Is there an incident response plan specific to AI-related attacks?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### Are AI-generated outputs monitored for anomalies?

* [ ] Yes
* [ ] No
{% endstep %}

{% step %}
#### What methods are used to detect adversarial attacks or prompt injection?
{% endstep %}

{% step %}
#### Are security patches and model updates tracked and applied?

* [ ] Yes
* [ ] No
{% endstep %}
{% endstepper %}
