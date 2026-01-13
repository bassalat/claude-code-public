---
name: audit-sensitive
description: Audit repository for sensitive information before pushing. Checks for emails, credentials, client names, PII, and other data that shouldn't be public. Run before every push to ensure nothing sensitive leaks.
---

# Sensitive Information Audit

Audit the current repository for sensitive information that shouldn't be pushed to a public (or any) remote. This command helps prevent accidental exposure of credentials, client information, and PII.

## Core Principle

**Assume everything will be public.** Even private repos can be cloned, forked, or accidentally made public. Audit as if your repo will be on the front page of Hacker News.

---

## Audit Process

### Step 1: Discover All Files

Use Glob to find all text files in the repository:

```bash
**/*.md
**/*.json
**/*.yml
**/*.yaml
**/*.txt
**/*.py
**/*.js
**/*.ts
**/*.env*
**/*.config*
**/*.ini
**/*.cfg
```

Exclude `.git/` directory from all searches.

---

### Step 2: Run Pattern Searches

Execute each category of searches using Grep. Track results in a checklist.

#### 2.1 Credentials & Secrets

| Pattern | What It Catches |
|---------|-----------------|
| `(api[_-]?key\|secret\|token\|password\|credential)[^\s]*\s*[=:]\s*["']?[a-zA-Z0-9_-]{10,}` | Hardcoded API keys and secrets |
| `(aws_access_key_id\|aws_secret_access_key\|AKIA[0-9A-Z]{16})` | AWS credentials |
| `(ghp_[a-zA-Z0-9]{36}\|github_pat_[a-zA-Z0-9_]{22,})` | GitHub tokens |
| `-----BEGIN (RSA\|DSA\|EC\|OPENSSH) PRIVATE KEY-----` | Private keys |
| `(mongodb(\+srv)?:\/\/\|postgres:\/\/\|mysql:\/\/\|redis:\/\/)` | Database connection strings |

#### 2.2 Email Addresses

| Pattern | What It Catches |
|---------|-----------------|
| `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` | Email addresses |

**Exceptions:** Ignore `noreply@anthropic.com` and similar generic addresses.

#### 2.3 Personal Identifiable Information (PII)

| Pattern | What It Catches |
|---------|-----------------|
| `\b\d{3}[-.\s]?\d{2}[-.\s]?\d{4}\b` | SSN patterns |
| `\+?\d{1,3}[-.\s]?\(?\d{2,4}\)?[-.\s]?\d{3,4}[-.\s]?\d{3,4}` | Phone numbers |
| `\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b` | IP addresses |
| `(street\|avenue\|blvd\|road\|drive).*\d{5}` | Physical addresses with zip |

#### 2.4 Client & Business Identifiers

| Pattern | What It Catches |
|---------|-----------------|
| `(Inc\.\|LLC\|Corp\.\|Ltd\.\|Limited\|Company\|Co\.)` | Company legal names |
| `(client\|customer\|account)[_-]?(id\|name\|number)` | Client ID patterns |
| `(campaign\|project)[_-]?(name\|id)` | Project identifiers |

#### 2.5 Cloud & Infrastructure

| Pattern | What It Catches |
|---------|-----------------|
| `(s3:\/\/\|gs:\/\/\|az:\/\/)` | Cloud storage URLs |
| `[a-z0-9-]+\.(s3\|blob\.core\|storage\.googleapis)\.` | Cloud bucket names |
| `arn:aws:[a-z0-9-]+:[a-z0-9-]*:\d{12}:` | AWS ARNs |

#### 2.6 File System Leaks

| Pattern | What It Catches |
|---------|-----------------|
| `/Users/[^/]+/` | macOS user paths |
| `/home/[^/]+/` | Linux user paths |
| `C:\\Users\\[^\\]+\\` | Windows user paths |

#### 2.7 Internal URLs

| Pattern | What It Catches |
|---------|-----------------|
| `https?://[^\s]*\.(internal\|local\|corp\|private)` | Internal domain URLs |
| `https?://localhost` | Localhost references (usually fine, but flag) |
| `https?://\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` | URLs with IP addresses |

---

### Step 3: Check Git History

#### 3.1 Author Email Exposure

Run:
```bash
git log --all --format='%an <%ae>' | sort -u
```

Flag any personal email addresses that will be visible on the remote.

**Remediation:** Use GitHub's private email: `username@users.noreply.github.com`

#### 3.2 Commit Message Review

Run:
```bash
git log --all --format='%B'
```

Search commit messages for:
- Client names
- Internal project references
- Credentials accidentally committed

#### 3.3 Deleted File Check

Run:
```bash
git log --all --diff-filter=D --name-only --pretty=format:
```

Review if any deleted files might have contained sensitive data (still in history).

---

### Step 4: Generate Report

Output a structured report:

```markdown
## SENSITIVE INFORMATION AUDIT

**Repository**: [repo name]
**Date**: [audit date]
**Files Scanned**: X

---

### Summary

| Category | Status |
|----------|--------|
| Credentials & Secrets | PASS/FAIL |
| Email Addresses | PASS/FAIL |
| PII (SSN, Phone, etc.) | PASS/FAIL |
| Client Identifiers | PASS/FAIL |
| Cloud Resources | PASS/FAIL |
| File System Paths | PASS/FAIL |
| Git History | PASS/FAIL |

---

### Findings

#### [Category Name]
- **Location**: `file.md:123`
- **Found**: [what was found]
- **Risk**: [why this is a problem]
- **Remediation**: [how to fix]

---

### Verdict

**CLEAN** - Safe to push
or
**ISSUES FOUND** - Address findings before pushing
```

---

## Safe Patterns (Ignore List)

These patterns are typically safe and can be ignored:

| Pattern | Why Safe |
|---------|----------|
| `noreply@anthropic.com` | Generic service email |
| `example.com`, `example@test.com` | RFC 2606 reserved domains |
| `127.0.0.1`, `localhost` | Local development (usually fine) |
| `0.0.0.0` | Bind-all address |
| Generic placeholder IPs like `192.168.x.x`, `10.x.x.x` | Private ranges in examples |

---

## Quick Reference Commands

For manual verification:

```bash
# Find all emails
grep -rE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' --include="*.md" --include="*.json" .

# Find potential API keys
grep -rE '(api[_-]?key|secret|token).*[=:]' --include="*.md" --include="*.json" .

# Check git authors
git log --all --format='%ae' | sort -u

# Find hardcoded paths
grep -rE '/Users/|/home/|C:\\Users\\' .
```

---

## When to Run

Run this audit:
- Before every `git push`
- Before making a private repo public
- After adding new collaborators
- When onboarding to a new project
- Periodically as a sanity check

---

## Remediation Strategies

### For Credentials in History
Use BFG Repo-Cleaner or `git filter-branch` to remove from history, then force push (coordinate with team).

### For Exposed Git Email
```bash
git config user.email "username@users.noreply.github.com"
```

### For Sensitive Files
Add to `.gitignore` and remove from tracking:
```bash
echo "sensitive-file.json" >> .gitignore
git rm --cached sensitive-file.json
```

### For Client Names in Content
Replace with generic placeholders: "Acme Corp", "Client A", "[REDACTED]"
