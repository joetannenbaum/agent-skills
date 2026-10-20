---
name: Update Docs
---

Examine the PR diff, then examine that section of the docs in the repository. If there is a mismatch between the documentation (something has changed, something is missing, something was removed):

- Docs live at /Users/joetannenbaum/Dev/docs
- Ensure git state is clean, abort if it is not
- Check out the default branch
- Pull the any changes from the remote
- Create a new branch with the correct origin for the docs change (git switch -c)
- Amend the docs accordingly using a voice consistent with the existing docs
- Wait for me to approve, do not commit, do not push
