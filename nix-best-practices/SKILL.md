---
name: nix-best-practices
description: Nix patterns for flakes, overlays, unfree handling, and binary overlays. Use when working with flake.nix or shell.nix.
---

# Nix Best Practices

## Flake Structure

```nix
{
  description = "Project description";
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; };
      in {
        devShells.default = pkgs.mkShell {
          buildInputs = with pkgs; [ /* packages */ ];
        };
      });
}
```

## Follows Pattern (Avoid Duplicate Nixpkgs)

Use `follows` to share parent nixpkgs:

```nix
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  some-overlay.url = "github:owner/some-overlay";
  some-overlay.inputs.nixpkgs.follows = "nixpkgs";
  another-overlay.url = "github:owner/another-overlay";
  another-overlay.inputs.nixpkgs.follows = "some-overlay";
};
```

All inputs must be listed in outputs function even if not directly used:

```nix
outputs = { self, nixpkgs, some-overlay, another-overlay, ... }:
```

## Applying Overlays

```nix
let
  pkgs = import nixpkgs {
    inherit system;
    overlays = [
      overlay1.overlays.default
      (final: prev: { myPackage = prev.myPackage.override { ... }; })
    ];
  };
in
```

## Handling Unfree Packages

### Option 1: nixpkgs-unfree (Recommended for Teams)

```nix
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  nixpkgs-unfree.url = "github:numtide/nixpkgs-unfree/nixos-unstable";
  nixpkgs-unfree.inputs.nixpkgs.follows = "nixpkgs";
  proprietary-tool.url = "github:owner/proprietary-tool-overlay";
  proprietary-tool.inputs.nixpkgs.follows = "nixpkgs-unfree";
};
```

Chain: `proprietary-tool` → `nixpkgs-unfree` → `nixpkgs`

### Option 2: User Config

`~/.config/nixpkgs/config.nix`: `{ allowUnfree = true; }`

### Option 3: Specific Packages

```nix
pkgs = import nixpkgs {
  inherit system;
  config.allowUnfreePredicate = pkg: builtins.elem (lib.getName pkg) [ "specific-package" ];
};
```

Note: `config.allowUnfree` in flake.nix doesn't work with `nix develop` — use nixpkgs-unfree or user config.

## Binary Overlay Pattern

When nixpkgs builds community version lacking features, create overlay fetching official binaries. (See 0xBigBoss/atlas-overlay, 0xBigBoss/bun-overlay)

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = nixpkgs.legacyPackages.${system};
        version = "1.0.0";
        sources = {
          "x86_64-linux"   = { url = "https://example.com/tool-linux-amd64-v${version}"; sha256 = "sha256-AAA="; };
          "aarch64-linux"  = { url = "https://example.com/tool-linux-arm64-v${version}"; sha256 = "sha256-BBB="; };
          "x86_64-darwin"  = { url = "https://example.com/tool-darwin-amd64-v${version}"; sha256 = "sha256-CCC="; };
          "aarch64-darwin" = { url = "https://example.com/tool-darwin-arm64-v${version}"; sha256 = "sha256-DDD="; };
        };
        source = sources.${system} or (throw "Unsupported system: ${system}");
        toolPackage = pkgs.stdenv.mkDerivation {
          pname = "tool"; inherit version;
          src = pkgs.fetchurl { inherit (source) url sha256; };
          sourceRoot = "."; dontUnpack = true;
          installPhase = ''
            mkdir -p $out/bin
            cp $src $out/bin/tool
            chmod +x $out/bin/tool
          '';
          meta = with pkgs.lib; {
            description = "Tool description";
            homepage = "https://example.com";
            license = licenses.unfree;
            platforms = builtins.attrNames sources;
          };
        };
      in {
        packages.default = toolPackage;
        packages.tool = toolPackage;
        overlays.default = final: prev: { tool = toolPackage; };
      })
    // {
      overlays.default = final: prev: { tool = self.packages.${prev.system}.tool; };
    };
}
```

### Getting SHA256 Hashes

```bash
nix-prefetch-url https://example.com/tool-linux-amd64-v1.0.0
nix hash to-sri --type sha256 <base32-hash>
# Or SRI directly:
nix-prefetch-url --type sha256 https://example.com/tool-linux-amd64-v1.0.0
```

## Dev Shell Patterns

```nix
# Basic
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [ nodejs python3 ];
  shellHook = ''echo "Dev environment ready"'';
};

# With env vars
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [ postgresql ];
  DATABASE_URL = "postgres://localhost/dev";
  shellHook = ''export PROJECT_ROOT="$(pwd)"'';
};

# Native dependencies (C libs)
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [ openssl postgresql ];
  shellHook = ''
    export C_INCLUDE_PATH="${pkgs.openssl.dev}/include:$C_INCLUDE_PATH"
    export LIBRARY_PATH="${pkgs.openssl.out}/lib:$LIBRARY_PATH"
    export PKG_CONFIG_PATH="${pkgs.openssl.dev}/lib/pkgconfig:$PKG_CONFIG_PATH"
  '';
};
```

## Direnv Integration

`.envrc`: `use flake`

For unfree without nixpkgs-unfree:
```bash
export NIXPKGS_ALLOW_UNFREE=1
use flake --impure
```

## Common Commands

```bash
nix flake update              # Update all inputs
nix flake update some-input   # Update specific input
nix flake check               # Check validity
nix flake metadata            # Show metadata
nix develop                   # Enter dev shell
nix develop -c <command>      # Run command in dev shell
nix build .#packageName       # Build package
nix run .#packageName         # Run package
```

## Troubleshooting

**"unexpected argument"**: All inputs must be in outputs fn: `outputs = { self, nixpkgs, other-input, ... }:`

**Unfree errors with nix develop**: `config.allowUnfree` in flake.nix doesn't propagate. Use nixpkgs-unfree, user config, or `NIXPKGS_ALLOW_UNFREE=1 nix develop --impure`.

**Duplicate nixpkgs downloads**: Use `follows` to chain to single nixpkgs source.

**Overlay not applied**: Ensure overlay in `overlays` list when importing nixpkgs.

**Hash mismatch**: Re-fetch with `nix-prefetch-url`, update hash.
