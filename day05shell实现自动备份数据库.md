
# Shell 脚本 + Crontab 实现数据库自动备份

> 利用 **Shell 脚本**编写备份逻辑，配合 **Crontab** 定时调度，实现对 MariaDB / MySQL 数据库的**周期性自动备份**，并自动清理过期备份文件。

---

## 一、整体思路

| 环节 | 作用 |
| :--- | :--- |
| `mysqldump` | 把数据库导出为 SQL 文本 |
| `gzip` | 通过管道压缩，节省磁盘空间 |
| 备份文件命名 | 用时间戳保证文件名唯一，避免覆盖 |
| `if [ -s ]` 校验 | 判断备份文件是否非空，确认成败 |
| `find ... -delete` | 删除超过保留天数的旧备份，控制容量 |
| `crontab` | 让脚本按计划自动执行 |

---

## 二、编写 Shell 脚本

新建脚本文件（例如 `backup_db.sh`）：

```bash
#!/bin/bash
# 告诉系统使用 Bash 解释器来执行这个脚本

BACKUP_DIR="/var/backups/mariadb"   # 备份文件的存放路径，改位置只改这一行
DB_NAME="backup"                    # 要备份的数据库名
RETENTION_DAYS=7                    # 旧备份保留天数，超过则自动删除
DATE=$(date +%Y%m%d_%H%M%S)         # 当前时间戳，如 20261009_020000，用于命名防止冲突

mkdir -p "$BACKUP_DIR"              # 备份目录不存在则创建，已存在也不报错（-p）

FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"   # 拼接出完整备份文件路径

# 核心：mysqldump 导出 -> | 管道交给 gzip 压缩 -> > 写入到目标路径
mysqldump --single-transaction "$DB_NAME" | gzip > "$FILE"

# 校验备份是否成功（-s 判断文件大小是否大于 0）
if [ -s "$FILE" ]; then
    echo "$(date) backup success: $FILE"
else
    echo "$(date) backup failed: $FILE"
    exit 1                          # 返回非 0 状态码，让调度系统知道本次失败
fi

# 清理过期备份：查找修改时间超过 RETENTION_DAYS 天的文件并删除
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +"$RETENTION_DAYS" -delete
```

### 关键说明

- **`--single-transaction`**：以一致性快照导出，适用于 InnoDB，备份时不锁表，避免影响线上写入。
- **变量一律加双引号**：`"$FILE"`、`"$BACKUP_DIR"` 等，防止路径含空格时被错误拆分。
- **`-mtime +7`**：表示"修改时间在 7 天之前"，`+` 号是关键，表示"超过"。
- **清理逻辑放在 `if...fi` 之外**：无论备份成功与否都应执行清理（原稿把 `find` 误缩进在 `else` 分支内，已修正）。

---

## 三、赋予脚本执行权限

```bash
sudo chmod u+x backup_db.sh
```

> `u+x` 表示仅给文件属主增加可执行权限。`sudo` 需小写，Linux 命令区分大小写。

---

## 四、配置 Crontab 定时任务

编辑当前用户的计划任务：

```bash
crontab -e
```

追加一行（**务必使用脚本的绝对路径**，否则 cron 的工作目录为家目录，会找不到脚本）：

```cron
* * * * * /bin/bash /home/lzy/backup_db.sh
```

测试通过后，改为每天凌晨 2 点执行一次：

```cron
0 2 * * * /bin/bash /home/lzy/backup_db.sh
```

### Crontab 五个时间字段

```text
┌───────── 分钟 (0-59)
│ ┌───────── 小时 (0-23)
│ │ ┌───────── 日   (1-31)
│ │ │ ┌───────── 月   (1-12)
│ │ │ │ ┌───────── 星期 (0-7，0 和 7 都表示周日)
│ │ │ │ │
* * * * *   /bin/bash /绝对路径/backup_db.sh
```

| 表达式 | 含义 |
| :--- | :--- |
| `* * * * *` | 每分钟执行一次（仅用于测试） |
| `0 2 * * *` | 每天凌晨 2:00 执行 |
| `*/5 * * * *` | 每 5 分钟执行一次 |
| `0 3 * * 1` | 每周一凌晨 3:00 执行 |

> **权限提示**：若脚本内部用到 `sudo` 命令，后台执行时无法弹出密码框，会导致失败。应把任务加到 root 的计划表中（`sudo crontab -e`），或去掉脚本内的 `sudo`。

---

## 五、验证与排错

```bash
# 查看当前用户的定时任务是否生效
crontab -l

# 查看 root 用户的定时任务（用 sudo crontab -e 添加的任务在这里看）
sudo crontab -l

# 查看备份目录，确认是否生成了 .sql.gz 文件
ls -lh /var/backups/mariadb

# 查看 cron 服务日志，排查执行记录（CentOS/RHEL）
grep CRON /var/log/cron

# 手动跑一次脚本，观察输出是否正常
/bin/bash /home/lzy/backup_db.sh
```

---

## 六、完整可复制版

```bash
#!/bin/bash
# 数据库自动备份脚本：mysqldump + gzip 备份，并清理过期文件
BACKUP_DIR="/var/backups/mariadb"
DB_NAME="backup"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

mysqldump --single-transaction "$DB_NAME" | gzip > "$FILE"

if [ -s "$FILE" ]; then
    echo "$(date) backup success: $FILE"
else
    echo "$(date) backup failed: $FILE"
    exit 1
fi

find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +"$RETENTION_DAYS" -delete
```

```cron
# 部署步骤
sudo chmod u+x /home/lzy/backup_db.sh
# crontab -e 中追加（每天凌晨 2 点）
0 2 * * * /bin/bash /home/lzy/backup_db.sh
```

---

## 七、要点回顾

1. **变量加引号、留空格**：避免路径被拆分、命令粘连报错。
2. **校验用 `[ -s ]`**：确认备份文件非空即成功。
3. **清理逻辑独立于成败分支**：保证过期文件一定会被回收。
4. **Crontab 用绝对路径**：调度环境无当前目录概念。
5. **想清楚执行身份**：含 `sudo` 的任务交给 root 的计划表，避免后台取不到密码。
