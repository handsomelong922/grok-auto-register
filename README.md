# Grok 辅助注册助手 (Windows)

> **免责声明：本项目仅供学习和研究浏览器自动化技术使用，请勿用于任何违反相关平台服务条款或法律法规的用途。使用本工具所产生的一切后果由使用者自行承担，作者不承担任何责任。**

基于 Playwright + Chrome CDP 的 Grok (x.ai) 账号自动注册工具。

## 文件说明

| 文件 | 作用 |
|------|------|
| `assisted_register_windows.py` | 主脚本。通过 Chrome DevTools Protocol (CDP) 控制浏览器，自动完成 Grok 注册流程：打开注册页 → 填邮箱 → 输入验证码 → 填写密码和姓名 → 处理 Turnstile 人机验证 → 提交注册。支持多线程并发。注册成功后将账号信息（邮箱、密码、SSO cookie）写入 `result_grok/` 和 `result_sso/` 目录。 |
| `email_utils.py` | 邮箱工具模块。调用 [Mail.tm](https://mail.tm) API 创建一次性临时邮箱，并轮询收件箱获取 xAI 发送的验证码。 |

## 环境要求

- Windows 系统
- Python 3.8+
- Google Chrome 或 Microsoft Edge (Chromium 内核)

## 安装步骤 (Miniconda)

```bash
# 1. 创建并激活 conda 虚拟环境
conda create -n grok python=3.10 -y
conda activate grok

# 2. 安装 Python 依赖
pip install -r requirements.txt

# 3. 安装 Playwright 浏览器驱动（仅首次需要）
playwright install chromium
```

## 使用方法

```bash
conda activate grok
python assisted_register_windows.py
```

运行后会提示输入：
- **每个线程注册次数**：每个线程要注册多少个账号
- **线程数量**：建议 1-3，过多可能触发风控

## 输出文件

| 目录 | 内容 |
|------|------|
| `result_grok/grok{时间戳}.txt` | 完整账号信息（邮箱、密码、SSO cookie） |
| `result_sso/sso{时间戳}.txt` | 仅 SSO cookie（每行一个） |

## 工作原理

1. **创建临时邮箱** — 通过 Mail.tm API 生成一次性邮箱地址
2. **启动 Chrome** — 以远程调试模式 (CDP) 启动 Chrome/Edge，Playwright 通过 CDP 连接控制浏览器
3. **自动填表** — 打开 x.ai 注册页，自动填入邮箱、验证码、密码和姓名
4. **人机验证** — 尝试自动通过 Cloudflare Turnstile 验证
5. **提取结果** — 监听 HTTP 响应和浏览器 cookie，捕获注册成功后的 SSO token

## 注意事项

- 需要本机已安装 Chrome 或 Edge 浏览器
- 脚本会自动检测 Chrome/Edge 路径，无需手动配置
- 每次任务结束后会自动清理临时 Chrome 用户数据目录
- 如果连续失败 5 次，会自动等待 10 秒后重试
