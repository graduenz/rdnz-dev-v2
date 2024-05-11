+++
title = "Moving away from GitBook to Hugo"
slug = "moving-away-from-gitbook-to-hugo"
date = "2024-05-11T02:00:00-03:00"
description = "I recently moved my website away from GitBook and this is the first post in the new platform."

tags = ['decisions']
+++

I recently moved my website away from GitBook, and this is the first post on the new platform.

## GitBook

The platform is great, and I will be using it for documentation projects.

My initial goal was to focus on content writing instead of coding a website from scratch, but I wanted to have control over the data ([in a GitHub repository](https://github.com/graduenz/rdnz-dev)).

I wrote two articles there about stuff I was dealing with recently:
- [ASP.NET Core Integration Tests](/blog/asp-net-core-integration-tests)
- [Code analysis with SonarCloud and GitHub Actions](/blog/code-analysis-sonarcloud-github-actions)

## Problems

However, there was a simple problem on the platform that I never liked. When configuring a space, I have to choose a subdomain  for it (`gui.rdnz.dev`) without being able to use the root domain (`rdnz.dev`).

My approach to this was to:
- Create a worker on Cloudflare that, given any URL, fetches it from `gui.rdnz.dev` ([see below](#extra-cloudflare-worker)) by modifying the URL hostname;
- Assign the root domain to the worker &horbar; `rdnz.dev`.

It worked very well, but I didn't like it: it was a workaround, I just wanted it to literally bind the site to the root domain.

Also, there are some limits on the Cloudflare workers (100k requests/day) that contributed to the decision to get away from it.

## Choosing a new platform

When choosing a new platform for the website, some requirements had to be met:

- SSG, as fast as possible;
- Markdown-based, so I can reuse all GitBook content;
- Able to deploy on GitHub Pages;
- Able to use the root domain;
- Ready-to-use minimalist themes (thanks to [Hugo Bearcub](https://themes.gohugo.io/themes/hugo-bearcub/)).

I know there are many options, but after some research my choice was [Hugo](https://gohugo.io/).

I was able to build the whole website within a few hours, and its source code is also [available on GitHub](https://github.com/graduenz/rdnz-dev-v2).

## Extra: Cloudflare worker

In case you ever want to handle the problem with GitBook on Cloudflare like I did, here's the code for the worker:

```js
export default {
  async fetch(request, env, ctx) {

    // Fetch from the gitbook website
    const url = new URL(request.url);
    url.hostname = 'gui.rdnz.dev'
    const res = await fetch(url);

    const contentType = res.headers.get('Content-Type');
    const isHtml = contentType && contentType.startsWith('text/html');

    // If it's not HTML, just return the response
    if (!isHtml)
      return res;

    // Otherwise, goes ahead with fixing meta tags for LinkedIn
    var rewriter = new HTMLRewriter()
      .on('meta', {
        element(element) {
          const name = element.getAttribute('name');
          element.setAttribute('property', name);
        }
      });
    
    return rewriter.transform(res);
  },
};
```