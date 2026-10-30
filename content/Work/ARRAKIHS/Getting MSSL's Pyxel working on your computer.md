This guide discusses ESA's Pyxel project and specifically MSSL's version of the new code being delivered as part of the ARRAKIHS project.

## Where's the code?
The code is stored on a website called GitLab. This website has different containers called repositories. If GitLab is a library, a repository is a book. Each repository can also have different branches. Think of these branches as revisions and different editions to the same book. MSSL's Pyxel is currently on a branch of the Pyxel Repository on GitLab.

## Installing Git
Git is a version manager. This basically means that, by running some commands or clicking some buttons on the GUI, it will replace files on your laptop with files on a server. It can also update the files on the server from files on your laptop, depending on what you ask it to do. We'll use Git to get the files from the GitLab server onto our computer. Git is a piece of software, so we'll have to install it. The installation instructions differ by platform and if you're using a package manager, so I'll just link to their install page. Also, you may want to use the Graphical User Interface rather than the command line. GitHub also have their own GUI which I've heard good things about (in fact, I'd actually recommend using GitHub for Desktop for those not beginning a career in software engineering. If you *are* beginning a career in software engineering, it's up to you really. Being comfortable with the command line is very important, that's all I'll say).

Go ahead and install your chosen tool from one of these two places: 
- [git]([git](https://git-scm.com/install/))
- [GitHub for Desktop](https://github.com/apps/desktop)

## Getting the code
Alright, now for the juicy bit. Go to the [ESA Pyxel page](https://gitlab.com/esa/pyxel). You'll see near the top left of the page (just under the square Pyxel logo) there's a dropdown that is prefilled with `master`. This is the current branch you're viewing, and to have a look at other branches, you can simply click it and the rest will appear in the dropdown. We're actually going to use the dropdown to the right of the page labelled `Code`.

![[Pasted image 20261030111254.png]]

If you're using GitHub Desktop, you can use [this guide for adding the repository](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-a-repository-from-github-to-github-desktop). Otherwise, you can use the `Open Existing Repository` option in git GUI or, if you're using the command-line tool, type `git clone [url]` with `[url]` being the https link copied and pasted from the `Code` button in the screenshot above. Just paste the link as is, without using the square brackets.

## Now what?
Congratulations, you have the code! Now you can open this in your code editor of choice (I like [VS Code](https://code.visualstudio.com/)) and you have a fully working version of Pyxel. This is the `master` branch, the main flagship version of the code. To get MSSL's development branch, you need to switch to the `mssl_dev` branch. How to do that will depend on which tool you're using.

With the Git GUI tool, look at the `Branch` menu item at the top. For GitHub for Desktop, use the `Current Branch` menu item at the top and search. If you're using the command line, it's as simple as typing `git checkout mssl_dev`. 

You now have our special super cool version of the code.

## Is this everything?
No. You  have the code that we'll be delivering. You *don't* have the scripts and data files that I've been using to build the plots. If you need that, reach out to me and I'll find some way to get it to you.