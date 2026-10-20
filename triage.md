---
name: Triage
---

Figure out what repo I'm on and look at open issues. Pick the first open one and concisely analyze it:

- Is this a real problem experienced by a broad number of users?
- Is it an actual bug or a configuration/usage issue? If it's config/usage, are we able to make it clearer?

First: Tell me the issue number, who filed the issue, the date it was filed, and provide a direct link.

Provide your analysis and then ask if I want to tackle the issue. If I want to:

- Make sure the current branch is clean. If not, abort.
- Check out the default branch (or the target branch if the issue is referencing a different version associated with a specific branch).
- Create a branch for the fix (git switch -c)
- Create a fix for the issue
