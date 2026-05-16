# Guia de Estudo: Atividade II - Filtragem e Detecção de Bordas

Material de referência pra consolidar os conceitos usados no notebook.

---

## 1. Tipos de Ruído em Imagens Digitais

### Ruído Gaussiano
- Variações aleatórias de intensidade que seguem uma distribuição normal (curva em sino)
- Cada pixel tem seu valor somado a um valor aleatório com média 0
- Causa principal: amplificação eletrônica do sensor (ISO alto)
- Aparência: granulação uniforme espalhada pela imagem
- É o tipo dominante nas nossas imagens (ISO 800 a 5000)

### Shot Noise (Ruído de Disparo)
- Vem da natureza quântica da luz: fótons chegam ao sensor em quantidades discretas e aleatórias
- Segue distribuição de Poisson, não Gaussiana
- Mais forte onde o sinal é mais fraco (áreas escuras), porque lá chegam poucos fótons e a variação relativa é maior
- Na prática, em ISOs altos, se mistura com o gaussiano e fica difícil separar

### Dark Current Noise
- Ruído térmico gerado pelo próprio sensor, mesmo sem luz
- Aumenta com tempo de exposição longo e temperatura alta
- Nas nossas imagens, contribui em IMG_001 e IMG_005 (1/10 s de exposição)

### Ruído de Quantização
- Aparece quando a faixa dinâmica útil é pequena (ISO muito alto comprime a faixa)
- Os níveis de cinza disponíveis ficam mais "grosseiros"
- Não é exatamente ruído aleatório, mas sim perda de resolução tonal

### Artefatos de Compressão JPEG
- Não é ruído do sensor, mas sim da etapa de salvamento
- JPEG divide a imagem em blocos 8×8 e descarta coeficientes de alta frequência (DCT)
- Resultado: blocos visíveis e "halos" em bordas de alto contraste
- Nas nossas imagens aparece junto com o ruído do sensor, complicando a análise

### Como medir ruído: estimador de Immerkær
- Convolui a imagem com um kernel Laplaciano e mede o desvio absoluto médio
- Retorna um σ (sigma) que estima o desvio padrão do ruído
- Quanto maior o σ, mais ruidosa a imagem
- Assume ruído gaussiano aditivo, então não é perfeito pra shot noise ou artefatos JPEG
- No código: função `estimar_ruido(img_gray)`

---

## 2. Filtros de Suavização

A ideia geral: percorrer a imagem pixel por pixel, substituindo cada pixel por alguma combinação dos vizinhos. O "kernel" define quais vizinhos e com quais pesos.

### Filtro da Média (`cv2.blur`)
- Cada pixel vira a média aritmética dos vizinhos dentro do kernel
- Kernel 3×3: média de 9 pixels. Kernel 5×5: média de 25 pixels
- Todos os vizinhos têm peso igual
- Prós: simples, reduz bem ruído gaussiano
- Contras: borra bordas sem dó, porque trata borda e ruído da mesma forma
- Kernel maior = mais suavização = mais borramento

### Filtro Gaussiano (`cv2.GaussianBlur`)
- Parecido com a média, mas os pesos seguem uma curva gaussiana (sino)
- O pixel central tem peso máximo, os vizinhos mais distantes pesam menos
- O parâmetro σ (sigma) controla a largura da gaussiana:
  - σ pequeno (ex: 1): kernel efetivo pequeno, suavização leve
  - σ grande (ex: 3): kernel efetivo grande, suavização forte
- `ksize=(0,0)` no OpenCV faz ele calcular o tamanho do kernel automaticamente a partir do σ
- Prós: preserva bordas melhor que a média (pesos decaem nas pontas)
- Contras: ainda é linear, então borra bordas, só que menos

### Filtro da Mediana (`cv2.medianBlur`)
- NÃO é uma média. Ordena os valores dos vizinhos e pega o do meio
- É não-linear: o valor escolhido já existe na vizinhança, nunca cria valores intermediários
- Kernel 3×3: pega o 5º valor de 9 ordenados. Kernel 5×5: 13º de 25
- Prós: excelente pra ruído impulsivo (sal-e-pimenta), preserva bordas muito bem
- Contras: não é o mais eficaz contra ruído gaussiano puro; mais lento que os lineares
- Por que preserva bordas: numa borda, metade dos vizinhos é claro e metade escuro. A mediana cai em um dos lados, mantendo a transição abrupta

### Comparação rápida

| Filtro | Tipo | Reduz ruído gaussiano | Preserva bordas | Contra ruído impulsivo |
|---|---|---|---|---|
| Média | Linear | Bom | Ruim | Médio |
| Gaussiano | Linear | Bom | Médio | Médio |
| Mediana | Não-linear | Médio | Bom | Excelente |

### Conceitos-chave
- **Kernel (janela)**: a vizinhança usada pra calcular o novo valor. 3×3 = 9 pixels, 5×5 = 25 pixels
- **Filtro passa-baixa**: qualquer filtro que remove altas frequências (detalhes finos, ruído, bordas). Todos os três acima são passa-baixa
- **Variância**: mede a dispersão dos valores de cinza. Filtros passa-baixa sempre reduzem variância porque "achatam" as diferenças locais
- **Energia de borda**: média da magnitude do gradiente Sobel. Proxy de "quanta borda restou" após filtrar

---

## 3. Detecção de Bordas

Borda = transição brusca de intensidade. Detectar bordas = encontrar onde a derivada (gradiente) da imagem é alta.

### Gradiente de uma imagem

Uma imagem é uma função f(x, y) onde x e y são as coordenadas e f é a intensidade. O gradiente tem duas componentes:

- **Gx (gradiente em X)**: derivada parcial na horizontal. Detecta bordas verticais (transições da esquerda pra direita)
- **Gy (gradiente em Y)**: derivada parcial na vertical. Detecta bordas horizontais (transições de cima pra baixo)
- **Magnitude**: `sqrt(Gx² + Gy²)`. Combina os dois numa medida única de "intensidade da borda"

Na prática, aproximamos as derivadas com convoluções usando kernels pequenos.

### Operador de Sobel (`cv2.Sobel`)

Kernels 3×3:
```
Gx:                 Gy:
[-1  0  +1]         [-1  -2  -1]
[-2  0  +2]         [ 0   0   0]
[-1  0  +1]         [+1  +2  +1]
```

- Pesos [1, 2, 1]: dá mais peso ao pixel central da linha/coluna
- Esse peso extra funciona como uma suavização implícita perpendicular à direção do gradiente
- `cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)` → Gx (1 em dx, 0 em dy)
- `cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)` → Gy (0 em dx, 1 em dy)
- `cv2.CV_64F`: calcula em float64 pra não perder valores negativos (bordas escuro→claro e claro→escuro)

### Operador de Prewitt

Kernels 3×3:
```
Gx:                 Gy:
[-1  0  +1]         [-1  -1  -1]
[-1  0  +1]         [ 0   0   0]
[-1  0  +1]         [+1  +1  +1]
```

- Pesos uniformes [1, 1, 1]: todos os vizinhos pesam igual
- Não tem a suavização implícita do Sobel
- Resultado: ligeiramente mais sensível ao ruído que o Sobel
- OpenCV não tem função dedicada, por isso usamos `cv2.filter2D` com os kernels manuais
- Na prática a diferença visual entre Sobel e Prewitt é sutil

### Detector de Canny (`cv2.Canny`)

Não é um operador simples como Sobel/Prewitt. É um pipeline de 4 etapas:

1. **Suavização gaussiana**: aplica um blur pra reduzir ruído antes de calcular gradientes
2. **Cálculo do gradiente**: usa Sobel internamente pra obter Gx, Gy e magnitude
3. **Supressão non-maxima (NMS)**: percorre cada pixel de borda e verifica se ele é o máximo local na direção perpendicular à borda. Se não for, zera. Resultado: bordas de 1 pixel de espessura
4. **Histerese com dois limiares (low, high)**:
   - Pixel com magnitude > high → borda forte (aceito)
   - Pixel com magnitude entre low e high → borda fraca (aceito SÓ se conectado a uma borda forte)
   - Pixel com magnitude < low → descartado
   - Isso elimina respostas fracas isoladas (ruído) mas preserva bordas fracas que continuam uma borda real

- Parâmetros: `cv2.Canny(img, low, high)`
- Saída: imagem binária (0 ou 255), já pronta pra uso
- Prós: bordas finas, limpas, menos bordas falsas, não precisa de threshold posterior
- Contras: não expõe Gx e Gy separados, mais caro computacionalmente

### Comparação dos operadores

| | Sobel | Prewitt | Canny |
|---|---|---|---|
| Saída | Magnitude contínua (float) | Magnitude contínua (float) | Binária (0/255) |
| Espessura | Grossa (vários pixels) | Grossa (vários pixels) | 1 pixel |
| Precisa de threshold | Sim, pra binarizar | Sim, pra binarizar | Não (histerese interna) |
| Acesso a Gx, Gy | Sim | Sim | Não |
| Sensibilidade ao ruído | Média | Um pouco maior | Baixa |
| Quando usar | HoG, Harris, orientação de textura | Idem Sobel | Máscara binária de bordas, contornos |

### Threshold (limiar) pra contagem de bordas

Pra comparar operadores quantitativamente, precisamos contar "pixels de borda". Em Sobel/Prewitt a saída é contínua, então precisamos de um corte:

- **Threshold absoluto**: define um valor fixo (ex: 100) sobre a magnitude bruta. Pixels acima = borda. Consistente entre imagens diferentes
- **Threshold relativo**: define um percentual do máximo da imagem (ex: 30% do max). Problema: se o filtro reduz o máximo (ex: Gaussiano borrando bordas fortes), mais pixels de fundo passam o critério, distorcendo a comparação

No notebook usamos threshold absoluto (100) por causa desse problema.

### Componentes conectados

Técnica pra agrupar pixels de borda em "pedaços" contínuos:
- `cv2.connectedComponentsWithStats` agrupa pixels vizinhos que são ambos borda
- Cada grupo recebe um label e uma contagem de pixels
- Componentes grandes (≥50 pixels) → provavelmente bordas reais (silhueta do objeto)
- Componentes pequenos (<50 pixels) → provavelmente bordas falsas (ruído isolado)
- Permite medir separadamente: "quantas bordas falsas o filtro removeu?" e "quantas bordas reais preservou?"

---

## 4. O que cada célula do notebook faz

### Cell 0-4: Cabeçalho e enunciado
Markdown com título, objetivo e enunciado da atividade. Sem código.

### Cell 5: Seleção das imagens (markdown)
Justificativa da escolha das 4 imagens com base nos metadados EXIF. Cada imagem lista nome, classe, tipo de degradação observada e hipótese sobre a origem do ruído.

### Cell 6: Setup e imports
- Clona o repositório do GitHub pra ter acesso ao dataset no Colab
- Importa as bibliotecas (cv2, numpy, PIL, matplotlib, pandas)
- Define funções utilitárias:
  - `carregar_rgb(path)`: abre imagem respeitando rotação EXIF
  - `carregar_gray(path)`: converte pra escala de cinza
  - `estimar_ruido(img_gray)`: estimador de σ de Immerkær (convolui com Laplaciano)
  - Define os kernels Prewitt manualmente (OpenCV não tem função pronta)

### Cell 7: Dicionário de imagens selecionadas
Define a lista `SELECIONADAS` com classe, nome do arquivo, ISO e BV de cada imagem. Funções auxiliares `caminho()` e `nome_curto()`.

### Cell 8: Visualização das 4 imagens
Plota as 4 imagens selecionadas com título mostrando ISO, BV e σ estimado. Serve pra confirmar visualmente que as escolhas fazem sentido.

### Cell 9: Título "Filtros de Suavização"

### Cell 10: Função energia_borda + constantes
- `energia_borda(img)`: aplica Sobel em X e Y, calcula magnitude, retorna a média. Proxy de "quanta borda tem"
- Define `CROP_SIZE = 600` pra exibir zooms centrais
- Define o diretório de saída

### Cell 11: Título "Média"

### Cell 12: Filtro da Média
Pra cada imagem selecionada:
1. Carrega em cinza
2. Mede métricas do original (σ, variância, energia de borda)
3. Aplica `cv2.blur(img, (k, k))` com k=3 e k=5
4. Mede métricas depois do filtro
5. Salva resultado como PNG
6. Plota crop central antes/depois lado a lado
7. Guarda métricas na lista `resultados_media`

### Cell 13: Título "Gaussiano"

### Cell 14: Filtro Gaussiano
Mesma lógica da célula 12, mas com `cv2.GaussianBlur(img, (0, 0), sigmaX=s)` pra σ=1 e σ=3. O `(0,0)` deixa o OpenCV calcular o tamanho do kernel automaticamente.

### Cell 15: Título "Mediana"

### Cell 16: Filtro da Mediana
Mesma lógica, com `cv2.medianBlur(img, k)` pra k=3 e k=5.

### Cell 17: Análise da suavização (markdown)
Texto discutindo: qual reduziu mais ruído (Gaussiano σ=3 e Média 5×5), qual preservou melhor bordas (Mediana 3×3), o que aconteceu com a variância (todos reduziram, proporcional ao kernel).

### Cell 18: Painel comparativo
Gera uma figura 2×4 com todos os filtros lado a lado pra imagem mais ruidosa (pedra IMG_005). Linha 1: parâmetros menores (3×3/σ=1). Linha 2: parâmetros maiores (5×5/σ=3).

### Cell 19: Título "Detecção de bordas sem suavização"

### Cell 20: Funções auxiliares de bordas
- `normalizar(img_float)`: mapeia magnitude float pra uint8 [0,255] pra poder visualizar
- Define `THRESHOLD_BORDA_ABS = 100`

### Cell 21: Título "Sobel"

### Cell 22: Sobel sem suavização
Pra cada imagem:
1. Calcula Gx com `cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)` (derivada em X)
2. Calcula Gy com `cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)` (derivada em Y)
3. Magnitude = `sqrt(Gx² + Gy²)`
4. Normaliza pra uint8, salva, conta pixels acima do threshold
5. Plota original + Gx + Gy + magnitude

### Cell 23: Título "Prewitt"

### Cell 24: Prewitt sem suavização
Mesma lógica do Sobel, mas usando `cv2.filter2D` com os kernels Prewitt definidos no setup (PREWITT_X e PREWITT_Y).

### Cell 25: Título "Canny"

### Cell 26: Canny sem suavização
Testa 3 pares de limiares: (50,150), (100,200), (150,250). Pra cada par, aplica `cv2.Canny(img, low, high)` e conta pixels não-zero. Plota original + 3 versões.

### Cell 27-28: Análise da detecção de bordas (markdown)
Discussão sobre bordas falsas, sensibilidade ao ruído, tabela comparativa Sobel/Prewitt/Canny, efeito dos limiares do Canny.

### Cell 29: Tabela de métricas
Monta DataFrame com resultados de Sobel + Prewitt + Canny e imprime pivot table.

### Cell 30: Título "Detecção de bordas com suavização"

### Cell 31: Setup da Parte 4
- Define dois filtros candidatos: Gaussian σ=1 e Mediana 3×3
- Define limiares Canny pra Parte 4: (100, 200)
- Função `analisar_bordas`: usa `cv2.connectedComponentsWithStats` pra separar componentes grandes (bordas reais) de pequenos (bordas falsas)
- Função `calcular_metricas`: calcula redução de falsas e preservação de reais

### Cell 32-33: Sobel pós suavização
Pra cada imagem e cada filtro de pré-processamento:
1. Aplica Sobel no original (referência)
2. Aplica o filtro de suavização
3. Aplica Sobel no suavizado
4. Compara: contagem total, bordas falsas, bordas reais

### Cell 34-35: Prewitt pós suavização
Mesma lógica com Prewitt.

### Cell 36-37: Canny pós suavização
Mesma lógica com Canny. Saída já é binária, então contagem usa `count_nonzero`.

### Cell 38: Análise da detecção com suavização (markdown)
Nota metodológica sobre threshold relativo vs absoluto. Comparação Gaussian vs Mediana com tabela de métricas. Conclusão: Mediana preserva melhor a silhueta.

### Cell 39: Tabela final de métricas
DataFrame com todos os resultados da Parte 4. Pivot tables por operador e filtro.

### Cell 40: Conclusão (markdown)
Síntese geral, pipeline recomendado, aprendizados e limitações.

---

## 5. Resumo dos parâmetros do OpenCV usados

| Função | Parâmetros importantes | O que fazem |
|---|---|---|
| `cv2.blur(img, (k,k))` | k = tamanho do kernel | Média dos vizinhos. k maior = mais suave |
| `cv2.GaussianBlur(img, (0,0), sigmaX=s)` | s = desvio padrão da gaussiana | Média ponderada. σ maior = mais suave |
| `cv2.medianBlur(img, k)` | k = tamanho do kernel (ímpar) | Mediana dos vizinhos. Não-linear |
| `cv2.Sobel(img, cv2.CV_64F, dx, dy, ksize=3)` | dx, dy = ordem da derivada | dx=1,dy=0 → Gx. dx=0,dy=1 → Gy |
| `cv2.filter2D(img, -1, kernel)` | kernel = matriz de pesos | Convolução genérica. Usada pro Prewitt |
| `cv2.Canny(img, low, high)` | low, high = limiares da histerese | Saída binária. low/high controlam sensibilidade |
| `cv2.connectedComponentsWithStats(mask)` | mask = imagem binária | Agrupa pixels conectados, retorna stats |

---

## 6. Perguntas pra se testar

1. Por que a Mediana preserva bordas melhor que a Média?
2. Qual a diferença entre os pesos do Sobel [1,2,1] e do Prewitt [1,1,1]?
3. O que acontece se eu usar σ muito grande no Gaussiano?
4. Por que o Canny precisa de dois limiares (low e high) em vez de um só?
5. O que é supressão non-maxima e por que o Canny precisa dela?
6. Por que usamos threshold absoluto em vez de relativo pra comparar filtros?
7. Se meu objetivo é calcular HoG (histograma de gradientes orientados), qual operador uso: Sobel ou Canny? Por quê?
8. O que o σ de Immerkær mede e qual a sua limitação principal?
