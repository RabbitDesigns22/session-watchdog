# session-watchdog

**Wake the Claude Code sessions that got stuck, and leave the finished ones alone.**

`session-watchdog` is an agent skill for the Claude desktop app (Claude Code). A session that stops halfway looks exactly like one that finished: both are idle. You find out hours later, when you ask "are you stuck?".

This skill runs on a schedule and reads every session from the outside. It sorts each one into **DONE / WAITING-ON-USER / WAITING-ON-BACKGROUND / STOPPED-MID-TASK / ERROR / HUNG**, then acts:
- it **wakes only the stuck ones**, with a message that quotes their own pending step;
- it **never wakes** sessions that finished, that wait for you (a code, a login, an approval) or that are still running;
- it **reports** what only you can unblock.

## See the difference

> **You:** (Scheduled, every 30 minutes, nobody watching)

**Without the skill:** a session stops halfway on a rate limit and stays stopped until you ask "are you stuck?" the next morning.

**With the skill:** the watchdog reads every session's last turn, sorts it (done, waiting on you, stopped mid-task, error, hung) and wakes only the stuck ones, quoting their own pending step. It never wakes a finished session.

**Skip it if** you run one session and watch it yourself.

**Status:** in daily use in the author's own setup. Plain instructions plus a short Python script. MIT. Issues and suggestions welcome.

## Why you can trust it on a schedule

- **Hard limits in a log**: at most 1 wake per session every 3 h, 2 per day and 5 per run. `wake_log.py can-wake` enforces them before every message.
- **When unsure, it picks DONE.** A wrong wake costs money and can redo finished work. A missed one gets caught on the next run.
- **A dead-man switch watches the watchdog.** Every run ends with a heartbeat. A `UserPromptSubmit` hook prints one line in any session you type in when the heartbeat is more than 5 h old, or when the schedule was set up and no run ever finished.
- **Built from a real failure.** Its first 6 scheduled runs died silently on a permission prompt, and the app still marked them "succeeded". The skill now covers both traps: exact allow rules, and checking the run's real last line instead of the label.

The first clean run checked 36 sessions, woke none (none needed it) and left its heartbeat in 2 minutes.

## Install

```bash
npx skills add RabbitDesigns22/session-watchdog
```

Then:
1. Schedule a task every 1-2 hours whose prompt says: "Load the session-watchdog skill and follow it. Last step: `python <skill>/scripts/wake_log.py beat`."
2. Run `python <skill>/scripts/wake_log.py arm` once.
3. Add the hook (`~/.claude/settings.json`, `UserPromptSubmit`): `python <skill>/scripts/wake_log.py check-beat 5`.
4. Allow the exact tools the run uses (list in `SKILL.md`), then open the first run and check that it ends in `done (success)`.

It needs the desktop app's session tools (`list_sessions`, `list_events`, `SendMessage`).

## Works well with

- [`skill-harvest`](https://github.com/RabbitDesigns22/skill-harvest): reads all your sessions and finds the skills you are missing or forgetting.
- [`in-the-loop`](https://github.com/RabbitDesigns22/in-the-loop): many agents, one shared live board.

## Español

**Despierta a los chats de Claude que se quedaron pegados y no toca a los que ya terminaron.** Corre sola cada 1 o 2 horas:
- lee cada chat desde afuera;
- despierta solo a los que se quedaron a medias, en error o esperando tareas muertas, con su propio pendiente citado;
- te avisa lo que solo tú puedes destrabar.

Tiene límites duros y una marca de vida: si el vigilante deja de correr, cualquier chat donde escribas te lo dice.

Instalar: `npx skills add RabbitDesigns22/session-watchdog`

## More skills from the same setup

These come from one real workflow: a small agency run with 15+ Claude Code sessions in parallel. Each skill fixes one problem that kept happening there.

| Skill | What it fixes |
|---|---|
| [one-pass](https://github.com/RabbitDesigns22/one-pass) | get it right the first time; stop paying tokens for rework |
| [hora-exacta](https://github.com/RabbitDesigns22/hora-exacta) | real date and time in every message, so nothing lands on the wrong day |
| [hands-on](https://github.com/RabbitDesigns22/hands-on) | the agent does what it can and only asks you for codes, keys and payments |
| [easy-for-them](https://github.com/RabbitDesigns22/easy-for-them) | steps people actually follow: direct link, one action per line |
| [one-answer](https://github.com/RabbitDesigns22/one-answer) | one thought-through answer instead of plan, fix, new plan |
| [never-twice](https://github.com/RabbitDesigns22/never-twice) | a shared error log so no session repeats a known mistake |
| [report-now](https://github.com/RabbitDesigns22/report-now) | status checked live this minute, stamped with the time |
| [sort-me-out](https://github.com/RabbitDesigns22/sort-me-out) | one ordered list of what is urgent, waiting and forgotten |
| [in-the-loop](https://github.com/RabbitDesigns22/in-the-loop) | a live board so parallel sessions know what the others changed |
| [study-first](https://github.com/RabbitDesigns22/study-first) | reads everything in full and answers point by point, no invented details |
| [skill-harvest](https://github.com/RabbitDesigns22/skill-harvest) | reads your sessions and finds the skills you are missing |
| [small-budget-ads](https://github.com/RabbitDesigns22/small-budget-ads) | Meta Ads rules for 1-5 USD a day ad sets |
| [whatsapp-closer](https://github.com/RabbitDesigns22/whatsapp-closer) | evidence-ranked rules for a WhatsApp sales assistant that closes |
| [whatsapp-number-onboarding](https://github.com/RabbitDesigns22/whatsapp-number-onboarding) | connect a number to the WhatsApp Cloud API in the safe order |
| [proposal-pdf](https://github.com/RabbitDesigns22/proposal-pdf) | short PDF proposals with every price checked against its source |
| [published-vs-planned](https://github.com/RabbitDesigns22/published-vs-planned) | says "published" only for what it saw live on the public page, Facebook and Instagram checked apart |
| [source-every-fact](https://github.com/RabbitDesigns22/source-every-fact) | every price, phone and name on a post traced to its source, nothing added |

If one of them saved you a redo, a star helps other people find it.

## License

MIT. Made by [Rabbit Designs](https://github.com/RabbitDesigns22).
