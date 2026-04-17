# Contributing to the Housing Affordability Calculator

Thanks for your interest in contributing! This project is open to community improvements, bug fixes, and feature ideas. 
To keep the project maintainable and consistent, please follow the guidelines below.

## 🧭 How to Contribute

There are several ways to contribute:

- Reporting bugs
- Suggesting new features
- Improving documentation
- Submitting code changes via pull requests

Before contributing code, please read the workflow below.

---

## 🔀 Fork & Pull Request Workflow

Please keep pull requests focused on a single change. Large or multi‑purpose PRs are harder to review and may be requested to be split.

This repository uses a **fork → branch → pull request** workflow for all external contributors.

1. **Fork** the repository to your own GitHub account  
2. **Clone** your fork locally
    ## Keeping Your Fork Up to Date

    Before starting any work, make sure your fork is synced with the latest changes from the main repository. 
    This prevents merge conflicts and ensures your pull request is based on the most recent code.
    From your local main fork directory,

    1. Add the upstream remote (only needed once): git remote add upstream https://github.com/weaseljohnson/housing-affordability-calculator.git
    2. Fetch the latest changes: git fetch upstream
    3. Update your local main branch: git checkout main, git merge upstream/main
    4. Push the updated main branch to your fork: git push origin main

    Now you can safely create a new feature branch from an up‑to‑date main.

3. Create a new branch for your change: git checkout -b feature/my-new-feature
4. Make your changes  
5. Commit your work with a clear message: git commit -m "Add feature: description of change"
6. Push your branch to your fork: git push origin feature/my-new-feature
7. Open a **Pull Request** to the `main` branch of this repository

All pull requests will be reviewed before merging.

Direct pushes to main are restricted to maintainers to ensure code quality and stability.

## Pull Request Checklist

Before submitting your pull request, please confirm the following:

- [ ] My branch is up to date with `main` (see "Keeping Your Fork Up to Date")
- [ ] My changes build and run correctly locally
- [ ] I manually tested the calculator and verified the results behave as expected
- [ ] My PR is focused on a single change (no unrelated edits)
- [ ] I updated documentation if needed
- [ ] I wrote clear commit messages
- [ ] I reviewed my own diff to catch obvious issues

---

## ✔️ Code Style & Expectations

Please review the .md files in the repo: PROJECT_CONTEXT.MD AND UX_AND_CONTENT_GUIDELINES.MD

- Keep code readable and well‑commented  
- Avoid large, unrelated changes in a single PR  
- Test your changes locally before submitting  
- For UI changes, include screenshots if helpful  
- Keep HTML/CSS/JS clean and consistent with the existing style

---

## 🐛 Reporting Bugs

If you find a bug, please open a new discussion and include:

- A clear description of the problem  
- Steps to reproduce  
- Expected vs. actual behavior  
- Browser/device information (if relevant)  

---

## 💡 Suggesting Features

Feature suggestions are welcome!  
Please open a discussion describing:

- The problem the feature solves  
- Why it would be useful  
- Any ideas for implementation  

---

## 🙌 Thank You

Your contributions help improve the project and make it more useful for everyone.  
Thanks for taking the time to help!



