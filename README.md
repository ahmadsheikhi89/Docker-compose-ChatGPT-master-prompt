<div dir="rtl">

# 🎛️ طراحی Master Prompt برای Monitoring Stack (Prometheus + Grafana)

این سند یک نمونه‌ی کامل برای طراحی و استفاده از **Master Prompt** در حوزه‌ی مانیتورینگ (Prometheus + Grafana) است. هدف این است که هر فردی با تجربه‌ی معقول در DevOps بتواند:

* مفهوم Master Prompt را درک کند،
* ساختار آن را تکرارپذیر در پروژه‌های دیگر پیاده‌سازی کند،
* و با **یک پرامپت واحد**، خروجی production‑grade برای Monitoring Stack دریافت کند.

در انتهای این سند، یک **Master Prompt نهایی** در اختیار دارید که می‌توان آن را مستقیماً در ChatGPT یا هر LLM مشابهی استفاده کرد.

سناریوی مرجع این سند:

* Network: `10.10.10.0/24`
* Docker network name: `monitoring_net`
* سرویس‌ها: Prometheus، Alertmanager، Grafana، Node Exporter، cAdvisor

---

## 🎯 ۱. هدف Master Prompt در این سناریو

در این سند، Master Prompt طوری طراحی شده که هر بار اجرا شود، خروجی زیر را به‌صورت منسجم و تکرارپذیر تولید کند:

* معماری سطح بالا و دیاگرام (Mermaid)،
* ساختار دایرکتوری و Volumeها،
* فایل `docker-compose.yml` کامل،
* پیکربندی Prometheus (`prometheus.yml` + rules)،
* پیکربندی Alertmanager (`alertmanager.yml`)،
* Provisioning اولیه Grafana (datasource + dashboard نمونه)،
* `.env.example` برای متغیرهای حساس،
* توصیه‌های امنیتی و Runbookهای عملیاتی.

این رویکرد کمک می‌کند Monitoring Stack همیشه با یک استاندارد تکرارپذیر طراحی و پیاده‌سازی شود.

---

## 🧩 ۲. ساختار مفهومی یک Master Prompt

برای طراحی یک Master Prompt قابل استفاده‌ی مجدد، می‌توان آن را به پنج جزء منطقی تقسیم کرد. در متن نهایی، همه‌ی این اجزاء در قالب یک پرامپت واحد نوشته می‌شوند، اما بهتر است هنگام طراحی به‌صورت جداگانه به آن‌ها فکر کنیم:

1. **ROLE**
   تعیین می‌کند مدل در چه نقشی پاسخ بدهد؛ مثلاً:

   * Senior DevOps Engineer
   * Observability / SRE Engineer

2. **CONTEXT**
   محیط، محدودیت‌ها و شرایط کلی را مشخص می‌کند؛ مانند:

   * On‑prem یا Cloud
   * نوع سیستم‌عامل و Container runtime
   * Subnet، نام Docker network، نوع Registry
   * سطح حساسیت (بانکی، PCI، air‑gapped و ...)

3. **SCOPE**
   دقیقاً مشخص می‌کند چه خروجی‌هایی انتظار داریم؛ برای مثال:

   * docker-compose
   * فایل‌های کانفیگ Prometheus و Alertmanager
   * Provisioning Grafana
   * Runbook و توصیه‌های امنیتی

4. **RULES**
   استانداردهای نگارشی و فنی که پاسخ باید رعایت کند؛ مانند:

   * هر فایل در یک code block مجزا
   * نام فایل در خط اول code block
   * عدم استفاده از توضیحات غیرانگلیسی داخل کد
   * اعتبار سینتکس YAML، Dockerfile و ...

5. **OUTPUT FORMAT**
   قالب نهایی پاسخ و ترتیب بخش‌ها را مشخص می‌کند؛ مثلاً:

   * 0) Assumptions
   * 1. Architecture Overview
   * 2. Directory & Volume Layout
   * ... تا 10) Next Steps / Improvements

در عمل، تمام این اجزاء را در یک متن واحد به مدل می‌دهیم؛ اما تفکیک مفهومی آن‌ها باعث می‌شود طراحی Master Prompt برای تیم ساده و شفاف باشد.

---

## 📐 ۳. اصول طراحی Master Prompt در این سند

برای استانداردسازی، این سند سه اصل ساده را رعایت می‌کند:

1. **نقش شفاف (ROLE مشخص)**
   مدل دقیقاً می‌داند در چه جایگاهی پاسخ می‌دهد (Senior DevOps + SRE در حوزه‌ی Observability).

2. **خروجی ساختارمند و شماره‌گذاری‌شده (OUTPUT FORMAT)**
   پاسخ همیشه با شماره‌گذاری از ۰ تا ۱۰ ارائه می‌شود؛ این کار خوانایی و قابلیّت ارجاع را بالا می‌برد.

3. **قوانین فرمت و تمیزی کد (RULES)**
   تمام فایل‌ها در code blockهای مجزا، با نام فایل روی خط اول، و سینتکس معتبر ارائه می‌شوند.

این سه اصل را می‌توان در هر دامنه‌ی دیگری (Nexus، GitLab، Microservices و …) نیز تکرار کرد.

---

## 🧾 ۴. Master Prompt نهایی (متن واحد و قابل کپی)

این بخش، Master Prompt نهایی است که می‌توانید مستقیماً در یک گفت‌وگوی جدید با مدل استفاده کنید.

**نحوه‌ی استفاده:**

1. یک چت جدید در LLM باز کنید.
2. کل متن زیر را بدون تغییر یا با حداقل شخصی‌سازی (مثلاً جایگزینی `<REGISTRY_URL>` و مسیر داده‌ها) Paste کنید.
3. پیام را ارسال کنید.
4. در صورت نیاز، در پیام بعدی فقط جزئیات تکمیلی سناریوی خود را اضافه کنید.

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
   - A clear directory tree rooted at /data/monitoring (or <MONITORING_DATA_DIR> if the user overrides), including:
     - prometheus/config
     - prometheus/data
     - alertmanager/config
     - grafana/data
     - grafana/provisioning/datasources
     - grafana/provisioning/dashboards
   - Explain which directories are mounted as volumes in docker-compose.

3) docker-compose Stack
   - A complete, syntactically valid docker-compose.yml file that:
     - defines the "monitoring_net" network with subnet 10.10.10.0/24 (or a user-provided subnet);
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
   - Optional high-level note on integrating with SSO (e.g., Keycloak/LDAP).

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
     - integrating a logging stack (e.g., Loki) later;
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

## 🛠️ ۵. نحوه‌ی استفاده‌ی عملی در ریپو یا مستندات

برای استفاده‌ی عملی از این سند در GitHub/GitLab یا ویکی داخلی:

1. این متن را به‌عنوان `README.md` در یک ریپوی آموزشی (مثلاً `monitoring-stack-master-prompt`) قرار دهید.
2. هنگام نیاز به طراحی Monitoring Stack جدید، ابتدا این Master Prompt را در یک گفت‌وگوی جدید با مدل استفاده کنید.
3. خروجی حاصل (docker-compose، فایل‌های کانفیگ، Runbookها و …) را در یک ریپوی جدید (مثلاً `monitoring-stack-prod` یا مشابه) ذخیره و متناسب با محیط واقعی تنظیم کنید.
4. الگوی مفهومی این Master Prompt (ROLE، CONTEXT، SCOPE، RULES، OUTPUT FORMAT) را می‌توان برای سایر دامنه‌ها نیز بازاستفاده کرد؛ فقط کافی است محتوا و اقلام خروجی متناسب با سرویس جدید (Nexus، GitLab، Microservices و …) به‌روزرسانی شود.

به این شکل، طراحی و استقرار Monitoring Stack نه‌تنها استاندارد و تکرارپذیر می‌شود، بلکه می‌تواند به‌عنوان الگوی مرجع برای سایر سرویس‌های زیرساختی نیز مورد استفاده قرار گیرد.

</div>
