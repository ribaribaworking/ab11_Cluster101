# Best Coding Practices and Code Review

## Introduction

This documentation aims to standardize coding practices, improve code quality, and streamline collaboration within the Bioinformatics Core Unit (BiCU) at St. Anna CCRI and with its collaborators.

It serves as a practical guideline for existing and new BiCU members. It is focused on but not limited to best coding practices in bioinformatics.

The documentation describes:

- [Best Coding Practices](./best_coding_practices.md): outlines the standards and guidelines for writing clean, efficient, and maintainable code.
- [Repository Structure](./repository_structure.md): describes the repository's organization of files and directories within the repository to ensure consistency between projects and developers.
- [Code Reproducibility](./code_reproducibility.md): provides guidelines on code reproducibility in different environments.
- [Code Review](./code_review.md): explains the process and best practices for Code Review.
- [Code Testing](./code_testing.md): covers the methods and tools for code testing.
- [GitHub Best Practices](./github_best_practices.md): offers guidelines for using GitHub effectively and efficiently. It primarily focuses on structuring tasks, commits, and pull requests. Contains some guidelines for `git` best practices.
- [WiP - GitHub setup](./TODO_github_setup.md): provides instructions on setting up GitHub for your projects and repositories, including initial configuration and settings.
- [WiP - Tips and tricks](./TODO_tips_and_tricks.md): shares tips and tricks to help with coding, Code Review and development workflow overall, simplified checklists, etc.

## Glossary

- There are some differences between naming things in *regular* code development environment and on GitHub
    - Note: [GitLab](https://about.gitlab.com/) is more consistent with the standard development jargon
- The following terms are interchangeable:
    - **Ticket** – GitHub Project Item​; turns into GitHub Issue when assigned to a Repository; A general description for a task (both Feature and Subtaks).
    - **Feature** – a chunk of code that adds functionality or solves an issue; GitHub Project Item; GitHub Repository Issue​
    - **Subtask** – a smaller chunk of code solving a particular part of the parent feature; GitHub Project Sub-Issue​; GitHub Repository Issue​
    - **Merge Request** - request to merge a development branch code into a parent branch; GitHub Pull Request
    - **Code Review** - a process of reviewing the code by your peers
    - **`main` branch** - sometimes called `production` branch; GitHub used to call it `master`
    - **Epic** – A large body of work that can be broken down into smaller tasks or features; often used in project management tools. It could be a large project overarching several independently developed products.
