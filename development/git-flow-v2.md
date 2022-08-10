# A git flow that actually works

## TL; DR

## Branches

There are 3 kind of banches:

* **👑 Stable**
  * There are always only 2: `main` and `dev`.
  * No forced pushes allowed.
  * `dev` is developer's main branch. It must no contain bugs, it can be pushed to production anytime.
  * `main` acts as a pointer to deploy code to production.
* **🚧 Epic**
  * There are as many as concurrent epic the team is working on.
  * They have a short and easy to remember name. Eg `signup`, `feature-x`, `add-cache`.
  * Forced pushes are allowed.
  * Squash is forbidden.
  * Deleted after the epic is integrated to `dev`.
* **🛠 Task**
  * There are as many as needed.
  * Name is composed by its _ticket ID_, plus optionally the _task name_. Eg. `JIRA-1234`, `asana-5678-endpoint`.
  * Forced pushes are allowed.
  * Deleted after the task is integrated to `dev` or an `epic` branch.
  * Squash is required before merging.
  * These are **the only branches that require a peer review** 🔍.

| Branch kind | Naming | Forced pushes? | Squasing? | Peer review? |
|-------------|--------|----------------|-----------|--------------|
| Stable | `main` and `dev` | ❌ | ❌ | ❌ |
| Epic | `[short-and-rememberable]` | ✅ | ❌ | ❌ |
| Task | `[ticket id] + [task name]` | ✅ | ✅ | ✅ |

## Process

### Coding a task

#### 1. Checkout the branch

* **Is this a hotfix?:** Checkout from `main`.
* **The task belongs to an epic?:** Checkout from its `epic` branch.
* **Is this a standalone task?:** Checkout from `dev`.

```sh
# Make sure you have the latest changes
git pull <branch_you_are_checking_out_from>

# Create the branch
git checkout <new_branch_name>
```

Example:

```sh
git pull dev
git checkout JIRA-1234-my-task
```

[More info about checking out a branch here](https://support.atlassian.com/bitbucket-cloud/docs/check-out-a-branch/)

#### 2. Code and commit whatever you want

> 💡 **Pro tip:** When squahing, the first commit description will be the only that will remain. So, **make a good first commit description** from the begining.

* A good convention for commit description is: `[Feature | Improvement | Bugfix]: [Short description about the changes]`.
* You may want to split your code in several commits to facilitate peer review. For exmaple:
  * 1st commit: Create database migrations.
  * 2nd commit: Add model logic.
  * 3rd commit: Add tests. 

```sh
# Stage your changes
git add .

# Commit them
git commit -m "<commit_description>"

# Push the changes
git push <task_branch>
```

Example:

```sh
git add .
git commit -m "Feature: Creates User model"
git push origin JIRA-1234-user-model
```

Code and commit the times you need.

#### 3. Send to peer review

