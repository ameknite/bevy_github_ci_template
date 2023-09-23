# Bevy GitHub CI Template

Template for setting up continuous integration (CI) and continuous deployment (CD) on a GitHub project for Bevy. This template enables you to test your code, build for the Web, Linux, Windows, MacOS, and publish to GitHub Releases, Itch.io, and GitHub Pages.

It creates two workflows:

* [CI](#ci)
* [Release](#release)

## CI

Definition: [.github/workflows/ci.yaml](./.github/workflows/ci.yaml)

![ci workflow](https://user-images.githubusercontent.com/104745335/268799840-06b772e8-7901-4b86-9a88-e4afee8a0167.png)


This workflow runs on every commit to `main` branch, and on every PR targeting the `main` branch.

It will use Rust stable on Linux with caching between different executions. The following commands are executed:

* `cargo test`
* `cargo clippy -- -D warnings`
* `cargo fmt --all -- --check`

If you are using anything OS-specific or Rust nightly, you should update the [ci.yaml](./.github/workflows/ci.yaml) file  accordingly.

## Release

Definition: [.github/workflows/release.yaml](./.github/workflows/release.yaml)

### Run Workflow

This workflow runs every time you push a tag to your repo.

![Release workflow](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/270129182-904f75be-2e94-433c-956b-2b1758e9476a.png)

Example using git:

```sh
git tag -a "v1.0.0" -m "First official release"
git push --tags
```

You can use the workflow as it is, but check that the name of your repository matches the name of your binary. If they don't match, you can change the environment variable in [release.yaml](.github/workflows/release.yaml#L7) file. By default it will build for the Web, Linux, Windows, and MacOS, and then it will publish the archives on GitHub Release.

You can configure the builds and publish targets by changing the env variables in the [release.yaml](.github/workflows/release.yaml#L4) file.

You can configure and trigger the workflow directly in your GitHub repository. Navigate to the `Actions` section, click `Release` on the sidebar, then press the `Run workflow` button and select the configuration for the build and publish targets you want.

![Run workflow](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/268779376-85f4a503-1564-4075-b1ee-2a830fda2b7c.png)

The configuration in GitHub takes priority over the env variables in the [release.yaml](.github/workflows/release.yaml#L4) file, so you don't have to modify your env variables. The manual workflow also enables you to override the tag version eliminating the need to create another tag to trigger the workflow."

For the itch.io target, you can leave it empty if you already have the environment variable set up.

### Build

You can build for the following platforms:

* For Linux and Windows, create a .zip archive containing the executable and the `assets` folder.
* For MacOS, generate two versions: one for Intel x86_64 and one for Apple Silicon ARM64, each packaged in a .dmg image containing a .app file with the assets.
* For the Web, produce a .zip archive with the WebAssembly (wasm) binary, JavaScript bindings, an HTML file for loading it, and the assets.

By default, it builds for all platforms, but you can disable specific targets by setting their environment variable  `false` in the ([release.yaml](.github/workflows/release.yaml#L9)) file.

For example, `build_windows: false` will skip the Windows build.

The build files will be uploaded to the artifacts section of their respective workflow runs. To access them, go to your GitHub repository, navigate to the `Actions` section, click on `Release` in the sidebar, select the `workflow run` triggered by your tag, and scroll down to find your artifacts.

![Artifacts](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/268779709-2e1b3f0b-446b-40f1-8430-39583cb37cdd.png)

### Publish

You can publish to the following platforms:

- [Bevy GitHub CI Template](#bevy-github-ci-template)
  - [CI](#ci)
  - [Release](#release)
    - [Run Workflow](#run-workflow)
    - [Build](#build)
    - [Publish](#publish)
    - [Publish on Github Release](#publish-on-github-release)
    - [Publish on Itch.io](#publish-on-itchio)
    - [Publish on Github pages](#publish-on-github-pages)
  - [Adapting the Workflows for Your Project](#adapting-the-workflows-for-your-project)
  - [License](#license)
  - [Contribution](#contribution)

By default, it only publishes to GitHub Releases, but you can enable or disable any target by setting their environment variable to `true` or `false` in the ([release.yaml](.github/workflows/release.yaml#L19)) file.

For example, `publish_github_pages: true` will publish the web version of your game to GitHub Pages.

Every time you trigger this workflow, it will upload the files for every platform you enable in the build process.

The naming convention for the uploaded files will follow this structure:

`<Name of your binary>_<tag>_<platform>.<format>`

The format will be .zip for web, Windows, and Linux, and .dmg for MacOS.

For example: `bevy-game_v3.6_linux.zip`

### Publish on Github Release

This action will occur automatically every time you push a tag if you have enabled the environment variable `publish_github_releases: true` in the [release.yaml](./.github/workflows/release.yaml#L28) file.

However, if you prefer more configuration options to manage your releases, you can do so through the GitHub CLI or the web browser:

1. [Github CLI](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository?tool=cli): You can use `gh release create` to create a GitHub release iteratively from your terminal.
2. [Web Browser](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository?tool=webui)

Once you complete the process, a new release will be available on GitHub, with the archives for each platform accessible as downloadable assets.

![github release](https://user-images.githubusercontent.com/104745335/268805270-ff824032-4191-4528-9d45-a5511fef4f94.png)


### Publish on Itch.io

To publish to itch.io, follow this release flow:

1. Create an API key at <https://itch.io/user/settings/api-keys>
2. In your GitHub repository, go to the "Settings" tab, click on "Secrets" under the "Actions" section in the sidebar, and add a repository secret named `BUTLER_CREDENTIALS`, with the API key as its value.
3. Create your game page on itch.io if you haven't already.
4. In the [release.yaml](./.github/workflows/release.yaml#L22) file, set the environment variable `publish_itchio: true`.
5. Uncomment the `env.itchio_target` in [release.yaml](./.github/workflows/release.yaml#L25) and set it to your itch.io username and the name of the game on itch.io, separated by a slash (`/`). For example: `cart/build-a-better-buddy`. Double-check the URL of your game to ensure the name is correct.

Once these steps are completed, any tag pushed to GitHub will trigger an itch.io release, and it will use the tag as the [user version](https://itch.io/docs/butler/pushing.html#specifying-your-own-version-number).

If you want to make your game playable directly on your Itch.io page, follow the steps above, and make sure to build it for the web by setting the environment variable `env.build_web: true`, or by checking the box labeled `Build for Web` when manually triggering the workflow in GitHub.
Next, to make the game visible on your Itch.io page, go to your game's configuration on Itch.io, and change the `Kind of project` to `HTML`. Additionally, locate your uploaded web files and check the box that says, `This file will be played in the browser`."


![Kind of project](https://user-images.githubusercontent.com/104745335/268805329-fb70e23e-44ee-4f2f-9d20-11d58ddeec9a.png)


![Play in browser](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/268780679-fa14874c-040b-41ff-8a04-71cf141970dc.png)

### Publish on Github pages

If you want to publish your game to be playable in the browser on a [GitHub page](https://pages.github.com/) follow these steps:

1. In the [release.yaml](./.github/workflows/release.yaml#L28) file, verify that both environment variables `publish_github_pages` and `build_web` are set to `true`.
2. Trigger the [release.yaml](./.github/workflows/release.yaml) workflow by pushing a tag.
3. In your GitHub repository, go to the `Settings` tab, then click on `Pages` in the sidebar. Navigate to the `Build and Deployment` section, select the `gh-pages` branch and set the `root` folder. Finally, click on `Save`. ![Github Pages](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/268780368-af547adf-d8e8-4bdf-90e5-b7ee717493dc.png)
4. Wait a few minutes and your page will be available at a URL following this structure: `https://<Your GitHub username>.github.io/<Name of your repository>/`

## Adapting the Workflows for Your Project

If you'd like to use the GitHub workflows provided here for your own project, you may need to make a few adjustments:

1. For web builds, ensure that your project includes an `index.html` file under the `/wasm` directory.
2. Make sure that the name of your repository matches the name of your binary. If the name of your repository is different, simply change the environment variable `binary` in the ([release.yaml](.github/workflows/release.yaml#L7)) file to match the name of your binary.
3. If your project doesn't have an `assets` folder:
    1. You can create one and add a `.gitkeep` file to it to enable you to push it to your repository.
    2. Alternatively, if your project does not use assets, you can remove the `cp -r assets` statements from the build jobs in the workflow.
4. If you are using a nightly toolchain or a specific Rust version, make sure to adapt the toolchain version as needed.
5. If you encounter the error `Error: Resource not accessible by integration,`. Go to your GitHub repository's settings, and under `Actions -> General,` ensure that `Read and Write permissions` are selected under `Workflow permissions` near the bottom.

## License

Licensed under either of

* Apache License, Version 2.0
   ([LICENSE-APACHE-2.0](LICENSE-Apache-2.0) or <http://www.apache.org/licenses/LICENSE-2.0>)
* MIT License
   ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)
* CC0-1.0 License
   ([LICENSE-CC0-1.0](LICENSE-CC0-1.0) or <https://creativecommons.org/publicdomain/zero/1.0/legalcode>)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
triple licensed as above, without any additional terms or conditions.
