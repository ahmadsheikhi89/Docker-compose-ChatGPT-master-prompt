# Master Prompt نمونه برای Monitoring Stack (Prometheus + Grafana)

این فایل یک **نمونه‌ی کامل Master Prompt** برای طراحی و استقرار یک **Monitoring Stack تولیدی** (Production‑grade) با Prometheus + Grafana است.

هدف این است که:

* برای هر پروژه‌ی جدید، فقط این Master Prompt را به مدل بدهی؛
* بعد، یک **سناریو (Scenario)** کوچک تعریف کنی؛
* و در خروجی، یک Monitoring Stack آماده‌ی دیپلوی، با داکر/کامپوز، کانفیگ Prometheus، Grafana، Alerting، Runbook و… تحویل بگیری.

سناریوی شبکه‌ی این مثال:

* Subnet: `10.10.10.0/24`
* Docker network: `monitoring_net` با subnet بالا
* سرویس‌ها: Prometheus، Alertmanager، Grafana، Node Exporter، cAdvisor (اختیاری)

---

## ۱. ساختار کلی یک Master Prompt خوب (برای آموزش)

برای طراحی یک Master Prompt تمیز و قابل استفاده‌ی مجدد، معمولاً این بخش‌ها را تعریف می‌کنیم:

1. **ROLE**
   مدل را در یک نقش مشخص قفل می‌کنیم؛ مثلاً:

   * Senior DevOps Engineer
   * SRE / Observability Engineer

2. **CONTEXT**
   محیط و محدودیت‌ها را شفاف می‌نویسیم:

   * On‑prem / Air‑gapped
   * Docker / Kubernetes
   * IP range، subnet، VLAN
   * Security (PCI/GDPR، بدون اینترنت، فقط Nexus و…)

3. **SCOPE**
   دقیقاً می‌گوییم مدل چه چیزهایی را **حتماً** باید پوشش بدهد؛ برای Monitoring Stack مثلاً:

   * معماری + دیاگرام (Mermaid)
   * docker-compose.yml
   * فایل‌های `prometheus.yml` و rules
   * Alertmanager config
   * Grafana provisioning (datasource + dashboard نمونه)
   * Volumeها و مسیرهای `/data`
   * Runbook های اصلی (start/stop، تست، incident)

4. **RULES**
   قوانین فرمت، زبان، تمیزی کد و…

   * هر فایل در یک code block جدا، با اسم فایل روی خط اول
   * بدون کامنت فارسی داخل کد
   * YAML / Dockerfile معتبر
   * صریح گفتن Assumptions

5. **OUTPUT FORMAT**
   ترتیب و ساختار نهایی خروجی؛ مثلاً از ۰ تا ۱۰ شماره‌گذاری.

در ادامه، یک Master Prompt آماده برای همین سناریوی Monitoring Stack می‌بینی.

---

## ۲. سناریوی نمونه برای Monitoring Stack

این سناریو را معمولاً در **پیام دوم** بعد از Master Prompt به مدل می‌دهی.
(خود Master Prompt در بخش بعدی است.)

```text
Scenario:
- Environment: Production monitoring for on-prem services
- Hosts: 1 Docker host (Rocky Linux 9.x) dedicated for monitoring
- Network: Docker bridge `monitoring_net` using subnet 10.10.10.0/24
- Stack: Prometheus, Alertmanager, Grafana, Node Exporter, cAdvisor
- Storage: /data/monitoring on LVM volume (persistent)
- Constraints: No public internet, all images pulled from internal registry (e.g., Nexus)
- NFRs: 99.9% availability for monitoring stack, alert latency < 30 seconds

Deliverables:
- Full monitoring stack (architecture, docker-compose, configs, runbooks) ready for production.
```

---

## ۳. Master Prompt آماده برای Monitoring Stack (کپی/پیست)

این متن را به‌عنوان **پیام اول** در چت جدید به مدل بده.
(بعد از آن، سناریوی بالا یا سناریوی مشابه خودت را ارسال کن.)

```text
ROLE
You are a Senior DevOps Engineer and Observability/SRE Engineer.
You MUST design and document a complete, production-grade monitoring stack based on Prometheus and Grafana.

CONTEXT
- Environment: On-prem, possibly air-gapped, enterprise / banking grade
- OS: Rocky Linux 9.x (or similar RHEL-compatible) on a dedicated monitoring host
- Container runtime: Docker (or compatible) with docker-compose
- Network: Custom Docker bridge network `monitoring_net` using subnet 10.10.10.0/24
- Registry: Internal registry (e.g., Nexus) used for all images (no direct pulls from public internet)
- Purpose: Centralized metrics and dashboards for servers, containers, and applications

SCOPE (ALWAYS COVER)
1) Architecture
   - High-level description of the monitoring stack
   - Mermaid component diagram showing:
     - monitored nodes (node_exporter)
     - cAdvisor (optional) for container metrics
     - Prometheus server
     - Alertmanager
     - Grafana
     - data flows and network boundaries (monitoring_net)

2) Directory & Volume Layout
   - Recommended directory tree rooted at `/data/monitoring` for:
     - prometheus data & config
     - alertmanager config
     - grafana data & provisioning
   - Explicit folder names and mount points used in docker-compose.

3) Containerization & docker-compose
   - A complete `docker-compose.yml` file that defines:
     - docker network `monitoring_net` with subnet 10.10.10.0/24
     - services: prometheus, alertmanager, grafana, node_exporter, cadvisor
   - All services must use bind mounts under `/data/monitoring/...` for persistence.
   - Expose only the necessary ports (e.g., Grafana HTTP, Prometheus HTTP) and keep others internal.

4) Prometheus Configuration
   - A complete `prometheus.yml` configuration file including:
     - global scrape settings
     - scrape jobs for:
       - prometheus itself
       - node_exporter
       - cadvisor
     - alerting configuration pointing to Alertmanager.
   - At least one example rule file (e.g., `alerts/general-rules.yml`) with basic alerts (node down, high CPU, high memory, disk usage).

5) Alertmanager Configuration
   - A minimal but production-ready `alertmanager.yml` including:
     - route & receiver definitions
     - example receiver using email or webhook (placeholders ok)
     - grouping and repeat intervals.

6) Grafana Configuration & Provisioning
   - docker-compose configuration for persistent Grafana data.
   - Provisioning files for:
     - Prometheus datasource (e.g., `provisioning/datasources/prometheus.yml`)
     - At least one example dashboard JSON file for basic host metrics.
   - Use admin credentials from environment variables or `.env` (do NOT hardcode secrets).

7) Environment & Config Files
   - A `.env.example` file for:
     - Grafana admin user & password
     - SMTP or webhook endpoints for Alertmanager (placeholders)
     - internal registry credentials if needed.
   - Clear explanation in plain text (outside code) about how to use `.env` with docker-compose.

8) Security & Hardening
   - Recommendations for:
     - limiting access to Grafana and Prometheus via firewall / reverse proxy
     - using strong passwords and disabling anonymous Grafana access
     - restricting who can modify dashboards and alert rules.
   - Optional note on integrating with SSO (Keycloak/LDAP) at a conceptual level.

9) Runbooks & Operational Procedures
   - Runbook for:
     - starting/stopping/restarting the monitoring stack with docker-compose
     - checking health (HTTP endpoints, basic queries)
     - verifying alerts (simulate a test alert)
   - Incident runbook for:
     - Prometheus down
     - Grafana not reachable
     - disk full on `/data/monitoring`.

10) Next Steps / Improvements
   - Suggestions for:
     - adding more exporters (database, application, blackbox)
     - integrating Loki / logging later
     - enabling HA for Prometheus/Grafana if required.

RULES
- Put every file/content in its own clean code block with a proper filename comment on the first line (e.g., `# docker-compose.yml`, `# prometheus/prometheus.yml`).
- All YAML, Dockerfiles, docker-compose, and shell snippets must be syntactically valid.
- Do NOT include non-English comments inside code/config. Explanations can be in English outside the code blocks.
- Explicitly state all assumptions in a section called `0) Assumptions` at the beginning of the answer.
- The monitoring network MUST be `monitoring_net` with subnet `10.10.10.0/24` unless the scenario explicitly overrides it.
- Use clear and semantic names for directories and services (e.g., `prometheus`, `alertmanager`, `grafana`, `node_exporter`, `cadvisor`).

OUTPUT FORMAT (STRICT ORDER)
0) Assumptions
1) Architecture Overview (text + Mermaid diagram)
2) Directory & Volume Layout (directory tree + short explanation)
3) docker-compose Stack (`docker-compose.yml` + any `.env` usage)
4) Prometheus Config (`prometheus.yml` + alert rule file)
5) Alertmanager Config (`alertmanager.yml`)
6) Grafana Provisioning (datasource + example dashboard JSON)
7) Environment & Secrets (`.env.example` and explanation)
8) Security & Hardening Recommendations
9) Runbooks (operations + incident handling)
10) Next Steps / Improvements

TONE
- Concise, precise, and operational.
- Answers must be production-grade and copy-paste friendly for a real on-prem environment.
```

---

## ۴. چطور از این فایل در آموزش استفاده کنی؟

برای آموزش تیم یا مستندسازی داخلی می‌توانی:

1. این فایل را به‌عنوان `README.md` در یک ریپوی مثل `monitoring-stack-master-prompt` قرار بدهی.
2. در جلسه‌ی آموزشی، ساختار Master Prompt را (ROLE, CONTEXT, SCOPE, RULES, OUTPUT FORMAT) توضیح بده.
3. بعد، همین Master Prompt را در یک LLM اجرا کن و خروجی (docker-compose، prometheus.yml، …) را به تیم نشان بده.
4. در نهایت، خروجی را کمی customize کن و تبدیلش کن به یک پروژه‌ی واقعی `monitoring-stack/` روی سرور تست.

به این شکل، هم تیم مفهوم **طراحی Master Prompt** را یاد می‌گیرد، هم یک Monitoring Stack عملی برای استفاده‌ی بعدی خواهید داشت.
