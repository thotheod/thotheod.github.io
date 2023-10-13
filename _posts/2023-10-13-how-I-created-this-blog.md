---
layout: post
title:  "How I create this Blog"
date:   2023-10-10 13:00:00 +0300
categories: blog tech
tags: blog jekyll github-pages
---

I've never been much of a blogger. I didn't use to share my experiences, knowledge, or questions on forums or other social platforms. However, there were times when I found myself doing something, which could be either fairly complex or even quite simple but new to me. In such moments, I felt the need to document it, so I could easily replicate the process in the near future. You see, I tend to forget things rather quickly, so documenting my findings became essential. The big question was, where should I document them? I never had a structured approach for this. After experimenting with various options, I became enamored with [Markdown](https://en.wikipedia.org/wiki/Markdown). I truly appreciated its simplicity, flexibility, and the power it offered. But the issue of where to store these Markdown documents remained unsolved. It could be in local Git repositories, GitHub Repositories, GitHub Gists, plain files, and so on.


I had definitely toyed with the idea of starting a blog at one point, but I wasn't keen on "investing" in the common blogging platforms with their full-fledged [CMS](https://en.wikipedia.org/wiki/Content_management_system). I was aware of Github Pages, and I knew how to work with Markdown, but it was only when I stumbled upon [Thomas Stringer](https://trstringer.com/) that put everything together :smirk:. 


## TL:DR - What I need to get started
I will start by adding some links and basic tips in this paragrpaph for those that just want to get started quickly.
- [jekyll](https://jekyllrb.com/docs/): If you love markdown, and want to avoid html/css/javascript as much as possible, or CMS with databases etc, then Jekyll is all you need. This is __THE best__  static site generator (my blog - my opinion :grin:)
  - Linux Machine or [Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install). Jekyll works on windows as well, but is not [officially supported](https://jekyllrb.com/docs/installation/windows/). 
- GitHub Account to host/use [GitHub Pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll)
- [Chirpy Jekyll theme](https://chirpy.cotes.page/)

## How I create this blog - Detailed Steps

### Setup local machine
I am using a Windows 11 with WSL (running ubuntu 22.00). We need to setup our local machine, so we can author posts, render and test our site (blog) locally; it is really helpful. To do tht we follow the [official instructions for ubuntu](https://jekyllrb.com/docs/installation/ubuntu/)

If we need to verify that everything is working as expected (and if you are curious to have a first taste of how Jekyll is working) do the following steps in your ubuntu shell:

``` bash
# create default jekyll site
jekyll new test-jekyll

# get into the new folder
cd test-jekyll

# run locally
bundle exec jekyll serve
```

Now open the browser and head to `http://localhost:4000`. If you see something similar to the below image, you are all set!

![Default, empty Jekyll site](/images/how-to-blog/new-jekyll.jpg){: width="700" height="400" }
_Default, empty Jekyll site_



### Use Chirpy repo and theme
Jekyl is powerful, but the starting theme you get out-of-the-box is really limited, as you have seen above. You can of course use [Jekyll Themes](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll) and you can of course heavily customise your blog. But you can just focus on what you have to say, and start with a heavily customized and opinionated theme, suitable for technical blogs. Enter [Chirpy](https://chirpy.cotes.page/posts/getting-started/) :rocket: ! ! ! !

> Αnything documented here is tested for [release 6.2.3](https://github.com/cotes2020/jekyll-theme-chirpy/releases/tag/v6.2.3)
{: .prompt-info }

### Typography / Styling Tips

#### Use GitHub emojis
By default [github emojis](https://gist.github.com/rxaviers/7360908) will not show up in your posts. We need a [plugin](https://jekyllrb.com/docs/plugins/) for that, named [jemoji](https://github.com/jekyll/jemoji). To use this plugin do the following:

``` bash
# Add the following to your site's Gemfile
gem 'jemoji'

# And add the following to your site's _config.yml
plugins:
  - jemoji

```

#### Prompts

> Example showing the prompt-tip
{: .prompt-tip }

``` markdown
> Example showing the prompt-tip
{: .prompt-tip }
```

> Example showing the prompt-info
{: .prompt-info }

``` markdown
> example showing the prompt-info
{: .prompt-info }
```

> Example showing the prompt-warning 
{: .prompt-warning  }

``` markdown
> Example showing the prompt-warning
{: .prompt-warning }
```

> Example showing the prompt-danger 
{: .prompt-danger }

``` markdown
> Example showing the prompt-danger
{: .prompt-danger }
```

### Useful commands
Below I am gathering some of the basics commands I use to test or build my site

``` bash
# To run locally the site you need the following command
bundle exec jekyll serve

# If you wish to include drafts, or posts with future date, you can use the following alternative
bundle exec jekyll serve --drafts --future

# If you haven't made any "cosmetics" changes, and made changes just in the posts, you can use --incremental flag
bundle exec jekyll serve --drafts --future --incremental 

# if you have cloned from Chirpy repo you can use the following to run locall
bash tools/run

# if you have cloned from Chirpy repo you can use the following to build locally
JEKYLL_ENV=production bundle exec jekyll b
```