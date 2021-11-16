# Install

Based upo MacOS Monterey with Docker Desktop 4.x.

~~~bash
# Install on Docker Desktop
$ kubectl create namespace argo-rollouts
$ kubectl apply -n argo-rollouts \
   -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install the kubectl extension
$ brew install argoproj/tap/kubectl-argo-rollouts

# Install zsh auto-complete (one time)
$ source <(kubectl-argo-rollouts completion zsh)

# ... or for every login (open a new shell to activate)
$ kubectl-argo-rollouts completion zsh > "${fpath[1]}/_kubectl-argo-rollouts"
~~~