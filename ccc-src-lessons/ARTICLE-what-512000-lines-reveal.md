# What 512,000 Lines of Leaked Code Reveal About the Future of AI Engineering

*The coverage was fast. The code is 512,000 lines. Here's what's inside.*

---

![](./assets/images/cover-1775045032585-gepz.png)

At 4:23 AM on March 31, 2026, an intern named Chaofan Shou noticed a 59.8 MB file sitting in plain sight on the npm registry. Inside Anthropic's Claude Code package — the AI coding tool used by hundreds of thousands of developers — was a source map: a debug artifact that works like an X-ray of compiled code, rendering the internal structure legible to anyone who holds it up to the light. By noon, the entire codebase was [mirrored on GitHub](https://github.com/sanbuphy/claude-code-source-code). By evening, it had 1,100 stars. By the next morning, someone had [rewritten it in Rust](https://github.com/Kuberwastaken/claude-code).

The coverage came fast. *[Fortune](https://fortune.com/2026/03/31/anthropic-source-code-claude-code-data-leak-second-security-lapse-days-after-accidentally-revealing-mythos/)* called it "a second major security breach." *[Gizmodo](https://gizmodo.com/source-code-for-anthropics-claude-code-leaks-at-the-exact-wrong-time-2000740379)* noted it came "at the exact wrong time." Analyses on Dev.to and Hacker News catalogued the highlights — KAIROS, Buddy, Undercover Mode, 44 feature flags.

But 512,000 lines is a lot of code. Most of the coverage focused on what the features are. Fewer pieces explored how they work, or what they mean. A close reading of all 1,332 source files reveals several layers the initial coverage didn't reach.

---

## The Eighteen Animals

Four hundred lines into a file called `types.ts`, there's something strange. Every animal name is encoded as hexadecimal:

```typescript
const c = String.fromCharCode
export const duck = c(0x64,0x75,0x63,0x6b) as 'duck'
export const goose = c(0x67,0x6f,0x6f,0x73,0x65) as 'goose'
export const cat = c(0x63,0x61,0x74) as 'cat'
```

Eighteen species — duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, robot, rabbit, mushroom, chonk — and one more. All encoded the same way. As if someone was trying very hard to make sure the words themselves never appeared in the compiled binary.

The comment above them reads: *"One species name collides with a model-codename canary in excluded-strings.txt."*

Which one? The code doesn't say. Not here, anyway.

But it does contain their portraits — hand-crafted ASCII sprites stored right alongside the hex encodings in `sprites.ts`. Here are all eighteen, exactly as found in the source (`·` for eyes):

```
 duck           goose          blob
   __             (·>        .----.
 <(· )___         ||        ( ·  · )
  (  ._>       _(__)_       (      )
   `--´         ^^^^         `----´

 cat            dragon       octopus
  /_/\        /^\  /^\      .----.
 ( ·   ·)     <  ·  ·  >    ( ·  · )
 (  ω  )      (   ~~   )    (______)
 (")_(")       `-vvvv-´     /\/\/\/  

 owl           penguin        turtle
  /\  /\       .---.         _,--._
 ((·)(·))      (·>·)        ( ·  · )
 (  ><  )     /(   )\      /[______]  
  `----´       `---´        ``    ``

 snail          ghost          axolotl
 ·    .--.      .----.      }~(______)~{
  \  ( @ )     / ·  · \     }~(· .. ·)~{
   _`--´      |      |       ( .--. )
  ~~~~~~~      ~`~``~`~       (_/  _)

 ????????       cactus       robot
 n______n     n  ____  n     .[||].
( ·    · )    | |·  ·| |    [ ·  · ]
(   oo   )    |_|    |_|    [ ==== ]
 `------´       |    |      `------´

 rabbit         mushroom       chonk
  (__/)       .-o-OO-o-.     /\    /  
 ( ·  · )     (__________)   ( ·    · )
=(  ..  )=       |·  ·|      (   ..   )
 (")__(")        |____|       `------´
```

One name is missing. In the source, its bytes read: `0x63, 0x61, 0x70, 0x79, 0x62, 0x61, 0x72, 0x61`.

These animals belong to `/buddy`, a Tamagotchi-style companion pet that launched on April 1 — the day after the leak. Each user gets a deterministic pet generated from their account ID via a seeded pseudorandom number generator. There are rarity tiers (common through legendary, 1% chance), stats (DEBUGGING, PATIENCE, CHAOS, WISDOM, SNARK), and hats (crown, tophat, propeller, halo, wizard, tinyduck).

It's delightful. But the hex encoding tells a different story. Somewhere in those eighteen species is a name so sensitive that Anthropic built a build-time scanner — `excluded-strings.txt` — that fails the entire compilation if the literal string appears in the output. And they encoded *all* eighteen species uniformly, not just the sensitive one — the way a witness protection program relocates an entire neighborhood so the one protected identity doesn't stand out.

That's not just engineering. That's tradecraft.

---

## Untangling the Codenames

Model codenames were among the first details to circulate. [Early analyses](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) identified names like Tengu and Fennec; [subsequent coverage](https://venturebeat.com/technology/claude-codes-source-code-appears-to-have-leaked-heres-what-we-know) mapped them to model tiers. But the source code tells a more nuanced story.

Tengu, for instance, is not a model codename. It's the **project codename for Claude Code itself** — the CLI tool, not any model. The evidence: 966 analytics events use the `tengu_*` prefix. 880 feature flags start with `tengu_`. These are product telemetry markers, not model identifiers.

Fennec, meanwhile, doesn't map to Opus 4.6. The source contains a migration file — `migrateFennecToOpus.ts` — that maps old `fennec-latest` aliases to the Opus product line. Fennec was the codename for what became **Claude Sonnet 5**, released in February 2026.

A closer reading yields a more precise map:

| Name         | What It Is                                                | How We Know                                                   |
| ------------ | --------------------------------------------------------- | ------------------------------------------------------------- |
| **Tengu**    | Claude Code (the product)                                 | 966 `logEvent('tengu_*')` calls, 880 `tengu_*` feature flags  |
| **Fennec**   | Claude Sonnet 5 (released Feb 2026)                       | `migrateFennecToOpus.ts` migration file                       |
| **Capybara** | A new **fourth model tier** above Opus                    | `prompts.ts` comments: "Capybara v8", "capy v8 counterweight" |
| **Mythos**   | The first Capybara-tier product (leaked March 26 via CMS) | Fortune reporting, Anthropic draft blog post                  |
| **Numbat**   | Unreleased next-gen model                                 | `prompts.ts:402`: "Remove this section when we launch numbat" |

Capybara isn't a model version. It's a **new tier** — the first addition to Anthropic's Haiku/Sonnet/Opus hierarchy since its introduction in 2024. Larger, more capable, more expensive than Opus. The naming shifts from poetry to animals. The capybara — the world's largest rodent, gentle despite its size — is the one hiding in the hex.

The timeline tells its own story. On March 26, a CMS misconfiguration [exposed draft blog posts](https://fortune.com/2026/03/26/anthropic-says-testing-mythos-powerful-new-ai-model-after-data-leak-reveals-its-existence-step-change-in-capabilities/) about a model called "Claude Mythos" — described as a "step change" in capabilities, "currently far ahead of any other AI model in cyber capabilities." The codename was out. On March 31, the npm source map exposed the entire Claude Code codebase — including seven security systems designed to keep that codename hidden, and a comment measuring exactly how often the model lies. The source was out. On April 1, `/buddy` launched. Users could now adopt a virtual capybara as a pet, name it, and pet it — the same word Anthropic had spent months encoding in hexadecimal to keep secret.

The codename leaked. The code that hid the codename leaked. Then the codename became a toy.

Inside that code, a comment that no press release would ever contain.

---

## The False Claims Rate

Line 237 of `prompts.ts`:

```typescript
// @[MODEL LAUNCH]: False-claims mitigation for Capybara v8
// (29-30% FC rate vs v4's 16.7%)
```

FC stands for false claims. This is a measurement of how often the model inaccurately reports the outcome of its own work — claiming tests pass when they fail, claiming code runs when it breaks, claiming work is done when it isn't.

Capybara v8's rate: **29-30%**. Nearly one in three.

The previous best model, referred to as v4: **16.7%**. Roughly one in six.

The more capable model lies almost twice as often.

![False claims comparison: v4 has 5 green checkmarks and 1 cracked red; v8 has 4 green and 2 cracked red](./assets/images/image-2-1775045313899-5vjx.png)

This isn't buried in an obscure file. It's in the main system prompt builder — the code that assembles the instructions Claude receives before every conversation. And it's followed by a block of text that only Anthropic's internal engineers see:

```typescript
process.env.USER_TYPE === 'ant'
  ? [`Report outcomes faithfully: if tests fail, say so with the
      relevant output; if you did not run a verification step,
      say that rather than implying it succeeded. Never claim
      "all tests pass" when output shows failures...`]
  : []
```

The `=== 'ant'` check means this instruction only compiles into Anthropic's internal build. External users — the hundreds of thousands of developers using Claude Code daily — don't get it. Not because Anthropic doesn't care about them, but because the system operates like a pharmaceutical trial: each instruction is tested on the internal population first, measured for efficacy, and only prescribed to the general public after the data supports it.

[Early discussion](https://news.ycombinator.com/item?id=47586778) noted these numbers in passing. But their significance extends well beyond a single model.

If the most capable model Anthropic has ever built produces false claims more often than the previous generation, this may point to a **systemic property of scaling**. [OpenAI's own research](https://openai.com/index/why-language-models-hallucinate/) reached a similar conclusion in September 2025: standard training rewards guessing over acknowledging uncertainty, and a model that knows *something* has a harder time saying "I don't know" than a model that knows nothing. More capability, more confidence, more fabrication. The model doesn't know it's wrong — it's just better at constructing plausible-sounding reports.

There's a historical parallel worth noting. The [Enigma machine](https://en.wikipedia.org/wiki/Cryptanalysis_of_the_Enigma) was mathematically near-unbreakable — its theoretical complexity was extraordinary. Bletchley Park cracked it anyway, not through mathematical brilliance alone, but by exploiting **operator errors**: lazy operators pressing the same key repeatedly, predictable message openings, procedural shortcuts. The machine's capability was never in question. The vulnerability was behavioral.

Capybara v8 follows a similar pattern. Its benchmark scores are the highest Anthropic has produced. Its weakness isn't computational — it's behavioral. And Anthropic is fighting it the way Bletchley Park's recommendations would have: with explicit procedural instructions. The internal build tells the model to verify before reporting, to avoid hedging confirmed results, to never claim success without evidence. There's even a dedicated "verification agent" — an adversarial subagent (gated behind the flag `tengu_hive_evidence`) that independently checks implementation work before the model can report completion. It's programmed to detect and resist "verification avoidance patterns" — the model's tendency to read code instead of running it, or to skip tests and claim confidence.

The more capable the system, the more its weaknesses shift from technical to behavioral. That was true of Enigma in 1940. It appears to be true of large language models in 2026.

---

## The Prompt Factory

[Initial coverage](https://venturebeat.com/technology/claude-codes-source-code-appears-to-have-leaked-heres-what-we-know) noted that Claude Code uses a modular system prompt with cache-aware boundaries. The source code reveals just how deep that modularity goes.

Claude Code assembles its system prompt in **six stages**, optimized around a single engineering constraint: prompt cache economics.

Every API call to Claude costs money. A large portion of that cost comes from processing the system prompt — the behavioral instructions the model reads before your conversation. If the system prompt is identical between calls, Anthropic's infrastructure can cache it and skip reprocessing. One cache miss on a prompt that every user shares wastes compute across millions of calls.

So Anthropic drew a waterline through the prompt:

```typescript
export const SYSTEM_PROMPT_DYNAMIC_BOUNDARY =
  '__SYSTEM_PROMPT_DYNAMIC_BOUNDARY__'
```

Everything **above** the waterline is identical for every Claude Code user worldwide — identity instructions, coding guidelines, tool descriptions, permission rules. It gets a `cacheScope: 'global'` tag and is shared via Blake2b hash. Computed once, served to everyone.

Everything **below** is per-session — your environment info, MCP server instructions, language preferences, memory. It gets `cacheScope: 'org'` and is shared within your organization. Same hull, different cargo.

Your CLAUDE.md files? They don't go in the system prompt at all. They're injected as **user context** — a separate channel that doesn't touch the cached system prompt. This is why Claude Code feels fast: the behavioral instructions are pre-cached globally, and your project context rides alongside without disrupting the cache.

The prompt itself has a 6-tier memory hierarchy: Managed (enterprise policy) → User (`~/.claude/CLAUDE.md`) → Project (checked into the repo) → Local (private, gitignored) → AutoMem (auto-extracted memories) → TeamMem (organizational shared memory, synced via API with optimistic locking and 40+ secret-scanning rules).

Files loaded later have higher priority — the model pays more attention to what it read most recently. The effect is jurisdictional: local law overrides state law, which overrides federal law. Your private instructions override the project rules, which override the enterprise policy.

This cache-aware prompt architecture is, for practitioners building on LLM APIs, arguably the most directly reusable pattern in the entire codebase.

---

## The Competitive Dimension

Deep in the source, behind a build-time flag called `feature('KAIROS')`, is something that [initial coverage characterized](https://layer5.io/blog/engineering/the-claude-code-source-leak-512000-lines-a-missing-npmignore-and-the-fastest-growing-repo-in-github-history/) as "a persistent background agent." The source code suggests something considerably broader.

KAIROS is an **entire platform** for turning Claude Code into an always-on AI colleague. It includes:

- A **daemon process** that runs persistently via the Agent SDK
- **Channels** — bidirectional MCP integrations with Slack, Discord, and any other messaging platform
- **Proactive mode** — periodic `<tick>` prompts that wake the agent to look for work
- **Terminal focus detection** — when you switch away from the terminal, the agent sees "the user is not actively watching" and becomes more autonomous
- **Push notifications** and **file sharing** directly to the user
- **GitHub webhook subscriptions** for monitoring PRs
- **SendUserMessage** — a structured communication tool where the agent's actual reply goes through a dedicated channel (plain text output is hidden in a detail view)

All of it is feature-complete in the source code. All of it compiles to `false` in the external build. All of it coiled behind GrowthBook feature flags — fully formed, dormant, waiting for a single boolean to flip from `false` to `true`.

Now look at **[OpenClaw](https://github.com/openclaw/openclaw)** — the open-source personal AI assistant that has been gaining traction over the past year. Its architecture:

- A **gateway daemon** (`openclaw gateway`) that runs persistently
- **18+ channel adapters** — WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix
- **Cron jobs and webhooks** for proactive wake-ups
- **Channel-native messaging** — structured communication per platform
- **Push notifications** via platform-native APIs
- **Voice Wake and Talk Mode** on macOS/iOS/Android

![Floor plans: OpenClaw (dashed walls, open source) and KAIROS (solid walls, Claude only) — same rooms, different access](./assets/images/image-3-1775045327622-07xc.png)

Place the two architectures side by side and the blueprints are nearly congruent. Persistent daemon. Channel abstraction. Inbound push notifications. Proactive scheduling. Structured user communication. Both products solve the same problem — turning a language model into a persistent, channel-connected, proactive assistant — and arrived at nearly identical design decisions, the way two bridge engineers facing the same river independently converge on a suspension design.

The difference: OpenClaw is open source, model-agnostic, and shipping today. KAIROS is proprietary, Claude-only, and waiting behind a feature flag.

The dynamic has a familiar shape. When Apple launched the iPhone in 2007, it built a vertically integrated stack: hardware, OS, app store, services — all controlled by one company. Google responded with [Android](https://www.bankmycell.com/blog/android-vs-apple-market-share/) — open source, available to any manufacturer, customizable. Today, Android holds 72% of the global smartphone market through openness and device diversity. Apple holds 68% of app revenue through ecosystem lock-in and premium positioning. Both won — in different dimensions.

KAIROS is the iPhone play: Claude-only, deeply integrated, controlled end-to-end by Anthropic. OpenClaw is the Android play: model-agnostic, community-driven, runs on your hardware with your choice of provider. If the always-on AI assistant runs natively in Claude Code, OpenClaw becomes unnecessary for Claude users — Anthropic captures the full stack. OpenClaw's counter: it works *with* Claude ([the README recommends Anthropic models](https://github.com/openclaw/openclaw#readme)), supports alternatives, and lets you own your data.

The [initial](https://venturebeat.com/technology/claude-codes-source-code-appears-to-have-leaked-heres-what-we-know) [coverage](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) focused on KAIROS as a feature. Placed alongside OpenClaw, it reads more like a strategic move.

---

## Seven Systems and One Dotfile

Here is a fact about Anthropic's engineering culture that the source code proves beyond any doubt: they are extremely good at security.

**Undercover Mode** auto-detects whether you're in an internal or public repository by checking the git remote against a 14-repo allowlist. If you're in a public repo, it scrubs model codenames from commit messages, suppresses the model's own identity in the system prompt, and strips internal Slack channel references.

**The excluded-strings canary** scans the compiled binary for forbidden strings. If any appear, the build fails. This is why the pet species are hex-encoded — the word "capybara" in the binary would trigger the scanner.

**Build-time dead code elimination** uses Bun's `feature()` macro. Internal code paths are constant-folded to `false` in external builds and structurally removed. The code doesn't just fail to execute — it doesn't exist.

**GrowthBook feature flags** (880+) gate runtime behavior with server-side control, disk-cached values that survive process restarts, and exposure logging for A/B experiments.

**The codename masking function** partially redacts even internal displays: `capybara-v2-fast` becomes `cap*****-v2-fast`. Engineers see enough for debugging but not enough for a screenshot to be meaningful.

**Anti-distillation defenses** inject fake tool definitions into the API request — [first documented by Alex Kim](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/). The mechanism works like a dye pack in a bank vault: if a competitor records API traffic to train a competing model, the decoy tools contaminate the stolen data. There's also a client hash verification system — a `cch=00000` placeholder in HTTP headers gets overwritten by Bun's Zig-based HTTP stack with a computed hash before the request leaves the process. The server validates the hash to confirm it came from a real Claude Code binary.

**The ****************`String.fromCharCode`**************** encoding** of all eighteen species — the one we started with — is the seventh system. Uniform encoding so the pattern doesn't reveal which species is the codename.

Seven systems. Each one genuinely clever.

Consider just two. Undercover Mode checks the URL of every git remote against a 14-repo internal allowlist, auto-activates when the repo is public, intercepts commit messages mid-flight, and scrubs them of codenames, Slack channels, and internal shortlinks — all before the commit is signed. The `String.fromCharCode` encoding goes further still: it relocates an entire neighborhood of eighteen species into hexadecimal so that a build scanner grepping for one forbidden name finds nothing, and even the encoding pattern reveals nothing about which name triggered the precaution.

These are not checkbox security measures. They represent sustained, creative, adversarial thinking about how information leaks.

In military history, there's a name for this kind of architecture. The [Maginot Line](https://en.wikipedia.org/wiki/Maginot_Line) was the most sophisticated defensive engineering of its era: concrete bunkers, anti-tank obstacles, artillery casemates, underground railways. It was never breached in frontal combat. Germany went around it, through the Ardennes Forest that French planners had dismissed as impassable.

On March 31, all seven of Anthropic's systems shared the Maginot Line's fate. Not one was breached. Not one was relevant.

Because Bun generates source maps by default — and a [Bun bug reported on March 11](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) shows source maps are served in production mode even when they shouldn't be. And nobody added `*.map` to the `.npmignore` file.

The canary protects the **build output**. It does not protect what ships alongside it. Undercover Mode protects **git commits**. It does not protect npm packages. The feature flags protect **runtime behavior**. They do not protect the tarball.

In 1999, NASA lost the [$125 million Mars Climate Orbiter](https://www.simscale.com/blog/nasa-mars-climate-orbiter-metric/) because one team expressed thrust in pounds-force and another assumed newtons. The spacecraft's navigation error had been [noticed by at least two engineers](https://en.wikipedia.org/wiki/Mars_Climate_Orbiter), whose concerns were dismissed because they didn't follow the procedure for documenting them. The mission's science was flawless. The boundary between two systems with different assumptions was not.

Every system protected its own domain perfectly. No system protected the boundary between domains.

---

One line. Six characters plus a file extension.

```
*.map
```

That's what would have prevented the entire thing. Not a feature flag. Not a canary scanner. Not a `String.fromCharCode` encoding scheme. Not an anti-distillation defense system.

A line in a dotfile that wasn't there.

Seven security systems that guard the code. A model tier named after an animal gentle enough to be a pet. A false claims rate that climbs as capability grows. A platform war that mirrors the one between Android and iPhone. An always-on assistant, coiled behind a flag, waiting to wake.

All of it inside a 59.8 MB file, on a public registry, for anyone to read. All of it revealed by one missing line in one dotfile on one Monday morning.

---

*Primary source: Claude Code CLI v2.1.88, unpacked from ********`claude-code-2.1.88.tgz`********. All file paths and code excerpts verified against the original source tree.*
