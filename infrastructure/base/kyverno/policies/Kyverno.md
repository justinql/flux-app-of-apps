# Kyverno
There are a few sample Kyverno policies provided but Kyverno provides an archive of hundreds of sample policies on their website: https://kyverno.io/policies/ The sample Kyverno policies provided are designed to enforce security best practices for a Kubernetes deployment. These policies help maintain the integrity, compliance, and overall security of the cluster by automating the validation and enforcement of critical security measures, such as image signing, resource constraints, and other best practices.

## Enforcing Image Signing with Cosign:
The first Kyverno policy ensures that all container images deployed in the Kubernetes cluster are signed with a specific public key. This helps protect against supply chain attacks by ensuring that only verified and trusted images are allowed to run. By using Cosign, a tool for signing and verifying container images, the policy mandates that any image pulled into the cluster has been signed with a key that the cluster recognizes as trusted. This policy prevents the accidental or malicious deployment of unsigned or tampered images, enforcing security right from the image supply chain. This practice is considered best practice because it ensures that only verified software runs in production, helping mitigate the risks associated with deploying vulnerable or compromised images.

## Restricting Privileged Containers:
Another common policy restricts the use of privileged containers. Privileged containers have elevated permissions and access to the underlying node, making them a significant security risk if misused or compromised. This Kyverno policy ensures that containers are run with the least privilege necessary, requiring them to specify non-privileged security contexts unless explicitly needed. This policy aligns with the principle of least privilege, which is a cornerstone of secure system design. By enforcing the use of non-privileged containers, it helps reduce the attack surface of the Kubernetes environment.

## Blocking deployments into the Default Namespace:
An important policy for ensuring any deployments are isolated to specific namespaces rather than simply using the default namespace. This helps prevent accidentally deploying elements to the default namespace when the correct namespace is not explicitly specified. By blocking deployment you are alerted to the error and can correct. 

## Disallow Latest Tag:
This policy blocks the :latest tag as it is mutable and can lead to unexpected errors if the image changes. The best practice is to use an immutable tag that maps to a specific version of an application Pod.

## Enforce Read-Only on Root Filesystem:
Pods which are allowed to mount hostPath volumes in read/write mode pose a security risk even if confined to a "safe" file system on the host and may escape those confines. For this reason, we recommend blocking pods with editable file systems as a best practice.

## Restrict Image Registries:
You should only have a single image registry within your infrastructure, but this extra policy can prevent deployment of any unauthorized images should some future change enable access to any other Image Registry. 

## Block Auto-Mount of ServiceAccount credentials 
Kubernetes automatically mounts ServiceAccount credentials in each Pod.The ServiceAccount may be assigned roles allowing Pods to access API resources. Blocking this ability is an extension of the least privilege best practice and should be followed if Pods do not need to speak to the API server to function. 
