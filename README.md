# Automatic RoamResearch backup to a digital garden

First, create a note in Roam called Public and reference all the notes that you want to publish (use the `[[Notation]]`, `#This` won't work)

Then, clone this repo and mark it as private: <https://github.com/maximevaillancourt/digital-garden-jekyll-template>

Edit the Jekyll config in the repo (`_config.yml`) to contain:
```exclude: ['_includes/notes_graph.json', 'markdown', 'formatted', 'json', 'LICENSE', 'default.nix']```

Then configure GitHub secrets as per the instructions here: <https://github.com/MatthieuBizien/roam-to-git#configure-github-secrets>

And finally, create `.github/workflows/main.yml` with the following content:

```yaml
name: "Roam Research backup"
on:
  push:
    branches:
      - master
  schedule:
    -   cron: "0 * * * *"
jobs:
  backup:
    runs-on: ubuntu-latest
    name: Backup
    timeout-minutes: 15
    steps:
      -   uses: actions/checkout@v2
      -   name: Set up Python 3.8
          uses: actions/setup-python@v1
          with:
            python-version: 3.8
      -   name: Setup dependencies
          run: |
            pip install git+https://github.com/DoomHammer/roam-to-git.git@roam-to-garden
      -   name: Run backup
          run: roam-to-git --skip-git .
          env:
            ROAMRESEARCH_USER: ${{ secrets.ROAMRESEARCH_USER }}
            ROAMRESEARCH_PASSWORD: ${{ secrets.ROAMRESEARCH_PASSWORD }}
            ROAMRESEARCH_DATABASE: ${{ secrets.ROAMRESEARCH_DATABASE }}
      -   name: Commit changes
          uses: elstudio/actions-js-build/commit@v3
          with:
            commitMessage: Automated snapshot
```

To publish your digital garden, go to [Netlify](https://netlify.com/) and create a new site from the repository you've just created.

This will run every hour, fetch the notes referenced by your Public note, and publish them.
