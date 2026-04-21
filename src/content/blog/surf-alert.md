---
title: 'Surf Alert'
subtitle: 'Vibe coding a notification system for local surf conditions'
description: 'I got humbled by Jupiter, FL, so I built a Cloudflare Worker that scores the surf every hour and pings me when it is worth paddling out.'
date: 2026-04-20
heroImage: '/juno-beach-sunrise.jpg'
heroImageAlt: 'Juno Beach at sunrise'
heroImageCaption: 'Juno Beach at sunrise'
---

Last year I went surfing for the first time at Domes Beach in Rincón, Puerto Rico. It was awesome. I took a beating paddling out, but I was able to get up a few times and ride the waves back to shore. Dreams of becoming a surfer bro unlocked.

A few weeks ago I decided to give it another shot at my local beach in Jupiter, Florida, but it wasn't at all what I remembered. The waves at Juno Beach just didn't have the same power and consistency as Domes. I wasn't able to get up once.

Defeated, I shared my experience with a few friendly locals. They told me **conditions matter here**. Ocean swell. Wave interval. Wind direction. Tide patterns. These gents dropped so much knowledge I felt like I needed a PhD in wavology to shred the local gnar.

But they assured me when conditions were just right, the waves were magical. So, I had an idea — what if I built a notification system that analyzed publicly available weather data and sent me an email when the conditions were good?

## The vision

A cron job that wakes up every hour, checks the wave/wind/tide data for Jupiter, scores it 0–100, and pings me when it's worth suiting up. Simple serverless infra, specific problem, ~300 lines of TypeScript. The kind of project that used to take a weekend and now takes an afternoon thanks to AI.

## Vibe coding, mostly by spec

Here's what the workflow actually looked like: I didn't sit down and write code. I sat down and wrote a **spec**, argued with myself about the spec, then handed the spec to Claude and let it write the code.

Rough shape of the prompt:

```markdown
You are a senior TypeScript backend engineer. Build a production-ready
MVP for a "Surf Alert" system focused on Jupiter, Florida.

## Tech Stack (MANDATORY)
- Runtime: Cloudflare Workers
- Language: TypeScript
- Database: Neon (serverless Postgres)
- HTTP: native fetch (no heavy SDKs)
- Scheduling: Cloudflare Cron Triggers
- No frameworks

## Data Sources
- Open-Meteo Marine API (waves, wind)
- NOAA CO-OPS (tides)

## Core Logic
computeSurfScore(inputs) → { score: number, isGood: boolean }
- ideal wave height: 2.5–4.5 ft
- ideal period: >= 8 sec
- penalize strong onshore wind (>12 mph)

## Constraints
- Keep it under ~250–300 lines
- Clean, readable, production-quality code
- No unnecessary abstractions
- Prefer small pure functions
```

The interesting part is what's *not* in the prompt. I didn't tell Claude how to structure the file. I didn't specify a scoring formula — just the rules. I didn't write any SQL. The constraints ("no frameworks", "prefer small pure functions", "~300 lines") do more work than you'd expect: they kill the reflexive over-engineering that a lot of generated code suffers from.

A few iterations on the spec and one commit later, I had a working Worker deployed to the cloud.

## The scoring function

The heart of the whole thing is this. It's opinionated on purpose — Jupiter is a beginner-friendly beach break, and I want a high score to mean "easy to learn on," not "pro-grade swell."

```ts
function computeSurfScore(inputs: {
  waveHeightFt: number;
  wavePeriodS: number;
  windSpeedMph: number;
  windDirectionDeg: number;
  tideFt: number;
}): SurfScore {
  let score = 0;

  // Wave height: ideal 2.5–4.5 ft (max 40 pts)
  if (inputs.waveHeightFt >= 2.5 && inputs.waveHeightFt <= 4.5) score += 40;
  else if (inputs.waveHeightFt >= 1.5) score += 25;

  // Period: >= 10s = max 30 pts
  if (inputs.wavePeriodS >= 10) score += 30;
  else if (inputs.wavePeriodS >= 8) score += 25;

  // Wind: onshore (45°–135°) gets penalized
  const isOnshore =
    inputs.windDirectionDeg >= 45 && inputs.windDirectionDeg <= 135;
  if (inputs.windSpeedMph <= 5) score += 20;
  else if (inputs.windSpeedMph <= 10) score += isOnshore ? 10 : 15;

  // Tide: mid-tide is best at Jupiter
  if (inputs.tideFt >= 1 && inputs.tideFt <= 3) score += 10;

  return { score: Math.min(score, 100), isGood: score >= 70 };
}
```

Pure function, no I/O, easy to tweak. If my weights turn out to be wrong, the fix is just a few lines.

Initially, I set the "notification" mechanism to just store the results in a database and log to console. I ran it for a few days and took a look at the data.

<figure>
  <img src="/surf_score_plot.png" alt="Jupiter, FL surf score over time, Apr 14–18 2026" />
  <figcaption>Jupiter, FL surf score over time, Apr 14–18 2026</figcaption>
</figure>

Looking at the surf score over time, it looks like the threshold might be okay, but I might want to wait until two consecutive hours of good surf before sending an alert. Also, I'll have to put some guardrails on when to send the alert — don't need an email at 2 in the morning.

This week, I'll turn on the email alert and hit the ocean to verify the scoring algorithm. Surf's up dudes!
