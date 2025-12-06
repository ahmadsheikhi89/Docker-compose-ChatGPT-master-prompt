# آموزش طراحی Master Prompt برای Monitoring Stack (Prometheus + Grafana)

این فایل برای این است که **جونیورها** (و حتی سینیورها) یاد بگیرند چطور فقط با **یک پرامپت واحد**، یک خروجی کامل و production‑grade برای مانیتورینگ (Prometheus + Grafana) بگیرند.

در انتهای این آموزش، یک **Master Prompt نهایی** داری که:

* فقط یک بلوک متن است (یک پیام)؛
* می‌توانی مستقیم در ChatGPT یا هر LLM دیگری Paste کنی؛
* برایت معماری، docker-compose، `prometheus.yml`، `alertmanager.yml`، Grafana provisioning، `.env.example` و Runbook تولید می‌کند.

سناریوی نمونه ما:

* Network: `10.10.10.0/24`
* Docker network name: `monitoring_net`
* سرویس‌ها: Prometheus، Alertmanager، Grafana، Node Exporter، cAdvisor

---

## ۱. ایده‌ی اصلی Master Prompt (فهم برای جونیورها)

برای این‌که خروجی مدل «هر دفعه» تمیز و قابل استفاده باشد، ما یک **Master Prompt** می‌سازیم که ۵ جزء اصلی دارد، اما در عمل آن‌ها را به‌صورت **یک متن واحد** می‌نویسیم:

1. **ROLE** – مدل در چه نقشی باشد؟ (مثلاً Senior DevOps + SRE)
2. **CONTEXT** – محیط ما چیست؟ (On‑prem, Docker, IP range, Registry داخلی و…)
3. **SCOPE** – دقیقاً چه چیزهایی را باید تولید کند؟ (docker-compose، configs، Runbook و…)
4. **RULES** – قوانین تمیزی کد، فرمت خروجی، نام‌گذاری‌ها و…
5. **OUTPUT FORMAT** – ترتیب و شماره‌ی بخش‌های خروجی (۰ تا ۱۰)

نکته‌ی مهم برای آموزش:

* ما این ۵ بخش را **در ذهن** جدا می‌بینیم، ولی در عمل برای مدل، **همه را در یک بلوک متن** می‌نویسیم.
* جونیور فقط لازم است بداند:

  * ROLE یعنی «نقش» مدل؛
  * CONTEXT یعنی «کجا هستیم و چه محدودیت‌هایی داریم»؛
  * SCOPE یعنی «چی می‌خواهیم»؛
  * RULES یعنی «چطور بنویسد»؛
  * OUTPUT FORMAT یعنی «خروجی را با چه ساختاری بدهد».

---

## ۲. سه قانون طلایی برای Master Prompt خوب

این سه قانون را همیشه روی تخته/اسلاید برای جونیورها بگذار:

1. **یک نقش مشخص (ROLE) تعریف کن**
   نگویید «کمک کن مانیتورینگ راه بیندازم»، بگو:
   «You are a Senior DevOps Engineer and Observability/SRE Engineer…»

2. **خروجی را شماره‌گذاری کن (OUTPUT FORMAT)**
   مثلاً بگو:
   `0) Assumptions`, `1) Architecture Overview`, `2) Directory Layout`, …
   این کار باعث می‌شود هر بار خروجی ساختار ثابت داشته باشد.

3. **قوانین فرمت را صریح بنویس (RULES)**
   مثل:

   * هر فایل در یک code block جدا
   * روی خط اول نام فایل را بنویس (`# docker-compose.yml`)
   * YAML و Dockerfile معتبر باشند
   * کامنت فارسی داخل کد ممنوع

اگر این سه را رعایت کنی، بقیه‌ی داستان فقط «جزئیات دامنه» است.

---

## ۳. Master Prompt نهایی (یک بلوک، آماده‌ی استفاده)

این همان چیزی است که در نهایت جونیور باید کپی کند و به مدل بدهد.
**فقط کافی است مقادیر داخل <> را برای پروژه‌ی خودش عوض کند** (یا همین‌طور پیش‌فرض رها کند).

> مرحله استفاده:
>
> 1. یک چت جدید باز کن.
> 2. کل متن زیر را (با شخصی‌سازی مقادیر داخل < > در صورت نیاز) Paste کن.
> 3. Enter بزن.
> 4. اگر لازم بود، در پیام بعدی فقط جزئیات بیشتری از سناریو بده.

```text
ROLE
You are a Senior DevOps Engineer and Observability/SRE Engineer.
Your job is to design and document a complete, production-grade monitoring stack based on Prometheus and Grafana.

CONTEXT
- Environment: On-prem (or air-gapped friendly) enterprise/banking environment.
- OS: Rocky Linux 9.x (or any RHEL-compatible Linux) on a dedicated monitoring host.
- Container runtime: Docker with docker-compose.
- Network: A custom Docker bridge network named "monitoring_net".
- Default subnet: 10.10.10.0/24 (can be overridden if the user provides another subnet).
- Registry: All images should be pulled from an internal registry (e.g., <REGISTRY_URL>) and NOT directly from the public internet.
- Stack components:
  - Prometheus (metrics storage and scraping)
  - Alertmanager (alert routing)
  - Grafana (dashboards)
  - Node Exporter (host metrics)
  - cAdvisor (container metrics, optional but recommended)

SCOPE (ALWAYS COVER)
You MUST always cover at least the following items for the monitoring stack:

1) Architecture
   - High-level architecture description in text.
   - A Mermaid component diagram showing:
     - monitored nodes (node_exporter)
     - cAdvisor
     - Prometheus
     - Alertmanager
     - Grafana
     - Docker network "monitoring_net" and how components communicate.

2) Directory & Volume Layout
   - A clear directory tree rooted at /data/monitoring (or <MONITORING_DATA_DIR> if user overrides), including:
     - prometheus/config
     - prometheus/data
     - alertmanager/config
     - grafana/data
     - grafana/provisioning/datasources
     - grafana/provisioning/dashboards
   - Explain which directories are mounted as volumes in docker-compose.

3) docker-compose Stack
   - A complete, syntactically valid docker-compose.yml file that:
     - defines the "monitoring_net" network with subnet 10.10.10.0/24 (or user provided subnet);
     - defines the services: prometheus, alertmanager, grafana, node_exporter, cadvisor;
     - uses bind mounts under /data/monitoring for persistent data and configs;
     - exposes only necessary ports externally (e.g., Grafana and Prometheus HTTP), keeping everything else internal.

4) Prometheus Configuration
   - A complete prometheus.yml file with:
     - global scrape and evaluation intervals;
     - scrape jobs for:
       - Prometheus itself
       - node_exporter
       - cadvisor
     - alerting configuration pointing to Alertmanager.
   - At least one alert rules file (e.g., alerts/general-rules.yml) with basic alerts:
     - instance down
     - high CPU usage
     - high memory usage
     - high disk usage.

5) Alertmanager Configuration
   - A valid alertmanager.yml configuration file that includes:
     - at least one route;
     - at least one receiver with placeholders (e.g., email, webhook, chat ops);
     - reasonable grouping and repeat interval settings.

6) Grafana Provisioning
   - docker-compose configuration for Grafana with persistent storage.
   - Provisioning files:
     - a Prometheus datasource definition (e.g., provisioning/datasources/prometheus.yml);
     - at least one example dashboard JSON for host/container metrics.
   - Grafana admin credentials must come from environment variables or .env (no hardcoded secrets).

7) Environment & Secrets
   - A .env.example file that includes placeholders for:
     - Grafana admin user and password;
     - SMTP or webhook endpoints for Alertmanager (if applicable);
     - internal registry credentials (if needed).
   - A short explanation (outside code) of how to use the .env file with docker-compose.

8) Security & Hardening
   - Recommendations for:
     - restricting access to Prometheus and Grafana (firewall, reverse proxy, VPN);
     - disabling anonymous access to Grafana and restricting dashboard editing;
     - using strong credentials and not exposing ports unnecessarily.
   - Optional (but recommended) high-level note on integrating with SSO (e.g., Keycloak/LDAP).

9) Runbooks & Operational Procedures
   - A concise operational runbook that describes:
     - how to start, stop, and restart the monitoring stack using docker-compose;
     - how to check health (HTTP endpoints, basic PromQL queries, Grafana status);
     - how to validate that alerts work (e.g., firing a test alert).
   - An incident runbook for common issues:
     - Prometheus down or not scraping targets;
     - Grafana not reachable;
     - disk full on /data/monitoring.

10) Next Steps / Improvements
   - Clear suggestions for next steps, such as:
     - adding more exporters (database, application, blackbox);
     - integrating logging stack (e.g., Loki) later;
     - adding HA for Prometheus/Grafana if the environment requires higher availability.

RULES
- Put every file/content in its own clean code block with a proper filename comment on the first line (e.g., "# docker-compose.yml", "# prometheus/prometheus.yml").
- All YAML, Dockerfiles, docker-compose, and shell snippets MUST be syntactically valid.
- Do NOT include non-English comments inside any code or config file. Explanations and notes must be outside code blocks in English.
- Use the Docker network name "monitoring_net" and subnet 10.10.10.0/24 by default, unless the user explicitly provides a different network.
- Use clear, semantic names for services and directories (prometheus, alertmanager, grafana, node_exporter, cadvisor).
- If information is missing from the user, choose best-practice defaults and explicitly list them under "0) Assumptions" in the answer.

OUTPUT FORMAT (STRICT ORDER)
Always structure your answer in the following numbered sections:

0) Assumptions
1) Architecture Overview (text + Mermaid diagram)
2) Directory & Volume Layout (directory tree + explanation)
3) docker-compose Stack (docker-compose.yml and any related notes)
4) Prometheus Config (prometheus.yml + alert rule file)
5) Alertmanager Config (alertmanager.yml)
6) Grafana Provisioning (datasource + example dashboard JSON)
7) Environment & Secrets (.env.example + explanation)
8) Security & Hardening Recommendations
9) Runbooks (operations + incident handling)
10) Next Steps / Improvements

TONE
- Be concise, precise, and operational.
- All outputs must be production-grade and copy-paste friendly for a real on-prem environment.
```

---

## ۴. چطور این را برای دامنه‌های دیگر هم استفاده کنند؟

برای آموزش جونیورها، می‌توانی بگویی:

* اگر به‌جای Monitoring، مثلاً **Nexus**، **GitLab** یا **Application Stack** می‌خواهید، فقط کافی است:

  * بخش **CONTEXT** و **SCOPE** را با دامنه‌ی جدید عوض کنید؛
  * لیست خروجی‌ها (OUTPUT FORMAT) را متناسب با آن سرویس تنظیم کنید.
* اما همان ۵ جزء اصلی (ROLE، CONTEXT، SCOPE، RULES، OUTPUT FORMAT) را همیشه حفظ کنید.

به این شکل، با فقط **یک Master Prompt**، خروجی‌های خیلی تمیز و ثابت می‌گیرید و جونیورها هم دقیقاً می‌فهمند «چه چیزی را باید به مدل بگویند» تا خروجی استفاده‌پذیر تحویل بگیرند.
