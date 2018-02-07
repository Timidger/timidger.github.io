Based on the [chalk design](https://github.com/nielsenramon/chalk)

### Installation

If you haven't installed the following tools then go ahead and do so:

    dwnload ruby npm yarn

Next setup your environment:

    bin/setup

Note that you will probably run into permission issues with yarn. Just run the last command directly.

### Development

Run Jekyll:

    bundle exec jekyll serve

## Deploy to GitHub Pages

Before you deploy, commit your changes to any working branch except the `gh-pages` one and run the following command:

    bin/deploy

**Important note**: Chalk does not support the standard way of Jekyll hosting on GitHub Pages. You need to deploy your working branch (can be any branch, for xxx.github.io users: use another branch than `master`) with the `bin/deploy` script. Reason for this is because Chalk uses Jekyll plugins that aren't supported by GitHub pages. The `bin/deploy` script will automatically build the entire project, then push it to the `gh-pages` branch of your repo. The script creates that branch for you so no need to create it yourself.

You can find more info on how to use the `gh-pages` branch and a custom domain [here](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/).

[View this](https://github.com/nielsenramon/kickster#automated-deployment-with-circle-ci) for more info about automated deployment with Circle CI.

## License

MIT License
