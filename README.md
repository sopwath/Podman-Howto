# Podman-Howto
These are my personal notes on Podman usage, configuration, etc, blatantly copied from other sources I've found on the internet. I will try to provide links to those other references.

I have a few specific use-cases for Podman. This started as an attempt to duplicate the [Logging Made Easy](https://github.com/cisagov/LME) project, without the additional overhead of installing the Nix package manager to a Debian-based system. I expect to encounter numerous challenges related to using Fedora, rather than Ubuntu, but Fedora/RHEL both support the "latest" (within a few days rather than being several months behind the official release) version of Podman natively. I'm aware I could simply use the pre-release update repositories in Ubuntu, but I'm trying hard to learn more about Podman beyond simply searching, pulling, and running containers. Let's get to it!

A lot of this tooling involves several aspects of the [Containers](https://github.com/containers/) tools.

## Installation
The [Podman Installation Instructions](https://podman.io/docs/installation) page has very succinct installation instructions. I will, of course, make this more complex than it needs to be.

**Fedora Linux:**  
Like any installation command, we should run updates before actually installing a new set of packages:  
`sudo dnf upgrade` (this is similar to *apt update && apt upgrade*)

Fedora doesn't appear to have access to the same [Container Tools AppStream](https://access.redhat.com/support/policy/updates/containertools) as RHEL. This is fine, we can install [Podman](https://github.com/containers/podman), [Buildah](https://github.com/containers/buildah), [Skopeo](https://github.com/containers/skopeo), [CRIU](https://criu.org/Main_Page), [udica](https://github.com/containers/udica), etc, individually.

**Windows:**  
You will not install Podman on Windows, but the Podman desktop app should make it easier to deal with containers, pods, images, volumes, kubernetes, etc on the *remote* server.

You will need to configure a [remote connection](https://podman-desktop.io/docs/podman/podman-remote) between your Windows workstation and the server running Podman and Podman-related tools.

## Post-Installation checks

To check the version of podman you're running, `podman version` will give some basic info. Mr. Scaramuzzino pointed out a few specific items, GoVersion and Podman Version.

You can also get a bunch of in-depth information with `podman info`

## Container Registry Configuration

Obviously, [Docker Hub](https://hub.docker.com/) is a common container image registry, but it's not the only option available. There's also the [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) and the [RedHat Container Registry](https://quay.io/) (I'll point to the open quay.io rather than the official RHEL registries)

The default list of Podman container registries is located at `/etc/containers/registries.conf`
- Podman uses this file by default when you pull or push an image.
- There's a line showing `unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io"]`

You can setup a regisry configuration on a per-user basis by creating a separate file at `$HOME/.config/containers/registries.conf`
- You might need to `mkdir .config` within your home directory, then create the registries.conf file with a text editor of your choice.
- For now, I'll copy the example custom registries.conf file as follows:

```text
unqualified-search-registries = ["docker.io", "ghrc.io", "quay.io"]
```

## Searching for Container Images

Before pulling an image, you'll want to find the specific container image you want to download. Do this with `podman search <image_name>`

While you can generally trust the images on docker, GitHub, and Quay, its still a good idea to specify the registry you want to search for the image name: `podman search docker.io/library/busybox`
- This is not *just* for security reasons, but also because there tends to be many versions of similarly named containers and you want to ensure you're getting the right one.
- In this case, only the busybox container image on docker.io is the one your really want, others available are built by/for third parties or are unrelated container images.

### Searching for image tags

Simply downloading the default *latest* tag is often fine, but sometimes you may need to get a specific image build or version. You can generally do this by searching the image and listing tags. From what I've seen, the typical output from `podman search` limits the results to 20 items. This isn't great when the registry has many-many versions of the container.

To list the image tags, use the `--list-tags` option:

```bash
~$ podman search docker.io/library/busybox --list-tags
NAME                       TAG
<a few rows were snipped just to show the end of the output>
docker.io/library/busybox  1.25
docker.io/library/busybox  1.25-glibc
docker.io/library/busybox  1.25-musl
```

I was able to get a much longer list that included the *latest* tag, *stable*, and others by using the `--limit 1000` option. I'm sure there's a better way to filter based on a tag marked *stable*, but this is a little-bit interesting to see a bigger list of tags over time.

```bash
~$ podman search busybox --list-tags --limit 1000
NAME                       TAG
<many rows snipped>
docker.io/library/busybox  1.37
docker.io/library/busybox  1.37-glibc
docker.io/library/busybox  1.37-musl
docker.io/library/busybox  1.37-uclibc
docker.io/library/busybox  1.37.0
docker.io/library/busybox  1.37.0-glibc
docker.io/library/busybox  1.37.0-musl
docker.io/library/busybox  1.37.0-uclibc
docker.io/library/busybox  buildroot-2013.08.1
docker.io/library/busybox  buildroot-2014.02
docker.io/library/busybox  glibc
docker.io/library/busybox  latest
docker.io/library/busybox  musl
docker.io/library/busybox  stable
docker.io/library/busybox  stable-glibc
docker.io/library/busybox  stable-musl
docker.io/library/busybox  stable-uclibc
docker.io/library/busybox  ubuntu
docker.io/library/busybox  ubuntu-12.04
docker.io/library/busybox  ubuntu-14.04
docker.io/library/busybox  uclibc
docker.io/library/busybox  unstable
docker.io/library/busybox  unstable-glibc
docker.io/library/busybox  unstable-musl
docker.io/library/busybox  unstable-uclibc
```

## Downloading (Pulling) a Container Image

After actually finding the container image you want, you need to download it to your local server. You do this with the `podman pull <image_name>` command.

> [!IMPORTANT]   
> You should *always* use the fully qualified image names including the registry server, namespace, image name, and tag:
> `podman pull docker.io/library/busybox`
>
> Pulling by index digest further eliminates the ambiguity of tags. 
> *Note:* You really do need the at-sign and the hash like so *@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f* at the end of the full image name.
> `podman pull docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f`

Let's try that now, using the index digest option:

```bash
~$ podman pull docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f
Trying to pull docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f...
Getting image source signatures
Copying blob 97e70d161e81 done   |
Copying config ff7a7936e9 done   |
Writing manifest to image destination
ff7a7936e9306ce4a789cf5523922da5e585dc1216e400efb3b6872a5137ee6b
```

## Listing Container Images you have downloaded

Eventually you'll have a group of container images downloaded to your server. You can list the ones sitting locally with the `podman images` command

```bash
~$ podman images
REPOSITORY                       TAG         IMAGE ID      CREATED       SIZE
docker.io/library/elasticsearch  8.17.4      26649e4f96a1  2 weeks ago   1.33 GB
docker.io/library/elasticsearch  8.17.3      fe65524de871  5 weeks ago   1.33 GB
docker.io/library/busybox        <none>      ff7a7936e930  6 months ago  4.53 MB
```

This is a little odd that the busybox image is so old and that there's no image tag.

### Checking Image digests

Before you pull an image, you should try to verify the image's *index digest*. This is a best practice and I added an admonition above. I cannot find clear information on listing digest information from a remote repository. Rather, I can only find the `podman image inspect` command *after* the image has been pulled and saved locally.

Now that I've downloaded the busybox image, I should be able to inspect it. I'd really like to do that because docker hub shows the image was pushed 21 days ago (as I'm writing this at least) but the created column shows 6 months ago. Something is wacky!

```bash
~$ podman inspect ff7
[
     {
          "Id": "ff7a7936e9306ce4a789cf5523922da5e585dc1216e400efb3b6872a5137ee6b",
          "Digest": "sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f",
          "RepoTags": [],
          "RepoDigests": [
               "docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f",
               "docker.io/library/busybox@sha256:ad9fa4d07136a83e69a54ef00102f579d04eba431932de3b0f098cc5d5948f9f"
          ],
          "Parent": "",
          "Comment": "",
          "Created": "2024-09-26T21:31:42Z",
          "Config": {
               "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
               ],
               "Cmd": [
                    "sh"
               ]
          },
          "Version": "",
          "Author": "",
          "Architecture": "amd64",
          "Os": "linux",
          "Size": 4527719,
          "VirtualSize": 4527719,
          "GraphDriver": {
               "Name": "overlay",
               "Data": {
                    "UpperDir": "/home/justins/.local/share/containers/storage/overlay/068f50152bbc6e10c9d223150c9fbd30d11bcfd7789c432152aa0a99703bd03a/diff",
                    "WorkDir": "/home/justins/.local/share/containers/storage/overlay/068f50152bbc6e10c9d223150c9fbd30d11bcfd7789c432152aa0a99703bd03a/work"
               }
          },
          "RootFS": {
               "Type": "layers",
               "Layers": [
                    "sha256:068f50152bbc6e10c9d223150c9fbd30d11bcfd7789c432152aa0a99703bd03a"
               ]
          },
          "Labels": null,
          "Annotations": {
               "org.opencontainers.image.url": "https://github.com/docker-library/busybox",
               "org.opencontainers.image.version": "1.37.0-glibc"
          },
          "ManifestType": "application/vnd.oci.image.manifest.v1+json",
          "User": "",
          "History": [
               {
                    "created": "2024-09-26T21:31:42Z",
                    "created_by": "BusyBox 1.37.0 (glibc), Debian 12"
               }
          ],
          "NamesHistory": [
               "docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f"
          ]
     }
]
```

## Running a container image

You can run a Podman container pretty simply with `podman run <image_name>`
- Notice you do *not* need to run podman with sudo, you can run the image as a normal user.

There are some command options that are really helpful when running a container from the command-line:
- `podman run -it` (when I first saw this, I thought it was some type of joke like "just run it!")
  - The `-it` option tells Podman to install a virtual terminal within the container, allowing you to interact with the container directly via the command-line rather than having to issue some command in a more complex way... (more on that later)
- `podman run -it --rm <image_name>`
  - The `--rm` option instructs Podman to *remove* the container after execution.
  - It sounds like this is considered a best practice to prevent containers from filling up your disk space.

### Checking if your container is running

Since you wont always want to run your container with that interactive terminal, there will be times you need to check if it's even running in the first place. You can check if your container is running with the `podman ps` command

*recall:* earlier there was that `--rm` option. If you have a container that was running, but was not removed, then you'll be able to see that as well by using `podman ps -a`

```bash
~$ podman ps -a
CONTAINER ID  IMAGE                        COMMAND     CREATED         STATUS                    PORTS       NAMES
8e7164d2efda  quay.io/quay/busybox:latest  /bin/sh     35 seconds ago  Exited (0) 9 seconds ago              distracted_lamarr
```

