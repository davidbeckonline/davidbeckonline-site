---
title: "Hugo Relaunch"
date: 2025-03-08T20:03:52Z
draft: false

featured_image: "/images/2025/2025-03_hugo_relaunch/hugo_19_web.png"
---

## Setting up Hugo with Ananke Theme as Module

**The intention of this post is to walk through the steps I performed set up a static website with Hugo, add the Ananke theme as module, and deploy the website via AWS Amplify.**

The reason for me to write this blog post is that I was hoping to find exactly this documentation somewhere online. But as I could not find it and worked my way through different blogs and videos, I thought, maybe other people might benefit from such a summary.

Originally, I just wanted to relaunch my blog. This blog right here. But then I ran into issues because of updates on the Hugo side. And then I ran into even more issues with Ananke. From what I gathered, the Ananke theme transitioned to a new maintainer. And with that, a couple of updates were implemented. And at this point, I want to say "Thank you". Thank you to the poepl who created the theme. And Thank you to the people to help updating it.

## Setup

### Create a Hugo Site
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
$ hugo new site davidbeck-online-relaunch-test
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

![hugo_08_github](/images/2025/2025-03_hugo_relaunch/hugo_08_github.png)


### Install the Ananke theme as module

Next step is to set up Hugo for the usage of modules.

In VS Code Terminal

````
$ cd davidbeck-online-relaunch-test
````

````
$ hugo mod init github.com/davidbeckonline/davidbeck-online-relaunch-test
````

Returning:
> go: creating new go.mod: module github.com/davidbeckonline/davidbeck-online-relaunch-test
hugo: to add module requirements and sums:
hugo mod tidy


This step is creating a new go.mod file:

![hugo_09_vscode](/images/2025/2025-03_hugo_relaunch/hugo_09_vscode.png)


Next:

````
$ hugo mod tidy
````
(no output)

````
$ hugo mod get github.com/theNewDynamic/gohugo-theme-ananke/v2
````

Returning:

> go: added github.com/theNewDynamic/gohugo-theme-ananke/v2 v2.12.0

![hugo_10_vscode](/images/2025/2025-03_hugo_relaunch/hugo_10_vscode.png)

Adjust hugo.toml

````
baseURL = 'https://www.davidbeck.online'
languageCode = 'en-us'
title = 'Blog Relaunch Test'

theme = ['github.com/theNewDynamic/gohugo-theme-ananke/v2']
````

Test via

````
$ hugo server
````

Returning:

> Watching for changes in /Users/davibec/source_code/davidbeck-online-relaunch-test/{archetypes,assets,content,data,i18n,layouts,static}
Watching for config changes in /Users/davibec/source_code/davidbeck-online-relaunch-test/hugo.toml, /Users/davibec/source_code/davidbeck-online-relaunch-test/go.mod
Start building sites …
hugo v0.145.0+extended+withdeploy darwin/arm64 BuildDate=2025-02-26T15:41:25Z VendorInfo=brew

> | EN
-------------------+-----
Pages | 8
Paginator pages | 0
Non-page files | 0
Static files | 1
Processed images | 0
Aliases | 0
Cleaned | 0

> Built in 29 ms
Environment: "development"
Serving pages from disk
Running in Fast Render Mode. For full rebuilds on change: hugo server —disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop

![hugo_11_vscode](/images/2025/2025-03_hugo_relaunch/hugo_11_web.png)

Commit and push changes:

![hugo_12_vscode](/images/2025/2025-03_hugo_relaunch/hugo_12_vscode.png)

Resulting in:

![hugo_13_github](/images/2025/2025-03_hugo_relaunch/hugo_13_github.png)


### Deploy on AWS Amplify

To complete the setup, I deploy the website via AWS amplify.

AWS.

Amplify.

“Create new App”.

Select GitHub.

![hugo_14_aws](/images/2025/2025-03_hugo_relaunch/hugo_14_aws.png)

Select your repository:

![hugo_15_aws](/images/2025/2025-03_hugo_relaunch/hugo_15_aws.png)

![hugo_16_aws](/images/2025/2025-03_hugo_relaunch/hugo_16_aws.png)

Click “Edit YAML”

Update YAML to

````
version: 1
env:
  variables:
    # Application versions
    DART_SASS_VERSION: 1.81.0
    GO_VERSION: 1.23.5
    HUGO_VERSION: 0.145.0
    # Time zone
    TZ: Europe/Amsterdam
    # Cache
    HUGO_CACHEDIR: ${PWD}/.hugo
    NPM_CONFIG_CACHE: ${PWD}/.npm
frontend:
  phases:
    preBuild:
      commands:
        # Install Dart Sass
        - curl -LJO https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz
        - sudo tar -C /usr/local/bin -xf dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz
        - rm dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz
        - export PATH=/usr/local/bin/dart-sass:$PATH

        # Install Go
        - curl -LJO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
        - sudo tar -C /usr/local -xf go${GO_VERSION}.linux-amd64.tar.gz
        - rm go${GO_VERSION}.linux-amd64.tar.gz
        - export PATH=/usr/local/go/bin:$PATH

        # Install Hugo
        - curl -LJO https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz
        - sudo tar -C /usr/local/bin -xf hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz
        - rm hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz
        - export PATH=/usr/local/bin:$PATH

        # Check installed versions
        - go version
        - hugo version
        - node -v
        - npm -v
        - sass --embedded --version

        # Install Node.JS dependencies
        - "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci --prefer-offline || true"

        # https://github.com/gohugoio/hugo/issues/9810
        - git config --add core.quotepath false
    build:
      commands:
        - hugo --gc --minify
  artifacts:
    baseDirectory: public
    files:
      - '**/*'
  cache:
    paths:
      - ${HUGO_CACHEDIR}/**/*
      - ${NPM_CONFIG_CACHE}/**/*
````

Save and deploy.

![hugo_17_aws](/images/2025/2025-03_hugo_relaunch/hugo_17_aws.png)

Wait a little bit more than a minute...

![hugo_18_aws](/images/2025/2025-03_hugo_relaunch/hugo_18_aws.png)

Et voila.

![hugo_19_web](/images/2025/2025-03_hugo_relaunch/hugo_19_web.png)