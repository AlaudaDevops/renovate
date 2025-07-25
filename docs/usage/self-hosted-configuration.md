---
title: Self-Hosted configuration
description: Self-Hosted configuration usable in config file, CLI or environment variables
---

# Self-Hosted configuration options

Only use these configuration options when you _self-host_ Renovate.

Do _not_ put the self-hosted config options listed on this page in your "repository config" file (`renovate.json` for example), because Renovate will ignore those config options, and may also create a config error issue.

The config options below _must_ be configured in the bot/admin config, so in either a environment variable, CLI option, or a special file like `config.js`.

<!-- prettier-ignore -->
!!! note
     Renovate supports `JSONC` for `.json` files and any config files without file extension (e.g. `.renovaterc`).

Please also see [Self-Hosted Experimental Options](./self-hosted-experimental.md).

<!-- prettier-ignore -->
!!! note
    Config options with `type=string` are always non-mergeable, so `mergeable=false`.

## allowCustomCrateRegistries

## allowPlugins

## allowScripts

## allowedCommands

A list of regular expressions that decide which commands in `postUpgradeTasks` are allowed to run.

If you are using a template command, the regular expression should match the final resolved value.
If this list is empty then no tasks will be executed.

For example:

```json
{
  "allowedCommands": ["^tslint --fix$", "^tslint --[a-z]+$"]
}
```

This configuration option was formerly known as `allowedPostUpgradeCommands`.

## allowedEnv

Bot administrators can allow users to configure custom environment variables within repo config.
Only environment variables matching the list will be accepted in the [`env`](./configuration-options.md#env) configuration.

Examples:

```json title="renovate.json"
{
  "env": {
    "SOME_ENV_VARIABLE": "some_value",
    "EXTRA_ENV_NAME": "value"
  }
}
```

The above would require `allowedEnv` to be configured similar to the following:

```js title="config.js"
module.exports = {
  allowedEnv: ['SOME_ENV_*', 'EXTRA_ENV_NAME'],
};
```

`allowedEnv` values can be exact match header names, glob patterns, or regex patterns.
For more details on the syntax and supported patterns, see Renovate's [String Pattern Matching documentation](./string-pattern-matching.md).

## allowedHeaders

`allowedHeaders` can be useful when a registry uses a authentication system that's not covered by Renovate's default credential handling in `hostRules`.
By default, all headers starting with "X-" are allowed.
If needed, you can allow additional headers with the `allowedHeaders` option.
Any set `allowedHeaders` overrides the default "X-" allowed headers, so you should include them in your config if you wish for them to remain allowed.
The `allowedHeaders` config option takes an array of minimatch-compatible globs or re2-compatible regex strings.
For more details on this syntax see Renovate's [string pattern matching documentation](./string-pattern-matching.md).

Examples:

| Example header | Kind of pattern  | Explanation                                 |
| -------------- | ---------------- | ------------------------------------------- |
| `/X/`          | Regex            | Any header with `x` anywhere in the name    |
| `!/X/`         | Regex            | Any header without `X` anywhere in the name |
| `X-*`          | Global pattern   | Any header starting with `X-`               |
| `X`            | Exact match glob | Only the header matching exactly `X`        |

```json
{
  "hostRules": [
    {
      "matchHost": "https://domain.com/all-versions",
      "headers": {
        "X-Auth-Token": "secret"
      }
    }
  ]
}
```

Or with custom `allowedHeaders`:

```js title="config.js"
module.exports = {
  allowedHeaders: ['custom-header'],
};
```

## autodiscover

When you enable `autodiscover`, by default, Renovate runs on _every_ repository that the bot account can access.
You can limit which repositories Renovate can access by using the `autodiscoverFilter` config option.

## autodiscoverFilter

You can use this option to filter the list of repositories that the Renovate bot account can access through `autodiscover`.
The pattern matches against the organization/repo path.

This option supports an array of minimatch-compatible globs or RE2-compatible regex strings.
For more details on this syntax see Renovate's [string pattern matching documentation](./string-pattern-matching.md).

If you set multiple filters, then the matches of each filter are added to the overall result.

If you use an environment variable or the CLI to set the value for `autodiscoverFilter`, then commas `,` within filters are not supported.
Commas will be used as delimiter for a new filter.

```
# DO NOT use commas inside the filter if your are using env or cli variables to configure it.
RENOVATE_AUTODISCOVER_FILTER="/MyOrg/{my-repo,foo-repo}"


# in this example you can use regex instead
RENOVATE_AUTODISCOVER_FILTER="/MyOrg\/(my|foo)-repo/"
```

**Minimatch**:

The configuration:

```json
{
  "autodiscoverFilter": ["my-org/*", "!my-org/old-*"]
}
```

Glob patterns are case-insensitive.

**Regex**:

All text inside the start and end `/` will be treated as a regular expression.
If using negations, all repositories except those who match the regex are added to the result:

```json
{
  "autodiscoverFilter": ["/project/.*/", "!/project/old-/"]
}
```

## autodiscoverNamespaces

You can use this option to autodiscover projects in specific namespaces (a.k.a. groups/organizations/workspaces).
In contrast to `autodiscoverFilter` the filtering is done by the platform and therefore more efficient.

For example:

```json
{
  "platform": "gitlab",
  "autodiscoverNamespaces": ["a-group", "another-group/some-subgroup"]
}
```

<!-- prettier-ignore -->
!!! note
    On Gitea/Forgejo, you can't use `autodiscoverTopics` together with `autodiscoverNamespaces` because both platforms do not support this.
    Topics are preferred and `autodiscoverNamespaces` will be ignored when you configure `autodiscoverTopics` on Gitea/Forgejo.

## autodiscoverProjects

You can use this option to filter the list of autodiscovered repositories by project names.
This feature is useful for users who want Renovate to only work on repositories within specific projects or exclude certain repositories from being processed.

```json title="Example for Bitbucket"
{
  "platform": "bitbucket",
  "autodiscoverProjects": ["a-group", "!another-group/some-subgroup"]
}
```

The `autodiscoverProjects` config option takes an array of minimatch-compatible globs or RE2-compatible regex strings.
For more details on this syntax see Renovate's [string pattern matching documentation](./string-pattern-matching.md).

## autodiscoverRepoOrder

The order method for autodiscover server side repository search.

> If multiple `autodiscoverTopics` are used resulting order will be per topic not global.

## autodiscoverRepoSort

The sort method for autodiscover server side repository search.

> If multiple `autodiscoverTopics` are used resulting order will be per topic not global.

## autodiscoverTopics

Some platforms allow you to add tags, or topics, to repositories and retrieve repository lists by specifying those
topics. Set this variable to a list of strings, all of which will be topics for the autodiscovered repositories.

For example:

```json
{
  "autodiscoverTopics": ["managed-by-renovate"]
}
```

## baseDir

By default Renovate uses a temporary directory like `/tmp/renovate` to store its data.
You can override this default with the `baseDir` option.

For example:

```json
{
  "baseDir": "/my-own-different-temporary-folder"
}
```

## bbUseDevelopmentBranch

By default, Renovate will use a repository's "main branch" (typically called `main` or `master`) as the "default branch".

Configuring this to `true` means that Renovate will detect and use the Bitbucket [development branch](https://support.atlassian.com/bitbucket-cloud/docs/branch-a-repository/#The-branching-model) as defined by the repository's branching model.

If the "development branch" is configured but the branch itself does not exist (e.g. it was deleted), Renovate will fall back to using the repository's "main branch". This fall back behavior matches that of the Bitbucket Cloud web interface.

## binarySource

Renovate often needs to use third-party tools in its PRs, like `npm` to update `package-lock.json` or `go` to update `go.sum`.

Renovate supports four possible ways to access those tools:

- `global`: Uses pre-installed tools, e.g. `npm` installed via `npm install -g npm`.
- `install` (default): Downloads and installs tools at runtime if running in a [Containerbase](https://github.com/containerbase/base) environment, otherwise falls back to `global`
- `docker`: Runs tools inside Docker "sidecar" containers using `docker run`.
- `hermit`: Uses the [Hermit](https://github.com/cashapp/hermit) tool installation approach.

Starting in v36, Renovate's default Docker image (previously referred to as the "slim" image) uses `binarySource=install` while the "full" Docker image uses `binarySource=global`.
If you are running Renovate in an environment where runtime download and install of tools is not possible then you should use the "full" image.

If you are building your own Renovate image, e.g. by installing Renovate using `npm`, then you will need to ensure that all necessary tools are installed globally before running Renovate so that `binarySource=global` will work.

The `binarySource=docker` approach should not be necessary in most cases now and `binarySource=install` is recommended instead.
If you have a use case where you cannot use `binarySource=install` but can use `binarySource=docker` then please share it in a GitHub Discussion so that the maintainers can understand it.
For this to work, `docker` needs to be installed and the Docker socket available to Renovate.

## cacheDir

By default Renovate stores cache data in a temporary directory like `/tmp/renovate/cache`.
Use the `cacheDir` option to override this default.

The `baseDir` and `cacheDir` option may point to different directories.
You can use one directory for the repo data, and another for the cache data.

For example:

```json
{
  "baseDir": "/my-own-different-temporary-folder",
  "cacheDir": "/my-own-different-cache-folder"
}
```

## cacheHardTtlMinutes

This experimental feature is used to implement the concept of a "soft" cache expiry for datasources, starting with `npm`.
It should be set to a non-zero value, recommended to be at least 60 (i.e. one hour).

When this value is set, the `npm` datasource will use the `cacheHardTtlMinutes` value for cache expiry, instead of its default expiry of 15 minutes, which becomes the "soft" expiry value.
Results which are soft expired are reused in the following manner:

- The `etag` from the cached results will be reused, and may result in a 304 response, meaning cached results are revalidated
- If an error occurs when querying the `npmjs` registry, then soft expired results will be reused if they are present

## cachePrivatePackages

In the self-hosted setup, use option to enable caching of private packages to improve performance.

## cacheTtlOverride

Utilize this key-value map to override the default package cache TTL values for a specific namespace. This object contains pairs of namespaces and their corresponding TTL values in minutes.
For example, to override the default TTL of 60 minutes for the `docker` datasource "tags" namespace: `datasource-docker-tags` use the following:

```json
{
  "cacheTtlOverride": {
    "datasource-docker-tags": 120
  }
}
```

Valid codes for namespaces are as follows:

- `changelog-bitbucket-notes@v2`
- `changelog-bitbucket-release`
- `changelog-gitea-notes@v2`
- `changelog-gitea-release`
- `changelog-github-notes@v2`
- `changelog-github-release`
- `changelog-gitlab-notes@v2`
- `changelog-gitlab-release`
- `datasource-artifactory`
- `datasource-aws-machine-image`
- `datasource-aws-rds`
- `datasource-azure-bicep-resource`
- `datasource-azure-pipelines-tasks`
- `datasource-bazel`
- `datasource-bitbucket-tags`
- `datasource-bitrise`
- `datasource-cdnjs`
- `datasource-conan`
- `datasource-conda`
- `datasource-cpan`
- `datasource-crate-metadata`
- `datasource-crate`
- `datasource-deb`
- `datasource-deno`
- `datasource-docker-architecture`
- `datasource-docker-hub-cache`
- `datasource-docker-digest`
- `datasource-docker-hub-tags`
- `datasource-docker-imageconfig`
- `datasource-docker-labels`
- `datasource-docker-releases-v2`
- `datasource-docker-tags`
- `datasource-dotnet-version`
- `datasource-endoflife-date`
- `datasource-galaxy-collection`
- `datasource-galaxy`
- `datasource-git-refs`
- `datasource-git-tags`
- `datasource-git`
- `datasource-gitea-releases`
- `datasource-gitea-tags`
- `datasource-github-release-attachments`
- `datasource-gitlab-packages`
- `datasource-gitlab-releases`
- `datasource-gitlab-tags`
- `datasource-glasskube-packages`
- `datasource-go-direct`
- `datasource-go-proxy`
- `datasource-go`
- `datasource-golang-version`
- `datasource-gradle-version`
- `datasource-helm`
- `datasource-hermit`
- `datasource-hex`
- `datasource-hexpm-bob`
- `datasource-java-version`
- `datasource-jenkins-plugins`
- `datasource-maven:cache-provider`
- `datasource-maven:postprocess-reject`
- `datasource-node-version`
- `datasource-npm:data`
- `datasource-nuget-v3`
- `datasource-orb`
- `datasource-packagist`
- `datasource-pod`
- `datasource-python-version`
- `datasource-releases`
- `datasource-repology`
- `datasource-ruby-version`
- `datasource-rubygems`
- `datasource-sbt-package`
- `datasource-terraform-module`
- `datasource-terraform-provider`
- `datasource-terraform`
- `datasource-unity3d`
- `github-releases-datasource-v2`
- `github-tags-datasource-v2`
- `merge-confidence`
- `preset`
- `terraform-provider-hash`
- `url-sha256`

## checkedBranches

This array will allow you to set the names of the branches you want to rebase/create, as if you selected their checkboxes in the Dependency Dashboard issue.

It has been designed with the intention of being run on one repository, in a one-off manner, e.g. to "force" the rebase of a known existing branch.
It is highly unlikely that you should ever need to add this to your permanent global config.

Example: `renovate --checked-branches=renovate/chalk-4.x renovate-reproductions/checked` will rebase the `renovate/chalk-4.x` branch in the `renovate-reproductions/checked` repository.`

## containerbaseDir

This directory is used to cache downloads when `binarySource=docker` or `binarySource=install`.

Use this option if you need such downloads to be stored outside of Renovate's regular cache directory (`cacheDir`).

## customEnvVariables

This configuration will be applied after all other environment variables so you can use it to override defaults.

<!-- prettier-ignore -->
!!! warning
    Do not configure any secret values directly into `customEnvVariables` because they may be logged to stdout.
    Instead, configure them into `secrets` first so that they will be redacted in logs.

If configuring secrets in to `customEnvVariables`, take this approach:

```js
{
  secrets: {
    SECRET_TOKEN: process.env.SECRET_TOKEN,
  },
  customEnvVariables: {
    SECRET_TOKEN: '{{ secrets.SECRET_TOKEN }}',
  },
}
```

The above configuration approach will mean the values are redacted in logs like in the following example:

```
         "secrets": {"SECRET_TOKEN": "***********"},
         "customEnvVariables": {"SECRET_TOKEN": "{{ secrets.SECRET_TOKEN }}"},
```

## deleteConfigFile

If set to `true` Renovate tries to delete the self-hosted config file after reading it.

The process that runs Renovate must have the correct permissions to delete the config file.

<!-- prettier-ignore -->
!!! tip
    You can tell Renovate where to find your config file with the `RENOVATE_CONFIG_FILE` environment variable.

## detectGlobalManagerConfig

The purpose of this config option is to allow you (as a bot admin) to configure manager-specific files such as a global `.npmrc` file, instead of configuring it in Renovate config.

This config option is disabled by default because it may prove surprising or undesirable for some users who don't expect Renovate to go into their home directory and import registry or credential information.

Currently this config option is supported for the `npm` manager only - specifically the `~/.npmrc` file.
If found, it will be imported into `config.npmrc` with `config.npmrcMerge` set to `true`.

## detectHostRulesFromEnv

The format of the environment variables must follow:

- `RENOVATE_` prefix (at the moment this prefix optional, but usage of prefix will be required in the future)
- Datasource name (e.g. `NPM`, `PYPI`) or Platform name (only `GITHUB`)
- Underscore (`_`)
- `matchHost` (note: only domains or subdomains are supported - not `https://` URLs or anything with forward slashes)
- Underscore (`_`)
- Field name (`TOKEN`, `USERNAME`, `PASSWORD`, `HTTPSPRIVATEKEY`, `HTTPSCERTIFICATE`, `HTTPSCERTIFICATEAUTHORITY`)

Hyphens (`-`) in datasource or host name must be replaced with double underscores (`__`).
Periods (`.`) in host names must be replaced with a single underscore (`_`).

<!-- prettier-ignore -->
!!! note
    You can't use these prefixes with the `detectHostRulesFromEnv` config option: `npm_config_`, `npm_lifecycle_`, `npm_package_`.
    In addition, platform host rules will only be picked up when `matchHost` is supplied.

### npmjs registry token example

`NPM_REGISTRY_NPMJS_ORG_TOKEN=abc123`:

```json
{
  "hostRules": [
    {
      "hostType": "npm",
      "matchHost": "registry.npmjs.org",
      "token": "abc123"
    }
  ]
}
```

### GitLab Tags username/password example

`GITLAB__TAGS_CODE__HOST_COMPANY_COM_USERNAME=bot GITLAB__TAGS_CODE__HOST_COMPANY_COM_PASSWORD=botpass123`:

```json
{
  "hostRules": [
    {
      "hostType": "gitlab-tags",
      "matchHost": "code-host.company.com",
      "username": "bot",
      "password": "botpass123"
    }
  ]
}
```

### Datasource and credentials only

You can skip the host part, and use only the datasource and credentials.

`DOCKER_USERNAME=bot DOCKER_PASSWORD=botpass123`:

```json
{
  "hostRules": [
    {
      "hostType": "docker",
      "username": "bot",
      "password": "botpass123"
    }
  ]
}
```

### Platform with https authentication options

`GITHUB_SOME_GITHUB__ENTERPRISE_HOST_HTTPSCERTIFICATE=certificate GITHUB_SOME_GITHUB__ENTERPRISE_HOST_HTTPSPRIVATEKEY=private-key GITHUB_SOME_GITHUB__ENTERPRISE_HOST_HTTPSCERTIFICATEAUTHORITY=certificate-authority`:

```json
{
  "hostRules": [
    {
      "hostType": "github",
      "matchHost": "some.github-enterprise.host",
      "httpsPrivateKey": "private-key",
      "httpsCertificate": "certificate",
      "httpsCertificateAuthority": "certificate-authority"
    }
  ]
}
```

## dockerChildPrefix

Adds a custom prefix to the default Renovate sidecar Docker containers name and label.

For example, if you set `dockerChildPrefix=myprefix_` then the final container created from the `containerbase/sidecar` is:

- called `myprefix_sidecar` instead of `renovate_sidecar`
- labeled `myprefix_child` instead of `renovate_child`

<!-- prettier-ignore -->
!!! note
    Dangling containers are only removed when Renovate runs again with the same prefix.

## dockerCliOptions

You can use `dockerCliOptions` to pass Docker CLI options to Renovate's sidecar Docker containers.

For example, `{"dockerCliOptions": "--memory=4g"}` will add a CLI flag to the `docker run` command that limits the amount of memory Renovate's sidecar Docker container can use to 4 gigabytes.

Read the [Docker Docs, configure runtime resource constraints](https://docs.docker.com/config/containers/resource_constraints/) to learn more.

## dockerMaxPages

If set to an positive integer, Renovate will use this value as the maximum page number.
Setting a different limit is useful for registries that ignore the `n` parameter in Renovate's query string and thus only return 50 tags per page.

## dockerSidecarImage

By default Renovate pulls the sidecar Docker containers from `ghcr.io/containerbase/sidecar`.
You can use the `dockerSidecarImage` option to override this default.

Say you want to pull a custom image from `ghcr.io/your_company/sidecar`.
You would put this in your configuration file:

```json
{
  "dockerSidecarImage": "ghcr.io/your_company/sidecar"
}
```

Now when Renovate pulls a new `sidecar` image, the final image is `ghcr.io/your_company/sidecar` instead of `ghcr.io/containerbase/sidecar`.

## dockerUser

Override default user and group used by Docker-based tools.
The user-id (UID) and group-id (GID) must match the user that executes Renovate.

Read the [Docker run reference](https://docs.docker.com/engine/reference/run/#user) for more information on user and group syntax.
Set this to `1001:1002` to use UID 1001 and GID 1002.

```json title="Setting UID to 1001 and GID to 1002"
{
  "dockerUser": "1001:1002"
}
```

If you use `binarySource=docker|install` read the section below.

If you need to change the Docker user please make sure to use the root (`0`) group, otherwise you'll get in trouble with missing file and directory permissions.
Like this:

```
> export RENOVATE_DOCKER_USER="$(id -u):0" # 500:0 (username:root)
```

## dryRun

Use `dryRun` to preview the behavior of Renovate in logs, without making any changes to the repository files.

You can choose from the following behaviors for the `dryRun` config option:

- `null`: Default behavior - Performs a regular Renovate run including creating/updating/deleting branches and PRs
- `"extract"`: Performs a very quick package file scan to identify the extracted dependencies
- `"lookup"`: Performs a package file scan to identify the extracted dependencies and updates available
- `"full"`: Performs a dry run by logging messages instead of creating/updating/deleting branches and PRs

Information provided mainly in debug log level.

## encryptedWarning

Use this if you want to stop supporting `encrypted` configuration capabilities but want to warn users first to migrate.

If set to a string value, Renovate will log warnings with the `encryptedWarning` text, meaning the message will be visible to users such as on the Dependency Dashboard.

## endpoint

## executionTimeout

Default execution timeout in minutes for child processes Renovate creates.
If this option is not set, Renovate will fallback to 15 minutes.

## exposeAllEnv

To keep you safe, Renovate only passes a limited set of environment variables to package managers.
If you must expose all environment variables to package managers, you can set this option to `true`.

<!-- prettier-ignore -->
!!! warning
    Always consider the security implications of using `exposeAllEnv`!
    Secrets and other confidential information stored in environment variables could be leaked by a malicious script, that enumerates all environment variables.

Set `exposeAllEnv` to `true` only if you have reviewed, and trust, the repositories which Renovate bot runs against.
Alternatively, you can use the [`customEnvVariables`](./self-hosted-configuration.md#customenvvariables) config option to handpick a set of variables you need to expose.

Setting this to `true` also allows for variable substitution in `.npmrc` files.

## force

This object is used as a "force override" when you need to make sure certain configuration overrides whatever is configured in the repository.
For example, forcing a null (no) schedule to make sure Renovate raises PRs on a run even if the repository itself or its preset defines a schedule that's currently inactive.

In practice, it is implemented by converting the `force` configuration into a `packageRule` that matches all packages.

## forceCli

This is set to `true` by default, meaning that any settings (such as `schedule`) take maximum priority even against custom settings existing inside individual repositories.
It will also override any settings in `packageRules`.

## forkCreation

This configuration lets you disable the runtime forking of repositories when running in "fork mode".

Usually you will need to keep this as the default `true`, and only set to `false` if you have some out of band process to handle the creation of forks.

## forkOrg

This configuration option lets you choose an organization you want repositories forked into when "fork mode" is enabled.
It must be set to a GitHub Organization name and not a GitHub user account.
When set, "allow edits by maintainers" will be false for PRs because GitHub does not allow this setting for organizations.

This can be used if you're migrating from user-based forks to organization-based forks.

If you've set a `forkOrg` then Renovate will:

1. Check if a fork exists in the preferred organization before checking it exists in the fork user's account
1. If no fork exists: it will be created in the `forkOrg`, not the user account

## forkToken

If this value is configured then Renovate:

- forks the target repository into the account that owns the PAT
- keep this fork's default branch up-to-date with the target

Renovate will then create branches on the fork and opens Pull Requests on the parent repository.

<!-- prettier-ignore -->
!!! note
    Forked repositories will always be skipped when `forkToken` is set, even if `includeForks` is true.

## gitNoVerify

Controls when Renovate passes the `--no-verify` flag to `git`.
The flag can be passed to `git commit` and/or `git push`.
Read the documentation for [git commit --no-verify](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---no-verify) and [git push --no-verify](https://git-scm.com/docs/git-push#Documentation/git-push.txt---no-verify) to learn exactly what each flag does.
To learn more about Git hooks, read the [Pro Git 2 book, section on Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

## gitPrivateKey

This is a private PGP or SSH key for signing Git commits.

For PGP, it should be an armored private key, so the type you get from running `gpg --export-secret-keys --armor 92066A17F0D1707B4E96863955FEF5171C45FAE5 > private.key`.
Replace the newlines with `\n` before adding the resulting single-line value to your bot's config.

<!-- prettier-ignore -->
!!! note
    The private key can't be protected with a passphrase if running in a headless environment. Renovate will not be able to handle entering the passphrase.

It will be loaded _lazily_.
Before the first commit in a repository, Renovate will:

1. Run `gpg import` (if you haven't before) when using PGP
1. Run `git config user.signingkey`, `git config commit.gpgsign true` and `git config gpg.format`

The `git` commands are run locally in the cloned repo instead of globally.
This reduces the chance of unintended consequences with global Git configs on shared systems.

## gitTimeout

To handle the case where the underlying Git processes appear to hang, configure the timeout with the number of milliseconds to wait after last received content on either `stdOut` or `stdErr` streams before sending a `SIGINT` kill message.

## gitUrl

Override the default resolution for Git remote, e.g. to switch GitLab from HTTPS to SSH-based.

Possible values:

- `default`: use HTTPS URLs provided by the platform for Git
- `ssh`: use SSH URLs provided by the platform for Git
- `endpoint`: ignore URLs provided by the platform and use the configured endpoint directly

## githubTokenWarn

By default, Renovate logs and displays a warning when the `RENOVATE_GITHUB_COM_TOKEN` is not set.
By setting `githubTokenWarn` to `false`, Renovate suppresses these warnings on Pull Requests, etc.
Disabling the warning is helpful for self-hosted environments that can't access the `github.com` domain, because the warning is useless in these environments.

## globalExtends

Unlike the `extends` field, which is passed through unresolved to be part of repository config, any presets in `globalExtends` are resolved immediately as part of global config.
Use the `globalExtends` field if your preset has any global-only configuration options, such as the list of repositories to run against.

Use the `extends` field instead of this if, for example, you need the ability for a repository config (e.g. `renovate.json`) to be able to use `ignorePresets` for any preset defined in global config.

<!-- prettier-ignore -->
!!! warning
    `globalExtends` presets can't be private.
    When Renovate resolves `globalExtends` it does not fully process the configuration.
    This means that Renovate does not have the authentication it needs to fetch private things.

## httpCacheTtlDays

This option sets the number of days that Renovate will cache HTTP responses.
The default value is 90 days.
Value of `0` means no caching.

<!-- prettier-ignore -->
!!! warning
    When you set `httpCacheTtlDays` to `0`, Renovate will remove the cached HTTP data.

## includeMirrors

By default, Renovate does not autodiscover repositories that are mirrors.

Change this setting to `true` to include repositories that are mirrors as Renovate targets.

## inheritConfig

When you enable this option, Renovate will look for the `inheritConfigFileName` file in the `inheritConfigRepoName` repository before processing a repository, and read this in as config.

If the repository is in a nested organization or group on a supported platform such as GitLab, such as `topGroup/nestedGroup/projectName` then Renovate will look in `topGroup/nestedGroup/renovate-config`.

If `inheritConfig` is `true` but the inherited config file does _not_ exist then Renovate will proceed without warning.
If the file exists but cannot be parsed, then Renovate will raise a config warning issue and abort the job.

The inherited config may include all valid repository config and these config options:

- `bbUseDevelopmentBranch`
- `onboarding`
- `onboardingBranch`
- `onboardingCommitMessage`
- `onboardingConfig`
- `onboardingConfigFileName`
- `onboardingNoDeps`
- `onboardingPrTitle`
- `onboardingRebaseCheckbox`
- `requireConfig`

<!-- prettier-ignore -->
!!! note
    The above list is prepared manually and may become out of date.
    Consult the self-hosted configuration docs and look for `inheritConfigSupport` values there for the definitive list.

This way organizations can change/control the default behavior, like whether configs are required and how repositories are onboarded.

We disabled `inheritConfig` in the Mend Renovate App to avoid wasting millions of API calls per week.
This is because each `404` response from the GitHub API due to a missing org inherited config counts as a used API call.
We will add a smart/dynamic approach in future, so that we can selectively enable `inheritConfig` per organization.

## inheritConfigFileName

Change this setting if you want Renovate to look for a different file name within the `inheritConfigRepoName` repository.
You may use nested files, for example: `"some-dir/config.json"`.

## inheritConfigRepoName

Change this setting if you want Renovate to look in an alternative repository for the inherited config.
The repository must be on the same platform and endpoint, and Renovate's token must have `read` permissions to the repository.

## inheritConfigStrict

By default Renovate will silently (debug log message only) ignore cases where `inheritConfig=true` but no inherited config is found.
When you set `inheritConfigStrict=true` then Renovate will abort the run and raise a config error if Renovate can't find the inherited config.

<!-- prettier-ignore -->
!!! warning
    Only set this config option to `true` if _every_ organization has an inherited config file _and_ you want to make sure Renovate _always_ uses that inherited config.

## logContext

`logContext` is included with each log entry only if `logFormat="json"` - it is not included in the pretty log output.
If left as default (null), a random short ID will be selected.

## mergeConfidenceDatasources

This feature is applicable only if you have an access token for Mend's Merge Confidence API.

If set, Renovate will query the merge-confidence JSON API only for datasources that are part of this list.
Otherwise, it queries all the supported datasources (check default value).

Example:

```js
modules.exports = {
  mergeConfidenceDatasources: ['npm'],
};
```

## mergeConfidenceEndpoint

This feature is applicable only if you have an access token for Mend's Merge Confidence API.

If set, Renovate will retrieve Merge Confidence data by querying this API.
Otherwise, it will use the default URL, which is <https://developer.mend.io/>.

If you use the Mend Renovate Enterprise Edition (Renovate EE) and:

- have a static merge confidence token that you set via `MEND_RNV_MC_TOKEN`
- _or_ set `MEND_RNV_MC_TOKEN` to `auto`

Then you must set this variable at the _server_ and the _workers_.

But if you have specified the token as a [`matchConfidence`](configuration-options.md#matchconfidence) `hostRule`, you only need to set this variable at the _workers_.

This feature is in private beta.

## migratePresets

Use this if you have repositories that extend from a particular preset, which has now been renamed or removed.
This is handy if you have a large number of repositories that all extend from a particular preset which you want to rename, without the hassle of manually updating every repository individually.
Use an empty string to indicate that the preset should be ignored rather than replaced.

Example:

```js
modules.exports = {
  migratePresets: {
    '@company': 'local>org/renovate-config',
  },
};
```

In the above example any reference to the `@company` preset will be replaced with `local>org/renovate-config`.

<!-- prettier-ignore -->
!!! tip
    Combine `migratePresets` with `configMigration` if you'd like your config migrated by PR.

## onboarding

Only set this to `false` if all three statements are true:

- You've configured Renovate entirely on the bot side (e.g. empty `renovate.json` in repositories)
- You want to run Renovate on every repository the bot has access to
- You want to skip all onboarding PRs

## onboardingBranch

<!-- prettier-ignore -->
!!! note
    This setting is independent of `branchPrefix`.

For example, if you configure `branchPrefix` to be `renovate-` then you'd still have the onboarding PR created with branch `renovate/configure` until you configure `onboardingBranch=renovate-configure` or similar.
If you have an existing Renovate installation and you change `onboardingBranch` then it's possible that you'll get onboarding PRs for repositories that had previously closed the onboarding PR unmerged.

## onboardingCommitMessage

If `commitMessagePrefix` or `semanticCommits` values are set then they will be prepended to the commit message using the same logic that is used for adding them to non-onboarding commit messages.

## onboardingConfig

## onboardingConfigFileName

If set to one of the valid [config file names](./configuration-options.md), the onboarding PR will create a configuration file with the provided name instead of `renovate.json`.
Falls back to `renovate.json` if the name provided is not valid.

## onboardingNoDeps

The default `auto` setting is converted to `disabled` if `autodiscoverRepositories` is `true`, or converted to `enabled` if false.

In other words, the default behavior is:

- If you run Renovate on discovered repositories then it will skip onboarding those without dependencies detected, but
- If you run Renovate on _specific_ repositories then Renovate will onboard all such repositories even if no dependencies are found

## onboardingPrTitle

If you have an existing Renovate installation and you change the `onboardingPrTitle`: then you may get onboarding PRs _again_ for repositories with closed non-merged onboarding PRs.
This is similar to what happens when you change the `onboardingBranch` config option.

## onboardingRebaseCheckbox

## optimizeForDisabled

When this option is `true`, Renovate will do the following during repository initialization:

1. Try to fetch the default config file (e.g. `renovate.json`)
1. Check if the file contains `"enabled": false`
1. If so, skip cloning and skip the repository immediately

If `onboardingConfigFileName` is set, that file name will be used instead of the default.

If the file exists and the config is disabled, Renovate will skip the repo without cloning it.
Otherwise, it will continue as normal.

`optimizeForDisabled` can make initialization quicker in cases where most repositories are disabled, but it uses an extra API call for enabled repositories.

A second, advanced, use also exists when the bot global config has `extends: [":disableRenovate"]`.
In that case, Renovate searches the repository config file for any of these configurations:

- `extends: [":enableRenovate"]`
- `ignorePresets: [":disableRenovate"]`
- `enabled: true`

If Renovate finds any of the above configurations, it continues initializing the repository.
If not, then Renovate skips the repository without cloning it.

## password

## persistRepoData

Set this to `true` if you want Renovate to persist repo data between runs.
The intention is that this allows Renovate to do a faster `git fetch` between runs rather than `git clone`.
It also may mean that ignored directories like `node_modules` can be preserved and save time on operations like `npm install`.

## platform

## prCommitsPerRunLimit

Parameter to reduce CI load.
CI jobs are usually triggered by these events: pull-request creation, pull-request update, automerge events.
Set as an integer.
Default is no limit.

## presetCachePersistence

When this feature is enabled, resolved presets will be cached in Renovate's package cache, enabling reuse across multiple repositories.

TTL is 15 minutes by default, and it is adjustable in [cacheTtlOverride](#cachettloverride).

<!-- prettier-ignore -->
!!! warning
     Doing so improves efficiency because shared presets don't need to be reloaded/resolved for every repository, however it also means that private presets can be "leaked" between repositories.
     You should only enable this when all repositories are trusted, such as a corporate environment.

## privateKey

This private key is used to decrypt config files.

The corresponding public key can be used to create encrypted values for config files.
If you want a UI to encrypt values you can put the public key in a HTML page similar to <https://app.renovatebot.com/encrypt>.

To create the PGP key pair with GPG use the following commands:

- `gpg --full-generate-key` and follow the prompts to generate a key. Name and email are not important to Renovate, and do not configure a passphrase. Use a 4096bit key.

<details><summary>key generation log</summary>

```
❯ gpg --full-generate-key
gpg (GnuPG) 2.2.24; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Renovate Bot
Email address: renovate@whitesourcesoftware.com
Comment:
You selected this USER-ID:
    "Renovate Bot <renovate@whitesourcesoftware.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

gpg: key 0649CC3899F22A66 marked as ultimately trusted
gpg: revocation certificate stored as '/Users/rhys/.gnupg/openpgp-revocs.d/794B820F34B34A8DF32AADB20649CC3899F22A66.rev'
public and secret key created and signed.

pub   rsa4096 2021-09-10 [SC]
      794B820F34B34A8DF32AADB20649CEXAMPLEONLY
uid                      Renovate Bot <renovate@whitesourcesoftware.com>
sub   rsa4096 2021-09-10 [E]
```

</details>

<!-- prettier-ignore -->
!!! note
    If you use GnuPG `v2.4` (or newer) to generate the key, then you must disable `AEAD` preferences.
    This is needed to allow Renovate to decrypt the encrypted values.

<details><summary>key edit log</summary>

```bash
❯ gpg --edit-key renovate@whitesourcesoftware.com
gpg> showpref
[ultimate] (1). Renovate Bot <renovate@whitesourcesoftware.com>
     Cipher: AES256, AES192, AES, 3DES
     AEAD: OCB, EAX
     Digest: SHA512, SHA384, SHA256, SHA224, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, AEAD, Keyserver no-modify

gpg> setpref AES256 AES192 AES 3DES SHA512 SHA384 SHA256 SHA224 SHA1 ZLIB BZIP2 ZIP
Set preference list to:
     Cipher: AES256, AES192, AES, 3DES
     AEAD:
     Digest: SHA512, SHA384, SHA256, SHA224, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, Keyserver no-modify
Really update the preferences? (y/N) y
gpg> save
```

</details>

- Copy the key ID from the output (`794B820F34B34A8DF32AADB20649CEXAMPLEONLY` in the above example) or run `gpg --list-secret-keys` if you forgot to take a copy
- Run `gpg --armor --export-secret-keys YOUR_NEW_KEY_ID > renovate-private-key.asc` to generate an armored (text-based) private key file
- Run `gpg --armor --export YOUR_NEW_KEY_ID > renovate-public-key.asc` to generate an armored (text-based) public key file

The private key should then be added to your Renovate Bot global config (either using `privateKeyPath` or exporting it to the `RENOVATE_PRIVATE_KEY` environment variable).
The public key can be used to replace the existing key in <https://app.renovatebot.com/encrypt> for your own use.

Any PGP-encrypted secrets must have a mandatory organization/group scope, and optionally can be scoped for a single repository only.
The reason for this is to avoid "replay" attacks where someone could learn your encrypted secret and then reuse it in their own Renovate repositories.
Instead, with scoped secrets it means that Renovate ensures that the organization and optionally repository values encrypted with the secret match against the running repository.

<!-- prettier-ignore -->
!!! note
    You could use public key encryption with earlier versions of Renovate.
    We deprecated this approach and removed the documentation for it.
    If you're _still_ using public key encryption then we recommend that you use private keys instead.

## privateKeyOld

Use this field if you need to perform a "key rotation" and support more than one keypair at a time.
Decryption with this key will be tried after `privateKey`.

If you are migrating from the legacy public key encryption approach to use a PGP key, then move your legacy private key from `privateKey` to `privateKeyOld` and then put your new PGP private key in `privateKey`.
Doing so will mean that Renovate will first try to decrypt using the PGP key but fall back to the legacy key and try that next.

You can remove the `privateKeyOld` config option once all the old encrypted values have been migrated, or if you no longer want to support the old key and let the processing of repositories fail.

<!-- prettier-ignore -->
!!! note
    Renovate now logs a warning whenever repositories use non-PGP encrypted config variables.

## privateKeyPath

Used as an alternative to `privateKey`, if you want the key to be read from disk instead.

## privateKeyPathOld

Used as an alternative to `privateKeyOld`, if you want the key to be read from disk instead.

## processEnv

Used to set environment variables through the configuration file instead of using actual environment variables.

Example:

```json
{
  "processEnv": {
    "AWS_ACCESS_KEY_ID": "AKIAIOSFODNN7EXAMPLE",
    "AWS_SECRET_ACCESS_KEY": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
    "AWS_DEFAULT_REGION": "us-west-2"
  }
}
```

<!-- prettier-ignore -->
!!! note

- All values must be provided as strings, e.g., `"true"` instead of `true`
- Only supported in file configuration (not via CLI or environment).

## productLinks

Override this object if you want to change the URLs that Renovate links to, e.g. if you have an internal forum for asking for help.

## redisPrefix

If this value is set then Renovate will prepend this string to the name of all Redis cache entries used in Renovate.
It's only used if `redisUrl` is configured.

## redisUrl

If this value is set then Renovate will use Redis for its global cache instead of the local file system.
The global cache is used to store lookup results (e.g. dependency versions and changelogs) between repositories and runs.

For non encrypted connections,

Example URL structure: `redis://[[username]:[password]]@localhost:6379/0`.

For TLS/SSL-enabled connections, use rediss prefix

Example URL structure: `rediss://[[username]:[password]]@localhost:6379/0`.

Renovate also supports connecting to Redis clusters as well. In order to connect to a cluster, provide the connection string using the `redis+cluster` or `rediss+cluster` schema as appropriate.

Example URL structure: `redis+cluster://[[username]:[password]]@redis.cluster.local:6379/0`

## reportPath

`reportPath` describes the location where the report is written to.

If [`reportType`](#reporttype) is set to `file`, then set `reportPath` to a filepath.
For example: `/foo/bar.json`.

If the value `s3` is used in [`reportType`](#reporttype), then use a S3 URI.
For example: `s3://bucket-name/key-name`.

## reportType

Defines how the report is exposed:

- `<unset>` If unset, no report will be provided, though the debug logs will still have partial information of the report
- `logging` The report will be printed as part of the log messages on `INFO` level
- `file` The report will be written to a path provided by [`reportPath`](#reportpath)
- `s3` The report is pushed to an S3 bucket defined by [`reportPath`](#reportpath). This option reuses [`s3Endpoint`](#s3endpoint) and [`s3PathStyle`](#s3pathstyle)

## repositories

Elements in the `repositories` array can be an object if you wish to define more settings.
Example:

```js
{
  repositories: [{ repository: 'g/r1', bumpVersion: 'patch' }, 'g/r2'];
}
```

## repositoryCache

Set this to `"enabled"` to have Renovate maintain a JSON file cache per-repository to speed up extractions.
Set to `"reset"` if you ever need to bypass the cache and have it overwritten.
JSON files will be stored inside the `cacheDir` beside the existing file-based package cache.

## repositoryCacheType

```ts title="Set repositoryCacheType to an S3 URI to enable S3 backed repository cache"
{
  repositoryCacheType: 's3://bucket-name';
}
```

Renovate uses the [AWS SDK for JavaScript V3](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/welcome.html) to connect to the S3 instance.
Therefore, Renovate supports all the authentication methods supported by the AWS SDK.
Read more about [the default credential provider chain for AWS SDK for JavaScript V3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-credential-providers/#fromnodeproviderchain).

<!-- prettier-ignore -->
!!! tip
    If you're storing the repository cache on Amazon S3 then you may set a folder hierarchy as part of `repositoryCacheType`.
    For example, `repositoryCacheType: 's3://bucket-name/dir1/.../dirN/'`.

<!-- prettier-ignore -->
!!! note
    S3 repository is used as a repository cache (e.g. extracted dependencies) and not a lookup cache (e.g. available versions of dependencies). To keep the later remotely, define [Redis URL](#redisurl).

## requireConfig

By default, Renovate needs a Renovate config file in each repository where it runs before it will propose any dependency updates.

You can choose any of these settings:

- `"required"` (default): a repository config file must be present
- `"optional"`: if a config file exists, Renovate will use it when it runs
- `"ignored"`: config files in the repo will be ignored, and have no effect

This feature is closely related to the `onboarding` config option.
The combinations of `requireConfig` and `onboarding` are:

|                          | `onboarding=true`                                                                                                                                       | `onboarding=false`                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `requireConfig=required` | An onboarding PR will be created if no config file exists. If the onboarding PR is closed and there's no config file, then the repository is skipped.   | Repository is skipped unless a config file is added manually. |
| `requireConfig=optional` | An onboarding PR will be created if no config file exists. If the onboarding PR is closed and there's no config file, the repository will be processed. | Repository is processed regardless of config file presence.   |
| `requireConfig=ignored`  | No onboarding PR will be created and repo will be processed while ignoring any config file present.                                                     | Repository is processed, any config file is ignored.          |

## s3Endpoint

If set, Renovate will use this string as the `endpoint` when creating the AWS S3 client instance.

## s3PathStyle

If set, Renovate will enable `forcePathStyle` when creating the AWS S3 client instance.

For example:

| `s3PathStyle` | Path                               |
| ------------- | ---------------------------------- |
| `off`         | `https://bucket.s3.amazonaws.com/` |
| `on`          | `https://s3.amazonaws.com/bucket/` |

Read the [AWS S3 docs, Interface BucketEndpointInputConfig](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/bucketendpointinputconfig.html) to learn more about path-style URLs.

## secrets

Secrets may be configured by a bot admin in `config.js`, which will then make them available for templating within repository configs.
For example, to configure a `GOOGLE_TOKEN` to be accessible by all repositories:

```js
module.exports = {
  secrets: {
    GOOGLE_TOKEN: 'abc123',
  },
};
```

They can also be configured per repository, e.g.

```js
module.exports = {
  repositories: [
    {
      repository: 'abc/def',
      secrets: {
        GOOGLE_TOKEN: 'abc123',
      },
    },
  ],
};
```

It could then be used in a repository config or preset like so:

```json
{
  "hostRules": [
    {
      "matchHost": "google.com",
      "token": "{{ secrets.GOOGLE_TOKEN }}"
    }
  ]
}
```

Secret names must start with an upper or lower case character and can have only characters, digits, or underscores.

## token

## unicodeEmoji

If enabled emoji shortcodes are replaced with their Unicode equivalents.
For example: `:warning:` will be replaced with `⚠️`.

## useCloudMetadataServices

Some cloud providers offer services to receive metadata about the current instance, for example [AWS Instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-instance-metadata.html) or [GCP VM metadata](https://cloud.google.com/compute/docs/metadata/overview).
You can control if Renovate should try to access these services with the `useCloudMetadataServices` config option.

## userAgent

If set to any string, Renovate will use this as the `user-agent` it sends with HTTP requests.
Otherwise, it will default to `RenovateBot/${renovateVersion} (https://github.com/renovatebot/renovate)`.

## username

The only time where `username` is required is if using `username` + `password` credentials for the `bitbucket` platform.
You don't need to configure `username` directly if you have already configured `token`.
Renovate will use the token to discover its username on the platform, including if you're running Renovate as a GitHub App.

## variables

Variables may be configured by a bot admin in `config.js`, which will then make them available for templating within repository configs.
This config option behaves exactly like [secrets](#secrets), except that it won't be masked in the logs.
For example, to configure a `SOME_VARIABLE` to be accessible by all repositories:

```js
module.exports = {
  variables: {
    SOME_VARIABLE: 'abc123',
  },
};
```

They can also be configured per repository, e.g.

```js
module.exports = {
  repositories: [
    {
      repository: 'abc/def',
      variables: {
        SOME_VARIABLE: 'abc123',
      },
    },
  ],
};
```

It could then be used in a repository config or preset like so:

```json
{
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "addLabels": ["{{ variables.SOME_VARIABLE }}"]
    }
  ]
}
```

## writeDiscoveredRepos

By default, Renovate processes each repository that it finds.
You can use this optional parameter so Renovate writes the discovered repositories to a JSON file and exits.

Known use cases consist, among other things, of horizontal scaling setups.
See [Scaling Renovate Bot on self-hosted GitLab](https://github.com/renovatebot/renovate/discussions/13172).

Usage: `renovate --write-discovered-repos=/tmp/renovate-repos.json`

```json
["myOrg/myRepo", "myOrg/anotherRepo"]
```
