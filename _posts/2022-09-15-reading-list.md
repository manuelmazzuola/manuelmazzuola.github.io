---
layout: post
show_badges: false
title: My personal del.icio.us
description: How to setup a simple but effective reading list
summary:  How to setup a simple but effective reading list
tags: [readinglist, nerd, delicious, go]
---

I have recently read an article on HackerNews [I tracked everything I read on the Internet for a year
](https://www.tdpain.net/blog/a-year-of-reading) where the author explains how he created a static site generator
to display the list of articles he read.

Often I find myself looking for an article read some time ago, struggling to seek it in the browser history.  
So I decided to [fork the project](https://github.com/manuelmazzuola/readingList), do some customization and publish it to have a list of articles I can search easily.

Firstly I modified the [GitHub workflows](https://github.com/manuelmazzuola/readingList/actions) to make `go` point to the correct repository and the author email of the user used to commit new articles.  

Then I customized the title of the static site, some `href`s and the CSS to make it look like my blog.  
I added [GA](https://github.com/manuelmazzuola/readingList/commit/b366cb28695f80390a44fab45ef64d4d2026ad77) and, finally, I created a simple script that I execute in the terminal when I have to add a new item to the list:
```bash
#!/bin/sh

URL="$1"
title="$2"
description="$3"
image="$4"

payload="{\"event_type\":\"rl-append\",\"client_payload\":{\"URL\":\"$URL\",\"Title\":\"$title\",\"Description\":\"$description\",\"Image\":\"$image\"}}"

curl \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GH_TOKEN" \
  https://api.github.com/repos/manuelmazzuola/readingList/dispatches \
  -d "$payload"

echo 'OK'
```

Go check it out: [https://manuelmazzuola.dev/readingList](https://manuelmazzuola.dev/readingList)

