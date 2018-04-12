# jupyterhub-on-nautilus

Sample JupyterHub config to run z2jh on [nautilus.optiputer.net](https://nautilus.optiputer.net/)

## Set up

### Create a namespace for yourself

1. Create a namespace using the Nautilus portal
2. Download `kubeconfig` file from the Nautilus portal

### Set up helm in your namespace

1. Download and install `helm` locally:
   
   ```bash
   curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
   ```
   
   You can use [other installation methods](https://github.com/kubernetes/helm/blob/master/docs/install.md)
   if curling into bash bothers you.

2. Create a `ServiceAccount` for `helm`'s `tiller` component to use.

   ```bash
   kubectl --namespace=<your-namespace> create sa tiller
   ```

3. Create a `RoleBinding` to give the `tiller` `ServiceAccount` enough
   permissions.

   ```bash
   kubectl --namespace=<your-namespace> apply -f tiller-rolebinding.yaml
   ```

4. Initialize `helm`

   ```bash
   helm --tiller-namespace=<your-namespace> init --service-account tiller
   ```

5. Wait for a few seconds, and test if `helm` is working.

   ```bash
   helm --tiller-namespace=<your-namespace> list
   ```

   This should return an empty response, rather than an error.

6. Secure your `helm` to make it inaccessible to other users in the cluster.
  
   ```bash
   kubectl --namespace=<your-namespace> \
           patch deployment tiller-deploy \
           --type=json \
           --patch='[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/tiller", "--listen=localhost:44134"]}]'
    ```

Congratulations, you now have a working `helm` on nautilus!

### Install JupyterHub
   
More information on configuring JupyterHub with z2jh can be found at https://z2jh.jupyter.org/

This repo contains a minimal JupyterHub with the following properties:

1. No pre-pulling of user images for faster startup
2. 1Gi of persistent storage per user
3. Ingress into the hub so users can use it

It is set up using a deployment chart, to make it easy to customize it with additional
helm charts.

1. Add the Jupyterhub helm repository.

   ```bash
   helm --tiller-namespace=<your-namespace> repo add jupyterhub https://jupyterhub.github.io/helm-chart/
   helm --tiller-namespace=<your-namespace> repo update
   ```

   Note that you will need to add `--tiller-namespace=<your-namespace>` to all helm commands

2. Fetch the version of JupyterHub chart mentioned in `requirements.yaml`.

   ```bash
   cd nautilus-hub
   helm --tiller-namespace=<your-namespace> dep up
   cd ..
   ```

3. Install the hub!

   ```bash
   helm --tiller-namespace=<your-namespace> upgrade --install \
        --namespace=<your-namespace>
        nautilus-hub
        nautilus-hub
    ```

    You will need to modify `nautilus-hub/values.yaml` to set the hostname
    to something other than the current default.

4. Wait for the pods to be ready, and enjoy your JupyterHub!