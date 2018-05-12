---
title: 7zip gotchas
layout: post
tags : [7zip, archiving]

---

This is going to be a short post about some **gotchas** when archiving files with 7zip from the command line.

## Archiving files

Let's start from the following directory structure:  

    files
    |-- file1.txt
    `-- file2.txt

Here's the simplest way to archive those files using 7zip:  

    7z a archive.zip files

This will create the following structure:  

    archive.zip
    `-- files
        |-- file1.txt
        `-- file2.txt

This is fine, but what if you don't want the `files` top directory? Like in this example:  

    archive.zip
    |-- file1.txt
    `-- file2.txt

In this case you should use the following command:  

    7z a archive.zip .\files\*

Slashes, the initial dot (current directory) and the final asterisk are required or else 7zip will fallback to archive everything including the top level directory.

## Examples repository

I made a small PowerShell script that you can use to test the examples described in this post. Here's [the link](https://github.com/igorpopovio/7zip-usage-examples/blob/master/7zip-usage-examples.ps1).
