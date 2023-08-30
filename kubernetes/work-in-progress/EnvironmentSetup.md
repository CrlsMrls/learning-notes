# Environment Setup

## vim

Create a `.vimrc` file and the following lines for working with YAML files:

```
$ cat ~/.vimrc

set number relativenumber
set ai et tb=2 sw=2
```
- `rnu`: lines show relative numbers
- `ai`: autoindent: when go to new line keep same indentation
- `et` or `expandtab`: use spaces for tab
- `tb=2` or `tabstop=2`: amount of spaces used for tab
- `sw=2` or `shiftwidth=2`: amount of spaces used during indentation

## Bash aliases

```bash

cat <<EOF >> .bashrc
# enable autocompletion
source <(kubectl completion bash)

alias k=kubectl
complete -F __start_kubectl k # autocomplete k

alias ka="kubectl apply -f"
complete -F __start_kubectl ka # autocomplete k

# kubectl definition - to get yaml object instead of applying the changes
alias kd="kubectl describe"
complete -F __start_kubectl kd
EOF

source .bashrc

cat <<EOF |  .vimrc
set number relativenumber
set paste
set expandtab
set tabstop=2
set shiftwidth=2
set autoindent
set smartindent
EOF

```

## Status

In a dedicated terminal, show a minimal dashboard with the current status:

```bash
watch -n 0.5 "kubectl config current-context; echo ''; \
  kubectl config view | grep namespace; echo ''; \
  kubectl get namespace,node,ingress,pod,svc,job,cronjob,deployment,rs,pv,pvc,secret,ep -o wide"
```

## Resource short names

Most common list:

| Fullname                 | Shortcut 
| ------------------------ | -------- |
| namespaces               | ns       |
| services                 | svc      |
| deployments              | deploy   |
| replicasets              | rs       |
| pods                     | po       |
| configmaps               | cm       |

Full list (sorted alphabetically):

|Shortcut| Fullname |
| ------ | ------------------------ |
|csr     | certificatesigningrequests |
|cs      | componentstatuses |
|cm      | configmaps |
|ds      | daemonsets |
|deploy  | deployments |
|ep      | endpoints |
|ev      | events |
|hpa     | horizontalpodautoscalers |
|ing     | ingresses |
|limits  | limitranges |
|ns      | namespaces |
|no      | nodes |
|pvc     | persistentvolumeclaims |
|pv      | persistentvolumes |
|po      | pods |
|pdb     | poddisruptionbudgets |
|psp     | podsecuritypolicies |
|rs      | replicasets |
|rc      | replicationcontrollers |
|quota   | resourcequotas |
|sa      | serviceaccounts |
|svc     | services |

## Getting information

```
$ kubectl explain deployment.spec.strategy
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: strategy <Object>

DESCRIPTION:
     The deployment strategy to use to replace existing pods with new ones.

     DeploymentStrategy describes how to replace existing pods with new ones.

FIELDS:
   rollingUpdate        <Object>
     Rolling update config params. Present only if DeploymentStrategyType =
     RollingUpdate.

   type <string>
     Type of deployment. Can be "Recreate" or "RollingUpdate". Default is
     RollingUpdate.

     Possible enum values:
     - `"Recreate"` Kill all existing pods before creating new ones.
     - `"RollingUpdate"` Replace the old ReplicaSets by new one using rolling
     update i.e gradually scale down the old ReplicaSets and scale up the new
     one.
```

Show how it should look in the yaml file with the `--recursive` flag:

```
$ kubectl explain deployment.spec.strategy --recursive
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: strategy <Object>

DESCRIPTION:
     The deployment strategy to use to replace existing pods with new ones.

     DeploymentStrategy describes how to replace existing pods with new ones.

FIELDS:
   rollingUpdate        <Object>
      maxSurge  <string>
      maxUnavailable    <string>
   type <string>
```



# Links 

- https://killercoda.com/killer-shell-ckad - Interactive Scenarios for Kubernetes Application Developers
- https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681
- https://github.com/subodh-dharma/ckad
- https://github.com/dgkanatsios/CKAD-exercises


