---
layout: single
title:  "Regex CheatSheet"
date:   2020-09-20 16:40:00 +0800
categories: programming
tags: [regex]
---


## 基础

* dot `.` matches anything except for a newline

* `\d` matches any digit [0-9]

* `\D` matches any character that is **not** a digit.

* `\s` matches any whitespace character [ \r\n\t\f ].

* `\S` matches any **non-whitespace** character

* `\w` matches any word character, Word characters include alphanumeric characters (a-z, A-Z and 0-9) and underscores (_).

* `\W` matches any **non-word** character

* `^` matches the position at the start of a string

* `$` matches the position at the end of a string



## 重复

* `*` matches **zero or more** repetitions of character/character class/group

* `+` matches **one or more** repetitions of character/character class/group

* `?` 匹配 0 或者 1 个某字符。例如 `a?` 表示 **zero or one** of  a

* `{x}` will match **exactly** *x* repetitions of character/character class/groups

* `{x,y}` will match between *x* and *y* (both inclusive) repetitions of character/character class/groups。当省略 y 时表示重复 >=  x 次。



## Character Class

* `[]` matches only **one** out of several characters placed inside the square brackets.

* `[^]` matches any character that is **not** in the square brackets.

字符范围，常用的有 [a-z]，[A-Z]，[0-9]



## Grouping and Capturing

* `\b` assert position at a **word boundary.**

> Three different positions qualify for word boundaries :
► Before the first character in the string, if the first character is a word character.
► Between two characters in the string, where one is a word character and the other is not a word character.
► After the last character in the string, if the last character is a word character.


* `()` around a regular expression can **group** that part of regex together.

* `(?: )` can be used to create a non-capturing group. It is useful if we do not need the group to capture its match. 

* `|` 表示**或者**，match a single item out of several possible items separated by the vertical bar. When used inside a character class, it will match characters; when used inside a group, it will match entire expressions (i.e., everything to the left or everything to the right of the vertical bar).  也就是说 `|` 可以用在 `[]` 或者 `()` 中。


## Backreferences

* `\group_number`：This tool (**\1** references the first capturing group) matches the same text as previously matched by the capturing group. 比如，**(\d)\1**: It can match `00`, `11`, `22`, `33`, `44`, `55`, `66`, `77`, `88` or `99`.


## Assertions

* Positive lookahead：`regex_1(?=regex_2)` asserts `regex_1` to be immediately followed by `regex_2`. The lookahead is excluded from the match. It does not return matches of `regex_2`. The lookahead only asserts whether a match is possible or not.
* Negative lookahead: `regex_1(?!regex_2)` asserts `regex_1` *not* to be immediately followed by `regex_2`. Lookahead is excluded from the match (do not consume matches of `regex_2`), but only assert whether a match is possible or not.
* Positive lookbehind: `(?<=regex_2)regex_1` asserts `regex_1` to be immediately preceded by `regex_2`. Lookbehind is excluded from the match (do not consume matches of `regex_2`), but only assert whether a match is possible or not.
* Negative lookbehind: `(?<!regex_2)regex_1` asserts `regex_1` *not* to be immediately preceded by `regex_2`. Lookbehind is excluded from the match (do not consume matches of `regex_2`), but only assert whether a match is possible or not.


## 补充

* `*?` 表示**非贪婪匹配**，即“尽可能少的匹配”。如 `r\w*?` 对于 r, re, regex 都只会匹配到 r，而 `r\w*` 则会匹配到整个单词。