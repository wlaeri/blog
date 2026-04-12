---
title: 'Hello World'
subtitle: 'Testing the blog scaffolding'
description: 'The first post on this blog — testing markdown, code blocks, and syntax highlighting.'
date: 2026-04-12
---

This is the first post on my blog. Just testing that the scaffolding works — markdown rendering, syntax highlighting, and the overall layout.

## TypeScript

Here's a typed function:

```typescript
interface BlogPost {
  title: string;
  subtitle?: string;
  date: Date;
  tags: string[];
}

function formatPost(post: BlogPost): string {
  const date = post.date.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });

  return `${post.title} — ${date}`;
}
```

## JSON

And a config object:

```json
{
  "site": "https://blog.willlaeri.com",
  "output": "static",
  "markdown": {
    "syntaxHighlight": "shiki",
    "shikiConfig": {
      "theme": "github-dark"
    }
  }
}
```

If you're reading this, everything is wired up — content collections, static generation, syntax highlighting, and deployment. Time to write some real posts.
