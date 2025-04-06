# Podman-Howto
These are my personal notes on Podman usage, configuration, etc, blatantly copied from other sources I've found on the internet. I will try to provide links to those other references.

I have a specific use-case for Podman, that is to duplicate the [Logging Made Easy](https://github.com/cisagov/LME) project, without the additional overhead of installing the Nix package manager to a Debian-based system. I expect to encounter numerous challenges related to using Fedora, rather than Ubuntu, but Fedora/RHEL both support the "latest" (within a few days rather than being several months behind the official release) version of Podman natively. I'm aware I could simply use the pre-release update repositories in Ubuntu, but I'm trying hard to learn more about Podman beyond simply searching, pulling, and running containers. Let's get to it!

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
