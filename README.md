# Sample CKAD exercises and solutions

- A set of sample questions and solutions to assist in preparing for the CKAD exam. 
- Assumed practice environment is [Docker Desktop](https://www.docker.com/products/docker-desktop).

## Disclaimer

- Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
- Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only.

## Preparation for Docker Desktop

Please install these two software components, required to answer questions in later sections:
* [metrics server](https://github.com/kubernetes-sigs/metrics-server)
<details><summary>show</summary>
<p>

### Metrics Server

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
* [contour ingress](https://projectcontour.io/)
<details><summary>show</summary>
<p>

### Contour Ingress

By default the Contour Ingress required for the Ingress Networking question is not present on Docker Desktop.

Please install the [contour ingress](https://projectcontour.io/) with the following command:

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

</p>
</details>

## Questions by Domain

* [01. Example CKAD Application Design and Build Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/01-ckad-design-build.md)
* [02. Example CKAD Application Environment, Configuration and Security Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/02-ckad-env-configuration-security.md)
* [03. Example CKAD Application Deployment Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/03-ckad-networking-storage.md)
* [04. Example CKAD Services and Networking Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/04-ckad-services-networking.md)
* [05. Example CKAD Observability and Maintenance Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-observability-maintenance.md)
* [06. Example CKAD Miscellaneous Q&A](https://github.com/jamesbuckett/ckad-questions/blob/main/05-ckad-extensions.md)

## Logistics
* The current version of Kubernetes in the CKAD exam can be found [here](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks#what-application-version-is-running-in-the-exam-environment). 
* This has been tested on Docker Desktop v4.0.1.
* To remove all the resources that get created run the commands in the Clean Up section.

*End of Section*