# **Arch Linux Installation Guide - BIOS/DOS Mode**



#### Introdução

Atualmente muitos computadores já vem de fábrica com o suporte a BIOS e UEFI, a maioria das placas identifica o uso da BIOS como `BIOS legacy`, ou seja, BIOS legada, que vai deixar de existir. Para uso da BIOS em placas com UEFI, você deve selecionar o tipo de BIOS, entrando na configuração dela, algumas até mesmo exigem que seja desativado o `BIOS Secure Boot`.

O modo BIOS é usado por Máquinas virtuais, como a VirtualBox que costumo usar bastante e máquinas reais antigas, vale ressaltar que o uso do BIOS por padrão é vinculado a tabelas de partições do tipo MBR (ambos por serem antigos), que é usada na tabela de partição do tipo `msdos` ou somente `DOS` (as duas nomenclatura representam a mesma coisa) e o uso de UEFI é vinculado a tabelas de partições do tipo `GPT`., mas isso não é uma regra e você pode mudar isso.



Qual é melhor? 

Essa é uma pergunta que eu sempre digo, depende do seu hardware, o modo BIOS/MBR ou BIOS/DOS (mesma coisa) ainda é muito usado, mas você pode usar o modo BIOS/GPT, dependendo do tamanho do seu HD ou simplesmente se você quer usá-lo ou se você precisa de um número grande de partição e não quer ter partições estendidas, o bootloader tem que dar suporte, assim como o S.O instalado, por padrão todas as distros Linux suportam GPT e o grub vem fazendo um grande sucesso por sua compatibilidade.

 

Este tutorial descreve o uso do Arch Linux com o modelo BIOS/DOS, um outro será feito para BIOS/GPT e ainda terá outro sobre UEFI/GPT, eu só preciso de uma placa UEFI `:(`.



Se você tem dúvidas sobre GPT/DOS ou o que é uma tabela de partição, tipos de partições, recomendo que veja um pequeno tutorial no link: 



#### Após logar no Arch

Configure o teclado para o padrão ABNT2, caso esteja usando um teclado que tenha cedilha.

```bash
loadkeys br-abnt2
```



#### Particionando o disco

Vamos particionar o disco para que ele receba nosso Sistema Operacional.
Eu vou colocar o `/home` e `/boot` numa partição separada.

A Arch wiki recomenda que seja usada o ID de partição abaixo para discos DOS usando BIOS:

```
Partição	Nome							ID de partição: Descrição
/			Raiz do Sistema					83: Linux
/home		Diretório de Usuários			83: Linux
/boot		Diretório de Inicialização		83: Linux
swap		Diretório de Troca				82: Linux swap
```



Meu HD não possui nenhuma tabela de partição, portanto precisamos criar uma, caso o seu HD já tenha, certifique-se de que a tabela de partição seja do tipo DOS.

<span style="color:red">Esse procedimento irá destruir todos os dados de seu HD, recomendo que você faça backup do HD.



Vamos criar uma nova tabela do tipo DOS no nosso HD:

```sh
parted /dev/sda mklabel msdos

# Criando uma tabela de partição do tipo DOS.
```

Se você rodar o comando `fdisk -l /dev/sda | grep "Disklabel type"` você verá que o `Disklabel type` está como `dos`.



Agora vamos criar a partição do `/boot`, eu sempre deixo o `/boot` como `sda`, muitas distros não sobem o sistema se não for o `sda`, mesmo que você especifique, portanto, para evitar maiores problemas, mantenha o `/boot` como `sda` da sua máquina.



Criando a partição `/boot`:

```bash
echo -e "\nn\np\n1\n\n+5G\nw" | fdisk /dev/sda

# Esse comando cria a primeira partição com 5G de armazenamento.
```



Criando a partição de troca `swap`:

```bash
echo -e "n\np\n2\n\n+5G\nt\n2\n82\nw" | fdisk /dev/sda

# Esse comando cria a segunda partição com 5G de armazenamento para swap,
# ele também define o FS(File System) como SWAP.
```



Criando a partição `/home`:

```bash
echo -e "\nn\np\n3\n\n+5G\nw" | fdisk /dev/sda

# Esse comando cria a terceira partição com 5G de armazenamento.
```



Criando a partição Raiz `/`:

```bash
echo -e "\nn\np\n4\n\n\nw" | fdisk /dev/sda

# Esse comando cria a quarta e última partição,
# usando o espaço livre que sobrou do armazenamento.
```



Exibindo as partições criadas:

```bash
fdisk -l /dev/sda | grep -E "/dev/sd?"

Disk /dev/sda: 41 GiB, 44023414784 bytes, 85983232 sectors

/dev/sda1           2048 10487807 10485760   5G 83 Linux
/dev/sda2       10487808 20973567 10485760   5G 82 Linux swap / Solaris
/dev/sda3       20973568 31459327 10485760   5G 83 Linux
/dev/sda4       31459328 85983231 54523904  26G 83 Linux

# Perceba que o disco tem 41 GiB, temos 4 partições
# e apenas uma esta com o FS (file system) como SWAP (Código 82), 
# todos os outros possuem o FS como Linux (Código 83).
```



#### Entendendo o comando



Vamos entender como aquele comando de apenas 1 linha pode criar as partições e ainda definir o tipo de partição delas.

O que estamos fazendo é simulando a entrada no teclado, é como se uma pessoa estivesse digitando as informações, mas ao invés de termos alguém digitando, simulamos a entrada manual no comando `fdisk`.

Toda vez que o comando `echo` é usado com um redirecionador, nesse caso o pipe `|`, podemos simular a entrada manual. 

O `echo`  joga na tela a informação, o pipe pega essa informação e redireciona como entrada do próximo comando, só que isso não funciona com todos os comandos, há programas que identificam isso e bloqueiam, por saber que não é uma pessoa digitando, um exemplo disso é usando o `SSH`, ele nos retorna a seguinte mensagem: `Pseudo-terminal will not be allocated because stdin is not a terminal.`



Explicado como funciona, vamos as opções usadas, aqui usamos 3 comandos diferentes, um para criar as partições `sda1 e sda3`, outro para criar a partição `sda2` e o último para criar a partição `sda4`.



Primeiro comando:

```
Comando: echo -e "\nn\np\n1\n\n+5G\nw" | fdisk /dev/sda

Deixando melhor de visualizar:

echo -e "\n			--> Aqui podemos escolher o tabela de partição (DOS ou GPT).
		  n\n		--> Opção 'n', cria uma nova partição.
		  p\n		--> Opção 'p', escolhe essa partição como primária.
		  1\n		--> Opção '1', define o número da partição. 
		  \n		--> Aqui entra os blocos, deixe em branco que o sistema decide.
		  +5G\n		--> Tamanho da partição.
		  w"		--> Gravar a tabela no disco e sair.

A primeira opção podemos escolher entre g=GPT, G=SGI, o=DOS e s=Sun.

```



Segundo comando:

```
Comando: echo -e "n\np\n2\n\n+5G\nt\n2\n82\nw" | fdisk /dev/sda

Deixando melhor de visualizar:

echo -e  "n\n		--> Opção 'n', cria uma nova partição.
		  p\n		--> Opção 'p', escolhe essa partição como primária.
		  2\n		--> Opção '2', define o número da partição. 
		  \n		--> Aqui entra os blocos, deixe em branco que o sistema decide.
		  +5G\n		--> Tamanho da partição.
		  t\n		--> Muda o tipo de partição.
		  2\n		--> Qual partição você quer mudar.
		  82\n		--> Código para o tipo de partição SWAP.
		  w"		--> Gravar a tabela no disco e sair.

```

A tabela abaixo representa os códigos suportados pelo `fdisk`:

```
 0  Empty           24  NEC DOS         	81  Minix / old Lin 	bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 	82  Linux swap / So 	c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          	83  Linux           	c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  	84  OS/2 hidden or  	c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     	85  Linux extended  	c7  Syrinx         
 5  Extended        41  PPC PReP Boot   	86  NTFS volume set 	da  Non-FS data    
 6  FAT16           42  SFS             	87  NTFS volume set 	db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          	88  Linux plaintext 	de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 	8e  Linux LVM       	df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 	93  Amoeba          	e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      	94  Amoeba BBT      	e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 	9f  BSD/OS          	e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            	a0  IBM Thinkpad hi 	ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux 	a5  FreeBSD         	eb  BeOS fs        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      	a6  OpenBSD         	ee  GPT            
10  OPUS            55  EZ-Drive        	a7  NeXTSTEP        	ef  EFI (FAT-12/16/
11  Hidden FAT12    56  Golden Bow      	a8  Darwin UFS      	f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk    		a9  NetBSD          	f1  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       	ab  Darwin boot     	f4  SpeedStor      
16  Hidden FAT16    63  GNU HURD or Sys 	af  HFS / HFS+      	f2  DOS secondary  
17  Hidden HPFS/NTF 64  Novell Netware  	b7  BSDI fs         	fb  VMware VMFS    
18  AST SmartSleep  65  Novell Netware  	b8  BSDI swap       	fc  VMware VMKCORE 
1b  Hidden W95 FAT3 70  DiskSecure Mult 	bb  Boot Wizard hid 	fd  Linux raid auto
1c  Hidden W95 FAT3 75  PC/IX           	bc  Acronis FAT32 L 	fe  LANstep        
1e  Hidden W95 FAT1 80  Old Minix       	be  Solaris boot    	ff  BBT

```



Terceiro comando:

```
Comando: echo -e "\nn\np\n4\n\n\nw" | fdisk /dev/sda

Deixando melhor de visualizar:

echo -e  "\n
		  n\n		--> Opção 'n', cria uma nova partição.
		  p\n		--> Opção 'p', escolhe essa partição como primária.
		  4\n		--> Opção '2', define o número da partição. 
		  \n		--> Aqui entra os blocos, deixe em branco que o sistema decide.
		  \n		--> Tamanho da partição.
		  w"		--> Gravar a tabela no disco e sair.

Quando cria uma nova tabela, ele automaticamente define o FS como Linux,
código 83.

```





#### Formatar as partições

Vamos formatar as partições com o *FS* para que elas recebam os dados adequado de cada partição.

```bash
# Formata a primeira partição com o FS ext4
mkfs.ext4 /dev/sda1

# Formata a terceira partição com o FS ext4
mkfs.ext4 /dev/sda3

# Formata a quarta partição com o FS ext4
mkfs.ext4 /dev/sda4

```



Formatar partição swap:

```bash
mkswap /dev/sda2

```



Ligar swap:

```
swapon /dev/sda2

```



Montar a partição raiz para iniciar o processo de instalação:

```bash
mount /dev/sda4 /mnt/

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
mount /dev/sda3 /mnt/home

# Montando a partição que receberá a pasta BOOT
mount /dev/sda1 /mnt/boot

```



#### Instalação

Usando o script `pacstrap`, vamos instalar um grupo de pacote base para o funcionamento do sistema:

```bash
pacstrap /mnt base base-devel

# base = Base do linux que será instalado.

# base-devel = Programas mais voltado para desenvolvedores, aqui temos compiladores, compactadores, bibliotecas do GNU e alguns programas como: sed, grep, sudo, systemd entre outros pacotes.

```

Esse grupo não inclui todas as ferramentas da instalação *live*, tal como [btrfs-progs](https://www.archlinux.org/packages/?name=btrfs-progs) ou firmware de rede sem fio específico.



#### Configurar o sistema

Gerar o arquivo fstab com os discos montados, isso permite que sempre que o sistema inicializar, ele monte as partições/discos.

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab

# -p = Excluir montagens do pseudofs (comportamento padrão)
# -U = Define as partições/discos por seus UUID e não por Label (Nomes).

```



#### Mudando a raiz do sistema

Vamos mudar a raiz do sistema, quando fizermos isso, é como trocar o Sistema Operacional, com isso, todo comando digitado após mudar a raiz terá efeito no sistemas em que estivermos e não mais no antigo.



Exemplo:

Nosso sistema hospedeiro é um Ubuntu, se mudarmos a raiz para um Debian, todo comando digitado terá efeito apenas no Debian e não mais no Ubuntu.



Para isso digite o comando abaixo:

```bash
arch-chroot /mnt /bin/bash

# /bin/bash informa qual shell queremos usar ao trocar a raiz.

```



#### Configurando o Relógio

Para isso temos algumas formas de se fazer, vamos configurar o link simbólico contendo as configurações da nossa localidade:

```bash
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

```



Agora vamos atualizar a hora da BIOS para que ela tenha o mesmo horário que nosso sistema.

```bash
hwclock --systohc

```



#### Instalando o vim

```bash
pacman -Syy vim

```



#### Definir idioma no S.O. do Arch

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



#### Tornando o layout do teclado permanente

```bash
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf

```



#### Configurando nome da máquina

Edite o arquivo `echo "NomeDaMaquina" > /etc/hostname` adicionando o nome escolhido.



#### Configurando o arquivo HOSTS

Edite o arquivo `vim /etc/hosts` 

```bash
127.0.0.1	localhost.localdomain	localhost
::1			localhost.localdomain	localhost
127.0.1.1	NomeDaMaquina.localdomain	NomeDaMaquina

```



#### Configurando WIFI - Caso tenha um adaptador WIFI

Vamos instalar os pacotes necessários para configurar o WIFI:

```bash
pacman -S wireless_tools wpa_supplicant wpa_actiond dialog

```



#### Configurar senha do ROOT

Use o comando abaixo para trocar a senha do root, coloque uma senha forte.

```bash
passwd

```



#### Instalando a multilib

A multilib é um pacote que cria uma compatibilidade de pacote que tem arquitetura x86 num sistema x64, para configurar ela, execute o comando abaixo:

```bash
sed -i '94,95 s/^#//' /etc/pacman.conf


# Esse comando descomenta as linhas abaixo, verificar se estão nas linhas 94 e 95.

[multilib]
Include = /etc/pacman.d/mirrorlist

```

Agora atualize o sistema com o comando `pacman -Syu` e `pacman -Syy`.



#### Instalando o GRUB

Para ter dual-boot precisamos instalar um pacote a mais, sendo ele o `os-prober`.

```bash
# Instalando o grub
pacman -S grub

# Para instalar o os-prober execute o comando abaixo:
pacman -S os-prober

```



#### Configurando o GRUB

Para aplicar o `grub` na nossa distro rode o comando abaixo:

```bash
grub-install /dev/sda
# instalar o grub na MBR do /dev/sda

grub-mkconfig -o /boot/grub/grub.cfg
# Exportando a configuração do grub para um arquivo que será lido no momento da inicialização.

```



#### ACPI/ACPID para NOTEBOOKS

Caso esteja usando um notebook, instale os pacotes `acpi` e `acpid` para controle da bateria do notebook.

```bash
pacman -S acpi acpid

```



#### Mudando a raiz do sistema

Aperte `Ctrl + d` para sair da raiz do nosso sistema.



#### Desmontando nosso Sistema

No terminal digite `umount -R /mnt` para desmontar tudo daquele diretório.



#### Reiniciando a máquina

Use o comando `reboot` para reinicializar a máquina e remova a imagem bootavel do Arch.

