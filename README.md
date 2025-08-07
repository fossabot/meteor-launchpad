# Meteor Launchpad - Base Docker Image for Meteor Apps
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fcleverpush%2Fmeteor-launchpad.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fcleverpush%2Fmeteor-launchpad?ref=badge_shield)


Based on https://github.com/jshimko/meteor-launchpad

### Build this image

Setup for Apple Silicon devices:

```sh
docker buildx build -t yourname/appbuild --platform linux/amd64
```


### Build your app image

Add the following to a `Dockerfile` in the root of your app:

```Dockerfile
FROM gbhrdt/meteor-launchpad:1
```

Then you can build the image with:

```sh
docker build -t yourname/app .
```

**Setting up a .dockerignore file**

There are several parts of a Meteor development environment that you don't need to pass into a Docker build because a complete production build happens inside the container.  For example, you don't need to pass in your `node_modules` or the local build files and development database that live in `.meteor/local`.  To avoid copying all of these into the container, here's a recommended starting point for a `.dockerignore` file to be put into the root of your app.  Read more: https://docs.docker.com/engine/reference/builder/#dockerignore-file

```
.git
.meteor/local
node_modules
```

### Run

Now you can run your container with the following command...
(note that the app listens on port 3000 because it is run by a non-root user for [security reasons](https://github.com/nodejs/docker-node/issues/1) and [non-root users can't run processes on port 80](https://stackoverflow.com/questions/16573668/best-practices-when-running-node-js-with-port-80-ubuntu-linode))

```sh
docker run -d \
  -e ROOT_URL=http://example.com \
  -e MONGO_URL=mongodb://url \
  -e MONGO_OPLOG_URL=mongodb://oplog_url \
  -e MAIL_URL=smtp://mail_url.com \
  -p 80:3000 \
  yourname/app
```


### Build Options

Meteor Launchpad supports setting custom build options in one of two ways.  You can either create a launchpad.conf config file in the root of your app or you can use [Docker build args](https://docs.docker.com/engine/reference/builder/#arg).  The currently supported options are to install any list of `apt-get` dependencies (Meteor Launchpad is built on `debian:jesse`).  

If you choose to install Mongo, you can use it by _not_ supplying a `MONGO_URL` when you run your app container.  The startup script will then start Mongo inside the container and tell your app to use it.  If you _do_ supply a `MONGO_URL`, Mongo will not be started inside the container and the external database will be used instead.

Note that having Mongo in the same container as your app is just for convenience while testing/developing.  In production, you should use a separate Mongo deployment or at least a separate Mongo container.

Here are examples of both methods of setting custom options for your build:

**Option #1 - launchpad.conf**

To use any of them, create a `launchpad.conf` in the root of your app and add any of the following values.

```sh
# launchpad.conf

# Install a custom Node version (default: latest 8.x)
NODE_VERSION=8.9.0
```

**Option #2 - Docker Build Args**

If you prefer not to have a config file in your project, your other option is to use the Docker `--build-arg` flag.  When you build your image, you can set any of the same values above as a build arg.

```sh
docker build \
  --build-arg NODE_VERSION=8.9.0 \
  -t myorg/myapp:latest .
```

## Installing Private NPM Packages

You can provide your [NPM auth token](https://blog.npmjs.org/post/118393368555/deploying-with-npm-private-modules) with the `NPM_TOKEN` build arg.

```sh
docker build --build-arg NPM_TOKEN="<your token>" -t myorg/myapp:latest .
```


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fcleverpush%2Fmeteor-launchpad.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fcleverpush%2Fmeteor-launchpad?ref=badge_large)