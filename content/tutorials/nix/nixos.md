# NixOS

## Systemupgrades
NixOS selbst kann unabhängig von den installierten Paketen auf einen neueren Stand gebracht werden.
```bash
nixos-rebuild switch --upgrade
```
Dabei handelt es sich um den neuesten Stand *auf dem verwendeten Channel* (*channel*). Es versteht sich, dass das nicht notwendig die in einem absoluten Sinne neuste verfügbare Version sein muss. Sollen Systemupgrades automatisch durchgeführt werden, kann der Konfigurationsdatei die folgende Zeile hinzugefügt werden:
```nix
system.autoUpgrade.enable = true;
```
