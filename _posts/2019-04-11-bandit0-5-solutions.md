---
title: "OverTheWire’s Bandit 1-15 solutions"
date: 2019-04-11
tags: [overthewire, ctf, writeup, solutions,bandit]
excerpt: "OverTheWire’s Bandit 1-15 solutions"
---

## Level 0:

Goal
> The goal of this level is for you to log into the game using SSH. The host to which you need to connect is bandit.labs.overthewire.org, on port 2220. The username is bandit0 and the password is bandit0. Once logged in, go to the Level 1 page to find out how to beat Level 1.

Solution:

```
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
password:bandit0
cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```

## Level 0 --> 1:

Goal
> The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.

Solution:

```
cat < - 
cat ./-
```
@ https://unix.stackexchange.com/questions/16357/usage-of-dash-in-place-of-a-filename 
