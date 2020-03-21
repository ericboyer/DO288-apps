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

### cicd 
- First deploy Jenkins to "cicd" namespace and configure Jenkins Openshift Sync plugin to add your app's namespace
- Add "edit" role to user for all namespaces jenkins will be expected to interact with:
'oc policy add-role-to-user edit system:serviceaccount:<jenins-project>:jenkins -n <my-dev-project>'
- Create a pipeline buildconfig that points to the git url and branch (per application) and your Jenkinsfile
- Define Jenkinsfile
- Start the pipeline buildconfig using: 'oc start-build bc/<name>'

### external services
- OpenShift apps can talk using service endpoints. 
- If apps are in the same project, use <app-name>; if in other project use <app-name>.<project> 
- External services (outside OpenShift) can integrated using 'oc create service externalname myexternalservicename --external-name myexternal.service.com'
- OpenShift's internal DNS will add svc.cluster.local suffix 

### nexus
- Source here in this repo, added built image to quay.io/eboyer
- oc new-app --name nexus3 -f ../labs/nexus-service/nexus-template.yaml -p HOSTNAME=nexus-rhn-gps-eboyer.apps.ocp-na2.prod.nextcle.com

### Summary external services
- A service name becomes a local DNS host name for all pods inside an OpenShift cluster.
- An external service is created with the oc create service externalname command, using the the external-name option.
- Red Hat recommends that production deployments define health probes.
- Red Hat provides a set of middleware container images to deploy applications in OpenShift, including applications packaged as fat JAR files.
- The Fabric8 Maven plug-in provides features to generate OpenShift resources and trigger OpenShift processes, such as builds and deployments.

### review dockerfile
- Update dockerfile to run on OpenShift (optimize layers, anonymous user, etc.)
- Build image and push to external registry
- Create "common" project in openshift and create an imagestream pointing to your image in the external registry (import-image)
- Create "app" project, and use 'oc policy add-role-to-group -n rhn-gps-eboyer-review-common system:image-puller system:serviceaccounts:rhn-gps-eboyer-review-dockerfile' to grant service accounts from the "app" project to access image streams from your "common" project 
- From the "app" project, use oc new-app -i to point to the image stream created in your "common" project

## Cool troubleshooting techniques
- Ping from api pod to backend database running on port 3306: 
| `oc rsh quotesapi-1-r6f31 bash -c 'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'`


## Course in review
### Chapter 1, Deploying and Managing Applications on an OpenShift Cluster
- Deploy applications using various application packaging methods to an OpenShift cluster and manage their resources.
- Describe the architecture and new features in OpenShift 4.
- Deploy an application to the cluster from a Dockerfile with the CLI.
- Deploy an application from a container image and manage its resources using the web console.
- Deploy an application from source code and manage its resources using the command-line interface.

### Chapter 2, Designing Containerized Applications for OpenShift
- Select an application containerization method for an application and package it to run on an OpenShift cluster.
- Select an appropriate application containerization method.
- Build a container image with advanced Dockerfile directives.
- Select a method for injecting configuration data into an application and create the necessary resources to do so.

### Chapter 3, Publishing Enterprise Container Images
- Interact with an enterprise registry and publish container images to it.
- Manage container images in registries using Linux container tools.
- Access the OpenShift internal registry using Linux container tools.
- Create image streams for container images in external registries.

### Chapter 4, Building Applications
- Describe the OpenShift build process, trigger and manage builds.
- Describe the OpenShift build process.
- Manage application builds using the BuildConfig resource and CLI commands.
- Trigger the build process with supported methods.
- Process post build logic with a post-commit build hook.

### Chapter 5, Customizing Source-to-Image Builds
- Customize an existing S2I builder image and create a new one.
- Describe the required and optional steps in the Source-to-Image build process.
- Customize an existing S2I builder image with scripts.
- Create a new S2I builder image with S2I tools.

### Chapter 6, Creating Applications from OpenShift Templates
- Describe the elements of a template and create a multicontainer application template.
- Describe the elements of an OpenShift template.
- Build a multicontainer application from a custom template.

### Chapter 7, Managing Application Deployments
- Monitor application health and implement various deployment methods for cloud-native applications.
- Implement liveness and readiness probes.
- Select the appropriate deployment strategy for a cloud-native application.
- Manage the deployment of an application with CLI commands.

### Chapter 8, Implementing Continuous Integration and Continuous Deployment Pipelines in OpenShift
- Create and deploy Jenkins pipelines to facilitate continuous integration and deployment with OpenShift.
- The objectives for this chapter are not included in the Comprehensive Review lab.

### Chapter 9, Building Applications for OpenShift
- Create and deploy applications on OpenShift.
- Integrate a containerized application with non-containerized services.
- Deploy containerized third-party applications following recommended practices for OpenShift.
- Use a Red Hat OpenShift Application Runtime to deploy an application.
