# Using gh copilot in a workflow.

[`gh copilot`](https://github.com/github/gh-copilot) is a cli tool to interact with Copilot.

It's currently not really tooled to work in a workflow and is aimed at being a locally used cli tool auth'd via a your `gh cli` oauth token. It only works with oauth.

As this repo will show, we can use it in a workflow but it requires some things.

- Problem 1 - analytics

  There is no non interactive way to manage the accepting of the analytics data option. A hard requirement to using the app. `gh copilot config` can manage it but it's also interactive. So to solve this we create the config file and setting manually.

  ```bash
  mkdir -p ${HOME}/.config/{gh,gh-copilot}
  printf '%b\n' 'optional_analytics: false\nsuggest_execute_confirm_default: false' > ${HOME}/.config/gh-copilot/config.yml
  ```

- Problem 2 - oath

  To accept the oauth in the workflow and get the token requires you to open open a browser url and enter a code. Not what we want.

  On any device you are logged on with `gh cli` you need to do `gh auth token --hostname github.com` to see your `oauth` token.

  Set this as a repository secret to `secrets.COPILOT_CLI` and then in the workflow step we do this

  ```bash
  unset GITHUB_TOKEN GH_TOKEN
  printf '%s' "${{ secrets.COPILOT_CLI }}" | gh auth login --with-token
  mkdir -p ${HOME}/.config/{gh,gh-copilot}
  printf '%b\n' 'optional_analytics: false\nsuggest_execute_confirm_default: false' > ${HOME}/.config/gh-copilot/config.yml
  printf '%s\n' "GITHUB_TOKEN=$(gh auth token --hostname github.com)" >> $GITHUB_ENV
  printf '%s\n' "GH_TOKEN=$(gh auth token --hostname github.com)" >> $GITHUB_ENV
  gh extension install github/gh-copilot --force
  ```

- Problem 3 - `GITHUB_TOKEN` / `GH_TOKEN`

  When these are used in a workflow `gh cli` will default to using these for auth. `gh copilot` only works with `oauth` so it will ignore this and prompt for `oauth` flow. So you have a working `gh cli` but an unusable `go copilot`.

  To fix this we must set them to use the `secrets.COPILOT_CLI` so that the default tokens are using `oauth` - this means `gh cli` and `gh copilot` will both work.

  Then when finished with copilot we can restore their values.

Here are the workflows demonstrating this in action:

https://github.com/userdocs/gh-copilot-cli/blob/main/.github/workflows/gh-copilot-cli.yml

https://github.com/userdocs/gh-copilot-cli/blob/main/.github/workflows/test-1.yml

https://github.com/userdocs/gh-copilot-cli/blob/main/.github/workflows/test-2.yml
