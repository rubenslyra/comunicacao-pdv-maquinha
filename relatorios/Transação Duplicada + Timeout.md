# ✅ Bora resolver os dois AGORA — Transação Duplicada + Timeout!
São problemas que andam juntos na prática, então faz todo sentido fechar os dois de uma vez só. Vamos lá! 🚀

---

# 🎯 PARTE 1 — CENÁRIO: TRANSAÇÃO DUPLICADA

## 📋 Situação Real
> **Loja:** Supermercado Vila Velha  
> **O que acontece:** Cliente paga R$ 80,00 → PDV mostra "Erro de comunicação / Timeout" → Vendedor repete a transação → PDV diz "Aprovado" → No fim do dia: **2 cobranças no cartão do cliente, mas só 1 venda registrada no sistema** 😱  
> **Impacto:** Reclamação, estorno, prejuízo, insatisfação

---

### 🔍 Por que acontece? — A "Janela Escura"

```
1️⃣ PDV → Envia transação #123 — R$80,00
    ⬇️ ✅ Chega no autorizador
2️⃣ Autorizador → ✅ APROVA — valor é reservado
    ⬇️ ⏱️ A RESPOSTA demora mais que o tempo limite
3️⃣ PDV ⏱️ "Deu tempo esgotado — falhou!" ❌
    ❌ PDV NÃO sabe: JÁ FOI APROVADA!
4️⃣ Vendedor: "Vou passar de novo" → Envia transação #124 — R$80,00
    ⬇️ ✅ Autorizador aprova NOVAMENTE
5️⃣ Resultado: 2 cobranças | 1 venda registrada | Cliente insatisfeito
```

> 💡 **O problema:** O PDV acha que "não recebeu resposta = falhou", mas na realidade **a primeira já foi processada**. Chamamos isso de **ambiguidade de estado**.

---

### 🛡️ Como PREVENIR — 3 Pilares Essenciais

#### 📌 1️⃣ Idempotência — A REGRA DE OURO
> Uma transação com o MESMO ID Único **NUNCA** é processada duas vezes

```csharp
// ✅ SRP — Uma classe só para controlar duplicidade
public interface IRegistroTransacoes
{
    bool JaFoiProcessada(string idUnico);
    void MarcarComoProcessada(string idUnico, string codigoResposta = null);
}

// ✅ Antes de ENVIAR qualquer coisa:
if (_registro.JaFoiProcessada(transacao.IdUnico))
{
    // ✅ Já sabemos o resultado — devolve a resposta anterior
    return _registro.ObterRespostaAnterior(transacao.IdUnico);
}
```

**Na prática:** Sempre envie um **ID Único** (ex: `20261015-184230-001`) que NÃO se repita. Se precisar repetir a operação, **não mude o ID**. O autorizador/middleware reconhece: *"Já vi essa antes — aqui está a resposta"*.

---

#### 📌 2️⃣ Consulta de Status — Antes de Repetir
> Sempre PERGUNTE antes de refazer: *"Essa transação já existe lá?"*

```
❌ Ruim: Timeout → Envio de novo
✅ Bom: Timeout → Espero 2s → CONSULTO status → 
        Se já foi aprovada → uso aquela resposta
        Se realmente falhou → envio nova transação
```

---

#### 📌 3️⃣ Regra Humana — Treinamento da Equipe
> **"Se der erro de comunicação/TIMEOUT: NÃO REPITA IMEDIATAMENTE! Use a função CONSULTA/VERIFICA STATUS primeiro."**

- ✅ Consultar pelo número da transação ou valor
- ✅ Se confirmada → registrar venda com comprovante da maquininha
- ❌ Nunca passar o cartão de novo sem verificar

---

### ✅ Fluxo Correto — Na Prática

```
Venda R$80,00 → Enviar → ⏱️ Timeout!
    ⬇️
🔍 CONSULTAR: "Transação X foi processada?"
    ⬇️
✅ Sim, código 00 → Usar essa resposta → Venda concluída ✅
    OU
❌ Não existe → Enviar NOVA transação com ID diferente
```

---

## ⏳ PARTE 2 — TIMEOUT: O Que É, Por Que Acontece e Como Configurar

### 📌 O Que É Timeout?
> **"Esperei X segundos e não tive resposta — vou parar e avisar que falhou."**

É um **limite de segurança**. Sem ele, o sistema ficaria esperando para sempre se a rede cair.

---

### ⏱️ Tempos Típicos Recomendados

| Etapa | Tempo Recomendado | Por Que? |
|---|---|---|
| Conectar ao PinPad | 5–10s | Se não conecta rápido = problema físico |
| Enviar dados para maquininha | 3–5s | Processamento local é rápido |
| Espera de autorização (internet → autorizador) | **25–45s** | ⚠️ Não coloque menos que isso! Rede + autorizador demora |
| Consulta/Confirmação | 10–15s | Mais leve, deve responder rápido |

> ⚠️ **ERRO COMUM:** Colocar timeout de 10s na autorização → a transação foi aprovada mas o PDV já desistiu → **duplicidade na certa!**

---

### 🔧 Por Que Ocorre Timeout? (Causas Comuns)

| Causa | Explicação | Solução |
|---|---|---|
| 🌐 Internet instável | Pacotes se perdem no caminho | Verificar link; considerar link reserva |
| 📊 Autorizador ocupado | Horário de pico → banco/bandeira demora | Normal — aumentar timeout para 40s |
| 🖥️ Middleware sobrecarregado | Muitas transações ao mesmo tempo | Otimizar fila, reiniciar serviço periódico |
| ⚙️ Timeout muito curto | Configurado em 10s por padrão | ⚠️ MUDAR para 30–45s IMEDIATAMENTE |
| 🔄 DNS lento | Resolução de endereço atrasa tudo | Usar IP direto do autorizador se possível |

---

### 💻 Código — Tratamento Inteligente de Timeout

```csharp
public class ProcessadorTransacao
{
    private readonly int _tentativasMax = 2;
    private readonly TimeSpan _esperaEntreTentativas = TimeSpan.FromSeconds(3);

    public async Task<RespostaTransacao> ProcessarComSegurancaAsync(TransacaoVenda venda)
    {
        // ✅ 1. Verificar duplicidade ANTES de começar
        if (_registro.JaFoiProcessada(venda.IdUnico))
        {
            return _registro.ObterResposta(venda.IdUnico);
        }

        for (int tentativa = 1; tentativa <= _tentativasMax; tentativa++)
        {
            try
            {
                // ✅ 2. Enviar com timeout ADEQUADO
                using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(40));
                
                var resposta = await _integracaoTEF
                    .EnviarTransacaoAsync(venda, cts.Token);

                if (resposta.Sucesso)
                {
                    _registro.MarcarComoProcessada(venda.IdUnico, resposta.CodigoRetorno);
                    return resposta;
                }
            }
            catch (OperationCanceledException)
            {
                // ⏱️ Timeout — Não sabemos o resultado!
                _logger.Aviso($"Timeout — Tentativa {tentativa} para transação {venda.IdUnico}");
                
                // ✅ 3. CONSULTAR antes de tentar de novo
                await Task.Delay(_esperaEntreTentativas);
                var status = await _integracaoTEF.ConsultarStatusAsync(venda.IdUnico);
                
                if (status.Existe && status.Aprovada)
                {
                    _registro.MarcarComoProcessada(venda.IdUnico, "00");
                    return new RespostaTransacao
                    {
                        Sucesso = true,
                        CodigoRetorno = "00",
                        Mensagem = "Aprovada — confirmado via consulta após timeout"
                    };
                }
                
                if (tentativa == _tentativasMax)
                {
                    return new RespostaTransacao
                    {
                        Sucesso = false,
                        CodigoRetorno = "TIMEOUT",
                        Mensagem = "Sem resposta — VERIFIQUE STATUS antes de repetir!"
                    };
                }
            }
        }
        return FalhaGenerica();
    }
}
```

---

## 📝 RESUMÃO — FIXAR PARA SEMPRE

### 🚫 Duplicidade
- **Por que acontece:** Timeout + repetir sem consultar
- **Prevenção:** ID Único + Consulta antes de repetir + Regra de atendimento
- **Regra de ouro:** *"Timeout = CONSULTA primeiro, NÃO repete na hora!"*

### ⏳ Timeout
- **Não coloque menos de 30s** na etapa de autorização
- **Sempre consulta** quando acontece — pode já ter sido aprovada
- **Distinção importante:** Timeout ≠ Transação Negada — é "estado desconhecido"
