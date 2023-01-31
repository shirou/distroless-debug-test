This repository is a temporary repository for tracking distroless issues, at 2023/01/30.


It contains a very simple go program and a Dockerfile that is almost identical to the one in the distroless README, differing only in the debug tag.

https://github.com/GoogleContainerTools/distroless#examples-with-docker

It works fine with "latest" without debugging, but with "debug", problems arise.


# tl; dr;

Use ENTRYPOINT instead of CMD

```
- CMD ["/app"]
- ENTRYPOINT ["/app"]
```


# with debug docker tag

```
Step 6/8 : FROM gcr.io/distroless/static-debian11:debug
debug: Pulling from distroless/static-debian11
fc251a6e7981: Already exists
3508a3839ffc: Already exists
Digest: sha256:537761bfa493e3682aaf8e368aea2cd810f5f5db6ac55e41aec24e250afb47de
Status: Downloaded newer image for gcr.io/distroless/static-debian11:debug
 ---> faf9ba33476d
Step 7/8 : COPY --from=build /go/bin/app /
 ---> 8f66e76725f2
Step 8/8 : CMD ["/app"]
 ---> Running in 74bb2f4c4f37
Removing intermediate container 74bb2f4c4f37
 ---> d27199bec925
Successfully built d27199bec925
Successfully tagged test:latest


% docker run -t test
/app: line 1: syntax error: unterminated quoted string
```

We can run sh inside a container and run the application.

```
% docker run --entrypoint=sh -ti test
/ # ls
app      boot     dev      home     lib64    root     sbin     tmp      var
bin      busybox  etc      lib      proc     run      sys      usr
/ # ./app
It works!
```


# Build environment

- ubuntu: 22.04
- docker version: 20.10.21

# ContainerConfig of "docker inspect"

```
        "ContainerConfig": {
            "Hostname": "90202b673e6b",
            "Domainname": "",
            "User": "0",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/busybox",
                "SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/app\"]"
            ],
            "Image": "sha256:03471cc33939d30b9eac992d7dfe0b51506558037834ff82a75d67ca61503a98",
            "Volumes": null,
            "WorkingDir": "/",
            "Entrypoint": [
                "/busybox/sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
```

