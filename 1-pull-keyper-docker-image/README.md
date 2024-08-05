# 1. Pull Keyper Docker Image

There's two ways to use [Keyper](https://jarrid.xyz/keyper):

1. Run `./gradlew build` and run [Keyper](https://jarrid.xyz/keyper) with the combined JAR file: `java -jar ./lib/build/libs/lib-standalone.jar`. For more, go to [Keyper Development](https://jarrid.xyz/keyper/development/#jar).
2. Pull the pre-built [Keyper](https://jarrid.xyz/keyper) Docker image from GitHub. See the latest release [here](https://github.com/jarrid-xyz/keyper/pkgs/container/keyper). For more, go to [Keyper Development](https://jarrid.xyz/keyper/development/#docker).

In this tutorial, we will be using [Keyper](https://jarrid.xyz/keyper)'s Docker image as all the dependencies are already pre-packaged.

First, let's give [Keyper](https://jarrid.xyz/keyper) a try:

```sh {"id":"01J4HY3SJ31Y4KR0WANGJWPN6Y"}
docker run -it --rm --name keyper-cli ghcr.io/jarrid-xyz/keyper -h
```

You can also ping to specific version from [here](https://github.com/jarrid-xyz/keyper/pkgs/container/keyper) by setting:

```bash {"id":"01J4HYM947A89T60MBYH9F3FSY"}
export KEYPER_VERSION=[Enter Keyper Version]
docker run -it --rm --name keyper-cli ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} -h
```

If you can execute this successfully, [Keyper](https://jarrid.xyz/keyper) is ready for you to use.

## Bonus

Explore various [Keyper](https://jarrid.xyz/keyper) commands:

```bash {"id":"01J4HYQ4ZT0ETJHSRQT166EJEX"}
docker run -it --rm --name keyper-cli ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} resource -h
```

```bash {"id":"01J4HYQD9WCFGECVHGCK4A4Q91"}
docker run -it --rm --name keyper-cli ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} deploy -h
```

```bash {"id":"01J4HYVM8E6RF3MGJYWKVBCCYX"}
docker run -it --rm --name keyper-cli ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} data -h
```
