# SRUN_SZU

面向深圳大学粤海校区的深澜 (`Srun`) 认证自动登录方案。本仓库集成了上游开源项目 [`srun_main_source`](https://github.com/BH4ME/srun_main_source) 的源码压缩包、面向 ARMv8 的预编译二进制以及示例定时任务脚本，方便在路由器或其它嵌入式环境上快速部署。

## 仓库内容

- `srun-master.zip`：上游源码快照，便于本地编译或定制。
- `srun_arm64`：已在 WSL2 环境下交叉编译好的 ARMv8 可执行文件，可直接部署到支持 `aarch64` 的路由器（如 OpenWrt）。
- `config.json`：示例配置文件，用于存放登录服务器地址与账号信息。
- `定时一分钟启动一次.txt`：在路由器中通过 `crontab` 每分钟执行一次登录脚本的示例。
- `LICENSE`：上游遵循的 GPL-3.0 开源许可。

## 快速开始

### 1. 准备配置

1. 将仓库中的 `config.json` 上传到目标设备。
2. 按实际情况修改以下字段：
   - `server`：深澜认证服务器地址，例如 `https://net.szu.edu.cn`。
   - `acid`、`strict_bind`、`users`：根据校园网要求调整；多个账号可放在 `users` 数组中。
3. 出于安全考虑，建议将 `config.json` 的权限限制为仅管理员可读，例如在 OpenWrt 中执行 `chmod 600 /etc/srun/config.json`。

### 2. 使用预编译二进制

1. 将 `srun_arm64` 复制到路由器（示例路径 `/usr/bin/srun`），赋予执行权限：
   ```sh
   chmod +x /usr/bin/srun
   ```
2. 测试登录：
   ```sh
   /usr/bin/srun login -c /etc/srun/config.json
   ```
3. 如需要登出可执行：
   ```sh
   /usr/bin/srun logout -c /etc/srun/config.json
   ```

### 3. WSL2 下交叉编译 ARMv8 二进制

如需获得最新源码或启用额外特性（例如 TLS 支持），可按以下步骤自行编译：

1. 在 Windows 上安装 WSL2（推荐 Ubuntu），打开 WSL 终端。
2. 安装编译依赖：
   ```sh
   sudo apt update
   sudo apt install -y build-essential pkg-config libssl-dev musl-tools
   ```
3. 安装 Rust 工具链并添加目标架构：
   ```sh
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source "$HOME/.cargo/env"
   rustup target add aarch64-unknown-linux-musl
   ```
4. 解压 `srun-master.zip` 并进入源码目录：
   ```sh
   unzip srun-master.zip
   cd srun-master
   ```
5. 编译 ARMv8 静态链接版本（可通过 `--features tls` 启用 HTTPS）：
   ```sh
   cargo build --release --target aarch64-unknown-linux-musl --features tls
   ```
6. 可执行文件位于 `target/aarch64-unknown-linux-musl/release/srun`，将其复制到目标设备即可。

> 提示：若目标设备使用 `glibc`，也可以编译 `aarch64-unknown-linux-gnu` 版本。

## 部署定时任务

1. 将 `定时一分钟启动一次.txt` 中的命令写入路由器的 `crontab`，例如：
   ```sh
   * * * * * /usr/bin/srun login -c /etc/srun/config.json >/tmp/srun.log 2>&1
   ```
2. 根据需要调整执行频率、日志路径与脚本位置。
3. 确保 `cron` 服务已启用并运行：
   ```sh
   /etc/init.d/cron enable
   /etc/init.d/cron restart
   ```

## 常见问题

- **登录失败提示密码错误**：确认账号是否需要追加运营商后缀（如 `@cmcc`），可通过抓包或咨询网管确认。
- **提示证书错误**：针对 HTTPS 认证，请使用启用 `tls` 特性编译的二进制。
- **多拨需求**：在 `config.json` 中配置多个账号，程序会依次尝试登录。
- **权限不足**：确保二进制文件和配置文件位于正确路径且具备执行/读取权限。

## 致谢

-  [zu1k](https://github.com/zu1k) 对 `srun` 工具的开发。
- 本仓库仅对深圳大学的常用配置与部署方式进行整理。

## 许可证

项目遵循 GPL-3.0 许可证，详情参阅 `LICENSE`。
