# GladeKit MCP Bridge（Unity 2022.3 本地化版插件）

**GladeKit MCP Bridge** 的本地化维护版（`com.gladekit.mcp-bridge` v0.4.3，锁定 Unity 2022.3 LTS）。

> ⚠️ 此仓库**不跟踪官方 Unity 6 版本**——官方新版接口差异大，我们保持 2022.3 本地改，不随意升级。

## 这是什么

把 Unity 编辑器变成 HTTP + MCP 服务的桥插件：

- 本地 HTTP 服务（**8765 优先，被占自动回退 9000**——桥内置机制，无需配置）；
- 222+ 编辑器工具暴露给外部客户端（AI 助手 / 脚本 / 任何 HTTP 调用方）；
- 支持场景、UGUI、预制体、材质、光照、物理、动画、地形等全类目工具（工具实现位于 `Editor/Tools/`）。

## 安装（二选一）

**方式 A：UPM（推荐，新项目）** —— `Package Manager → ＋ → Add package from git URL`：

```
https://github.com/PdxGame/unity-gladekit-bridge.git
```

依赖（`com.unity.inputsystem`、`com.unity.ai.navigation`）自动安装。
> 若你的项目是 Assets/Plugins 直放方式，UPM 安装会与旧副本冲突——**先删掉旧目录** `Assets/Plugins/com.gladekit.mcp-bridge Unity2022.3LTS/` 再装。

**方式 B：拷贝（现项目迁移/离线）** —— 把本仓库整个目录放进 `Assets/Plugins/com.gladekit.mcp-bridge Unity2022.3LTS/`（保留 .meta，引用不丢）。

## 端口机制（内置，客户端无需切换逻辑）

```csharp
// Editor/Bridge/UnityBridgeServer.cs
/// Runs on localhost:8765 (automatically falls back to 9000 if the port is taken)
private static readonly int[] CandidatePorts = { 8765, 9000 };
```

- 8765 优先；被占 → 自动回退 9000；两个都忙 → 轮换尝试；
- 运行期端口：`UnityBridgeServer.CurrentPort`（桥窗口显示 `Running on localhost:{port}`）；
- 客户端只要"探测两个端口，哪个活着用哪个"即可（见下）。

## 端口探测脚本（外部调用方用）

```powershell
# 探测 8765 / 9000 哪个活着
foreach ($p in @(8765, 9000)) {
  try { $h = Invoke-RestMethod -Uri "http://localhost:$p/api/health" -TimeoutSec 3; if ($h.status -eq 'ok') { Write-Output "可用端口: $p" } } catch { }
}
```

常用端点：

| 端点 | 用途 |
|---|---|
| `GET /api/health` | 存活探测 |
| `GET /api/compilation/status` | 编译状态 |
| `POST /api/tools/execute` | 单工具（UTF-8 JSON body） |
| `POST /api/batch` | 批量（≤50 调用） |

## 维护约定

- 只改本仓库（本地化版），不合并官方新版；
- 改动后同步回项目 `Assets/Plugins/`（或反之，保持两处一致）；
- 工具 222+ 全在 `Editor/Tools/Implementations/**`，加工具 = 加一个 ITool 实现并注册（`Editor/Tools/ToolRegistry.cs`）。

## License

见仓库 `LICENSE`（MIT）。