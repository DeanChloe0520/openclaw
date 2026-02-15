OpenClaw Telegram Bot 部署总结

遇到的问题和解决方案

问题 1：Webhook 模式配置失败

症状：



Telegram webhook 返回 404 Not Found

OpenClaw 无法接收 Telegram 消息

原因：

OpenClaw 的 webhook 监听器启动了，但没有正确注册 /telegram-webhook 路由

解决方案：

删除了 webhook 配置

调用 Telegram API 的 deleteWebhook 方法删除已注册的 webhook

切换到轮询模式（polling mode）



问题 2：OpenClaw 需要 Anthropic API Key

症状：




No API key found for provider "anthropic"  


用户只有 OpenAI API Key，没有 Anthropic API Key

OpenClaw 默认使用 Anthropic 作为 AI 提供商

原因：

OpenClaw 被设计为使用 Anthropic Claude 作为默认 AI 模型

环境变量无法覆盖硬编码的默认配置

解决方案：

修改源代码中的默认模型配置

编辑 /src/agents/defaults.ts 文件

将 DEFAULT_PROVIDER 从 "anthropic" 改为 "openai"

将 DEFAULT_MODEL 从 "claude-opus-4-6" 改为 "gpt-4o-mini"



问题 3：TypeScript 编译错误

症状：




error TS2367: This comparison appears to be unintentional because the types '"api_key"' and '"oauth"' have no overlap.  

原因：



在 /src/config/defaults.ts 中修改代码时，将 authMode 硬编码为字符串 "api_key"

后续代码仍然在检查 authMode === "oauth"，导致类型不匹配

解决方案：

修改第 365 行的代码

将 every: authMode === "oauth" ? "1h" : "30m", 改为 every: "30m",

这样避免了不必要的类型比较



问题 4：模型别名配置不生效

症状：



修改了 DEFAULT_MODEL_ALIASES 来使用 OpenAI

但 OpenClaw 仍然启动时使用 Anthropic 模型

原因：

模型别名只是别名映射，不是默认模型设置

OpenClaw 在启动时从 defaults.ts 中读取 DEFAULT_PROVIDER 和 DEFAULT_MODEL

这两个常量才是真正的默认值

解决方案：

直接修改 /src/agents/defaults.ts 中的常量值

这是 OpenClaw 启动时读取的真正默认配置



问题 5：Telegram 配对需要多次重试

症状：



首次发送消息时收到配对代码

批准后仍然需要重新发送消息才能正常工作

原因：

OpenClaw 的 Telegram 集成需要完整的初始化流程

配对完成后需要重新启动或重新连接

解决方案：

批准配对代码后，重新发送消息

OpenClaw 会在第二次消息时正常响应



部署流程总结



1. 初始部署 OpenClaw  
   ↓
2. 配置 Telegram Bot Token 和 Admin ID  
   ↓
3. 尝试 Webhook 模式 → 失败（404 错误）  
   ↓
4. 切换到轮询模式  
   ↓
5. 发现需要 Anthropic API Key  
   ↓
6. 修改源代码使用 OpenAI  
   ↓
7. 修复 TypeScript 编译错误  
   ↓
8. 重新部署  
   ↓
9. Telegram 配对和测试  
   ↓
10. ✅ 成功！Bot 正常工作  


关键修改文件

文件	修改内容
/src/agents/defaults.ts	改 DEFAULT_PROVIDER 和 DEFAULT_MODEL
/src/config/defaults.ts	改 heartbeat.every 的条件判断
/src/config/defaults.ts	改 DEFAULT_MODEL_ALIASES 为 OpenAI 模型
Zeabur 环境变量	添加 OPENAI_API_KEY


最终成果

✅ OpenClaw Telegram Bot 完全部署成功



使用 OpenAI gpt-4o-mini 作为 AI 模型

Telegram 轮询模式正常工作

Bot 能够接收消息并生成智能回复
