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

## Running a local version

- Step 1: Run `bundle exec jekyll serve --incremental`. The `--incremental` flag must added for incremental build which results in a faster build. However, when configuration files like [_config.yml](https://github.com/suyash67/homepageJekyll/blob/master/_config.yml) or [Gemfile](https://github.com/suyash67/homepageJekyll/blob/master/Gemfile) are changed, you should build without the `--incremental` flag.

- Step 2: Open <a href="http://127.0.0.1:4000/homepage/">http://127.0.0.1:4000/homepage/</a> and check your changes. Note here my `baseurl` parameter in [_config.yml](https://github.com/suyash67/homepageJekyll/blob/master/_config.yml) is set to `/homepage`. If your `baseurl` is empty, the local site will be hosted at [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

- Step 3: If the desired changes are visible on the local site, proceed by running `cd _site` and then push the changes in `_site` folder to `gh-pages`.

Your changes in the github page would be visible after a minute.
