---
layout: single
classes: wide
title: "Effective Code Browsing - Part 1"
categories: posts
excerpt: "Exploring the Powerful Grep Utility"
tags: [linux, tools, grep, regex, cscope, vim, git-grep, opengrok]
comments: true
toc: true
toc_label: "Table of Content"
share: true
---

## Overview
Contrary to what my friends and family think I do as a software engineer—which is that I type code all day in front of a computer—I actually spend a lot of time browsing code bases, reading code, debugging issues, and designing systems. When triaging a bug, I usually browse the surrounding code and read and understand how the system is supposed to behave under good conditions (i.e. normal operation without a bug), and then try to figure out what is going wrong, or what is causing the bug. For browsing
the code, there are no better tools than `grep` & `git-grep`, `cscope` (for C/C++ projects) and `opengrok`. These tools are literally life saving, and one `grep` could save hours of manual searching. Let me explain and show the usefulness of these tools.

## Grep
`Grep` is one of the most powerful open source tools out there. It is installed by default by almost every installation of Linux, BSD, and Unix, regardless of the distribution. Simply stated, `grep` is a command-line utility that finds plain-text in a file or output efficiently and quickly. It searches the provided input files (if any) or standard input, for lines containing a match to a given pattern. The pattern is a regular expression (regex).

### Examples
#### Search Standard Output
{% highlight bash %}
$ grep [options] search_string
{% endhighlight %}

Suppose you want to search for all instances of the word "License" in your working directory:
{% highlight shell %}
# Output truncated for sake of brevity
[andy@kubuntu:~/dev/demo/plotly.py]$ grep -R "License"
packages/python/chart-studio/LICENSE.txt:The MIT License (MIT)
packages/python/plotly/plotly/package_data/plotly.min.js:* Licensed under the MIT license
packages/python/plotly/plotly/package_data/plotly.min.js: * Licensed under the MIT license
packages/python/plotly/plotly/package_data/plotly.min.js: * Licensed under the MIT License.
packages/python/plotly/versioneer.py:* License: Public Domain
packages/python/plotly/versioneer.py:## License
packages/python/plotly/README.md:        <td>License</td>
README.md:        <td>License</td>
README.md:## Copyright and Licenses
LICENSE.txt:The MIT License (MIT)
{% endhighlight %}

#### Search String in a File
{% highlight bash %}
$ grep [options] search_string path/to/file
{% endhighlight %}
Suppose you have a requirements.txt file containing all the required packages for a library, and you would like to know the required version for `pandas` library. You can open the file and search for `pandas` and see the required version and then close the file, but that will take too long, and life is too short, so why not just `grep` for `pandas` like the example below?

{% highlight bash %}
[andy@kubuntu:~/dev/demo/plotly.py/doc]$ grep pandas requirements.txt
pandas==1.0.3
geopandas==0.8.1
{% endhighlight %}

#### Find if a File Contains a string
{% highlight bash %}
$ grep [options] search_string path/to/file
{% endhighlight %}
What if you want to find if a specific library  (for example `opengl`) is part of the requirements.txt file from the last example? Again, you can open the file, and search for the library name inside the file. If found, then the library is required and not otherwise. But that would take more time than just `grepping` for the lib name.
{% highlight bash %}
[andy@kubuntu:~/dev/demo/plotly.py/doc]$ grep opengl requirements.txt
[andy@kubuntu:~/dev/demo/plotly.py/doc]$
[andy@kubuntu:~/dev/demo/plotly.py/doc]$ grep wget requirements.txt
wget
{% endhighlight %}
`grep` did not find `opengl` inside requirements.txt, but it did find `wget`, so `opengl` is not in the requirement list.

#### Grep Options Explained
`-i`    ignores case sensitivity
{% highlight bash %}
[andy@kubuntu:~/dev/demo/plotly.py]$ ifconfig | grep -i ':4F:AB'
        ether 50:7b:9d:30:4f:ab  txqueuelen 1000  (Ethernet)
{% endhighlight %}
`-w` searches for the full word
```bash
[andy@kubuntu:~/dev/demo/plotly.py]$ grep -w connect /var/log/syslog
Aug 17 10:23:35 kubuntu redshift[945]: Could not connect to wayland display, exiting.
Aug 17 10:23:35 kubuntu redshift[977]: Could not connect to wayland display, exiting.
```
As an exercise to the reader, try running `grep connect /var/log/syslog` (i.e. without the `-w` option). Are you surprised to see connection, connected, disconnection, and disconnected in the results?

`-B [N]` and `-A [N]` display N lines before and after the text
{% highlight bash %}
andy@kubuntu:~/dev/demo/plotly.py]$ ifconfig | grep -i -B2 ':4F:AB'
enp0s25: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 50:7b:9d:30:4f:ab  txqueuelen 1000  (Ethernet)
[andy@kubuntu:~/dev/demo/plotly.py]$ ifconfig | grep -i -A2 ':4F:AB'
        ether 50:7b:9d:30:4f:ab  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
{% endhighlight %}

`-r` will do a recursive search within sub directories
`-c` will count the number of matches
`-n` will show the line number of the matches
and many more other useful options which you can read all about in [grep's manual page](https://linux.die.net/man/1/grep)

#### Regular Expressions (RegEx)
The examples above introduce `grep` utility for absolute beginners, but they do *no justice* in showcasing `grep`'s power. Grep's power comes from combining `grep` with regular expressions.
Regular expressions are a compact way of describing complex patterns in text. You can use regex to search for all kinds of expressions, such as finding phone numbers, email addresses, lines that start or end with a specific word or letter or even a pattern. Regex are very powerful. Let's go over a couple of basic examples of `grep` + regex
Supposed you have a file called `space_oddity.txt` that contains the lyrics to the famous song [Space Oddity](https://genius.com/David-bowie-space-oddity-lyrics), you can search for all lines starting with the word "Can":
{% highlight bash %}
[andy@kubuntu:~/dev/demo]$ grep "^Ground" space_oddity.txt
[andy@kubuntu:~/dev/demo]$ grep -n "^Can" space_oddity.txt
26:Can you hear me, Major Tom?
27:Can you hear me, Major Tom?
28:Can you hear me, Major Tom?
29:Can you "Here am I floating 'round my tin can
{% endhighlight %}

Search for all lines ending with "Tom"
{% highlight bash %}
[andy@kubuntu:~/dev/demo]$ grep "Tom$" space_oddity.txt
Ground Control to Major Tom
Ground Control to Major Tom
This is Ground Control to Major Tom
Ground Control to Major Tom
{% endhighlight %}

Search for all lines ending with "Tom" + zero or more characters. The `.` modifier means any character and the `{0,1}` means at least zero and at most one occurrence. 
{% highlight bash %}
[andy@kubuntu:~/dev/demo]$ egrep "Tom.{0,1}$" space_oddity.txt 
Ground Control to Major Tom
Ground Control to Major Tom
This is Ground Control to Major Tom
Ground Control to Major Tom
Can you hear me, Major Tom?
Can you hear me, Major Tom?
Can you hear me, Major Tom?
{% endhighlight %}

Grep for an IPv4 Address
{% highlight bash %}
[andy@kubuntu:~/dev/demo]$ ifconfig | grep  -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"
        inet 127.0.0.1  netmask 255.0.0.0
        inet 192.168.1.21  netmask 255.255.255.0  broadcast 192.168.1.255
{% endhighlight %}

For a detailed explanation of grep and regex, checkout [GNU grep documentation](https://www.gnu.org/software/grep/manual/grep.html)

### Grep Summary
#### Option examples

<table>
<thead>
<tr class="header">
<th>Option</th>
<th>Example</th>
<th>Operation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>-i</code></td>
<td>grep -i ':4F:AB' net_interfaces.txt</td>
<td>Ignores case sensitivity</td>
</tr>
<tr class="even">
<td><code>-w</code></td>
<td>grep -w "connect" /var/log/syslog </td>
<td>Search for the full word</td>
</tr>
<tr class="odd">
<td><code>-A</code></td>
<td>grep -A 3 'Exception' error.log</td>
<td>Display 3 lines of context after matching string</td>
</tr>
<tr class="even">
<td><code>-B</code></td>
<td>grep -B 4 'Exception' error.log</td>
<td>Display 4 lines of context before matching string</td>
</tr>
<tr class="odd">
<td><code>-C</code></td>
<td>grep -C 5 'Exception' error.log</td>
<td>Display 5 lines around matching string</td>
</tr>
<tr class="even">
<td><code>-r</code></td>
<td>grep -r 'quickref.me' /var/log/nginx/</td>
<td>Recursive search within subdirs</td>
</tr>
<tr class="odd">
<td><code>-v</code></td>
<td>grep -v 'warning' /var/log/syslog</td>
<td>Returns all non-matching lines</td>
</tr>
<tr class="even">
<td><code>-e</code></td>
<td>grep -e '^Can' space_oddity.txt</td>
<td>Use regex <em>(lines starting with 'Can')</em></td>
</tr>
<tr class="odd">
<td><code>-E</code></td>
<td>grep -E 'ja(s|cks)on' filename</td>
<td>Extended regex <em>(lines containing jason or jackson)</em></td>
</tr>
<tr class="even">
<td><code>-c</code></td>
<td>grep -c 'error' /var/log/syslog</td>
<td>Count the number of matches</td>
</tr>
<tr class="odd">
<td><code>-l</code></td>
<td>grep -l 'reboot' /var/log/*</td>
<td>Print the name of the file(s) of matches</td>
</tr>
<tr class="even">
<td><code>-o</code></td>
<td>grep -o search_string filename</td>
<td>Only show the matching part of the string</td>
</tr>
<tr class="odd">
<td><code>-n</code></td>
<td>grep -n "start" demo.txt</td>
<td>Show the line numbers of the matches</td>
</tr>
</tbody>
</table>

### Grep Regular Expressions

<table>
<tbody>
<tr class="odd">
<td><code>^ </code></td>
<td>Beginning of line.</td>
</tr>
<tr class="even">
<td><code>$ </code></td>
<td>End of line.</td>
</tr>
<tr class="odd">
<td><code>^$</code></td>
<td>Empty line.</td>
</tr>
<tr class="even">
<td><code>\&lt;</code></td>
<td>Start of word.</td>
</tr>
<tr class="odd">
<td><code>\&gt;</code></td>
<td>End of word.</td>
</tr>
<tr class="odd">
<td>.</td>
<td>Any character.</td>
</tr>
<tr class="even">
<td><code>?            </code></td>
<td>Optional and can only occur once.</td>
</tr>
<tr class="odd">
<td><code>*            </code></td>
<td>Optional and can occur more than once.</td>
</tr>
<tr class="even">
<td><code>+            </code></td>
<td>Required and can occur more than once.</td>
</tr>
<tr class="odd">
<td><code>{n}          </code></td>
<td>Previous item appears exactly n times.</td>
</tr>
<tr class="even">
<td><code>{n,}         </code></td>
<td>Previous item appears n times or more.</td>
</tr>
<tr class="odd">
<td><code>{,m}         </code></td>
<td>Previous item appears n times maximum.</td>
</tr>
<tr class="even">
<td><code>{n,m}        </code></td>
<td>Previous item appears between n and m times.</td>
</tr>
<tr class="odd">
<td><code>[:alpha:]   </code></td>
<td>Any lower and upper case letter.</td>
</tr>
<tr class="even">
<td><code>[:digit:]   </code></td>
<td>Any number.</td>
</tr>
<tr class="odd">
<td><code>[:alnum:]   </code></td>
<td>Any lower and upper case letter or digit.</td>
</tr>
<tr class="even">
<td><code>[:space:]    </code></td>
<td>Any whites­pace.</td>
</tr>
<tr class="odd">
<td><code>[A-Z­a-z]    </code></td>
<td>Any lower and upper case letter.</td>
</tr>
<tr class="even">
<td><code>[0-9]        </code></td>
<td>Any number.</td>
</tr>
<tr class="odd">
<td><code>[0-9­A-Z­a-z]</code></td>
<td>Any lower and upper case letter or digit.</td>
</tr>
</tbody>
</table>

