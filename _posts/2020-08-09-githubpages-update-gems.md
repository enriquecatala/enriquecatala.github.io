---
layout: post
title:  "Howto update gems on Github pages"
date:   2020-08-09 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Docker Blog
---
It seems that kramdown has a big security issue and i want this to be fixed.

![kramdown security issue](/img/posts/gem-update-github-pages/kramdown-security-1.png)

To solve the problem, we can click to the "See dependabot alert"

<!--end_excerpt-->

And this is what we found: 

![kramdown security issue2](/img/posts/gem-update-github-pages/kramdown-security-2.png)

# Fix

The fix seems easy to solve...we only need to fix this on the gemfiles...but as i´m using Windows10 to work with jekyll, and i don´t have any jekyll instalation on my machine...i only need to make a little change to the docker-compose.yml file

>NOTE: [Check this post](https://enriquecatala.com/2020/05/28/Creating-my-new-blog-with-Jekyll.html) to understand why i need to do this

1) Edit [docker-compose](/docker-compose.yml)
   
2) <mark>Change the command to</mark>
 ```bash
   command: bundle update
 ```

3) Edit [Gemfile](/Gemfile) file and add the following to the end of the file
   ```bash
   # vulnerability found 
   gem "kramdown", ">= 2.3.0"
   ```

4) Now, cleanup the container
   ```bash
   docker-compose down   
   ```

5) Fix the dependencies
   ```bash   
   docker-compose up   
   ```

>NOTE: you should see the following text after the container ends successfully

```bash
jekyll_1  | Bundle updated!
```

And your [Gemfile.lock](/Gemfile.lock) file should be updated accordingly, with kramdown among other gems

![gemfile.lock update](/img/posts/gem-update-github-pages/gemfilelock.png)

Now, you can edit again your [docker-compose](/docker-compose.yml) file to set the value to the previous one
```bash
command: jekyll serve --watch --force_polling --verbose --safe
```

And thats it!

>NOTE: Please remind to execute a _docker-compose down_ prior working again :)