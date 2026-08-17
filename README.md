Ubuntu Hardening Lab

Laboratório prático de hardening em Ubuntu Linux, desenvolvido para aplicar conceitos básicos de segurança de sistemas e administração Linux.
 Objetivo

Analisar e reduzir a superfície de exposição de uma máquina Ubuntu por meio da configuração de serviços, portas, firewall, atualizações e privilégios de usuários.
 Tecnologias utilizadas

    Ubuntu Linux
    VirtualBox
    UFW
    OpenSSH
    systemd / systemctl
    APT
    Git e GitHub

 Atividades realizadas

    Análise de portas e serviços ativos;
    Configuração e teste do firewall UFW;
    Testes controlados com SSH e porta TCP 22;
    Análise de ssh.service e ssh.socket;
    Verificação de atualizações automáticas;
    Análise de usuários e privilégios;
    Auditoria final do sistema.

 Aprendizados

O laboratório permitiu compreender na prática a relação entre serviços, sockets, portas e regras de firewall, além da importância de testar e validar cada alteração realizada durante um processo de hardening.

Um dos principais aprendizados foi identificar que interromper o ssh.service não necessariamente fecha a porta 22, pois o ssh.socket também pode permanecer ativo.
 Relatório

A documentação completa dos testes e resultados está disponível em relatorio.md.
