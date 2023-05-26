---
title : Adding changes to last commit
date : 2023-04-21 23:52:00 +0900
categories : [git]
tags : [git] #소문자만 가능
pinned : 0
---

To include new changes into the last commit, not to make a separate commit and rewrite history.

```
git add -u
git commit --amend --no-edit
```
No edit for not changing the message.

```
git push --force
```
