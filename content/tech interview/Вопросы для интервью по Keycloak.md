Keycloak — это open-source решение для управления идентификацией и доступом (IAM), поддерживающее **OAuth 2.0, OpenID Connect (OIDC) и SAML**. Вопросы разделены по темам, начиная от базовых понятий и заканчивая продвинутыми настройками.

---

### **1️⃣ Основные концепции Keycloak**

1. [[Что такое Keycloak и для чего он используется]]?
2. [[Какие основные компоненты архитектуры Keycloak]]?
3. [[Что такое Realm в Keycloak]]?
4. [[Какие есть типы Clients в Keycloak]]?
5. [[Как Keycloak управляет Users]]?
6. [[Что такое Roles в Keycloak, и какие бывают их типы]]?
7. [[Что такое Groups, и как они связаны с ролями]]?
8. Какие способы аутентификации поддерживает Keycloak?
9. Что такое **Identity Providers (IdP)** в Keycloak?

---

### **2️⃣ Настройка аутентификации и авторизации**

1. Как работает процесс аутентификации в Keycloak?
2. Какие протоколы аутентификации поддерживает Keycloak?
3. Чем отличается OAuth 2.0 от OpenID Connect?
4. Как настроить **Single Sign-On (SSO)** в Keycloak?
5. Как настроить **многофакторную аутентификацию (MFA)** в Keycloak?
6. Что такое **Authentication Flows** и как их настроить?
7. Как настроить **OTP (One-Time Password)** в Keycloak?
8. Как организовать **логин через социальные сети** (Google, Facebook, GitHub)?
9. Как работает **passwordless authentication** в Keycloak?

---

### **3️⃣ Работа с токенами (JWT, Refresh, Access)**

1. Какие типы токенов использует Keycloak?
2. Что содержит **Access Token** в Keycloak?
3. В чём разница между **Access Token** и **ID Token**?
4. Как настроить **Refresh Token** в Keycloak?
5. Как изменить **время жизни токенов** в Keycloak?
6. Как работают **scopes** и **claims** в Keycloak?
7. Как настроить **подпись токенов (JWT Signature Algorithm)**?
8. Как обновить или отозвать токен (Token Revocation)?

---

### **4️⃣ Интеграция с внешними системами**

1. Как интегрировать Keycloak с **Spring Boot / Java / Node.js**?
2. Как настроить аутентификацию через **LDAP / Active Directory**?
3. Как добавить Keycloak в **NGINX или Apache** в качестве reverse proxy?
4. Как реализовать **API Gateway с Keycloak**?
5. Как использовать Keycloak с **Kubernetes и Istio**?
6. Какие основные различия между Keycloak и других IAM решений (Okta, Auth0, AWS Cognito)?

---

### **5️⃣ Конфигурация и управление Keycloak**

1. Как можно управлять Keycloak через **Admin REST API**?
2. Как настроить **Keycloak Client Adapters** для различных языков?
3. Как работают **Client Scopes**, и зачем они нужны?
4. Как настроить **Role-Based Access Control (RBAC)**?
5. Как настроить **Attribute-Based Access Control (ABAC)**?
6. Как создать **custom user attributes** в Keycloak?
7. Как настроить **E-mail notifications** (SMTP)?
8. Как можно экспортировать и импортировать Keycloak настройки?

---

### **6️⃣ Безопасность и продвинутые настройки**

1. Какие механизмы защиты от атак (например, brute-force) есть в Keycloak?
2. Как настроить **IP-рестрикции (IP Restrictions)** в Keycloak?
3. Как работает **Trusted Hosts Policy**?
4. Как настроить **Fine-Grained Permissions** в Keycloak?
5. Как реализовать **декларативные политики безопасности (Authorization Policies)**?
6. Как защитить API с помощью Keycloak?
7. Как можно добавить **PKCE (Proof Key for Code Exchange)**?
8. Как настроить Keycloak для работы с **MTLS (Mutual TLS)**?

---

### **7️⃣ Мониторинг и развертывание**

1. Как развернуть Keycloak в **Docker / Kubernetes**?
2. Какие базы данных поддерживает Keycloak (и как их настроить)?
3. Как Keycloak управляет **кластеризацией**?
4. Как мониторить Keycloak с помощью **Prometheus и Grafana**?
5. Как настроить **логирование и аудит** в Keycloak?
6. Какие механизмы **failover и high availability (HA)** поддерживает Keycloak?
7. Как правильно бэкапить Keycloak и восстанавливать данные?

---

### **8️⃣ Кастомизация и расширение функционала**

1. Как создать **custom authentication flow** в Keycloak?
2. Как добавить **новый провайдер аутентификации**?
3. Как реализовать **Custom Event Listeners**?
4. Как расширить Keycloak с помощью **SPI (Service Provider Interface)**?
5. Как изменить UI страниц логина (Freemarker templates)?

---

## 🎯 **Дополнительные вопросы для senior-уровня**

1. Как работает механизм **Offline Sessions** в Keycloak?
2. Как Keycloak управляет кэшированием с помощью Infinispan?
3. В чём разница между **Keycloak.X и стандартным Keycloak**?
4. Как Keycloak может работать в **multi-tenant** среде?
5. Как работает **User Federation**?
6. Как настроить Keycloak для работы в режиме **Read-Only Mode**?
7. Какие есть альтернативы Keycloak, и когда стоит их использовать?
8. Как масштабировать Keycloak в облачных средах (AWS, Azure, GCP)?
9. Как работают **Cross-Realm Token Exchange**?
10. Как защитить Keycloak от DDoS-атак?