# Task4. Ролевой доступ к Kubernetes (RBAC)

## Принципы

- **Разделение по оргструктуре.** Каждый домен PropDevelopment работает в своём неймспейсе: `sales` — группа сервисов продаж, `housing` — сервисы ЖКУ, включая «Умный дом», `finance` — финансы. Команда видит и меняет только свой неймспейс.
- **Секреты.** Читать секреты во всём кластере может только служба ИБ. В своём неймспейсе секретами управляет только DevOps-инженер команды.
- **Наблюдатели не видят логи подов.**
- **Изменять RBAC** (роли и привязки) может только администратор платформы.
- **Группы пользователей.** Роли привязаны к группам, а не к людям.

## Роли

| Роль | Права роли | Группы пользователей |
| --- | --- | --- |
| `cluster-viewer` (ClusterRole) | Только чтение (`get`, `list`, `watch`) во всём кластере. **Без** `secrets` и `pods/log` | `cluster-viewers` — бизнес-аналитики, владельцы продуктов, поддержка; `platform-operators` |
| `cluster-operator` (ClusterRole) | Настройка кластера: управление неймспейсами, нодами (cordon, drain), PV, StorageClass. **Без** секретов, RBAC и рабочих нагрузок приложений | `platform-operators` — инженеры по эксплуатации платформы |
| `security-auditor` (ClusterRole) | Чтение секретов во всех неймспейсах, чтение ролей и привязок (Role, ClusterRole, RoleBinding, ClusterRoleBinding), чтение логов подов. Изменять ничего не может | `security-auditors` — специалист по ИБ |
| `cluster-admin` (встроенная ClusterRole) | Полный доступ, включая изменение RBAC. Аварийный доступ (break-glass): действия журналируются, используется по заявке | `platform-admins` — руководитель платформы |
| `developer` (Role в `sales`, `housing`, `finance`) | В своём неймспейсе: создание, изменение и удаление Deployment; чтение логов и событий. **Без** секретов, `exec` и RBAC | `sales-developers`, `housing-developers`, `finance-developers` — разработчики продуктовых команд |
| `devops` (Role в `sales`, `housing`, `finance`) | Всё, что у `developer`, плюс полное управление `secrets`, `pods/exec`, `pods/portforward`, NetworkPolicy в своём неймспейсе | `sales-devops`, `housing-devops`, `finance-devops` — DevOps-инженеры продуктовых команд |

## Пользователи (тестовые)

| Пользователь | Группы | Что ожидаем |
| --- | --- | --- |
| `anna` | `cluster-viewers` | Видит ресурсы всего кластера, но не секреты и не логи |
| `petr` | `platform-operators` | Создаёт неймспейсы и StorageClass, не видит секреты |
| `olga` | `security-auditors` | Читает секреты во всех неймспейсах, ничего не меняет |
| `ivan` | `sales-developers` | Деплоит в `sales`, ничего не может в `housing`, секреты не видит |
| `maria` | `housing-devops` | Управляет `housing`, включая секреты; в `sales` доступа нет |

