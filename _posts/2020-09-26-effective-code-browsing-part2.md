---
layout: single
classes: wide
title: "Effective Code Browsing - Part 2"
categories: posts
excerpt: "Exploring Git-Grep, Cscope and Opengrok"
tags: [linux, tools, grep, regex, cscope, vim, git-grep, opengrok]
comments: true
toc: true
toc_label: "Table of Content"
share: true
---

## Overview
In the [previous post]({% post_url 2020-08-16-effective-code-browsing-part1 %}), we talked about `grep` and `git-grep` command utilities for quickly and efficiently searching through a code base. We showed good examples of how to use these tools effectively. In this post, we will continue where we left off, and talk about `git-grep`, `cscope` and `opengrok`.

## Git-Grep
[git-grep](https://git-scm.com/docs/git-grep) is a git feature that searches for text in a repo's tracked files. It works the same as the `grep` utility, but is much faster than `grep` because it only searches tracked project files. It connects with the repo's git database, and allows you to search specific branches or tags. 

### Practical Examples
#### Search for specific keyboard in a repo:
{% highlight bash %}
git grep YouTube
{% endhighlight %}
![Basic Git Usage](../../assets/images/git_grep_basic.png)

#### Limit search to a specific file type
The results from the previous example show matches for markdown and text files. We might be only interested in the Python files, so we can limit the search to all types of Python files by using regex.
{% highlight bash %}
$ git grep YouTube -- '*.py'
{% endhighlight %}
![Limit to Specific Files](../../assets/images/git_grep_py_only.png)

We can extend on this to search a string in a selection of file types. For example, we would like to search Python and text files.
{% highlight bash %}
$ git grep YouTube -- '*.py' '.*txt'
{% endhighlight %}
![Limit to Specific Files](../../assets/images/git_grep_multiple_files.png)

#### Search in another branch
Because `git-grep` is a git utility, it provides us access to git's database. It allows us to search a string or a regex on a specific branch. For example, first let's search for the word `token` in the current and then let's repeat the search, but this time searching specifically in the `env-tokens` branch
{% highlight bash %}
$ git grep token env-tokens
{% endhighlight %}
![Search in another branch](../../assets/images/git_grep_search_branch.png)

We can extend this by searching specific files (as show in previous examples)
{% highlight bash %}
$ git grep token env-tokens -- '*.md'
{% endhighlight %}
![Limit to Specific Files in another branch](../../assets/images/git_grep_search_branch_file_type.png)

#### Search in another branch
Because `git-grep` is a git utility, it provides us access to git's database. It allows us to search a string or a regex on a specific branch. For example, first let's search for the word `token` in the current and then let's repeat the search, but this time searching specifically in the `env-tokens` branch
{% highlight bash %}
$ git grep token env-tokens
{% endhighlight %}
![Search in another branch](../../assets/images/git_grep_search_branch.png)

#### Search files containing multiple keywords
As an example, we search for files containing "dog" and "fox" on the same line, and then "Downloads" and "YouTube" on the same line.
{% highlight bash %}
$ git grep -e "fox" --and -e "dog"
{% endhighlight %}
![Search in another branch](../../assets/images/git_grep_multiple_keywords.png)

There are many other useful options. Refer to [git-grep documentation](https://git-scm.com/docs/git-grep) for a list of available options




## Cscope
`cscope` is an interactive Linux tool that allows the user to browse through source files for specified elements of code in a terminal environment. It is best suited for C/C++ or Java projects, but can be [extended to other projects such as Python](https://stackoverflow.com/questions/3718868/using-cscope-to-browse-python-code-with-vim/3753503). The first time `cscope` is invoked, it builds a symbol database on the source files, which it then uses for answering user queries. [Vim's Help File](https://vimhelp.org/if_cscop.txt.html#cscope) summarize `cscope` as:

> Cscope is an interactive screen-oriented tool that helps you:
> Learn how a C program works without endless flipping through a thick listing.
> Locate the section of code to change to fix a bug without having to
> learn the entire program.
> Examine the effect of a proposed change such as adding a value to an
> enum variable.
>
> Verify that a change has been made in all source files such as adding
> an argument to an existing function.
>
> Rename a global variable in all source files.
>
> Change a constant to a preprocessor symbol in selected lines of files.
> 
> It is designed to answer questions like:
       Where is this symbol used?
       Where is it defined?
       Where did this variable get its value?
       What is this global symbol's definition?
       Where is this function in the source files?
       What functions call this function?
       What functions are called by this function?
       Where does the message "out of space" come from?
       Where is this source file in the directory structure?
       What files include this header file?

### Using Cscope
#### Setup
Before we begin using `cscope`, you should setup the editor that Cscope will open the search in. Usually, the default editor is `vi/vim`, but sometimes it's set to `emacs` or none. You can change the cscope editor using the environment variable `CSCOPE_EDITOR`. For example to change to `emacs`:
{% highlight bash %}
$ export CSCOPE_EDITOR=`which emacs`
{% endhighlight %}

You can also just add this to your `~/.bashrc` file so that way it's permanent.
{% highlight bash %}
$ echo 'export CSCOPE_EDITOR=`which emacs`' >> ~/.bashrc
{% endhighlight %}

#### Generating Databse
On the first using `cscope`, navigate to your project's home directory and run `cscope -R` to generate a symbol database. Once `cscope` finishes generating the database, you will see an interactive screen as show below:
![cscope interactive screen](../../assets/images/cscope_interactive_screen.png)

To exit out of the interactive screen, press `Ctrl + d`
In the same directory, you will find a new file called `cscope.out`. This is the generated database file containing all the generated symbols.

{% highlight bash %}
$ ll -h | grep cscope
-rw-rw-r--   1 andy andy 734M Aug 22 22:33 cscope.out
{% endhighlight %}

#### Finding Symbols and function definitions
Now we're ready to use `cscope` to find symbols or function definitions. For example, let's say we're interested in finding the definition of `struct task_struct` defined in the [Linux Kernel](https://github.com/torvalds/linux). Run `cscope -d` and navigate to "Find this global definition:", and enter `task_struct`. In the results, navigate through the results until you find a file that might have the definition, then press enter (you can also just press the letter mapping to the file
shown on the left most column) to open the file. Press the **spacebar** to navigate to the next screen of the results.
{% highlight bash %}
$ cscope -d
-rw-rw-r--   1 andy andy 734M Aug 22 22:33 cscope.out
{% endhighlight %}
![Cscope Global definition for task scope](../../assets/images/cscope_global_def_task_struct.png)

Found the file containing definition of `struct task_struct` on line 661:
![Cscope Global definition found the file](../../assets/images/cscope_global_def_found_file.png)

Now, let's find the definition for the function `create_kthread` (as an example). Start by running `cscope -d` and then typing `create_kthread` into the *"Find this global definition:"*

Found the function definition inside the file `kthread.c` in line 334:
![Cscope Global function definition found](../../assets/images/cscope_global_func_def.png)

All the options in cscope's interactive prompt are pretty much self explanatory. Using `cscope`, we can find symbols, function definitions, functions calling particular functions, functions called by a particular function and many other things such as changing a text in all files and finding pre-processor constants. Try to explore the other options. Complete list of features and options are found in [cscope's manual](http://cscope.sourceforge.net/cscope_man_page.html)

## OpenGrok
`opengrok` is another powerful and popular tool for browsing source source code. It can understand version control history of many version control systems such as `git`. Unlike the previous tools explained, `opengrok` provides a web interface to the user to search, cross-reference and navigate source code trees. It comes very handy for browsing source code for particular branches, specifically when those branches track different code releases. In addition, since it has a web UI, it makes
referencing code very easy. You can create a clickable link to any line in any source code file. This is very powerful when discussing code over an email or instant messaging like Teams or WebEx.

### Setup
Setup is very simple. You just need to create directories for `src` and `data` which will hold source code projects and opengrok's data respectively, and then download and run a docker image.

1) Make a directory for source code and data configs
{% highlight bash %}
$ mkdir -p /usr/local/opengrok/ /usr/local/opengrok/src /usr/local/opengrok/data/
{% endhighlight %}

2) Clone the git repositories into `src`
{% highlight bash %}
$ cd /usr/loca/opengrok/src
$ sudo git clone https://github.com/plotly/plotly.py.git
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
{% endhighlight %}

3) Download the docker image for opengrok
{% highlight bash %}
$ docker pull scue/docker-opengrok
{% endhighlight %}

4) Run the docker image
{% highlight bash %}
$ docker run -v /usr/local/^Cengrok/src:/src -v /usr/local/opengrok/data:/data -p 8888:8080 scue/docker-opengrok
{% endhighlight %}

5) Navigate to  [http://localhost:8888/source/](http://localhost:8888/source/)

### Browse Source Code
By this step, the source code should be browseable on [http://localhost:8888/source/](http://localhost:8888/source/)
![OpenGrok Home Screen](../../assets/images/opengrok_home_screen.png)

Now you can search for a specific symbol for function name. As an example: `kthreadadd`
![OpenGrok Home Screen](../../assets/images/opengrok_search_function.png)

One of the main features of `opengrok` is the clickable link to an exact line number in a source file. For example, [http://localhost:8888/source/xref/linux-stable/kernel/kthread.c#657](http://localhost:8888/source/xref/linux-stable/kernel/kthread.c#657). This comes in handy when referencing specific line of code over email. Also, because of the nature of `opengrok`,
one can have each release branch as a project in the `src` directory. This makes it easy and possible to reference shipped code or code released some time ago.

This is the end of this blog series.      

Feel free to reach out to me, comment, or ask questions if needed.
