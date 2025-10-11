---
layout: posts
title:  "Creating your first GitHub page with Jekyll"
date:   2025-09-28 21:31:00 -0500
categories: github jekyll
---
<!-- excerpts go here, right after the front matter -->
Ever wonder how people create blogs and posts in the blink of an eye? Well, now you can too!!! using GitHub pages and Jekyll, create your very first blog in under an hour--well, maybe two--and amaze your friends and family and dog with your new found skills.

---

# Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Installing Ruby and Jekyll](#installing-ruby-and-jekyll)
- [Create a Blank Repository on GitHub](#create-a-blank-repository-on-github)
- [Using Jekyll to create a barebones page](#using-jekyll-to-create-a-barebones-page)
- [Visualizing changes locally](#visualizing-changes-locally)
- [Overriing a theme's design](#overriing-a-themes-design)
- [Using different themes](#using-different-themes)
- [Conclusion](#conclusion)

---

# Introduction

This blog is a compilation of notes I took while going through this [tutorial](https://www.youtube.com/watch?v=fV01b0duZwU) on YouTube on how to build your first website on GirHub with Jekyll.

# Installing Ruby and Jekyll
1. Go to [Ruby Installer](https://rubyinstaller.org/downloads/) and get the latest installer (including DEVKIT) for Windows.
2. Run the installation with the default settings; when it asks to install the MSYS toolchain, choose option 3: MSYS and MSYS2. Once the install is done, it'll ask the same set of questions again, hit ENTER to exit.
3. Since I will be using the bash terminal to run ruby, modify the `.bashrc` to include the newly installed ruby binaries in the system path,

    ```bash
    export PATH=${PATH}:/c/Ruby34-x64/bin
    ```

4. run `gem update`.
5. Install `jekyll` and `bundler`.
    ```bash
    $ gem install jekyll bundler
    ```
6. run `jekyll -v`, you should get a response like `jekyll 4.4.1`

# Create a Blank Repository on GitHub
1. Log into your github account
1. Create a new repository with the following name: `<github_username>.github.io`

![Create a new GitHub Repository][img_create_repo]

# Using Jekyll to create a barebones page
We are going to force jekyll to create a page in our github webpage repository, `vishalcoelho.github.io` in my case.

1. cd into the root of the repo
1. Create a new site

    ```bash
    $ jekyll new --skip-bundle .
    **Conflict**: C:/Users/savio/repositories/vishalcoelho.github.io exists and is not empty.
    Ensure C:/Users/savio/repositories/vishalcoelho.github.io is empty or else try again with `--force` to proceed and overwrite any files.
    ```
    > :warning: It's complaining about the repo being empty, use `--force`

    ```bash
    $ jekyll new --skip-bundle . --force
    New jekyll site installed in C:/Users/savio/repositories/vishalcoelho.github.io.
    Bundle install skipped.
    ```
    <!-- If you want to center an image, you need to use the path (not link) in an img tag, as opposed to -->
    <!-- ![Jekyll creates a blank project][img_jekyll_blank] -->
    <img
        style="display: block;
               margin-left: auto;
               margin-right: auto;
               width = 30%;"
        src="/assets/images/2025_09_28_post_first_github_page/new_jekyll_creation.png">

1. Edit the *Gemfile*:
   1. comment `gem "jekyll", "~> 4.4.1"` as GitHub is currently pinned to v3
   2. Uncomment `gem "github-pages", group: :jekyll_plugins` and add the version for github-pages, which you can fine on [GitHub versions](https://pages.github.com/versions/)

      `gem "github-pages", "~>232", group: :jekyll_plugins`
   3. Run `bundle install`

        ```bash
        $ bundle install
        [DEPRECATED] Platform :mingw, :x64_mingw, :mswin is deprecated. Please use platform :windows instead.
        Fetching gem metadata from https://rubygems.org/...........
        Resolving dependencies...
        ...stuff...
        Bundle complete! 7 Gemfile dependencies, 100 gems now installed.
        Use `bundle info [gemname]` to see where a bundled gem is installed.
        ```

2. Verify changes in `Gemfile.lock` then add this file to `.gitignore`

# Visualizing changes locally

Run `bundle exec jekyll serve` to serve up the site locally, and `Ctrl+C` to shut it down when done live editing.

   ![Host site on local server][img_local_server]


# Overriing a theme's design

Run `bundle info --path <theme>` to reveal where that particular theme's gem was installed on your local machine. For
example, `bundle info --path minimal-mistakes` shows us where the minimal-mistakes theme is installed

 *C:/Ruby34-x64/lib/ruby/gems/3.4.0/gems/minimal-mistakes-jekyll-4.27.3*

![Minimal Mistakes theme directory][img_minimal_mistakes_theme]

If I wanted to change the design of the posts, I simply override it by including my own *_layout/posts.html* in my GitHub repository. Any local html files supercede the default ones packaged in the theme's gem.

# Using different themes

You can find more themes at [GitHub Pages Themes](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll). To use a theme simply edit the _config.yml

1. Change theme from the default `minima` to `jekyll-theme-<theme>`. For example, if i wanted to change to the hacker theme, the line would look like

    ```ruby
    theme: jekyll-theme-hacker
    ```

1. This alone isn't sufficient to render locally

    ![Fail to use theme locally][img_jekyll_serve_fail]

    It is complaining about layouts `page`, `home` not existing. `hacker` only supports the `default` and `post` layouts (check the _layout folder in the theme's repository).

    So, change all `home` & `page` -> `default`

1. We are missing the links `about`, `contact`, so how do we customize this theme?
   1. create a new folder `_layouts` where we  will override the theme. Create an `index.html`
   1. lets borrow from the [Minima](https://) theme, specifically `home.html` and place it in our newly created `index.html`; we will be overriding the `default` layout
   1. update index.markdown to point to the new layout in the front matter.

   change `layout:default` to `layout:index`

1. Copy `Hacker's` [default layout theme](https://github.com/pages-themes/hacker/blob/master/_layouts/default.html)
and edit it to alter how your main page shows up. Having our own altered version of the default layout will override GitHub's version.

# Conclusion

Each jekyll theme has a GitHub page which you can use as a template for your own website. Add and subtract parts as necessary.

[img_create_repo]: /assets/images/2025_09_28_post_first_github_page/create_repo.png
[img_jekyll_blank]: /assets/images/2025_09_28_post_first_github_page/new_jekyll_creation.png
[img_local_server]: /assets/images/2025_09_28_post_first_github_page/serving_site_locally.png
[img_jekyll_serve_fail]: /assets/images/2025_09_28_post_first_github_page/jekyll_serve_fail.png
[img_minimal_mistakes_theme]: /assets/images/2025_09_28_post_first_github_page/minimal_mistakes_theme.png