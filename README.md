# fundamentos_IDS
# Fundamentos de IDS — Intrusion Detection System
> TryHackMe | Network Security  
> Área: Blue Team / SOC Analyst

---

## O que é um IDS?

O firewall atua como porteiro — controla o que entra e sai. Mas e se um atacante entrar por uma conexão aparentemente legítima e começar a agir de dentro?

É aí que entra o IDS (Intrusion Detection System). Ele funciona como as câmeras de vigilância do prédio, fica posicionado dentro da rede, monitorando o tráfego em busca de comportamentos suspeitos. A cada detecção, gera um alerta para os administradores.

**Diferença importante:** o IDS não bloqueia nada. Ele apenas detecta e notifica.

---

## Tipos de IDS por implantação

### HIDS — Host-based Intrusion Detection System
Instalado individualmente em cada host. Monitora apenas aquela máquina; processos, arquivos, logs do sistema.

- Vantagem: visibilidade detalhada de cada host
- Desvantagem: difícil de gerenciar em redes grandes, consome recursos em cada máquina

### NIDS — Network-based Intrusion Detection System
Instalado na rede, monitora o tráfego de todos os hosts de forma centralizada.

- Vantagem: visão geral de toda a rede em um único ponto
- Desvantagem: não tem visibilidade do que acontece dentro de cada host

---

## Tipos de IDS por método de detecção

### Baseado em assinaturas
Cada ataque deixa um padrão único, uma assinatura. O IDS mantém um banco de dados dessas assinaturas e compara o tráfego em tempo real contra elas.

- Vantagem: detecção rápida e precisa de ataques conhecidos
- Desvantagem: não detecta ataques de dia zero — se não tem assinatura no banco, passa despercebido

### Baseado em anomalias
Primeiro aprende o comportamento normal da rede (linha de base). Depois alerta quando há desvio desse comportamento.

- Vantagem: consegue detectar ataques de dia zero
- Desvantagem: gera muitos falsos positivos — qualquer comportamento incomum é marcado como suspeito, mesmo que seja legítimo

### Híbrido
Combina os dois métodos. Usa assinaturas para ataques conhecidos e anomalias para ataques novos.

- Melhor cobertura geral
- Recomendado para ambientes que precisam detectar tanto ameaças conhecidas quanto modernas

| Método | Detecta ataques conhecidos | Detecta dia zero | Falsos positivos |
|---|---|---|---|
| Assinatura | Sim | Não | Baixo |
| Anomalia | Sim | Sim | Alto |
| Híbrido | Sim | Sim | Médio |

---

## Snort — o IDS mais usado

O Snort é a solução de IDS open source mais popular do mundo, criada em 1998. Usa detecção por assinaturas e anomalias. Suas regras ficam em arquivos no diretório `/etc/snort/rules/`.

### Modos de operação

| Modo | O que faz | Quando usar |
|---|---|---|
| Packet Sniffer | Lê e exibe pacotes sem analisar | Diagnóstico de problemas de rede |
| Packet Logging | Registra o tráfego em arquivo PCAP | Investigação forense de ataques anteriores |
| NIDS | Monitora tráfego em tempo real e aplica regras | Monitoramento ativo da rede |

---

## Estrutura de uma regra Snort

Toda regra Snort segue este formato:

    [action] [protocol] [src_ip] [src_port] -> [dst_ip] [dst_port] (metadados)

Exemplo — detectar qualquer ping chegando na rede interna:

    alert icmp any any -> $HOME_NET any (msg:"Ping Detected"; sid:10001; rev:1;)

### Componentes da regra

| Campo | Descrição | Exemplo |
|---|---|---|
| action | O que fazer quando a regra disparar | `alert` |
| protocol | Protocolo do tráfego | `icmp`, `tcp`, `udp` |
| src_ip | IP de origem | `any` = qualquer IP |
| src_port | Porta de origem | `any` = qualquer porta |
| dst_ip | IP de destino | `$HOME_NET` = rede interna |
| dst_port | Porta de destino | `any`, `22`, `80` |
| msg | Mensagem exibida no alerta | `"Ping Detected"` |
| sid | ID único da regra | `10001` |
| rev | Número de revisão da regra | `1` |

---

## Criando e testando uma regra personalizada

**1. Editar o arquivo de regras:**

    sudo nano /etc/snort/rules/local.rules

**2. Adicionar a regra:**

    alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)
    
    <img width="1130" height="317" alt="image" src="https://github.com/user-attachments/assets/967da12a-7325-45d5-b609-49c47bad9d61" />


**3. Executar o Snort em modo NIDS:**

    sudo snort -q -l /var/log/snort -i lo -A console -c /etc/snort/snort.conf

**4. Testar a regra:**

    ping 127.0.0.1

**Saída esperada:**

<img width="1572" height="523" alt="image" src="https://github.com/user-attachments/assets/feea073b-20d0-4f05-948a-42f3855d4b9f" />


---

## Executando o Snort em arquivos PCAP

Para investigação forense de tráfego histórico:

    sudo snort -q -l /var/log/snort -r "arquivo.pcap" -A console -c /etc/snort/snort.conf

---

## Exercício prático — Análise de PCAP

**Cenário:** Uma empresa forneceu um arquivo `Intro_to_IDS.pcap` capturado durante um ataque. Executando o Snort no arquivo:

    sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A console -c /etc/snort/snort.conf

**Resultados:**

| Pergunta | Resposta |
|---|---|
| IP que tentou conexão SSH | `10.11.90.211` |
| Outras mensagens detectadas além de SSH | `Ping Detected` |
| SID da regra que detecta SSH | `1000002` |

<img width="1222" height="846" alt="image" src="https://github.com/user-attachments/assets/57f78dee-0776-4de7-93a4-201f0ad7e2f2" />

---

*TryHackMe | Network Security — Fundamentos de IDS*
