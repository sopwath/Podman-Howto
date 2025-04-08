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
- You will probably need to `mkdir .config` within your home directory
- For now, I'll copy the example custom registries.conf file is as follows:

```text
unqualified-search-registries = ["docker.io", "ghrc.io", "quay.io"]
```

## Searching for Container Images

Before pulling an image, you'll want to find the specific container image you want to download. Do this with `podman search <image_name>`

While you can generally trust the images on docker, GitHub, and Quay, its still a good idea to specify the registry you want to search for the image name: `podman search docker.io/library/elasticsearch`
- In this case, only the elasticsearch container image on docker.io is the one your really want, others available at built by/for third parties or are unrelated container images.

## Downloading (Pulling) a Container Image

After actually finding the container image you want, you need to download it to your local server. You do this with the `podman pull <image_name>` command.

> [!IMPORTANT] ⚠️   
> It will require some additional work, but you should *always* use the fully qualified image names including the registry server, namespace, image name, and tag:
> `podman pull docker.io/library/elasticsearch:8.17.4`
>
> Pulling by digest further eliminates the ambiguity of tags. You really do need the *@sha256:91afdfb99a446bd35855e2b13ad2944c1f691bfed436f96de30d849fa59e91b8* at the end of the full image name:
> `podman pull docker.io/library/elasticsearch@sha256:91afdfb99a446bd35855e2b13ad2944c1f691bfed436f96de30d849fa59e91b8`

## Listing Container Images you have downloaded

Eventually you'll have a group of container images downloaded to your server. You can list the ones sitting locally with the `podman images` command

```bash
~$ podman images
REPOSITORY                       TAG         IMAGE ID      CREATED      SIZE
docker.io/library/elasticsearch  8.17.4      26649e4f96a1  2 weeks ago  1.33 GB
docker.io/library/elasticsearch  8.17.3      fe65524de871  5 weeks ago  1.33 GB
```

