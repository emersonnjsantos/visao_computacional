```markdown
#  Visão Computacional — Residência em TIC43

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/emersonnjsantos/visao-computacional-tic43/blob/main/notebooks/relatorio_atividade_1.ipynb)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)
![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green.svg)
![Status](https://img.shields.io/badge/Status-Concluído-brightgreen.svg)

Repositório dedicado às atividades e projetos práticos da disciplina de **Visão Computacional** na **Residência em TIC43** (Iniciativa Softex / MCTI).

---

##  Atividade 1: Fundamentos de Processamento Digital de Imagens (PDI)

* **Aluno:** Emerson Santos
* **Docente:** Prof. Alyson Bezerra
* **Objetivo:** Construção de um mini dataset próprio com 2 classes e 10 imagens no total, aplicando transformações de processamento de imagens e análise técnica do impacto em sistemas de Visão Computacional.

---

##  Estrutura do Repositório

```text
visao-computacional-tic43/
│
├── dataset/                        # Mini Dataset com 10 amostras originais
│   ├── classe_A/                   # Amostras de Flores (img001 a img005)
│   └── classe_B/                   # Amostras de Folhas (img006 a img010)
│
├── outputs/                        # Imagens derivadas das transformações (Parte 3)
│   ├── resolucao/                  # Reduções em 50% e 20%
│   ├── espaco_cor/                 # Conversões RGB, HSV e Escala de Cinza
│   ├── quantizacao/                # Versões em 256, 64, 32 e 2 níveis de cinza
│   └── formatos/                   # Comparações entre JPEG (comprimido) e PNG (sem perdas)
│
├── notebooks/                      # Relatório técnico e códigos executáveis
│   └── relatorio_atividade_1.ipynb
│
├── .gitignore
├── README.md
└── requirements.txt

```

---

##  Metadados de Aquisição do Dataset

* **Dispositivo de Captura:** Smartphone Xiaomi 11 Pro (sensor principal de 108 MP, abertura f/1.9).
* **Configurações:** Câmera nativa sem filtros automáticos, inteligência artificial ou embelezamento.
* **Classes Definidas:**
* **Classe A:** Flores (`img001_original.jpg` a `img005_original.jpg`)
* **Classe B:** Folhas (`img006_original.jpg` a `img010_original.jpg`)


* **Condições de Iluminação:**
1. *Luz Natural Direta:* Sol pleno com alto contraste.
2. *Luz Natural Difusa / Sombra:* Distribuição homogênea de iluminação.
3. *Luz Artificial Interna:* Iluminação LED de ambiente interno.


* **Distâncias e Ângulos:** Variação de 15 cm a 25 cm (close-up/macro) e 60 cm a 1 metro (plano geral); ângulos zenital e 45°.

---

##  Transformações e Análises Realizadas

1. **Análise de Resolução:** Avaliação da degradação de altas frequências espaciais e texturas finas em reduções de 50% e 20%, ponderando o custo computacional no treinamento de redes neurais (CNNs).
2. **Espaços de Cor (RGB, HSV e Escala de Cinza):** Análise do desacoplamento de matiz/saturação e brilho (HSV) para segmentação invariante a sombras versus redução de dimensionalidade em escala de cinza.
3. **Quantização de Intensidade:** Efeitos de falso contorno (*color banding*) em 64 e 32 níveis e limiarização binarizada (2 níveis) para segmentação e extração de máscaras.
4. **Formatos de Arquivo (JPEG vs. PNG):** Comparativo de tamanho em disco (KB/MB) e análise de artefatos de compressão por blocos no JPEG frente à integridade de pixels no formato PNG.

---

##  Como Executar o Projeto

### Opção 1: Google Colab (Recomendado)

Basta clicar no botão abaixo para abrir e rodar o relatório interativo diretamente na nuvem:

### Opção 2: Localmente

1. Clone o repositório:

```bash
git clone [https://github.com/emersonnjsantos/visao-computacional-tic43.git](https://github.com/emersonnjsantos/visao-computacional-tic43.git)
cd visao-computacional-tic43

```

2. Instale as dependências:

```bash
pip install -r requirements.txt

```

3. Abra o Jupyter Notebook:

```bash
jupyter notebook notebooks/relatorio_atividade_1.ipynb

```

---

##  Tecnologias Utilizadas

* **Linguagem:** Python 3.x
* **Bibliotecas:** OpenCV (`cv2`), NumPy, Matplotlib, Pillow
* **Ambiente:** Google Colab / Jupyter Notebook
* **Controle de Versão:** Git e GitHub

```

```
