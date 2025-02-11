# Publishing Modules

The `Publisher` component supports publishing modules to various package
managers. It is designed to be attached to a GitHub workflow that takes care of
building the module and uploading a "ready to publish" artifact.

> This component is utilized in `NodeProject` and `JsiiProject` to publish modules.

Supported package managers:

- npm
- Maven
- PyPI
- NuGet
- Go (GitHub)

This is how a publisher is initialized:

```ts
const publisher = new Publisher(project, {
  workflow: releaseWorkflow,
  buildJobId: 'my-build-job',
  artifactName: 'dist',
});
```

`workflow` is a `GithubWorkflow` with at least one job (in this case named `my-build-job`) which is responsile to build the code and upload a GitHub workflows artifact (named `dist` in this case) which will then be consumed by the publishing jobs.

This component is opinionated about the subdirectory structure of the artifact:

|Subdirectory|Type|Contents|
|-|-|-|
|`js/*.tgz`|NPM|npm tarballs
|`dotnet/*.nupkg`|NuGet|Nuget packages
|`python/*.whl`|PyPI|Python wheels
|`go/**/go.mod`|Go|Go modules. Each subdirectory should have its own `go.mod` file
|`java/**`|Maven|Maven artifacts in local repository structure

Then, you should call `publishToXxx` to add publishing jobs to the workflow:

For example:

```ts
publisher.publishToNuGet();
publisher.publishToMaven(/* options */);
// ...
```

See API reference for options for each target.

## Publishing to GitHub Packages

Some targets come with dynamic defaults that support GitHub Packages.
If the respective registry URL is detected to be GitHub, other relevant options will automatically be set to fitting values.
It will also ensure that the workflow token has write permissions for Packages.

**npm**
```ts
publisher.publishToNpm({
  registry: 'npm.pkg.github.com'
  // also sets npmTokenSecret
})
```

**Maven**
```ts
publisher.publishToMaven({
  mavenRepositoryUrl: 'https://maven.pkg.github.com/USER/REPO'
  // also sets mavenServerId, mavenUsername, mavenPassword
  // disables mavenGpgPrivateKeySecret, mavenGpgPrivateKeyPassphrase, mavenStagingProfileId
})
```

## Publishing to AWS CodeArtifact

The NPM target comes with dynamic defaults that support AWS CodeArtifact.
If the respective registry URL is detected to be AWS CodeArtifact, other relevant options will automatically be set to fitting values.
The authentication will done using [AWS CLI](https://docs.aws.amazon.com/codeartifact/latest/ug/tokens-authentication.html). It is neccessary to provide AWS IAM CLI `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in GitHub Secrets.

**npm**
```ts
publisher.publishToNpm({ 
  registry: 'my-domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/npm/my_repo/',
});
```

The names of the GitHub Secrets can be overridden if different names should be used.
```ts
publisher.publishToNpm({ 
  registry: 'my-domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/npm/my_repo/',
  codeArtifactOptions: {
    accessKeyIdSecret: 'CUSTOM_AWS_ACCESS_KEY_ID',
    secretAccessKeySecret: 'CUSTOM_AWS_SECRET_ACCESS_KEY',
  },
});
```
## Handling Failures

You can instruct the publisher to create GitHub issues for publish failures:

```ts
const publisher = new Publisher(project, {
  workflow: releaseWorkflow,
  buildJobId: 'my-build-job',
  artifactName: 'dist',
  issueOnFailure: true,
  failureIssueLabel: 'failed-release'
});
```

This will create an issue labeled with the `failed-release` label for every individual failed publish task.
For example, if Nuget publishing failed for a specific version, it will create an issue titled *Publishing v1.0.4 to Nuget gallery failed*.

This can be helpful to keep track of failed releases as well as integrate with third-party ticketing systems by querying issues labeled with `failed-release`.
