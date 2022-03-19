# Git Flow

## TL; DR

Use 3 kind of branches:

|Branch kind|Naming|Integration strategy|Forced pushes?|
|-----------|------|--------------------|--------------|
|Stable|A fixed name. Eg. `main`, `master`, `dev`, `staging`.|Merge (`git rebase`).|No. Forbidden.|
|Epic|A short, feature related name. Eg. `signup`, `feature-x`, `add-cache`.|First rebase (`git rebase`) the stable branch (eg. `dev`), then merge to stable branch using _"no fast forward"_ option (`git merge --no-ff`).|Yes: when rebasing stable branches.|
|Task|Ticket ID + task related name. Eg. `VF1234-endpoint`, `#123456-fix-x-bug`.|Squashed rebase (`git rebase -i`), then merge to either epic or stable branch.|Yes: before integration or when integrating changes.|

## Cheat sheet

**Squashed rebase**

1. `git rebase -i`
2. This command will open a vim edit will all commits being rebased. Press `i`to start editing.
3. Leave `pick` in the first commit, change `pick` to `f` (fixup) in the rest.
4. Press `Esc` to exit editing mode.
5. Type `:wq` to save changes and leave vim.

## Intro

This guide explains the best process I've found to to develop projects using git. The process is based on [Git Flow by Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/):

![Git Flow](/assets/gitflow-model.png)

Before you start with the process, you should know some key concepts and background of our projects:

## Types of assignments

In our process, there are basically two types of assignments, Epics and Tasks:

* **Epic**: It’s a feature that due to its complexity, requires multiple tasks, even affecting different modules of the platform. An epic might be resolved by multiple developers.

* **Task**: This is a manageable and well delimited coding assignment. Tasks should be small enough to be resolved in less than 1 day (including tests) and easy to review. A task is always resolved by a single developer.

## Branches

There are three types of branches, depending on their use case:

### Main branches

Those branches are always 2: `master` and `dev`.

`master` is intended for production environment. This branch must be always ready to be deployed to production and no bugs should be present there.

`dev` branch is the integration branch for developers. Once a feature is tested and works properly it can be integrated to `dev`.

`dev` must ALWAYS stay functional and NO BUGS should be present either. For developers, `dev` branch should have enough quality to be in a production environment.

### Epic branches

This branches depends on the number of epics we are developing at the moment.

If the epic requires tasks on multiple projects, epic branches on each project must have the same name. For example, if an epic called “Epic 1” requires changes in frontend and backend projects, each of them must have a branch called `epic-1`.

The name should be a short but very descriptive of its goal. Words on names can be separated by a dash (`-`). Examples of this branches are: `stats-refactor`, `login`, `facebook-integration`.

Epic branches are always created from `dev`.

### Task branches

Each task (that might be related to an epic) must have a branch (1 task -> 1 branch).

The name of these branches is the ID of the task. You can find this ID in our task manager (Jira). It usually has the format: `VF-####`, where #### represents a number. Examples are: `VF-1`, `VF-1234`.

Task branches are checked out from `dev` in case of isolated tasks, or from its epic branch in case of tasks related to one.

## Process for Tasks

[1. Create a new branch](#1-create-a-new-branch)

[2. Commit your code](#2-commit-your-code)

[3. Send to peer review](#3-send-to-peer-review)

[4. Make sure tests are passing](#4-make-sure-tests-are-passing)

[5. Rebase and Squash](#5-rebase-and-squash)

[6. Integrate to epic branch](#6-integrate-to-epic-branch)

[7. Summary](#7-summary)

Once you know the key concepts it’s time to follow the process to resolve a task. The following process is intended for tasks that belong to an epic, for isolated tasks (task itself is a feature) use [process for epics](#process-for-epics).

### 1. Create a new branch

Create a new branch from the epic branch and use task ID as name (see [Task branches](#task-branches)).

If you are resolving an epic with other developers, make sure you pull changes from the epic brach before creating the branch for the task.

For example, let say we create a branch for task `VF-1234` which belongs to `epic-1`:

```bash
git checkout epic-1
git pull
git checkout -b VF-1234
```

### 2. Commit your code

Code you task and commit your changes. Make sure to create AUTOMATED TESTS that probes your code is working properly.

You might use as many commits as you need, however, try to group changes on commits that make sense, this will make code reviews easier for you peers.

### 3. Send to peer review

Once your code is ready, create a pull request on github from the task branch (ex. `VF-1234`) to the epic branch (ex. `epic-1`).

Pull request title must have the following structure:

```
<Task ID> <Type>: <Description of the task>
```

Where,

* Task ID: Is the ID of the task.
* Type: It can be one of `Feature`, `improvement` or `bugfix`; depending on the task.
* Description: A brief description of what you solved with this task.

Some examples of titles with this structure are:

```
VF-1256 feature: Add button to register with Facebook
VF-1398 improvement: Reduced response time on endpoints.
VF-1785 bugfix: Fixed cost per post calculation.
```

Pull request must also include a link to the Ticket on Jira and the link to the Tech Design (if it was necessary for the task). An example of how to create a pull request is:

![Pull Request Creation](/assets/create-pull-request.png)

Pull requests must be EASY TO REVIEW, therefore try to send changes with less than 200 lines of codes, group changes in commits that help reviewer to understand your work and add any comment that you need necessary.

### 4. Make sure tests are passing

After rebasing the epic branch, make sure all automated tests still passing. It’s easy that you code breaks some other modules our viceversa, therefore make sure all tests still fine.

### 5. Rebase and Squash

During solving the task and sending it to peer review you might use as many commits as you need, however, when the task is finished and you will merge it to the epic there must be ONLY ONE COMMIT PER TASK and the message of that commit must have the same STRUCTURE used on pull request’s title (see [Send to peer review](#3-send-to-peer-review)):

```
<Task ID> <Type>: <Description of the task>
```

To have a single commit per task you should *rebase* your branch and *squash* all your commits (except the first one).

[This guide from Atlasssian](https://www.atlassian.com/git/tutorials/merging-vs-rebasing) is a good introduction to rebasing and is useful to understand the differences between merging and rebasing. However, here’s a simple example:

Suppose we coded task `VF-1234` that belongs to epic `epic-1`; and task `VF-1234` is composed by 4 commits:

```
Commit 1: First commit
Commit 2: Solved task
Commit 3: Automated tests
Commit 4: Solved bugs
```

Then, to rebase it and keep a single commit with all changes. Use `git rebase -i` (a rebase with interactive option).

```bash
git rebase -i epic-1
```

This will prompt a `vi` editor listing all your commits:

```bash
pick First commit
pick Solved task
pick Automated tests
pick Solved bugs
```

Press letter `i` to start editing, and then replace the word `pickup` for `squash` (or its abbreviation `s`) on each commit except the first one (you need to keep at least 1 commit):

```bash
pick First commit
squash Solved task
squash Automated tests
squash Solved bugs
```

Then press `esc` to exit editing mode, and type `:wq` to write and quit the file. After this, you will continue to another file where you can edit the message of you final commit.

```bash
First commit
Solved task
Automated tests
Solved bugs
```

Again, press `i` to start editing. Write the message of your final commit (using structure described above) and add a `#` at the beginning of each other message to ignore those lines:

```bash
VF-1234 feature: Module for storing users.
# First commit
# Solved task
# Automated tests
# Solved bugs
```

Press `esc` to exit editing mode and type `:wq` to save it, once you did this, you will end with a single commit:

```
Commit 1: VF-1234 feature: Module for storing users.
```

To push your changes you’ll need to use `force` option because local branch `VF-1234` will conflict with remote branch. If you followed the process correctly, you don’t have to worry about push-forcing.

```bash
git push -f origin VF-1234
```

### 6. Integrate to epic branch

Finally, it’s time to merge your code to the epic branch. It might be possible that some changes conflict with you code; if that happens, use a tool or ask a peer to solve them.

USE REBASE! Avoid merges. This way we keep a clean and readable git history. To accomplish this, follow the commands:

```bash
git rebase epic-1
git checkout epic-1
git merge VF-1234
git push origin epic-1
```

### 7. Summary

In summary, the process to solve a task can be described with the next diagram:

```
 +-----------------+
 |  Create branch  |
 |    for task     |
 +--------+--------+
          |
          |
 +--------v--------+
 |   Commit your   |
 |      code       <-----+
 +--------+--------+     |
          |              |
          |              |
 +--------v--------+     |
 |  Send to peer   |     |
 |     review      |     |
 +--------+--------+     |
          |              |
          |              |
+---------v---------+    |
|All automated tests|    |
|   are passing?    +-No-+
+---------+---------+
          |
          |
 +--------v--------+
 |   Rebase and    |
 |     squash      |
 +--------+--------+
          |
          |
 +--------v--------+
 |   Integrate to  |
 |   epic branch   |
 +-----------------+
```

## Process for Epics

[1. Create a new descriptive branch for your epic](#1-create-a-new-descriptive-branch-for-your-epic)

[2. Resolve each task](#2-resolve-each-task)

[3. Rebase `dev`](#3-rebase-dev)

[4. Make sure tests are passing](#4-make-sure-tests-are-passing)

[5. Make functional tests](#5-make-functional-tests)

[6. Send to quality assurance](#6-send-to-quality-assurance)

[7. Integrate to `dev`](#7-integrate-to-dev)

[8. Summary](#8-summary)

The following process is intended for epics with multiple tasks or isolated tasks (where a single task represents a feature):

### 1. Create a new descriptive branch for your epic

Create a new branch from `dev` and set a descriptive name (see [Epic branches](#epic-branches)).

Make sure you pull changes from `dev` before you create your branch. Otherwise you’ll start the epic from an old codebase.

For example, let say we create a branch `epic-1`:

```bash
git checkout dev
git pull
git checkout -b epic-1
```

### 2. Resolve each task

Resolve each task that is related to the epic (see [Process for Tasks](#process-for-tasks)). As you resolve tasks, integrate them to the epic branch.

Don’t forget to rebase `dev` periodically (once per day), otherwise if the epic is complex it will be really difficult to integrate to `dev` due to conflicts.

### 3. Rebase `dev`

Once the epic is finished, rebase the branch with `dev`. Don’t forget to pull changes from `dev` first.

```bash
git checkout dev
git pull
git checkout epic-1
git rebase dev
```

DON’T MERGE to `dev` at this point. The epic is not tested yet, therefore if you merge it, you might be introducing bugs to `dev`.

DON’T SQUASH the commits! Epic branches must keep the commits of each task. This way, git history preserves a commit per each task:

![Example of a rebase](/assets/epic-rebase-example.png)

In example above you can see how tasks `VF-1382`, `VF-1478`, `VF-1480` and `VF-1734` composed the epic `pg-stats` and they remained after rebase with `dev`.

### 4. Make sure tests are passing

After rebase with `dev`, make sure all automated tests still passing. It’s easy that other changes in `dev` affected your code and break tests, therefore make sure tests still fine.

### 5. Make functional tests

Once you finished your epic (or you think it’s finished). Make functional tests along with all modules that compose your epic. These tests means:

* Read the product requirements document and make sure you understand it, ask somebody if you are not sure.
* Read all test cases and completion criteria included in the product required document.
* Think how you will test each case and if they are complete or there’s a case missing.
* Lift you local environment. Run all modules you need for your tests.
* Test each case you listed along with those included in the product requirements document.

If all functional tests passed, you can continue to next step, otherwise create new tasks and solve them.

### 6. Send to quality assurance

At this point, everything seems working and code is updated. You can mark this epic as “Ready to Test”. Quality assurance team will perform functional tests described in the product requirements document.

If you did a good job with you functional tests you shouldn’t hear a complain at this stage and your code will pass seamlessly to the next.

If bugs are found, create new tasks to solve them and return to step 2.

### 7. Integrate to `dev`

After all tests are passing, its time to merge you code to `dev`. It might be possible that some changes conflict with you code; if that happens, use a tool or ask a peer to solve them.

After all conflicts were resolved you can merge your epic branch using the `merge` command, specifying the `--no-ff` option in order to prevent git from trying the fast forward strategy and forcing it to create a merge commit. Make sure to comment the default merge message and provide your epic description instead, including it's identifier.

**Note:** This merge instructions only apply for epic branches with more than one commit, if your task is resolved in only one commit you can merge it onto dev branch without specifying the `--no-ff` option.

```bash
# 1. Rebase
git checkout dev
git pull
git checkout epic-1
git rebase dev

# 2. Solve conflicts if necessary

# 3. Integrate to dev

  # 3.1 Move to dev branch
  git checkout dev

  # 3.2 Merge your branch onto dev using the proper strategy
    # 3.2.1 For epic branches with more than one commit
    git merge epic-1 --no-ff
    #Replace the default merge message with your epic description (E.g. VF-1234 epic: New register)

    # 3.2.2 For branches with only one commit
    git merge epic-1

# 4. Push your changes
git push origin dev
```

After this, you can delete you epic branch from both local and remote.

### 8. Summary

In summary, the process to solve an epic can be described with the next diagram:

```
 +-----------------+
 |  Create branch  |
 |    for epic     |
 +--------+--------+
          |
          |
 +--------v--------+
 |  Resolve tasks  <------+
 +--------+--------+      |
          |               |
          |               |
 +--------v--------+      |
 |   Rebase dev    |      |
 +--------+--------+      |
          |               |
          |               |
+---------v---------+     |
|All automated tests|     |
|   are passing?    +-No-->
+---------+---------+     |
          |               |
          |               |
 +--------v--------+      |
 |Functional tests |      |
 |  are passing?   +--No-->
 +--------+--------+      |
          |               |
          |               |
 +--------v--------+      |
 |   Passed QA     |      |
 |     tests?      +--No--+
 +--------+--------+
          |
          |
 +--------v--------+
 |Integrate to dev |
 +-----------------+

```

## Reference

[Git Flow, by Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/)

[Merging vs. Rebasing, by Atlassian](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

[Thread about merge vs. rebase strategies](https://dev.to/booorkoo/rebase-or-merge-6ea)
