---
name: pre-push-secrets
description: Checks the commits about to be pushed for leaked secrets before every git push. Looks for committed env files and key files, hardcoded credentials and tokens, and logs or prints that would write a secret. Reports what it finds with the value masked and asks whether to push anyway. Use whenever running or proposing git push, including push to a new branch or a merge being pushed, even when secrets are not mentioned.
---

# Pre-push secrets

A pushed secret has to be treated as public: removing it later does not take it out of the history, forks or caches, and most leaked secrets stay valid for years. Check before the push, while the commits are still local and easy to fix.

This is a warning, not a block. Report what was found and let Joel decide.

## 1. What is about to be pushed

Check the commits the push sends, not only the last diff: a secret added in one commit and deleted in the next is still in the history that gets pushed.

```bash
branch=$(git rev-parse --abbrev-ref HEAD)
base=$(git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null) \
  || base=$(git merge-base HEAD origin/HEAD 2>/dev/null) \
  || base=$(git merge-base HEAD origin/main)
git log --oneline "$base..HEAD"
```

If the push names another ref, such as `git push origin feature:main`, the range is `origin/main..feature`. If the range is empty, there is nothing to check.

Collect every file any commit touched and every added line, commit by commit:

```bash
git log --name-only --format= "$base..HEAD" | sort -u
git log -p --format='commit %h' "$base..HEAD" | grep -E '^(commit |\+\+\+ |\+[^+])'
```

If `gitleaks` is installed, run it as well. It knows hundreds of formats and gives fewer false positives:

```bash
gitleaks git --redact --log-opts="$base..HEAD"
```

Read the code of the changed lines too. Patterns catch formats; reading catches the generic secret and the log line that prints one.

## 2. What to look for

### Files that should never be committed

| Kind | Examples |
|---|---|
| Env files with real values | `.env`, `.env.local`, `.env.production`; `.env.example`, `.env.sample` and `.env.template` are fine if their values are empty or fake |
| Private keys and certificates | `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks`, `*.keystore`, `*.ppk`, `id_rsa`, `id_ed25519` |
| Cloud and tool credentials | `credentials.json`, `service-account*.json`, `.aws/credentials`, `.docker/config.json`, `kubeconfig`, `.npmrc`, `.pypirc`, `.netrc`, `.git-credentials`, `.htpasswd` |
| Infrastructure state | `terraform.tfstate*`, `*.tfvars` |
| Data | database dumps (`*.sql` with `INSERT` of real rows), `*.sqlite`, `*.db`, `*.bak`, exports with personal data |
| Local tool settings | `.claude/settings.local.json`, IDE files that store tokens |

### Values with a known format

| Secret | Pattern |
|---|---|
| Private key block | `-----BEGIN (RSA \|EC \|DSA \|OPENSSH \|PGP )?PRIVATE KEY-----` |
| AWS access key | `(AKIA\|ASIA)[0-9A-Z]{16}`, and `aws_secret_access_key` next to 40 characters |
| GitHub token | `ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_`, `github_pat_` |
| GitLab token | `glpat-` |
| Slack | `xox[abprs]-`, `hooks.slack.com/services/` |
| Stripe | `sk_live_`, `rk_live_` (`sk_test_` is lower risk, still report it) |
| Google | `AIza[0-9A-Za-z_-]{35}`, a JSON with `"type": "service_account"` and `"private_key"` |
| AI services | `sk-ant-` (Anthropic), `sk-proj-` and `sk-` (OpenAI), `hf_` (Hugging Face), Azure OpenAI keys |
| Package registries | `npm_`, `pypi-` |
| Email and messaging | `SG.` (SendGrid), Twilio `SK` + 32 hex, Mailgun `key-` |
| Azure storage | `AccountKey=`, `DefaultEndpointsProtocol=` with a key |
| JWT | `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+` |
| Connection string with a password | `mongodb(+srv)?://`, `postgres(ql)?://`, `mysql://`, `redis://`, `amqp://` followed by `user:password@`, and Go DSNs like `user:password@tcp(` |

### Generic secrets

Most leaks have no known prefix: a password, a token or a key assigned to a variable. Look for names such as `password`, `passwd`, `pwd`, `secret`, `token`, `api_key`, `apikey`, `access_key`, `client_secret`, `private_key`, `auth`, `credential` that get a literal value.

In code the value is a quoted literal, which skips `password := cfg.Password`:

```
(password|passwd|pwd|secret|token|api[_-]?key|access[_-]?key|client[_-]?secret|private[_-]?key|auth|credential)\w*\s*[:=]\s*["'`][^"'`\s]{8,}["'`]
```

In env and config files (`.env*`, `.yaml`, `.yml`, `.toml`, `.ini`, `.properties`, `.json`) the value usually has no quotes:

```
(PASSWORD|PASSWD|PWD|SECRET|TOKEN|API_?KEY|ACCESS_?KEY|CLIENT_?SECRET|PRIVATE_?KEY|AUTH|CREDENTIAL)\w*\s*[=:]\s*[^\s$]{8,}
```

A value that reads from somewhere else, such as `${JWT_SECRET}` or `os.Getenv("TOKEN")`, is not a leak.

Long random strings (32 or more characters of mixed letters and digits) in code or config deserve a look even with a neutral name.

### Secrets written to logs or output

A secret printed to a log ends up in log storage, terminals and screenshots. In the changed lines, check every log, print and error message:

- Go: `log.*`, `fmt.Print*`, `fmt.Errorf`, `slog.*`, `zap`, `logrus`.
- JavaScript and TypeScript: `console.*`, `logger.*`.
- Python: `print`, `logging.*`, `logger.*`.
- Others: `System.out`, `println!`, `puts`, `echo` in shell scripts, `set -x` in scripts that handle secrets.

Report any that print:

- a variable or field named like a secret (`password`, `token`, `secret`, `apiKey`, `authorization`, `cookie`, `dsn`, `credentials`, `privateKey`);
- a whole config struct or object (`%+v`, `%#v`, `JSON.stringify(config)`), which carries every secret inside it;
- whole request or response headers (`r.Header`, `req.headers`), which carry `Authorization` and `Cookie`;
- a database DSN or connection string, including inside an error message.

## 3. How to judge a finding

| Level | Meaning | Examples |
|---|---|---|
| Real | A credential that works somewhere | an API key, a production password, a private key, a `.env` with real values |
| Review | Could be real, Joel has to look | a long random string, a token-shaped value in a test, a log that prints a config struct |
| Local default | Known throwaway value for a local container | `root`/`root`, `scheduler:scheduler@tcp(localhost:...)`, `changeme`, `example.com` |

Report local defaults too, in one line at the end, so Joel knows they were seen. Never mark a finding as safe only because the file is a test: test fixtures leak real keys too.

## 4. Report and ask

Never print a secret in full, not even in the report: show the first 4 characters and mask the rest (`AKIA****`, `ghp_****`).

If nothing was found, say so in one line, with the number of commits and files checked, and go on with the push.

If something was found, stop before pushing and show:

```
Pre-push check: 3 commits, 12 files.

Real
- executor-service/.env.local (added in a1b2c3d): DATABASE_DSN with a password (prod****)

Review
- internal/service/worker.go:88 (e4f5a6b): slog.Info prints the whole config struct, which holds the DSN

Local defaults
- docker-compose.yml: scheduler/scheduler for the local MySQL
```

Then ask with the question tool whether to push anyway or stop to clean it. Push only on an explicit yes.

## 5. How to clean it

Only when Joel chooses to clean. The commits are not pushed yet, so rewriting them is safe:

- Last commit: fix the file, `git add`, `git commit --amend --no-edit`.
- Older commits: `git reset --soft "$base"` and commit again, or remove a file from every commit in the range with `git filter-branch --index-filter 'git rm --cached --ignore-unmatch <file>' -- "$base..HEAD"`. Make a backup branch first.
- Add the file to `.gitignore` so it does not come back, and keep it on disk.

If the secret was already pushed anywhere, or shared in any other way, rewriting history is not enough: tell Joel to rotate it first.
