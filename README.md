# Sample CKAD exercises and solutions

- A set of sample questions and solutions to assist in preparing for the [CKAD](https://www.cncf.io/certification/ckad/) exam.
- Assumed practice environment is [Docker Desktop](https://www.docker.com/products/docker-desktop) with a [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install) backend.

![kubernetes-ckad-color-300x294](https://user-images.githubusercontent.com/18049790/135700768-0f5735b3-4681-4abd-9075-ece42f4ef134.png)

## Disclaimer

- Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
- Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only.
<br />

## Questions by Domain

- [Example CKAD Application Design and Build Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md)
- [Example CKAD Application Environment, Configuration and Security Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md)
- [Example CKAD Application Deployment Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-deployment.md)
- [Example CKAD Services and Networking Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md)
- [Example CKAD Observability and Maintenance Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md)
- [Example CKAD Miscellaneous Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/06-ckad-miscellaneous.md)
<br />

## Docker Desktop Setup

### Linux access to Docker desktop

- Windows Subsystem for Linux requires you to install a Linux distribution from the Microsoft Store.
  - Tick the 'Use the WSL 2 based engine' under the 'General' panel
- To mimic the CKAD exam please execute all the commands in this repo from this Linux distribution that you installed.
  - The CKAD exam terminal is Ubuntu based.
- Start the distribution directly by searching for Ubuntu and starting the application
<br />

### Prerequisite software for Docker desktop

Please install these software components, required to answer questions in later sections:

<details class="faq box"><summary>Metrics Server</summary>
<p>

Metrics Server installs into Kubernetes

By default the metrics server required for the `kubectl top` command is not present on Docker Desktop.

Please install the [metrics server](https://github.com/kubernetes-sigs/metrics-server) with the following command:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```bash
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

</p>
</details>

<details class="faq box"><summary>Contour Ingress</summary>
<p>

Contour installs into Kubernetes

![contour](https://user-images.githubusercontent.com/18049790/136644054-a6dc7100-cfd5-499c-9c34-01ec4fffbb01.png)

By default the Contour Ingress required for the Ingress Networking question is not present on Docker Desktop.

Please install the [contour ingress](https://projectcontour.io/) with the following command:

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

</p>
</details>

<details class="faq box"><summary>Calico</summary>
<p>

Calico installs into Kubernetes

![calico](https://user-images.githubusercontent.com/18049790/136644040-3b4cbfe8-3eb6-4c46-b238-adb51c2bb09c.png)

Calico is required for the non native Kubernetes resources lookup question.

```bash
curl https://docs.projectcalico.org/manifests/calico.yaml | kubectl apply -f -
```

</p>
</details>

<details class="faq box"><summary>Helm</summary>
<p>

Helm installs into WSL Linux

Please install the [Helm](https://helm.sh/docs/) with the following command:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

</p>
</details>

<details class="faq box"><summary>kubectl convert</summary>
<p>

kubectl convert  installs into WSL Linux

Please install the [kubectl convert](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#migrate-to-non-deprecated-apis) with the following command:

```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert.sha256"
echo "$(<kubectl-convert.sha256) kubectl-convert" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
kubectl convert --help
```

</p>
</details>
<br />

### Optional software for Docker desktop

<details class="faq box"><summary>Octant</summary>
<p>

Helm installs into Windows 

![octant](https://user-images.githubusercontent.com/18049790/136644006-b0009cea-690c-4303-88a0-e06738fd28de.png)

- Consider installing Octant to visualize Kubernetes Resource Types
- This will help you understand the interaction and relationships between Kubernetes Resource types
- Installation instructions for Octant can be found [here](https://octant.dev/)

</p>
</details>
<br />

## Logistics

- The current version of Kubernetes in the CKAD exam can be found [here](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks#what-application-version-is-running-in-the-exam-environment).
- This has been tested on Docker Desktop v4.1.1 (69879)
- To remove all the resources that get created run the commands in the Clean Up section.

_End of Section_
<br />