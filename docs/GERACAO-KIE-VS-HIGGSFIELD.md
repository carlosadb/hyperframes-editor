# Camada de geração: Kie.ai × Higgsfield

Comparação feita em 08/11/2026, antes do piloto **023 Alma Lee** (vídeo de produto: catálogo real + B-roll gerado + fotos do catálogo). Preços mudam com frequência: conferir a página oficial no momento de gerar.

Critérios (do briefing do upgrade 0.9): custo por clipe útil, facilidade de o agente chamar, qualidade no nicho do cliente.

## Decisão

**Kie.ai para o piloto.** Pago por uso, créditos que não vencem, sem assinatura (regra do studio: "não assinar nada"), e o kit já tem cliente (`.claude/skills/motion-showreel/scripts/kie.mjs`). Higgsfield só volta à mesa se o volume de vídeo gerado for alto e recorrente, ou se o cliente pedir uma modelo virtual recorrente (Soul ID).

## Custo por geração (clipe de 5 s, 1080p)

Os dois revendem quase os mesmos modelos (Kling, Seedance, Veo, Nano Banana, Seedream): a qualidade vem do modelo, não da plataforma.

| Modelo | Kie.ai (1 crédito ≈ US$ 0,005) | Higgsfield (Plus: US$ 49 / 1.000 créditos/mês) |
|---|---|---|
| Seedance 2.0 | a partir de US$ 0,057/s → ~US$ 0,29 por tentativa (tarifa mínima; 1080p custa mais) | 60 créditos ≈ US$ 2,94 por tentativa |
| Kling 3.0 Turbo (o que o `kie.mjs` usa) | a partir de US$ 0,101/s → ~US$ 0,50–0,75 | não tabelado (Kling 3.0 aparece só como "7 dias ilimitado" em alguns planos) |
| Kling 4.0, image-to-video | 42,5 créditos/s = US$ 0,2125/s → US$ 1,06 por tentativa | não tabelado |
| Imagem (Nano Banana / Seedream) | US$ 0,02–0,09 por imagem | — |

Hipótese (não medida): ~3 tentativas por clipe aproveitável → Kie US$ 1–3 por clipe útil; Higgsfield US$ 4,50–9 no mesmo modelo.

Observação: a página do Kling 4.0 no Kie tem números conflitantes (o blog de lançamento cita 15 créditos/s em 1080p). Confirmar na hora.

## Facilidade de o agente chamar

- **Kie.ai:** REST com `KIE_API_KEY` no `.env`. Endpoints `jobs/createTask`, `jobs/recordInfo` e `chat/credit` (saldo), já usados pelo `kie.mjs`. Tarefa que falha não é cobrada. **Status 08/11: `.env` sem `KIE_API_KEY`.**
- **Higgsfield:** MCP oficial hospedado (`https://mcp.higgsfield.ai/mcp`, OAuth, sem chave) e CLI `@higgsfield/cli`. O MCP usa créditos do plano e exige plano pago; a API REST usa um saldo em dólar separado (créditos do plano não valem). Os termos tratam o MCP como Developer Access: a conta responde pelo que o agente gerar.

## Créditos e licença

- **Kie.ai:** créditos não vencem, sem assinatura. Termos de uso comercial não constam na página de preços: ler os Terms antes de entregar ao cliente.
- **Higgsfield:** créditos do plano não acumulam (renovam todo mês); créditos avulsos vencem em 90 dias. O plano Free não permite uso comercial; Starter, Plus e Ultra permitem.

## Qualidade no nicho (moda feminina / activewear)

- Fidelidade da peça real: o Kling 4.0 aceita até 10 imagens de referência + keyframes (primeiro, último e intermediários), o que ajuda a manter estampa e corte.
- Vantagens próprias do Higgsfield: Soul ID (mesma pessoa em várias cenas) e Marketing Studio (UGC: 70 créditos por 10 s em 720p). Não são essenciais para vídeo de produto a partir de fotos do catálogo.

## Próximo passo: teste pago (aguarda OK do Carlos)

1. Carlos cria a conta no Kie.ai, compra créditos e coloca `KIE_API_KEY` no `.env` (nunca commitar).
2. Usar 3 fotos reais do catálogo da Alma Lee e gerar 1 clipe de 5 s de cada no **Seedance 2.0** e no **Kling 4.0** (~6 tentativas, **US$ 4–8**). Mostrar o custo estimado antes de rodar.
3. Medir as tentativas por clipe útil e a fidelidade da peça → escolher o modelo do piloto 023.

## Fontes

- https://kie.ai/pricing · https://kie.ai/kling-4-0 · https://kie.ai/kling-3-0
- https://higgsfield.ai/blog/higgsfield-pricing-explained
- https://creatify.ai/blog/what-is-higgsfield-mcp-what-it-does-what-it-costs-and-how-to-use-it
- https://claudefa.st/blog/tools/mcp-extensions/higgsfield-mcp
- https://blog.segmind.com/higgsfield-credits-explained-expiry-rules-and-real-cost/
