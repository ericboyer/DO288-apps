# DO288 Containerized Example Applications

This repository contains a collection of sample containerized applications.  To complete the course you need to fork this repo into your personal Github account.

## Notes
### app-config
`oc create configmap myappconf --from-literal APP_MSG="Test Message"`
`oc set env dc/myapp --from configmap/myappconf`
`curl myapp-rhn-gps-eboyer-app-config.apps.ocp-na2.prod.nextcle.com`
`oc set volume dc/myapp --add -t secret -m /opt/app-root/secure --name myappsec-vol --secret-name myappfilesec`

### external-registry
`sudo podman login -u eboyer quay.io`
`oc new-project rhn-gps-eboyer-external-registry
`oc new-app --name sleep --docker-image quay.io/eboyer/ubi-sleep:1.0`
`oc create secret generic quayio --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type kubernetes.io/dockerconfigjson`
`oc secrets link default quayio --for pull`

### access internal registry
oc get route -n openshift-image-registry
curl -X GET default-route-openshift-image-registry.apps.ocp-na2.prod.nextcle.com/v2/_catalog 
INTERNAL_REGISTRY=$(oc get route default-route -n openshift-image-registry -o jsonpath='{.spec.host}')
echo $INTERNAL_REGISTRY 
oc new-project rhn-gps-eboyer-common
TOKEN=$(oc whoami -t)
skopeo copy --dest-creds=rhn-gps-eboyer:${TOKEN} oci:/home/student/DO288/labs/expose-registry/ubi-info docker://${INTERNAL_REGISTRY}/rhn-gps-eboyer-common/ubi-info:1.0
oc get is
sudo podman login -u rhn-gps-eboyer -p $TOKEN $INTERNAL_REGISTRY
sudo podman pull $INTERNAL_REGISTRY/rhn-gps-eboyer-common/ubi-info:1.0
sudo podman images
sudo podman run --name info $INTERNAL_REGISTRY/rhn-gps-eboyer-common/ubi-info:1.0
oc delete is ubi-info
oc delete project rhn-gps-eboyer-common

### image streams
- To build and deploy applications using an image stream that is defined in another project:
podman login -u myuser registry.example.com
oc project shared
oc create secret generic regtoken --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type kubernetes.io/dockerconfigjson
oc import-image myis --confirm --reference-policy local --from registry.example.com/myorg/myimage
oc policy add-role-to-group system:image-puller system:serviceaccounts:myapp
oc project myapp
oc new-app -i shared/myis

### expose images
- In this section we copied an image from disk to our personal account on quay.io, then created an openshift project which imported the image. The import-image action created an imageStream that would then be referenced by a second project created to run the application. Key takeaways 1) secret needs to be created for openshift to access the external image and 2) the image-puller role needs to be assigned to the project when referencing resources in another project (see grant-puller-role.sh for details).

### templates
- You can create manually, concatenate using new-app, or export existing resources using `oc get -o yaml --export <resources>`, note order of resources is important if exporting.
'oc get -o yaml --export is,secret,bc,dc,svc,route > mytemplate.yaml'
Example of creating apps from templates:
- oc new-app --file mytemplate.yaml -p PARAM1=value1 -p PARAM2=value2
- oc process -f mytemplate.yaml -p PARAM1=value1 -p PARAM2=value2 > myresourcelist.yaml | oc create -f -
- Red Hat recommends using the new-app options and only using 'oc process -f my.yaml --parameters' to list parameters

### simple pipeline
- MEMORY_LIMIT parameter increases the amount of memory allocated to the jenkins container to improve performance. The JVM max heap size is set to 50% of the value of MEMORY_LIMIT by default. The default value is 1GB.
## Cool troubleshooting techniques
- Ping from api pod to backend database running on port 3306: 
| `oc rsh quotesapi-1-r6f31 bash -c 'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'`
