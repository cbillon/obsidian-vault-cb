---
link: https://github.com/moodlehq/moodle-plugin-release/pull/15
---
From now on, you can continue using [https://github.com/moodlehq/moodle-plugin-release](https://github.com/moodlehq/moodle-plugin-release) for uploading new plugin versions. Just replace CI script at local repo, it has been updated to use Marketplace API.

This new ci github workflow definition looks rather complicated to what has been used with the old plugins directory. So probably you do not really want to _copy_ this to all of your repositories, because you will have to change every single one as soon as something little changes. Wouldn't it be a good idea if the moodle-plugin-releases provided this not as an example but a reusable github workflow I can integrate into my plugins with "uses"?

I mean now that everyone has to touch his release yml anyway that would be the time to make plugin devs' life easier, right? Before everyone is copying/pasting that yml

hat would be nice to make it work via `uses` one day, there ticket from Plugin Dir times [https://github.com/moodlehq/moodle-plugin-release/issues/12](https://github.com/moodlehq/moodle-plugin-release/issues/12)

The complication of current one comes from making it work for private repos, where passing zip URL for callback is not possible (although new API support it too), it also determines frankenstyle itself. For most cases workflow will work without modifications (or minimum changes, like when tag is not semver). Full [API docs](https://moodledev.io/general/community/plugincontribution/moodlemarketplaceapi) have more info in case one needs more customisation.

[@PhMemmel](https://github.com/PhMemmel) suggested to create reusable workflow, which indeed would simplify setup. It allows including release call as part of different workflow file (e.g. generate notes and release). This also allows us to manage changes without people need to update whole script if we change/fix something, although we need to make sure any change we merge would not break existing configurations.

Use example (testing PR): [https://github.com/kabalin/moodle-media_jwplayer/blob/master/.github/workflows/release.yaml](https://github.com/kabalin/moodle-media_jwplayer/blob/master/.github/workflows/release.yaml)  
Test run: [https://github.com/kabalin/moodle-media_jwplayer/actions/runs/32901652672/job/97976650829#logs](https://github.com/kabalin/moodle-media_jwplayer/actions/runs/32901652672/job/97976650829#logs)

Fixes [#12](https://github.com/moodlehq/moodle-plugin-release/issues/12)

https://github.com/moodlehq/moodle-plugin-release

# Releasing Moodle plugin versions in Moodle Marketplace from GitHub Actions
[link](https://github.com/moodlehq/moodle-plugin-release)
[](https://github.com/moodlehq/moodle-plugin-release#releasing-moodle-plugin-versions-in-moodle-marketplace-from-github-actions)

This is a [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows). Instead of copying the whole file, add a small caller workflow to your plugin repository.

1. Create `.github/workflows/moodle-release.yml` in your plugin repository with the following content:
    
    ```yaml
    name: Release Plugin version to Moodle Marketplace
    
    on:
      push:
        tags:
          - 'v*'
    
    jobs:
      release-to-marketplace:
        uses: moodlehq/moodle-plugin-release/.github/workflows/moodle-release.yml@main
        with:
          tag: ${{ github.event.inputs.tag }}
        secrets:
          MOODLE_MARKETPLACE_TOKEN: ${{ secrets.MOODLE_MARKETPLACE_TOKEN }}
    ```
    
2. Log in to the Moodle Marketplace. Navigate to "Account Settings" > "Security" ([https://marketplace.moodle.com/account/security](https://marketplace.moodle.com/account/security)) and create a new API token. Copy the token immediately, as it is only displayed once.
    
3. Go to your plugin repository on GitHub. Navigate to "Settings" > "Secrets and Variables" > "Actions". Click "New repository secret", name it `MOODLE_MARKETPLACE_TOKEN`, and paste your API access token as the value.
    
4. That's it! Now when you tag the repository with a tag that matches the configured condition (starts with `v`, e.g. `v1.4.0`), the tagged version will be released in Moodle Marketplace.
    

## Tips

[](https://github.com/moodlehq/moodle-plugin-release#tips)

- Provide release notes when creating a GitHub Release. The workflow will automatically use your GitHub Release description.
    
- If your release tags do not start with `v` character (such as `v9.0.1`) and you want to trigger the workflow for any tag, change the condition in your caller workflow as:
    
    ```
    on:
      push:
        tags:
          - '*'
    ```
    
- Marketplace API documentation is located at [moodledev.io](https://moodledev.io/general/community/plugincontribution/moodlemarketplaceapi).