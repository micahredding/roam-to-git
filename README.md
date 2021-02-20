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
    -   cron: "0 0,4,8,12,16,20 * * *"

jobs:
  backup:
    runs-on: ubuntu-latest
    name: Backup
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Set up Python 3.8
        uses: actions/setup-python@v1
        with:
          python-version: 3.8

      - name: Run Backup
        run: |
          cd /tmp
          git clone -q https://github.com/everruler12/roam2github.git roam2github
          cd $_
          npm i
          npm run start -s
        env:
          R2G_EMAIL: ${{ secrets.ROAMRESEARCH_USER }}
          R2G_PASSWORD: ${{ secrets.ROAMRESEARCH_PASSWORD }}
          R2G_GRAPH: ${{ secrets.ROAMRESEARCH_DATABASE }}

      - name: Setup dependencies
        run: |
          pip install git+https://github.com/DoomHammer/roam-to-git.git@roam-to-garden
      - name: Generate notes
        run: roam-to-git --skip-fetch --skip-git .
        env:
          ROAMRESEARCH_USER: ${{ secrets.ROAMRESEARCH_USER }}
          ROAMRESEARCH_PASSWORD: ${{ secrets.ROAMRESEARCH_PASSWORD }}
          ROAMRESEARCH_DATABASE: ${{ secrets.ROAMRESEARCH_DATABASE }}

      - name: Commit Changes
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Automated snapshot
```

*Important*: if you've been using this tool before you may need to delete the
`_notes`, `formatted`, `markdown`, and `json` directories created by the
previous version of roam-to-garden.

To publish your digital garden, go to [Netlify](https://netlify.com/) and create a new site from the repository you've just created.

This will run every four hours, fetch the notes referenced by your Public note, and publish them.
