# Uma forma rápida e objetiva para instalar o Arch em UEFI

Lembra-se que isso não é metodo recomendado para todos os equipamentos e talvez não seja o ideal para você, qualquer duvida use a [ArchWiki](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs))

Irei pular algumas partes mais claras do Wiki: 
- [Verificar a assinatura](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Verificar_a_assinatura)
- [Inicializar o ambiente live](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Inicializar_o_ambiente_live)
- [Definir o layout do teclado](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Definir_o_idioma_do_ambiente_live)
- [Conectar à internet](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Conectar_%C3%A0_internet)
- [Atualizar o relógio do sistema](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Atualizar_o_rel%C3%B3gio_do_sistema)

### Vamos continuar desse ponto em diante; 
##### Observações: 
- Não irei Fazer Swap durante a instalação, irei aplicar o [zram](https://wiki.archlinux.org/index.php/Swap#Using_zswap_or_zram) após a instalação
- Irei usar o cfdisk para particionar o disco
- Irei usar o [Kernel](kernel.org) padrão do Linux 
- Irei usar a [Grub](https://wiki.archlinux.org/index.php/GRUB_(Portugu%C3%AAs)) para o [Bootloader](https://wiki.archlinux.org/index.php/Arch_boot_process_(Portugu%C3%AAs)#Gerenciador_de_boot)
- Estou usando um SSD de 240GB e o HDD de 1TB 

### Primeiro passo, definir as partições: 
Execute: 
> cfdisk /dev/sdX 

ou

> cfdisk -z /dev/sdX

Caso queira zerar tudo e criar uma nova tabela de partição (Recomendo que use **GPT**)

**Lembra-se de alterar o X para o HDD/SSD que você queira particionar, use o __fdisk -l__ para obter uma lista**

Feito isso vamos definir nossas partições, segue meu modelo ou [leia sobre outros exemplos](https://wiki.archlinux.org/index.php/Partitioning_(Portugu%C3%AAs)#Exemplos_de_leiaute):

| Tamanho  | Tipo | Ponto de montagem |
| ----- | ----- | ----- |
| 223GB  | Linux | /mnt  |
| 512MB  | [EFI System](https://wiki.archlinux.org/index.php/EFI_system_partition_(Portugu%C3%AAs))  | /mnt/boot |
| 1TB  | Linux | /mnt/home |

***/mnt e /mnt/boot estão usando o meu SSD de 240GB como armazenamento e o /mnt/home está usando meu HDD de 1TB***

### Após usar o cfdisk você precisa definir um [Sistema de arquivos](https://wiki.archlinux.org/index.php/File_systems_(Portugu%C3%AAs)) são apenas 3 comandos :) 

#### Lembra-se de alterar o "X" para o disco correto.

> mkfs.f2fs /dev/sdX1 -f

sdX1 é referente ao /mnt e o formato f2fs é para SSD, use **mkfs.ext4 /dev/sdX1** caso não seja um SSD;

.

> mkfs.fat -F32 /dev/sdX2

Lembrando que o sdX2 é referente ao /mnt/boot e o formato sempre deve ser FAT32

.

> mkfs.ext4 /dev/sdX3

Lembrando que o sdX3 é referente ao /mnt/home

### Agora nossas partições estão desta forma: 

| Tamanho  | Tipo | Ponto de montagem | [Sistema de arquivos](https://wiki.archlinux.org/index.php/File_systems_(Portugu%C3%AAs)) |
| ----- | ----- | ----- | ----- |
| 223GB  | Linux | /mnt  | F2FS |
| 512MB  | [EFI System](https://wiki.archlinux.org/index.php/EFI_system_partition_(Portugu%C3%AAs))  | /mnt/boot | FAT32 |
| 1TB  | Linux | /mnt/home  | ext4 |

Agora precisamos montar nossas partições :) 

> mount /dev/sdX1 /mnt

> mkdir -p /mnt/boot

> mount /dev/sdX2 /mnt/boot

> mkdir -p /mnt/home

> mount /dev/sdX3 /mnt/home

### Então nossas partições estão prontas e montadas para receber o Arch, vamos agora para proxima parte.

#### Vamos atualizar os espelhos do pacman, para o download dos arquivos serem mais rápidos. 

> pacman -Sy [reflector](https://wiki.archlinux.org/index.php/Reflector)

> reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist

Esse comando pode demorar alguns minutos. 

> pacman -Syy

##### Agora vamos instalar os pacotes essenciais dentro da /mnt usando o [pacstrap](https://projects.archlinux.org/arch-install-scripts.git/tree/pacstrap.in)

> pacstrap /mnt base base-devel linux linux-firmware grub nano mkinitcpio

__Caso você receba aviso do aic94xx e/ou wd719x não se preocupe, mais na frente vamos resolver esse problema__

Vamos definir os rótulos no fstab. 

> genfstab -p /mnt >> /mnt/etc/fstab

Você pode usar o nano para checar se o fstab está correto e editar ele caso exista algum error; Porém antes de fazer qualquer alteração eu recomendo [ler isso primeiro](https://wiki.archlinux.org/index.php/Fstab_(Portugu%C3%AAs))

### Agora nosso Arch está quase pronto, vamos entrar na raiz usando o Chroot e finalizar as configurações de bootloader, aplicativos e afins. 

> arch-chroot /mnt

Agora você está dentro do seu Arch Linux, porém não desligue a maquina ainda. 
Irei pular algumas partes que estão bastante claras no ArchWiki:
- [Fuso horário](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Fuso_hor%C3%A1rio)
- [Localização](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Localiza%C3%A7%C3%A3o)
- [Configuração de rede](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Configura%C3%A7%C3%A3o_de_rede)

Antes de seguir para o Grub, vamos precisar alguns pacotes: 

> pacman -Sy ntfs-3g fuse3 dosfstools efibootmgr exfat-utils mtools f2fs-tools reflector

> mkinitcpio -P

__Caso você receba aviso do aic94xx e/ou wd719x não se preocupe, mais na frente vamos resolver esse problema__

#### Defina uma senha para o root e para internet:

> passwd root

> pacman -S wireless_tools wpa_supplicant dialog network-manager-applet networkmanager

#### Caso você use um notebook, use o libinput para configurar seu touchpad: 
https://wiki.archlinux.org/index.php/Libinput

#### Vamos agora para instalação da GRUB UEFI

> grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

> grub-mkconfig -o /boot/grub/grub.cfg

#### Vamos instalar um ambinete gráfico, essa parte é muito pessoal então sinta-se livre para usar um DE ou uma WM, nesse caso irei usar o gnome e para meu DM irei usar o GDM. 

> pacman -S gnome gdm

Esse momento é seu, instale quantos pacotes você quiser, irei dar alguns exemplos: 

> pacman -S git python gimp sakura kitty

#### Agora vamos reiniciar a maquina;

> exit 

> umount -R /mnt

> reboot

### Seu sistema não encontrou o grub? Isso é normal para alguns notebooks.

Dê boot novamente no pendriver, porém escolha "EFI Shell 1.2" alguma coisa, provavelmente a terceira ou quarta opção. 

Agora vamos achar o EFI/grub dentro do EFI Shell. 

> map

Provavelmente vai aparecer 
**FS0: ...**
**FS1: ...**
**BLCK0: ...**
ou semlhantes, nossa GRub está dentro do FS1(Provavelmente):

> FS1:

> ls 

procure a pasta EFI (Cuidado para não confundir com a grub do pendriver 

> cd EFI

> cd grub

Dentro dessa pasta grub deve existir um arquivo único chamado grubx86_64.efi

> bcfg boot add 3 grubx86_64.efi "rEFInd Boot Manager"

Agora é só apertar Ctrl + Alt + Del e remover o pendriver, agora o Efi deve conseguir dar boot na Grub. 


### Dentro do seu sistema operacional (você vai estar dentro do TTY) você vai precisar habilitar alguns itens, vamos lá: 

__Lembra-se que o login é root e a senha você colocou lá atrás com o passwd root_

> systemctl enable gdm NetworkManager

> reboot 

Agora o GDM vai iniciar normalmente.

Abra o Terminal e execute os seguintes comandos: 

> reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist

> pacman -Syy

Você pode criar um user pelo painel do Gnome ou seguir a [ArchWiki](https://wiki.archlinux.org/index.php/Users_and_groups#User_management) e fazer um user por linha de comando

Para iniciar a ZRAM, basta seguir o [Wiki](https://wiki.archlinux.org/index.php/Improving_performance_(Portugu%C3%AAs)#Swap_na_zRAM_usando_uma_regra_do_udev): 
https://wiki.archlinux.org/index.php/Improving_performance_(Portugu%C3%AAs)#Swap_na_zRAM_usando_uma_regra_do_udev

#### Agora vamos resolver o problema do aic94xx e wd719x; 
**(Você precisa ter criado a conta, não tem como fazer isso pelo user root)**

> sudo pacman -S git go

> git clone https://aur.archlinux.org/yay.git

> cd yay

> makepkg -si

> cd ..

> rm -rf yay/

> yay -S aic94xx-firmware wd719x-firmware

> sudo mkinitcpio -P

Você não vai mais receber o aviso da falta desses dois firmware :) 

> reboot

Dai pra frente é só se divertir com seu sistema operacional. 
