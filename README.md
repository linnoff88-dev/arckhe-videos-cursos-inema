# ðŸŽ¬ videos-cursos-inema

Skill + motor que transforma um **curso INEMA** (site estÃ¡tico, ex.: [FEP](https://inematds.github.io/FEP)) numa **sÃ©rie de vÃ­deos narrados e animados** â€” HTML â†’ MP4 via [HyperFrames](https://hyperframes.heygen.com), no padrÃ£o visual **dark premium Ã¢mbar** do INEMA, com narraÃ§Ã£o por voz clonada local e CTA pra **[INEMA.CLUB](https://inema.club)**.

> **PÃ¡gina / guia de uso:** https://inematds.github.io/videos-cursos-inema/

---

## O que ele faz

VocÃª aponta pra um curso INEMA e ele gera vÃ­deos em **3 nÃ­veis selecionÃ¡veis**:

| Parte | O que Ã© | DuraÃ§Ã£o | Voz |
|-------|---------|---------|-----|
| **1 Â· Landing** | 1 vÃ­deo de visÃ£o geral (a landing + as trilhas: o que Ã©, nÃºmeros, as N trilhas) | ~45â€“60s | `rachel` |
| **2 Â· Trilhas** | 1 vÃ­deo por trilha (mÃ³dulos + tÃ³picos de cada um); opcional 1 intro por mÃ³dulo | ~30â€“90s | `rachel` |
| **3 Â· ConteÃºdo completo** | **1 vÃ­deo profundo por MÃ“DULO** â€” aula completa que percorre os ~6 tÃ³picos, bem ilustrada com SVG | ~3â€“4 min | `bella` |

> âš ï¸ A aula profunda Ã© **por mÃ³dulo, nÃ£o por tÃ³pico**. Um curso com 6 trilhas Ã— 6 mÃ³dulos = **36 vÃ­deos** profundos (nÃ£o 216).

## Por que assim

- **Timing Ã© fonte Ãºnica.** O `build.mjs` lÃª a duraÃ§Ã£o **real** de cada WAV de narraÃ§Ã£o â†’ Ã¡udio, animaÃ§Ã£o e legenda saem sempre sincronizados.
- **Trocar a voz = trocar sÃ³ o TTS.** Build, render, captions e CTA ficam idÃªnticos.
- **DeterminÃ­stico e local.** Fontes locais (`@font-face`, sem CDN), narraÃ§Ã£o local (inemavox), sem `Date.now()`/`Math.random()`/rede no render.

---

## Estrutura do repo

Este repo guarda **a skill completa** e a **pÃ¡gina-guia** â€” nada mais:

```
README.md                       # este arquivo
docs/index.html                 # landing + guia de uso (GitHub Pages)
skills/videos-cursos-inema/     # A SKILL COMPLETA (instalÃ¡vel) + projeto rodÃ¡vel
  â”œâ”€ SKILL.md                   # instruÃ§Ãµes da skill (Claude Code)
  â”œâ”€ engine.mjs                 # motor: tipos de cena + renderHTML()
  â”œâ”€ build.mjs Â· write-txt.mjs Â· tts-inemavox.mjs Â· fetch-fonts.mjs Â· voice-sample.mjs
  â”œâ”€ specs.mjs Â· specs-deep.mjs Â· render-all.sh   # specs de exemplo (curso FEP)
  â”œâ”€ package.json Â· hyperframes.json Â· meta.json Â· index.html   # config HyperFrames
  â”œâ”€ assets/   (spec-template.mjs, fonts/, deep-t*.mjs, txt/, audio/)
  â””â”€ references/   (extraÃ§Ã£o de conteÃºdo, ilustraÃ§Ãµes SVG, vozes, formato)
```

---

## Instalar a skill (Claude Code)

```bash
cp -r skills/videos-cursos-inema ~/.claude/skills/
# reinicie a sessÃ£o do agente; depois invoque com /videos-cursos-inema
```

A skill ensina o agente a extrair o conteÃºdo do curso, escrever os specs e rodar o pipeline. Detalhes em
[`skills/videos-cursos-inema/SKILL.md`](skills/videos-cursos-inema/SKILL.md).

---

## Como usar (pipeline por vÃ­deo)

Tudo roda **dentro de `skills/videos-cursos-inema/`**. Cada vÃ­deo Ã© uma entrada no objeto `VIDEOS`
(em `specs.mjs` para as Partes 1â€“2, `specs-deep.mjs` para a Parte 3). O id comeÃ§a sempre pelo slug do
curso (ex.: `fep-00-overview`, `fep-deep-t1-m1`).

```bash
cd skills/videos-cursos-inema

# uma vez por projeto: fontes locais
node fetch-fonts.mjs

# por vÃ­deo, na ordem:
node write-txt.mjs <id>            # narraÃ§Ãµes do spec -> assets/txt/<id>-sN.txt
node tts-inemavox.mjs <id> bella   # gera os WAVs (voz bella; use rachel p/ Partes 1-2)
node build.mjs <id>                # lÃª duraÃ§Ãµes reais e escreve index.html (16:9)
#   9:16 (Shorts/Reels):  node build.mjs <id> --vertical
npx hyperframes lint               # tem que dar 0 erros
npx hyperframes render --quality high --output renders/<id>.mp4
```

> **Onde tudo mora â€” default `~/projetos/output/<curso>-videos/`.** O projeto inteiro do curso (specs,
> `assets/`, `frames/`, `index.html` e os MP4 em `renders/`) fica nessa pasta Ãºnica dentro de `output/`,
> salvo vocÃª definir outro lugar â€” cada curso todo num lugar sÃ³, fÃ¡cil de achar.

### PrÃ©-requisitos

- **Node.js 18+** (fetch nativo, ESM) e **FFmpeg** (`ffmpeg` + `ffprobe`) no PATH.
- **HyperFrames CLI** via `npx hyperframes@0.6.91 â€¦` (fixado no `package.json`).
- **Daemon de voz inemavox / Chatterbox-VC** em `http://127.0.0.1:7860` (vozes `bella` / `rachel`).
  Override com `INEMAVOX_TTS_URL`. Alternativas (Kokoro grÃ¡tis, ElevenLabs) em `references/vozes.md`.

### Tipos de cena (em `engine.mjs`)

`title` Â· `topic` Â· `lead` Â· `bullets` Â· `cards` Â· `steps` Â· `term` Â· `compare` Â· `illus` (SVG) Â· `cta`

A **Ãºltima cena Ã© sempre `cta`** (INEMA.CLUB). Na Parte 3, invista nas cenas `illus` (SVG) â€” Ã© a alma da
aula profunda (ver `references/ilustracoes-svg.md`). Copie `assets/spec-template.mjs` como base.

---

## Estilo (house style INEMA)

Dark premium: fundo `#0D1321`, painel `#1D2D44`, borda `#3E5C76`, texto `#F0EBD8`, secundÃ¡rio `#748CAB`, **accent Ã¢mbar `#FFC300`**, cÃ³digo `#2EC4B6`. Fontes locais **Sora** (tÃ­tulos), **Inter** (corpo), **JetBrains Mono** (cÃ³digo). **CTA** (sempre a Ãºltima cena): `CONTINUA EM` + **INEMA.CLUB** + `ðŸŒ inema.club`.

---

Feito para o ecossistema **[INEMA.CLUB](https://inema.club)**.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe