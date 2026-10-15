# 📚 MODO ESTUDO — TEF na Camada de Aplicação + Cenário Real ✅

---

## 🧩 PARTE 1 — Como funciona o Protocolo TEF na Camada de Aplicação

### 📂 Arquitetura em 3 Camadas do TEF

```
┌─────────────────────────────────────────────────────────────┐
│ 🏪 PDV / Seu ERP (Sua Aplicação)                            │
│    ↓ Chama / envia dados                                    │
├─────────────────────────────────────────────────────────────┤
│ 🖥️ Middleware TEF / Cliente TEF (Ex: CliSiTef)              │
│    ↓ Formata mensagem → PinPad → Rede → Autorizador         │
├─────────────────────────────────────────────────────────────┤
│ 💳 PinPad / Maquininha de Pagamento                         │
└─────────────────────────────────────────────────────────────┘
```

> 🎯 **Ponto chave:** A sua aplicação **não fala diretamente com a maquininha**. Você fala com o **middleware TEF**, que cuida de toda a comunicação, formatação e criptografia

---

### 📤 Estrutura de uma Mensagem TEF

Cada fabricante tem seu padrão, mas a estrutura geral é:

```
┌─────────────────────────────────────────────────────┐
│ 🏷️ CABEÇALHO                                         │
│ • Código de Transação (ex: 020 = Venda Crédito)     │
│ • Identificação do Terminal / Loja                   │
│ • Número de Sequência (para controle)               │
├─────────────────────────────────────────────────────┤
│ 📦 CORPO DA MENSAGEM                                │
│ • Valor (formato: sem vírgula, ex: 150,50 = 0000015050)
│ • Tipo de Pagamento: Crédito/Débito/Pix             │
│ • Número de Parcelas                                │
│ • CNPJ da Loja / Código de Estabelecimento          │
├─────────────────────────────────────────────────────┤
│ 🔐 RODAPÉ / VERIFICAÇÃO                             │
│ • Código de Controle / CRC / Assinatura             │
└─────────────────────────────────────────────────────┘
```

### 🔄 Fluxo Completo da Transação — Camada de Aplicação

```
1️⃣ PDV → Middleware TEF
   "Inicia transação: R$ 150,50 — Crédito — 2x"
        ↓
2️⃣ Middleware → PinPad
   Envia valor formatado → mostra na tela do PinPad
        ↓
3️⃣ PinPad → Cliente
   Cliente insere cartão → digita senha
        ↓
4️⃣ PinPad → Rede/Autorizador
   Dados criptografados → Adquirente → Bandeira → Emissor
        ↓
5️⃣ Resposta volta:
   ✅ Código 00 = Aprovada
   ❌ Outros códigos = Negada / Erro
        ↓
6️⃣ Middleware → PDV
   Devolve status + comprovante + dados para NF
        ↓
7️⃣ PDV registra venda, imprime cupom, finaliza
```

---

### 📋 Códigos de Resposta Mais Comuns

| Código | Significado | Ação |
|---|---|---|
| **00** | ✅ Transação Aprovada | Sucesso! Finalizar venda |
| **05** | ❌ Não autorizada | Orientar cliente a ligar no banco |
| **12** | ⚠️ Transação inválida | Rever dados, refazer |
| **51** | 💳 Saldo insuficiente | Outra forma de pagamento |
| **70** | 🔧 Modo inválido | Recarregar tabelas/configuração |
| **91** | ⏱️ Sem resposta do host | Verificar internet, tentar novamente |
| **T22** | 🚫 Loja suspensa | Contato com adquirente |

---

## 🔧 PARTE 2 — Código com OOP + SOLID (Conceito Aplicado)

Vamos modelar o fluxo de forma limpa

```csharp
// ==============================================
// ✅ SRP — Responsabilidade Única
// A classe só carrega os dados da transação
// ==============================================
public class Transacao
{
    public string Id { get; set; }
    public decimal Valor { get; set; }
    public TipoPagamento Tipo { get; set; }
    public int Parcelas { get; set; } = 1;
    public string CodigoVenda { get; set; }
}

public enum TipoPagamento
{
    Debito, Credito, Pix
}

// ==============================================
// ✅ ISP — Interfaces Segregadas
// Contrato mínimo necessário
// ==============================================
public interface IProcessadorPagamento
{
    Task<RespostaTransacao> ProcessarAsync(Transacao dados);
}

public interface IComunicacaoTEF
{
    Task<RespostaTransacao> EnviarParaPinPadAsync(Transacao dados);
}

public class RespostaTransacao
{
    public bool Sucesso { get; set; }
    public string CodigoResposta { get; set; }
    public string Mensagem { get; set; }
    public string ComprovanteTexto { get; set; }
    public DateTime DataHora { get; set; }
}
```

---

## 🎯 PARTE 3 — CENÁRIO REAL DE PROBLEMA — Prática de Diagnóstico

### 📋 Cenário
> **Loja:** Farmácia Vila Velha  
> **Sistema:** PDV conectado via USB a PinPad SiTef  
> **Sintoma:** Ao tentar passar um cartão de crédito de R$ 89,90, aparece:  
> **"Falha na comunicação com o dispositivo TEF"**  
> O PDV está funcionando, internet está boa, outras coisas funcionam. Só o TEF não responde.

---

### 🔍 DIAGNÓSTICO — Passo a Passo (Camada por Camada)

| Passo | Camada | O que verificamos | Resultado |
|---|---|---|---|
| **1** | 🔌 Física | Cabo USB conectado? PinPad ligado? Luz acesa? | ✅ OK — Luz verde acesa |
| **2** | 🔗 Enlace | Windows reconhece o dispositivo? Gerenciador de dispositivos | ✅ OK — Aparece como "PinPad USB" |
| **3** | 🌐 Rede | Não aplicável (conexão USB direta) | — |
| **4** | 🚚 Transporte | Porta COM correta configurada no PDV? | ⚠️ **AQUI ESTÁ O PROBLEMA!** O PDV está configurado na COM3, mas o Windows alocou o PinPad na **COM4** |
| **5–7** | 📄 Aplicação | Middleware TEF consegue abrir comunicação? | ❌ Falha — porque não encontra a porta correta |

---

### ✅ SOLUÇÃO
1. Abrir Gerenciador de Dispositivos do Windows → Portas (COM/LPT)
2. Anotar qual COM o PinPad está usando
3. Abrir configuração do PDV → TEF → alterar para a porta correta
4. Reiniciar o middleware TEF
5. ➡️ **Testar transação — Funcionando! ✅**

---

### 🧠 Outros Cenários Comuns para Treinar

| Cenário | Mensagem Erro | Causa Provável | Solução Rápida |
|---|---|---|---|
| 1 | "Cancelado — Time Out Host" | Internet caiu ou adquirente lento | Verificar conexão; repetir |
| 2 | "Terminal Inválido" | CNPJ/terminal mal configurado no TEF | Corrigir CNPJ na configuração do SiTef |
| 3 | "Erro 42 — PinPad" | Firmware/DLLs desatualizados | Atualizar CliSiTef e reiniciar |
| 4 | "Erro 70 — Modo Inválido" | Tabelas/cartões não carregados | Enviar carga de tabelas pro PinPad |

---

## 📝 RESUMO PARA FIXAR

- **Middleware é o intermediário:** PDV → Middleware TEF → PinPad → Autorizador 🔁
- **Mensagem = Cabeçalho + Corpo + Rodapé de verificação**
- **Sempre diagnostique de baixo pra cima:** Física → Enlace → Rede → Transporte → Aplicação
- **Erro de aplicação ≠ erro de rede** — se ping funciona mas transação não, olhe logs e códigos de resposta
