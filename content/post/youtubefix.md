---
title: "Youtube Hugo-Shortcode workaround - when {{< youtube >}} won´t work"
date: 2018-12-09T17:23:27+01:00
draft: true
image: /img/youtubefix_cover.jpg
tags:
- Hugo
- Shortcode
---
Inspired by a recent post from my friend, <a href="https://gohugo.io/" target="_blank">(Go)Hugo</a> and Git mentor <a href="https://twitter.com/embano1" target="_blank">Michael Gasch</a> :wink:, where he described how he fixed the Hugo <a href="https://gohugo.io/content-management/shortcodes/" target="_blank">Shortcode</a> for Twitter to suppress the default thread view, it came immediately into my mind that I faced a similar issue when I started blogging with the Shortcode which embedded a Youtube-Video into your post.

See how Michael fixed the normal behavior for the Twitter Shortcode `{{</* tweet */>}}` here...

{{< tweet 1071143487528206336 >}}

...and add it to your layouts|_Shortcode folder to make advantages from.

### *When `{{</* youtube */>}}` won´t work for your theme*

Besides embedding a tweet into your post for e.g. to give a statement more expression/ more emphasis, or to comment a topic-related tweet, or simply because it´s pretty cool, a Youtube-Video is also a good alternative.

I´m running the <a href="https://themes.gohugo.io/hugo-casper-two/" target="_blank">*Casper-Two*</a> Theme for my Blog and back then when I´ve written <a href="https://rguske.github.io/post/sometimes-you-gotta-run-before-you-can-walk/" target="_blank">"*Sometimes you gotta run before you can walk*"</a>, I absolutely wanted to have the <a href="https://www.youtube.com/watch?time_continue=2&v=1VrHeInDwwg" target="_blank">*IRON MAN I First Armor test*</a> Video included and it didn´t work straight from the start.

###### *Let me demo you what I mean* 

I´ll add the video <a href="https://www.youtube.com/watch?v=2xkNJL4gJ9E&list=PLLAZ4kZ9dFpOnyRlyS-liKL5ReHDcj4G3&index=9" target="_blank">"Shortcodes | Hugo - Static Site Generator | Tutorial 9"</a> here at this point of the post by using `{{</* youtube 2xkNJL4gJ9E */>}}` in my post `*.md` file. :point_down:

---
{{< youtube 2xkNJL4gJ9E >}}

---

Normally you would see here :point_up: the mentioned Youtube video but it seems that the Shortcode won´t work for my chosen theme. 

###### You are not alone!

The best thing what you can do first, from my point of view, is not to google the (mis-)behavior, rather to go to the Github-Repo from your theme and have a look at the *Issues*. If there still exists one, comment it...if not...raise one. 

So I had a look at the issues for <a href="https://github.com/eueung/hugo-casper-two" target="_blank">Casper-Two</a> and see there, there´s already <a href="https://github.com/eueung/hugo-casper-two/issues/5" target="_blank">one</a> open which covers this issue. I commented this as well and a really short period of time later, <a href="https://twitter.com/SalarRahmanian" target="_blank">Salar Rahmanian</a> had the code snippet for it. 

This is simply great and one thing of thousands I´m really impressed of regarding Github and the Community.

###### The Code

Like Michael already described it in his post, we have to create a new empty file and this time we´ll call it `yt.html`.

```bash
$ cd <HUGO_BLOG_ROOT>
$ touch layouts/shortcodes/yt.html
```

Put the following lines of code into the `yt.html` file and save it.

<iframe
  src="https://carbon.now.sh/embed/?bg=rgba(19%2C20%2C19%2C1)&t=cobalt&wt=sharp&l=htmlmixed&ds=false&dsyoff=41px&dsblur=39px&wc=true&wa=true&pv=5px&ph=5px&ln=false&fm=Hack&fs=14px&lh=133%25&si=false&code=%253Ciframe%2520src%253D%2522https%253A%252F%252Fwww.youtube.com%252Fembed%252F%257B%257B%2520index%2520.Params%25200%2520%257D%257D%253Fstart%253D%257B%257B%2520index%2520.Params%25201%2520%257D%257D%2522%250Astyle%253D%2522position%253A%2520absolute%253B%2520top%253A%25200%253B%2520left%253A%25200%253B%2520width%253A%2520560%253B%2520height%253A%2520315%253B%2522%2520allowfullscreen%2520frameborder%253D%25220%2522%2520title%253D%2522YouTube%2520Video%2522%253E%253C%252Fiframe%253E&es=4x&wm=false"
  style="transform:scale(1.0); width:850px; height:300px; border:1; overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

This time I´ll make use of our new Shortcode `{{</* yt */>}}` instead of `{{</* youtube */>}}` *et voilà*...

{{< yt 2xkNJL4gJ9E >}}

...here we go :thumbsup:. Thanks again Salar!

{{< tweet 1024709843712724993 >}}

---

*Any comment? Drop me a PM on Twitter.*