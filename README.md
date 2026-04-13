# 🎛️ Espectrograma Sonoro Interativo

**Aplicativo web para visualização e síntese de espectrogramas sonoros**  
Desenvolvido por **Prof. Ivan Simurra** · [NICS/UNICAMP](https://www.nics.unicamp.br)  
Disciplina: *Criação Musical com Novos Suportes Tecnológicos*

---

🔗 **[Abrir aplicativo → ivanSimurra.github.io/espectrograma-sonoro](https://ivanSimurra.github.io/espectrograma-sonoro/)**

---

## Sobre

Este aplicativo transforma o espectrograma — normalmente uma ferramenta de *análise* — em uma **superfície de criação sonora**. O estudante desenha livremente ou importa qualquer imagem, e o sistema sintetiza o resultado como som em tempo real.

O mapeamento é direto:

| Dimensão visual | Parâmetro sonoro |
|---|---|
| **Eixo X** (horizontal) | Tempo |
| **Eixo Y** (vertical) | Frequência (escala logarítmica) |
| **Brilho do pixel** | Amplitude / intensidade |
| **Posição do playhead** | Cursor de reprodução sincronizado com `AudioContext.currentTime` |

---

## Funcionalidades

### ✏️ Modo Desenho
- Pincéis coloridos com tamanho ajustável
- Modo **borracha** para apagar regiões
- Modo **linha** para criar harmônicos, glissandos e melodias em diagonal
- Paleta de 6 cores mapeadas a timbres visuais distintos

### 📂 Importação de Imagem
- Suporte a `.jpg`, `.png`, `.gif`, `.webp`
- A imagem é redimensionada para o canvas e tocada como espectrograma
- Qualquer imagem — rostos, texturas, pinturas — produz um resultado sonoro único

### ◈ Visualização Heatmap
Converte o canvas em representação científica de espectrograma com quatro paletas:

| Paleta | Descrição |
|---|---|
| **inferno** | Preto → roxo → laranja → amarelo (estilo Matplotlib) |
| **turbo** | Azul escuro → ciano → amarelo → vermelho |
| **viridis** | Roxo → teal → verde → amarelo |
| **ciano** | Fundo escuro com brilho azul-verde (estilo fala) |

### 🔊 Síntese Sonora
- Síntese **aditiva** via Web Audio API
- Leitura de pixels a cada 3 colunas × 4 linhas para desempenho
- Envelope **ADSR** por oscilador (ataque configurável)
- 4 tipos de onda: `sine`, `triangle`, `square`, `sawtooth`
- Playhead **sincronizado** com `AudioContext.currentTime` (sem deriva temporal)

### 📊 Monitor em tempo real
- Forma de onda (oscilloscope view)
- Barra de volume (VU meter)
- Frequência dominante da coluna atual
- Progresso percentual da reprodução

---

## Parâmetros de Síntese

| Parâmetro | Faixa | Padrão |
|---|---|---|
| Duração | 1 – 8 s | 3 s |
| Frequência base | 40 – 500 Hz | 80 Hz |
| Frequência topo | 500 – 10.000 Hz | 4.000 Hz |
| Tipo de onda | sine · triangle · square · sawtooth | sine |
| Tempo de ataque | 1 – 150 ms | 10 ms |

---

## Como usar

### Opção 1 — GitHub Pages (recomendado)
Acesse diretamente: **[ivanSimurra.github.io/espectrograma-sonoro](https://ivanSimurra.github.io/espectrograma-sonoro/)**

### Opção 2 — Localmente
```bash
# Clone
git clone https://github.com/ivanSimurra/espectrograma-sonoro.git
cd espectrograma-sonoro

# Abrir diretamente (sem servidor)
open index.html

# Ou com servidor local
python3 -m http.server 8080
# Acesse: http://localhost:8080
```

> ⚠️ Para importar imagens locais, é necessário usar um servidor HTTP (mesmo o `python3 -m http.server` resolve). Abrir o `index.html` diretamente via `file://` pode bloquear o FileReader em alguns navegadores.

---

## Arquitetura técnica

### Zero dependências externas

O aplicativo usa apenas APIs nativas do navegador — sem npm, sem frameworks, sem CDNs externos.

```
Web Audio API         → síntese (OscillatorNode, GainNode, AnalyserNode)
Canvas 2D API         → desenho e leitura de pixels (getImageData / putImageData)
requestAnimationFrame → animação do playhead
FileReader API        → carregamento de imagens locais
```

### Sincronização temporal

O principal problema de versões anteriores era a dessincronização entre o playhead visual e o som. A solução usa `AudioContext.currentTime` diretamente no loop de animação:

```javascript
function animate() {
  const elapsed = audioCtx.currentTime - playStartTime;
  const progress = Math.min(elapsed / duration, 1);
  
  // Posição exata baseada no relógio do AudioContext
  playhead.style.left = (YAXIS_WIDTH + progress * drawWidth) + 'px';
  
  requestAnimationFrame(animate);
}
```

### Mapeamento frequência ↔ posição Y (escala logarítmica)

```javascript
// Pixel → Frequência
function yToFreq(y) {
  const lo = Math.log(freqBase), hi = Math.log(freqTop);
  return Math.exp(lo + (1 - y / HEIGHT) * (hi - lo));
}

// Frequência → Pixel
function freqToY(f) {
  const lo = Math.log(freqBase), hi = Math.log(freqTop);
  return Math.round((1 - (Math.log(f) - lo) / (hi - lo)) * HEIGHT);
}
```

---

## Estrutura do repositório

```
espectrograma-sonoro/
├── index.html          ← aplicativo completo + documentação web
├── README.md           ← esta documentação
├── LICENSE             ← MIT License
└── docs/
    └── screenshots/    ← imagens de referência para o README
```

---

## Configurar GitHub Pages

1. Faça o push do repositório para o GitHub
2. Vá em **Settings** → **Pages**
3. Em *Source*, selecione **Deploy from a branch**
4. Branch: `main` · Folder: `/ (root)`
5. Clique em **Save**

O site ficará disponível em `https://<seu-usuario>.github.io/espectrograma-sonoro/` em alguns minutos.

---

## Contexto pedagógico

Este aplicativo foi criado para a disciplina **Criação Musical com Novos Suportes Tecnológicos** da UNICAMP, que cobre:

- **Semestre 1**: Acusmática + REAPER/Pure Data básico
- **Semestre 2**: Síntese sonora (AM, FM, RM, aditiva, subtrativa, granular)
- **Semestre 3**: Processamento espectral + espacialização + live electronics
- **Semestre 4**: Controle gestual, OSC, MIDI avançado

O espectrograma sonoro é utilizado como ponte entre **representação visual** e **síntese**, permitindo que estudantes de perfis heterogêneos explorem conceitos de:

- Síntese aditiva e timbral
- Relação frequência / altura musical
- Escala logarítmica da percepção auditiva
- Leitura e criação de espectrogramas
- Relação imagem–som (acusmática visual)

### Sugestões de atividades

**Atividade 1 — Reconhecimento espectral**  
Importe espectrogramas de sons conhecidos (vogais, instrumentos). Identifique formantes, harmônicos e ruído.

**Atividade 2 — Composição por desenho**  
Desenhe uma peça de 8 segundos usando apenas pincéis. Discuta as decisões composicionais.

**Atividade 3 — Imagem → Som**  
Importe uma fotografia do rosto de um compositor. Ouça o "rosto" como espectrograma (referência: Aphex Twin – *ΔMi−1 = −αΣDi[n]*[D_i^{−1}[n−1]]).

**Atividade 4 — Análise comparativa de paletas**  
Compare as mesmas informações espectrais nas quatro paletas (inferno, turbo, viridis, ciano). Discuta como a escolha cromática afeta a leitura.

---

## Referências

- [Web Audio API — MDN](https://developer.mozilla.org/pt-BR/docs/Web/API/Web_Audio_API)
- [Canvas API — MDN](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API)
- Wishart, T. (1994). *Audible Design*. Orpheus the Pantomime.
- Roads, C. (1996). *The Computer Music Tutorial*. MIT Press.
- Nierhaus, G. (2009). *Algorithmic Composition*. Springer.

---

## Licença

[MIT License](LICENSE) — uso livre para fins educacionais e de pesquisa.

---

<div align="center">
  <sub>NICS · Núcleo Interdisciplinar de Comunicação Sonora · UNICAMP · 2026</sub>
</div>
