# octostache-action

This action will scan the file(s) provided in the `files-with-substitutions` argument for Octopus variable substitution syntax `#{VariableName}`.  If the files contain any `#{Variables}` that match an item in the `variables-file` or environment variables, it will replace the template with the actual value. If a variable is found in both the `variables-file` and in the environment variables, then the environment variable value will be used.

This is a container action so it will not work on Windows runners.

## Inputs

| Parameter                  | Is Required | Description                                                                                                                                                                                                                                              |
| -------------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `variables-file`           | false       | An optional yaml file containing variables to use in the substitution.                                                                                                                                                                                   |
| `files-with-substitutions` | true        | A comma separated list of files with `#{variables}` that need substitution.                                                                                                                                                                              |
| `output-files`             | false       | An optional comma separated list of files to output.<br/>If defined, the program assumes the index of the output file is the same as the index of the template file in the template-files list. They therefore need to have the same number of elements. |

## Outputs

No Outputs

## Usage Example

### Variables File
When using the `variables-file` argument, this is the structure of the file containing the variable names and values that will be substituted.
```yaml
Environment: Dev
Version: 1.3.62
LaunchDarklyKey: abc
GoogleAnalyticsKey: 123
AppInsightsKey: a1b2c3
```

### Environment Variables
In addition to the `variables-file` argument, substitutions can be provided in the `env:` section of the action.  The format matches what is supplied in the `variables-file`: `<var-name>: <var-value>`.  

If the same item is provided in the `variables-file` and the `env:` section, the value in the `env:` section will be used.

### Example Files that contain Octostache Substitution Syntax
These are some sample files that contain the Octostache substitution syntax `#{}`.  These files would be included in the `files-with-substitutions` argument above.

`DemoApp19.csproj`
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
    <Version>#{VersionToReplace}</Version>
  </PropertyGroup>
</Project>
```

`build-variables.js`
```js
const { profiles } = require('../../Properties/launchSettings.json');
const { environmentTypes } = require('./constants');

const substitutionVariables = {
  BUILD_APPINSIGHTS_INSTRUMENTATION_KEY: '#{AppInsightsKey}',
  BUILD_GA_KEY: '#{GoogleAnalyticsKey}',
  BUILD_LAUNCH_DARKLY_KEY: '#{LaunchDarklyKey}}'
};
```

### Workflow

```yml
name: Deploy

on:
  workflow_dispatch:

jobs:
  substitute-variables:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2

      - uses: im-open/octostache-action@v2.0.1
        with:
          variables-file: ./substitution-variables.json
          files-with-substitutions: ./src/DemoApp19/DemoApp19.csproj,./src/DemoApp19/Bff/FrontEnd/scripts/build-variables.js
          output-files: ./src/DemoApp19/DemoApp19.csproj,./src/DemoApp19/Bff/FrontEnd/scripts/build-variables.js
        env:
          # Note that this value would be used over the value from the example variables file
          LaunchDarklyKey: ${{ secrets.LAUNCH_DARKLY_API_KEY }}
```


## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).

[Authenticating to GitHub Packages - nuget]: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry#authenticating-to-github-packages
[dotnet nuget add source]: https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-nuget-add-source
[Authenticating to GitHub Packages - npm]: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry#authenticating-to-github-packages
[npm private packages in ci/cd workflow]: https://docs.npmjs.com/using-private-packages-in-a-ci-cd-workflow