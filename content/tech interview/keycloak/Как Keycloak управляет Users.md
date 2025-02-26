Keycloak предоставляет **гибкую систему управления пользователями**, включая их создание, аутентификацию, управление ролями и группами, а также интеграцию с внешними источниками (LDAP, Active Directory, соцсети).

---

## 📌 **1. Где хранятся пользователи?**

Пользователи в Keycloak хранятся в **Realm** (области безопасности).

Каждый **realm** управляет **своими пользователями**.  
✔ **Пользователь из одного realm НЕ может войти в другой realm (если не настроен federation или cross-realm SSO).**

📌 **Пример:**

- В `ecommerce-realm` могут быть пользователи покупателей.
- В `admin-realm` могут быть только сотрудники компании.

---

## 🔹 **2. Как управлять пользователями в Keycloak?**

Есть **три способа** управления пользователями:

### 🟢 **1. Через Admin Console (веб-интерфейс)**

1. Войдите в **Keycloak Admin Console** (`http://localhost:8080/admin`).
2. Выберите **Realm** → `Users` → `Add User`.
3. Заполните:
    - **Username**: `john_doe`
    - **Email**: `john@example.com`
    - **Enabled**: ✅
    - **Email Verified**: ✅ (если email подтверждён)
4. Нажмите **Save**, затем установите пароль во вкладке **Credentials**.

✅ **Результат:**  
Создан пользователь `john_doe`, он может войти в систему.

---

### 🟢 **2. Через REST API**

Keycloak позволяет программно управлять пользователями через REST API.

📌 **Пример: создать пользователя через API**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '{
           "username": "john_doe",
           "email": "john@example.com",
           "enabled": true
         }'

```

📌 **Пример: задать пароль пользователю**

```bash
curl -X PUT "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/reset-password" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '{
           "type": "password",
           "value": "newpassword",
           "temporary": false
         }'

```

✅ **Результат:**  
Пользователь `john_doe` может войти с паролем `newpassword`.

---

### 🟢 **3. Через Keycloak Admin CLI**

Если у вас установлен **Keycloak CLI**, вы можете управлять пользователями через командную строку.

📌 **Пример: получить список пользователей**

```bash
kcadm.sh get users -r myrealm
```

📌 **Пример: создать пользователя**

```bash
kcadm.sh create users -r myrealm \
  -s username=john_doe -s enabled=true

```

✅ **Результат:**  
Пользователь `john_doe` создан.

---

## 🔹 **3. Как пользователи аутентифицируются в Keycloak?**

Keycloak поддерживает **несколько способов аутентификации**:

### ✅ **Стандартная аутентификация (логин + пароль)**

- Пользователь вводит **username / email + пароль**.
- Keycloak проверяет учётные данные и выдаёт `access_token`.

📌 **Пример запроса на логин (Authorization Code Flow)**:

```bash
curl -X POST "https://keycloak.example.com/auth/realms/myrealm/protocol/openid-connect/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=password" \
     -d "client_id=my-client" \
     -d "username=john_doe" \
     -d "password=mypassword"

```

✅ **Результат:**  
Выдаётся `access_token`, который можно использовать для работы с API.

---

### ✅ **SSO (Single Sign-On)**

- Позволяет пользователю **один раз войти в Keycloak** и получить доступ ко всем сервисам.
- Работает через **OAuth 2.0, OpenID Connect, SAML**.

📌 **Пример сценария:**

1. Пользователь заходит в `app1.example.com`, аутентифицируется в Keycloak.
2. Потом переходит в `app2.example.com` → уже аутентифицирован (SSO работает).

---

### ✅ **Многофакторная аутентификация (MFA)**

- **OTP (One-Time Password)** (Google Authenticator, FreeOTP).
- **WebAuthn (FIDO2)** (аппаратные ключи, Face ID, Touch ID).
- **SMS / Email-коды**.

📌 **Как включить MFA в Keycloak:**

1. Откройте **Authentication → Flows**.
2. Выберите **Browser Flow**.
3. Добавьте **OTP Form** перед логином.

✅ **Результат:**  
Пользователь вводит **пароль + OTP-код**.

---

### ✅ **Социальная аутентификация (Social Login)**

Keycloak поддерживает **Google, Facebook, GitHub, Microsoft и другие IdP**.

📌 **Как включить Social Login в Keycloak:**

1. Перейдите в **Identity Providers**.
2. Выберите, например, **Google**.
3. Укажите **Client ID / Secret** из Google API Console.

✅ **Результат:**  
Пользователь может войти через Google.

---

## 🔹 **4. Роли и Группы пользователей**

Keycloak позволяет управлять **доступом через роли и группы**.

### 🟢 **Роли (Roles)**

- **Realm Role** → Общая роль в рамках Realm.
- **Client Role** → Роль, специфичная для клиента (приложения).

📌 **Пример: Назначить роль пользователю через API**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/role-mappings/realm" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "admin" } ]'

```

✅ **Результат:**  
Пользователь `john_doe` теперь администратор.

---

### 🟢 **Группы (Groups)**

Группы помогают **объединять пользователей** и назначать им роли.

📌 **Пример:**

- Группа **"customers"** (все клиенты e-commerce).
- Группа **"managers"** (управляющие, имеют админ-доступ).

✅ Если пользователь состоит в группе `managers`, он автоматически получает роли **admin, manager**.

---

## 📊 **Краткое сравнение способов управления пользователями в Keycloak**

|**Метод**|**Как использовать**|**Пример**|
|---|---|---|
|**Admin Console**|Вручную через веб-интерфейс|Создание пользователей, назначение ролей|
|**REST API**|Программное управление|Создание пользователей, сброс пароля|
|**Keycloak CLI**|Через командную строку|Автоматизация, DevOps сценарии|

---

## 🎯 **Итог**

- **Пользователи управляются в Realm** → изолированные группы пользователей.
- Можно **создавать, удалять, обновлять** пользователей через **Admin Console, REST API, CLI**.
- Keycloak поддерживает **SSO, Social Login, MFA**.
- Контроль доступа осуществляется через **Roles и Groups**.