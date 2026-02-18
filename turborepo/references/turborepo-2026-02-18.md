# Turborepo Reference (2026-02-18)

## Context7 Sources
- Library: `/vercel/turborepo`
- Docs and examples for task graph, cache inputs/outputs, env hashing.

## Practical Guidance
- Define deterministic `tasks` with explicit `dependsOn`.
- Declare `outputs` and `inputs` precisely to prevent stale cache hits.
- Use `env` and `globalEnv` selectively for cache-key correctness.
- Keep dev/watch tasks uncached.
- Set remote cache credentials through CI secrets.

## Example Pattern
```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"],
      "env": ["NODE_ENV", "API_URL"]
    }
  }
}
```

## Source URLs
- <https://github.com/vercel/turborepo>
- <https://turborepo.com/docs>
