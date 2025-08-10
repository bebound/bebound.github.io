+++
title = "How to copy files temporarily in Dockerfile"
date = 2023-08-24T11:49:00+08:00
lastmod = 2025-08-10T18:44:06+08:00
tags = ["dockerfile"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

It's very common to copy a local file into the container when build docker image. In general, we use `COPY` command. But it creates a new layer and increase the final image size. If this is a temporal file and we don't want users waste their storage space, how can we remove it? Here are some approaches.


## Download the File Dynamically {#download-the-file-dynamically}

If the file can be download from URL or you can create a local HTTP server to share the file, you can download the file, use it and delete it in one `RUN` command. For example:

```dockerfile
RUN wget xxxx && unzip xxx && rm xxx
```


## `RUN --mount` Command {#run-mount-command}

You can also mount file when build image if your file can't be download from Internet or the file is secret. Use it to bind files or directories to the build container.

A bind mount is read-only by default, add `rw` parameter to make it writable. The changes during the build are discared after the build is complete.

The mounted folder are kept in the image, but the files are gone. Don't forget to delete the empty folder if you want to keep image clean.


### Mount folder {#mount-folder}

```dockerfile
RUN --mount=type=bind,target=/target_dir/,source=./source_dir/,rw
```


### Mount file {#mount-file}

```dockerfile
RUN --mount=type=bind,target=/azure-cli.rpm,source=./docker/azure-cli.rpm tdnf install ca-certificates /azure-cli.rpm -y && tdnf clean all
```


## `--squash` option in `docker build` {#squash-option-in-docker-build}

You can also use `--squash` to reduce image size.
Once the build is complete, Docker creates a new image loading the diffs from each layer into a single new layer and references all the parent's layers. So the extra space created by `COPY` command can be freed by `squash`.


## Ref {#ref}

1.  [Dockerfile best practice](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
2.  [RUN --mount](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/reference.md#run---mount)
3.  [How does the new Docker --squash work](https://stackoverflow.com/questions/41764336/how-does-the-new-docker-squash-work)
