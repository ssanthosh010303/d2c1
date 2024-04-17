# Git Operations

- **Author:** Sakthi (Sandy) Santhosh
- **Created on:** 09/04/2024

## Introduction

Following are the operations that are specific to the operations performed on a multi-branch project with multiple programmers working on it. Following are the `git` operations:

1. Rebasing
    1. Multi-branch
    2. Single Branch
2. Merging
    1. Fast-forward
    2. Three-way Merge
3. Cherry-picking
4. Reflog (reference/pointer log)

## Merging

Merging in Git is the process of combining changes from different branches into a single branch, typically into the main branch (e.g., `main` or `master`). It allows multiple developers to work on different features or fixes independently and then integrate their changes into a common codebase.

### Fast-forward Merge

In a fast-forward merge, the target branch can be directly moved to the most recent commit of the source branch because there are no new commits on the target branch since the divergence point.

**Before Merging:**

```
A -- B -- C (main) (HEAD)
          \
           D -- E (feature-branch)
```

**After Merging:**

```
                    main (HEAD)
                    |
A -- B -- C -- D -- E (feature_branch)
```

In this scenario, the `main` branch is behind the `feature_branch`, so a fast-forward merge is possible. Git simply moves the `main` branch pointer to the commit `E` of the `feature_branch`, resulting in a linear history.

> **Note:** After a fast-forward merge, the auxiliary branch no longer exists as a separate branch; its commits become part of the target branch's commit history. Note in the above example that all the pointers point to the commit `E`.
>

> **Note:** Merging from main to feature and feature to main branches are different.
>

### Recursive (three-way) Merge (default):

In a recursive merge, the branches being merged have diverged, meaning there are new commits on both branches since they diverged from a common ancestor commit. Git analyzes the changes made on both branches and creates a new merge commit to integrate them.

**Before Merging:**

```
          main (HEAD)
          |
A -- B -- C -- D (main)
          \
           E -- F (feature_branch)
```

**After Merging (three-way, feature to main):**

```
                    main (HEAD)
                    |
A -- B -- C -- D -- M (merge_commit)
          \       /
           E --- F (feature_branch)
```

In this scenario, Git creates a new merge commit `M` that combines the changes from both the `main` and `feature_branch`. The merge commit `M` has two parent commits, representing the divergent branches, and includes the changes from both branches in the final result.

### Local Merge vs. Remote Merge

We can merge branches either locally and then push the changes to upstream or merge branches in upstream and pull the changes.

Local Merging: It can be carried out if you’re the owner/collaborator of the upstream repository. This means you have access to make changes to the `main/master` branch.

**Remote/Upstream Merging:** This can be carried out if you’re not an owner of a repository but a contributor to it. In this scenario, you can branch-out from a particular commit, make changes and push it to remote. Now, you must make a pull request so that the owner of the repository can review the changes and merge.

## Rebasing

Rebasing in Git is a process that allows you to change the base of your current branch from one commit to another. It's essentially a way to rewrite the commit history by moving or reapplying commits from one branch onto another branch's history. Unlike merging, which creates a new commit to merge the changes, rebasing integrates the commits by applying them onto the base branch directly.

### Scenarios

There are two scenarios you need to use rebasing for:

1. **Muti-branch Rebasing:** Changing Parent of Auxiliary Branch and Restructuring
2. **Single Branch Rebasing**

### How Rebasing Works

When you rebase a branch onto another branch, Git identifies the common ancestor commit between the two branches. It then "replays" the commits from the current branch on top of the base branch, effectively incorporating the changes into the base branch's history. This results in a linear history without merge commits, as if the changes were always made directly on top of the base branch.

Interactive rebasing in Git allows you to modify the commit history by interactively selecting, reordering, squashing, or editing commits. This gives you more control over your commit history, allowing you to create a cleaner, more organized history before merging or pushing your changes. Here's how to carry out interactive rebasing:

#### Before Rebase

Initially, your feature branch (`feature_branch`) diverged from the main branch (`main`) at commit `C`. Meanwhile, new commits (`X`, `Y`, `Z`) were made on the main branch after the split:

```
                                 main
                                 |
A -- B -- C ---------- X -- Y -- Z
          \
           D -- E (feature_branch)
```

#### During Rebase

When you rebase `feature_branch` onto `main`, Git identifies the common ancestor commit (`C`) between the two branches. It then replays the commits from `feature_branch` on top of the latest commit of `main` (`Z`):

```
                                 main
                                 |
A -- B -- C ---------- X -- Y -- Z
          \                      \
           \                      D' -- E' (feature_branch)
            D -- E (feature_branch_old)
```

#### After Rebase

Once the rebase is complete, the branch pointers are updated to reflect the new commit structure:

```
                                 main
                                 |
A -- B -- C ---------- X -- Y -- Z
          \                      \
           \                      D' -- E' (feature_branch) (HEAD)
            D -- E (feature_branch_old)
```

> **Note (scenario explained):** The changes from `C` to `Z` will be reflected till the last commit of the feature branch. This kind of rebasing is useful in a scenario where you have a important bug fix (say) in master branch that you want to incorporate while working with feature branch. In that case, you can move the parent of the feature branch from `C` to `Z` (where `Z` is the commit with bug fix). However, note that conflicts may arise while doing so.
>

> **Conflict Management:** In this scenario, `git` will compare code in commit `Z` with code in commit `E'`. All the merge conflicts (if exists) must be resolved.
>

> **Note:** Usually after this, the main branch is fast forwarded to the `HEAD`. The history then becomes:
>

```python
                                 main
                                 |
A -- B -- C ---------- X -- Y -- Z -- D' -- E' (main) (HEAD)
          \
           \
            D -- E (feature_branch_old)
```

### Actions for Each Commit

Certainly, Master Sakthi Santhosh. Let's delve into each of the actions available during interactive rebasing in Git:

1. **Pick:** The `pick` action is the default action during interactive rebasing. It instructs Git to *keep the commit as-is* without making any modifications. The commit will be applied exactly as it is in the original sequence of commits.
2. **Reword:** The `reword` action allows you to *edit the commit message* of a specific commit during the interactive rebase process. When you select the `reword` action for a commit, Git will pause the rebase process and open a text editor, allowing you to modify the commit message. Once you've edited the commit message and saved the changes, Git will continue with the rebase process.
3. **Edit:** The `edit` action pauses the rebase process to *allow you to make changes to the content of the selected commit.* When you choose the `edit` action for a commit, Git will pause the rebase process immediately after applying the commit, allowing you to make modifications to the commit's content (e.g., add, edit, or remove files). After making the necessary changes, you can stage the modifications and use the `git commit --amend` command to finalize the changes. Once you're done, you can continue the rebase process by executing `git rebase --continue`.
4. **Squash:** The `squash` action allows you to *combine the selected commit with the previous commit,* effectively merging the changes from both commits into a single commit. When you choose the `squash` action for a commit, Git will prompt you to edit the commit message for the combined commit. You can then modify the commit message as desired. The `squash` action is useful for consolidating multiple small commits into a single, more cohesive commit.
5. **Fixup:** The `fixup` action is similar to the `squash` action but *discards the commit message of the selected commit.* It combines the changes from the selected commit with the changes from the previous commit, without preserving the commit message of the selected commit. This action is useful for combining small, incidental commits into a single commit without retaining individual commit messages.
6. **Drop:** The `drop` action *removes the selected commit entirely from the commit history* during the interactive rebase process. When you choose the `drop` action for a commit, Git will exclude the commit from the final commit history, effectively removing it from the branch's history altogether. Use the `drop` action to eliminate unnecessary or redundant commits from the commit history.

> **Difference Between Squash and Fixup:**
>
> - **Squash**: Combines multiple commits into a single commit and allows you to edit the commit message for the combined commit.
> - **Fixup**: Combines commits into a single commit and automatically discards the commit message of the selected commit, using the commit message of the previous commit for the combined commit.

### When to Use Rebasing

> **Note:** Rebasing is used in both single-branch and multi-branch configurations.
>
1. **Keeping a Clean and Linear History:** Rebasing is often used to maintain a clean and linear project history. By rebasing feature branches onto the latest changes in the main branch before merging them, you can create a more straightforward and easier-to-follow commit history without unnecessary merge commits.
2. **Integrating Changes from Multiple Feature Branches:** When working on a project with multiple feature branches, rebasing can help integrate changes from those branches into the main branch more seamlessly. By rebasing each feature branch onto the main branch before merging, you can ensure that each feature's commits are applied on top of the latest changes in the main branch.
3. **Resolving Merge Conflicts:** Rebasing can also help in resolving merge conflicts more easily. Since rebasing applies commits one by one, you can resolve conflicts as they arise during the rebase process. This allows for more granular conflict resolution compared to resolving conflicts in a single merge commit.
4. **Squashing Commits:** During interactive rebasing, you have the option to squash or combine multiple commits into a single commit. This can help in cleaning up the commit history and presenting a more concise and logical sequence of changes.
5. **Preparing Clean Pull Requests:** When contributing to open-source projects or collaborating with a team, rebasing your feature branch onto the latest changes in the main branch before creating a pull request can help keep the pull request clean and focused on the specific changes introduced by your branch.

## Resetting

Resetting is the process of erasing the history of Git. It’s like you have a time stone. There are three things affected here (not space, time and Avengers):

1. Staged Files
2. Changed Code
3. Commits

There are three types of resets in `git`:

1. **Hard:** This’ll remove the N commits you mention while removing the code and the staged files.
2. **Mixed:** This’ll keep the code but remove the staged files for commit.
3. **Soft:** This’ll remove the commits alone.

## Cherry-picking

Cherry-picking in Git refers to the process of selecting and applying specific commits from one branch to another. It allows you to choose individual commits from any branch and apply them to another branch, effectively incorporating specific changes into different parts of your project without merging entire branches.

> **Note:** When you cherry-pick a commit in Git, Git *essentially applies the changes introduced by that commit onto the current branch* (the branch you're currently on). This means that Git will take the differences introduced by the cherry-picked commit and apply them to the current branch's codebase, effectively incorporating those changes into the current branch's history. This means that the cherry-picked commit itself is not changed but it incorporates changes into the feature branch.
>

### Before Cherry-picking

```python
               main
               |
A -- B -- C -- D
      \
       X -- Y -- Z (new_feature_branch)
```

Let’s say I wanna cherry-pick `C` and place it in `new_feature_branch`, I’ll do the following:

```bash
git checkout new_feature_branch
git cherry-pick <commit-hash>
```

### After Cherry-picking

```
              main
               |
A -- B -- C -- D
      \
       X -- Y -- Z -- C' (new_feature_branch)
```

> **Note:** The changes from `X`, `Y` and `Z` are not applied to `C’`. Instead, the successive branches to `C'` incorporate the changes of `C’` and `X`, `Y` and `Z`.
>

## Reflog

It is similar to `git log` but then it only logs the changes made to the pointers such as `HEAD`.

## Next Document

[Example: Rebasing a Single Branch](./git-operations/rebase-single.md)
