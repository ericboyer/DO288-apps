# DO288 Containerized Example Applications

This repository contains a collection of sample containerized applications.  To complete the course you need to fork this repo into your personal Github account.

## Notes

## Cool troubleshooting techniques
- Ping from api pod to backend database running on port 3306: 
| `oc rsh quotesapi-1-r6f31 bash -c 'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'`
