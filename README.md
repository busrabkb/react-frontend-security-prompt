# React Auth Security Prompt

A structured prompt template for setting up enterprise-grade authentication security in React frontend projects, designed to be used with AI coding assistants (Claude, ChatGPT, Cursor, Claude Code, etc.).

This prompt adapts to your project's scale (small / medium / large) and infrastructure preferences (e.g. whether you use Redis), asking clarifying questions before scaffolding the right security architecture and generating the necessary files.

## What is this for?

Most React projects ship with shallow or incomplete authentication security: tokens stored in plain localStorage, no protection against accessing protected routes via URL after a session expires, no refresh mechanism. This prompt is designed to close those gaps — providing an industry-standard authentication layer that scales from a small MVP to a full enterprise system.

When you hand this prompt to an AI coding assistant, it will:

1. First ask about your project's scale and infrastructure preferences (Redis availability, database, backend framework)
2. Choose an appropriate architecture tier based on your answers
3. Generate all the necessary files (route guards, context, interceptors, middleware, etc.)

## Features

### Automatic session enforcement
- When a token expires or a session is terminated, users cannot access protected pages (e.g. `/dashboard`) simply by typing the URL
- A route guard (`ProtectedRoute`) checks token validity on every route entry

### Architecture that scales with your project

| Tier | Scope |
|---|---|
| Small scale | localStorage + axios interceptor + route guard |
| Medium scale | httpOnly cookies + access/refresh tokens + refresh token rotation + CSRF protection |
| Large scale | Medium tier + role-based access control + multi-device session management + OAuth2/OIDC + audit logging |

### Centralized API security
- All requests go through a single axios instance
- On a 401 response, the app either silently refreshes the token or redirects the user to login

### Flexible infrastructure support
- Redis is never assumed — the assistant asks whether you use it or want to set it up
- If Redis isn't available, the same logic (refresh token storage, token blacklisting) is implemented using database tables with periodic cleanup (cron jobs)

### Additional security layers (for large-scale systems)
- Refresh token rotation with token theft detection
- Rate limiting and brute-force protection
- Security headers (CSP, helmet.js)
- Audit logging and anomaly detection

## Usage

1. Copy the text from `prompt.md`
2. Paste it into your AI assistant of choice (Claude, Claude Code, Cursor, etc.)
3. Answer the questions the assistant asks (project scale, Redis preference, existing infrastructure)
4. Let the assistant generate the files matching your project's folder structure

## Who is this for?

- Developers adding authentication to a new React project from scratch
- Teams looking to upgrade a basic localStorage-based auth setup to a more secure architecture
- Projects that need enterprise-level features (role management, multi-device sessions, OAuth2)

## Notes

- This is not a code generator by itself — it's an **architectural guide**. Generated code should be reviewed against your project's actual structure (state management, backend framework, database)
- For security-critical projects (finance, healthcare, etc.), have the generated code reviewed by a security professional

## License

MIT
