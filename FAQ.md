# Common Errors

This section describes some of the common issues faced while
running demo examples.

## Docker images pull error

```
ERROR: unauthorized: Your request could not be authenticated by the GitHub Packages service. Please ensure your access token is valid and has the appropriate scopes configured.
```
- If above error is observed while pulling docker images,
create new github developer token and execute the below command.

```
docker login -u USERNAME -p TOKEN docker.pkg.github.com
```