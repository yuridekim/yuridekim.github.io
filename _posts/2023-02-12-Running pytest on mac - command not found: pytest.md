---
title : "Running pytest on mac - command not found: pytest"
date : 2023-02-12 00:56:00 +0900
categories : [tools]
tags : [pytest, python] #소문자만 가능
---

Installing pytest with pip will return the following error.

```
learnGPT-team2 % pytest test_ex.py -s -vv
zsh: command not found: pytest
```
In this case, try python -m pytest

Installing pytest via pip doesn't make it a system command, it installs it to python. The -m command runs pytest as its own command and then any proceeding script will be an argument.


```
python3 -m pytest test_ex.py -s -vv
```

[stack overflow](https://stackoverflow.com/questions/53154896/installed-pytest-but-running-pytest-in-bash-returns-not-found)