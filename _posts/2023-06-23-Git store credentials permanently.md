---
title : Git store credentials permanently
date : 2023-06-13 02:00:00 +0900
categories : [Tools, Git]
tags : [git] #소문자만 가능
pinned : 0
---
When being prompted to enter your git username and password repeatedly, store the information permanently.

```bash
git config credential.helper store
```

Once the credential.helper is activated, you'll be asked once again to input the two credentials and will be stored permanently.