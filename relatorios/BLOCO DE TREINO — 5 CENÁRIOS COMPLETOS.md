# 🎯 BLOCO DE TREINO — 5 CENÁRIOS COMPLETOS ✅

Instrução: Para cada cenário, tente responder **antes** de ver a solução. Identifique: **camada do problema**, **causa raiz**, **solução**, e se quiser praticar, pense: "como meu código OOP/SOLID trataria isso?" 💪

---

## 📋 CENÁRIO 1 — "PINPad não responde após atualização do Windows"

> **Situação:** Loja com PDV funcionando perfeitamente. Após atualização automática do Windows, na manhã seguinte, toda tentativa de pagamento retorna: **"Sem resposta do dispositivo TEF — verifique conexão"**. O cabo não foi mexido, a maquininha liga normalmente.

### 🤔 Seu Diagnóstico:
> Camada: ________ | Causa provável: ________ | Solução: ________

---

### ✅ Resposta Completo

| Item | Detalhe |
|---|---|
| **Camada do problema** | 🔌 Camada 2 — Enlace / Configuração de porta |
| **Causa raiz** | Atualização do Windows reinstalou o driver USB e **mudou a porta COM** que o PinPad estava usando. O PDV continua apontando para a porta antiga |
| **Solução** | 1. Gerenciador de Dispositivos → verificar qual COM está ativo<br>2. Atualizar configuração do PDV/TEF para a porta correta<br>3. Reiniciar serviço/middleware<br>4. ✅ Testar |
| **No código OOP** | `IComunicacaoTEF.Conectar()` deve validar e lançar exceção específica: *"Porta COM3 não encontrada — verifique configuração"*, em vez de genérico "falha na comunicação" |

---

## 📋 CENÁRIO 2 — "Transação demora 30 segundos e diz 'Sem resposta do host'"

> **Situação:** PDV envia transação → maquininha fica processando → depois de muito tempo aparece: **"91 — Sem resposta do host"**. Outros computadores da loja navegam na internet normalmente. Às vezes funciona, às vezes não.

### 🤔 Seu Diagnóstico:
> Camada: ________ | Causa provável: ________ | Solução: ________

---

### ✅ Resposta Completo

| Item | Detalhe |
|---|---|
| **Camada do problema** | 🌐 Camada 3 (Rede) + ⏱️ Camada 7 (Aplicação) |
| **Causa raiz** | **Perda de pacotes ou latência alta** entre a loja e o autorizador. A internet funciona, mas a rota específica para o IP da adquirente está instável. O autorizador pode até responder, mas depois do **timeout** configurado no middleware |
| **Solução** | 1. Testar com `tracert` ou `mtr` até o IP do autorizador → onde está a perda<br>2. Aumentar o **timeout** de resposta no middleware TEF<br>3. Configurar DNS estático em vez de DHCP automático<br>4. Se persistir → contatar provedor de internet ou adquirente |
| **No código OOP** | `ProcessadorPagamento` deve implementar **retentativa automática** com backoff exponencial e distinguir: *"Erro de rede temporário — tentando novamente"* vs *"Falha permanente — contate suporte"* |

---

## 📋 CENÁRIO 3 — "Valor aparece errado na maquininha: R$ 1,50 em vez de R$ 150,00"

> **Situação:** Venda de R$ 150,00 no PDV. Quando o valor aparece na maquininha, mostra **R$ 1,50**. Caixa percebe antes de passar o cartão. Isso acontece esporadicamente, não sempre.

### 🤔 Seu Diagnóstico:
> Camada: ________ | Causa provável: ________ | Solução: ________

---

### ✅ Resposta Completo

| Item | Detalhe |
|---|---|
| **Camada do problema** | 📄 Camada 7 — Aplicação (Formatação ISO 8583) |
| **Causa raiz** | **Erro de formatação do Campo 4 (Valor) no ISO 8583.** Esse campo é fixo: 12 dígitos sem vírgula. R$150,00 = `000000150000`. Se o código enviar `000000001500` → vira R$1,50. Faltou preencher com zeros à esquerda |
| **Solução** | 1. Corrigir função de formatação → sempre 12 dígitos<br>2. Adicionar **validação no PDV**: valor na maquininha == valor enviado<br>3. No fluxo: PDV pede confirmação visual do valor antes de liberar cartão |
| **No código OOP** | ✅ SRP: Criar uma classe `FormatadorValorISO8583` **exclusiva** para isso. Ninguém mais formata valor. Teste unitário obrigatório: `Assert.Equal("000000150000", Formatador.Formatar(150.00m))` |

---

## 📋 CENÁRIO 4 — "Código 55 — Senha inválida, mas cliente diz que digitou certo"

> **Situação:** Cliente digita a senha. Maquininha mostra: **"55 — Senha inválida"**. Cliente jura que está correta. Segunda tentativa funciona. Isso já aconteceu com outros clientes.

### 🤔 Seu Diagnóstico:
> Camada: ________ | Causa provável: ________ | Solução: ________

---

### ✅ Resposta Completo

| Item | Detalhe |
|---|---|
| **Camada do problema** | 📄 Camada 7 — Aplicação + 🔒 Segurança |
| **Causa raiz** | Duas causas mais comuns:<br>• ⚠️ Cliente confundiu teclado numérico (1/7 ou 3/9 invertidos em algumas maquininhas)<br>• ⚠️ Cartão de chip sujo/contato ruim → dados lidos com erro → validação falha antes mesmo de enviar a senha |
| **Solução** | 1. Pedir para o cliente **limpar o chip** com pano seco<br>2. Pedir para conferir cada dígito ao digitar<br>3. Orientar: "Se persistir, pode ser bloqueio por tentativas — contate seu banco"<br>4. ⚠️ **Nunca pergunte ou anote a senha!** |
| **No código OOP** | A classe `RespostaTransacao` pode ter um método auxiliar: `ObterOrientacaoUsuario()` que retorna instruções amigáveis para cada código: *"Código 55: Verifique se digitou corretamente"* |

---

## 📋 CENÁRIO 5 — "Venda aprovada na maquininha mas PDV registrou como cancelada"

> **Situação:** Comprovante da maquininha: **APROVADA — Código 00**. Mas no relatório de vendas do PDV, essa transação aparece como **Cancelada**. No fim do dia, caixa tem diferença no caixa: dinheiro entrou no banco mas não aparece no sistema.

### 🤔 Seu Diagnóstico:
> Camada: ________ | Causa provável: ________ | Solução: ________

---

### ✅ Resposta Completo

| Item | Detalhe |
|---|---|
| **Camada do problema** | 📄 Camada 7 — Aplicação (Confirmação e Conciliação) |
| **Causa raiz** | **Falha na confirmação de retorno.** A transação foi enviada → autorizada → resposta chegou ao middleware → mas antes de confirmar para o PDV, houve uma queda momentânea de energia/processo. PDV marcou como "desconhecido" e assumiu como falha |
| **Solução — Contingência Imediata** | 1. ✅ **Comprovante da maquininha = documento válido** → venda EXISTE<br>2. 🔍 Pesquisar no log do autorizador/middleware pelo número da transação<br>3. ✅ Se código 00 → **registrar manualmente a venda no PDV**<br>4. ⚙️ Implementar **conciliação automática noturna**: PDV compara vendas x arquivo da adquirente → ajusta diferenças sozinho |
| **No código OOP — Prevenção** | Criar classe `ControladorIdempotencia`: toda transação tem um ID único. Antes de processar, verifica: *"Essa transação já foi concluída?"* — evita duplicidade/descarte incorreto. Estado da transação: `Pendente → Aprovada → Confirmada` nunca pula etapas |

---

## 📝 TABELA DE CONSULTA — Códigos Mais Comuns

| Código | Significado | Ação Imediata |
|---|---|---|
| **00** | ✅ Aprovada | Concluir venda |
| **05** | ❌ Não autorizada | Cliente ligar no banco |
| **51** | 💳 Saldo insuficiente | Outra forma de pgto |
| **55** | 🔑 Senha inválida | Redigitar, limpar chip |
| **91** | ⏱️ Sem resposta/timeout | Verificar internet, repetir |
| **12** | ⚠️ Transação inválida | Rever dados, refazer |
| **70** | 🔧 Modo inválido | Recarregar configurações |

---

## 🧠 DESAFIO FINAL

> **"Quando a maquininha diz APROVADA = a transação existe. Se o PDV não recebeu a confirmação = problema de retorno, NÃO = transação cancelada."**

Guarde essa frase — vai te salvar em qualquer atendimento! 🎯
