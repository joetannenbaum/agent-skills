---
name: GH Actions Fail
---

DO NOT COMMIT. DO NOT PUSH.

Given the URL to a GitHub actions failure (auto detect if possible):

1. Determine the branch associated with the failure
2. Make sure current branch is clean, otherwise stop and inform user
3. Check out the correct branch
4. Assess the failure and apply the fix, DO NOT COMMIT
