title: 修改git log中的author邮箱
author: ming
tags: []
categories:
  - git
date: 2018-11-27 21:16:00
---
```
#!/bin/sh
git filter-branch --env-filter '
OLD_EMAIL="ming.liu@dfs168.com"
CORRECT_NAME="liumingmad"
CORRECT_EMAIL="liumingmad@gmail.com"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```