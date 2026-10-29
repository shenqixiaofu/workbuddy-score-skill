# WorkBuddy 积分助手

读取本机 WorkBuddy 登录态，直查「Buddy 加油站」接口，生成每日积分播报；支持自动签到、派猫猫旅行全自动闭环（先领后派）、余额查询与多渠道推送。

**fork 自 SkillHub 上的同技能（v3.1.7，作者 totorosir，MIT-0），本仓库在其基础上修了 bug、补了两套积分口径，并加入「加载即自动创建每日定时任务」。**

## 装法

把整个目录放到技能目录下：

```bash
cp -r workbuddy-score-skill ~/.workbuddy/skills/
```

WorkBuddy 会自动识别 `SKILL.md`（frontmatter 里 `name: totorosir-workbuddy-score`）。

## 用法

```bash
# 每日播报（默认弹系统通知，不写文件）
python3 scripts/buddy_station.py

# 自动签到 + 旅行闭环（先领后派）
python3 scripts/buddy_station.py checkin --travel-auto

# 只看数据不推送、不弹通知（调试用）
python3 scripts/buddy_station.py --no-push --no-toast

# 探测三通道授权可用性
python3 scripts/buddy_station.py --probe

# 生成桌面 HTML 报告
python3 scripts/buddy_station.py --html
```

授权三通道，脚本自动择优：本机登录态（零配置）、一次性网页授权、长效 refresh token（客户端长期不开也能跑）。

## 定时任务

加载技能后，agent 会**自动创建**每日 06:00 的自动化任务（`buddy_station.py checkin --travel-auto`），不需要手动交代。

幂等：同名任务只保留一条，已存在就跳过；改时间用 update，不要删了重建。

## 相对原版的改动

| 文件 | 改动 |
|---|---|
| `scripts/push_notify.py` | 修企微 markdown 字段名错误（`content` → `markdown`）。原版 HTTP 200 但服务端返回 `44004 empty content`，静默失败 |
| `scripts/buddy_station.py` | 新增 `fetch_balance()`，读真实账户余额（`/billing/meter/get-user-resource-summary`）。**该接口按 User-Agent 拦截，缺浏览器 UA 会返回 403 code 10085**，脚本已内置 |
| `scripts/buddy_station.py` | 播报区分两套口径：「账户积分余额」vs「活动累计积分」。原版把活动累计当成账户余额 |
| `SKILL.md` | 新增「零、加载即执行：自动创建每日定时任务」 |

## 注意

- **推送凭据不在本仓库**。webhook 存在 `~/.workbuddy/totorosir-push-message/config.json`（0600），仓库里只有占位符。
- 引用的是**非公开接口**（`/v2/billing/meter/*`、`/activity/growth/buddy/*`），官方随时可能改；改了只能抓包重找，脚本不保证长期有效。
- 写操作仅限三个已验证接口：签到、旅行领取、旅行派出。其余一律拒绝。

## License

MIT-0（沿用原技能）。
