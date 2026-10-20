---
name: Review Remote PR
description: Review a PR by URL without a local checkout and publish a review guide artifact
argument-hint: <pr-url>
---

PR to review: $ARGUMENTS

1. Read the PR with `gh pr view $ARGUMENTS --json title,body,author,baseRefName,headRefName,files,reviews,commits` and its diff with `gh pr diff $ARGUMENTS`. Do not check out or clone the repo. If you need more context than the diff gives, read files at the PR's head with `gh api repos/{owner}/{repo}/contents/{path}?ref={headRefName}` and compare against the base branch the same way. Understand what problem is being solved and how.

2. **Is this solving a real, universal problem?** - Evaluate whether the problem this PR addresses is something that a broad set of developers/users would actually encounter. Is it a common pain point, or is it niche/contrived? Be honest and direct — if it's solving a problem nobody really has, say so.

3. **Is the solution any good?** - Evaluate the implementation quality. Is the approach sound? Is it over-engineered or under-engineered? Are there edge cases being missed? Would you suggest a fundamentally different approach? Again, be direct — if the solution is mediocre, say so. Make sure to comment on code quality as well.

4. For each point where you feel like there's an improvement to be made, suggest the improvement explicitly. Also, make sure it's a real problem, take the time to examine the code and ensure that it's something that needs addressing. If that means re-analyzing the points one by one to double check, do it. DO NOT REPORT ANYTHING THAT IS NOT A REAL PROBLEM. If you output it as an issue, you should be POSITIVE that it is one.

5. If possible (not always possible) show how the PR is used with a realistic code example.

6. If I (joetannenbaum) have already reviewed this PR, at the end highlight what has changed since I last reviewed.

Be concise and opinionated. Don't hedge excessively. Give me a straight read.

## Review guide artifact

If the PR is big enough that a file-by-file read would be hard to follow — enough files or distinct concepts that a reader needs an ordered walkthrough — publish a review guide as an Artifact, in the style of Linear's review guides (https://linear.app/docs/diffs#guides). Small or single-concern PRs don't need one; use your judgment, there's no line count that triggers it.

The guide is a shareable review doc containing both the walkthrough and the review:

- Open with the PR title, author, repo, a link to the PR, and a short verdict.
- Order sections by what a reviewer should read first. Core implementation leads; supporting changes (tests, docs, config, mechanical renames) are grouped separately at the end.
- Each section explains *why* that part of the change exists before showing the code.
- Include the full relevant diff hunks inline, so the page stands on its own without opening GitHub. Link each one to the file on GitHub.
- When a diff line is its own block element (e.g. `display: block` spans) inside a `<pre>`, join the lines with no newline between them. A `<pre>` keeps every `\n`, so newline-joined block lines render with a blank line between each one.
- Fold in the highlights and lowlights from the review above — put each finding in the section it belongs to, so the issue sits next to the code it's about.

End with the review in text. If you published an artifact, follow it with the artifact URL on its own line.
