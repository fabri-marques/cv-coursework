# Visao Computacional — Atividades

**Aluna:** Fabricia Cristina Marques Jung
**Disciplina:** Visao Computacional
**Instituicao:** IREDE/AVI

## Sobre

Repositorio com as atividades praticas da disciplina de Visao Computacional, utilizando um mini dataset com 2 classes (concha vs pedra, 5 imagens cada).

## Atividades

### Atividade I — Transformacoes de Imagem (`avi_transf/`)

Construcao do mini dataset e aplicacao dos fundamentos de Processamento de Imagens Digitais:

- Aquisicao controlada (variacao de iluminacao, angulo, distancia)
- Analise de resolucao (100%, 50%, 20%)
- Conversao de espacos de cor (RGB, HSV, escala de cinza)
- Quantizacao (256, 64, 32, 2 niveis)
- Comparacao de formatos (PNG lossless vs JPEG q50)

### Atividade II — Filtragem Espacial (`avii_filtros/`)

Aplicacao de tecnicas de filtragem para suavizacao, remocao de ruido e deteccao de bordas:

- Selecao de imagens com ruido real (ISO alto, baixa luminancia)
- Filtros de suavizacao: Media, Gaussiano, Mediana, Bilateral
- Deteccao de bordas: Sobel, Laplaciano, Canny
- Analise comparativa com metricas (PSNR, SSIM) e justificativa tecnica

## Estrutura

```
├── avi_transf/
│   └── visao_computacional_avi_.ipynb       # Notebook Atividade I
├── avii_filtros/
│   └── visao_computacional_avii_.ipynb      # Notebook Atividade II
├── dataset/
│   ├── concha/                              # 5 imagens originais
│   └── pedra/                               # 5 imagens originais
├── .gitignore
└── README.md
```

As transformacoes e filtros sao gerados automaticamente ao executar cada notebook.

## Como executar

1. Abra o notebook desejado no [Google Colab](https://colab.research.google.com/)
2. Clone o dataset na primeira celula:
```python
!git clone https://github.com/fabri-marques/cv-coursework.git /content/repo
!ln -s /content/repo/dataset /content/dataset
```
3. Execute todas as celulas em sequencia

## Equipamento

Xiaomi Redmi Note 13 Pro 5G — lente principal f/1.65, 23mm eq., flash desativado.
Todas as imagens capturadas em 09/05/2026, ambiente interno.
