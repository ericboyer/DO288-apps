Instructions

All of the review labs for this course are based on the same use case; that is, a multicontainer version of the To Do List application. When you have finished all the review labs, the application should be deployed as three pods:

A single-page web front-end pod, based on Nginx.

An HTTP API back-end pod, based on Node.js.

A database pod, based on MySQL.

The second review lab builds the To Do List back end using the OpenShift Source-to-Image (S2I) process. The back-end development team is starting to use OpenShift, and your job is to make sure that the build process meets your organization's internal standards. You must also ensure that the deployment follows Red Hat's recommendations for externalizing the configuration of applications deployed to OpenShift.

You must deploy the To Do List back end according to the following specifications:

Deploy a MySQL server on the youruser-review-service project, using tododb as the name of your OpenShift resources and the rhscl/mysql-57-rhel7 container image from the public Red Hat registry.

|'oc new-project rhn-gps-eboyer-review-service'
|'oc new-app --name tododb --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7 \
|     -e MYSQL_USER=todoapp \
|     -e MYSQL_PASSWORD=mypass \
|     -e MYSQL_DATABASE=todo' 

Store database access credentials in a secret called tododb with keys: user and password. Use this secret to initialize environment variables for both the database and the back end pods.

|'oc create secret generic tododb --from-literal user=todoapp --from-literal password=mypass'

|'oc set env dc/tododb --prefix MYSQL_ --from secret/tododb'

|'oc set env dc/tododb --list'

|'oc rsh tododb-<uniquenumber> env | grep MYSQL_'

The environment variables required by the database pod are MYSQL_USER, MYSQL_PASSWORD, and MYSQL_DATABASE.

The user name to access the database is todoapp, the password is mypass, and the database name is todo.

Retrieve the To Do List back end Node.js sources and S2I scripts from a local clone of your personal fork of the DO288-apps Git repository, in the todo-backend folder. Create a branch named review-service to save your changes.

Build and deploy the To Do List back end on the youruser-review-service project using backend as the name of your OpenShift resources. Use the nodejs:8 image stream tag as the S2I builder.

Download the required npm module dependencies from the following URL, which you provide as the value of the npm_config_registry build environment variable:

http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs

The environment variables required by the application pod are DATABASE_USER, DATABASE_PASSWORD, DATABASE_NAME, and DATABASE_SVC. Initialize them from the same tododb secret used to initialize the environment variables for the database pod.

|'oc new-app --name backend \
|    --build-env npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs \
|    --context-dir todo-backend \
|    -e DATABASE_NAME=todo \
|    -e DATABASE_USER=todoapp \
|    -e DATABASE_PASSWORD=mypass \
|    -e DATABASE_SVC=tododb \
|    nodejs:8~https://github.com/<username>/DO288-apps#review-service'

|'oc set env dc/backend --prefix DATABASE_ --from secret/tododb'

|'oc set env dc/backend --list'

|'oc rsh backend-<uniquenumber> env | grep DATABASE_'

Store application configuration parameters in a configuration map called todoapp, with the init key.

|'oc create cm todoapp --from-literal init=true'

The only configuration parameter not related to database access is the DATABASE_INIT environment variable. Pass the value true to force the application to create database tables on startup.

|'oc set env dc/backend --prefix DATABASE_ --from=cm/todoapp'

|'oc set env dc/backend --list'

Integrate the build process with the application life cycle management system for the organization. Use the lifecycle.sh script in the ~/DO288/labs/review-service folder as a placeholder, until the real integration script is available. Do not make any changes to the script, except to call the S2I assemble script provided by the Node.js builder image.

Test the To Do List back end using the default host name that OpenShift generates for new routes:

http://backend-youruser-review-service.apps.cluster.domain.example.com

Use the following HTTP API entry point that returns the number of items in the To Do List:

/todo/api/items-count

If you get zero items, then the back end is working and able to access the database.

|'oc expose svc backend'

|'curl -si <backend-route>/todo/api/items-count'

Inspect DB to confirm count:

|'oc port-forward tododb-3-zt4sx 30306:3306'

Open new tab and run following to inspect db:

|'mysqlshow -utodoapp -pmypass -h127.0.0.1 -P30306 todo'
