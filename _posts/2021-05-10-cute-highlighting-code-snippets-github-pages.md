---
layout: post
title:  "Cute highlighting for code-snippets in GitHub pages"
date:   2021-05-10 00:00:00 +0200
comments: true
tipue_search_active: false
excerpt_separator: <!--end_excerpt-->
tags: Blog Dev GitHub DataNinja
---

A long time ago I switched my personal blog to [GitHub Pages](https://enriquecatala.com/2020/05/28/Creating-my-new-blog-with-Jekyll.html). In this post I want to share with you how easy is to change the look and feel of your code snippets.

> NOTE: Please go and read how to [configure your blog with GitHub Pages](https://enriquecatala.com/2020/05/28/Creating-my-new-blog-with-Jekyll.html)

<!--end_excerpt-->

## 1) Integrate kramdown and rouge

 The first thing to do is to configure kramdown and rouge via the Gemfile. To do so, you only need to add the following lines to your _Gemfile_

 ```gem
 gem "kramdown", ">= 2.3.1"
 ```
>NOTE: you can check mine [here](https://github.com/enriquecatala/enriquecatala.github.io/blob/master/Gemfile)

## 2) Configure

 Next thing to do is to configure your theme to use kramdown.

 Go and edit your [_config.yml](https://github.com/enriquecatala/enriquecatala.github.io/blob/master/_config.yml) file and add the following

 ```yaml
 # BUILD SETTINGS
markdown: kramdown
kramdown:
  input: GFM
  syntax_highlighter: rouge
 ```

## 3) Check your own style

There are a few syntax hightlighters that you can use. Now it´s time to check them and select the one that you like the most. 

Go to this nice [demo page](https://spsarolkar.github.io/rouge-theme-preview/) and take a look :)

## 4) Generate your template

After knowing the one that you want to use, now it´s time to generate the .css template. Suppose that you liked the _"base16.monokai.dark"_

```bash
# Install rougify
sudo apt install ruby-rouge
# Go to the folder where you have your css templates
cd /your/css/template/for/jekyll/css/
# Generate the template
rougify style base16.monokai.dark > base16.monokai.dark.scss
```

## 5) Setup highlighter

To activate the css, you need to open your _main.scss_ and include the relative path to load the file:

```css
//kramdown
@import 'kramdown/base16monokaidark';
```
>NOTE: You can see how I did in my own blog [here](https://github.com/enriquecatala/enriquecatala.github.io/blob/master/css/main.scss)

## 6) Use it!

To use the code in your post, you only need to include in markdown the "code" section, inlcuding the language you are using. Nothing else :)

You can check a post example [here](https://enriquecatala.com/2021/03/01/remote-jupyter-notebook-file-root-vscode.html), with the real [markdown](https://github.com/enriquecatala/enriquecatala.github.io/blob/master/_posts/2021-03-02-remote%20jupyter%20notebook%20file%20root%20vscode.md)