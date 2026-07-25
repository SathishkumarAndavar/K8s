# CKA Practice Repositories and Labs

These are supplementary study resources. Treat this repository as your primary notes; verify commands against the Kubernetes version in your lab because community repositories can age quickly.

## Official resources

- [Linux Foundation CKA program](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) — current exam details, curriculum links, and the official simulator included with an exam enrollment.
- [Kubernetes documentation](https://kubernetes.io/docs/) — the best source to verify API versions, feature status, and `kubectl` behavior.

## Community repositories

- [KodeKloud CKA course notes](https://github.com/kodekloudhub/certified-kubernetes-administrator-course) — broad topic notes and practice-test material.
- [techiescamp CKA certification guide](https://github.com/techiescamp/cka-certification-guide) — hands-on learning path and scenario practice.
- [aeciopires/learning-cka](https://github.com/aeciopires/learning-cka) — exercises, simulator links, and a Kubernetes-versioned study path.
- [sv222 CKA exam preparation](https://github.com/sv222/certified-kubernetes-administrator-CKA-exam-preparation) — question-and-answer style review.
- [theplatformlab CKA guide](https://github.com/techwithmohamed/CKA-Certified-Kubernetes-Administrator) — recent hands-on exercises, templates, and mock-exam structure.

## Safe use rules

- Do not treat any community question list as an official or leaked exam bank.
- Run destructive exercises only in disposable clusters.
- Prefer current upstream documentation over old examples, especially for CRI, CSI, Ingress/Gateway API, and autoscaling.
- For every lab, finish with a validation command and clean up the created resources.
