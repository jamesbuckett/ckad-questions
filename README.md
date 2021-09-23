# Sample CKAD exercises and solutions

- A set of sample questions and solutions to assist in preparing for the CKAD exam. 
- Assumed practice environment is [Docker Desktop](https://www.docker.com/products/docker-desktop).

## Disclaimer

- Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
- Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only.

## Preparation for Docker Desktop

Please install these two software components, required to answer questions in later sections:

### metrics server

By default the metrics server required for the `kubectl top` command is not present on Docker Desktop.

<details><summary>show</summary>
<p>

Please install the [metrics server](https://github.com/kubernetes-sigs/metrics-server) with the following command:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```bash
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

</p>
</details>

### Contour Ingress

By default the Contour Ingress required for the Ingress Networking question is not present on Docker Desktop.

<details><summary>show</summary>
<p>

Please install the [contour ingress](https://projectcontour.io/) with the following command:

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

</p>
</details>

## Questions by Domain

* [01. Example CKAD Application Design and Build questions with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-workload.md)
* [02. Example CKAD Application Environment, Configuration and Security questions with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-configuration.md)
* [03. Example CKAD Application Deployment with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/04-core-pod.md)
* [04. Example CKAD Services and Networking questions with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-networking-storage.md)
* [05. Example CKAD Observability and Maintenance questions with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/06-ckad-operations.md)
* [06. Example CKAD Miscelanous questions with answers](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-extensions.md)

## Logistics
* The current version of Kubernetes in the CKAD exam can be found [here](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks#what-application-version-is-running-in-the-exam-environment). 
* This has been tested on Docker Desktop v4.0.1.
* To remove all the resources that get created run the commands in the Clean Up section.

*End of Section*