# set-image

## Overview

<!--mdtogo:Short-->

The `set-image` function sets the image for all instances of a provided image
`name` with the provided image `newName:newTag` or `newName@digest`. A common
deployment pattern is to update images to the latest tags or, ideally, digests.
Sometimes people want to change the image repo, also, such as for dev builds.

<!--mdtogo-->

You can learn more about images [here][image].

<!--mdtogo:Long-->

## Usage

This function can be used with any KRM function orchestrators (e.g. kpt).

For all resources which specify an image of a given `name`, the `set-image`
function sets that image field to the specified `newName:newTag` or
`newName@digest`.

By default the function updates the standard [fields][commonimage] used to
specify an image.

This function can be used both declaratively and imperatively.

### FunctionConfig

There are 2 kinds of `functionConfig` supported by this function:

- `ConfigMap`
- A custom resource of kind `SetImage`

To use a `ConfigMap` as the `functionConfig`, the desired image spec must be
specified using the following fields:
- `data.name`: Original name of image to replace. Should not contain the image
tag or digest. This field is required.
- `data.newName`: New name to set for images matching `data.name`.
Will not change name if omitted.
- `data.newTag`: New tag to set for images matching `data.name`.
Will not change tag/digest if omitted.
- `data.digest`: New digest to set for images matching `data.name`.
Will not change tag/digest if omitted.

The function will return an error for the following scenarios:
- `name` is omitted
- `newName`, `newTag`, and `digest` are all omitted
- `newTag` and `digest` are both provided

To set the image `nginx` to `bitnami/nginx:1.21.4` for all resources, we use the
following `functionConfig`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  name: nginx
  newName: bitnami/nginx
  newTag: 1.21.4
```

To set an image `nginx` to
`nginx@sha256:3cbbd11b65aab276c8578c039d0c21d0ffb7a496e09c0f632bac1a1b2c115256`
for all resources, we use the following `functionConfig`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  name: nginx
  newName: nginx
  digest: sha256:3cbbd11b65aab276c8578c039d0c21d0ffb7a496e09c0f632bac1a1b2c115256
```

To use a `SetImage` custom resource as the `functionConfig`, the desired
image specification must be specified in the `image` field. Sometimes you have
resources (especially custom resources) that have image fields in fields
other than the [defaults][commonimage], you can specify such label fields
using `additionalImageFields`. It will be used jointly with the
[defaults][commonimage].

`additionalImageFields` has following fields:

- `group`: Select the resources by API version group. Will select all groups if
  omitted.
- `version`: Select the resources by API version. Will select all versions if
  omitted.
- `kind`: Select the resources by resource kind. Will select all kinds if
  omitted.
- `path`: Specify the path to the field that the value needs to be updated. This
  field is required.
- `create`: If it's set to true, the field specified will be created if it
  doesn't exist. Otherwise, the function will only update the existing field.

To set image `nginx` to `bitnami/nginx:1.21.4` for all built-in resources and
the path `spec/manifest/images[]/image` in `MyKind` resource, we use the
following `functionConfig`. Note that `images[]` indicates a nested list.

```yaml
apiVersion: fn.kpt.dev/v1alpha1
kind: SetImage
metadata:
  name: my-func-config
image:
  name: nginx
  newName: bitnami/nginx
  newTag: 1.21.4
additionalImageFields:
- kind: MyKind
  create: false
  group: dev.example.com
  path: spec/manifest/images[]/image
  version: v1
```

<!--mdtogo-->

[image]: https://kubernetes.io/docs/concepts/containers/images/

[commonimage]: https://github.com/kubernetes-sigs/kustomize/blob/master/api/konfig/builtinpluginconsts/images.go#L7
