# tectonic cache

Cache files for reproducible builds with
[tectonic](https://tectonic-typesetting.github.io/).

In order to generate a cache, run the following commands.

```bash
mkdir -p .cache/Tectonic
export XDG_CACHE_HOME="$(realpath .cache)"
export TECTONIC_CACHE_DIR="$(realpath .cache/Tectonic)"
tectonic main.tex
```

The cache will then be in `.cache/Tectonic` and can be tested with

```bash
tectonic --only-cached main.tex
```

## Building

For use with the [Nix](https://nixos.org/)
package manager, use a derivation like so.

```nix
{ lib, stdenvNoCC, fetchFromGitHub, tectonic }:

stdenvNoCC.mkDerivation {
  nativeBuildInputs = [ tectonic ];

  cache = fetchFromGitHub {
    owner = "stephen-huan";
    repo = "tectonic-cache";
    rev = "<rev>";
    hash = "<hash>";
  };

  buildPhase = ''
    runHook preBuild

    mkdir -p .cache
    export XDG_CACHE_HOME=$(realpath .cache)
    export TECTONIC_CACHE_DIR=${cache}
    export SOURCE_DATE_EPOCH=0
    tectonic --only-cached -Z deterministic-mode main.tex

    runHook postBuild
  '';

  installPhase = ''
    runHook preInstall

    install -Dm644 main.pdf -T $out/main.pdf

    runHook postInstall
  '';
}
```
