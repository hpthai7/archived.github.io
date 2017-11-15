---
layout: post
title: Common utilities for terminal
tags: [terminal]
categories:
- blog
---

### Navigation

#### [Autojump](https://github.com/wting/autojump)

Navigation in terminals using command `cd` is a real hassle when we have to switch back and forth between multiple directories.
Autojump is a convenient tool to jump to a visited directory. While `cd` is handy to jump to a near relative location, i.e. a children
or the parent, `autojump` complements `cd` when users jumps to farly different locations. It is not necessarily meant to be a complete
replacement of `cd`.

Autojump takes advantage of the history of visits, ranks the list of visited sites. Users need to `cd` for a while before the database
is collected. The next time, the long typed `cd` can be shortened to

```bash
$ j <directory-component>
```

Where `directory-component` can be e constituent part in the expected directory, i.e. `/home/udev/workspace/project-<directory-component>`.
 
#### [HyperJump](https://github.com/x0054/hyperjump)

`HyperJump` is another navigation gadget for terminal. Unlike `autojump`, `HyperJump` uses manually bookmarked locations. Users have to
tag location with custom names before using. It also provides a user interface based on the package `dialog`, allows choosing the locations
visually. It features three commands for bookmarking and jumping.

```bash
# Bookmark current directory
$ jr
$ jr TagName

# Unbookmarking current directory
$ jf

# Unbookmarking directory with tag
$ jf TagName

# Open dialog interface to choose location
$ jj

# Jump immediately to directory
$ jj TagName
 
```