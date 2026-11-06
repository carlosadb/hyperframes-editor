# Briefing: upgrade do HyperFrames 0.7.109 → 0.9.19

_Preparado em 2026-11-06 na sessão do Assistente-Pessoal. Decisão registrada em `Assistente-Pessoal/decisions/log.md` (06/11)._

## Objetivo

Atualizar o pacote `hyperframes` de **0.7.109** (14/08/2026) para **0.9.19** (06/11/2026) sem perder as customizações do kit (pipeline 2 estágios com video-use, cards da style-library, skills ajustadas) e sem mudar o resultado dos projetos de cliente sem perceber.

## Diagnóstico do changelog (252 releases lidas)

**Breaking changes: só 3, nenhuma nos afeta**

| Versão | Removido | Impacto aqui |
|---|---|---|
| 0.8.12 | `window.__hf.setAudioSolo` + controle "Hear only this" | Nenhum. `animation-map.mjs` usa só `__hf.seek` (mantido) |
| 0.8.156 | Agent tools/MCP stdio do `@hyperframes/inspector` | Nenhum, não usamos |
| 0.9.0 | Plan v1 do render distribuído (Lambda/Cloud Run), opção `planProtocol` | Nenhum, render é local |

**Mudanças silenciosas de comportamento (o risco real)**

- **0.8.35:** música de fundo sob voz passa a ser "carved" (ducking) por padrão, mais forte. A mixagem muda
- **0.8.38:** encode padrão muda para o preset "cleaner looks"; local renders usam o caminho de captura rápido
- **0.8.182:** áudio dos renders muda levemente (correção de pico no início)
- **0.8.14:** skills oficiais deixaram de ensinar 4 regras de composição que faziam o render falhar. As skills do Student Kit (set/2026) podem estar desatualizadas
- **0.9.9:** a CLI instala skills casadas com a própria versão e não faz mais auto-update. **Pode sobrescrever as skills customizadas** de `.claude/skills/` e `.agents/skills/`

**Ganhos relevantes**

- Studio: painéis dockáveis, seek bar, trim/split, snap, copiar/colar/duplicar, keyframes, speed ramps, fades com medidor, histórico completo com undo, export GPU (0.8.145) e GIF (0.8.205)
- Agente dentro do Studio via WebMCP (0.8.21 e 0.8.25): vê o frame, seleciona e edita ao vivo
- Windows: dezenas de correções (Chrome travado, `check` com máquina ocupada, render na raiz do drive, lint 30 s → 3 s); app HyperFrames para Windows (0.8.205)
- Render: `--start/--end` (0.8.222), motion blur, menos falhas por frame vazio e travamento em 90%, iPhone/celular mais estável
- `transcribe` grava SRT/VTT direto (0.8.144); Parakeet opcional (0.8.82)
- Planos pagos novos (cloud render, `templates pull`, billing). **Local continua grátis, não assinar nada**

## Estado atual do workspace (06/11)

- `package.json`: `hyperframes 0.7.109`, `gsap 3.14.2`, `playwright ^1.59.1`
- Último commit: `e81d822` merge do Student Kit set/2026
- **Não versionados:** `.claude/settings.json` e `.claude/skills/video-use/`. O `video-use` é a base do pipeline 2 estágios: **commitar antes de qualquer outra coisa**
- `skills-lock.json` registra o hash de cada skill vinda de `heygen-com/hyperframes`

## Plano de execução

1. **Proteger o que não está no git:** commitar `.claude/settings.json` e `.claude/skills/video-use/` em `master` (conferir antes que não há segredo no settings)
2. Criar a branch `upgrade-hyperframes-0.9`
3. Backup fora do repo: copiar `.claude/skills/hyperframes*`, `.claude/skills/gsap`, `.agents/skills/` e `skills-lock.json` para `E:\tmp\hf-skills-backup-2026-11-06\`
4. Antes do upgrade, renderizar ou localizar um **render de referência** do `041-enayram-hero-ad` (draft) para comparar depois
5. `package.json` → `"hyperframes": "0.9.19"`, rodar `npm install`
6. Validar:
   - `node scripts/check-kit.mjs`
   - `node scripts/preflight-all.mjs`
   - `npx hyperframes lint` e `npx hyperframes check` dentro de `video-projects/041-enayram-hero-ad/`
   - `npx hyperframes doctor`
7. Abrir o Studio (`npx hyperframes preview`, porta 3002) no 041: testar timeline, trim, keyframes, undo e o agente embutido
8. Render draft do 041 e **comparar com a referência**: áudio (ducking/volume), cor/encode, timing dos cards
9. **Skills:** NÃO rodar `hyperframes update` nem reinstalar skills às cegas. Comparar a versão oficial 0.9.19 das skills com as nossas (diff) e decidir uma por uma: manter a nossa, adotar a oficial ou mesclar. Atualizar `skills-lock.json` só no fim
10. Se tudo passar: merge em `master` e atualizar a memória `hyperframes-editor-workspace` (versão, data, achados). Se quebrar: descartar a branch, a 0.7.109 segue intacta em `master`

## Adendo: vídeo do Nate Herk (06/11)

Fonte: "Opus 5.5 Just Changed Video Editing Forever (free skills)" (https://youtu.be/7jHXoPGnA4c). Transcrição em `E:\Downloads\7jHXoPGnA4c.txt`.

1. **Atualizar o Student Kit junto com a 0.9 (passo 9b do plano).** Depois do passo 9, puxar a versão atual do Student Kit: a skill nova `motion-showreel` e os exemplos estão na comunidade gratuita do Nate na Skool (Classroom > YouTube resources / AI skills), não só no GitHub. Nosso fork é de 14/09. Mesma regra das skills oficiais: diff e decisão uma a uma, sem sobrescrever as customizações
2. **Camada de geração: comparar Kie.ai × Higgsfield antes do piloto.** O Nate usa Kie.ai (agregador pago por uso: Nano Banana para imagem → Kling para vídeo → de volta à composição). O Higgsfield tem MCP/CLI e ferramentas próprias (Soul ID, Marketing Studio), mas é assinatura com crédito que expira. Critério: custo por clipe útil, facilidade de o agente chamar, qualidade no nicho do cliente
3. **Primeiro caso real depois do upgrade:** vídeo de produto da **023 Alma Lee** no estilo do exemplo Lululemon (catálogo real via WooCommerce Store API + B-roll gerado + fotos do catálogo). Testar junto o ciclo **referência → skill**: analisar um anúncio bom da Biblioteca de Anúncios, transformar em skill de estilo (embrião da `ad-premium`) e refinar a cada uso

**Outras ideias anotadas (sem prioridade):**
- Mapa de beats com "quando eu disser X, entra Y" + descrição da emoção do vídeo como modelo de briefing
- Ciclo de verificação obrigatório (o agente tira prints e assiste o render antes de entregar), usando `snapshot`/`check` da 0.9
- Transcrição com ElevenLabs (`scripts/transcribe-elevenlabs.mjs` já existe; falta a chave da API)
- Novos pacotes de estilo: quadro branco desenhado, aula com câmera arredondada, liquid glass
- Vídeo de evento a partir de uma pasta de material bruto para a **037** (Imersão Moda & Negócio)
- Skill ScrollCraft (sites com animação no scroll) para o web-studio, só depois de passar pela `skill-security-audit`

## Definição de pronto (Gate 0)

- Lint/check/preflight sem erro no 041
- Studio abre e edita o 041
- Render draft do 041 sai com cards no tempo certo e áudio aceito por Carlos depois de ouvir lado a lado com a referência
- Skills customizadas preservadas (diff revisado)

## Pontos abertos

1. **Ducking automático** pode não servir para anúncio com trilha forte. Verificar se há como desligar por composição, se o resultado não agradar
2. **App HyperFrames para Windows:** avaliar depois do upgrade se vale usar no lugar do Studio no navegador
3. **Notas de release completas** baixadas em `%TEMP%\hfrel\` (temporário; fonte oficial: pasta `releases/` do repo `heygen-com/hyperframes`)
