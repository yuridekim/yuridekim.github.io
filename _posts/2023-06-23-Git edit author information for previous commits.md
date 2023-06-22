---
title : Git edit author information for previous commits
date : 2023-06-23 02:23:00 +0900
categories : [git]
tags : [git] #소문자만 가능
pinned : 0
---

To rewrite author information such as author usernames and emails for previous commits,
first add an alias for the change-commits command.
```bash
git config --global alias.change-commits '!'"f() { VAR=\$1; OLD=\$2; NEW=\$3; shift 3; git filter-branch --env-filter \"if [[ \\\"\$\`echo \$VAR\`\\\" = '\$OLD' ]]; then export \$VAR='\$NEW'; fi\" \$@; }; f"
```

To change author name and email address,

```bash
git change-commits GIT_AUTHOR_NAME "old name" "new name"
git change-commits GIT_AUTHOR_EMAIL "old@email.com" "new@email.com"
```

To specify the number of previous commits to rewrite,
```bash
# e.g. edit past 3 commits
git change-commits GIT_AUTHOR_NAME "old name" "new name" HEAD~3..HEAD
```

However when you attempt to edit both of them(i.e. more than once), it crashes with the existing backup.
```
Cannot create a new backup.
A previous backup already exists in refs/original/
Force overwriting the backup with -f
```

Remove the original backup.

```bash
git update-ref -d refs/original/refs/heads/<branch-name>
```


This changes the author without changing the commit date.  
However, it does <b>NOT</b> reconfigure the commit author.  
![rewrite_author](/assets/img/posts/rewrite_author.png)

---
[Git, rewrite previous commit usernames and emails](https://stackoverflow.com/questions/2919878/git-rewrite-previous-commit-usernames-and-emails)