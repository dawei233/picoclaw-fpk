# PicoClaw for 飞牛 fnOS（.fpk）

为飞牛 NAS 用户打包官方 [PicoClaw](https://picoclaw.io) 最新 Linux x86_64 版本，免 Docker、原始生运行。

> 官方唯一站点：[picoclaw.io](https://picoclaw.io)｜上游仓库：[sipeed/picoclaw](https://github.com/sipeed/picoclaw)

## 为什么不用飞牛应用商店里的旧版

飞牛官方应用商店里的 PicoClaw 旧版通常落后 GitHub release 数月。本仓库的 GitHub Actions **每次 build 都拉 GitHub 最新 release**，始终与上游同步。

## 安装到飞牛 NAS

1. 点 [Actions](../../actions) 跑一次最新的 build，下载产物 `picoclaw-<version>.fpk`
2. 把它传到飞牛 NAS（任一可写目录都行）
3. 飞牛桌面 → **应用中心** → 右上角 **设置** → **手动安装应用** → 选 `.fpk` 文件安装
4. 安装完成后桌面上会出现 **PicoClaw 控制台** 图标，点击在浏览器中打开 `http://<NAS-IP>:18800`
5. 进入配置（应用中心 → PicoClaw → 配置）编辑 `config.json`：
   - 替换 `model_list[0].api_key` 为你的 LLM API key
   - 在 `channel_list` 加 Telegram / Discord / 飞书 / QQ 等渠道
6. 应用中心 → PicoClaw → 重启，控制台可用了

> 数据保留：升级或重新安装不会丢，`config.json` 和会话记录都在飞牛托管的 `${TRIM_PKGHOME}` 与 `${TRIM_PKGETC}` 目录里。

## 升级

重新跑一次 Actions，下载新的 `.fpk` 到飞牛再装一次即可。`install_callback` 不会覆盖你已修改的 `config.json`。

## 架构说明

```
fnOS 应用包: picoclaw-<version>.fpk
├── ${TRIM_APPDEST}                ← cmd 实际可执行根（= app/）
│   ├── bin/
│   │   ├── picoclaw              ← 主程序（CLI / onboard / agent）
│   │   ├── picoclaw-launcher     ← Web 控制台（监听 18800）
│   │   └── picoclaw-launcher-tui ← 终端控制台（可选，本包未注册桌面入口）
│   ├── default-config.json       ← 首次启动自动复制到 ${TRIM_PKGETC}/
│   └── cmd/                      ← 8 个生命周期脚本（main + 7 hook）
├── ${TRIM_PKGETC}/config.json    ← 用户最终改这个
├── ${TRIM_PKGHOME}/workspace     ← 会话 / 记忆 / 缓存
├── ${TRIM_PKGVAR}/app.log        ← 主进程日志
└── ${TRIM_PKGTMP}/picoclaw.pid   ← 进程 PID
```

## 本地构建（可选）

```bash
# 装 fnpack 1.2.3
curl -fsSL -o fnpack https://static2.fnnas.com/fnpack/fnpack-1.2.3-linux-amd64
chmod +x fnpack

# 拉 PicoClaw Linux x86_64（也可手动指定版本，参见 .github/workflows/build.yml）
TAG=$(curl -fsSL https://api.github.com/repos/sipeed/picoclaw/releases/latest | jq -r '.tag_name')
mkdir -p picoclaw/app/bin
curl -fsSL -o picoclaw.tar.gz \
  "https://github.com/sipeed/picoclaw/releases/download/${TAG}/picoclaw_Linux_x86_64.tar.gz"
tar -xzf picoclaw.tar.gz -C picoclaw/app/bin --strip-components=1
chmod +x picoclaw/app/bin/*

# 注入 version
VERSION="${TAG#v}"
sed -i "s/__VERSION__/${VERSION}/" picoclaw/manifest

# fnpack build
./fnpack build picoclaw
# -> picoclaw.fpk
```

## 已知限制

- **仅 x86_64**：fnOS 第三方应用当前不支持 ARM（飞牛官方策略）
- **官方警示**：PicoClaw 处于 v1.0 之前的快速迭代阶段，不要用于生产关键业务
- **配置 API key 不走 wizard**：安装时让你直接编辑 config.json，不在向导里弹窗（向导可以再加，见 `picoclaw/wizard/config`）

## 故障排查

| 现象 | 处理 |
|------|------|
| 桌面图标点了没反应 | SSH 进 NAS：`bash ${TRIM_APPDEST}/cmd/main status` 看进程；`tail -f ${TRIM_PKGVAR}/app.log` 看日志 |
| 装机报错版本不兼容 | 飞牛系统版本低于 1.1.3100，请先升级 fnOS |
| icon 没显示 | 桌面 ctrl+shift+R 强刷；或重装 `.fpk` |
| 需要重置配置 | 删 `${TRIM_PKGETC}/config.json`，重启应用会自动重置为默认 |

## License

本仓库脚本与配置：MIT（见 `LICENSE`）。打包的二进制的版权与许可属上游 [PicoClaw](https://github.com/sipeed/picoclaw) 项目作者所有。
