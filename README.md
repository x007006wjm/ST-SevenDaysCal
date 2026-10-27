# ST-SevenDaysCal · 构画（host API 分支）

这是 [atonal519/ST-SevenDaysCal](https://github.com/atonal519/ST-SevenDaysCal) v3.10.2 的派生版本，
只加了一件事：**支持「跟随酒馆当前 API」**，也就是不再需要在构画里另填 Base URL + API Key，
而是直接使用酒馆当前生效的聊天补全 API。

## 为什么需要

Vertex AI **服务账号（Full 模式）** 这类渠道，凭证只存在于酒馆服务端（由酒馆签 JWT、换 access token），
浏览器端拿不到也不该拿。构画原来的「自填 URL + Key」模型对这种渠道无解。

## 实现方式

host 模式下请求体交给酒馆自己的构造器生成：

```js
getContext().ChatCompletionService.presetToGeneratePayload({}, {}, {
    messages, model, stream, temperature,
});
```

所以 `vertexai_auth_mode` / `vertexai_region` / `secret_id` / `use_sysprompt` 等字段与主聊天
逐字段同源，构画不接触任何凭证。构画只覆盖 `messages / model / stream / temperature`，
并主动剥离 `json_schema` / `response_format` / `n`（连接继承，输出合同不继承）。

## 安装（TauriTavern）

扩展 → Install extension，填：

```
https://github.com/x007006wjm/ST-SevenDaysCal
```

仓库必须是 public（TauriTavern 只支持匿名 HTTP(S) Git）。装完 → 设置 → API →
选择「跟随酒馆当前 API」。

## 安装（SillyTavern 桌面端）

把本仓库整目录放到 `public/scripts/extensions/third-party/ST-SevenDaysCal/`，
或直接用上面同样的 Git URL 安装。

## 从源码重建

```bash
npm install
npm run build      # 产出 gouhua-app.js
```

## 与上游的差异

- `runtime/settings.js`：`apiMode`（custom/host）、`ready` 语义、预设携带 `mode`
- `api/client.js`：host 模式请求体构造、连接/鉴权字段保护、剔除参数先于附加参数
- `api/diagnostics.js`、`api/sse.js`：`host-unavailable` 诊断码、更完整的受保护路由键
- `index.js`：设置面板连接来源切换、状态条、预设通用化
- `business/*`：约 19 处 URL/Key 门禁改为 ready 感知（保持 host-free 单测架构）
- `gouhua-app.js`：重新构建的产物

测试：`node --test` 416 例，407 通过 / 9 失败，与改造前基线一致（无新增失败）。

## 许可

上游未附许可证文件；本派生版本沿用上游的一切约定，仅供个人使用。
