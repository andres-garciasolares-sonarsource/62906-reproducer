This is a reproducer for an issue with the cache of the Kotlin analyzer on SonarQube Cloud, reported on ticket 62906.

1. Create a new project in SQC/staging.
2. Run a main branch analysis.
```bash
sonar-scanner -Dsonar.host.url=https://sc-staging.io -Dsonar.token=<token> -Dsonar.organization=<org> -Dsonar.projectKey=<key> -X &> log-master
```
In `log-master` we see the cache is uploaded:
```log
14:22:21.825 INFO  Sensor cache published successfully
```
3. Run a PR analysis with unchanged code.
```bash
sonar-scanner -Dsonar.host.url=https://sc-staging.io -Dsonar.token=<token> -Dsonar.organization=<org> -Dsonar.projectKey=<key> -Dsonar.pullrequest.key=myPR -Dsonar.pullrequest.branch=myBranch -Dsonar.pullrequest.base=master -X &> log-pr
```
In `log-pr` we see that `sample.kt` is unchanged, but the cache missed and the file is analyzed fully:
```log
14:23:27.044 DEBUG Cache contained a different hash for file sample.kt
14:23:27.045 INFO  Only analyzing 1 changed Kotlin files out of 1.
```
However, for `hello.js`, it seems like the cache worked, hinting that this problem is specific to the Kotlin sensor:
```log
14:23:30.528 DEBUG Cache entry extracted for key 'js:filemetadata:11.3.0.34350:62906-test-kotlin-file:hello.js'
14:23:30.529 DEBUG Cache entry extracted for key 'jssecurity:ucfgs:JSON:11.3.0.34350:62906-test-kotlin-file:hello.js'
14:23:30.529 DEBUG Cache entry extracted for key 'jssecurity:ucfgs:SEQ:11.3.0.34350:62906-test-kotlin-file:hello.js' containing 0 file(s)
14:23:30.581 DEBUG Cache entry extracted for key 'js:ast:11.3.0.34350:62906-test-kotlin-file:hello.js'
[...]
14:23:31.670 INFO  0 source files to be analyzed
[...]
14:23:31.674 INFO  Hit the cache for 1 out of 1
```
