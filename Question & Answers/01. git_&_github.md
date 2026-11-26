# Git

<details>
<summary>1. How would you create a new branch in Git based on the latest commit on the master branch?</summary>

<pre>
git checkout master
git pull
git checkout -b &lt;new_branch_name&gt;
</pre>

</details>

<details>
<summary>2. How would you squash multiple commits into one?</summary>

<pre>
git rebase -i HEAD~&lt;number_of_commits&gt;
</pre>

→ Change `pick` to `squash` (or `s`) for the commits you want to combine, save, and edit the message.

</details>

<details>
<summary>3. How would you discard changes in Git (revert a file to its previous state)?</summary>

<pre>
git checkout -- &lt;filename&gt;
# or in newer Git
git restore &lt;filename&gt;
</pre>

</details>

<details>
<summary>4. How would you view the differences between current changes in your working directory and the last commit?</summary>

<pre>
git diff
</pre>

</details>

<details>
<summary>5. How would you stage only some of the changes for commit?</summary>

<pre>
git add -p
</pre>

→ Git will prompt you to accept, skip, or split each hunk (hunk-by-hunk staging).

</details>

<details>
<summary>6. How would you remove a file from Git without deleting it from your local file system?</summary>

<pre>
git rm --cached &lt;filename&gt;
</pre>

</details>

<details>
<summary>7. How would you rename a file in Git while preserving its history?</summary>

<pre>
git mv &lt;old_filename&gt; &lt;new_filename&gt;
git commit -m "rename file"
</pre>

</details>

<details>
<summary>8. How would you discard all local changes and make your working directory match the last commit?</summary>

<pre>
git reset --hard HEAD
</pre>

</details>

<details>
<summary>9. How would you undo the last commit and make changes to the previous commit?</summary>

<pre>
git commit --amend
</pre>

→ This replaces the last commit with the amended one.

</details>

<details>
<summary>10. How would you check out a specific commit in Git to review its contents?</summary>

<pre>
git checkout &lt;commit_hash&gt;
</pre>

→ Return to your branch with `git checkout &lt;branch_name&gt;`.

</details>

<details>
<summary>11. How would you create a new branch in Git based on a specific commit?</summary>

<pre>
git checkout -b &lt;new_branch_name&gt; &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>12. How would you cherry-pick a specific commit from one branch to another?</summary>

<pre>
git checkout &lt;target_branch&gt;
git cherry-pick &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>13. How would you view the commit history of a specific file?</summary>

<pre>
git log -- &lt;filename&gt;
</pre>

</details>

<details>
<summary>14. How would you revert a specific commit and remove its changes from the repository?</summary>

<pre>
git revert &lt;commit_hash&gt;
</pre>

→ Creates a new commit that undoes the changes — safe for shared branches.

</details>

<details>
<summary>15. How would you merge changes from one branch into another?</summary>

<pre>
git checkout &lt;target_branch&gt;
git merge &lt;source_branch&gt;
</pre>

</details>

<details>
<summary>16. How would you discard all local changes, including untracked files, and revert to the last commit?</summary>

<pre>
git reset --hard HEAD
git clean -fd
</pre>

</details>

<details>
<summary>17. How would you see the changes introduced by a specific commit?</summary>

<pre>
git show &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>18. How would you delete a branch from your local repository?</summary>

<pre>
git branch -d &lt;branch_name&gt;
# force delete
git branch -D &lt;branch_name&gt;
</pre>

</details>

<details>
<summary>19. How would you view the differences between two branches?</summary>

<pre>
git diff &lt;branch1&gt; &lt;branch2&gt;
</pre>

</details>

<details>
<summary>20. How would you create a tag in Git to mark a specific commit?</summary>

<pre>
git tag -a &lt;tag_name&gt; -m "release notes"
git push origin &lt;tag_name&gt;
</pre>

</details>

<details>
<summary>21. How would you stash your current changes temporarily and come back later?</summary>

<pre>
git stash
git stash pop
</pre>

→ `git stash list` shows all stashes.

</details>

<details>
<summary>22. How would you revert a specific file to its state in a previous commit?</summary>

<pre>
git checkout &lt;commit_hash&gt; -- &lt;filename&gt;
</pre>

</details>

<details>
<summary>23. How would you change the message of your last commit?</summary>

<pre>
git commit --amend -m "new message"
</pre>

</details>

<details>
<summary>24. How would you see the differences between two commits?</summary>

<pre>
git diff &lt;commit1&gt; &lt;commit2&gt;
</pre>

</details>

<details>
<summary>25. How would you see the list of files changed in a specific commit?</summary>

<pre>
git show --name-only &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>26. How would you create a new branch and switch to it, all in one command?</summary>

<pre>
git checkout -b &lt;new_branch_name&gt;
# or
git switch -c &lt;new_branch_name&gt;
</pre>

</details>

<details>
<summary>27. How would you apply a specific commit from one branch to another without creating a new commit?</summary>

<pre>
git cherry-pick -n &lt;commit_hash&gt;
</pre>

→ `-n` (no commit) stages the changes without committing.

</details>

<details>
<summary>28. How would you set up Git to ignore specific files or directories?</summary>

→ Create or edit a `.gitignore` file in the repo root.

<pre>
*.log
node_modules/
.env
</pre>

</details>

<details>
<summary>29. How would you configure Git to automatically pull changes from a remote repository whenever you switch to a branch?</summary>

→ There's no fully automatic "pull on branch switch", but you can enable auto-fetch and auto-rebase-on-pull:

<pre>
git config --global fetch.prune true
git config --global pull.rebase true
</pre>

→ Best practice is to pull manually or via a script after checkout.
<pre>
git checkout &lt;branch&gt; && git pull
</pre>

</details>

<details>
<summary>30. How would you configure Git to automatically push changes to a remote repository whenever you make a commit?</summary>

→ Git has no built-in "auto-push on commit". The closest is a `post-commit` hook.

<pre>
# .git/hooks/post-commit
#!/bin/sh
git push origin HEAD
</pre>

→ Make the script executable:
<pre>
chmod +x .git/hooks/post-commit
</pre>

→ Note: auto-pushing on every commit is discouraged; it interferes with rebases and history rewrites.

</details>

<details>
<summary>31. What is your Git branching strategy?</summary>

→ **Trunk-based** for small teams — short-lived feature branches merged to `main` frequently.

→ **Git Flow** for larger release-driven teams — `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`.

→ **GitHub Flow** — a single `main` with feature branches and PR-based merges.

→ We choose based on team size, release cadence, and how strict environment isolation needs to be.

</details>

<details>
<summary>32. How will verticals be included in the Git branch?</summary>

→ Each vertical (business unit/product line) can have its own repository or a top-level folder/branch namespace.

→ Main approaches:
  - **Poly-repo per vertical** — cleanest isolation, separate pipelines.
  - **Mono-repo with folder per vertical** — shared tooling, path-filtered pipelines.
  - **Branch namespacing** — prefix branches with the vertical (`vertical-a/feature-x`).

→ Ownership is enforced through CODEOWNERS files and protected branches.

</details>

<details>
<summary>33. How are naming and tagging conventions managed?</summary>

→ **Branch names**: `feature/&lt;ticket&gt;-&lt;short-desc&gt;`, `bugfix/...`, `hotfix/...`, `release/&lt;version&gt;`.

→ **Tags**: Semantic versioning — `v1.2.3`.

→ **Commits**: Conventional commits (`feat:`, `fix:`, `chore:`) to enable changelog automation.

→ Enforced via branch protection rules, commit hooks, and PR templates.

</details>

---
