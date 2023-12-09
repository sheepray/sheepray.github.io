## local development
### init
1. [install Jekyll](https://jekyllrb.com/docs/installation/macos/)
2. `bundle install`
[ref](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)
### run
```bash
cd docs
bundle exec jekyll serve
```
Go to http://127.0.0.1:4000/

### Known issues
1. `LoadError: cannot load such file -- webrick`

    Solution: `bundle add webrick` as per https://github.com/jekyll/jekyll/issues/8523.

2. `CMD + SHIFT + V` to preview markdown file in vs code.
