**Groups (Группы)** в Keycloak — это механизм, который позволяет **объединять пользователей** и автоматически назначать им **роли** и **атрибуты**.

## 📌 **1. Зачем нужны группы?**

Группы позволяют **упрощать управление доступом** и ролями, особенно когда в системе много пользователей.

### **Без групп**:

- Администратор вручную назначает **каждому пользователю** роли (`admin`, `manager`, `user`).
- Это неудобно и сложно администрировать.

### **С группами**:

- Пользователь добавляется в группу (например, `Managers`).
- Группа автоматически наследует все **роли**.
- Пользователь получает доступ ко всем привилегиям, связанным с группой.

📌 **Пример:**

- Группа **"Managers"** → включает роли `admin`, `manager`, `view_reports`.
- Если пользователь `john_doe` добавлен в **Managers**, он автоматически получает **все роли этой группы**.

---

## 🔹 **2. Как создать группу в Keycloak?**

### ✅ **Через Admin Console**

1. Перейдите в **Users → Groups**.
2. Нажмите **Create Group**.
3. Введите имя группы (`managers`).
4. Нажмите **Save**.

### ✅ **Через REST API**

📌 **Создать группу `managers` через API:**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/groups" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '{ "name": "managers" }'

```

✅ Теперь группа `managers` создана.

---

## 🔹 **3. Как назначить роли группе?**

Чтобы **группа автоматически передавала пользователям определённые роли**, нужно добавить их в группу.

### ✅ **Через Admin Console**

1. Перейдите в **Groups → Выберите группу → Role Mappings**.
2. Выберите роли (`admin`, `manager`).
3. Нажмите **Assign**.

### ✅ **Через REST API**

📌 **Назначить группе `managers` роль `admin`:**

```bash
curl -X POST "https://keycloak.example.com/auth/admin/realms/myrealm/groups/<group_id>/role-mappings/realm" \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '[ { "name": "admin" } ]'

```

✅ Теперь все пользователи из **managers** получат роль `admin`.

---

## 🔹 **4. Как добавить пользователя в группу?**

### ✅ **Через Admin Console**

1. Перейдите в **Users → Выберите пользователя → Groups**.
2. Выберите группу и нажмите **Join**.

### ✅ **Через REST API**

📌 **Добавить пользователя `john_doe` в `managers`:**

```bash
curl -X PUT "https://keycloak.example.com/auth/admin/realms/myrealm/users/<user_id>/groups/<group_id>" \
     -H "Authorization: Bearer <admin_token>"

```

✅ Теперь `john_doe` автоматически получает все роли группы.

---

## 🔹 **5. Как связаны группы и роли?**

|**Действие**|**Как работает**|
|---|---|
|**Группа содержит роли**|Если группа `Managers` имеет роли `admin`, `manager`, то все пользователи в этой группе получают эти роли.|
|**Пользователь наследует роли от группы**|Если `john_doe` состоит в группе `Managers`, он автоматически получает `admin`, `manager`.|
|**Группа может включать вложенные группы**|Например, группа `IT Team` может содержать `Developers` и `Admins`.|
|**Можно комбинировать роли и группы**|Пользователь может входить сразу в несколько групп.|

📌 **Пример структуры групп:**

```
Company Employees
├── Managers (admin, manager)
│   ├── Sales Team (sales_manager)
│   ├── HR Team (hr_manager)
├── Developers (dev, read_code)
│   ├── Backend Team (java_dev, python_dev)
│   ├── Frontend Team (react_dev, angular_dev)

```

- Если пользователь входит в `Sales Team`, он автоматически получает **все роли** `Managers` + `sales_manager`.

---

## 🔹 **6. Как проверить роли пользователя в токене?**

Когда пользователь входит в систему, Keycloak **включает роли** в JWT-токен.

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
  },
  "groups": ["Managers", "Sales Team"]
}

```

✅ Теперь API может проверять **как роли, так и группы**.

---

## 🔹 **7. Как использовать группы в приложении?**

### ✅ **В Spring Boot (Java)**

```java
@PreAuthorize("hasRole('admin')")
@GetMapping("/admin")
public String adminEndpoint() {
    return "Только для администраторов";
}

```

### ✅ **В Express.js (Node.js)**

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
Пользователь сможет выполнить запрос, если он состоит в группе `Managers` и у него есть роль `admin`.

---

## 🎯 **Итог**

- **Groups (Группы)** позволяют объединять пользователей и автоматически назначать им **роли**.
- Пользователь получает **все роли, связанные с его группой**.
- Группы можно **создавать, назначать роли и добавлять пользователей через Admin Console или API**.
- JWT-токен **содержит информацию** о группах и ролях.