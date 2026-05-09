# Visão Computacional — Atividade I

**Aluna:** Fabricia Cristina Marques Jung
**Disciplina:** Visão Computacional
**Instituição:** IREDE/AVI

## Sobre

Mini dataset com 2 classes (concha vs pedra, 5 imagens cada) construído do zero para aplicar fundamentos de Processamento de Imagens Digitais:

- Aquisição controlada (variação de iluminação, ângulo, distância)
- Análise de resolução (100%, 50%, 20%)
- Conversão de espaços de cor (RGB, HSV, escala de cinza)
- Quantização (256, 64, 32, 2 níveis)
- Comparação de formatos (PNG lossless vs JPEG q50)

## Estrutura

```
├── cv_irede_avi (1).ipynb    # Notebook com código + relatório
├── dataset/
│   ├── concha/               # 5 imagens originais
│   └── pedra/                # 5 imagens originais
└── .gitignore
```

As transformações (`dataset/transformacoes/`) são geradas automaticamente ao executar o notebook.

## Como executar

1. Abra o notebook no [Google Colab](https://colab.research.google.com/)
2. Clone o dataset na primeira célula:
```python
!git clone https://github.com/fabri-marques/cv-coursework.git /content/repo
!ln -s /content/repo/dataset /content/dataset
```
3. Execute todas as células em sequência

## Equipamento

Xiaomi Redmi Note 13 Pro 5G — lente principal ƒ/1.65, 23mm eq., flash desativado.
Todas as imagens capturadas em 09/05/2026, ambiente interno.
