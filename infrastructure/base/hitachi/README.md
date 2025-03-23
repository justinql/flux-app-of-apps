https://www.hitachivantara.com/en-us/pdf/architecture-guide/hybrid-cloud-with-google-anthos-on-ucp-platform.pdf
Page 49

https://github.com/hitachi-vantara/csi-operator-hitachi/tree/main/hspc/v3.15.0

# Set up storage using Hitachi Storage Plug-in for Containers

Hitachi Storage Plug-in for Containers can be easily deployed to GKE on Bare Metal using
the following steps:
* Install Hitachi Storage Plug-in for Containers
* Configure Secret settings to access Hitachi VSP Storage system
* Configure StorageClass settings
* Configure Multipathing (Fibre Channel or iSCSI)
> Note: If there is a previous version of Storage Plug-in for Containers, remove it before performing the installation procedure.
# Installing Storage Plug-in for Containers on GKE on Bare Metal

To install Hitachi Storage Plug-in for Containers using the Operator, perform the steps
described in the “Installation on Kubernetes” section on https://
knowledge.hitachivantara.com/Documents/Adapters_and_Drivers/
Storage_Adapters_and_Drivers/Containers/Storage_Plug-in_for_Containers. Here some of
these steps:
## Procedure
1. Download the Storage Plug-in for Containers package from Hitachi Support Connect.
2. Copy and extract the Storage Plug-in for Containers package to the GKE on Bare Metal
admin workstation and move it to the directory yaml/operator.
3. Create the namespace for the Operator.
  ```
  kubectl create -f hspc-operator-namespace.yaml
  ```
4. Create a secret to get the container images from Red Hat registry.
  ```
  kubectl create secret docker-registry regcred-redhat-com \
  --namespace=hspc-operator-system \
  --docker-server=registry.connect.redhat.com \
  --docker-username=<user> \
  --docker-password=<password>
  ```
5. Create the Operator and verify that it is running:
  ```
  kubectl get deployment -n hspc-operator-system
  NAME READY UP-TO-DATE AVAILABLE AGE
  hspc-operator-controller-manager 1/1 1 1 24s
  ```
6. If deploying on the same namespace as the Operator, you need to create one more secret to registry.redhat.io. If deploying in a separate namespace, create two secrets on the previous URL. For this validation we installed on the same namespace.
  ```
  kubectl create secret docker-registry regcred-redhat-io \
  --namespace=${SPC_NAMESPACE} \
  --docker-server=registry.redhat.io \
  --docker-username=<user> \
  --docker-password=<password>
  ```
7. Verify that hspc_v1_hspc.yaml has the correct namespace, and proceed to deploy Storage Plug-in for Containers with the following command:
  ```
  kubectl create -f hspc_v1_hspc.yaml
  ```
8. Use the following command verify that all the PODs are in the running state:
   ```
   [root@jp-gke-admin-bm operator]# kubectl get pods -n hspc-operator-system
   NAME READY STATUS RESTARTS AGE
   hspc-csi-controller-69c68db7d5-hgd47 6/6 Running 0 2m54s
   hspc-csi-node-7jxgp 2/2 Running 0 2m54s
   hspc-csi-node-bdjnt 2/2 Running 0 2m54s
   hspc-csi-node-lw4mw 2/2 Running 0 2m54s
   hspc-csi-node-stl48 2/2 Running 0 2m54s
   hspc-operator-controller-manager-6b8684c94d-c8sn7 1/1 Running 0 69m
   ```
## Result
 Hitachi Storage Plug-in for Containers has been successfully installed. If you want to make an advanced configuration, see Configuration of Storage Plug-in for Containers.

# Configure secret

The secret contains storage system information that enables access to Storage Plug-in for Containers. It contains the storage URL (VSP REST API), user, and password settings. The following is an example of the YAML manifest file:
```
apiVersion: v1
   kind: Secret
metadata:
 name: hitachi-vsp-secret
 namespace: ucp-anthos
type: Opaque
data:
 url: <BASE64>
 user: <BASE64>
 password: <BASE64>
```

```
kubectl create secret generic secret-hspc --from-literal "url=https://123.123.123.123" --from-literal "user=admin" --from-literal "password=password"
```


# Configure StorageClass
The StorageClass contains storage settings that are necessary for Storage Plug-in for
Containers to work with your environment. The following YAML manifest file provides
information about the required parameters for Storage Plug-in for Containers with Hitachi
VSP storage:
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
 name: hitachi-vsp-sc
 annotations:
 kubernetes.io/description: Hitachi Storage Plug-in for Containers
provisioner: hspc.csi.hitachi.com
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
parameters:
 serialNumber: "715021"
 poolID: "2"
 portID: CL5-A,CL6-A
 connectionType: fc csi.storage.k8s.io/fstype: ext4
 csi.storage.k8s.io/provisioner-secret-namespace: "ucp-anthos"
 csi.storage.k8s.io/provisioner-secret-name: "hitachi-vsp-secret"
 csi.storage.k8s.io/node-stage-secret-name: "hitachi-vsp-secret"
 csi.storage.k8s.io/controller-expand-secret-name: "hitachi-vsp-secret"
 csi.storage.k8s.io/node-publish-secret-namespace: "ucp-anthos"
 csi.storage.k8s.io/controller-publish-secret-name: "hitachi-vsp-secret"
 csi.storage.k8s.io/controller-publish-secret-namespace: "ucp-anthos"
 csi.storage.k8s.io/node-publish-secret-name: "hitachi-vsp-secret"
 csi.storage.k8s.io/controller-expand-secret-namespace: "ucp-anthos"
 csi.storage.k8s.io/node-stage-secret-namespace: "ucp-anthos"
```
Here are additional details about some of these parameters:
* serialNumber: VSP serial number
* provisioner: For Storage Plug-in for Containers, the default is hspc.csi.hitachi.com
* poolID: Pool ID on the VSP used to carve dynamically persistent volumes
* portID: VSP storage ports, use a comma separator for multipath
* connectionType: It is the connection type between storage and nodes. Fibre Channel and iSCSI are supported. If blank, Fibre Channel is set.
* fstype: set filesystem type as ext4
* secret-name: define VSP secret name
* secret-namespace: enter the same namespace used for the secret

One way to create a StorageClass is using the following command:
```
kubectl create -f <storage-class-manifest-file>
```
Use the following command to list the storage classes on GKE on Bare Metal clusters:
```

```

# Configure Multipathing

For data-plane nodes connected to Hitachi VSP storage using Fibre Channel or iSCSI, it is recommended that multipathing is enabled. The requirement is to create the multipath.conf and ensure that the user_friendly_names option is set to yes and the multipathd.service is enabled.

Consider the following before applying the multipath configuration:
* For Fibre Channel, ensure that Fibre Channel switches are configured with proper zoning for the compute nodes and Hitachi VSP storage systems are accessible to each other.
* For iSCSI, ensure the Hitachi VSP storage is properly configured for iSCSI and the compute nodes can access the iSCSI targets. Also, for iSCSI, check the Hitachi Storage Plug-in for Containers Release Notes for additional considerations regarding IQN configurations. 
* RHEL already includes the device-mapper-multipath package which is required to support multipathing. For solutions with iSCSI, RHEL already has the iSCSI initiator tools installed by default. There is no need to install an additional package, just apply the configurations as indicated in this section. 
Here is an example of how to enable multipath on a RHEL node using the mpathconf utility. The following command creates the /etc/multipath.conf file:
```
mpathconf --enable --with_multipathd y
``
The multipath.conf file will look like this:
```
[root@ucpbm-gke-wnode2 ~]# cat /etc/multipath.conf
...
defaults {
 user_friendly_names yes
 find_multipaths yes
 enable_foreign "^$"
}
blacklist_exceptions {
 property "(SCSI_IDENT_|ID_WWN)"
}
blacklist {
}
`
To check status of `multipathd.service`, use the following command:
```
systemctl status multipathd.service
```
If you are using iSCSI, make sure the `iscsid.service` is enabled.

