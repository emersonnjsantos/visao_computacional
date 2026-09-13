# 👁️ Projeto: Suavização, Remoção de Ruído e Detecção de Bordas

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/emersonnjsantos/visao_computacional/blob/main/ATIVIDADE_1/visao_computacional_mini_dataset/notebooks/relatorio_atividade_2_projeto.ipynb)

##  Objetivo do Projeto
Este projeto integra o módulo de Pré-Processamento Básico de Imagens da Residência em Visão Computacional. O objetivo prático é demonstrar, por meio de formulações matemáticas no domínio espacial, como a preparação de dados ruidosos é crucial antes de aplicar algoritmos de extração de características (como derivadas para detecção de bordas).

##  Tecnologias e Bibliotecas Utilizadas
* **Linguagem:** Python 3
* **Visão Computacional:** OpenCV (`cv2`)
* **Matemática e Matrizes:** NumPy
* **Visualização de Dados:** Matplotlib

---

##  Arquitetura do Pipeline (Passo a Passo)

### 1. Seleção e Diagnóstico de Degradação
Análise crítica de um dataset real contendo texturas complexas (folhagens), identificando anomalias no domínio espacial, como:
* Ruído de ganho (ISO alto) gerando granulação em áreas de sombra.
* Desfoque de movimento (motion blur) e dificuldades de foco em macros.
* Artefatos de compressão visual (blocos JPEG).

### 2. Filtros Espaciais e Suavização (Passa-Baixa)
Aplicação de operações matemáticas em matrizes de pixels para estabilizar a distribuição estatística das imagens em escala de cinza:
* **Filtros Lineares:** Filtro da Média (sensível a outliers) e Filtro Gaussiano (suavização com preservação de estrutura geométrica).
* **Filtros Não Lineares:** Filtro da Mediana (alta robustez contra ruídos impulsivos).
* **Análise Estatística:** Comprovação matemática de que a suavização reduz a variância dos níveis de cinza da imagem.

### 3. Detecção de Bordas em Dados Brutos
Extração do gradiente de intensidade utilizando operadores matemáticos diferenciais **sem suavização prévia**, provando a alta sensibilidade ao ruído:
* **Operador de Sobel:** Aproximação da derivada com leve fator de suavização central (matriz 3x3).
* **Operador de Prewitt:** Aproximação da derivada com pesos uniformes (altamente sensível ao ruído de alta frequência).
* **Algoritmo de Canny:** Formulação de otimização revelando rastros de falsas bordas (falsos positivos) gerados por granulação da imagem bruta.

### 4. Pipeline Otimizado (Suavização + Detecção)
Implementação da arquitetura correta de pré-processamento, aplicando o **Filtro Gaussiano ($\sigma=1$)** antes da detecção geométrica:
* **Redução de Falsos Positivos:** O filtro atenuou ruídos de alta frequência, limpando as matrizes gradientes.
* **Continuidade Estrutural:** A etapa de limiarização por histerese do detector de Canny conseguiu focar exclusivamente nas ranhuras e contornos reais das folhagens, gerando mapas de bordas limpos, finos (1 pixel) e bem conectados.

---

##  Conclusão Técnica
Este projeto valida que o desempenho de métodos de extração de características em Visão Computacional depende fundamentalmente do pré-processamento. A união de filtros passa-baixa (como o Gaussiano) com operadores diferenciais estruturados (como o Canny) forma uma base robusta não apenas para análise tradicional de imagens, mas para a padronização de dados exigida no treinamento de Redes Neurais Convolucionais (CNNs) e na engenharia de sistemas embarcados.