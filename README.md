# k8s-github-action-runner

Below, you can see that I have set the resource limits for runners to 7 CPUs and 14 Gi of RAM. An instance with those specs does not exist in our node groups, so Karpenter will see that and create the instance for the Actions to run on. This saves you on costs, and it only takes about two minutes for the instance to come up and run your job.

Set your GitHub URL and the PAT token, then run the Helm commands.
```bash
helm install arc \
    --namespace "arc-systems" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```    

```bash
helm install "arc-runner-set" \
    --namespace "arc-runners" \
    --create-namespace \
    --set githubConfigUrl="https://github.com/yourusername/yourreponame" \
    --set githubConfigSecret.github_token="ghp_somepattokenhere342342234" \
    --set "template.spec.containers[0].resources.requests.cpu=7" \
    --set "template.spec.containers[0].resources.requests.memory=14Gi" \
    --set "template.spec.containers[0].resources.limits.cpu=7" \
    --set "template.spec.containers[0].resources.limits.memory=14Gi" \
    --set "template.spec.containers[0].name=runner" \
    --set "template.spec.containers[0].image=ghcr.io/actions/actions-runner:latest" \
    --set "template.spec.containers[0].command[0]=/home/runner/run.sh \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set

```
    

We can now create our GitHub Actions Workflow. If you have an existing workflow, then go ahead. If not, use the simple one below and commit to the main branch of your repo you put in the githubConfigUrl.

name: CI
on:
  workflow_dispatch:
jobs:
  build:
    runs-on: arc-runner-set
    steps:
      - uses: actions/checkout@v4
      - name: Run a one-line script
        run: echo Hello, world!

      - name: Run a multi-line script
        run: |
          cat README.md
         
