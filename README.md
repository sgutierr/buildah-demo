# buildah-demo
This repo shows how Buildah works using a Tekton pipeline.  
This technical document details the process of building container images and subsequently pushing them to an OpenShift image registry, using tools like Buildah, Kaniko, and Tekton. Each of these tools offers specific advantages that make them suitable for different container build scenarios within an OpenShift environment.  

![Buildah Demo Diagram](docs/buildah-demo-diagram.png)

## Tools

**Check service account permission:**

oc auth can-i create imagestreams --as system:serviceaccount:<NAMESPACE>:builder -n <NAMESPACE>
oc auth can-i create imagestreams --as system:serviceaccount:mytest2:pipeline -n mytest2

oc auth can-i create pipelineruns.tekton.dev --as system:serviceaccount:<NAMESPACE>:builder -n <NAMESPACE>
oc auth can-i create pipelineruns.tekton.dev --as system:serviceaccount:mytest2:builder -n mytest2


**Example of docker registry secret creation:**

oc create secret docker-registry my-pull-secret   --docker-server=image-registry.openshift-image-registry.svc:5000   --docker-username=$(oc whoami)   --docker-password=$(oc whoami -t)   --docker-email=unused@example.com  

**Example of buildah push command with --authfile (for quay):**

buildah --storage-driver=vfs push --debug --tls-verify=false --authfile=/var/tmp/auth.json --digestfile /workspace/source/image-digest localhost/test:1.0 docker://image-registry.openshift-image-registry.svc:5000/mytest2/test:1.0

**Example of curl for testing registry credentials**


curl -k -X POST https://image-registry.openshift-image-registry.svc:5000/v2/buildah-demo/test/blobs/uploads/   -H "Authorization: Bearer sha256~tqz_j3cOr8NPF8JkrfqEV3YbmzC7xU2FId_qIwloVgU"

**Another approach:**

 Assign "privileged" SCC to the pipeline ServiceAccount

## Notes
1. To run the task in OpenShift, the storage driver must be: vfs  
2. The image comes from: 'quay.io/buildah/stable:v1.17.0' which seems outdated.  
3. To view debug logs, add the extra parameter "--debug".  
4. In the push command, I was testing by adding a parameter to reference a dockercfg or dockerconfigjson file with the credentials to authenticate to the image registry. The parameter is: --authfile=/workspace/dockerconfig/auth.json. It doesn't make sense to use this parameter to connect to the internal OpenShift registry.  
5. In case of troubleshooting, you can debug the task in the pod and access the terminal.  
