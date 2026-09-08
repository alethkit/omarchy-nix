# omarchy-nix ARM64 port notes

Target upstream: `zicochaos/omarchy-nix`, current `main` inspected on 2026-09-08.

## What the proposed patch does

- Advertises both `x86_64-linux` and `aarch64-linux` flake package outputs.
- Allows the NixOS module wrapper to resolve ARM64 packages.
- Keeps the existing full behavioral/UX checks on x86_64, because the UX fixture deliberately selects Steam.
- Adds a native ARM64 check that builds both the vendored Omarchy package and a full reference NixOS system.
- Adds `nixosConfigurations.example-aarch64` as a directly buildable ARM reference configuration.
- Turns the managed Steam feature into an explicit x86_64-only feature with a useful assertion on ARM, instead of failing indirectly through `hardware.graphics.enable32Bit`.
- Updates the public docs/examples to mention ARM64.

## Why this is the conservative port

The repository's current ARM blocker is not an x86 dependency in the core Omarchy module. The x86-only `hardware.graphics.enable32Bit` setting is pulled in by the UX test's managed `steam` feature. The flake also has a second problem: its full check set assumes the global `nixosConfigurations.demo` is x86_64, so blindly adding `aarch64-linux` to `systems` makes architecture-sensitive checks compare an x86 demo against ARM packages.

This patch avoids claiming more than we have tested: it makes the actual package/module/reference-system path ARM-native while leaving the full behavioral suite on its existing architecture. A follow-up can make `tests/ux.nix` architecture-parameterized and move more checks to ARM.

## Validation to run on an ARM64 Nix machine

```sh
nix flake show
nix build .#packages.aarch64-linux.omarchy
nix build .#nixosConfigurations.example-aarch64.config.system.build.toplevel
nix flake check
```

For exhaustive cross-system evaluation from another architecture:

```sh
nix flake check --all-systems --no-build
```

Then, on an ARM64 host with KVM/QEMU available, build/run the reference VM:

```sh
nix build .#nixosConfigurations.example-aarch64.config.system.build.vm
./result/bin/run-nixos-vm
```

## Current automation limitation

The ready unified diff is committed on branch `aarch64-support` as `aarch64-support.patch`. This ChatGPT runtime cannot resolve github.com from its local shell, and the GitHub connector does not expose a server-side 'apply patch' operation; its ordinary file update API requires whole-file replacements. The connector also receives a 403 when trying to create a cross-repository PR into `zicochaos/omarchy-nix`.
