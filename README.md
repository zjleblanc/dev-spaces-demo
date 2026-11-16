# Ansible Dev Spaces Demo

Customer-facing demo repository for Ansible development in OpenShift Dev Spaces. The workspace uses the Ansible Automation Platform 2.7 Dev Spaces image and Ansible Development Tools (ADT).

## Prerequisites

### Nested containers

Podman and execution environments need container run capabilities enabled on the cluster:

```yaml
spec:
  devEnvironments:
    disableContainerRunCapabilities: false
```

Patch the `CheCluster` (update the name and namespace to match your cluster):

```shell
export CHECLUSTER_NAME=<insert name>
export CHECLUSTER_NAMESPACE=<insert namespace>

oc patch -n $CHECLUSTER_NAMESPACE checluster $CHECLUSTER_NAME --type='json' \
  -p='[{"op": "replace", "path": "/spec/devEnvironments/disableContainerRunCapabilities", "value": false}]'
```

### Pulling from registry.redhat.io

This workspace pulls `registry.redhat.io/ansible-automation-platform-27/ansible-devspaces-rhel9:latest`. The cluster or the user project must be able to authenticate to [registry.redhat.io](https://registry.redhat.io).

Create a pull secret in the user project and label it so Dev Spaces applies it to workspace pods. See [Create an image pull Secret with oc](https://docs.redhat.com/en/documentation/red_hat_openshift_dev_spaces/3.29/html/user_guide/creating-image-pull-secrets_user-guide):

```shell
oc create secret docker-registry rhcc-pull-secret \
  --docker-server=registry.redhat.io \
  --docker-username=<username> \
  --docker-password=<registry-token>

oc label secret rhcc-pull-secret \
  controller.devfile.io/devworkspace_pullsecret=true \
  controller.devfile.io/watch-secret=true
```

If you cannot reach the Red Hat Container Catalog, point `devfile.yaml` at your organization's internal mirror of the AAP 2.7 Dev Spaces image.

### Create SSH Key Secret for Ansible

```shell
oc create secret generic ansible-ssh-key \
  --from-file=id_rsa={private_key_file} \
  --from-file=id_rsa.pub={public_key_file} \
  -n {namespace}
```

## Launch the workspace

1. Open your organization's OpenShift Dev Spaces dashboard and sign in.
2. Select **Create Workspace**.
3. In **Import from Git**, enter `https://github.com/zjleblanc/dev-spaces-demo`.
4. Click **Create & Open** and wait for the workspace to start.

## Configure ansible-navigator

1. In the workspace terminal, rename the sample settings file:

   ```shell
   cp ansible-navigator.yaml.sample ansible-navigator.yaml
   ```

2. Edit `ansible-navigator.yaml` as needed (for example, uncomment the AAP 2.7 EE image or `mode: stdout` for a live demo).
3. Run `ansible-navigator`.

## Repository rename

Dev Spaces clones this repository to `/projects/dev-spaces-demo`. If you rename the Git repository, update that path in both `devfile.yaml` (`VSCODE_DEFAULT_WORKSPACE`) and `.code-workspace` (`folders[].path` and `folders[].name`).

## Demo content

Playbooks live under `playbooks/`. `ansible-navigator.yaml` points at `inventory/hosts.yml`, which targets localhost so the workspace demos run without extra hosts. Add managed nodes to that inventory when the restart playbook should affect another machine.

```shell
ansible-navigator run playbooks/hello-world.yml
ansible-navigator run playbooks/gather-facts.yml
ansible-navigator run playbooks/restart-service.yml -e service_name=sshd
ansible-navigator run playbooks/ec2-instance.yml \
  -e instance_id=i-0123456789abcdef0 \
  -e instance_action=stop
```

`playbooks/restart-service.yml` requires `service_name`.

`playbooks/ec2-instance.yml` starts, stops, or terminates an existing EC2 instance (`instance_action` is `start`, `stop`, or `terminate`). It runs on localhost and calls the AWS API, so it needs the `amazon.aws` collection and credentials from the environment or the AWS shared credentials file. The AAP 2.7 supported execution environment includes that collection. Uncomment its image in `ansible-navigator.yaml` when the workspace image does not already provide it. `terminate` removes the instance.

## Attribution

Workspace scaffold adapted from [cgruver/ansible-devspaces](https://github.com/cgruver/ansible-devspaces) and the [AAP 2.7 Create and launch an Ansible development workspace](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7/html/getting_started_with_ansible_automation_platform/develop-assembly_workspaces_create_launch) documentation.
