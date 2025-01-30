# wsl-
wsl配置clash连接代理
# deepseek R1

在 WSL 中使用 Windows 上 Clash 配置的代理需要以下步骤：

---

### **1. 允许 Clash 局域网连接**
- 打开 Clash 的 **「Allow LAN」** 选项（通常在系统托盘图标右键菜单中）。
- 确认 Clash 监听的端口（默认 HTTP 代理端口 `7890`，SOCKS5 端口 `7891`）。

---

### **2. 获取 Windows 主机的 IP 地址**
- **WSL2**：
  ```bash
  # 方法 1：通过 resolv.conf 获取（推荐）
  cat /etc/resolv.conf | grep nameserver | awk '{print $2}'

  # 方法 2：使用命令获取网关 IP
  ip route show default | awk '{print $3}'
  ```
  > 输出类似 `172.31.xxx.1`，这是 WSL2 中 Windows 主机的 IP。

- **WSL1**：直接使用 `127.0.0.1`（与 Windows 共享网络栈）。

---

### **3. 设置 WSL 代理环境变量**
在 WSL 终端中临时设置（仅当前会话生效）：
```bash
export http_proxy="http://<Windows_IP>:7890"
export https_proxy="http://<Windows_IP>:7890"
export ALL_PROXY="socks5://<Windows_IP>:7891"
```
替换 `<Windows_IP>` 为实际获取的 IP（如 `172.31.xxx.1`）。

---

### **4. 永久生效配置（可选）**
将环境变量添加到 Shell 配置文件（如 `~/.bashrc` 或 `~/.zshrc`）：
```bash
echo -e "\nexport http_proxy='http://<Windows_IP>:7890'" >> ~/.bashrc
echo "export https_proxy='http://<Windows_IP>:7890'" >> ~/.bashrc
echo "export ALL_PROXY='socks5://<Windows_IP>:7891'" >> ~/.bashrc
source ~/.bashrc
```

---

### **5. 配置 Windows 防火墙**
确保 Windows 防火墙允许 Clash 端口：
1. 打开 **控制面板 > Windows Defender 防火墙 > 高级设置**。
2. 新建入站规则：
   - 规则类型：端口
   - 端口号：`7890, 7891`
   - 允许连接。

---

### **6. 验证代理是否生效**
```bash
# 检查 HTTP 代理
curl -x http://<Windows_IP>:7890 http://ipinfo.io/ip

# 或直接使用环境变量
curl http://ipinfo.io/ip
```
如果返回的 IP 是 Clash 代理的 IP，则配置成功。

---

### **附加配置**
- **Git 代理**：
  ```bash
  git config --global http.proxy http://<Windows_IP>:7890
  git config --global https.proxy http://<Windows_IP>:7890
  ```
- **APT 代理**：
  创建 `/etc/apt/apt.conf.d/proxy.conf`：
  ```
  Acquire::http::Proxy "http://<Windows_IP>:7890";
  Acquire::https::Proxy "http://<Windows_IP>:7890";
  ```

---

### **常见问题**
1. **无法连接代理**：
   - 检查 Clash 的 `Allow LAN` 是否开启。
   - 确认防火墙放行端口。
   - 确保 WSL 中使用的 IP 正确（WSL2 需用主机 IP，WSL1 用 `127.0.0.1`）。

2. **部分应用不遵循代理**：
   - 手动为应用配置代理（如 `curl -x` 或应用自身的代理设置）。

--- 

完成以上步骤后，WSL 的网络流量将通过 Clash 代理转发。
