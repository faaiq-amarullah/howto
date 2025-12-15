# Run Code Analysis with Sonar Scanner

## Run sonar-scanner with Docker

When you run this command below, you will enter inside the container environment. 

```bash
docker run -it --rm \
  -v "${PWD}:/usr/src" \
  sonarsource/sonar-scanner-cli bash
```

Then you can type command like this to start sonar-scanner.

```bash
sonar-scanner -D sonar.token=your-token -D sonar.host.url=http://your-server-ip-or-domain:9000 -D sonar.projectKey=test
```

You can simplify this command with bash alias. Add this line to `~/.bash_aliases`

```bash
sonarScanner() {
  docker run --rm \
    -e SONAR_HOST_URL="http://your-server-ip-or-domain:9000" \
    -e SONAR_TOKEN="your-token" \
    -v "${PWD}:/usr/src" \
    sonarsource/sonar-scanner-cli sonar-scanner "$@"
}
```

And then you just type this command to run sonar scanner on your terminal. Make sure you have already on the right folder.

```
sonarScanner --version
sonarScanner --help
sonarScanner -D sonar.projectKey=test
```

> [!WARNING]
> Do not use `localhost` or `127.0.0.1` in your `SONAR_HOST_URL`, the command will fail. Replace with your IP.