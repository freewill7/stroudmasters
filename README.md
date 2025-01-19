# Stroud Masters Website

## Background
Broadly speaking there are three common ways of hosting small websites

- Website platforms (e.g. Squarespace, Wix)
  - Easy to use
  - Low maintenance
  - Starts from ~£12 a month
- Content Management Systems (e.g. Wordpress)
  - Fairly easy to use
  - Often requires additional maintenance (backups, updates...)
  - Security can be a concern
  - Starts from ~£6 a month
- Statically generated sites
  - Just a bunch of files
  - Not quite as user friendly to update... but a lot fewer moving parts
  - Starts from ~£0.50 a month

We originally used a CMS (Wordpress) but the ongoing maintenance was a bit of a pain, and keeping costs low usually involved changing hosting providers every year.

Experimenting with a statically generated site was pretty encouraging- there was a reasonable workflow for making updates, maintenance tasks were low and the pricing was very competitive.

However this is coming from someone with a computer science background. Other factors may be important for future maintainers of the site.

## Overview

Here is the rough architecture of the current site

```mermaid
architecture-beta

group github(cloud)[Github]
service git(disk)[Version control] in github

service start[Start here]

service sources(disk)[Website sources]
service hugo(server)[Hugo Generator]
service generated(disk)[Generated website]

group hosting(cloud)[AWS Hosting]
service s3(disk)[S3 Storage] in hosting
service cloudfront(internet)[Cloudfront] in hosting
service route53(server)[Route53] in hosting

service client(server)[Visitor]

start:R --> L:sources
sources:T <--> B:git
s3:R --> L:cloudfront
sources:R --> L:hugo
hugo:R --> L:generated
generated:R --> L:s3
cloudfront:R <-- L:client
route53:T <-- L:client
```

Moving from the start point on the far left

- Website sources

  These are the sources used to generate the website. They are composed of Markdown (.md) files and other associated media that get processed by a program called [Hugo](https://gohugo.io) to generate the HTML required for the website.

  The nice thing about Markdown is that it's a lightweight language and so allows us to focus on the content rather than it's presentation. Hugo takes the Markdown and does all the time consuming and annoying stuff for us, for example

  - Creating html that works correctly on mobile and desktop
  - Applying themes to make content look nice
  - Automatically organising content by date and time
  - Creating sitemaps and other structures required by search engines

- Version control (optional)

  The sources for the site are stored in a Version control system.

  Version control systems allow us to register files and formally record how things change over time. Most importantly they are an excellent way of performing backups- an accidental deletion of a page can be easily restored from the Version control system.

  There are variety of Version control systems out there but the trend has generally been towards `git`. Microsoft provides a free git hosting service called github which is what we currently use.

  If my laptop were to break then all the website sources are safely backed up on our [github repository](https://github.com/freewill7/stroudmasters).

- Hugo Generator

  Previously mentioned- this is a program that changes the website sources into html. More on this later.

- Generated website

  We now have a generated website (the "public" directory) but this needs to publicly available in order for people to see it (i.e. "hosted). This is where we use Amazon Web Services (AWS).

  There are lots of providers that can be used for hosting websites (Microsoft Azure, Google Cloud...) but we went for AWS as it's well established and reasonably priced.

- S3 Storage

  This is a file storage service provided my Amazon. It can be configured in lots of different ways... but we keep things simple and have everything read-only (so only we can modify things).
  
  Our `Generated website` is stored in S3.

- Cloudfront

  This is a caching service provided by Amazon. Traditionally this is used to improve website performance, however in our case it's also a cost saving feature.

  Fetching an individual object directly from S3 means Amazon has to fully retrieve the object (in case it's been modified since the last time it was read). And this is reflected in the cost ($0.0004	per 1000 requests).

  However by using a cache, and alerting Amazon when an object has changed, this allows them to better optimise their computing resources. And in exchange we receive 10,000,000 free requests every month before prices return to normal S3 pricing.
  
  In practice this is probably a negligible saving but it's good practice and protects us if we somehow go viral. (We also have budget caps to ensure no surprises when it comes to billing).

- Route 53

  This is the service used for domain name registration.

  We pay them a fixed amount of every year so that requests to stroudmasters.org go to the appropriate place.

- Visitor

  Finally we have a website visitor.

  When a link is clicked or someone types stroudmasters.org into their browser then the Route53 service redirects them to our Cloudfront service.
  
  Cloudfront then serves a cached version of our website and everyone is happy.

## Generating the website 

### Setting up hugo
The only true requirement for building the website is **hugo** which has install instructions on their website [https://gohugo.io](https://gohugo.io). Ideally you should be able to install it through a package manager, or worst case use a pre-built version (e.g. at [https://github.com/gohugoio/hugo/releases/tag/v0.138.0](https://github.com/gohugoio/hugo/releases/tag/v0.138.0) ).

With hugo installed, I can open the terminal on my Mac and run

```sh
hugo version
```

And have the following output 

```
hugo v0.123.8-5fed9c591b694f314e5939548e11cc3dcb79a79c+extended darwin/arm64 BuildDate=2024-03-07T13:14:42Z VendorInfo=brew
```

### Website sources

The website sources can be found at [https://github.com/freewill7/stroudmasters](https://github.com/freewill7/stroudmasters).

Clicking on the `<> Code` button should present options to download the sources.

If you want to continue using git to manage changes then you'll want to select one of the `Clone` options or the `Open with GitHub Desktop`.

If you just want to download the sources to see what's currently being used, then `Download ZIP` is an easy option with minimal dependencies.

After you've downloaded the sources go to the `stroudmasters` folder on your computer (this may require unzipping the downloaded file.)

```sh
cd stroudmasters
ls -1 # or DIR on windows
```

This results in output such as the following

```
README.md
config.toml
content
layouts
public
static
sync.sh
themes
```

### Testing the sources

With everything set up you should be able to run

```
hugo serve
```

This will compile the sources and set-up a local webserver to test the generated content. On my laptop the command returns

```
...
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1) 
```

Visting this address in a browser will give you a preview of the site. You can stop the webserver by pressing `Ctrl-c` in your terminal

### Guide to the sources

* config.toml 

  These are the main settings for hugo, things like
  - Theme
  - Menu entries
  
  We rarely have to change these

* content

  This is where the textual content for the site lives. Each directory maps will map onto a URL with the accompanying `_index.md` being the content.
  
  For example, the home page is `content/_index.md` (equivalent to https://stroudmasters.org). And the calendar lives at `content/calendar/_index.md` (equivalent to https://stroudmasters.org/calendar).

  The `posts` directory is a bit more fancy. hugo will automatically order entries based upon their date (this is how we implement https://stroudmasters.org/news ). 

  For example, the `content/posts/2024/11/tuesday_12_session.md` has **date** and **title** metadata used for creating lists.

* layouts

  A directory used by hugo- rarely has to be touched.

* static
  
  This is where static, binary resources such as images and documents are stored. These are organised into date based directories to help keep things manageable.

  For example the new membership form lives in `images/2024/11/Membership_Form_2025.pdf` which reflects when it was uploaded.

* themes

  This is where hugo retrieves it's theming info from. Rarely needs to be touched.

* public

  This is where hugo will put the generated website. This should never be modified directly.

### Modifying the sources

Due to the sources being text files there are lots of options for modifying them- from something as simple as Notepad to more sophisticated options such as [vscode](https://code.visualstudio.com).

I'd recommend looking at into the vscode option. It provides a simple UI and has lots of tools to help with the job (e.g. git integration and markdown previews).

### Common tasks

Here is my workflow for some common tasks

#### Updating the membership form

1. Rename the form so it doesn't contain spaces

    ```sh
    mv Membership\ Form\ 2025.pdf Membership_Form_2025.pdf
    ```

2. Copy the membership form into the static resources directory 

    ```sh
    cp ~/Downloads/Membership_Form_2025.pdf static/images/2024/11/
    ```

3. (Optionally) register the file with the version management system

    ```sh
    git add static/images/2024/11/Membership_Form_2025.pdf
    ```

4. Update the `about` section to point to the new form

    ```sh
    open content/about/_index.md
    ```

    ```md
    - [Membership Form 2024](/images/2024/11/Membership_Form_2025.pdf)
    ```

5. Test the changes

    ```sh
    hugo serve
    ```

    Browse to `http://localhost:1313` and click through to `About` and check that the new form can be accessed.

5. (Optionally) Add the changes to the about page to version control

    ```sh
    git add content/about/_index.md
    ```

5. (Optionally) Commit the changes to version control

    ```
    git commit
    ```

6. (Optionally) back up version control changes remotely

    ```
    git push
    ```

7. Publish to AWS

    See the `Updating AWS copy of generated website` later in this document.

#### Adding a news post

1. Create a directory for the current month

    ```sh
    mkdir content/posts/2024/11
    ```

2. Copy a previous entry or fill in a template

    ```sh
    vim content/posts/2024/11/exciting_news.md
    ```

    Example:

    ```md
    ---
    title: "TODO TITLE"
    date: 2024-11-06T17:44:00+0000
    draft: false
    ---
    TODO CONTENT
    <!--more-->
    ```

    Note:
    - date is in GMT which might trip you up in the summer
    - the `<!--more>` tag prevents content from being clipped on the news page

3. Test the changes

    ```
    hugo serve
    ```

4. (Optionally) Add the new entry to version control

    ```sh
    git add content/posts/2024/11/exciting_news.md
    ```

5. (Optionally) Commit the change to version control

    ```sh
    git commit
    ```

6. (Optionally) back up version control changes remotely

    ```
    git push
    ```

7. Publish to AWS

    See the `Updating AWS copy of generated website` later in this document.


### Updating AWS copy of generated website

To perform the synchronisation with AWS you'll need the [awscli](https://aws.amazon.com/cli/) utility along with ssh keys.

- Generate website

  This is slightly different to the `hugo serve` command that we use to test changes

  ```sh
  hugo
  ```

- Synchronise with s3

  ```
  python3 -m awscli s3 sync --exclude "*.DS_Store*" ./public s3://stroudmasters.org
  ```

- Refresh cloudfront cache

  ```
  python3 -m awsclient cloudfront create-invalidation --distribution-id [SECRET] --paths '/*'
  ```

