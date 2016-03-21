---
layout: post
title:  "Installing C"
date:   2016-03-20 23:03:03 -0500
categories: hacking c mac keychain homebrew gdb codesign gcc
---

TODO: explain points:
brew install gcc
brew install gobjdump
brew tap homebrew/dupes
brew install gdb
Open Keychain Access
- Keychain Access/Certificate Assistant/Create a Certificate...
- name gdb-cert
- Self Signed Root
- Code Signing
- Let me override defaults
- Click all continues
- Location of cert should be on system!
which gdb
codesign -s gdb-cert <path to gdb>
* If created incorrect cert, use -f to replace in code sign command
