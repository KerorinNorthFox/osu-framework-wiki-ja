All PRs are automatically built by appveyor into debug nuget packages. You can test these against a project dependent on the framework by adding the appveyor nuget source to your local sources.

The appveyor source is `https://ci.appveyor.com/nuget/osu-framework`

## Rider

![](https://puu.sh/ADKjx/7f501eab2f.png)

## CLI

`nuget sources Add -Name "appveyor osu-framework" https://ci.appveyor.com/nuget/osu-framework`

Once this is added, you will find recent appveyor builds in your versions listing as `0.0.<appveyor_build_id>`. You can find the appveyor build ID by clicking through to the appveyor results page of a PR and checking the ID in the top-right corner (or URL).

![](https://puu.sh/ADKqC/2f93d99271.png)

