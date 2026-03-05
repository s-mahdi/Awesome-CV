---
name: build
description: Build the resume PDF and report any LaTeX errors
user-invocable: true
allowed-tools: Bash(make resume.pdf:*), Bash(make clean:*)
---

# Build Resume

Run `make resume.pdf` to compile the resume. Report whether it succeeded or failed.

If the build fails:
1. Show the relevant LaTeX error lines
2. Suggest a fix based on the error message
3. After fixing, rebuild to confirm

If the build succeeds, confirm with a short message.
