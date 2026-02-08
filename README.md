# Detecção de Placas de Trânsito Circulares

Trabalho final da disciplina de **Processamento de Imagens** — detecção automática de placas de trânsito circulares com borda vermelha em imagens reais, utilizando exclusivamente técnicas clássicas de PI.

## 🎥 Vídeo de apresentação

> **Link:** <!-- COLE O LINK DO YOUTUBE AQUI -->

## Objetivo

Dado uma imagem de entrada contendo (ou não) uma placa de trânsito circular com borda vermelha, o pipeline deve:

1. Localizar a placa na imagem.
2. Desenhar o círculo detectado sobre a imagem original.
3. Rejeitar falsos positivos (objetos vermelhos não circulares, triângulos, retângulos, etc.).

## Pipeline de processamento

O notebook `trabalho_final_unico.ipynb` implementa as seguintes etapas:

```
Imagem RGB
    │
    ▼
Conversão RGB → HSV
    │
    ▼
Segmentação por cor vermelha (duas faixas de Hue)
    │
    ▼
Pré-processamento morfológico (abertura + fechamento)
    │
    ▼
Componentes conectados (rotulagem + filtragem por área)
    │
    ▼
Transformada Circular de Hough (CHT) por componente
    │
    ▼
Validação geométrica (7 critérios simultâneos)
    │
    ▼
Seleção do melhor círculo + visualização de debug
```

## Técnicas utilizadas

| Etapa | Técnica | Referência da disciplina |
|---|---|---|
| Conversão de cor | RGB → HSV | Aula 26 — Cores |
| Segmentação | Limiarização no espaço HSV | Aula 26 — Cores |
| Limpeza da máscara | Abertura e fechamento morfológico | Morfologia matemática |
| Detecção de círculos | Transformada Circular de Hough (CHT) | Notebook `cht-skimage`, AP04 |
| Validação | Circularidade, cobertura angular, variância radial, etc. | Métricas geométricas clássicas |

## Critérios de validação (7 métricas)

| Métrica | Limiar | O que rejeita |
|---|---|---|
| `filled_circularity` ≥ 0.75 | Forma não circular | Triângulos (~0.60) |
| `angular_coverage` ≥ 0.60 | Distribuição não uniforme | Arcos parciais |
| `radial_cv` ≤ 0.30 | Distâncias irregulares ao centro | Formas alongadas |
| `border_coverage` ≥ 0.55 | Pixels fora do anel | Formas que não encaixam |
| `ring_fill_ratio` ≤ 0.72 | Disco sólido | Placa PARE totalmente vermelha |
| `eq_radius_error` ≤ 0.40 | Tamanho inconsistente | Raio CHT ≠ raio real |
| `score` ≥ 0.16 | Confiança baixa da CHT | Fits fracos |

## Estrutura do repositório

```
├── trabalho_final_unico.ipynb   # Notebook principal (código + execução)
├── README.md                    # Este arquivo
├── data/                        # Imagens de entrada
│   ├── img01.jpg
│   ├── img02.jpg
│   └── ...
├── results/
│   └── debug_images/            # Figuras de debug geradas pelo pipeline
├── cht-skimage (1).ipynb        # Notebook de referência da disciplina (CHT)
├── AP04_Givaldo_Neto.ipynb      # Atividade prática 04 (Hough manual)
└── trabalho-final.pdf           # Enunciado do trabalho
```

## Como executar

### Opção 1 — Google Colab (mais simples)

1. Abra o notebook `trabalho_final_unico.ipynb` no [Google Colab](https://colab.research.google.com/).
2. No painel lateral do Colab, crie a pasta `data/` e faça upload das imagens de teste para dentro dela.
3. Execute todas as células sequencialmente (Runtime → Run all).
4. Os resultados de debug serão exibidos no próprio notebook e salvos em `results/debug_images/`.

> **Nota:** As imagens não são versionadas no repositório. É necessário fazer upload manual no Colab a cada sessão.

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

3. Coloque as imagens de teste na pasta `data/`.

4. Abra e execute o notebook:
   ```bash
   jupyter notebook trabalho_final_unico.ipynb
   ```

5. Execute todas as células sequencialmente. Os resultados de debug serão salvos em `results/debug_images/`.

## Restrições atendidas

- ✅ Sem OpenCV
- ✅ Sem PIL / Pillow
- ✅ Sem aprendizado de máquina
- ✅ Apenas `numpy`, `matplotlib`, `scikit-image` e biblioteca padrão do Python

## Autor

**Givaldo Neto**

Disciplina de Processamento de Imagens