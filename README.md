# 🏆 Contify

A Discord bot that tracks upcoming **Codeforces** contests and sends automated reminders to your server.

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python) ![discord.py](https://img.shields.io/badge/discord.py-1.x-5865F2?logo=discord) ![Codeforces](https://img.shields.io/badge/API-Codeforces-1F8ACB) ![License](https://img.shields.io/badge/License-GPL--3.0-blue)

---

## ✨ Features

- 📅 Fetches all upcoming Codeforces contests via the public API
- 🕐 Displays contest name, date, and time (converted to IST)
- 📢 Auto-sends reminders to a channel for contests happening within 3 days
- 🔁 Polls for new contests every hour automatically
- 📌 `$tc` command to check today's contests instantly

---

## 🏗️ Architecture

```
Discord Server
      │
      │  $sendhere / $contests / $tc
      ▼
┌─────────────────────────────────────────┐
│               bot.py                    │
│                                         │
│  ┌──────────────┐   ┌─────────────────┐ │
│  │   Commands   │   │  Background     │ │
│  │  $contests   │   │  Task           │ │
│  │  $tc         │   │  SendContest()  │ │
│  │  $sendhere   │   │  (every 1 hr)   │ │
│  └──────┬───────┘   └────────┬────────┘ │
│         │                    │          │
│         └──────────┬─────────┘          │
│                    │                    │
│             getContest()                │
│                    │                    │
└────────────────────┼────────────────────┘
                     │
                     ▼
        Codeforces API
        /api/contest.list
```

---

## 🔄 Flow

### Contest Fetch Flow

```
getContest()
     │
     ▼
GET https://codeforces.com/api/contest.list?gym=false
     │
     ▼
Filter: phase != "FINISHED"  (only upcoming/running)
     │
     ▼
For each contest:
  - Convert startTimeSeconds → IST (UTC+5:30)
  - Skip contests whose date has already passed
  - Build discord.Embed (name, date, time, link)
     │
     ▼
Return list of [embed, datetime_string]
```

### Auto-reminder Flow (`$sendhere`)

```
User: $sendhere
     │
     ▼
Bot confirms channel, starts background task SendContest()
     │
     ▼
Loop every 60 minutes:
     │
     ▼
getContest()  →  check each contest
     │
     ▼
If contest is within 3 days AND not already sent:
     │
     ▼
sendList.append(contest)  →  channel.send(embed)
```

---

## 🤖 Commands

| Command | Description |
|---|---|
| `$contests` | List all upcoming Codeforces contests |
| `$tc` | Show only today's contests |
| `$sendhere` | Register this channel for automatic contest reminders |

---

## 📁 Project Structure

```
contify/
├── bot.py      # All bot logic — commands, API fetch, background task
├── README.md
└── .env        # Bot token (not committed — add manually)
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.x
- A Discord bot token from the [Discord Developer Portal](https://discord.com/developers/applications)

### Setup

```bash
# 1. Clone the repo
git clone https://github.com/tarunyadav148/contify.git
cd contify

# 2. Install dependencies
pip install discord.py requests

# 3. Add your bot token
#    Open bot.py and replace the last line:
#    bot.run('put your token here')
#    with your actual token, or use python-dotenv:
#    bot.run(os.getenv('TOKEN'))

# 4. Run the bot
python bot.py
```

> **Tip:** Use `python-dotenv` and a `.env` file to keep your token out of source code:
> ```env
> TOKEN=your_discord_bot_token_here
> ```

---

## 🌐 API Used

**Codeforces Contest List API**

```
GET https://codeforces.com/api/contest.list?gym=false
```

Returns all contests. The bot filters to `phase != "FINISHED"` to get only upcoming and ongoing contests, then converts `startTimeSeconds` (UTC) to IST (UTC+5:30).

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Python 3.x | Core language |
| discord.py | Discord API wrapper, commands, embeds |
| requests | HTTP calls to Codeforces API |
| threading / asyncio | Background hourly reminder task |
| datetime | Timestamp conversion UTC → IST |

---

## 📄 License

GPL-3.0
