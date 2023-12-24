---
layout: post
title:  "Git Modify Author and Committer"
date:   2019-06-13 07:23:58 +0545
categories: [Git, Modify Author And Committer]
tags: [git]
---

# How to change all commits to have the same newly added author and committer?

#### Go to appropriate branch and project directory and run the following command on console.

```
  git filter-branch -f --env-filter "
    GIT_AUTHOR_NAME='ShivRaj' 
    GIT_AUTHOR_EMAIL='shivrajbadu@gmail.com'
    GIT_COMMITTER_NAME='ShivRaj'
    GIT_COMMITTER_EMAIL='shivrajbadu@gmail.com'
  " HEAD
 ```

#### Above command run successfully with following output

```
Rewrite cd130b5306f93f52a1ef7cce7fd8c25ad5a68b14 (1/1) (0 seconds passed, remaining 0 predicted)    
Ref 'refs/heads/master' was rewritten
```
