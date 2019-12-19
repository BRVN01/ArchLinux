# <span style="color:#d86c00">**Customizando o Arch Linux**</span>

[TOC]

### <span style="color:#d86c00">**Introdução**</span>

<span style="color:#696969">Após a instalação do Arch, devemos fazer a instalação de alguns pacotes para que o sistema funcione mais redondo (tenha uma aparencia de desktop e não de um servidor), isso porque o Arch tem o propósito de ser um sistema enxuto, é de responsabilidade do usuário que usa o sistema instalar somente o que ele precisa e nada a mais.</span>

<span style="color:#696969">Vale ressaltar que as instalações feitas não irão respeitar isso, pois, na instalação de um pacote, muitas recomendações serão instaladas juntas, essas recomendações são chamadas de **Recommends**, descrito no [Chapter 4. Required files under the debian directory](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#control), essa recomendação são pacotes que geralmente são instalados juntos com nosso pacote principal e suas dependências, mas você pode alterar esse comportamento e instalar somente o essencial.</span>



<span style="color:#696969">Exmeplos de Recomendações num pacote:</span>

```
:: There are 38 members in group xfce4-goodies:
:: Repository extra
1) mousepad						2) orage		
3) ristretto					4) thunar-archive-plugin  	
5) thunar-media-tags-plugin  	6) xfburn  
7) xfce4-artwork 				8) xfce4-battery-plugin 		
9) xfce4-clipman-plugin 		10) xfce4-cpufreq-plugin 	
11) xfce4-cpugraph-plugin 		12) xfce4-datetime-plugin  
13) xfce4-dict 					14) xfce4-diskperf-plugin 		
15) xfce4-eyes-plugin 			16) xfce4-fsguard-plugin 	
17) xfce4-genmon-plugin 		18) xfce4-mailwatch-plugin 
19) xfce4-mount-plugin 			20) xfce4-mpc-plugin 			
21) xfce4-netload-plugin 		22) xfce4-notes-plugin 		
23) xfce4-notifyd 				24) xfce4-pulseaudio-plugin 
25) xfce4-screensaver 			26) xfce4-screenshooter 		
27) xfce4-sensors-plugin 		28) xfce4-smartbookmark-plugin 
29) xfce4-systemload-plugin 	30) xfce4-taskmanager 
31) xfce4-time-out-plugin 		32) xfce4-timer-plugin 			
33) xfce4-verve-plugin 			34) xfce4-wavelan-plugin 	
35) xfce4-weather-plugin 		36) xfce4-xkb-plugin

:: Repository community
37) parole  38) xfce4-whiskermenu-plugin
```

<span style="color:#696969">Esses são os pacotes recomendados do pacote **xfce4-goodies**, perceba que eu não preciso de todos os pacotes, como o *xfce4-screensaver* numero 25, eu posso remover esses pacotes ou fazer uma instalação personalizada, alias, essa é uma ação mais que recomendada, dessa forma, você não terá pacotes instalados do qual não vai usar.</span>



### <span style="color:#d86c00">**Instalação de aplicações necessárias**</span>

<span style="color:#696969">Vamos instalar algumas aplicações para que possamos prosseguir com a customização do sistema.</span>

<span style="color:#696969">Antes de iniciarmos a instalação dos pacotes, vamos atualizar o sistema:</span>

```bash
sudo pacman -Syu

# -S = Instala o pacote
# -y = Faz download de uma cópia atualizada do banco de dados dos principais pacotes, ele pega esses pacotes de  pacman.conf 
# -u = Faz upgrades dos pacotes.
```



<span style="color:#696969">Instalação dos pacotes:</span>

```bash
sudo pacman -S  git wget vim gedit python3 python-pip shotwell --noconfirm

# O Git é um sistema de controle de revisão distribuído,  rápido, escalável e com um conjunto de comandos incomumente 
# rico que fornece operações de alto nível e acesso total aos internos, vamos usar para baixar códigos fontes e compilar 
# alguns programas. 

# o wget é um gerenciador de download.

# O vim é um editor de texto em modo texto.

# gedit é um editor de texto em modo gráfico.

# Vamos instalar o python3 e um gerenciador de pacotes do próprio python (python-pip).

# Shotwell é um gerenciador de wallpaper.
```



### <span style="color:#d86c00">**Instalação de um ambiente gráfico**</span>

<span style="color:#696969">Para usar um ambiente gráfico precisamos instalar o X, segue uma descrição do que é o servidor X utilizados por ambientes gráficos:</span>

<span style="color:#696969">ENZOTIB.askubuntu. Disponível em \<https://askubuntu.com/questions/7881/what-is-the-x-server>. Acesso em 20 ago.  2019.</span>

> X é um aplicativo que gerencia uma ou mais exibições gráficas e um ou mais dispositivos de entrada (teclado, mouse, etc.) conectados ao computador.
>
> Ele funciona como um servidor e pode ser executado no computador local ou em outro computador na rede. Os serviços podem se comunicar com o servidor X para exibir interfaces gráficas e receber informações do usuário.
>
> Vale a pena notar, um componente comum usado com um servidor X é o Gerenciador de Janelas, um aplicativo que gerencia o redimensionamento e a movimentação de janelas e elementos decorativos de janelas, como barras de título, minimizar e fechar botões.
>
> O servidor X pode ser iniciado com o comando 'startx', ou mais comumente, de um gerenciador de exibição, como o gdm.



<span style="color:#696969">Para um Ambiente de Trabalho gráfico, nós temos algumas opções como: XFCE, GNOME, KDE, Deepin entre outros, esses são os mais usados, nesse tutorial, vamos utilizar o XFCE versão 4.</span>

```bash
sudo pacman -S xorg xfce4 xfce4-goodies --noconfirm

# XORG = Gerenciador de exibições gráficas.

# O xfce4-goodies nos fornece alguns plugins para podermos
# deixar o ambiente de trabalho mais agradável, 
# como por exemplo: Arte extra, Monitor de nível de bateria, 
# Extensão para data e hora, Mini lançadores entre outros.

# O xfce4 é o nosso ambiente de trabalho.

# --noconfirm faz com que ele não peça permissão e aceite tudo.
```



<span style="color:#696969">Para instalar o Deepin, siga as opções abaixo:</span>

```bash
sudo pacman -S xorg deepin deepin-extra

# XORG = Gerenciador de exibições gráficas.

# O deepin-extra nos fornece alguns plugins para podermos
# deixar o ambiente de trabalho mais agradável, 
# como por exemplo: Arte extra, Monitor de nível de bateria, 
# Extensão para data e hora, Mini lançadores entre outros.

# O deepin é o nosso ambiente de trabalho.

# --noconfirm faz com que ele não peça permissão e aceite tudo.


## Caso você use o Deepin, faça o próximo (Instalação do Gerenciador de Exibição)
## passo e volte para aplicar o comando abaixo:

# Adicionando a sessão do lightdm para deepin
echo "greeter-session=lightdm-deepin-greeter" >> /etc/lightdm/lightdm.conf
```



#### <span style="color:#d86c00">**Instalação do Gerenciador de Exibição**</span>

<span style="color:#696969">O gerenciador de exibição é um software que tem como função desenhar uma interface de login sobre o X, ou seja, o X cria uma ambiente gráfico e Display manager (Gerenciador de Exibição) desenha as interfaces de login.</span>

<span style="color:#696969">Você pode usá-lo sem o Xorg, mas para isso, você deve ter o um login automatico</span>

```bash
sudo pacman -S lightdm lightdm-webkit2-greeter lightdm-gtk-greeter --noconfirm

# Lightdm = Gerenciador de Exibição.

# lightdm-gtk-greeter = É uma GUI que solicita ao usuário credenciais, como: nome e senha, permite a troca de sessão, ambiente gráfico a ser usado entre outras coisas, chamamos isso de greeter.

# lightdm-webkit2-greeter = Um greeter que usa Webkit2 para temas. 
```



#### <span style="color:#d86c00">**Habilitando o serviço para iniciar com o Sistema**</span>

<span style="color:#696969">Após fazer a instalação de certas aplicações, precisamos ativar o serviço na inicialização do sistema, caso contrário, toda vez que o sistema iniciar, precisaremos iniciar esses serviços.</span>

```bash
# Reiniciando o serviço:
sudo systemctl restart lightdm

# Ativando o serviço na inicialização do Sistema:
sudo systemctl enable lightdm
```



### <span style="color:#d86c00">**Configurando a Rede**</span>

```bash
sudo pacman -S networkmanager network-manager-applet --noconfirm

# networkmanager = Fornece uma interface de alto nível para a configuração das interfaces de rede.

# network-manager-applet = Plugin de interface para gerenciar as conexões.

# Iniciando o gerenciador da rede
sudo systemctl start NetworkManager

# Habilitando para subir na inicialização dos sistema
sudo systemctl enable NetworkManager
```



### <span style="color:#d86c00">**Instalando o SSH**</span>

<span style="color:#696969">Vamos instalar o SSH para podermos nos conectar a nossa máquina caso necessário.</span>

```bash
# Instalando o SSH:
sudo pacman -S openssh --noconfirm

# Ativando o serviço do SSH na inicialização:
sudo systemctl enable sshd
```



### <span style="color:#d86c00">**Instalando o Firefox e o Flash**</span>

<span style="color:#696969">Vamos instalar o Firefox para que possamos navegar na Web, vamos instalar também o plugin do flash.</span>

```bash
# Instalando Firefox e o Flash:
sudo pacman -S firefox flashplugin pepper-flash --noconfirm

# flashplugin = Adobe Flash Player NPAPI

# pepper-flash = Adobe Flash Player PPAPI
```



### <span style="color:#d86c00">**Instalando pacotes para Impressora**</span>

<span style="color:#696969">Para podermos usar a impressora devemos instalar o [CUPS](https://pt.wikipedia.org/wiki/CUPS) (Sistema Comum de Impressão Unix).</span>

```bash
sudo pacman -S cups cups-pdf system-config-printer hplip --noconfirm

# cups = Sistema de Impressão.

# cups-pdf = Impressora PDF

# system-config-printer = Uma ferramenta de configuração de impressoras 
# e um applet de status do CUPS

# hplip = Biblioteca de Drivers para impressora HP.
```

<span style="color:#696969"><span style="color:#C0C0C0">**Observação**</span>: Ao importar uma impressora pela rede usando o parâmetro***ipp://IP***eu tive vários problemas ao imprimir, a impressão tinha caracteres estranhos e ficava imprimindo infinitamente.</span>

<span style="color:#696969">Esse problema foi corrigido ao importar uma impressora pela rede usando o parâmetro***socket://IP***, por padrão a maioria das distros ao encontrar uma impressora usando o*ipp*, o sistema salva como *socket*, mas isso não estava acontecendo.</span>



#### <span style="color:#d86c00">**Habilitando o serviço para iniciar com o Sistema**</span>

```bash
# Iniciando o serviço:
sudo systemctl start org.cups.cupsd 

# Ativando o serviço do SSH na inicialização: 
sudo systemctl enable org.cups.cupsd
```



### <span style="color:#d86c00">**Instalando o LibreOffice**</span>

<span style="color:#696969">Vamos instalar o LibreOffice para que possamos trabalhar com documentos, powerpoint entre outros tipos de documentos.</span>

```bash
sudo pacman -S libreoffice-still libreoffice-still-pt-br --noconfirm

# libreoffice-still = Pacote do LibreOffice.

# libreoffice-still-pt-br = Pacote de idioma Português Brasil.
```



### <span style="color:#d86c00">**Instalando um leitor de PDF**</span>

<span style="color:#696969">Vamos instalar um leitor de PDF para que possamos ler documentos em PDF.</span>

```bash
pacman -S evince --noconfirm
```



### <span style="color:#d86c00">**Instalando o Google-Chrome**</span>

<span style="color:#696969">Vamos instalar o Google Chrome.</span>

```bash
git clone https://aur.archlinux.org/google-chrome.git
cd google-chrome/
makepkg -si
```



### <span style="color:#d86c00">**Instalando o Typora**</span>

<span style="color:#696969">Vamos instalar um editor de MarkDown.</span>

```bash
git clone https://aur.archlinux.org/typora.git
cd typora/
makepkg -si
```



### <span style="color:#d86c00">**Instalando Codecs de audio**</span>

<span style="color:#696969">Vamos instalar o servidor de som, codecs de audio e o front-ent para controlarmos o som.</span>

```bash
sudo pacman -S mplayer --noconfirm

pacman -S pulseaudio-alsa pavucontrol --noconfirm

# pavucontrol = front-end para controlar o som.
# mplayer = Codecs de audio.
# pulseaudio-alsa = PulseAudio é um projeto de servidor de som em rede multi-plataforma.
```



### <span style="color:#d86c00">**Instalando o MTP, NFS e SMB**</span>

<span style="color:#696969">Para que possamos conectar nosso dispositivo USB (como um smartphone) no sistema, temos que instalar alguns pacotes para isso:</span>

```bash
sudo pacman -S gvfs-mtp gvfs-nfs gvfs-smb python-pysmbc smbclient

# gvfs-mtp = Virtual filesystem implementation for GIO (MTP backend; Android, media player)

# gvfs-nfs = GVfs vem com um conjunto de back-end, incluindo suporte a lixeira, SFTP, SMB, HTTP, DAV e muitos outros.

# smbclient = Cliente semelhante ao ftp para acessar recursos SMB/CIFS em servidores.
```



