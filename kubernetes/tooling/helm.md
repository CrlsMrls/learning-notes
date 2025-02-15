# Helm

Helm is a Kubernetes package manager currently maintained by the Cloud Native Computing Foundation (CNCF), which is a cloud industry consortium composed of Google, Microsoft, and Bitnami amongst other companies. Helm has become quite popular with cloud developers largely because it simplifies Kubernetes application management, the roll out of updates, and options to share applications.

Helm is a tool that streamlines Kubernetes application installation and management. You can think of it like apt, yum, or homebrew for Kubernetes. Using helm charts is recommended, since they are maintained and typically kept up-to-date by the Kubernetes community. There are two parts to Helm: The client (**Helm**) and the server (**Tiller**).

- **Tiller** runs inside your Kubernetes cluster and manages releases (installations) of your Helm Charts.
- **Helm** runs on your laptop, CI/CD, or in this case, the Cloud Shell.

## Installation

Fetch the script by running the following command:

```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
```

Next, run the following commands to get the Helm client installed:

```
chmod 700 get_helm.sh
./get_helm.sh
```

Now initialize helm:

```
helm init
```
