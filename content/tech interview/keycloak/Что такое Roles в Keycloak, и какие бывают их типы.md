**Roles (Роли)** в Keycloak — это механизм **управления доступом** пользователей к ресурсам.  
Роли позволяют определять, какие действия может выполнять пользователь в системе.

Например, администратор (`admin`) может управлять пользователями, а обычный пользователь (`user`) может только просматривать контент.

---

## 📌 **1. Какие бывают типы ролей в Keycloak?**

В Keycloak существует **два типа ролей**:

|**Тип роли**|**Описание**|
|---|---|
|**Realm Roles**|Глобальные роли, применяемые ко всему Realm.|
|**Client Roles**|Специфичные роли для конкретного клиента (приложения).|

---

## 🔹 **2. Realm Roles (Роли на уровне Realm)**

- Realm Role **не привязана к конкретному приложению**.
- Может быть назначена **пользователям или группам**.
- Используется для контроля **общих прав доступа** в рамках Realm.

📌 **Примеры Realm Roles:**

- `admin` — Администратор системы.
- `manager` — Менеджер, который управляет заказами.
- `user` — Обычный пользователь.

**📌 Как создать Realm Role через Admin Console:**

1. Перейдите в **Realm Settings → Roles**.
2. Нажмите **Create Role**.
3. Введите имя роли (`admin`).
4. Нажмите **Save**.

✅ Теперь можно назначать эту роль пользователям.

📌 **Как назначить Realm Role пользователю через API:**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/role-mappings/realm" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "admin" } ]'

```

✅ Теперь пользователь `john_doe` получил роль `admin`.

---

## 🔹 **3. Client Roles (Роли на уровне клиента)**

- Применяются **только к конкретному клиенту** (например, `ecom-api` или `crm-app`).
- Ограничивают доступ внутри конкретного приложения или сервиса.

📌 **Примеры Client Roles:**

- В клиенте `ecom-api` может быть роль `order_manager`.
- В клиенте `crm-app` может быть роль `crm_user`.

**📌 Как создать Client Role через Admin Console:**

1. Перейдите в **Clients → Выберите клиент → Roles**.
2. Нажмите **Create Role**.
3. Введите имя роли (`order_manager`).
4. Нажмите **Save**.

📌 **Как назначить Client Role пользователю через API:**

```curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/role-mappings/realm" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "admin" } ]'

```

✅ Теперь пользователь `john_doe` имеет роль `order_manager` в клиенте `ecom-api`.

---

## 🔹 **4. Различие между Realm Roles и Client Roles**

|**Характеристика**|**Realm Role**|**Client Role**|
|---|---|---|
|**Где применяется?**|На уровне всего Realm|Только в конкретном клиенте (приложении)|
|**Где создаётся?**|В `Realm Settings → Roles`|В `Clients → <Client> → Roles`|
|**Примеры**|`admin`, `manager`, `user`|`order_manager`, `crm_user`|

---

## 🔹 **5. Как назначить роли пользователям?**

### ✅ **1. Через Admin Console**

1. Перейдите в **Users → Выберите пользователя → Role Mappings**.
2. Выберите нужную роль и нажмите **Assign**.

### ✅ **2. Через REST API**

📌 **Назначить пользователю роль `admin` (Realm Role):**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/role-mappings/realm" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "admin" } ]'

```

📌 **Назначить пользователю роль `order_manager` в клиенте `ecom-api`:**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/role-mappings/clients/<client_id>" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "order_manager" } ]'

```

✅ Теперь пользователь получил нужные роли.

---

## 🔹 **6. Как проверять роли в приложении?**

Когда пользователь аутентифицируется, Keycloak выдаёт **JWT-токен**, который содержит роли.  
Приложение может проверять роли при каждом запросе.

📌 **Пример декодированного Access Token (JWT):**

```json
{
  "realm_access": {
    "roles": ["admin", "manager"]
  },
  "resource_access": {
    "ecom-api": {
      "roles": ["order_manager"]
    }
  }
}

```

📌 **Пример проверки ролей в Spring Security (Java)**

```java
@PreAuthorize("hasRole('admin')")
@GetMapping("/admin")
public String adminEndpoint() {
    return "Доступ только для администраторов";
}

```

📌 **Пример проверки ролей в Express.js (Node.js)**

```javascript
function checkRole(role) {
    return (req, res, next) => {
        if (!req.user.roles.includes(role)) {
            return res.status(403).json({ message: "Доступ запрещён" });
        }
        next();
    };
}
app.get("/admin", checkRole("admin"), (req, res) => {
    res.send("Только для администраторов");
});

```

✅ **Результат:**  
Если у пользователя есть роль `admin`, он сможет выполнить запрос.

---

## 🎯 **Итог**

- **Realm Roles** → Глобальные роли, действующие на весь Realm.
- **Client Roles** → Роли, специфичные для определённого приложения.
- Роли можно **назначать вручную через Admin Console или API**.
- **Приложения могут проверять роли** в JWT-токене.