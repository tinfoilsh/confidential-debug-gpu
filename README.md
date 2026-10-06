# Confidential Debug GPU

A GPU development variant of [confidential-debug](https://github.com/tinfoilsh/confidential-debug)
with the Tinfoil shim and a small HTTP starter service. The shim obtains a publicly
trusted TLS certificate through Tinfoil's certificate proxy and serves
`confidential-debug-gpu` at the assigned HTTPS domain. The configuration uses CVM
image `0.14.12`, 32 vCPUs, 512 GiB of memory, and one GPU.

## Launch in debug mode

Sign in with the [Tinfoil CLI](https://github.com/tinfoilsh/tinfoil-cli) and register
your SSH public key once:

```sh
tinfoil login
tinfoil ssh-key create laptop --public-key-file ~/.ssh/id_ed25519.pub
```

Launch on an available GPU host with enough CPU and memory:

```sh
tinfoil container create gpu-dev \
  --repo tinfoilsh/confidential-debug-gpu \
  --tag v0.1.0 \
  --debug \
  --ssh-key laptop

tinfoil container get gpu-dev --debug-mode
```

Open the assigned HTTPS domain in a browser or with `curl`. Connect to the debug
toolbox using the SSH command shown in the dashboard:

```sh
ssh -p <port> root@console.tinfoil.sh
```

## Develop

The base image initializes the shim, guest networking, Docker, and GPU support.
Inside the debug toolbox, edit the copied configuration and apply it:

```sh
vim ~/tinfoil-config.debug.yml
tindbg boot
tindbg status
```

Replace the starter with your application and update `shim.upstream-container`
and `shim.upstream-port` to match. Keep the injected `tinfoil-debug-toolbox`
container. Run development tools in separate writable containers; the toolbox
itself is read-only. Runtime edits and ordinary container data are ephemeral.

Debug instances have HTTPS encryption but do not pass production attestation
verification. See the [debug-mode guide](https://docs.tinfoil.sh/containers/debug-mode)
for development commands and access details.

## Release

Run the **Tinfoil Release** workflow with a new version, such as `v0.1.1`.
It creates a tag and dispatches the publishing workflow, which uses
`measure-image-action` `v0.13.2` to measure the configuration, attest it, and
publish the deployment artifacts. Action references are pinned to commit SHAs.
