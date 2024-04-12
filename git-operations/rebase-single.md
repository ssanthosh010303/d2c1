# Example: Rebasing a Single Branch

> **Note:** You can reorder lines in the interactive commit to change the ordering of history itself.
>

Following are the operations that can be performed while rebasing:

1. Pick
2. **Reword:** Change Commit Message
3. Edit
4. Squash
5. Fixup
6. Drop

## Editing a Specific Commit

Let’s say I have a `git` repository with the following version history:

```
v1.0.0 -- v2.0.0 -- v3.0.0 -- v4.0.0 -- v5.0.0 (HEAD->main)
```

We are adding a line to a file called `/commit-info.txt` for every commit. So at the HEAD, the file contains:

```
v1.0.0
v2.0.0
v3.0.0
v4.0.0
v5.0.0
```

Now, let’s say I forgot to add some code at commit `v3.0.0` and made commits after that, I can use the `edit` operation of rebasing to make changes. Let us understand the step-by-step process of how `git` handles those changes.

1. **Apply Interactive Rebasing:** Apply interactive rebasing by running the command `git rebase HEAD~2`. This opens a interactive editor with the following changes.

    ```
    pick <commit-hash> v3.0.0
    pick <commit-hash> v4.0.0
    pick <commit-hash> v5.0.0
    ```

2. **Change Operation:** In the interactive editor, change the operation for `v3.0.0` from `pick` to `edit` and save the file.
3. **Head Detachment:** Now the `HEAD` stops pointing the branch but instead points to a specific commit (`v3.0.0` here). This means you’re current context is at this commit and you can make changes to the file at this point in time in history. The file at this point in time is:

    ```
    v1.0.0
    v2.0.0
    v3.0.0
    ```

    > **Note:** As seen, we don’t have the last two lines because at this point in history, those changes are yet to be made. We’re already time traveling bro.
    >
4. **What `git` is Doing:** `git` kinda maintains a queue for the operations to be performed. Since we’ve opted for editing in the first operation itself, `git` waits until we finish it and then does the `pick` operation on rest of the commits.
5. **Make Changes to File:** Now, let’s make change to the file as follows:

    ```
    v1.0.0
    v2.0.0
    v3.0.1
    ```

    As seen, we’ve changed the last line of the code. Now once the changes are made, we need to add it to the staging area with the `git add ./` command.

6. **Continue with Changes:** You can let `git` proceed with the changes (commit is performed with interactive editor again) with the command `git rebase --continue`. This’ll cause conflicts between `v3.0.1` and `v4.0.0`. But why? Let’s see the file at these points in history:
    1. **File at `v3.0.1`**

        ```
        v1.0.0
        v2.0.0
        v3.0.1
        ```

    2. **File at `v4.0.0`**

        ```
        v1.0.0
        v2.0.0
        v3.0.0
        v4.0.0
        ```


    The difference between `v3.0.1` and v4.0.0 is:

    ```
    v1.0.0
    v2.0.0
    <<<<<<<<< HEAD
    v3.0.1
    =========
    v3.0.0
    v4.0.0
    >>>>>>>>> v4.0.0
    ```

    We need to choose the changes from `HEAD` instead of the already available changes in `v4.0.0`. The file after we fix the conflicts is:

    ```
    v1.0.0
    v2.0.0
    v3.0.1
    v4.0.0
    ```

7. **Moving Forward:** Now `git` picks the commit `v4.0.0` and moves to the last operation in its queue: to pick commit `v5.0.0`. Now, there won’t be any conflicts between `v4.0.0` and `v5.0.0` and hence git will complete the rebasing successfully (`HEAD` moves to branch pointer (top)). Ultimately, the changes made in `HEAD~2` is reflected to the latest commit of branch.

## Squashing Commits

Squasing is very simple, the commit for which you mention `squash` will automatically be combined with its previous commit. You’ll be given an option to choose either the commit message of the commit where you’re applying the `squash` operation or the commit message of its previous commit. Choose either one by commenting the other one. Now, both commits will be merged together. You can squash more than two branches too.

**Before Squashing**

```
A -- B -- C (HEAD) -- D (squash) -- E (main)
```

**After Squashing**

```
A -- B -- C (or D's commit message) -- E (HEAD->main)
```

## Fixup Commits

In `git squash`, we were given an option to select message for a commit. Here’ we’re not given an option.

**Before Fixup**

```
A -- B -- C (HEAD) -- D (fixup) -- E (main)
```

**After Fixup**

```
A -- B -- C -- E (HEAD->main)
```
