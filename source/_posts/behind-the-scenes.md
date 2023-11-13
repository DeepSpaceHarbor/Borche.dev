title: The tech behind Borche.dev
featured_image: files/images/featured/setup-behind-site.jpg
summary: "A different type of \U0001F44B\U0001F30D post."
date: '2022-04-15T02:18:53.000Z'
tags: []
categories: []
---
The blog runs on [Hexo](https://hexo.io/), a content management system for static blogs.
I'm using [Bridge](https://github.com/DeepSpaceHarbor/hexo-bridge) for the admin panel and [BrowserSync](https://github.com/hexojs/hexo-browsersync) for an instant preview of every change I've made.
There is a custom theme built with the [Neumorphism UI](https://themesberg.com/product/ui-kit/neumorphism-ui-kit-bootstrap) kit. For now, its purpose is to highlight the content without the extra features that come with blogs. As the content grows, I will add search, post suggestions, and so on.
[Prism](https://prismjs.com/) handles the code highlighting with the Synthwave '84 theme.
The site lives in a private GitHub repository. Every time there's a new commit, it will build and deploy the new version through [custom GitHub action](https://github.com/DeepSpaceHarbor/hexo-deploy-action).



