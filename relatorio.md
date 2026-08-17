# laboratório de análise de Segurança de redes

## 1. objetivo

Criar um ambiente de estudos para compreender, na prática, conceitos
fundamentais de redes de computadores e segurança da informação
utilizando uma máquina virtual Linux.

## 2. ambiente

- Sistema operacional: Ubuntu Linux
- Ambiente de virtualização: VirtualBox
- Interface de rede: enp0s3
- Configuração de rede: NAT

## 3. Identificação da máquina

Usuário: daniel
Hostname: daniel-Virtualbox

## 4. configuração de rede

Endereço IP: 10.0.2.15/24
Rede: 10.0.2.0/24
Gateway: 10.0.2.2

## 5. Análise inicial

A través dos comandos `ip addr` e `ip route`, foi possível identificar a 
interface de rede ativa, o endereço IP da máquina, a rede local e o gateway
utilizado para comunicação externa.

Essa etapa permitiu compreender de forma prática como o sistema operacional
identifica e encaminha o tráfego de rede. 

## 6. Inventario e análise dos serviços de rede

Foi realizado um levantamento dos serviços de rede em execução no sistema
utilizando os comandos **ss -tuln** e **sudo ss -tuln**. A análise permitiu
relacionar as portas em uso com seus respectivos processos e serviços.

Foram identificados os seguintes serviços:

*systemd-resolved: associado à porta 53 TCP/UDP e utilizado para resolução
de nomes DNS;

*Chrony: associado à porta 5353 UDP e utilizado para sincronização de data
e hora através do protocolo NTP;

*Avahi-daemon: associado à porta 5353 UDP e utilizado para descoberta de
serviços na rede local através mDNS;

*CUPS: associado à porta 631 TCP e utilizado para gerenciamento de serviços
de impressão.

O estado dos quatro serviços foi verificado através do **systemctl**. Todos se 
encontravam ativos ( **active** ) e habilitados ( **enabled** ) para inicialização automática.

Tambem foi analisado o endereço no qual cada serviço estava escutando. 
Alguns serviços, como CUPS e Chrony, apresentaram sockets associados ao
endereço de loopback, enquanto o Avahi estava associado às interfaces
de rede disponiveis para comunicação mDNS.

A atividade permitiu compreender que a presemça de uma porta em uso não
representa necessariamente uma vulnerabilidade. É necessário identificar
o processo responsável, compreender a finalidade do serviço e avaliar
sua configuração e exposição antes de determinar possíveis riscos de
segurança.

## 7. Tratamento de aviso no serviço Avahi

Durante a análise do serviço avahi-daemon, o **systemctl** apresentou um aviso
imformando que o arquivo da unidade havia sido alterado no disco e que a 
versão carregada pelo systemd estava desatualizada.

Antes de realizar qualquer alteração, foi investigada a unidade atraves dos
comandos **systemctl show** e **systemctl cat**, confirmando o arquivo utilizado 
pelo serviço e a existência da divergência indicada pelo systemd.

Após a análise, foi exevutado o comando:
**sudo systemctl daemon-reload**

Esse procedimento fez que o systemd recarregasse as definições das unidades
existentes no sistema.

Em seguida, o estado do avahi-daemon foi verificado novamente. O serviço 
permaneceu *active (running)* e o aviso anteriormente apresentado deixou de
aparecer.

A experiência reforçou a importância de não executar comandos de correção sem 
antes compreender a casuda do problema, seguindo a metodologia de:

**Identificar > Investigar > Compreender > Corrigir > Verificar.

## 8. Configuração e teste do firewall UFW

Como parte do processo de hardering do sistema, o firewall UFW foi ativado
com a politica padrão de negar conexões de entrada e permitir conexões de
saída. A configuração foi verificada utilizando os comandos **sudo ufw status verbose**
e **sudo ufw status numbered**.

Para compreender a relação entre firewall, portas e serviõs, foi criada temporariamente
uma regra permitindo conexões TCP na porta 22:

**sudo ufw allow 22/tcp**

Inicialmente, mesmo com a porta 22 permitida pelo firewall, não havia nenhum serviço
SSH escutando nessa porta. Isso demostrou que uma regra no firewall apenas autoriza
o tráfego, mas não inicia ou instala o serviço correspondente.

Depois, o OpenSSH Server foi instalado e o serviço SSH foi iniciado. Após a
inicialização, o comando **sudo ss -tulpn** mostrou o sshd escutando na porta 22 em
IPv4 (0.0.0.0:22) e IPv6 ( [::]:22).

Durante o teste de desactivação, foi observado que interromper apenas o ssh.service
não eliminou a porta 22 da lista de portas em escuta. A verificação mostrou que o 
ssh.socket continuava em **active (listerning)**. Após interromper também o socket
com **sudo systemctl stop ssh.socket**, a porta 22 deixou de aparecer no resultado
de **sudo ss -tulpn**.

Como o acesso remoto foi removido por SSH não é necessario neste ambiente de 
laboratorio, a regra temporária foi removida do UFW com o comando:

**sudo ufw delete allow 22/tcp**

Ao final do procedimiento, o UFW permaneceu ativo, a exceção para a porta 22 foi
removida e não havia serviço ou socket SSH escutando nessa porta. O teste permitiu
compreender, na prática, que a existência de uma regra de firewall e a existência
de um serviço escutando em uma porta são mecanismos diferentes e devem ser analisados
separadamente durante o handening de um sistema.

## 9. Atualizações automáticas e segurança do sistema

Durante o processo de hardening, foi verificado o funcionamento do serviço
**unattended-upgrades**, responsável pela execução automática de atualizações
configuradas no sistema. O serviço apresentou os estados **active (running)** e 
**enabled**.

também foi analisado o arquivo **/etc/apt/apt.conf.d/20auto-upgrades**, no qual as
opções de atualização da lista de pacotes e execução de atualizações automaticas estavam
habilitadas com o valor *"1"*.

Inicialmente, o comando **apt list --upgradable** indentificou 132 pacotes atualizáveis.
Após a atualização dos índices dos repositórios com **sudo apt update**, uma nova 
verificação mostrou apenas 3 pacotes pendentes: **libnautilus-extension4** , **nautilus**
e **nautilus-data**.

Ao executar **sudo apt upgrade**, foi identificado que esses três pacotes estavam
temporariamente retidos pelo mecanismo de atualizações graduais (phased updates) do 
Ubuntu. Por esse motivo, as atualizações não foram forçadas.

A verificação reforçou a importância de manter o sistema atualizado como medida de 
reduçao da exposição a vulnerabilidades conhecidas.

## 10. Auditoria final

Ao final do laboratório, foi realizada uma auditoria para verificar se as medidas
de hardening permaneceram aplicadas corretamente.

Durante a verificação, foi identificado que, embora o **ssh.service** estivesse inativo.
o **ssh.socket**. havia voltado ao estado ativo e mantinha a porta TCP 22 em escuta. isso
ocurreu porque anteriormente o socket havia sido apenas interrompido, sem ser deshabilitado
permanentemente.

A situação foi corrigida com a desativação do **ssh.socket**, seguida de uma nova
verificação. Ao final, tanto **ssh.service** quanto **ssh.socket** permaneceram inativos
e desabilitados, e a porta TCP 22 deixou de aparecer entre as portas em escuta.

A auditoria final também confirmou que o firewall UFW permaneceu ativo, com conexões
de entrada negadas por padrão e conexões de saída permitidas. O serviço **unattended-upgrades**
permaneceu ativo e habilitado, e a senha da conta root apresentou estado bloqueado **(L)**.

## Conclusão

O laboratório permitiu aplicar e verificar diferentes práticas básicas de hardening
em um sistema Ubuntu, incluindo análise de serviços e portas, configuração de firewall,
gerenciamento do SSH, atualizações automaticas e revisão de privilégios de usuários.

Além da execução dos comandos, os testes permitiram compreender a relação entre serviços,
sockets, portas e regras de firewall. A auditoria final demostrou também a importância de
validar as configurações após sua aplicação, pois um serviço aparentemente desativado
ainda pode possuir outros mecanismos capazes de manter uma porta em escuta

Como resultado, o sistema terminou o laboratório com uma superfície de exposição reduzida
e com mecanismos básicos de proteção verificados.

