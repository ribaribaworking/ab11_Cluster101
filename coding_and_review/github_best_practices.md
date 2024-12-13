# GitHub Best Practices Guidelines

## GitHub Projects

- GitHub Projects are extremely helpful in tracking the overall project status and in collaborations
- They allow us to write Items (Issues; Tickets) with detailed descriptions, track the development progress (Item status), assign tasks to individual developers, etc.
- They serve as the main project documentation for code development together with README
    - For example, we can link individual branches and commits to the GitHub Project Item using the Item # id and get all the information used to make decisions during the development

### GitHub Project Items

- The individual GitHub Project *tickets* are called **Items**
    - Items, if assigned to a GitHub Repository, can be further structured as **Issues** (**Features**) or **Sub-Issues** (**Subtasks**)
    - Note: In code development, GitHub Items would be called **Tickets**
- Item without assigned Repository is a **Draft** ticket
    - We can also create draft tickets on purpose - a quick idea with a simple description that is not ready to be started to work on, but we want to remember
    - We can prefix it with `Draft:` to make it obvious it is not a complete ticket
    - This helps, for example, to distinguish Items with unassigned Repository by accident from those that really are drafts
- Assigning a Repository to the Item *copies* the Item to the tagged Repository and crates an **Issue**
    - Repository-assigned tickets should have a complete description (see below)
- If, for some reason, you make a mistake and the Item is not ready, remove the tagged Repository and move the Item back to Draft (should happen automatically)
- Before creating a new Item, look into the Project columns and look for potentially similar Items to avoid duplication
    - If duplication still happens, comment on the Item with a link to the *other* duplicated Item and delete it

#### Ticket Types

- Issue name should be a brief description of the actual task that it solves
- We also add a task prefix to further highlight the ticket type and for faster orientation in the commit history:
    - **`feature`** (or Issue on GitHub) - main tickets for the new functionality. This is what we aim to implement and include in the *main* part of the code. The Feature description includes the overall goal, relevant literature, examples, etc. The Feature itself can be divided into Subtasks (Sub-issues). In [Code Review](./code_review.md), we review both Feature and Subtask PRs. If we use the `subtask`->`feature`->`main` development style, most development should be done on the Subtask level and complete tasks should be merged into the Feature branch. We can add changes on the Feature level as well if necessary. The complete Feature is then merged into the `main` branch. If there are no changes on the Feature level, the reviewer can (but doesn't have to) approve the Feature level merge directly. This assumes **all the `subtasks`->`feature` merges** were previously reviewed.
    - **`subtask`** (or Sub-issue on GitHub) - These are detailed *chunks* of work that together *define* a Feature. Most development is usually done here using the `subtask`->`feature`->`main` development style. Individual Subtasks are merged into the Feature. Ticket description of Subtasks further develops the general description of the Feature Item. As mentioned above, we also do CR on `subtask`->`feature` PR.
    - **`experiment`** - tickets that **do not change the code** but include experiment-/analysis-specific files that we want to keep track of. For example, config files for the latest diagnostic run. Experiment-specific files should **not be included in the `main` code**. They will stay in their `experiment` branch. `experiment` branches are not deleted.
    - **`test`** - tickets primarily for testing and trial purposes. For example, trying new settings or tools, experimental combinations, etc. The code **will not be included** in the `main` and the branch **might be deleted**. These branches can include additional test results required for the CR.
    - **`janitor`** - smaller code improvements, README updates, and other smaller fixes that are not crucial for the code execution but improve or complete the code. These changes can be cumulative over, for example, a month based on continuous observations or a collaborator's input.
    - **`bugfix`** - correction of a *bug* found during the development or testing phases. Bugfixes are typically planned and integrated into regular updates, addressing less urgent problems than hotfixes.
    - **`hotfix`** - tickets with crucial bug fixes that must be implemented ASAP. They can bypass the regular development process. These should be used only if absolutely necessary. Use **`janitor`** or **`bugfix`** for regular code updates, smaller improvements, or fixes.
    - **`release`** - ATM, we don't have code releases
- Note: You can decide on what level you want to work and how to structure the *merging* order:

 1. `subtask`->`feature`->`main`
 2. `feature`->`main`
 3. `subtask` -> `main`
    - The decision should be based on the issue *size* and urgency
        - For smaller tasks that add new functionality, you can do `feature`->`main`
        - For bigger tasks, it is recommended to go `subtask`->`feature`->`main`
        - The third option is usually used if we collaborate on a project with another person and the finished Subtask makes it easier for the other person to continue the development.

#### Items (Tickets/Issues) Description

- Ticket description should include enough information so another person can start working on the task without any (~much) additional information if necessary
- As we do not do many collaboration projects ATM, writing a detailed ticket description might sound like too much overhead, but it is still recommended for keeping track of what has been done, what needs to be done, why and how
- Some Tickets types might not need full description if they are *simple* enough so the commit/PR message can describe them well enough
    - For example, `janitor` or `hotfix` tickets

#### Sub-Issues (Subtasks)

- GitHub now offers creating Subtasks (Sub-Issues) automatically
    - When writing the Feature description, you can use check-box markdown notation `- [ ]`  in the description *body* and click on "*Create Sub-issue*" on the right
- You can also assign an existing Item as a Sub-Issue ("*Add existing issue*") or create a Sub-Issue with "*Create sub-issue*"

#### Ticket status in GitHub Project

- GitHub Project offers *status columns*
    - These are useful when we need to fully track the development process (for example, for ISO certification) when multiple people collaborate on the same project or when we want to keep track of where we left off if we have to switch between projects
- As we do not follow any ISO (we might have to follow IVDR regulations in the future), and we don't have many collaboration projects, following GitHub Project status might not be necessary
    - It is still recommended to use this functionality to better track the project progress and, for example, to enable better project handover
- See [GitHub Setup - GitHub Project Ticket status](./TODO_github_setup.md#github-project-ticket-status) for more details

## `git` Branches

- We do **all the development** in branches
- Branches can be reviewed, deleted, reverted, merged
- Committing everything into the parent (~`main`) branch directly makes the `git` history difficult to read, very difficult to revert if necessary, doesn't allow version releases properly, and is generally a bad practice

### Branch Naming

- Name your current development branch the same as the Project Item, including the ticket number
    - This way, it's easy to find the full task description and track the origin of the code
- The easiest is to use the default GitHub functionality to create the ticket branch directly from the ticket options
    - GitHub then names the branch according to the Issue name
- **Don't forget** to change the *parent* branch to the Feature branch when using this GitHub functionality!
    - It avoids unnecessary `git rebase` commands and conflict resolution
- Don't forget to include the Ticket prefix and number as described above if you create the branch yourself

#### Branch Name Structure

- The branch name structure: `<ticket number id>-<ticket type>-<rest-of the-ticket-name>`
- Regex: `^\d+-(feature|subtask|experiment|test|janitor|bugfix|hotfix|release)-([a-zA-Z0-9]+(-[a-zA-Z0-9]+)*)$`
- For example: `4-subtask-assemble-chromosomes`

## Commits

- Commits have to have a meaningful message so it's clear what has been done
    - It makes it easier for Code Review, follow the code changes, and revert the code if necessary
- Commit **messages** are written in [**imperative**](https://cbea.ms/git-commit/#imperative) verb form
    - For example: "*Fix missing variable*" and **not** "*Fixed missing variable*" or "*Fixes missing variable*"
    - Note: An easy way to remember is to say to yourself: "*If applied, my commit will (INSERT COMMIT MESSAGE TEXT)*"
- Use the ticket number, including the hash symbol `#` as the first character of the commit message
    - This allows easier traceability (you can directly click the ticket number in the commit to see the ticket description)
- The next word of the commit message starts with a capital letter
    - This makes it clear the commit message is complete and as intended
- More on how to write a good commit message [here](https://cbea.ms/git-commit/) and [here](https://www.freecodecamp.org/news/how-to-write-better-git-commit-messages/)

### Commit Message Structure

- The commit message structure: `#<ticket number id>-<Commit message starting with a capital letter>`
- Regex:  `^#\d+-[A-Z][a-zA-Z0-9\s]*$`
- For example: `#14 Filter unwanted reads`

### Commit Message Verbs

- One action can be described using different verbs with the same meaning
    - Using different verbs for the same action might be confusing when the commit history gets longer - try to stick to the same set of verbs
- This improves clarity and consistency
- This is a **non-exhaustive** list

#### General Verbs for Git Commit Messages

1. **Initialize** - For initial setup of a repository or a new
   - Example: `Initialize repository`
2. **Add** - For adding new files, features, or functionality.
   - Example: `Add login feature`
3. **Update** - For making non-breaking changes or enhancements.
   - Example: `Update documentation`
4. **Modify** - For making changes to existing code or resources.
   - Example: `Modify user validation logic`
5. **Improve** - For optimization or quality improvements.
   - Example: `Improve performance of sorting algorithm`
6. **Fix** - For bug fixes or addressing issues.
   - Example: `Fix crash on app startup`
7. **Remove** - For deleting files, features, or functionality.
   - Example: `Remove deprecated API endpoint`
8. **Refactor** - For code restructuring without changing functionality.
   - Example: `Refactor authentication module`
9. **Rename** - For renaming files, variables, or components.
   - Example: `Rename User class to Account`
10. **Reorganize** - For changing the project *higher* structure.
   - Example: `Reorganize project directory`
11. **Deprecate** - For marking features as outdated.
   - Example: `Deprecate legacy logging system`

#### Verbs for Reversals or Cleanup

1. **Revert** - For undoing previous commits.
   - Example: `Revert changes to login feature`
2. **Clean** - For cleaning up code or resources.
   - Example: `Clean unused imports`

#### Verbs for Experimental or Prototyping Work

1. **Prototype** - For initial experimental code or features.
   - Example: `Prototype new UI layout`
2. **Experiment** - For testing new ideas or approaches.
   - Example: `Experiment with caching mechanism`

#### Specialized Verbs

1. **Document** - For updating or adding documentation.
   - Example: `Document API usage`
2. **Test** - For adding or modifying test cases.
   - Example: `Test edge cases for login`
3. **Build** - For changes related to build or deployment systems.
   - Example: `Build production release`
4. **Configure** - For changes to configuration files or settings.
   - Example: `Configure database connections`
5. **Enhance** - For minor upgrades to functionality.
   - Example: `Enhance error message display`
6. **Secure** - For adding or improving security measures.
   - Example: `Secure API endpoint with authentication`
7. **Optimize** - For performance improvements.
   - Example: `Optimize image loading`
8. **Translate** - For adding or updating translations.
   - Example: `Translate application to French`

## Pull Request

- PR is the last step in code development before merging the code into the parent branch
- The PR has a title and a final commit message

### Pull Request Title

- The PR title should be named according to the Ticket name, including the ticket prefix and the ticket `#` at the end
- An exception to this rule is when the main implemented Feature(s) significantly differs from the original ticket name and content
    - You can also consider changing the Ticket title, but be cautious not to break any links

#### Pull Request Title Structure

- The RP title structure: `<feature ticket type>-<Rest of the feature ticket name> #<ticket number id>`
- Regex: `^(feature|subtask|experiment|test|janitor|bugfix|hotfix|release)-[A-Z][a-zA-Z0-9\s]*\s#\d+$`
- For example: `feature-Implement performance testing #1`

### Pull Request Commit Message

- The PR commit message should be written **in third person** and **summarize what has been implemented or what it does** or what is being added by the PR to the main code.
- Do not confuse the **PR commit message** with **PR comment** (not included in the git history)
    - Unfortunately, GitHub puts PR comment on the top, making it slightly less obvious what is what
    - You can use the PR comment section to make notes and keep track of the development and then copy the text (or part of it) to the PR commit message
    - Note: You can use the `- [ ]` markdown check-box option to keep track of your own development progress on both Subtask and Feature levels
- For a Subtask merge, the PR commit message can be a sentence or bullet points summarizing the commits
- For a Feature, this can be a list of included Subtasks (similar to the PR message) + the description of the additional commit at the feature level (on the top of the PR commit message):

```git
- Removes unused dependencies

- Includes:
  - [x] subtask-Reference datasets #2
  - [x] subtask-Performance testing framework #3

closes #14
```

- You can use the following [GitHub keywords](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue#linking-a-pull-request-to-an-issue-using-a-keyword) in the **PR commit message**
    - We primarily use `closes #<ticket_number` to highlight this PR closes the particular ticket
    - GitHub also automatically closes both Issue and the GitHub Project Item when the PR is merged into the parent branch (merge also closes the PR - GitHub treats PR requests as Issues)
        - You can also close the Issues manually by clicking on `Close issue` in the Issues tab and by moving the GitHub Project Item to "*Done*"
- More on how to write a good PR commit message [here](https://medium.com/@koklitheen/writing-a-good-merge-request-3916cfce0518) - you can be as verbose as necessary
- [Squashing the commits](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/configuring-commit-squashing-for-pull-requests) before merging to the `main` or parent branch is strongly encouraged
    - This keeps the history of the parent branch clean and [linear](https://stackoverflow.com/questions/20348629/what-are-the-advantages-of-keeping-linear-history-in-git)
- You can decide whether you want to **keep** or **delete** the **development** branch after the merge:
    - *Advantage of keeping*: keeps a detailed history of changes and easier tracking of feature development
    - *Disadvantage of keeping*: potential clutter and increased complexity in branch management
    - For example, we don't have to keep branches with simple single tasks

## GitHub Rules

- GitHub offers a lot of [rules/rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) to help us enforce the agreed *rules* for commit names, branch names, PR, etc.
- We primarily use these rules:

 1. Commit message structure
 2. Branch name structure
 3. PR title structure
 4. `main` branch protection
 5. CR rules (resolving threads, approval, merging)

- See more information in [GitHub Setup](./TODO_github_setup.md)
- Note: In the future, we will include [GitHub Actions](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions) to include tests in the PR approval rulesets

## Code publishing

- Code on GitHub can be changed and/or deleted
- To permanently publish and share your code, you can use one of the following:
	- Zenodo
	- FigShare
- Both provide **DOI to your code** - *freeze* the software version and allow citations