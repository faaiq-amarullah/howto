# Run Sonar Scanner Over Docker

## Run Sonar Scanner CLI

When you run this command below, you will enter inside the container environment. 

```bash
# Replace <TOKEN> with your SonarQube analysis token
docker run \
  --network sonar \
  --rm \
  -e SONAR_HOST_URL="http://your-server-ip:9000" \
  -e SONAR_TOKEN="<TOKEN>" \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli bash
```

You can simplify this command with bash alias. Add this line to `~/.bash_aliases`

```bash
# Replace <TOKEN> with your SonarQube analysis token
sonarScanner() {
  docker run \
    --network sonar \
    --rm \
    -e SONAR_HOST_URL="http://sonar:9000" \
    -e SONAR_TOKEN="<TOKEN>" \
    -v "$(pwd):/usr/src" \
    sonarsource/sonar-scanner-cli sonar-scanner "$@"
}
```

And then you just type this command to run sonar scanner on your terminal.

```
sonarScanner --version
sonarScanner --help
```

> [!WARNING]
> Do not use `localhost` or `127.0.0.1` in your `SONAR_HOST_URL`, the command will fail. Replace with your IP.