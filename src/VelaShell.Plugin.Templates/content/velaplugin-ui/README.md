# VelaPlugin1

VelaShell 插件(带一个 Avalonia 面板)。插件 id:`PLUGIN_ID`。

## 三步跑起来

```bash
dotnet tool install -g VelaShell.Plugin.Cli   # 只需装一次
dotnet build
vela-plugin dev init                          # 配好 IDE 启动配置
```

然后在 IDE 里按 **F5**(启动配置名 `VelaShell`)。它做的事:

- 从 `~/.velashell/host.json` 找到本机安装的 VelaShell —— 这份文件由 VelaShell 每次启动时自己写,
  所以不必手工填安装路径(**前提是本机至少启动过一次 VelaShell**);
- 以调试器附着的方式启动宿主,并用 `--dev-root` 把本工程的构建输出挂进去,插件在管理页带 **DEV** 角标;
- 用 `--data-root ~/.velashell-dev` 起一个**独立数据根的调试实例**,于是你日常那份 VelaShell
  可以一直开着(共用数据根会撞上单实例保护,直接退出);
- 隔离插件(`hostMode: isolated`)加上 `--wait-debugger PLUGIN_ID`,插件进程会在装载程序集之前
  挂起等你附加,pid 显示在管理页上、也落在 `~/.velashell/logs/plugin-host-PLUGIN_ID.pid`。

`vela-plugin doctor` 一次性体检:宿主版本、清单兼容性、构建产物是否干净、启动配置是否可用。

## 改了代码之后

```bash
dotnet build
```

回到 VelaShell 的插件管理页,在本插件那一行点 **重新加载** —— 不必重启宿主。
(开发期插件从影子副本装载,所以宿主运行时不会锁住 `bin`,随时可以重编。)

想更省事:`vela-plugin dev init --watch`,重编后自动重载。

## 出包

```bash
dotnet build -c Release -t:PackVpx
# → bin/vpx/PLUGIN_ID-0.1.0.vpx

# 带签名(密钥用 `vela-plugin keygen` 生成,别提交进仓库)
dotnet build -c Release -t:PackVpx -p:VelaSigningKey=/path/to/key.pem
```

`.vpx` 在 VelaShell 的插件管理页"安装 .vpx…"一键装上。发布到插件商店见
<https://github.com/VelaShellLabs/velashell-docs/blob/main/zh/templates/publishing.md>。

## 要点

- `plugin.json` 的 `id` 发布后**不要再改**:它同时是命令 id 前缀与插件私有数据/机密的命名空间。
- 本工程针对宿主使用的 Avalonia `AVALONIA_VERSION` 编译 AXAML,不随插件分发 Avalonia。
  改投其它宿主版本时,更新工程中的 `VelaAvaloniaVersion`;新建项目也可用 `--avaloniaVersion` 指定。
- `VelaShell.PluginSdk.dll` 与 `Avalonia*.dll` 永远不进插件目录:装载器强制共享宿主那一套。

完整开发指南:<https://github.com/VelaShellLabs/velashell-docs/blob/main/zh/templates/dev-guide.md>
命令行工具手册:<https://github.com/VelaShellLabs/velashell-docs/blob/main/zh/cli/cli.md>
