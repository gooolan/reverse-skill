# 2026-10-10 受保护 Android 样本的离线分析

## 场景与范围

用户授权本地 APK 离线静态分析和所需 jadx/apktool 安装。目标标识、业务类名、域名、密钥、密码和文件路径已省略。auth_basis=user_authorized_local_sample；network_profile=offline；asset_types=[android_apk]；lead_role=lead；specialists=[]。

## 执行链与可复用方法

1. 通过 master-route、case-init、case-guard 确认 R1 和离线范围。
2. ZIP/DEX 清单发现极小壳 DEX、受保护数据和 Ijiami native 库。高熵与不存在 DEX magic 都只作为辅助证据。
3. 从工具索引解析实际路径；在已授权范围安装 jadx 1.5.6 / apktool 3.0.3。
4. jadx 首次因默认配置目录超出写入范围失败，用 JADX_CONFIG_DIR / JADX_CACHE_DIR 指向 case 内目录；对提取 DEX 反编译成功。
5. apktool 成功导出资源和 smali，但特殊 Manifest 顶层头使解码失败。按实际 headerSize 和 string-pool 字段独立解析原二进制 XML，结合资源表映射进行交叉验证。
6. 仅标准化顶层 XML 头仍有 string-pool 异常，记录失败后停止同类修补，不把分析副本称作有效 APK。
7. 对可读 ELF 提取导入/导出和精确地址；对受保护 ELF 的贫乏符号明确注明“有限可见性”。
8. 将已确认的配置与未经验证的漏洞候选分别记入 Findings，保存报告、流程图、证据指纹和时间线。

## Evidence / Finding / Path 摘要

| E-id | severity | status | 可复用命令模式 | 作用 |
|---|---|---|---|---|
| E-004 | info | observed | Python ELF64 DT_NEEDED/dynsym 解析 | 给出可读库导入和 JNI 地址，保护库不得作负向能力判断 |
| E-011 | info | observed | 二进制 XML 独立解析 | 从资源引用确认网络信任配置 |
| E-020 | info | observed | JADX 与 smali 调用交叉引用 | 分清组件工厂路径与壳 Application 路径 |

核心 Finding：平台网络策略放宽是可确认的配置事实，业务流量泄露或 pinning 绕过仍需额外证据。path_type=callflow；模式：Manifest 入口 → 壳库加载 → native 加载器 → 真实 Application，native 内部处理保留未知。

## 踩坑与限制

| 问题 | 处理 |
|---|---|
| 工具安装在受限环境不能写入 Homebrew | 使用已授权安装的环境审批机制；不执行权限重置命令 |
| 默认 jadx 用户配置目录不可写 | 使用 case 内专用配置及缓存目录 |
| 仅修补 XML 顶层头不足 | 保留失败证据，改用按格式字段解析和资源交叉验证 |
| 壳 .so 静态导入少 | 标记受保护，不宣称无能力 |
| 内嵌 RSA 公钥 | 区分公开 license key、业务签名密钥和私钥 |
| 有 SOCKS 导出却未恢复调用者 | 只确认接口，不宣称服务正在监听；结合产品配置检查 |

本次未动态运行、未访问目标域名、未脱壳、未获得业务签名算法。现有 R1 路由适用，无需新增路由。本文仅分享通用方法与证据边界；原始样本、完整报告和原始证据保留在本地。
