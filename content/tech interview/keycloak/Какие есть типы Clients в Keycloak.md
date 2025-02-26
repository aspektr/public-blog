В **Keycloak**, клиент (**Client**) — это **приложение или сервис**, который использует Keycloak для аутентификации и авторизации.

Clients могут быть:  
✅ **Веб-приложения
✅ **Мобильные приложения**
✅ **Бэкенд-сервисы**

---

## 📌 **Типы Clients в Keycloak**

Keycloak поддерживает **четыре типа клиентов**, каждый из которых подходит для определённого сценария:

| **Тип клиента**                               | **Описание**                                                                                 | **Примеры**                                                |
| --------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Confidential**                              | Закрытый клиент, который аутентифицируется через **client_id + client_secret**.              | Бэкенды                                                    |
| **Public**                                    | Открытый клиент, который **не имеет client_secret**.                                         | SPA-приложения (React, Vue, Angular), мобильные приложения |
| **Bearer-Only**                               | Клиент, который **не выполняет аутентификацию** пользователей, а только проверяет их токены. | Бэкенды                                                    |
| **Service Account (Client Credentials Flow)** | Клиент, который используется **без участия пользователя**, а только с client_secret.         | Бэкенды, серверные фоновые задачи                          |

---

## 🔹 **1. Confidential (Закрытый клиент, для бэкенда)**

- Используется для серверных приложений.
- Требует аутентификацию **client_id + client_secret**.
- Может использовать **Authorization Code Flow** или **Client Credentials Flow**.

📌 **Пример конфиденциального клиента (Spring Boot, backend API)**


```
keycloak.auth-server-url=https://keycloak.example.com/auth
keycloak.realm=myrealm
keycloak.resource=my-backend
keycloak.credentials.secret=your_secret

```


📌 **Пример получения токена через client_id и client_secret (cURL):**


```bash
curl -X POST "https://keycloak.example.com/auth/realms/myrealm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_id=my-backend" \
     -d "client_secret=your_secret" \
     -d "grant_type=client_credentials"

```

✅ **Результат:**  
Бэкенд получает **access_token** и использует его для доступа к API.

---

## 🔹 **2. Public (Открытый клиент, для фронтенда и мобильных приложений)**

- Используется в **SPA (React, Angular, Vue), мобильных приложениях**.
- **НЕ имеет client_secret** (т.к. клиентский секрет нельзя хранить в открытом коде).
- Использует **PKCE (Proof Key for Code Exchange)** для защиты авторизации.

📌 **Пример настройки клиента в Keycloak:**

- **Access Type**: `public`
- **Standard Flow Enabled**: ✅ (OAuth 2.0 Authorization Code Flow)
- **Implicit Flow Enabled**: ❌ (не рекомендуется)
- **Direct Access Grants Enabled**: ❌ (запрещено)

📌 **Пример получения токена в браузере (JavaScript + Keycloak JS SDK)**:

```javascript
const keycloak = new Keycloak({
    url: "https://keycloak.example.com/auth",
    realm: "myrealm",
    clientId: "my-frontend"
});

keycloak.init({ onLoad: "login-required" }).then(authenticated => {
    console.log("User authenticated:", authenticated);
    console.log("Access Token:", keycloak.token);
});

```

✅ **Результат:**  
Пользователь аутентифицируется, а SPA-приложение получает JWT-токен.

---

## 🔹 **3. Bearer-Only (Только проверка токена, без аутентификации)**

- Используется для **API и микросервисов**, которые принимают только **готовые токены**.
- Клиент **не выполняет аутентификацию**, а просто валидирует JWT-токены пользователей.
- **Access Type: bearer-only**.

📌 **Пример настройки Bearer-Only клиента:**

```json
{
  "clientId": "my-api",
  "enabled": true,
  "bearerOnly": true,
  "publicClient": false
}

```

📌 **Пример запроса к API с токеном (Bearer Token):**

```bash
curl -X GET "https://api.example.com/protected-resource" \
     -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR..."

```

✅ **Результат:**  
Keycloak проверяет токен, API обрабатывает запрос **только если токен валиден**.

---

## 🔹 **4. Service Account (Client Credentials Flow, для сервисов)**

- Используется для **фоновых задач, интеграций, микросервисов**.
- Не требует участия пользователя.
- Работает только с **client_id + client_secret**.
- Использует **Client Credentials Grant**.

📌 **Пример настройки в Keycloak:**

- **Access Type**: `confidential`
- **Service Accounts Enabled**: ✅

📌 **Пример получения access token (Service-to-Service Auth)**

```bash
curl -X POST "https://keycloak.example.com/auth/realms/myrealm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials" \
     -d "client_id=my-service" \
     -d "client_secret=your_secret"

```

✅ **Результат:**  
Токен выдаётся без участия пользователя и может использоваться сервисом.

---

## 📊 **Сравнение всех типов Clients в Keycloak**

| Тип клиента         | Где используется?                      | Имеет client_secret? | Выполняет аутентификацию?       | Может работать без пользователя? |
| ------------------- | -------------------------------------- | -------------------- | ------------------------------- | -------------------------------- |
| **Confidential**    | Backend API, серверные приложения      | ✅ Да                 | ✅ Да                            | 🔸 Нет                           |
| **Public**          | SPA (React, Vue), мобильные приложения | ❌ Нет                | ✅ Да                            | 🔸 Нет                           |
| **Bearer-Only**     | API, микросервисы                      | ❌ Нет                | ❌ Нет (только проверяет токены) | ✅ Да                             |
| **Service Account** | Фоновые сервисы, автоматизация         | ✅ Да                 | ❌ Нет                           | ✅ Да                             |

---

## ✅ **Вывод**

- **Confidential** → для **бэкенд-приложений**, которые выполняют аутентификацию.
- **Public** → для **SPA и мобильных приложений**, без хранения client_secret.
- **Bearer-Only** → для **API и микросервисов**, которые проверяют токены.
- **Service Account** → для **автоматизированных сервисов**, которым не нужен пользователь.