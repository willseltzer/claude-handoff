# Handoff Plugin

Create handoff documents that let **any AI coding agent** continue your work.

## Install

Run these commands inside Claude Code:

```
/plugin marketplace add willseltzer/claude-handoff
/plugin install handoff
```

## Why?

AI sessions have limited context. When you switch tools, take a break, or hit a context limit, the next agent starts from scratch. Handoffs solve this by capturing:

- What you're trying to do
- What's done, what's not
- What failed (so the next agent doesn't repeat mistakes)
- Key decisions and their rationale
- Exactly how to continue

## Commands

| Command | Description |
|---------|-------------|
| `/handoff:create` | Full handoff with complete context |
| `/handoff:quick` | Minimal handoff - just the essentials |
| `/handoff:resume` | Continue from an existing handoff |

## Usage

### Creating a Handoff

When you're done (or switching agents):

```
/handoff:create
```

This generates `HANDOFF.md` with:
- Goal and progress
- Failed approaches (so they're not repeated)
- Key decisions with rationale
- Current state (what works, what's broken)
- Specific resume instructions

For simple tasks:

```
/handoff:quick
```

### Resuming from a Handoff

In a new session (Claude Code or any AI):

```
/handoff:resume
```

The agent will:
1. Check if the repo state has drifted
2. Summarize the handoff
3. Continue from where you left off

### For Non-Claude Agents

Just tell them:

> "Read HANDOFF.md and continue the work"

The format is agent-agnostic.

## Handoff Format

```markdown
# Handoff: [Task Title]

**Generated**: 2024-01-15 14:30
**Branch**: feature/auth
**Status**: In Progress

## Goal
Add user authentication with OAuth2.

## Completed
- [x] Set up OAuth2 provider config
- [x] Created login endpoint

## Not Yet Done
- [ ] Add token refresh logic
- [ ] Handle logout

## Failed Approaches (Don't Repeat These)
Tried passport.js first - it conflicted with our Express middleware.
`req.user` was always undefined because passport's session middleware
ran after our custom auth check. Switched to oauth4webapi which works
directly with fetch and doesn't touch Express.

## Key Decisions
| Decision | Rationale |
|----------|-----------|
| oauth4webapi over passport | Lighter weight, no middleware conflicts |
| Store refresh token in httpOnly cookie | More secure than localStorage |

## Current State
**Working**: Login flow returns valid tokens
**Broken**: Refresh endpoint returns 500 - `TokenExpiredError` at refresh.ts:42

## Code Context

// Hook signature - src/hooks/useAuth.ts
function useAuth(): {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

// POST /api/auth/login response
{ "accessToken": "eyJ...", "expiresIn": 3600 }
// Refresh token set as httpOnly cookie, not in response body

## Resume Instructions
1. Fix refresh endpoint: the JWT verify call at `refresh.ts:42` uses wrong secret
   - Expected: Returns new access token
   - Current: Throws TokenExpiredError even for valid tokens
2. Add logout: clear the httpOnly cookie, POST /api/auth/logout
3. Test full flow:
   - Login with test@example.com / testpass123
   - Wait 5 seconds, trigger refresh (or manually call /api/auth/refresh)
   - Expected: New token returned, no errors

## Setup Required
- `OAUTH_CLIENT_SECRET` env var must be set (see .env.example)
- Test user exists: test@example.com / testpass123

## Warnings
- OAuth provider sandbox resets daily at midnight UTC
- Don't use `localStorage` for tokens - we decided httpOnly cookies only
```

## Plugin Structure

```
claude-handoff/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── create.md      # /handoff:create
│   ├── quick.md       # /handoff:quick
│   └── resume.md      # /handoff:resume
├── skills/
│   └── handoff/
│       └── SKILL.md   # Auto-detection
└── README.md
```

## Tips

1. **Failed approaches are mandatory** - This is the most valuable part. "Tried X, it didn't work because Y" saves hours.

2. **Show code, don't describe** - "Created a hook" is useless. Show the signature so the next agent knows how to use it.

3. **Test steps need expected outcomes** - Not "verify it works" but "POST to /api/X, expect 200 with { status: 'ok' }".

4. **Include the actual error messages** - "It threw an error" vs "TokenExpiredError at line 42" - huge difference.

5. **Use `/handoff:quick` for simple tasks** - Not everything needs 10 sections.

## License

MIT
