# Supabase-Keepalive

最简单的自动保活 Supabase 项目，防止因不活跃而被暂停。

## 功能

- ✅ 使用 GitHub Actions 定时执行
- ✅ 每天自动运行 2 次（UTC 0:00 和 12:00）
- ✅ 支持手动触发
- ✅ 完全免费

## 配置步骤

### 1. 在 Supabase 创建保活表

登录 Supabase Dashboard，进入 SQL Editor，执行：

```sql
-- 创建保活表
CREATE TABLE IF NOT EXISTS keepalive (
  id SERIAL PRIMARY KEY,
  pinged_at TIMESTAMP DEFAULT NOW()
);

-- 创建索引
CREATE INDEX IF NOT EXISTS idx_keepalive_pinged_at ON keepalive(pinged_at);

-- 自动清理函数（只保留最近 100 条记录）
CREATE OR REPLACE FUNCTION cleanup_keepalive()
RETURNS void AS $$
BEGIN
  DELETE FROM keepalive
  WHERE id NOT IN (
    SELECT id FROM keepalive
    ORDER BY pinged_at DESC
    LIMIT 100
  );
END;
$$ LANGUAGE plpgsql;
```


### 2. 配置 GitHub Secrets
可直接fork项目 https://github.com/ongwu/supabase-keepalive/fork

fork后 ，在仓库设置中添加以下 Secrets：

```
进入仓库 → Settings → Secrets and variables → Actions
点击 New repository secret
添加以下两个 secrets：
SUPABASE_URL	你的 Supabase 项目 URL（例如：https://xxx.supabase.co）
SUPABASE_KEY	你的 Supabase Service Role Key（以 sb_secret_ 开头）
```
<img width="1660" height="875" alt="image" src="https://github.com/user-attachments/assets/71057109-4098-430c-a664-07be975ecd90" />

### 3. 测试运行
```
进入 Actions 标签
选择 Supabase Keepalive workflow
点击 Run workflow 按钮手动触发
```
<img width="1745" height="663" alt="image" src="https://github.com/user-attachments/assets/697d1412-6c57-400a-b4ef-93d8f4750c90" />

查看运行日志确认成功,成功可在Supabase仪表盘查看Requests：
<img width="1602" height="636" alt="image" src="https://github.com/user-attachments/assets/ebd2dade-661e-4580-abc4-25fb7574bd0c" />


### （可选）修改运行频率
编辑 .github/workflows/keepalive.yml 中的 cron 表达式。

### （可选）查看运行历史
进入 Actions 标签可以查看所有运行记录和日志。
