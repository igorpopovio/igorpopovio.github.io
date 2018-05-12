---
title: Find files and copy/move them with xargs
layout: post
category : linux
tags : [linux, bash, command, xargs, cp, mv, find]

---

## Command

    find . -iname '*.bak' -print0 | xargs -0 -I {} mv {} ~/old.files

## Explanation
`{}` is called the default argument list marker. This needs to be used with all the commands that take multiple arguments.
For example, the `mv` command takes 2 arguments: the source file and the destination file.

