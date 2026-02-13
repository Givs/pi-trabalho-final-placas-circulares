# Detecção de Placas de Trânsito Circulares

Trabalho final da disciplina de **Processamento de Imagens** — detecção automática de placas de trânsito circulares com borda vermelha em imagens reais.

## 🎥 Vídeo de apresentação

> **Link:** https://youtu.be/-NdLZKekGn8

## Objetivo

Dado uma imagem de entrada contendo (ou não) uma placa de trânsito circular com borda vermelha, o pipeline deve:

1. Segmentar as regiões vermelhas da imagem.
2. Isolar componentes conectados candidatos a placa.
3. Aplicar a Transformada Circular de Hough (CHT) em cada componente.
4. Refinar e validar geometricamente o melhor círculo de cada componente.
5. Selecionar o melhor círculo aprovado como resultado final.

## Pipeline de processamento

O notebook `trabalho_final_unico.ipynb` implementa as seguintes etapas:

```
Imagem RGB
    │
    ▼
Conversão RGB → HSV
    │
    ▼
Segmentação por cor vermelha (duas faixas de Hue + saturação + valor)
    │
    ▼
Pré-processamento morfológico (abertura + fechamento)
    │
    ▼
Componentes conectados (rotulagem + filtragem por área absoluta e relativa)
    │
    ▼
Transformada Circular de Hough (CHT) por componente
  (faixa de raios adaptativa ao tamanho do componente)
    │
    ▼
Refino local do melhor círculo (pequenos ajustes em centro e raio)
    │
    ▼
Validação geométrica (cobertura_perimetro + radial_cv)
    │
    ▼
Seleção do melhor círculo aprovado (maior score)
    │
    ▼
Visualização de debug (6 subplots + acumulador CHT por componente)
```

### Detalhamento das etapas

1. **Conversão RGB → HSV** — separa a informação de cor (Hue) da iluminação (Value), tornando a segmentação robusta a variações de brilho.

2. **Segmentação por cor vermelha** — limiarização multivariável no espaço HSV. O vermelho ocupa duas faixas de Hue (próximo de 0° e próximo de 360°), com filtragem adicional por saturação mínima (evita cinzas) e valor mínimo (evita pretos).

3. **Pré-processamento morfológico** — abertura (erosão + dilatação) remove ruídos pequenos; fechamento (dilatação + erosão) preenche lacunas e conecta regiões fragmentadas. Elemento estruturante: disco.

4. **Componentes conectados** — rotulagem por conectividade-8 (`skimage.measure.label`) separa regiões isoladas. Cada componente é filtrado por área mínima absoluta (150 px) e área relativa ao maior componente (12%), descartando ruído.

5. **CHT por componente** — para cada componente, extrai bordas por `máscara XOR erosão(máscara)` e aplica a Transformada Circular de Hough com faixa de raios proporcional ao raio equivalente do componente (0.55× a 1.35×). O espaço acumulador 3D (centro_x, centro_y, raio) vota nos centros mais prováveis.

6. **Refino local do círculo** — após escolher o melhor pico da CHT no componente, o algoritmo testa pequenas variações de centro e raio (janela local) para corrigir desalinhamentos leves. O critério de escolha prioriza maior aderência geométrica da borda ao círculo.

7. **Validação geométrica simples** — o círculo refinado é aceito apenas se passar em dois critérios:
   - **Cobertura por perímetro** (`cobertura_perimetro`): fração de pixels de borda próximos ao perímetro esperado do círculo (`2πr`), com tolerância radial.
   - **Dispersão radial** (`radial_cv`): coeficiente de variação das distâncias dos pixels de borda ao centro do círculo. Quanto menor, mais circular é o componente.

   Critério atual de aprovação:
   - `cobertura_perimetro >= 0.40`
   - `radial_cv <= 0.20`

8. **Seleção por score** — entre os candidatos aprovados na validação, o de maior score da CHT é retornado como resultado final.

### Parâmetros de validação (implementação atual)

- `tolerancia_borda = 3.0`
- `cobertura_borda_minima = 0.40` (aplicada como `cobertura_perimetro`)
- `radial_cv_maximo = 0.20`
- `janela_refino_centro = 3` (±3 px)
- `janela_refino_raio = 3` (±3 px)

## Técnicas utilizadas

| Etapa | Técnica | Função principal |
|---|---|---|
| Conversão de cor | RGB → HSV | `skimage.color.rgb2hsv` |
| Segmentação | Limiarização multivariável (H + S + V) | `segmentar_vermelho_hsv()` |
| Limpeza da máscara | Abertura e fechamento morfológico | `skimage.morphology.opening/closing` |
| Separação de regiões | Componentes conectados (conectividade-8) | `skimage.measure.label + regionprops` |
| Detecção de círculos | Transformada Circular de Hough (CHT) | `skimage.transform.hough_circle` |
| Refino geométrico | Busca local no melhor círculo da CHT | `refinar_circulo_por_borda()` |
| Validação geométrica | `cobertura_perimetro` + `radial_cv` | `calcular_metricas_circulo_borda()` |
| Visualização | Debug multi-painel + heatmap do acumulador + métricas | `matplotlib` |

## Limitações e possíveis melhorias

O pipeline atual **já realiza pós-processamento geométrico** para reduzir falsos positivos da CHT. Ainda assim, existem limitações práticas:

- **Limiar fixo por dataset** — os thresholds de HSV, morfologia e validação geométrica podem exigir ajuste ao mudar iluminação, câmera ou tipo de imagem.
- **Sensível a oclusão e cortes severos** — quando a borda vermelha está incompleta, o `radial_cv` pode aumentar e a cobertura cair, gerando rejeição.
- **Deformações de perspectiva** — placas muito inclinadas podem parecer elipses, reduzindo a aderência ao modelo circular.
- **Objetos vermelhos circulares não placa** — o método detecta circularidade geométrica, não o significado semântico da placa.

Melhorias futuras possíveis:

- Ajuste automático de limiares por estatística da própria imagem.
- Inclusão de métricas adicionais (por exemplo, cobertura angular explícita).
- Correção de perspectiva antes da validação circular.
- Etapa opcional de reconhecimento do conteúdo interno da placa (ex.: velocidade), caso o escopo permita.

## Estrutura do repositório

```
├── trabalho_final_unico.ipynb   # Notebook principal (pipeline completo)
├── README.md                    # Este arquivo
├── data/                        # Imagens de entrada (test cases)
│   ├── img01.jpg
│   ├── img05.jpg
│   ├── img06.jpg
│   ├── placa-velocidade-maxima-permitida.jpg
│   └── ...
├── results/
│   ├── debug_images/            # Figuras de debug geradas pelo pipeline
│   └── metrics.csv              # Métricas de execução por imagem
└── .gitignore
```

> As pastas `data/` e `results/` estão versionadas no repositório com as imagens de teste e os resultados gerados.

## Como executar

### Opção 1 — Google Colab (mais simples)

1. Abra o notebook `trabalho_final_unico.ipynb` no [Google Colab](https://colab.research.google.com/).
2. No painel lateral do Colab, faça upload da pasta `data/` com as imagens de teste.
3. Execute todas as células sequencialmente (**Runtime → Run all**).
4. Os resultados de debug serão exibidos no próprio notebook e salvos em `results/debug_images/`.

### Opção 2 — Localmente

**Requisitos:** Python 3.8+

1. Clone o repositório:
   ```bash
   git clone https://github.com/<usuario>/pi-trabalho-final-placas-circulares.git
   cd pi-trabalho-final-placas-circulares
   ```

2. Crie um ambiente virtual e instale as dependências:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install numpy matplotlib scikit-image notebook
   ```

3. As imagens de teste já estão na pasta `data/`. Para adicionar novas imagens, basta copiá-las para essa pasta.

4. Abra e execute o notebook:
   ```bash
   jupyter notebook trabalho_final_unico.ipynb
   ```

5. Execute todas as células sequencialmente. Os resultados serão salvos em `results/debug_images/`.

## Autor

**Givaldo Neto**

Disciplina de Processamento de Imagens
