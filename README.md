# Basic commands

1. Select a ruby version to work on by running 

```bash
chruby 3.2.2  
```

2. Build the site and make it available on a local server.
```bash
bundle exec jekyll serve --drafts --livereload
```
+ Pass the `--livereload` option to serve to automatically refresh the page with each change you make to the source files.

+ Drafts are posts without a date in the filename. To preview the site with drafts pass `--drafts` option.

Note: If you want to redirect *haccks.com* to *www.haccks.com* then set **Custom Domain** to *www.haccks.com* in the **[GitHub Pages](https://github.com/haccks/blogs/settings/pages)** section. If you want to redirect *www.haccks.com* to *haccks.com* then set it to *haccks.com*. Read m=in details: [Configuring an apex domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain).

