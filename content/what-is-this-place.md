+++
title = "What is this place"
date = 2024-11-16
+++

## What is this place

Well, this shall be my blogging environment for the future. Currently, I'm working on multiple interesting projects that deserve some write-ups. I do think making such documents might help me deepen my understanding of said subjects.

## How was this place created

I never had any experience with blogging before. Initially, I wanted to host my own website with some custom dynamic features (such as a built-in editor for writing these blog Markdown files), but I soon realized that putting in so much work for a technology that already exists is not worth it. That's when I got Hugo recommended by `pskrgag` (who has a great personal website built with it [here](https://pskrgag.github.io/)). So I started reading about it and quickly realized how awesome it is. A static site generator not only ensures quick load times but also makes it possible to host my blogs on GitHub paired with GitHub Actions! I love automation so I decided to ditch the idea about making my own blogging platform. This is when I started looking into [Hugo](https://github.com/gohugoio/hugo) and that's how I came across [Zola](https://github.com/getzola/zola). A bit of a history lesson: Jelkyll was the original tool used to do static site generation, however it became increasingly slow. That's how Hugo came along with its blazing fast `Go` generator. However `Hugo`'s template files were notoriously hard to understand. That's how `Zola` came along. Written in `Rust`, it often provides faster generation times (although the difference between `Hugo`'s and `Zola`'s performance is marginal) and offers a more transparent template system. Given the growing popularity of `Zola` and my Rust fanaticism, I ended up creating this site with `Zola`. As you can see, it works perfectly well and offers a beautiful (well, at least in my eyes) UI.
