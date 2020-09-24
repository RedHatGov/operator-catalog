# RedHatGov Operator Catalog

This is a very minimal repository with very simple code and little to no automated testing. This is by design, as this project requires an OLM-enabled cluster to run and contains a lot of different tools. It may be part of an E2E testing flow elsewhere that provisions full clusters, but for the purposes of versioning our catalog index, strict versioning and manual checks will suffice.

## Usage

This repository exists to serve as the source of truth for the contents of the RedHatGov Operator Framework Catalog index, accessible [here](https://quay.io/redhatgov/operator-catalog). As such, it has some simple automation to download [opm](https://github.com/operator-framework/operator-registry) and package a catalog index from a small configuration file that defines the images that should be in the catalog.

In short, you don't use this repository. Travis does. If you want to add this catalog index to your OLM-enabled cluster, you could install the catalog source using the following:

```shell
curl https://raw.githubusercontent.com/redhatgov/operator-catalog/main/default/catalog_source.yml | kubectl apply -f -
```

If you want to add the development version of this catalog index to your OLM-enabled cluster, you could use kustomize like the following:

```shell
kustomize build github.com/redhatgov/operator-catalog?depth=1&ref=develop//default | kubectl apply -f -
```

### Contributing

If you have an operator you would like included in the catalog index, the steps are something like this:

1. Follow the [Operator SDK documentation](https://sdk.operatorframework.io/docs/olm-integration/generation/#generate-your-first-release) for generating a release of your operator based on the Operator SDK version 1.0.0 or higher.
1. Publish your release bundle image somewhere public. If you're in Red Hat's North American Public Sector organization, you should have or can get access to the quay.io/redhatgov project which is where we try to keep our operators and bundles.
1. Fork this repository from the GitHub UI to your own GitHub account and clone it down locally.
1. Create a new local branch with a moderately descriptive name like `add_gitea` to add Gitea to the index (it's already here) with something like:

    ```shell
    git checkout -b add_gitea
    ```

1. Edit operator-index.yml to add your bundle image. If you track the latest tag, ensure you're updating that tag regularly so that when the index is rebuilt it pulls in your changes. If you prefer to pin versions, prepare to maintain your fork of this repo. **NOTE**: If your bundle, operator, or application images aren't maintained, they will probably be cut.
1. Commit the operator-index.yml changes to add your bundle to your local branch in its own commit with something like:

    ```shell
    git add operator-index.yml
    git commit -m 'Added my Gitea operator to the index.'
    ```

1. Push your updates to a new branch on your fork with something like:

    ```shell
    git push -u origin add_gitea
    ```

1. Create a Pull Request against the main repository's `develop` branch from your new branch. In your pull request, be relatively descriptive about what you've added. Some links to the code you used to build your operator would be nice for historical and auditing reasons, for example.

### Releasing

For maintainers of this index, the release process should be relatively controlled. Although this is not an engineering effort and these operators are not officially supported Red Hat products, we should strive to maintain a quality catalog for our workshops and demos that use this content, and be mindful of the fact that this will probably end up in production somewhere anyways...

This is the current expectation of release flow:

- The `latest` tag from Quay should generally be the one that is used as a `CatalogSource` in clusters in order to enable updates to the operator images on live clusters.
- Pinning to a version tag for a deployment into a cluster that we can babysit (workshops, etc.) is not a bad idea, but we should expect to be able to track `latest` and only pin to a version tag if necessary.
- No `$VERSION` or `latest` tags should be applied to images built from the `develop` branch. Travis is configured to push anything from the `develop` branch to the `$VERSION-dev` and `develop` tags on Quay.
- Bundles get added in `develop` and the operators they install should be tested on an OpenShift cluster, manually for the time being, from the `develop` tag on Quay.
- When the decision is made to release an update to the index, the version should be set appropriately in `operator-index.yml`:
    1. Edit operator-index.yml to update the version of the index. In general, we should follow the [semantic versioning standard](https://semver.org/) by implementing the following conventions:
        1. If we remove a bundle from the index, we should increase the major version, as this is a breaking change.
        1. If we add a bundle to the index, we should increase the minor version, as this is a backwards-compatible change.
        1. If we update a bundle's tracking tag, or perform some other small bugfix, we should increase the patch version, as this is likely a minor bugfix/enhancement. If an operator has a pinned version change that breaks its own API (for example, moving from major version `1` to `2`), we should consider making a major release ourselves.
    1. Commit the version bump to a commit on the `develop` branch.
    1. Perform a final test from the `develop` tag on Quay.
    1. Open a Pull Request from the `develop` branch to the `main` branch that includes the list of changes and a description of the testing performed.
- After a review of the PR against `main`, if it is merged, the `main` branch should be tagged with a _signed_ git tag marking the version number. This provides an easy way to correlate image tags on Quay with GitHub releases.
- Travis will automatically push a new `latest` and `$VERSION` tag to Quay from the `main` branch.

## Contributors

This is a list of people who have contributed to code used in the operators indexed here. It is not exhaustive. Feel free to update and include yourself in your PR if you add an operator.

- [Wolfgang Kulhanek](https://github.com/wkulhanek)
- [Andrew Block](https://github.com/sabre1041)
- [James Harmison](https://github.com/jharmison-redhat)
- [Andy Krohg](https://github.com/andykrohg)
- [Griffin College](https://github.com/griffincollege)
