# VBOX Help

## start Headless VM

```bash
VBoxManage startvm "Besturing_Systemen_II" --type headless
```

## Power off VM

```bash
VBoxManage controlvm "Besturing_Systemen_II" poweroff
```

## Discard state

```bash
VBoxManage discardstate "Besturing_Systemen_II"
```

## Return to previous state

```bash
cd ~/Virtual\ VMs/Besturing_Systemen_II/
cp Besturing_Systemen_II.vbox-prev Besturing_Systemen_II.vbox
```

## VER_FILE_NOT_FOUND: Image file, .vdi, not found

In case that folder location of .vdi gets changed.
Either delete and recreate vm or change UUID of .vdi and change the .vbox file.

change UUID of .vdi:

```bash
VBoxManage internalcommands sethduuid <file-name>.vdi
```

Than edit .vbox file, change all occurences of uuid={...}.
Machine uuid={...},  HardDisk uuid={...} and Image uuid={...}.
Change uuid to new uuid of .vdi.

```bash
cd ~/Virtual\ VMs/Besturing_Systemen_II/
vim Besturing_Systemen_II.vbox
```
