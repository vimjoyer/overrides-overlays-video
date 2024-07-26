# overrides & overlays video
https://youtu.be/jHb7Pe7x1ZY

## overriding a package located in ./default.nix

```nix
pkgs.callPackage ./default.nix {
  stdenv = pkgs.gcc7stdenv;
}

```

## overriding 2

```nix
pkgs.rofi-rbw.override (oldAttrs: {
    waylandSupport = true;
    rbw = oldAttrs.rbw.override { 
        withPass = true; 
    };
})
```

## overrideAttrs example

```nix
pkgs.hello.overrideAttrs {
  pname = "super hello";

  buildInputs = [ pkgs.cargo-mommy ];

  cmakeFlags = [
    "-SOME_REALLY_NEEDED_FLAG=OFF"
  ];

  src = ./custom-src;
}
```


## overlay flake example

```nix
{
  description = "flake";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs }: 
  let
    pkgs = import nixpkgs {
      system = "x86_64-linux";
      overlays = [
        (final: prev: {
          myCoolAttr = 5;
          
          rofi-rbw = prev.rofi-rbw.override {
            waylandSupport = true;
          };

          rofi-rbw-custom = final.rofi-rbw;
        })
      ];
    };
  in
  {
    someCoolOutput = pkgs.myCoolAttr;
  };
}
```

```nix
{
  description = "Nixos config flake";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs }: 
  let
    pkgs = import nixpkgs {
      system = "x86_64-linux";
      overlays = [
        (final: prev: {
          myCoolAttr = 5;
          gnome = prev.gnome.overrideScope (nfinal: nprev: {
            anotherCoolAttr = 6;
          });
        })
      ];
    };
  in
  {
    anotherCoolOutput = pkgs.gnome.anotherCoolAttr;
  };
}
```

## overlay in a nixos module (home-manager is the same)

```nix
{ pkgs, lib, ... }: 

{
  nixpkgs.overlays = [
    (final: prev: {
      rofi-rbw = prev.rofi-rbw.override {
        waylandSupport = true;
      }

      rofi-rbw-custom = final.rofi-rbw;
    })
  ];
    
  # home.packages = with pkgs; [ # for home-manager
  environment.systemPackages = with pkgs; [
    vim
    firefox

    rofi-rbw-custom
  ];

}
```

## nixpkgs references

### rofi packkage
https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/applications/misc/rofi/default.nix#L65

### rofi wayland
https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/applications/misc/rofi/wayland.nix#L25
