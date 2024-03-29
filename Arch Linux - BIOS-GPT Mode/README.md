# **Arch Linux Installation Guide - BIOS/GPT Mode**

- [Introdução](#Introdução)
- [O problema do GPT na BIOS](#O_Problema_do_GPT_na_BIOS)
	- [A solução](#A_solução)
- [Após logar no Arch](#Após_logar_no_Arch)
- [Particionando o disco](#Particionando_o_disco)
- [Criando uma tabela de partição do tipo GPT](#Criando_uma_tabela_de_partição_do_tipo_GPT)
- [Formatar as partições](#Formatar_as_partições)
- [Instalação](#Instalação)
	- [Configurar o sistema](#Configurar_o_sistema)
	- [Mudando a raiz do sistema](#Mudando_a_raiz_do_sistema)
	- [Configurando o Relógio](#Configurando_o_Relógio)
	- [Instalando o vim](#Instalando_o_vim)
	- [Definir idioma no S.O. do Arch](#Definir_idioma_no_S.O_do_Arch)
	- [Tornando o layout do teclado permanente](#Tornando_o_layout_do_teclado_permanente)
	- [Configurando nome da máquina](#Configurando_nome_da_máquina)
	- [Configurando o arquivo HOSTS](#Configurando_o_arquivo_HOSTS)
	- [Configurando WIFI - Caso tenha um adaptador WIFI](#Configurando_WIFI_Caso_tenha_um_adaptador_WIFI)
	- [Configurar senha do ROOT](#Configurar_senha_do_ROOT)
	- [Instalando a multilib](#Instalando_a_multilib)
	- [Instalando o GRUB](#Instalando_o_GRUB)
	- [Configurando o GRUB](#Configurando_o_GRUB)
	- [ACPI/ACPID para NOTEBOOKS](#ACPI/ACPID_para_NOTEBOOKS)
	- [Saindo do nosso sistema](#Saido_do_nosso_sistema)
- [Desmontando nosso Sistema](#Desmontando_nosso_Sistema)
- [Reiniciando a máquina](#Reiniciando_a_máquina)


#### Introdução

Atualmente muitos computadores já vem de fábrica com o suporte a BIOS e UEFI, a maioria das placas identifica o uso da BIOS como `BIOS legacy`, ou seja, BIOS legada, que vai deixar de existir. Para uso da BIOS em placas com UEFI, você deve selecionar o tipo de BIOS, entrando na configuração dela, algumas até mesmo exigem que seja desativado o `BIOS Secure Boot`.

O modo BIOS é usado por Máquinas virtuais, como a VirtualBox que costumo usar bastante e máquinas reais antigas, vale ressaltar que o uso do BIOS por padrão é vinculado a tabelas de partições do tipo MBR (ambos por serem antigos), que é usada na tabela de partição do tipo `msdos` ou somente `DOS` (as duas nomenclatura representam a mesma coisa) e o uso de UEFI é vinculado a tabelas de partições do tipo `GPT`., mas isso não é uma regra e você pode mudar isso.



Qual é melhor? 

Essa é uma pergunta que eu sempre digo, depende do seu hardware, o modo BIOS/MBR ou BIOS/DOS (mesma coisa) ainda é muito usado, mas você pode usar o modo BIOS/GPT, dependendo do tamanho do seu HD ou simplesmente se você quer usá-lo ou se você precisa de um número grande de partição e não quer ter partições estendidas, o bootloader tem que dar suporte, assim como o S.O instalado, por padrão todas as distros Linux suportam GPT e o grub vem fazendo um grande sucesso por sua compatibilidade.

 

Este tutorial descreve o uso do Arch Linux com o modelo BIOS/GPT, um outro será feito para UEFI/GPT.



Se você tem dúvidas sobre tabelas GPT e MBR, ou o que é uma tabela de partição, tipos de partições, recomendo que veja um pequeno tutorial no link abaixo: 

[GPT versus MBR - O que você não vê!](https://www.linkedin.com/pulse/gpt-versus-mbr-o-que-voc%25C3%25AA-n%25C3%25A3o-v%25C3%25AA-bruno-silva)



#### O_Problema_do_GPT_na_BIOS

Diferente de uma tabela MBR que foi desenvolvida para trabalhar com BIOS, o GPT foi desenvolvido para trabalhar com UEFI e, por natureza, a BIOS não reconhece o cabeçalho GPT. Por esse motivo, quando uma tabela GPT é criada, o primeiro setor do disco recebe a PMBR (Protective Master Boot Record), criando uma compatibilidade entre BIOS e GPT.

Logo após a PMBR, temos os cabeçalhos iniciais do GPT (setor 0). Isso é um "problema" porque como o bootloader inicial está na PMBR, os próximos setores depois dele deveriam estar vazios (setor 1 até 62). Esses setores seriam usados para armazenar a imagem principal do grub que iria acessar /boot/grub, e iniciar a chamada ao Sistema (sem entrar em detalhes, começar a iniciar o S.O). Esse espaço vazio é estritamente pequeno e serve para armazenar a imagem principal do grub (core.img).

Como a GPT inicia seus cabeçalhos (setor 1 até 33) logo após a PMBR (setor 1), depois dos cabeçalhos GPT já teremos as partições do HD (teríamos que ter um espaço vazio para o core.img), com isso, não sobra espaço para armazenar a imagem principal do grub, muito menos para ela ser inicializada, na verdade.



#### A_solução

Devemos criar uma partição no inicio do disco (sda1 para sistemas baseado em Unix). Essa partição fica logo após o setor 33, sendo o setor 34 do disco. A imagem pesa cerca de 32KiB, e a  recomendação é que seja usado uma partição de 1MiB, mas por via das dúvidas, crie uma partição de 1 GB para armazenar.



#### Após_logar_no_Arch

Configure o teclado para o padrão ABNT2, caso esteja usando um teclado que tenha cedilha.

```bash
loadkeys br-abnt2
```



#### Particionando_o_disco

Vamos particionar o disco para que ele receba nosso Sistema Operacional.
Eu vou colocar o `/home` e `/boot` numa partição separada.

A Arch wiki recomenda que seja usada o ID de partição abaixo para discos GPT usando BIOS:

```
Partição	Nome							ID de partição: Descrição
NONE		Partição VAZIA					BIOS boot partition
/			Raiz do Sistema					Linux x86-64 root (/) 
/home		Diretório de Usuários			Linux /home 
/boot		Diretório de Inicialização		BIOS boot partition
swap		Diretório de Troca				Linux swap


- Códigos:

21686148-6449-6E6F-744E-656564454649 :BIOS boot partition
4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709 :Linux x86-64 root (/) 
933AC7E1-2EB4-4F13-B844-0E14E2AEF915 :Linux /home
0657FD6D-A4AB-43C4-84E5-0933C84B4F4F :Linux swap
```

Vale lembrar que o tipo de partição acima é apenas uma tag para facilitar que o administrador saiba do que se trata cada partição.



Meu HD não possui nenhuma tabela de partição, portanto precisamos criar uma, caso o seu HD já tenha, certifique-se de que a tabela de partição seja do tipo GPT.

<span style="color:red">Esse procedimento irá destruir todos os dados de seu HD, recomendo que você faça backup do HD.



Vamos criar uma nova tabela do tipo GPT no nosso HD:

```sh
parted /dev/sda mklabel gpt

# Criando uma tabela de partição do tipo GPT.
```

Se você rodar o comando `fdisk -l /dev/sda | grep "Disklabel type"` você verá que o `Disklabel type` está como `gpt`.



Agora vamos criar a partição do `/boot`, eu sempre deixo o `/boot` como `sda`, muitas distros não sobem o sistema se não for o `sda`, mesmo que você especifique, portanto, para evitar maiores problemas, mantenha o `/boot` como `sda` da sua máquina.



Criando a partição do GRUB:

```bash
echo -e "n\n1\n\n+1G\nt\n21686148-6449-6E6F-744E-656564454649\nw" | fdisk /dev/sda
```



Criando a partição do boot:

```bash
echo -e "n\n2\n\n+5G\ny\nt\n2\n21686148-6449-6E6F-744E-656564454649\nw" | fdisk /dev/sda
```



Criando a partição da swap:

```bash
echo -e "n\n3\n\n+5G\ny\nt\n3\n0657FD6D-A4AB-43C4-84E5-0933C84B4F4F\nw" | fdisk /dev/sda
```



Criando a partição do home:

```bash
echo -e "n\n4\n\n+5G\ny\nt\n4\n933AC7E1-2EB4-4F13-B844-0E14E2AEF915\nw" | fdisk /dev/sda
```



Criando a raiz do sistema:

```bash
echo -e "n\n5\n\n\ny\nt\n5\n4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709\nw" | fdisk /dev/sda
```



Exibindo as partições criadas:

```bash
fdisk -l /dev/sda | grep -E "/dev/sd?"

Disk /dev/sda: 41 GiB, 44023414784 bytes, 85983232 sectors
/dev/sda1      2048  2099199  2097152   1G BIOS boot
/dev/sda2   2099200 12584959 10485760   5G BIOS boot
/dev/sda3  12584960 23070719 10485760   5G Linux swap
/dev/sda4  23070720 33556479 10485760   5G Linux home
/dev/sda5  33556480 85983198 52426719  25G Linux root (x86-64)


# Perceba que o disco tem 41 GiB, temos 5 partições
# Perceba que o tipo de partição é apenas uma tag.
```



A tabela abaixo representa os códigos suportados pelo `fdisk` para tabelas *GPT*:

```
  1 EFI System                     C12A7328-F81F-11D2-BA4B-00A0C93EC93B
  2 MBR partition scheme           024DEE41-33E7-11D3-9D69-0008C781F39F
  3 Intel Fast Flash               D3BFE2DE-3DAF-11DF-BA40-E3A556D89593
  4 BIOS boot                      21686148-6449-6E6F-744E-656564454649
  5 Sony boot partition            F4019732-066E-4E12-8273-346C5641494F
  6 Lenovo boot partition          BFBFAFE7-A34F-448A-9A5B-6213EB736C22
  7 PowerPC PReP boot              9E1A2D38-C612-4316-AA26-8B49521E5A8B
  8 ONIE boot                      7412F7D5-A156-4B13-81DC-867174929325
  9 ONIE config                    D4E6E2CD-4469-46F3-B5CB-1BFF57AFC149
 10 Microsoft reserved             E3C9E316-0B5C-4DB8-817D-F92DF00215AE
 11 Microsoft basic data           EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
 12 Microsoft LDM metadata         5808C8AA-7E8F-42E0-85D2-E1E90434CFB3
 13 Microsoft LDM data             AF9B60A0-1431-4F62-BC68-3311714A69AD
 14 Windows recovery environment   DE94BBA4-06D1-4D40-A16A-BFD50179D6AC
 15 IBM General Parallel Fs        37AFFC90-EF7D-4E96-91C3-2D7AE055B174
 16 Microsoft Storage Spaces       E75CAF8F-F680-4CEE-AFA3-B001E56EFC2D
 17 HP-UX data                     75894C1E-3AEB-11D3-B7C1-7B03A0000000
 18 HP-UX service                  E2A1E728-32E3-11D6-A682-7B03A0000000
 19 Linux swap                     0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
 20 Linux filesystem               0FC63DAF-8483-4772-8E79-3D69D8477DE4
 21 Linux server data              3B8F8425-20E0-4F3B-907F-1A25A76F98E8
 22 Linux root (x86)               44479540-F297-41B2-9AF7-D131D5F0458A
 23 Linux root (ARM)               69DAD710-2CE4-4E3C-B16C-21A1D49ABED3
 24 Linux root (x86-64)            4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709
 25 Linux root (ARM-64)            B921B045-1DF0-41C3-AF44-4C6F280D3FAE
 26 Linux root  (IA-64)             993D8D3D-F80E-4225-855A-9DAF8ED7EA97
 27 Linux reserved                 8DA63339-0007-60C0-C436-083AC8230908
 28 Linux home                     933AC7E1-2EB4-4F13-B844-0E14E2AEF915
 29 Linux RAID                     A19D880F-05FC-4D3B-A006-743F0F84911E
 30 Linux extended boot            BC13C2FF-59E6-4262-A352-B275FD6F7172
 31 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
 32 FreeBSD data                   516E7CB4-6ECF-11D6-8FF8-00022D09712B
 33 FreeBSD boot                   83BD6B9D-7F41-11DC-BE0B-001560B84F0F
 34 FreeBSD swap                   516E7CB5-6ECF-11D6-8FF8-00022D09712B
 35 FreeBSD UFS                    516E7CB6-6ECF-11D6-8FF8-00022D09712B
 36 FreeBSD ZFS                    516E7CBA-6ECF-11D6-8FF8-00022D09712B
 37 FreeBSD Vinum                  516E7CB8-6ECF-11D6-8FF8-00022D09712B
 38 Apple HFS/HFS+                 48465300-0000-11AA-AA11-00306543ECAC
 39 Apple UFS                      55465300-0000-11AA-AA11-00306543ECAC
 40 Apple RAID                     52414944-0000-11AA-AA11-00306543ECAC
 41 Apple RAID offline             52414944-5F4F-11AA-AA11-00306543ECAC
 42 Apple boot                     426F6F74-0000-11AA-AA11-00306543ECAC
 43 Apple label                    4C616265-6C00-11AA-AA11-00306543ECAC
 44 Apple TV recovery              5265636F-7665-11AA-AA11-00306543ECAC
 45 Apple Core storage             53746F72-6167-11AA-AA11-00306543ECAC
 46 Solaris boot                   6A82CB45-1DD2-11B2-99A6-080020736631
 47 Solaris root                   6A85CF4D-1DD2-11B2-99A6-080020736631
 48 Solaris /usr & Apple ZFS       6A898CC3-1DD2-11B2-99A6-080020736631
 49 Solaris swap                   6A87C46F-1DD2-11B2-99A6-080020736631
 50 Solaris backup                 6A8B642B-1DD2-11B2-99A6-080020736631
 51 Solaris /var                   6A8EF2E9-1DD2-11B2-99A6-080020736631
 52 Solaris /home                  6A90BA39-1DD2-11B2-99A6-080020736631
 53 Solaris alternate sector       6A9283A5-1DD2-11B2-99A6-080020736631
 54 Solaris reserved 1             6A945A3B-1DD2-11B2-99A6-080020736631
 55 Solaris reserved 2             6A9630D1-1DD2-11B2-99A6-080020736631
 56 Solaris reserved 3             6A980767-1DD2-11B2-99A6-080020736631
 57 Solaris reserved 4             6A96237F-1DD2-11B2-99A6-080020736631
 58 Solaris reserved 5             6A8D2AC7-1DD2-11B2-99A6-080020736631
 59 NetBSD swap                    49F48D32-B10E-11DC-B99B-0019D1879648
 60 NetBSD FFS                     49F48D5A-B10E-11DC-B99B-0019D1879648
 61 NetBSD LFS                     49F48D82-B10E-11DC-B99B-0019D1879648
 62 NetBSD concatenated            2DB519C4-B10E-11DC-B99B-0019D1879648
 63 NetBSD encrypted               2DB519EC-B10E-11DC-B99B-0019D1879648
 64 NetBSD RAID                    49F48DAA-B10E-11DC-B99B-0019D1879648
 65 ChromeOS kernel                FE3A2A5D-4F32-41A7-B725-ACCC3285A309
 66 ChromeOS root fs               3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC
 67 ChromeOS reserved              2E0A753D-9E48-43B0-8337-B15192CB1B5E
 68 MidnightBSD data               85D5E45A-237C-11E1-B4B3-E89A8F7FC3A7
 69 MidnightBSD boot               85D5E45E-237C-11E1-B4B3-E89A8F7FC3A7
 70 MidnightBSD swap               85D5E45B-237C-11E1-B4B3-E89A8F7FC3A7
 71 MidnightBSD UFS                0394EF8B-237E-11E1-B4B3-E89A8F7FC3A7
 72 MidnightBSD ZFS                85D5E45D-237C-11E1-B4B3-E89A8F7FC3A7
 73 MidnightBSD Vinum              85D5E45C-237C-11E1-B4B3-E89A8F7FC3A7
 74 Ceph Journal                   45B0969E-9B03-4F30-B4C6-B4B80CEFF106
 75 Ceph Encrypted Journal         45B0969E-9B03-4F30-B4C6-5EC00CEFF106
 76 Ceph OSD                       4FBD7E29-9D25-41B8-AFD0-062C0CEFF05D
 77 Ceph crypt OSD                 4FBD7E29-9D25-41B8-AFD0-5EC00CEFF05D
 78 Ceph disk in creation          89C57F98-2FE5-4DC0-89C1-F3AD0CEFF2BE
 79 Ceph crypt disk in creation    89C57F98-2FE5-4DC0-89C1-5EC00CEFF2BE
 80 VMware VMFS                    AA31E02A-400F-11DB-9590-000C2911D1B8
 81 VMware Diagnostic              9D275380-40AD-11DB-BF97-000C2911D1B8
 82 VMware Virtual SAN             381CFCCC-7288-11E0-92EE-000C2911D0B2
 83 VMware Virsto                  77719A0C-A4A0-11E3-A47E-000C29745A24
 84 VMware Reserved                9198EFFC-31C0-11DB-8F78-000C2911D1B8
 85 OpenBSD data                   824CC7A0-36A8-11E3-890A-952519AD3F61
 86 QNX6 file system               CEF5A9AD-73BC-4601-89F3-CDEEEEE321A1
 87 Plan 9 partition               C91818F9-8025-47AF-89D2-F030D7000C2C
 88 HiFive Unleashed FSBL          5B193300-FC78-40CD-8002-E86C45580B47
 89 HiFive Unleashed BBL           2E54B353-1271-4842-806F-E436D6AF6985
```



#### Formatar_as_partições

Vamos formatar as partições com o *FS* para que elas recebam os dados adequado de cada partição.

```bash
# Formata a segunda partição com o FS ext4 - /boot
mkfs.ext4 /dev/sda2

# Formata a quarta partição com o FS ext4 - /home
mkfs.ext4 /dev/sda4

# Formata a quinta partição com o FS ext4 - /
mkfs.ext4 /dev/sda5
```



Formatar partição swap:

```bash
mkswap /dev/sda3
```



Ligar swap:

```
swapon /dev/sda3
```



Montar a partição raiz para iniciar o processo de instalação:

```bash
mount /dev/sda5 /mnt/
```

Todas as outras partições devem ser montadas dentro dessa pasta (/mnt/).



Criar as pastas para montarmos:

```bash
# Cria a pasta HOME
mkdir /mnt/home

# Cria a pasta BOOT
mkdir /mnt/boot
```



Montar a partição `boot` e `home`:

```bash
# Montando a partição que receberá a pasta HOME
mount /dev/sda4 /mnt/home

# Montando a partição que receberá a pasta BOOT
mount /dev/sda2 /mnt/boot
```



#### Instalação

Usando o script `pacstrap`, vamos instalar um grupo de pacote base para o funcionamento do sistema:

```bash
pacstrap /mnt base base-devel

# base = Base do linux que será instalado.

# base-devel = Programas mais voltado para desenvolvedores, aqui temos compiladores, compactadores, bibliotecas do GNU e alguns programas como: sed, grep, sudo, systemd entre outros pacotes.
```

Esse grupo não inclui todas as ferramentas da instalação *live*, tal como [btrfs-progs](https://www.archlinux.org/packages/?name=btrfs-progs) ou firmware de rede sem fio específico.



#### Configurar_o_sistema

Gerar o arquivo fstab com os discos montados, isso permite que sempre que o sistema inicializar, ele monte as partições/discos.

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab

# -p = Excluir montagens do pseudofs (comportamento padrão)
# -U = Define as partições/discos por seus UUID e não por Label (Nomes).
```



#### Mudando_a_raiz_do_sistema

Vamos mudar a raiz do sistema, quando fizermos isso, é como trocar o Sistema Operacional, com isso, todo comando digitado após mudar a raiz terá efeito no sistemas em que estivermos e não mais no antigo.



Exemplo:

Nosso sistema hospedeiro é um Ubuntu, se mudarmos a raiz para um Debian, todo comando digitado terá efeito apenas no Debian e não mais no Ubuntu.



Para isso digite o comando abaixo:

```bash
arch-chroot /mnt /bin/bash

# /bin/bash informa qual shell queremos usar ao trocar a raiz.
```



#### Configurando_o_Relógio

Para isso temos algumas formas de se fazer, vamos configurar o link simbólico contendo as configurações da nossa localidade:

```bash
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```



Agora vamos atualizar a hora da BIOS para que ela tenha o mesmo horário que nosso sistema.

```bash
hwclock --systohc
```



#### Instalando_o_vim

```bash
pacman -Syy vim
```



#### Definir_idioma_no_S.O_do_Arch

Edite o arquivo abaixo para escolher o idioma do sistema, no nosso caso, o idioma Português Brasileiro:

```bash
sed -i 's/#pt_BR.UTF-8 UTF-8/pt_BR.UTF-8 UTF-8/' /etc/locale.gen

# Esse comando descomenta a linha '#pt_BR.UTF-8 UTF-8'
```

Gere as variáveis de localidade usando o comando `locale-gen`.



Crie o arquivo `locale.conf` dentro de `/etc/` para definir a variável `LANG` adequadamente:

```bash
echo "LANG=pt_BR.UTF-8" > /etc/locale.conf
```



#### Tornando_o_layout_do_teclado_permanente

```bash
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf
```



#### Configurando_nome_da_máquina

Edite o arquivo `echo "NomeDaMaquina" > /etc/hostname` adicionando o nome escolhido.



#### Configurando_o_arquivo_HOSTS

Edite o arquivo `vim /etc/hosts` 

```bash
127.0.0.1	localhost.localdomain	localhost
::1			localhost.localdomain	localhost
127.0.1.1	NomeDaMaquina.localdomain	NomeDaMaquina
```



#### Configurando_WIFI_Caso_tenha_um_adaptador_WIFI

Vamos instalar os pacotes necessários para configurar o WIFI:

```bash
pacman -S wireless_tools wpa_supplicant wpa_actiond dialog
```



#### Configurar_senha_do_ROOT

Use o comando abaixo para trocar a senha do root, coloque uma senha forte.

```bash
passwd
```



#### Instalando_a_multilib

A multilib é um pacote que cria uma compatibilidade de pacote que tem arquitetura x86 num sistema x64, para configurar ela, execute o comando abaixo:

```bash
sed -i '94,95 s/^#//' /etc/pacman.conf


# Esse comando descomenta as linhas abaixo, verificar se estão nas linhas 94 e 95.

[multilib]
Include = /etc/pacman.d/mirrorlist
```

Agora atualize o sistema com o comando `pacman -Syu` e `pacman -Syy`.



#### Instalando_o_GRUB

Para ter dual-boot precisamos instalar um pacote a mais, sendo ele o `os-prober`.

```bash
# Instalando o grub
pacman -S grub

# Para instalar o os-prober execute o comando abaixo:
pacman -S os-prober
```



#### Configurando_o_GRUB

Para aplicar o `grub` na nossa distro rode o comando abaixo:

```bash
grub-install /dev/sda
# instalar o grub na MBR do /dev/sda

grub-mkconfig -o /boot/grub/grub.cfg
# Exportando a configuração do grub para um arquivo que será lido no momento da inicialização.
```



#### ACPI/ACPID_para_NOTEBOOKS

Caso esteja usando um notebook, instale os pacotes `acpi` e `acpid` para controle da bateria do notebook.

```bash
pacman -S acpi acpid
```



#### Saido_do_nosso_sistema

Aperte `Ctrl + d` para sair da raiz do nosso sistema.



#### Desmontando_nosso_Sistema

No terminal digite `umount -R /mnt` para desmontar tudo daquele diretório.



#### Reiniciando_a_máquina

Use o comando `reboot` para reinicializar a máquina e remova a imagem bootavel do Arch.

