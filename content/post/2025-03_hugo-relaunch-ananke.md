---
title: "Hugo Relaunch"
date: 2025-03-08T20:03:52Z
draft: false

featured_image: ""
---

## Setting up Hugo with Ananke Theme as Module

**The intention of this post is to walk through the steps I performed set up a static website with Hugo, add the Ananke theme as module, and deploy the website via AWS Amplify.**

The reason for me to write this blog post is that I was hoping to find exactly this documentation somewhere online. But as I could not find it and worked my way through different blogs and videos, I thought, maybe other people might benefit from such a summary.

Originally, I just wanted to relaunch my blog. This blog right here. But then I ran into issues because of updates on the Hugo side. And then I ran into even more issues with Ananke. From what I gathered, the Ananke theme transitioned to a new maintainer. And with that, a couple of updates were implemented. And at this point, I want to say "Thank you". Thank you to the poepl who created the theme. And Thank you to the people to help updating it.

## Setup

### VS Code - Terminal
I am working on a Mac and in VS Code.

First, make sure Hugo is installed:

````
$ hugo version
`````

returning

> hugo v0.145.0+extended+withdeploy darwin/arm64 BuildDate=2025-02-26T15:41:25Z VendorInfo=brew


Also, check the go version
````
$ go version
````

returning

> go version go1.23.5 darwin/arm64


Pre-requisite: VS Code is already connected to GitHub account.

````
$ cd source_code
````

taking me to 
`/Users/someusername/source_code`

````
hugo new site davidbeck-online-relaunch-test
````

Returning:
> Congratulations! Your new Hugo site was created in /Users/davibec/source_code/davidbeck-online-relaunch-test.
Just a few more steps...
Change the current directory to /Users/someusername/source_code/davidbeck-online-relaunch-test.
Create or install a theme:
Create a new theme with the command "hugo new theme <THEMENAME>"
Or, install a theme from https://themes.gohugo.io/
Edit hugo.toml, setting the "theme" property to the theme name.
Create new content with the command "hugo new content <SECTIONNAME>/<FILENAME>.<FORMAT>".
Start the embedded web server with the command "hugo server —buildDrafts".
See documentation at https://gohugo.io/.

Then, in VS Code - open folder

![hugo_01_vscode](/images/2025/2025-03_hugo_relaunch/hugo_01_vscode.png)

Go to “Source Control”
* “Initialize Repository”

![hugo_02_vscode](/images/2025/2025-03_hugo_relaunch/hugo_02_vscode.png)

Add changes to Stage via PLUS.

![hugo_03_vscode](/images/2025/2025-03_hugo_relaunch/hugo_03_vscode.png)

Add Comment and Commit.

![hugo_04_vscode](/images/2025/2025-03_hugo_relaunch/hugo_04_vscode.png)

![hugo_05_vscode](/images/2025/2025-03_hugo_relaunch/hugo_05_vscode.png)

Next, “publish branch”.
In my case, I select to create a public branch.

![hugo_06_vscode](/images/2025/2025-03_hugo_relaunch/hugo_06_vscode.png)

First commit is confirmed.

![hugo_07_vscode](/images/2025/2025-03_hugo_relaunch/hugo_07_vscode.png)

Resulting repository is:
https://github.com/davidbeckonline/davidbeck-online-relaunch-test
