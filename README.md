# Knative Service Deploy

[![CI Checks](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/ci.yml/badge.svg)](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/ci.yml)
[![Deploy Knative service](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/example.yml/badge.svg)](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/example.yml)
[![Link checker](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/link_check.yml/badge.svg)](https://github.com/redhat-actions/kn-service-deploy/actions/workflows/link_check.yml)
<br></br>
[![tag badge](https://img.shields.io/github/v/tag/redhat-actions/kn-service-deploy)](https://github.com/redhat-actions/kn-service-deploy/tags)
[![license badge](https://img.shields.io/github/license/redhat-actions/kn-service-deploy)](./LICENSE)

**kn-service-deploy** is a GitHub Action to manage [Knative Services](https://kn.dev) on a Kubernetes cluster, using the [Knative Client](https://github.com/knative/client).

## Prerequisites

A Kubernetes Cluster with [Knative installed](https://knative.dev/docs/install/). To try an OpenShift cluster, visit [try.openshift.com](https://try.openshift.com) or sign up for our [Developer Sandbox](https://developers.redhat.com/developer-sandbox).

## Action Inputs

| Input | Description | Default |
| ----- | ----------- | ------- |
| service_name | The name of the Knative service to create or manage. | **Required** |
| command | The command to pass to `kn`. Commands are `create`, `update`, `apply`, and `delete`. | create |
| container_image | The container image to deploy. Required unless command is `delete`. | None |
| force_create | If command is `create`, append the `--force` argument. | false |
| kn_extra_args | Any extra arguments to append to the `kn <command>` call. Refer to [Passing extra service arguments](#passing-extra-service-arguments). | None |
| namespace | Kubernetes namespace to target. | Current context |
| registry_user | The user to authenticate as to pull the `container_image`. Required if the image is private. | None
| registry_pasword | The `registry_user`'s password or token. Required if the image is private.| None |

**Note:**
When a username and a token or password are provided to pull the image, the action will create a Kubernetes `docker-registry` secret with the provided credentials. The secret's `docker-server` value will be the domain of the `container_image` input.

## Action Outputs

| Output | Description |
| ------ | ----------- |
| service_url | URL to the Knative service that was created or updated. Empty if `command` input is `delete`. |

## Example

The example below shows how the `kn-service-deploy` action can be used to deploy a Knative service to OpenShift.

You must be logged into your Kubernetes cluster before running this action. If you are using OpenShift, use [oc-login](https://github.com/redhat-actions/oc-login).

```yaml
# Login into the Openshift cluster
# with your username and password/token
- name: Authenticate and set context
  id: oc_login
  uses: redhat-actions/oc-login@v1
  with:
    openshift_server_url: ${{ secrets.OPENSHIFT_SERVER }}
    openshift_token: ${{ secrets.OPENSHIFT_TOKEN }}
    insecure_skip_tls_verify: true
    namespace: ${{ env.NAMESPACE }}

# Deploy knative service using container image
- name: Knative Service Deploy
  id: kn_service_deploy
  uses: redhat-actions/kn-service-deploy@v1
  with:
    service_name: getting-started-knative
    container_image: ${{ env.IMAGE_NAME }}
```

For a complete example see [the example workflow](./.github/workflows/example.yml).

## Passing extra service arguments

This action provides basic options such as namespace, service name, image and the command to run. There might be cases where you might want to pass extra arguments to the `kn service <command>`. In those cases you can use `kn_extra_args`.

For example, if you want to add `--min-scale=1` and`--max-scale=5`, then your action snippet will be:

```yaml
- name: Knative Service Deploy
  id: kn_service_deploy
  uses: redhat-actions/kn-service-deploy@v1
  with:
    service_name: getting-started-knative
    container_image: "${{ steps.push-to-quay.outputs.registry-path }}"
    kn_extra_args: >
      --min-scale=1
      --max-scale=5
```
