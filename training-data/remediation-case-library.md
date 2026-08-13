# 等级保护整改案例库

> 基于真实测评案例提取的整改方案，覆盖三级系统常见问题。
> 本库作为 AI Agent 的整改建议参考，提供具体技术方案和配置命令。

---

## 目录

- [1. 口令复杂度策略](#1-口令复杂度策略)
- [2. 登录失败锁定](#2-登录失败锁定)
- [3. 双因素认证](#3-双因素认证)
- [4. 三权分离](#4-三权分离)
- [5. SELinux 启用](#5-selinux-启用)
- [6. 数据存储完整性保护](#6-数据存储完整性保护)
- [7. 异地备份 + 恢复测试](#7-异地备份--恢复测试)
- [8. WAF 部署](#8-waf-部署)
- [9. 外包开发安全审查](#9-外包开发安全审查)
- [10. 可信验证](#10-可信验证)
- [11. 密码合规（商密认证）](#11-密码合规商密认证)
- [12. VPN网关商密改造](#12-VPN网关商密改造)

---

## 1. 口令复杂度策略

### 适用系统

- 服务器（Linux）
- 数据库（MySQL / PostgreSQL）
- 应用系统

### 具体方案

#### Linux 服务器（CentOS 7+）

```bash
# 安装 pwquality
yum install -y libpwquality

# 编辑 /etc/pam.d/system-auth 或 /etc/pam.d/password-auth
# 添加或修改以下行：
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= minlen=8 dcredit=-1 ucredit=-1 lcredit=-1 ocredit=-1

# 编辑 /etc/login.defs
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_MIN_LEN    8
PASS_WARN_AGE   7
```

#### MySQL 8.0+

```sql
INSTALL PLUGIN validate_password SONAME 'validate_password.so';
SET GLOBAL validate_password_policy = MEDIUM;
SET GLOBAL validate_password_length = 8;
SET GLOBAL validate_password_mixed_case_count = 1;
SET GLOBAL validate_password_number_count = 1;
SET GLOBAL validate_password_special_char_count = 1;

-- 持久化配置
-- 在 my.cnf 中添加：
-- [mysqld]
-- validate_password_policy=MEDIUM
-- validate_password_length=8
```

### 验证方法

```bash
# 服务器
chage -l username    # 检查口令有效期

# 数据库
SHOW VARIABLES LIKE 'validate_password%';
```

---

## 2. 登录失败锁定

### 适用系统

- Linux 服务器（SSH 登录）
- 数据库（远程登录）
- 应用系统（控制台登录）

### 具体方案

#### Linux - pam_faillock（CentOS 7+）

```bash
# 安装
yum install -y pam

# 编辑 /etc/pam.d/password-auth
# 在 auth 部分首行后添加：
auth        required      pam_faillock.so preauth audit silent deny=5 unlock_time=900
auth        [default=die] pam_faillock.so authfail audit deny=5 unlock_time=900

# 在 account 部分添加：
account     required      pam_faillock.so

# 同样的配置应用到 /etc/pam.d/system-auth 和 /etc/pam.d/sshd
```

#### MySQL - Connection Control 插件

```sql
INSTALL PLUGIN CONNECTION_CONTROL SONAME 'connection_control.so';
INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS SONAME 'connection_control.so';

SET GLOBAL connection_control_failed_connections_threshold = 5;
SET GLOBAL connection_control_min_connection_delay = 60000;  -- 毫秒
SET GLOBAL connection_control_max_connection_delay = 2147483647;

-- my.cnf 持久化
-- connection_control_failed_connections_threshold=5
```

#### 应用系统登录失败处理

```nginx
# Nginx 限流（辅助）
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

location /login {
    limit_req zone=login burst=1 nodelay;
    proxy_pass http://backend;
}
```

### 验证方法

```bash
# 服务器
faillock --user username --stats    # 查看失败记录
faillock --user username --reset    # 解锁

# 数据库
SHOW STATUS LIKE 'Connection_control_%';
```

---

## 3. 双因素认证

### 方案对比

| 方案 | 优点 | 缺点 | 成本 |
|------|------|------|:---:|
| **Google Authenticator PAM** | 免费、配置简单、支持TOTP | 无统一管理、密钥在手机上 | ¥0 |
| **RADIUS + OTP** | 统一管理、支持多种Token | 需要额外服务器、配置复杂 | ¥2k-10k |
| **硬件Token（如YubiKey）** | 安全性最高、无手机依赖 | 硬件成本、丢失处理 | ¥100-300/个 |
| **短信OTP** | 用户友好 | 短信被截获风险、运营商成本 | ¥0.05-0.1/条 |

### 快速方案：Google Authenticator（Linux）

```bash
# 服务器端
yum install -y google-authenticator

# 为每个用户执行一次
google-authenticator  # 按提示配置，扫描二维码

# 编辑 /etc/pam.d/sshd
# 添加：
auth required pam_google_authenticator.so

# 编辑 /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

### 验证方法

手机 TOTP 应用生成验证码 → SSH 登录 → 输入密码 → 输入验证码 → 成功登录

---

## 4. 三权分离

### 概念

| 角色 | 职责 | 权限范围 |
|------|------|---------|
| **系统管理员** | 系统配置、日常运维 | 配置修改权限，无审计日志查看权限 |
| **安全管理员** | 安全策略、安全审计 | 安全配置权限，无操作系统命令权限 |
| **审计管理员** | 日志审计、事件追溯 | 仅审计日志查看权限，无任何操作权限 |

### Linux 实现

```bash
# 创建三个用户组
groupadd sysadmin
groupadd secadmin
groupadd audadmin

# 创建用户
useradd -m -G sysadmin -s /bin/bash sysadmin
useradd -m -G secadmin -s /sbin/nologin secadmin
useradd -m -G audadmin -s /sbin/nologin audadmin

# sudo 权限配置（/etc/sudoers）
# 系统管理员
%sysadmin ALL=(ALL) ALL

# 安全管理员 - 仅安全相关命令
%secadmin ALL=(ALL) /usr/bin/passwd, /usr/sbin/useradd, /usr/sbin/usermod, /usr/sbin/groupadd, /bin/chmod, /bin/chown

# 审计管理员 - 仅审计日志
%audadmin ALL=(ALL) /usr/bin/cat /var/log/secure, /usr/bin/cat /var/log/audit/audit.log, /usr/bin/journalctl, /sbin/ausearch, /sbin/aureport
```

### 应用系统实现

```text
1. 应用程序中实现 RBAC 角色模型
2. 数据库层面：
   - 创建3个独立账户（app_admin / app_sec / app_aud）
   - 配置不同权限级别
   - 确保系统管理员不能查看审计日志
   - 确保审计管理员不能修改任何数据
```

---

## 5. SELinux 启用

### 适用系统

CentOS/RHEL 7+

### 实施步骤

```bash
# 1. 检查当前状态
getenforce

# 2. 从 Disabled 切换到 Permissive（先不强制，记录违规）
# 编辑 /etc/selinux/config
SELINUX=permissive
# 重启
reboot

# 3. 运行至少72小时，检查日志
ausearch -m avc -ts recent | audit2why

# 4. 修复常见违规
# 例如 Nginx 端口绑定：
semanage port -a -t http_port_t -p tcp 8080

# 5. 确认无违规后切为 enforcing
# 编辑 /etc/selinux/config
SELINUX=enforcing

# 6. 验证
getenforce  # 输出 Enforcing
```

### 常见问题

| 违规 | 解决命令 |
|------|---------|
| Web服务器端口绑定 | `semanage port -a -t http_port_t -p tcp <port>` |
| 数据库端口 | `semanage port -a -t mysqld_port_t -p tcp 3306` |
| 自定义脚本 | `chcon -t bin_t /path/to/script` |
| 文件访问 | `restorecon -Rv /var/www/html` |

---

## 6. 数据存储完整性保护

### 方案对比

| 方案 | 技术 | 适用场景 | 性能影响 |
|------|------|---------|:-------:|
| **HMAC** | 共享密钥哈希 | 数据库表行校验 | 低（~5%） |
| **数字签名** | 非对称密钥签名 | 日志、审计记录 | 中（~15%） |
| **文件完整性检查** | AIDE / Tripwire | 系统文件、配置 | 低（定期扫描） |
| **数据库审计日志签名** | MySQL Enterprise Audit | 企业版专用 | 中 |

### HMAC 实现（推荐 - 数据库表级别）

```sql
-- 在需要完整性保护的表中添加校验列
ALTER TABLE user_accounts ADD COLUMN data_hash CHAR(64);

-- 使用触发器自动计算（示例）
DELIMITER //
CREATE TRIGGER trg_user_hash_insert BEFORE INSERT ON user_accounts
FOR EACH ROW
BEGIN
    SET NEW.data_hash = SHA2(
        CONCAT(NEW.user_id, NEW.username, NEW.role, NEW.department),
        256
    );
END;//
CREATE TRIGGER trg_user_hash_update BEFORE UPDATE ON user_accounts
FOR EACH ROW
BEGIN
    SET NEW.data_hash = SHA2(
        CONCAT(NEW.user_id, NEW.username, NEW.role, NEW.department),
        256
    );
END;//
DELIMITER ;

-- 验证完整性
SELECT *, 
    CASE WHEN data_hash = SHA2(CONCAT(user_id, username, role, department), 256) 
         THEN 'INTACT' ELSE 'TAMPERED' 
    END AS integrity_check
FROM user_accounts;
```

> ⚠️ 国家标准要求应使用**国家商用密码算法**（SM3哈希、SM2签名）而非 SHA-256。上例仅作演示，实际应替换为 SM3/SM2。

---

## 7. 异地备份 + 恢复测试

### 备份架构要求

```text
[生产系统] ──本地备份──→ [本地存储（NAS/SAN）]    ← 常规恢复
    │                                                 每日/每周
    ├──异步复制──→ [同城灾备中心]                   ← RPO < 1小时
    │
    └──远程复制──→ [异地灾备中心]                   ← RPO < 24小时
                                                         RTO < 72小时
```

### 实施步骤

```bash
# 1. 数据库异地备份（使用 mysqldump + rsync）
mysqldump -u root -p --all-databases --single-transaction --routines --triggers > /backup/db_$(date +%Y%m%d).sql
rsync -avz /backup/ backup@remote:/backup/

# 2. 自动化脚本示例（crontab）
# 0 2 * * * /usr/local/bin/backup_to_remote.sh

# 3. 恢复测试脚本
# !/bin/bash
# 恢复测试 - 每季度执行一次
BACKUP_FILE="/backup/db_$(date +%Y%m%d).sql"
TEST_DB="test_restore_$(date +%Y%m%d)"

mysql -e "CREATE DATABASE $TEST_DB"
mysql $TEST_DB < $BACKUP_FILE
if [ $? -eq 0 ]; then
    echo "[$(date)] 恢复测试通过：$BACKUP_FILE"
    mysql -e "DROP DATABASE $TEST_DB"
else
    echo "[$(date)] 恢复测试失败：$BACKUP_FILE"
fi
```

### 恢复测试频率建议

| 系统等级 | 全量恢复测试 | 增量恢复测试 |
|:-------:|:----------:|:----------:|
| 二级 | 每半年 | 每月 |
| 三级 | 每季度 | 每月 |
| 四级 | 每月 | 每周 |

---

## 8. WAF 部署

### 方案对比

| 方案 | 部署方式 | 优点 | 缺点 |
|------|---------|------|------|
| **云平台WAF** | 云服务商提供 | 免运维、规则库自动更新、高可用 | 成本较高、依赖厂商 |
| **ModSecurity** | 自建+OWASP规则 | 免费、灵活定制、社区支持 | 需自运维、性能需调优 |
| **商业WAF** | 硬件/软件 | 专业支持、低误报率 | 价格高 |

### 快速方案：ModSecurity + Nginx

```nginx
# 安装（参考 modsecurity 官方文档）
# Nginx 配置
server {
    location / {
        ModSecurityEnabled on;
        ModSecurityConfig /etc/nginx/modsecurity/modsecurity.conf;
    }
}

# modsecurity.conf 基本配置
SecRuleEngine On
SecRequestBodyAccess On
SecResponseBodyAccess Off
SecDefaultAction "phase:1,deny,log,status:403"

# OWASP CRS 规则（标准规则集）
Include /etc/nginx/modsecurity/crs/crs-setup.conf
Include /etc/nginx/modsecurity/crs/rules/*.conf
```

---

## 9. 外包开发安全审查

### 审查流程

```text
交付物接收
    │
    ├── 1. 恶意代码检测（SAST工具扫一次）
    │      ├── 源码扫描：SonarQube / Fortify / Checkmarx
    │      └── 二进制扫描：VirusTotal / YARA规则
    │
    ├── 2. 源码安全审计
    │      ├── OWASP Top 10 合规检查
    │      ├── 硬编码密码/密钥检查
    │      ├── SQL注入/ XSS/CSRF 专项检查
    │      └── 第三方组件漏洞检查（SCA）
    │
    ├── 3. 验收测试
    │      ├── 功能测试（对照需求文档）
    │      ├── 渗透测试（至少SAST+DAST）
    │      └── 性能测试（不低于合同要求的150%）
    │
    └── 4. 安全检查验收单
           ├── 安全测试报告
           ├── 漏洞修复确认
           └── 三方组件清单+版本+许可证
```

### 最小可执行方案（预算有限时）

```bash
# 1. Graves 快速扫描（免费）
grype /path/to/artifact  # 容器镜像/文件系统漏洞扫描
trivy filesystem /path/to/code

# 2. 硬编码密钥检测
git secrets --scan-history
trufflehog filesystem /path/to/code

# 3. 基本的 SAST（免费）
semgrep --config=auto /path/to/code
```

---

## 10. 可信验证

### 技术路线

| 方案 | 适用平台 | 成本 | 复杂度 |
|------|---------|:---:|:-----:|
| TPM 2.0 + Linux IMA | 支持TPM的服务器 | ¥200-500/块TPM | 🔴 高 |
| 可信固件（UEFI SecureBoot） | 所有新服务器 | 免费（BIOS功能） | 🟢 低 |
| OpenCAS（完整性度量架构） | Linux | 免费 | 🟡 中 |
| 等保专用可信机 | Windows/Linux | ¥3000-10000 | 🟢 低 |

### 最低可行方案：UEFI SecureBoot + IMA

```bash
# 1. 启用 UEFI SecureBoot（BIOS设置）
# 重启 → 进入 BIOS → Secure Boot → Enabled

# 2. 确认已启用
mokutil --sb-state  # 输出 SecureBoot enabled

# 3. 启用 IMA（Integrity Measurement Architecture）
# 编辑 /etc/default/grub
GRUB_CMDLINE_LINUX="... ima_tcb ima_appraise=fix"

# 4. 设置 IMA 策略
echo "appraise func=BPRM_CHECK" > /sys/kernel/security/ima/policy
echo "measure func=FILE_MMAP mask=MAY_EXEC" >> /sys/kernel/security/ima/policy

# 5. 验证
cat /sys/kernel/security/ima/ascii_runtime_measurements
```

---

## 11. 密码合规（商密认证）

### 国家密码管理局要求

根据《密码法》第26条和GB/T 39786-2021：

- **三级系统**：身份鉴别、通信传输、数据存储三个环节应使用**国家商用密码算法**
- 合规算法：**SM2**（公钥密码）/ **SM3**（哈希）/ **SM4**（对称加密）
- 密码产品应通过国家商密认证（产品列表可在国家密码管理局官网查询）

### 常见改造场景

| 场景 | 原技术 | 合规方案 |
|------|-------|---------|
| SSH 服务 | RSA 2048 | 替换为 SM2 证书（或OpenSSH + openssl SM2 patch） |
| HTTPS | RSA/SHA-256 | 替换为 SM2/SM3 证书（国密SSL） |
| 数据库加密 | AES-256 | 替换为 SM4 加密 |
| 数据完整性 | SHA-256 | 替换为 SM3 哈希 |
| 数字签名 | RSA签名 | 替换为 SM2 签名 |
| VPN | IPsec/IKE | 替换为国密SSL VPN |

### 快速自查命令

```bash
# 检查 SSH 使用算法
ssh -Q cipher | grep -i sm   # 如无输出则未支持国密
ssh -Q kex | grep -i sm      # 如无输出则密钥交换未支持国密

# 检查 OpenSSL 版本（1.1.1+ 开始支持国密）
openssl version
openssl ciphers | grep -i sm

# 检查 HTTPS
openssl s_client -connect example.com:443 -tls1_2 2>/dev/null | grep "Cipher"
```

---

## 12. VPN网关商密改造

### 替代方案

| 方案 | 说明 | 成本 |
|------|------|:---:|
| **深信服SSLVPN商密版** | 主流商密VPN含证书 | ¥10k-50k |
| **华为SecoManager + 商密模块** | 运营商级 | ¥30k-80k |
| **天融信SSLVPN-C** | 成熟替代方案 | ¥8k-30k |
| **IPsec VPN + SM2证书** | 自建方案 | ¥3k-10k |

### 最低可行方案（更换证书）

```text
1. 联系当前VPN厂商确认是否支持商密证书
2. 如支持：直接向CA机构申请SM2证书替换现有RSA证书
3. 如不支持：采购支持商密的VPN产品替代
4. 替换后验证：全功能测试+性能测试+兼容性测试
5. 更新安全策略文档
```

---

## 附录：整改优先级速查表

| 整改项 | 难度 | 时间 | 成本 | 影响范围 | 优先级 |
|-------|:---:|:---:|:---:|:-------:|:-----:|
| 口令复杂度 | 🟢 低 | 30min/台 | ¥0 | 服务器+数据库 | 🥇 |
| 登录失败锁定 | 🟢 低 | 30min/台 | ¥0 | 服务器+数据库 | 🥇 |
| 恢复测试 | 🟢 低 | 1天/系统 | ¥0 | 全部系统 | 🥇 |
| SELinux | 🟡 中 | 1-3天/台 | ¥0 | 服务器 | 🥇 |
| 管理IP白名单 | 🟢 低 | 15min/台 | ¥0 | 服务器+数据库 | 🥇 |
| 双因素认证 | 🟡 中 | 2-4周 | ¥-¥10k | 全部系统 | 🥈 |
| 三权分离 | 🟢 低 | 1-2周 | ¥0 | 操作系统+应用 | 🥈 |
| 异地备份 | 🟡 中 | 1-3月 | ¥1k-100k/年 | 全部系统 | 🥈 |
| 存储完整性 | 🟡 中 | 1-2周 | ¥0-10k | 全部数据 | 🥈 |
| 外包安全审查 | 🟡 中 | 1-2周/系统 | ¥-50k | 6个应用 | 🥈 |
| WAF | 🔴 高 | 2-3周 | ¥10k-100k | 区域边界 | 🥉 |
| VPN商密改造 | 🔴 高 | 1-2月 | ¥8k-80k | 通信链路 | 🥉 |
| 可信验证 | 🔴 高 | 2-3月 | ¥5k-100k | 通信+边界+计算 | 🥉 |
| 服务商评审机制 | 🟡 中 | 1月 | ¥0 | 管理流程 | 🥉 |
| 全面安全检查制度 | 🟡 中 | 1-2月 | ¥0 | 管理流程 | 🥉 |
