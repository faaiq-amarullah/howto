# Run Code Analysis with Maven

## Prerequisites

- Maven 3.2.5+
- At least the minimal version of Java supported by your SonarQube Server instance

## Run Maven with Docker

Edit `pom.xml` file to add sonar properties configuration.

```xml
<properties>
    <sonar.host.url>http://your-server-ip-or-domain:9000</sonar.host.url>
    <sonar.token>your-token</sonar.token>
</properties>
```

When you run this command below, you will enter inside the container environment. 

```bash
docker run -it --rm \
  -v "${PWD}:/usr/src/maven" \
  -w /usr/src/mymaven maven:3.9.11-eclipse-temurin-17 bash
```

Then you can type command like this to start maven.

```bash
mvn clean verify
mvn sonar:sonar
```

You can simplify this command with bash alias. Add this line to `~/.bash_aliases`

```bash
sonarMvn() {
  docker run --rm \
    -v "${PWD}:/usr/src/mymaven" \
    -w /usr/src/mymaven maven:3.9.11-eclipse-temurin-17 mvn "$@"
}
```

And then you just type this command to run maven on your terminal. Make sure you have already on the right folder.

```
sonarMvn clean verify
sonarMvn sonar:sonar
```