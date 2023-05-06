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

