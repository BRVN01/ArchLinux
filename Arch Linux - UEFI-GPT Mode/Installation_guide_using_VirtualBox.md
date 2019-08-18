# **Arch Linux Installation VirtualBox - UEFI/GPT Mode**

- [Introdução](#Introdução)
- [Gerenciador de Boot](#Gerenciador_de_Boot)
- [Configurando o Carregador - loader.conf](#Configurando_o_Carregador_loader.conf)
- [Projeto no GitHub](https://github.com/BRVN01/ArchLinux)



### **Introdução**

Esse tutorial é uma extensão do [Arch Linux Installation Guide - UEFI/GPT Mode](https://www.linkedin.com/pulse/arch-linux-installation-guide-uefigpt-mode-bruno-silva), onde é ensinado a instalar o Arch usando UEFI em uma máquina "real", apesar de estar fazendo o processo numa máquina virtual, o processo é exatamente o mesmo na máquina real, porém, para que funcione na máquina virtual, alguns passos devem ser diferentes.

Como explicado no final do tutorial, o VirtualBox possui um bug com o uso do efibootmgr. 

EFIBOOTMGR é uma ferramenta que interage com o firmware EFI do sistema para gerenciar entradas de inicialização do UEFI, você pode criar uma entrada de inicialização EFI e reiniciar, as entradas criadas pelo efibootmgr são apagadas todas vez que a máquina desliga, isso não acontece no reboot por que os dados estão carregados na NVRAM.



Para criar essa entrada você precisa iniciar por um LiveCD, montar as partições, entrar com chroot e usar o comando abaixo:

```bash
efibootmgr --create --disk /dev/sda --part 1 --write-signature --loader /EFI/Arch_GRUB/grubx64.efi --label "Arch_GRUB" --verbose

# Arch_GRUB é onde está instalado o firmware EFI dentro de /boot, 
# mas /boot deve ser omitido.
```



Mas isso só resolve o problema parcialmente, já que poderia ser inicializado o sistema pelo shell do efi, e após o desligamento do sistema o problema retorna.

Um dos métodos de correção para o bug é customizar o grub ou usar outro bootloader, há especulações que o uso de bootloader com configuração para boot seguro também resolva o problema, mas particularmente, não cheguei a testar a veracidade dessa especulação.

Para correção vamos instalar o bootloader do systemd, que já é nativo em sistemas Linux e possui uma configuração super simples. Para iniciarmos então, vamos começar do ponto em que o grub foi instalado e no nosso caso, ele não será instalado.



### **Gerenciador_de_Boot**

Vamos aplicar as configurações do nosso gerenciador de boot, depois vamos configurar dois aquivos, para aplicar o nosso gerenciador de boot, permitindo que ele entre em execução, rode o comando abaixo:



```bash
[root@archiso /]\# bootctl install
Created "/boot/EFI".
Created "/boot/EFI/systemd".
Created "/boot/EFI/BOOT".
Created "/boot/loader".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/systemd/systemd-bootx64.efi".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/BOOT/BOOTX64.EFI".
Created "/boot/loader/entries".
Created "/boot/580fdeffe39b41b2b78f3135ed5fdf2a".
Created EFI boot entry "Linux Boot Manager".

# Instalando o bootctl, para podermos configurar o 
# gerenciador de boot.
```



Agora vamos entrar na pasta  **/boot/loader/** para configurarmos o gerenciador de boot.

```bash
cd /boot/loader/ 
```



### **Configurando_o_Carregador_loader.conf**



Vamos editar o arquivo **loader.conf**  que possui as configurações do nosso carregador.

```bash
vim loader.conf
```



Vamos colocar a seguinte configuração nesse arquivo.

```bash
default arch
timeout 10

# A opção default, indica que sempre que a máquina for 
# iniciada, ele seleciona por padrão nosso sistema.

# Timeout é o tempo de espera em segundos que a janela do nosso
# bootloader irá ficar visível e caso você não aperte nenhuma tecla,
# o sistema inicia automaticamente em 10 segundos.
```





### **Adicionando_Entradas_ao_Carregador**



Agora vamos adicionar entradas ao nosso carregador, para que quando a máquina iniciar, ele identifique e possa carregar nosso sistema, devemos criar um arquivo dentro de **/boot/loader/entries**, com o nome da default de **loader.conf**.



Editando o aquivo do nosso carregador.

```bash
vim entries/arch.conf

# Inicialmente o arquivo não existe e devemos criar.
```



Agora vamos configurar nosso carregador.

```bash
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=daf6deaa-96bb-ea41-9ccf-34168bfb1d4a rw

# TITLE = O nome que será apresentado para você
# na tela do boot.

# Linux = Indica o caminho da imagem do Kernel
# Nesse caso, o vmlinuz é um kernel Linux comprimido 
# que é capaz de carregar o sistema operacional na memória RAM
# para que o computador se torne utilizável.

# initrd = Passa para o bootloader a localização
# da imagem do initramfs, o propósito de um initramfs 
# é montar o nosso sistema, a raiz do linux.

# options = Passa para o carregador qual é partição raiz do sistema,
# que deve ser montada como RW (leitura e escrita), para isso,
# usamos o PARTUUID da partição.
```

Para pegar o PARTUUID da partição raiz, voce pode usar o comando **blkid**, observe um exemplo abaixo:

```bash
Comando: blkid 
/dev/sda1: UUID="5526-CF48" TYPE="vfat" PARTUUID="10711fbb-738d-a147-ac0b-8c99452be744"

/dev/sda2: UUID="2d55a544-d3aa-4ea3-bd70-0c8903d6683d" TYPE="swap" PARTUUID="7f836529-7ebb-ee47-a16f-56d087d9067a"

/dev/sda3: UUID="79ec5bdc-0506-48f2-968f-f0e35a41d170" TYPE="ext4" PARTUUID="82200069-3078-bd43-9a12-207f07890da5"

/dev/sda4: UUID="f91f0835-4a88-4aac-911c-6995cab156ca" TYPE="ext4" PARTUUID="daf6deaa-96bb-ea41-9ccf-34168bfb1d4a"
```



Agora vamos configurar também as entradas de carregadores do Shell EFI, isso não é necessário, você pode acessá-lo por outros meios.

Eu copiei as imagens dos Shells EFI da ISO do arch, que fica em **/run/archiso/bootmnt/EFI/**.

```bash
# Com o a raiz do meu sistema montada em /mnt e o /boot montado em /mnt/boot,
# criei uma pasta la chamada 'shell' para comportar as imagens EFI.

mkdir /mnt/boot/EFI/shell


# Depois copiei as imagens para essa pasta.

cp /run/archiso/bootmnt/EFI/shellx64_v1.efi /mnt/boot/EFI/shell/
cp /run/archiso/bootmnt/EFI/shellx64_v2.efi /mnt/boot/EFI/shell/ 
```

Agora é só criar as entradas no carregador.

```bash
# As entrfas foram criadas em '/boot/loader/entries/'.

# Entrada versão 1 dei o nome de 'uefi-shell-v1-x86_64.conf'
title   UEFI Shell x86_64 v1
efi     /EFI/shell/shellx64_v1.efi


# Entrada versão 2 dei o nome de 'uefi-shell-v2-x86_64.conf'
title   UEFI Shell x86_64 v2
efi     /EFI/shell/shellx64_v2.efi
```



Pronto, sua máquina Virtual está pronta para iniciar em modo UEFI.



[Projeto no GitHub](https://github.com/BRVN01/ArchLinux)