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

## Cool troubleshooting techniques
- Ping from api pod to backend database running on port 3306: 
| `oc rsh quotesapi-1-r6f31 bash -c 'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'`
