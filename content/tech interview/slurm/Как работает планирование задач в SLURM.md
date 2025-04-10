SLURM использует **планировщик (scheduler)** внутри `slurmctld`, который регулярно:

1. Проверяет доступные ресурсы (узлы, CPU, GPU, память)
    
2. Сортирует задачи по приоритету
    
3. Определяет, какие задачи можно запустить сейчас
    
4. Назначает задачи на подходящие ноды
``

🔁 Цикл планирования:
```bash
Новая задача → Очередь → Планировщик (приоритет + ресурсы) → Запуск на узле

```

Если ресурсы заняты — задача висит в `PD` (Pending).

---

## 🧩 Приоритет задачи: из чего складывается?

SLURM по умолчанию использует **модель Multi-Factor Priority** — приоритет задачи рассчитывается как сумма весов по разным факторам:

### 📊 Основные факторы приоритета:

|Фактор|Обозначение|Описание|
|---|---|---|
|`PriorityAge`|⏳|Чем дольше задача в очереди — тем выше приоритет|
|`PriorityFairShare`|⚖️|Кто давно ничего не запускал — тот выше в очереди|
|`PriorityJobSize`|📦|Большие задачи могут иметь выше приоритет|
|`PriorityPartition`|📌|Учитывается вес очереди (partition weight)|
|`PriorityQOS`|🎫|Уровень обслуживания (например, `premium`)|
|`PriorityTRES`|💻|Учитываются запрашиваемые ресурсы (TRES — Trackable Resources)|

---

### 🔧 Пример формулы (упрощённо):
```bash
priority = weight_age * age_score
         + weight_fairshare * fs_score
         + weight_partition * partition_score
         + ...

```
Все веса задаются в `slurm.conf`

🔒 Пример настроек в `slurm.conf`:
```bash
PriorityType=priority/multifactor
PriorityWeightAge=1000
PriorityWeightFairshare=20000
PriorityWeightPartition=1000
PriorityWeightJobSize=100
PriorityMaxAge=7-00:00:00

```