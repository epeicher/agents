---
name: pr-writer
description: Writes clear, punchy, human-oriented PR descriptions that focus on what matters. Use this skill whenever the user asks to write or draft a PR description, write a PR body, describe a pull request, or summarize a branch's changes for review, including phrasings like "write a PR description", "draft the PR", "describe this PR", "what should the PR say", or "rewrite the PR body" (even if they don't say "skill" explicitly). Follows the repo's PULL_REQUEST_TEMPLATE when present, otherwise a What it does / Rationale / Implementation / Testing instructions structure.
---

When writing PR descriptions, be concise and structured. Write the right things, not a lot of things.

## Process

First, check for a repo PR template (`PULL_REQUEST_TEMPLATE.md` in `.github/`, the repo root, or `docs/`; or a `.github/PULL_REQUEST_TEMPLATE/` directory of named templates). If one exists, follow its structure instead of the default sections below, filling in every section it asks for and applying the same writing guidelines.

Then compare the branch against the default branch (not just the last commit):

```bash
git diff main...HEAD
git log main..HEAD --oneline
```

Replace `main` with the repo's default branch (`trunk`, `master`, etc.). This shows the full scope of changes, not just recent commits.

## Structure

When there's no PR template, default to these four sections, each kept focused:

- **What it does**: the user-visible behavior change, in a sentence or two.
- **Rationale**: the why (the bug being fixed, the race condition, the missing capability).
- **Implementation**: how it works now, with code for the non-obvious parts.
- **Testing instructions**: the concrete steps or test command a reviewer runs to verify.

Use markdown formatting to highlight important details.

## Writing Guidelines

**Be specific.** Use inline code for technical terms: `TokenRefresher`, `SESSION_TIMEOUT`, `/api/v2/auth`.

**Be concise.** Every sentence should add new information. Cut the fluff.

**Use formatting.** `code`, **bold**, _emphasis_, and code blocks make descriptions scannable.

**Show, don't tell.** Code examples beat vague descriptions.

## Examples

### Good: Focused and Structured

````markdown
## What it does

Prevents users from getting logged out mid-navigation. The `TokenRefresher` now blocks route transitions until any in-flight token refresh completes.

## Rationale

Token expiration and navigation could race. If your token expired at `t=0`, navigation started at `t=10ms`, refresh completed at `t=150ms`, the API call would use the expired token and redirect to login mid-flow.

## Implementation

Added `waitForRefresh()` to `NavigationGuard`. When navigation starts:
1. Check if `TokenRefresher.isRefreshing`
2. If true, await the refresh promise (max 200ms)
3. Proceed with fresh token

```typescript
async canActivate(): Promise<boolean> {
  if (this.tokenRefresher.isRefreshing) {
    await this.tokenRefresher.currentRefresh;
  }
  return this.auth.isAuthenticated();
}
```

Considered making API calls retry with new tokens instead, but that's complex for non-idempotent requests.

## Testing instructions

1. Set `SESSION_TIMEOUT=30` in `.env.local`
2. Wait 25 seconds, then navigate between routes rapidly
3. Verify no login redirects occur
4. Check network tab shows refresh completing before route API calls
````

### Bad: Generic AI Slop

````markdown
## Summary
This PR enhances the authentication system with improved session management capabilities and robust error handling mechanisms! 🚀

## Key Changes
- Enhanced token refresh functionality
- Improved navigation flow
- Better race condition handling
- Optimized user experience
- Added comprehensive error handling

## Technical Implementation
Leveraged modern authentication patterns to implement a scalable, enterprise-grade solution for token lifecycle management. The system now handles edge cases more effectively and provides enhanced reliability.

## Benefits
- Reduced session timeouts
- Better performance
- More robust authentication
- Enhanced security
- Improved developer experience

## Testing
- [ ] All tests pass
- [ ] Manual testing completed
- [ ] No regressions found

🤖 Generated with Claude Code
````

**Why it's bad:**
- No specific details about what changed
- Buzzwords: "enhanced," "improved," "robust," "scalable," "enterprise-grade"
- Emojis and bot signatures
- Bullets listing vague improvements
- No code examples or technical specifics
- Doesn't explain the actual problem or solution

### Good: Bug Fix with Code

````markdown
## What it does

Fixes bookmark invalidation when the document changes between creating and seeking to a bookmark.

## Rationale

`Bookmark` stored direct references to `HTMLNode` objects. If you inserted text before the bookmark position, the node reference stayed valid but pointed to the wrong content.

```typescript
// Before: breaks when document is modified
class Bookmark {
  node: HTMLNode;
  constructor(node: HTMLNode) {
    this.node = node;
  }
}
```

## Implementation

Changed bookmarks to store character offsets instead of node references. When seeking, we walk the document from the start counting characters until reaching the offset.

```typescript
// After: resilient to document changes
class Bookmark {
  offset: number;
  constructor(doc: HTMLDocument, node: HTMLNode) {
    this.offset = doc.getCharacterOffset(node);
  }
  seek(doc: HTMLDocument): HTMLNode {
    return doc.getNodeAtOffset(this.offset);
  }
}
```

This matches how browser `Selection` APIs work.

## Testing instructions

Run `npm test -- bookmark.test.ts`. New tests cover:
- Creating bookmark, inserting text before it, seeking to bookmark
- Multiple bookmarks in same document
- Bookmarks across element boundaries
````

### Good: Feature Addition

````markdown
## What it does

Adds `parseFragment()` for parsing HTML snippets without a full document context.

## Rationale

`parseDocument()` requires `<html>` and `<body>` tags. For parsing HTML that will be inserted via `innerHTML` (like `<tr><td>Cell</td></tr>`), we need fragment parsing that infers context.

## Implementation

Added `parseFragment(html, contextElement)` that creates a temporary parsing context based on where the fragment will be inserted:

- Context is `<table>` → use "in table" mode
- Context is `<div>` → use "in body" mode
- Context is `<select>` → use "in select" mode

Special handling for orphaned elements like `<tr>` or `<option>` that are only valid inside specific parents:

```typescript
if (isOrphanedTableRow(fragment)) {
  return parseFragment(fragment, document.createElement('tbody'));
}
```

This matches the HTML5 fragment parsing algorithm.

## Testing instructions

```bash
npm test -- fragment-parser.test.ts
```

Tests cover `<tr>`, `<option>`, `<li>`, `<td>` fragments and verify they're wrapped correctly.
````

## What to Avoid

**Don't:**
- Use emojis (🚀, 🎉, ✅)
- Write "enhanced," "improved," "optimized" without specifics
- List files changed (that's what the diff shows)
- Add bot signatures
- Use corporate buzzwords: "leverage," "synergy," "robust," "scalable," "enterprise-grade"
- Write "key functionalities," "core capabilities," "key improvements"
- Create bullet lists of vague changes
- Make every section 5+ paragraphs

**Do:**
- Use inline `code` for technical terms
- Show code examples for complex changes
- Explain **why** decisions were made
- Keep it scannable with formatting
- Focus on what matters
- Compare against the default branch, not last commit
