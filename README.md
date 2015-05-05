# Dart client for the Docker remote API

See https://docs.docker.com/reference/api/docker_remote_api_v1.18/#image-tarball-format 
for more details.

This package provides a typed interface to the Docker REST API and deserializes
JSON response to Dart classes.

This package is intended to make it easy to automate Docker tasks like for 
example in [Grinder](https://pub.dartlang.org/packages/grinder) tasks.

## Usage

See example below or the unit tests.

Ensure you have Docker listening on a TCP port (for more details see 
[Bind Docker to another host/port or a Unix socket](https://docs.docker.com/articles/basics/#bind-docker-to-another-hostport-or-a-unix-socket)


## Supported commands

### Containers
- containers (List containers)
- create (Create a container)
- container (Inspect a container)
- top (List processes running inside a container) 
- logs (missing test) (Get container logs)
- export (Export a container)
- stats (Get container stats based on resource usage)
- resize (missing test) (Resize a container TTY)
- start (Start a container)
- stop (Stop a container)
- restart (Restart a container)
- kill (Kill a container)
- rename (Rename a container)
- pause (Pause a container)
- unpause (Unpause a container)
- attach (missing test) (Attach to a container)
- wait (Wait a container)
- remove (Remove a container)

### Images
- images (List images)
- build (Build image from a Dockerfile)
- create (missing test) (Create an image)
- image (Inspect an image)
- history (Get the history of an image)
- push (missing test) (Push an image into a repository)
- tag (Tag an image into a repository)
- remove (Remove an image)
- search (Search images)

### Misc
- auth (Check auth configuration)
- info (Display system-wide information)
- version (Show the Docker version information)
- ping (Ping the docker server)
- commit (Create a new image from a containers changes)
- events (Monitor Dockers events)
- get (Get a tarball containing all images in a repository)
- get all (missing test) (Get a tarball containing all images)
- load (missing test) (Load a tarball with a set of images and tags into Docker)
- exec create (Set up an exec instance)
- exec start (missing test) (Start an exec instance)
- exec resize (missing test) (Resize the tty session) 
- exec inspect (Inspect an exec command) 

## Feedback

I'm not a Docker specialist. If something isn't working as expected or you are
missing a feature just create a bug report in the 
[GitHub repo](https://github.com/bwu-dart/bwu_docker/issues).

## TODO
- container attach (missing test) 
- container attachWs (Attach to a container using websocket)
- container logs (missing test)
- container changes (Inspect changes on a containers filesystem)
- container resize (missing test)
- container copy (missing test)
- image build (missing test)
- misc load (missing test)
- image push (missing test)
- exec start (missing test) 
- exec resize (missing test)


## Example

```dart
library bwu_docker.example.images;

import 'package:bwu_docker/src/remote_api.dart';
import 'package:bwu_docker/src/data_structures.dart';

const dockerPort = 2375;

main() async {
  // initialize the connection to the Docker service
  final conn = new DockerConnection('localhost', dockerPort);
  await conn.init();

  // create a container from an image
  CreateResponse created = await conn.createContainer(
      new CreateContainerRequest()
    ..image = 'busybox'
    ..hostConfig.logConfig = {'Type': 'json-file'});

  // start the container
  await conn.start(created.container);

  // load the list of containers
  Iterable<Container> containers = await conn.containers();

  // investigate response
  var found = containers.firstWhere((c) => c.id == created.container.id);
  print('found: ${found.id}, name: ${found.names.join(', ')}\n');

  print('all:');
  containers.forEach((Container c) {
    print(
        'Container ID: ${c.id}, names: ${c.names.join(', ')}, status: ${c.status}');
  });

  // clean up
  await conn.stop(created.container);
  await conn.removeContainer(created.container);
}
```

for more examples check out the unit tests.
