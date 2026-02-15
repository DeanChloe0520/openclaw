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

硬盘（Volumes）挂载配置：

硬盘 ID	openclaw-state	
挂载目录	/root/.openclaw


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
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
部署 OpenClaw 项目修改的所有变量

1. 环境变量（Zeabur 中添加）

变量名	值	用途
OPENAI_API_KEY	sk-...	OpenAI API 认证
TELEGRAM_BOT_TOKEN	85595...	Telegram Bot 认证
TELEGRAM_ADMIN_ID	8062134750	Telegram 管理员 ID
NODE_ENV	production	Node.js 环境
OPENCLAW_GATEWAY_PASSWORD	Zs15600770520.	Gateway 密码
OPENCLAW_GATEWAY_TOKEN	JonesyBaby0520.	Gateway Token
PASSWORD	DY0W8OcKlI5vN73xV1F26SdkZzoqw9X4	服务密码
PORT	${WEB_PORT}	端口变量


2. 源代码变量修改

文件：/src/agents/defaults.ts

修改前：


typescript


export const DEFAULT_PROVIDER = "anthropic";  
export const DEFAULT_MODEL = "claude-opus-4-6";  

修改后：


typescript


export const DEFAULT_PROVIDER = "openai";  
export const DEFAULT_MODEL = "gpt-4o-mini";  


文件：/src/config/defaults.ts

修改 1 - DEFAULT_MODEL_ALIASES（第 15-26 行）

修改前：


typescript


const DEFAULT_MODEL_ALIASES: Readonly<Record<string, string>> = {  
  // Anthropic (pi-ai catalog uses "latest" ids without date suffix)  
  opus: "anthropic/claude-opus-4-6",  
  sonnet: "anthropic/claude-sonnet-4-5",  
  // OpenAI  
  gpt: "openai/gpt-5.2",  
  "gpt-mini": "openai/gpt-5-mini",  
  // Google Gemini (3.x are preview ids in the catalog)  
  gemini: "google/gemini-3-pro-preview",  
  "gemini-flash": "google/gemini-3-flash-preview",  
};  

修改后：


typescript


const DEFAULT_MODEL_ALIASES: Readonly<Record<string, string>> = {  
  // Anthropic (pi-ai catalog uses "latest" ids without date suffix)  
  opus: "openai/gpt-4o-mini",  
  sonnet: "openai/gpt-4o-mini",  
  // OpenAI  
  gpt: "openai/gpt-4o-mini",  
  "gpt-mini": "openai/gpt-4o-mini",  
  // Google Gemini (3.x are preview ids in the catalog)  
  gemini: "openai/gpt-4o-mini",  
  "gemini-flash": "openai/gpt-4o-mini",  
};  


修改 2 - applyContextPruningDefaults 函数（第 269-365 行）

修改前：


typescript


const authMode = resolveAnthropicDefaultAuthMode(cfg);  
if (!authMode) {  
  return cfg;  
}  

修改后：


typescript


//const authMode = resolveAnthropicDefaultAuthMode(cfg);  
//if (!authMode) {  
//  return cfg;  
//}  
const authMode = "api_key";  


修改 3 - heartbeat.every 条件（第 365 行）

修改前：


typescript


every: authMode === "oauth" ? "1h" : "30m",  

修改后：


typescript


every: "30m",  


3. 变量修改总结表

类别	变量/常量	修改前	修改后	原因
源代码	DEFAULT_PROVIDER	"anthropic"	"openai"	使用 OpenAI 作为默认 AI 提供商
源代码	DEFAULT_MODEL	"claude-opus-4-6"	"gpt-4o-mini"	使用 OpenAI 的 gpt-4o-mini 模型
源代码	DEFAULT_MODEL_ALIASES	多个 Anthropic 模型	全部改为 "openai/gpt-4o-mini"	统一使用 OpenAI 模型
源代码	authMode 赋值	函数调用	硬编码 "api_key"	跳过 Anthropic 认证检查
源代码	heartbeat.every	条件判断	硬编码 "30m"	避免 TypeScript 类型错误
环境变量	OPENAI_API_KEY	无	你的 API Key	OpenAI 认证
环境变量	TELEGRAM_BOT_TOKEN	无	Bot Token	Telegram 认证
环境变量	TELEGRAM_ADMIN_ID	无	8062134750	Telegram 管理员权限


关键要点

✅ 最重要的修改：

/src/agents/defaults.ts - 改 DEFAULT_PROVIDER 和 DEFAULT_MODEL

/src/config/defaults.ts - 改 DEFAULT_MODEL_ALIASES

Zeabur 环境变量 - 添加 OPENAI_API_KEY、TELEGRAM_BOT_TOKEN、TELEGRAM_ADMIN_ID

⚠️ 未生效的修改：

环境变量 OPENCLAW_AGENTS_DEFAULTS_MODEL_PRIMARY 等（OpenClaw 不读取这些）
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
OpenClaw 服务环境变量

完整环境变量列表


#	变量名	值	用途
1	OPENCLAW_GATEWAY_PASSWORD	Zs15600770520.	OpenClaw Gateway 密码
2	NODE_ENV	production	Node.js 环境（生产环境）
3	OPENAI_API_KEY	sk-...	OpenAI API 认证密钥
4	TELEGRAM_BOT_TOKEN	85595...	Telegram Bot 认证令牌
5	PORT	${WEB_PORT}	Web 服务端口（Zeabur 变量）
6	PASSWORD	DY0W8OcKlI5vN73xV1F26SdkZzoqw9X4	服务访问密码
7	OPENCLAW_GATEWAY_TOKEN	JonesyBaby0520.	OpenClaw Gateway 令牌
8	TELEGRAM_ADMIN_ID	8062134750	Telegram 管理员 ID
9	OPENCLAW_AGENTS_DEFAULTS_MODEL_PROVIDER	openai	AI 提供商（OpenAI）
10	OPENCLAW_AGENTS_DEFAULTS_MODEL_MODEL	gpt-4o-mini	AI 模型名称


环境变量分类

🔐 认证相关（4个）


OPENAI_API_KEY - OpenAI 认证

TELEGRAM_BOT_TOKEN - Telegram Bot 认证

OPENCLAW_GATEWAY_PASSWORD - Gateway 密码

OPENCLAW_GATEWAY_TOKEN - Gateway 令牌


🤖 AI 模型配置（2个）


OPENCLAW_AGENTS_DEFAULTS_MODEL_PROVIDER - 提供商：openai

OPENCLAW_AGENTS_DEFAULTS_MODEL_MODEL - 模型：gpt-4o-mini


📱 Telegram 配置（1个）


TELEGRAM_ADMIN_ID - 管理员 ID：8062134750


⚙️ 系统配置（3个）


NODE_ENV - 环境：production

PORT - 端口：${WEB_PORT}

PASSWORD - 服务密码



总计：10 个环境变量
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
在 OpenClaw 的系统提示词中添加 Web 抓取指导。
（原方式：在Dockerfile找到 COPY . . 这一行

# Modify system prompt to include web_fetch guidance    
RUN sed -i '/If a task is more complex or takes longer/a\    "## Web Fetching",\n    "When the user asks about current information, news, weather, stock prices, or anything requiring real-time data:",\n    "1. Use web_fetch to retrieve content from relevant websites",\n    "2. Extract the information needed from the fetched content",\n    "3. Provide the user with the most recent information",\n    "Do not say you cannot access the internet; instead, use web_fetch to get the information.",\n    "Always prefer using web_fetch for current/real-time information.",' src/agents/system-prompt.ts  

）

代码详解

bash


RUN sed -i '/If a task is more complex or takes longer/a\  
    "## Web Fetching",\n  
    "When the user asks about current information, news, weather, stock prices, or anything requiring real-time data:",\n  
    "1. Use web_fetch to retrieve content from relevant websites",\n  
    "2. Extract the information needed from the fetched content",\n  
    "3. Provide the user with the most recent information",\n  
    "Do not say you cannot access the internet; instead, use web_fetch to get the information.",\n  
    "Always prefer using web_fetch for current/real-time information.",' src/agents/system-prompt.ts  

命令分解

部分	说明
sed -i	使用 sed 命令直接修改文件（in-place edit）
/If a task is more complex or takes longer/	查找这行文本作为锚点
a\	在找到的行之后添加内容（append）
"## Web Fetching"	添加的新章节标题
src/agents/system-prompt.ts	要修改的文件


具体作用

修改前的系统提示词：



...  
If a task is more complex or takes longer...  
[其他内容]  

修改后的系统提示词：


...  
If a task is more complex or takes longer...  
## Web Fetching  
When the user asks about current information, news, weather, stock prices, or anything requiring real-time data:
1. Use web_fetch to retrieve content from relevant websites
2. Extract the information needed from the fetched content
3. Provide the user with the most recent information  
Do not say you cannot access the internet; instead, use web_fetch to get the information.  
Always prefer using web_fetch for current/real-time information.  
[其他内容]  


这段代码的目的

告诉 AI 模型（OpenClaw）：


✅ 有 Web 抓取能力 - 可以使用 web_fetch 工具

✅ 何时使用 - 当需要实时信息时

✅ 如何使用 - 具体的使用步骤

✅ 改变行为 - 不要说"我无法访问互联网"，而是使用 web_fetch



实际效果

用户问： "今天天气怎么样？"

修改前的行为：




AI: 我无法访问实时天气信息，因为我没有互联网访问权限。  

修改后的行为：




AI: 我来帮你查询最新的天气信息。  
[使用 web_fetch 获取天气数据]  
AI: 根据最新数据，今天的天气是...  


在你的 OpenClaw 项目中的应用

如果你想添加这个功能到你的 OpenClaw Bot，可以：



在 Dockerfile 中添加这行命令

重新部署服务

Telegram Bot 就会获得 Web 抓取能力


修改后的 Dockerfile 示例：

dockerfile


FROM [base image]  
...  
# 添加 Web 抓取指导到系统提示词  
RUN sed -i '/If a task is more complex or takes longer/a\  
    "## Web Fetching",\n  
    "When the user asks about current information, news, weather, stock prices, or anything requiring real-time data:",\n  
    "1. Use web_fetch to retrieve content from relevant websites",\n  
    "2. Extract the information needed from the fetched content",\n  
    "3. Provide the user with the most recent information",\n  
    "Do not say you cannot access the internet; instead, use web_fetch to get the information.",\n  
    "Always prefer using web_fetch for current/real-time information.",' src/agents/system-prompt.ts  
...  
