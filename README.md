# 🏀 NBA Recap Agent

Ask an NBA question. The agent looks up the ESPN scoreboard, chooses
which games to investigate, and uses box scores to explain what stood out.

[Open in Colab](https://colab.research.google.com/github/akashmathur/nba-recap-agent/blob/main/nba_agent.ipynb)

# Example run

**Question:** What was the best NBA game to watch on November 12, 2026, and why?

## Detroit Pistons at Washington Wizards

**Score:** Detroit Pistons 140, Washington Wizards 144

**Why this was the best game to watch:**

This was the standout game of the night on November 12, 2026, featuring a thrilling 4-point victory for the Washington Wizards over the Detroit Pistons in a high-scoring offensive duel.

**Key highlights:**

- **Cade Cunningham** put on a historic performance with **52 points, 5 rebounds, and 11 assists** for the Pistons, nearly carrying his team to victory single-handedly.
- **Trae Young** answered with an equally impressive **37 points and 14 assists** for the Wizards, orchestrating the offense and leading his team to the win.
- The game was decided by just 4 points, making it the closest competitive matchup of the four games played that night.
- Both teams combined for 284 points, showcasing an entertaining, fast-paced offensive showcase.

While the Hornets-Suns game was also close (5-point margin), the Pistons-Wizards matchup delivered more star power, higher scoring, and a more dramatic individual performances, making it the clear best game to watch on November 12, 2026

# How I Built It

I built this while working through the Hugging Face Agents Course. The product question was simple: **Can I ask which NBA game is worth watching and get a recommendation I can verify?** I wanted the answer to explain why one game stood out, with specific stats and a link back to ESPN.

### Model choice

I kept **NVIDIA Nemotron 3.5 Super VL 120B via Baseten**, accessed through Hugging Face Inference Providers. The course’s default Qwen setup returned a `model_not_supported` error. I got `gpt-oss-120b` working with mock game tools, which let me test the agent loop before connecting ESPN. My goal was to find the most capable model while minimizing input/output token cost.

I kept the Nemotron/Baseten configuration because it demonstrated the workflow I wanted to ship. It fetched the slate, investigated two games, cited a 14-for-20 shooting performance, and returned the ESPN link from the tool. My selection was based on that end-to-end run. I’d need to run the same prompts against the same data to compare model quality, cost, or latency.

### What I built and what I reused

| I built | I reused |
|---|---|
| Python tools to fetch the ESPN scoreboard and game summaries, pass game IDs between calls, and return selected box-score stats and a source link. | ESPN’s scoreboard and summary endpoints. |
| A date tool that reads the browser’s timezone in Colab to resolve “today.” | Colab’s browser bridge and Python’s timezone library. |
| Instructions for how to handle a full-slate recap, a specific matchup, or a single recommendation. I also defined which claims need direct evidence and how the answer should read. | `smolagents` for the agent loop, tool registration, inference client, and built-in search, webpage, and final-answer tools. |

### What I tested and what I learned

| Test | What happened | What it taught me |
|---|---|---|
| **Can the model call my tools?** | In the mock run, it passed `date` to a function that didn’t accept it. The trace showed the exact mismatch. | The tool signature and description are part of the interface the model relies on. |
| **Can it make a recommendation from ESPN data?** | It fetched the slate, pulled box scores for two games, and cited a concrete stat and an ESPN link in its recommendation. | A useful answer needs both a reason to watch and evidence the reader can check. |
| **What happens when someone just asks for “the best game today”?** | It got the local date right, then fetched the same slate twice because tipoff timestamps showed the next day in UTC. It also inferred overtime from minutes played and returned a dictionary. | I used those failures to clarify how to interpret UTC timestamps, require explicit evidence for overtime, and ask for readable Markdown. |

The current version can explain **who stood out** using scores and box scores. To reliably explain **what changed the game**, I’d expose play-by-play and use it to verify the turning point.

# Next steps

I want to turn this into an always-on NBA assistant that helps me decide **what to watch before tipoff, when to tune in during a game, and what to catch up on at night.**

| What I’d build | How it would work |
|---|---|
| **Upcoming game recommendations** | Before tipoff, recommend one or two games from the upcoming slate and explain why they’re worth watching using available context like team form, standings, and player availability. |
| **Memory and personalization** | Remember favorite teams and players, spoiler preferences, and which games I’m following. Keep enough game history to pick up where the last update left off. |
| **Streamlit chat** | Give the agent a browser interface where I can ask a question, follow up on a recommendation, and tell it to follow a game. |
| **Live Telegram check-ins** | Run a background worker that checks followed games and sends a Telegram message when something worth tuning in for happens, like a big comeback or a close game late in the fourth. |
| **Nightly recap** | Send one short Telegram digest with the game worth catching up on, standout performances, and turning points backed by play-by-play. |

I’d start with one end-to-end flow: **recommend an upcoming game → let me follow it → send a live update if something changes → recap it that night.** I’d judge the live updates by whether I’d actually want to stop what I’m doing and tune in.

# Try it yourself

1. Click **Open in Colab**, then choose **Runtime → Run all**.
2. Sign in when prompted with your own Hugging Face token that has
   **Make calls to Inference Providers** permission.
3. Enter an NBA question when the last cell prompts you.
