# News Sources Configuration

Source configuration for the `/today` skill. Edit this file to add or remove sources — the skill reads it automatically.

## RSS Feeds

### 📚 Deep Thinking & Blogs

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| paul-graham | Paul Graham | https://paulgraham.com/rss | thinking |
| stratechery | Stratechery (Ben Thompson) | https://stratechery.com/feed/ | thinking |
| lenny | Lenny's Newsletter | https://lennysnewsletter.com/feed | product |
| acx | Astral Codex Ten (Scott Alexander) | https://astralcodexten.substack.com/feed | thinking |
| joel | Joel on Software | https://joelonsoftware.com/feed/ | thinking |
| ethan-mollick | Ethan Mollick (One Useful Thing) | https://oneusefulthing.org/feed | ai |
| hn | Hacker News (Frontpage 50+) | https://hnrss.org/frontpage?points=50 | tech |

### 🔬 AI Research & Engineering

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| karpathy | Andrej Karpathy | https://karpathy.bearblog.dev/feed/ | ai |
| deepmind | DeepMind Blog | https://deepmind.google/blog/rss.xml | ai |
| google-research | Google Research | https://research.google/blog/rss.xml | ai |
| arxiv-ai | arXiv cs.AI | http://export.arxiv.org/rss/cs.AI | ai |
| langchain | LangChain Blog | https://blog.langchain.dev/rss/ | ai |
| raschka | Sebastian Raschka | https://sebastianraschka.com/rss.xml | ai |
| fastai | fast.ai (Jeremy Howard) | https://www.fast.ai/index.xml | ai |
| distill | Distill.pub | https://distill.pub/rss.xml | ai |

### 🧠 Key Figures

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| sam-altman | Sam Altman | http://blog.samaltman.com/posts.atom | ai |
| dwarkesh | Dwarkesh Patel (Podcast) | https://www.dwarkeshpatel.com/podcast?format=rss | ai |
| amjad | Amjad Masad (Replit CEO) | https://amasad.me/rss | ai |

## Twitter/X Accounts

Twitter has no public RSS. The skill uses WebSearch to catch recent high-engagement tweets from key accounts.

### 🔬 Core Models & Algorithm Insight (The Scientists)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @karpathy | Andrej Karpathy | ai | Also has RSS — Twitter supplements real-time updates |
| @ilyasut | Ilya Sutskever | ai | Very low frequency but very high signal |
| @_akhaliq | AK | ai | Human arXiv RSS — first-hand paper dispatch |
| @polynoamial | Noam Brown | ai | OpenAI — RL/Search/Reasoning |
| @DrJimFan | Jim Fan | ai | NVIDIA — Foundation Agents / Embodied AI |

### 🛠️ Product & AI-Native Design (The Builders)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @amjad | Amjad Masad | ai | Replit CEO — also has RSS |
| @lennysan | Lenny Rachitsky | product | Product growth methodology — also has RSS |
| @yoheinakajima | Yohei Nakajima | ai | BabyAGI author — Autonomous Agents frontier |

### ⚙️ Engineering & Agent Infra (The Engineers)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @hwchase17 | Harrison Chase | ai | LangChain — Multi-agent orchestration |
| @rasbt | Sebastian Raschka | ai | LLM math-to-code visualizations — also has RSS |
| @jeremyphoward | Jeremy Howard | ai | fast.ai — pragmatism + compute critique — also has RSS |
| @karpathy_out | Karpathy Out of Context | ai | Karpathy engineering quotes aggregator |

### 🌐 Macro Trends & Visionaries

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @dwarkesh_sp | Dwarkesh Patel | thinking | Long-form interview highlights — also has RSS |
| @VitalikButerin | Vitalik Buterin | thinking | Decentralized governance / complex systems |
| @balajis | Balaji Srinivasan | thinking | Macro tech predictions — first principles |
| @linus_lee | Linus Lee | product | AI UI/UX design inspiration |

## Non-RSS Sources (WebSearch fallback)

People/orgs without RSS or active Twitter presence worth tracking separately.

| Name | Tracking Method | Notes |
|------|----------------|-------|
| Ilya Sutskever / SSI | WebSearch "Ilya Sutskever Safe Superintelligence" | Rare but high-signal, supplements @ilyasut |

## Curation Rules

- **Daily quota**: 3–5 items total (not per source)
- **Priority**: ai > product > thinking > tech
- **Recency**: prefer posts from last 24h, allow up to 48h for low-frequency sources
- **Dedup**: same story across multiple sources = one item, pick best source
- **arXiv**: only surface papers with unusually high engagement or from known labs

## Customization

To add sources specific to your domain, append rows to the tables above.
To remove sources you don't care about, delete or comment out rows.
The skill will read whatever is present in this file at runtime.
