# 飞牛NAS Home Assistant 集成

基于 [xiaochao99/fn_nas](https://github.com/xiaochao99/fn_nas) 二次开发，通过 SSH 连接飞牛 NAS（fnOS），在 Home Assistant 中监控和管理 NAS 系统。

## 功能

- **系统监控**：CPU、内存、网络、温度、风扇转速
- **磁盘管理**：硬盘健康状态（S.M.A.R.T.）、温度、通电时间
- **ZFS 存储池**：容量、健康、碎片率、Scrub 状态
- **Docker 管理**：容器列表查看、启停控制
- **虚拟机管理**：VM 列表查看、启停控制
- **UPS 监控**：电量、负载、电压、剩余时间
- **远程控制**：重启、关机、WOL 唤醒

## 项目结构

```
custom_components/fn_nas/
├── __init__.py          # 组件入口，初始化协调器和管理器
├── manifest.json        # 组件元信息（版本、作者、依赖）
├── config_flow.py       # UI 配置流程（输入 SSH 信息）
├── const.py             # 常量定义（平台列表、配置键、图标）
├── coordinator.py       # 数据协调器（30 秒轮询 NAS API）
├── sensor.py            # 传感器实体（CPU、内存、磁盘、UPS 等）
├── binary_sensor.py     # 二进制传感器（网络在线、磁盘健康）
├── switch.py            # 开关实体（Docker 容器、VM 启停）
├── button.py            # 按钮实体（重启、关机、WOL）
├── system_manager.py    # 系统操作（SSH 命令执行、温度数据）
├── disk_manager.py      # 磁盘管理（列表、S.M.A.R.T.、挂起）
├── docker_manager.py    # Docker 管理（容器列表、启停）
├── vm_manager.py        # 虚拟机管理（列表、启停）
├── ups_manager.py       # UPS 管理（状态、电量、电压）
└── translations/
    └── zh-Hans.json     # 中文翻译
```

## 架构说明

```
┌──────────────────────────────────────────┐
│              FlynasCoordinator            │
│          (DataUpdateCoordinator)          │
│           每 30 秒通过 SSH 轮询            │
├──────────────────────────────────────────┤
│  SystemManager │ DiskManager │ UPSManager │
│  DockerManager │ VMManager   │            │
├──────────────────────────────────────────┤
│  sensor.py  │ binary_sensor.py            │
│  switch.py  │ button.py                   │
└──────────────────────────────────────────┘
```

1. **Coordinator**：核心数据协调器，通过 SSH 连接 NAS，定时执行系统命令获取数据
2. **Manager 层**：封装各类 SSH 命令（`nas-tool`、`smartctl`、`zpool`、`docker`、`virsh` 等）
3. **Entity 层**：将 Manager 返回的数据映射为 HA 实体

## 安装指南

### 方式一：HACS 安装（推荐）

1. 确保已安装 [HACS](https://hacs.xyz/)
2. HACS → 集成 → 右上角菜单 → 自定义仓库
3. 填入仓库地址 `https://github.com/Aidan1777/fn_nas`，类型选择「集成」
4. 搜索「飞牛NAS」→ 下载
5. 重启 Home Assistant

### 方式二：手动安装

```bash
cd /path/to/homeassistant/config/custom_components
git clone https://github.com/Aidan1777/fn_nas.git
```

重启 Home Assistant。

### 配置

1. HA 界面 → 设置 → 设备与服务 → 添加集成
2. 搜索「飞牛NAS」
3. 填写 SSH 连接信息：
   - **主机地址**：NAS 的 IP 地址
   - **端口**：SSH 端口（默认 22）
   - **用户名**：SSH 用户名
   - **密码**：SSH 密码
   - **Root 密码**：用于执行系统命令（如与 SSH 密码相同可留空）
4. 提交后等待自动发现设备实体

### 配置选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| 扫描间隔 | 数据刷新频率（秒） | 60 |
| UPS 扫描间隔 | UPS 数据刷新频率（秒） | 30 |
| 启用 Docker 监控 | 是否监控 Docker 容器 | 关闭 |
| 忽略磁盘 | 忽略指定磁盘设备名 | 无 |

## 版本历史

- **1.0.0** — 基于上游 `xiaochao99/fn_nas` 二次开发，更新为个人仓库信息

## 许可证

基于上游项目，继承其开源协议。
