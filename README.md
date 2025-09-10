This is a reproducer for an issue with the cache of the Kotlin analyzer on SonarQube Cloud, reported on ticket 62906.

1. Create a new project in SQC/staging.
2. Run a main branch analysis.
```bash
sonar-scanner -Dsonar.host.url=https://sc-staging.io -Dsonar.token=<token> -Dsonar.organization=<org> -Dsonar.projectKey=<key> -X &> log-master
```
In `log-master` we see the cache is uploaded:
```log
15:25:10.707 INFO  Sensor cache published successfully
```
3. Run a PR analysis with unchanged code.
```bash
sonar-scanner -Dsonar.host.url=https://sc-staging.io -Dsonar.token=<token> -Dsonar.organization=<org> -Dsonar.projectKey=<key> -Dsonar.pullrequest.key=myPR -Dsonar.pullrequest.branch=myBranch -Dsonar.pullrequest.base=master -X &> log-pr
```
In `log-pr` we see that, although the file is unchanged, the cache missed:
```log
15:28:55.767 DEBUG Cache contained a different hash for file sample.kt
15:28:55.769 INFO  Only analyzing 1 changed Kotlin files out of 1.
```
