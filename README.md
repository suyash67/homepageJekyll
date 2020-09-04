# Homepage

Welcome to the personal webpage repository of <a href="https://suyash67.github.io/{{ site.baseurl }}">Suyash Bagad</a>
built using the Jekyll's [minima](https://github.com/jekyll/minima) theme. Jekyll offers some of the most *elegant* and *easy-to-use* themes. Check them out [here](https://jekyllrb.com/resources/).


Theme preview is available [here](https://jekyll.github.io/minima/).

![minima theme preview](/screenshot.png)

## Customizing a Jekyll Theme
 
*To be added soon*

## Hosting your Jekyll Website on GitHub

*To be added soon*


*Note: The website content in my repository is static content generated using Jekyll. Feel free to start by cloning my website from <a href="https://github.com/suyash67/homepage">github</a>. Feel free to customize this theme to your site without linking back to me or including a disclaimer.* 

## Using my Template Website

- Step 1: Clone this repository.

- Step 2: Edit the page settings in the `_config.yml` file. Edit the markdown files for customizing pages. You can add blogs/posts by creating a new file in the `_posts` folder by naming it of the form `yyyy-mm-dd-url-of-the-blog.md` and use the template of any existing file in `_posts`. To add images to your blog, add images into the `assets` folder according to the directory structure in `assets`.
- -

- Step 3: Run `bundle exec jekyll serve --incremental`. The `--incremental` flag must added for incremental build which results in a faster build. However, when configuration files like [_config.yml](https://github.com/suyash67/homepageJekyll/blob/master/_config.yml) or [Gemfile](https://github.com/suyash67/homepageJekyll/blob/master/Gemfile) are changed, you should build without the `--incremental` flag.

- Step 4: Open <a href="http://127.0.0.1:4000/homepage/">http://127.0.0.1:4000/homepage/</a> and check your changes. Note here my `baseurl` parameter in [_config.yml](https://github.com/suyash67/homepageJekyll/blob/master/_config.yml) is set to `/homepage`. If your `baseurl` is empty, the local site will be hosted at [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

- Step 5: If the desired changes are visible on the local site, proceed by running `cd _site` and then push the changes in `_site` folder to `gh-pages`. Note that remote repository for contents in `_site` must be hosted on [github pages](https://pages.github.com/).

Your changes in the github page would be visible at `your_username.github.io/baseurl` after a minute.
