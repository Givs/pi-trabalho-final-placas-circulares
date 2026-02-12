# Detecção de Placas de Trânsito Circulares

Trabalho final da disciplina de **Processamento de Imagens** — detecção automática de placas de trânsito circulares com borda vermelha em imagens reais.

## 🎥 Vídeo de apresentação

> **Link:** <!-- COLE O LINK DO YOUTUBE AQUI -->

## Objetivo

Dado uma imagem de entrada contendo (ou não) uma placa de trânsito circular com borda vermelha, o pipeline deve:

1. Segmentar as regiões vermelhas da imagem.
2. Isolar componentes conectados candidatos a placa.
3. Aplicar a Transformada Circular de Hough (CHT) em cada componente.
4. Selecionar o círculo com maior evidência (score) como resultado.

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
Seleção do melhor círculo (maior score da CHT)
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

6. **Seleção por score** — dentre todos os círculos detectados, o de maior score (mais votos no acumulador) é selecionado como resultado final.

## Técnicas utilizadas

| Etapa | Técnica | Função principal |
|---|---|---|
| Conversão de cor | RGB → HSV | `skimage.color.rgb2hsv` |
| Segmentação | Limiarização multivariável (H + S + V) | `segmentar_vermelho_hsv()` |
| Limpeza da máscara | Abertura e fechamento morfológico | `skimage.morphology.opening/closing` |
| Separação de regiões | Componentes conectados (conectividade-8) | `skimage.measure.label + regionprops` |
| Detecção de círculos | Transformada Circular de Hough (CHT) | `skimage.transform.hough_circle` |
| Visualização | Debug multi-painel + heatmap do acumulador | `matplotlib` |

## Limitações e possíveis melhorias

O pipeline atual **não realiza pós-processamento** para validar se o círculo retornado pela CHT realmente corresponde a uma placa circular. A CHT sempre retorna picos no acumulador, mesmo para componentes que não são circulares (por exemplo, faixas vermelhas retangulares, partes de letreiros, objetos vermelhos genéricos). Isso significa que:

- **Falsos positivos são possíveis**: qualquer região vermelha com tamanho suficiente receberá um "melhor círculo", mesmo que a forma real não seja circular.
- **A filtragem depende da qualidade da segmentação**: se a segmentação HSV isolar bem apenas as bordas de placas, os componentes já são bons candidatos. Caso contrário, componentes espúrios podem gerar resultados incorretos.

Uma melhoria natural seria adicionar uma **etapa de pós-processamento / validação geométrica** que verificasse, por exemplo:
- Se a forma do componente é de fato circular (circularidade, cobertura angular).
- Se o raio detectado pela CHT é compatível com o tamanho real do componente.
- Se os pixels do componente estão distribuídos ao longo do perímetro do círculo (cobertura de borda).

Isso permitiria rejeitar contraexemplos como triângulos de sinalização vermelhos, faixas de texto ou objetos vermelhos não circulares que eventualmente passem pela segmentação.

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