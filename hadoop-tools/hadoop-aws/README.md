# What happened

We currently use hadoop 2.7 due to spark/accumulo dependencies, but hadoop 2.7
has a poor implementation of s3a, such that we were not comfortable using it in
production to write from spark to s3. Hadoop 3x has a much more robust version
of s3a, but is not binary compatible with hadoop 2.7. So, we forked hadoop 3x
and modified the hadoop-aws module so that it could compile with hadoop 2.7,
and published this build to our own maven repo, for usage in our spark apps to
write data to s3.


# Brief steps

* Change pom.xml to use hadoop-common 2.7.3

* Compile Java with `mvn compile -DskipTests`, it will definitely have a lot of errors
* Fix errors with as little effort as possible. Usually by commenting out code,
  or inlining code that was added after hadoop 2.7 and is used by hadoop-aws.

# Validate it works

Eventually when compilation succeeds, you can use some basic steps to validate that the s3a filesystem works.

* install to your local maven cache: `mvn install`

* run a simple CLI test using the `hadoop` command line interface

```shell
hadoop fs \
    -libjars "${HOME}/.m2/repository/amperity/hadoop-aws/3.0.1/hadoop-aws-3.0.1.jar,${HOME}/.m2/repository/com/amazonaws/aws-java-sdk-bundle/$VERSION/x.jar" \
    -Dfs.s3a.server-side-encryption-algorithm=AES256 \
    -Dfs.s3a.access.key=XXX \
    -Dfs.s3a.secret.key=XXX \
    -fs s3a://amperity-test-storage/ -ls /
```
