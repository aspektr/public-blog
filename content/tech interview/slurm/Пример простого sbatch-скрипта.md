```bash
#!/bin/bash
#SBATCH --job-name=bert-train
#SBATCH --output=logs/%x-%j.out
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=04:00:00

echo "Running on $(hostname)"
source ~/venv/bin/activate
python train.py --epochs 10 --batch-size 64

```

### 🔍 Объяснение ключевых параметров:

|Параметр|Значение|
|---|---|
|`#!/bin/bash`|Указывает, что скрипт исполняется в bash|
|`#SBATCH --job-name=bert-train`|Имя задачи, отображается в `squeue`, логах и UI|
|`#SBATCH --output=logs/%x-%j.out`|Путь к файлу вывода stdout/stderr (`%x` — имя, `%j` — job ID)|
|`#SBATCH --partition=gpu`|Выбор очереди, в которой запускать (например, GPU-ноды)|
|`#SBATCH --gres=gpu:1`|Запрос 1 GPU (generic resource)|
|`#SBATCH --cpus-per-task=4`|Кол-во CPU на задачу (важно при многопоточности)|
|`#SBATCH --mem=32G`|Запрашиваемая память|
|`#SBATCH --time=04:00:00`|Максимальное время выполнения — 4 часа|

---

### 🚀 Поведение скрипта:

- Задача попадёт в очередь `gpu`
    
- Как только освободится GPU-нода — начнётся выполнение
    
- Все логи пойдут в `logs/bert-train-<jobid>.out`
    
- После завершения — SLURM освободит ресурсы