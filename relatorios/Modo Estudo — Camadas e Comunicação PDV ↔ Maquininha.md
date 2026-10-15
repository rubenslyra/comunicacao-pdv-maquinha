# 📡 Modo Estudo — Camadas e Comunicação PDV ↔ Maquininha

---

## ✅ Primeira Resposta: Qual tipo de comunicação?

Estamos lidando com **dois segmentos distintos** — é essencial separá-los:

### 🟢 1️⃣ Segmento Local: PDV ↔ Maquininha
- **Escopo:** Rede de área local (LAN) ou conexão ponto a ponto
- **Protocolos mais usados:**
  - **TCP/IP** → Ethernet, Wi-Fi (portas comuns: 9000, 9100, 5000–5050)
  - **Protocolo serial/USB** → RS-232 emulada sobre USB (antigo, mas ainda usado)
  - **Bluetooth RFCOMM** → maquininhas sem fio
- **Característica:** Comunicação **direta, dentro da loja, sem sair para a internet**

### 🔵 2️⃣ Segmento Externo: Maquininha → Adquirente/Autorizador
- **Escopo:** Internet pública
- **Protocolos:** TCP/IP → **HTTPS/TLS**, protocolo específico da adquirente (ex: protocolo TEF, ISO 8583)
- **Característica:** Dados saem da rede local em direção ao serviço externo

> 🎯 **Resumindo:** A comunicação entre PDV e maquininha é **TCP/IP (camada de transporte) sobre rede local**. É isso que precisamos dominar primeiro.

---

## 🧩 Segunda Resposta: As Camadas para Diagnóstico de Problemas

Usamos o **Modelo OSI** — de **BAIXO PARA CIMA**. Sempre comece de baixo! Se a camada 1 falhar, tudo o resto não funciona.

### 🔽 Ordem de verificação — DO FUNDO AO TOPO:

---

### 📌 CAMADA 1 — Física
> *"O sinal existe? O cabo/conector está bom?"*

**O que verificar:**
- Cabos: danificados, mal conectados, trincados
- Portas: LED acende? Tem luz na interface de rede?
- Adaptadores USB/Serial: reconhecidos pelo sistema operacional?
- Energia: maquininha ligada, bateria com carga, fonte funcionando

**Sintomas típicos:** Sem sinal, dispositivo não aparece, LED apagado

---

### 📌 CAMADA 2 — Enlace
> *"Consegue se comunicar dentro da rede local?"*

**O que verificar:**
- Endereço MAC: está visível na rede? (`arp -a` no Windows/Linux)
- Switch/Hub: a porta está ativa, mesma VLAN?
- Conflito de MAC?
- Bluetooth: pareamento ativo?

**Comando útil:**
```bash
# Ver dispositivos na rede local
arp -a
```

**Sintomas:** IP configurado mas não responde, dispositivo some e aparece

---

### 📌 CAMADA 3 — Rede
> *"Consegue chegar até o outro dispositivo?"*

**O que verificar:**
- Endereço IP: correto, na mesma faixa da rede? Ex: PDV = 192.168.0.50 / Maquininha = 192.168.0.70
- Máscara de sub-rede: igual nos dois lados
- **PING:** PDV consegue pingar a maquininha?

**Comandos:**
```bash
ping 192.168.0.70          # IP da maquininha
ping -c 3 192.168.0.70     # Linux/Mac
```

**Sintomas:** "Destino inacessível", tempo esgotado, perda de pacotes

---

### 📌 CAMADA 4 — Transporte
> *"A porta está aberta e aceita conexões?"*

**O que verificar:**
- Porta TCP correta: ex: 9100, 9001, 8080 — conforme fabricante
- Firewall: está bloqueando a porta? No Windows, na maquininha, no roteador
- Serviço na maquininha: está rodando, ouvindo na porta?

**Comandos úteis:**
```bash
# Testar se a porta está aberta (Windows)
telnet 192.168.0.70 9100

# Ou com PowerShell
Test-NetConnection 192.168.0.70 -Port 9100
```

**Sintomas:** Conexão recusada, tempo esgotado na porta, "não foi possível conectar"

---

### 📌 CAMADAS 5, 6, 7 — Sessão, Apresentação, Aplicação
> *"Conectou, mas os dados não fazem sentido?"*

**O que verificar:**
- Protocolo correto: TEF? ISO 8583? JSON/HTTP?
- Codificação: ASCII? UTF-8? CP1252?
- Chaves de criptografia: TLS/SSL — certificados válidos?
- Formato da mensagem: campos na ordem certa, comprimento correto
- Timeout: muito curto para o processamento?

**Sintomas:** Conecta, mas a maquininha não responde ou responde com erro, mensagem ilegível, código de erro de protocolo

---

## 🗺️ FLUXO PRÁTICO DE DIAGNÓSTICO

```
❌ Problema: PDV não envia transação para maquininha
    ⬇️
1️⃣ Camada 1: Cabo bom? Maquininha ligada? ✅
    ⬇️
2️⃣ Camada 2: Dispositivo aparece na rede? arp -a ✅
    ⬇️
3️⃣ Camada 3: Ping funciona? ✅
    ⬇️
4️⃣ Camada 4: Porta está aberta? Test-NetConnection ✅
    ⬇️
5️⃣ Camada 7: Protocolo e dados corretos? ✅
```

> ⚠️ **Regra de Ouro:** Se o PING não funciona → pare nas camadas 1-3. Não perca tempo olhando código de aplicação!

---

## 📝 Resumo Rápido — Fixar na Memória

| Camada | Nome | Pergunta chave | Ferramenta |
|---|---|---|---|
| 1 | Física | Tem sinal/luz? | Olhar, testar cabo |
| 2 | Enlace | O dispositivo é visto? | `arp -a` |
| 3 | Rede | Chega até lá? | `ping` |
| 4 | Transporte | A porta está aberta? | `telnet` / `Test-NetConnection` |
| 5-7 | Aplicação | Fala a "mesma língua"? | Logs, monitoramento de rede |

