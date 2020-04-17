Instructions

All course review labs are based on the same use case; that is, a multicontainer version of the To Do List application. When you have finished all the review labs, the application will be deployed as three pods:

A single-page web front-end pod, based on Nginx.

An HTTP API back-end pod, based on Node.js.

A database pod, based on MySQL.

The third review lab creates a template to deploy the complete To Do List application, using the resource definitions created during the two previous review labs. To save time, you are provided with the resources already exported to a YAML file and cleaned up. Your job is to make the template reusable, and to implement Red Hat recommended practices by adding health probes to the template.

You must deploy the complete To Do List application according to the following specifications:

Create the todoapp template resource in the youruser-review-common project.

The template takes the following parameters. All of these parameters are required:

DATABASE_IMAGE: The URL of the MySQL server container image. Its default value is:

registry.access.redhat.com/rhscl/mysql-57-rhel7.

BACK_END_REPO The URL of the back-end sources. Its default value is:

https://github.com/yourgituser/DO288-apps.

BACK_END_CTXDIR The folder that contains the back-end sources. Its default value is: todo-backend.

BACK_END_BRANCH The the branch to fetch the back-end sources. Its default value is: master.

NPM_PROXY: The URL of the npm repository server. Its default value is:

http://nexus-common.apps.cluster.domain.example.com/repository/nodejs

SECRET: The secret for OpenShift webhooks. Its default value is randomly generated.

PASSWORD: The database connection password. It has no default value.

HOSTNAME: The host name used to access the To Do List front end from a web browser. It has no default value.

BACKEND: The host name used by the To Do List front end to access the To Do List back end. It has no default value.

CLEAN_DATABASE: Indicates whether the application initializes the database on start up. Its default value is "false". The double quotes are required.

Use the nodejs-mongodb-example standard template as a reference for the syntax to define parameters, and to reference parameters inside the template resource list.

The database user name todoapp is fixed in the template, as well as the database name todo.

Use the following entry points from the HTTP API of the To Do List back-end as health probes:

Readiness: /todo/api/host

Liveness: /todo/api/items-count

The liveness health probe fails until the database is populated.

Configure both probes with the following attributes:

initialDelaySeconds: 10

timeoutSeconds: 3

Deploy the template to the youruser-review-multicontainer project. Pass parameters to the template to get the following results:

The front-end application is available at:

http://todoui-youruser.apps.cluster.domain.example.com

The back-end application is available at:

http://todoapi-youruser.apps.cluster.domain.example.com

On the two previous URLs, use the shell variable RHT_OCP4_WILDCARD_DOMAIN to provide the value of apps.cluster.domain.example.com, and the shell variable RHT_OCP4_DEV_USER to provide the value of youruser. These variables are defined in the /usr/local/etc/ocp4.config shell file.

The database password is: redhat

The back-end application does not initialize the database.

Populate the database using the todo.sql script in the ~/DO288/labs/review-multicontainer folder.

Because the template deploys the To Do List front end from the image stream you created during the first comprehensive review lab, you must grant permissions to service accounts from the youruser-review-multicontainer project. Do not make any changes to the template file, except to use the parameters, as stated above.

The todoapp.yml starter template file is exported from the youruser-review-dockerfile and youruser-review-service projects, created during the previous comprehensive review labs. Although you will not work with these projects any further, this lab requires the youruser-review-common project, and the container image and image stream, created by performing the first review lab.
