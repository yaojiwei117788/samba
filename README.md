# Ubuntu 匿名 Samba 共享配置全指南
---

# Samba 服务配置指南  
## （针对 `/mnt/usb` 目录匿名访问）

### 1. 配置环境信息
- **用户名**：yaojiwei  
- **共享目录路径**：`/mnt/usb`  
- **目标客户端**：Windows 访问（通过 `\\10.10.10.130`）  

---

## 2. 操作步骤

### 步骤 1：更新系统并安装 Samba

sudo apt update
sudo apt install samba-common samba


### 步骤 2：配置共享目录权限

# 设置目录基础权限（755）
sudo chmod 755 /mnt/usb

# 修改目录所有者为当前用户（yaojiwei 用户所在组需为 yao）
sudo chown yao:yao /mnt/usb

# 开放全局写入权限（注意安全风险！）
sudo chmod 777 /mnt/usb


### 步骤 3：配置 Samba 共享  
编辑 `smb.conf` 文件：

sudo nano /etc/samba/smb.conf

在文件末尾添加以下内容：

[shared]
   comment = Anonymous Share
   path = /mnt/usb         # 共享的物理路径
   browseable = yes        # 允许浏览
   writable = yes          # 允许写入
   guest ok = yes          # 启用匿名访问
   read only = no          # 取消只读（如果需要可写）
   create mask = 0755      # 新建文件权限
   directory mask = 0755   # 新建目录权限


### 步骤 4：重启 Samba 服务

sudo systemctl restart smbd


---

## 3. 客户端访问方法  
在 **Windows** 系统中，按以下方式访问共享：
- 按 `Win + R` 打开运行对话框  
- 输入 `\\10.10.10.130\shared`（直接访问 `[shared]` 共享）或  
  `\\10.10.10.130` 查看所有共享目录  

---

## 4. 注意事项与常见问题

### 安全性提示
- **匿名访问风险**：通过 `guest ok = yes` 和 `chmod 777` 允许所有人直接读写，建议仅在私有局域网使用。
- 如果需限制权限，可将目录权限改为 `755` 并指定用户组：
  
  sudo chmod 755 /mnt/usb  
  

### 防火墙设置（如已启用）
若无法连接，请检查防火墙规则：

# 开放 SMB 端口（445）示例（使用 ufw）
sudo ufw allow 445/tcp
sudo ufw reload


### 共享不可见或访问被拒？
1. **权限问题**：确认目录权限为 `755` 或 `777`：
     
   ls -ld /mnt/usb
   
2. **配置文件语法错误**：检查 `smb.conf` 无拼写错误后重启服务：
   
   sudo systemctl restart smbd
   
3. **日志排查**：查看 Samba 日志：
     
   tail -f /var/log/syslog | grep smbd
   

---

## 5. 可选进阶配置建议
### 方案1：使用用户名密码认证（更安全）
- 若需取消匿名访问，可删除 `guest ok = yes` 并添加用户：
    
  sudo smbpasswd -a yaojiwei  
  
- 客户端连接时需输入用户名和密码。

### 方案2：限制共享目录权限

[shared]
   # 允许特定组访问（例如仅 yao 组）
   valid users = @yao
   write list = @yao


---

## 附录：配置文件关键参数说明  
| 参数          | 作用                                                                 |
|---------------|----------------------------------------------------------------------|
| `path`        | 指定共享的本地目录路径                                               |
| `guest ok`    | 是否允许匿名访问（yes/no）                                           |
| `writable`    | 允许写入权限                                                         |
| `create mask` | 新建文件权限（此处设置为 0755）                                      |

---
