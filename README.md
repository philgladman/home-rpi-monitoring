# home-rpi-monitoring
How to configure and install Grafana, Loki, and Promtail on a Raspberry Pi K3s Cluster

## Prereq Steps
### Raspberry Pi Setup
- Please follow this link for instructions on [How to install and setup K3s Cluster on raspberry pi](https://github.com/philgladman/home-rpi-k3s-cluster.git)
- This link will show you how to;
  - Prepare your raspberry pi for kubernetes
  - Spin up K3s
  - Install MetalLb
  - Install Nginx Ingress
- If you already have a Raspberry pi configured with K3s, MetalLb, and Nginx ingress, please move on to [Step 1.)](README.md#step-1---configure-monitoring-stack)

## Step 1.) - Configure monitoring stack
- Make applicable changes to the `kustomize/monitoring/values.yaml` file.
- The current configuration has Grafana, Loki, and Promtail enabled.
- This also has Loki configured as a Data Source in Grafana
- To enable or disable any services, simply set `enabled: ` to `true` or `false` in the values.yaml file.

## Step 2.) - Create Deployment with Helm Templating
- Install Helm `curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash`
- __OPTIONAL__ - update monitoring stack dependencies. From the `home-rpi-monitoring/charts/monitoring`, run `helm dependency build`
- from the `home-rpi-monitoring` directory, run `helm template loki-stack charts/monitoring -f kustomize/monitoring/values.yaml --include-crds --debug > kustomize/monitoring/release.yaml`

## Step 3.) - Deploy monitoring stack
- Depoly with `kubectl apply -k kustomize/.`
- Wait for all pods to spin up
- Get Grafana password `kubectl get secret --namespace monitoring loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo`
- Port forward the grafana svc to 3000
- Go to `localhost:3000` and login with user=`admin` and the password from above

## Step 3.) - Configure ingress
- If you have pihole running, you can configure a custom Local DNS name for the Grafana UI
- To install pihole, follow the instructions [here](https://github.com/philgladman/home-rpi-pihole.git).
- Login to pihole ui.
- On the left bar, click on `Local DNS`, and then click on `DNS Records`.
- In the `Domain:` field, type in your custom DNS name for grafana (ex = `grafana.phils-home.com`)
- For the `IP Address:` field, type in the output to the following command `kubectl get svc -n nginx-ingress nginx-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[].ip}'`, this should be the ip address of your nginx ingress.
- Click `add`
- Edit the grafana ingress file to have your custom DNS name `sed -i "s|grafana.example.com|<your-local-dns-name-for-grafana>|g" kustomize/monitoring/ingress.yaml`
- Apply the ingress file `kubectl apply -f kustomize/monitoring/ingress.yaml`
- Now you can access the Grafana UI from this custom DNS name, assuming you are using your pihole as your DNS Server.
