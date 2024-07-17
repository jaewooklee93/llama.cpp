# llama.cpp의 미리 빌드된 버전 설치하기

## Homebrew

Mac과 Linux에서 홈브루  pakege 관리자를 사용할 수 있습니다.

```sh
brew install llama.cpp
```
포뮬러는 새롭게 출시된 `llama.cpp` 버전으로 자동으로 업데이트됩니다. 자세한 내용은: https://github.com/ggerganov/llama.cpp/discussions/7668

## Nix

Mac과 Linux에서 Nix 패키지 관리기를 사용할 수 있습니다.

```sh
nix profile install nixpkgs#llama-cpp
```
flake 활성화된 설치를 위해.

또는

```sh
nix-env --file '<nixpkgs>' --install --attr llama-cpp
```

flake 사용이 비활성화된 설치를 위해.

이 표현식은 [nixpkgs repo](https://github.com/NixOS/nixpkgs/blob/nixos-24.05/pkgs/by-name/ll/llama-cpp/package.nix#L164) 내에서 자동으로 업데이트됩니다.

## Flox

Mac과 Linux에서 Flox를 사용하여 Flox 환경 내에서 llama.cpp를 설치할 수 있습니다.

```sh
flox install llama-cpp
```

Flox는 llama.cpp의 nixpkgs 빌드를 따릅니다.
