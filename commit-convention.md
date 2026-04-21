## Commit Convention

This project follows a **strict, concise commit message format**.

### 1. Commit Message Structure

```
<type>: <short imperative description>
```

* **Single line only**
* **No body/description unless explicitly necessary**
* **Maximum clarity, minimum verbosity**
* **No trailers or metadata**

### 2. Rules

* Use **imperative mood** ("Add", "Fix", "Update", not "Added", "Fixes")
* Keep the title **short and precise**
* **One logical change per commit**
* Do **not** add a commit description unless:

  * the change is complex
  * context is required to understand the decision

❗ **Do NOT add any of the following:**

* `Co-Authored-By`
* AI / bot attribution (Claude, ChatGPT, Gemini, etc.)
* Sign-offs, trailers, or footers of any kind
* Explanatory paragraphs outside the title

> Commits must reflect **human ownership** and follow a **single-line format only**.

### 3. Allowed Commit Types

| Type       | When to use it                                    |
| ---------- | ------------------------------------------------- |
| `feat`     | New features or user-facing functionality         |
| `fix`      | Bug fixes                                         |
| `docs`     | Documentation-only changes                        |
| `refactor` | Code changes that do not add features or fix bugs |
| `test`     | Adding or updating tests                          |
| `chore`    | Maintenance tasks (deps, configs, tooling)        |
| `style`    | UI/styling changes with no logic impact           |
| `perf`     | Performance improvements                          |

### 4. Examples (Valid)

```
feat: Add push notification support for room service requests
fix: Resolve chat message not appearing in real-time
docs: Update README with Supabase setup steps
refactor: Extract push utils to lib/utils/push
chore: Update Supabase client dependencies
style: Adjust chat bubble layout for mobile
perf: Debounce message polling interval
```

### 5. Examples (Invalid)

❌ Too verbose

```
feat: Add push notification support for room service with VAPID key configuration
```

❌ Past tense

```
fix: Fixed chat messages not loading on refresh
```

❌ Missing type

```
Add push notification support
```

❌ Unnecessary description / metadata

```
fix: Resolve real-time subscription disconnect

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```
