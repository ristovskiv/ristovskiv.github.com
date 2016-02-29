---
layout: post
title:  "Recovering lost commits and deleted branches with git reflog"
categories: [git]
tags: [git]
image: git-reflog.png
---

Ever read a tutorial or used instructions from a friend about "deleting" a commit from a git history. They most likely use the `reset` tool for that purpose and of course they mention that using reset is dangerous. Well in a way it is, if you use it on commits that are already stored on some remote repository, but that is a topic for another day. But what if I told you the reset tool is not that dangerous as you think, what if I told you there is a way that you can control it, what if I told you there is a way that you can restore a commit that you've accidentally "deleted", and last but not least

![Morpheus-git](https://monosnap.com/file/YZj7glV6Am03eLTynUiHGgPpmbyPP9.png)

Yes, I read it in Morpheus's voice as well, but the main purpose is git actually never deletes commits. But where does it store the ones that we think we've 'deleted' them?

## Enter 'git reflog'

`This is when someone tries to sell something again after failing the first, second time etc.` - [Urban Dictionary][urban-dictionary] about the meaning of the word 'reflog'. Interesting definition, but we really want to know what does git say about his `reflog` tool. Let me walk you through it with a practical example. 

We already have a git repo in your project with some commits on the master branch.

![git-log-oneline](https://monosnap.com/file/OsHp0dKg4LC2QXMFxBYL283B3l0mJc.png "git log --pretty=oneline")

Now, for some unknown reasons we decide to drop the last commit. We can do that (as mentioned before) with `git reset --hard HEAD~1`. Seconds after we run that, we realize that we made a mistake and we need that commit back. But how can we do that when that commit is no where to be found in the log? Well, git keeps a second log, but remember this, only in your local repo, which is called, drum roll...`reflog`. And if we read through the reflog using `git reflog` we can see the results

![git-reflog](https://monosnap.com/file/qHkLvyD5NmgbzM7ncU7lgIgBQ9kvfA.png "git reflog")

Hmm...So our three commits are there, but how does reflog know about the reset that we've done? Git updates the reflog anytime our HEAD moves. So that means if we have a new commit, or change branches, or do a reset, another entry is put in the reflog. Now, let me explain the output. In the first column you have the SHA of the commit that the HEAD was pointing at in its current state. In the second column we have a short name that it can be used as a reference for the particular commit and in the third column there is a description of the operation that caused the HEAD to move. One quick note that some of you may have noticed by now, HEAD@{0} is always the current commit. Our goal is to return the commit with the commit message 'the third commit'. In order to do that again we'll use the reset tool, but we have two choices. We can specify the SHA of the commit or the short name used as a reference, that means we can either to `git reset --hard 7e56f87` or `git reset --hard HEAD@{1}`, and if we check the log again we'll see that it's the same as before

![git-log-oneline](https://monosnap.com/file/OsHp0dKg4LC2QXMFxBYL283B3l0mJc.png "git log --pretty=oneline")

I did mention before, and I want to emphasize it again, reflog only tracks the local repo, that means that when you clone someone elses repo your reflog starts from scratch.

## Branch in distress

Let's see another frequent real world case where we can witness the power of reflog. [GRRM][george-martin] wanted us to help him and assigned us to write the remaining story for one of his characters from "Game of Thrones", Tyrion Lannister. So we create a separate branch called tyrion-lannister and we start working and commiting. 

![tyrion-lannister-branch](https://monosnap.com/file/gDu8GjoisrB80MNm5MmZzb5KyjWB2e.png "git log --pretty=oneline")

Suddenly GRRM decides to kill our character and all of our additional work becomes useless, so we don't need our branch anymore even though our commits are not merged with master and we delete it by checking out to master first `git checkout master` and then running `git branch -D tyrion-lannister`. But the script was leaked before being published and the readers and viewers of the show aren't happy, in fact they are furious. GRRM changes his mind and wants us to continue with our work. But wait...our whole work was on the tyrion-lannister branch and we didn't merge that branch anywhere. Reflog to the rescue. As we mentioned before git never deletes commits, he just deleted the branch. So if we can find the latest commit from that branch we can recreate it. To find the last commit from that branch we need to see the reflog

![git-reflog-branch-deleted](https://monosnap.com/file/NOhKq8qfEURyFrEv4JJ3zWvaX5Sc4K.png "git reflog")

We see that before we've checked out to master from the branch tyrion-lannister there is a commit, which actually is the last commit from that branch. So in order to recreate the branch we can run `git branch tyrion eca3d37` or we can use the reference `git branch tyrion HEAD@{1}`. Note: the name of the branch doesn't have to be the same, what it matters is that the branch will have the same history as the deleted branch.

## Where to learn more

I have barely scratched the surface of the possibilities that reflog can offer. In order to learn more check out the [official documentation](https://git-scm.com/docs/git-reflog).


[urban-dictionary]:      http://www.urbandictionary.com/define.php?term=Reflog&defid=6923465#image-6923465
[george-martin]:         https://en.wikipedia.org/wiki/George_R._R._Martin



