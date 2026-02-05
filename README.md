🤖 Mr. Robot – Relatório Técnico de Exploração
Plataforma: VulnHub | Ambiente Local

---

📋 Informações Gerais
| Item             | Descrição                                                 |
| ---------------- | --------------------------------------------------------- |
| Máquina          | Mr. Robot                                                 |
| Plataforma       | VulnHub                                                   |
| Ambiente         | Máquina virtual executada localmente                      |
| IP Alvo          | 192.168.122.248                                           |
| Sistema Atacante | Kali Linux                                                |
| Objetivo         | Comprometimento total do sistema (root)                   |
| Metodologia      | Reconhecimento → Enumeração → Exploração → Pós-exploração |
| Nível            | Iniciante / Intermediário                                 |

---

🎯 Escopo e Objetivo

O objetivo deste laboratório foi realizar a exploração completa da máquina Mr. Robot, partindo de um cenário de acesso externo sem credenciais até a obtenção de privilégios administrativos (root), documentando todas as decisões técnicas tomadas ao longo do processo.

Este writeup foi elaborado com foco em avaliação técnica, demonstrando raciocínio, metodologia e compreensão das vulnerabilidades exploradas.
---
🔎 1. Reconhecimento de Rede

A primeira etapa consistiu na identificação dos serviços expostos pelo host alvo
*print
Análise técnica:

- A porta 22 (SSH) encontra-se fechada, descartando acesso remoto direto.

- Apenas serviços web estão expostos (HTTP/HTTPS).

- Isso indica que a superfície de ataque está concentrada em aplicações web, direcionando o foco da exploração para enumeração HTTP.
---
🌐 2. Enumeração de Diretórios Web

Com base no reconhecimento inicial, foi realizada enumeração de diretórios para identificar recursos ocultos ou não indexados.
*print
Diretórios relevantes identificados:
/wp-login.php
/wp-admin/
/robots.txt
/license.txt
/wp-content
/blog
/readme
---
Análise técnica:
- A presença de /wp-login.php e /wp-content confirma o uso de WordPress.
- Ambientes WordPress mal configurados frequentemente permitem:
- enumeração de usuários
- exploração via editor de temas
- vazamento de informações sensíveis

🤖 3. Análise do robots.txt

Acessando o arquivo:
*print
Análise técnica:
- O arquivo robots.txt revelou recursos sensíveis explicitamente ocultados.
- O arquivo fsocity.dic aparenta ser uma wordlist customizada, útil para brute force ou enumeração de usuários.
- O arquivo key-1-of-3.txt representa uma das flags do desafi
---
🔐 4. Vazamento de Credenciais (Base64)
*print
Decodificação:
*print
Análise técnica:
- Credenciais armazenadas em Base64 representam ofuscação, não criptografia.
- Vazamento direto de usuário e senha caracteriza falha grave de segurança.
- As credenciais foram testadas no painel administrativo do WordPress.
---
🖥️ 5. Acesso Administrativo ao WordPress
*print
Credenciais válidas:
Usuário: elliot
Senha: ER28-0652
Impacto:
Acesso administrativo concede controle total sobre:
- edição de temas
- upload de arquivos
- execução de código PHP
Isso torna possível a obtenção de Remote Code Execution (RCE).
---
💣 6. Execução Remota de Código (RCE)
*print
Utilizando o editor de temas do WordPress foi inserido o seguinte payload PHP e simultaneamente, foi iniciado um listener na máquina atacante
*print
Resultado:
- Reverse shell obtida com sucesso.
- Usuário inicial: daemon
---
🐚 7. Estabilização da Shell

A shell obtida via Netcat apresentava limitações típicas:
- ausência de TTY
- comandos interativos instáveis
- falta de controle de terminal

Antes de tentar estabilizar a sessão, foi realizada enumeração para verificar a presença de interpretadores disponíveis:
*print
O Python estava disponível no sistema, permitindo o uso do módulo pty.
*print
Resultado:
- Shell totalmente interativa
- Ambiente estável para enumeração e escalonamento de privilégios
---
👤 8. Enumeração de Usuários e Acesso ao Usuário robot

Durante a enumeração do diretório /home, foi identificado o usuário robot.

Ao acessar o diretório:
cd /home/robot

Foi encontrado o arquivo:
password.raw-md5

Conteúdo:

robot:c3fcd3d76192e4007dfb496cca67e13b

Análise técnica:
- A hash utiliza o algoritmo MD5, considerado criptograficamente fraco.
- Foi realizada a quebra da hash utilizando base pública.

Senha recuperada:
*print

Login realizado:
*print 
---
🚀 9. Escalação de Privilégios – Root
Com acesso ao usuário robot, foi possível realizada enumeração de binários SUID:
*print find
Binário relevante identificado:
/usr/local/bin/nmap

Análise técnica:
Versões antigas do Nmap possuem modo interativo vulnerável.
Quando configurado como SUID, permite execução de comandos como root
Exploração:
*print
Escalação de privilégios concluída com sucesso.
---
🏁 Conclusão Técnica

Este laboratório demonstrou uma cadeia completa de comprometimento, explorando falhas comuns encontradas em ambientes reais:
- Exposição de credenciais
- WordPress mal configurado
- Execução remota de código
- Hash fraca (MD5)
- Binário SUID vulnerável
- Falhas de hardening no sistema
- O sistema foi comprometido integralmente, culminando em acesso root.
>>>>>>> 79563f5 (Relatório Técnico)
