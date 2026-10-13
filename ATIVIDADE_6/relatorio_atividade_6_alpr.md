# Relatório Técnico de Engenharia: Análise Crítica dos Fundamentos de Imagens Digitais em Sistemas de Leitura Automática de Placas Veiculares (ALPR / Free-Flow ITS)

**Programa de Residência em TIC43 — Módulo de Visão Computacional**  
*Parceria: Softex / Ministério da Ciência, Tecnologia e Inovações (MCTI) / Governo Federal / iRede*  
**Unidade 1 | Capítulo 1 | Tarefa 1 (Atividade 6 no Ambiente Virtual de Aprendizagem Homero)**  
**Docente Responsável:** Prof. Rafael Carmo  
**Docentes do Módulo:** Prof. Alyson Bezerra | Prof. Matheus Araújo  
**Autor (Engenheiro Responsável):** Emerson Noe Jose dos Santos  
**Repositório do Projeto:** [github.com/emersonnjsantos/visao_computacional](https://github.com/emersonnjsantos/visao_computacional)  

---

## 1. Resumo Executivo

Este relatório técnico apresenta a concepção, o dimensionamento e a fundamentação físico-matemática de um sistema de Visão Computacional voltado ao **Reconhecimento Automático de Placas Veiculares (*Automatic License Plate Recognition* — ALPR / LPR)**. O projeto foi estruturado sob a perspectiva de um Engenheiro de Visão Computacional sênior alocado na concessionária de rodovias de grande porte (*AutoVia Brasil Concessionária de Rodovias S.A.*), com o objetivo de operar pórticos de pedágio 100% eletrônico sem cancelas (*Free-Flow*) e fiscalização de evasão tributária e de velocidade em rodovias de classe 1 (BR-101 / SP-330).

A análise articula minuciosamente os seis pilares fundamentais das imagens digitais:
1. **Estrutura matricial da imagem** e representação matemática discreta;
2. **Pixel, resolução espacial e teorema de amostragem de Nyquist-Shannon**;
3. **Resolução radiométrica (profundidade de bits), alcance dinâmico (WDR) e quantização**;
4. **Espaços de cor (RGB, HSV, Escala de Cinza) e a física óptica no espectro infravermelho próximo (NIR ~850 nm)**;
5. **Pipeline de aquisição física** (sensores CMOS com *Global Shutter*, lentes industriais, iluminação estroboscópica e processamento ISP);
6. **Formatos de armazenamento, compressão com/sem perdas e impacto de artefatos de alta frequência na inferência algorítmica**.

Adicionalmente, o relatório responde de maneira argumentativa, matemática e crítica às seis questões centrais demandadas pela atividade, culminando em uma reflexão aprofundada sobre maturidade de engenharia, a falácia do "Deep Learning resolve qualquer imagem de entrada" (*Garbage In, Garbage Out*) e as trade-offs econômico-operacionais para viabilizar um sistema com disponibilidade ininterrupta (24/7/365) e índice de acurácia de leitura superior a 99,5%.

---

## 2. Contextualização do Cenário Operacional Escolhido

### 2.1 Perfil Corporativo e Escopo de Operação
* **Empresa Operadora:** *AutoVia Brasil Concessionária de Rodovias S.A.*
* **Segmento:** Concessão Rodoviária e Sistemas Inteligentes de Transporte (*Intelligent Transportation Systems* — ITS).
* **Porte da Organização:** Grande porte (administração de 620 km de malha rodoviária federal sob regime de concessão).
* **Aplicação-Alvo:** Pórticos de cobrança automática *Free-Flow* (pedágio eletrônico aberto, sem praças físicas nem praças com cancelas mecânicas), integrados a subsistemas de pesagem dinâmica em movimento (*WIM — Weigh-in-Motion*) e fiscalização metrológica de velocidade média e pontual.
* **Demanda de Tráfego:** Volume Médio Diário (VMD) de 80.000 veículos por pórtico, distribuídos em 3 faixas de rolamento por sentido mais acostamento transitável em horários de pico.
* **Velocidade de Passagem:** Veículos transitando entre 0 km/h (situação de congestionamento severo) e até 140 km/h (velocidade máxima regulamentar acrescida de margens de evasão), sem qualquer desaceleração obrigatória.

### 2.2 Desafios Ambientais e Operacionais Críticos
A engenharia de um sistema ALPR rodoviário em regime *Free-Flow* enfrenta um dos ambientes mais hostis e dinâmicos para a visão computacional:
1. **Variação Extrema de Iluminação:** Insolação direta de verão (iluminância de até 100.000 lux), contraluz provocada por sol poente no horizonte da pista, escuridão noturna completa em trechos ermos (iluminância < 0,1 lux) e sombras projetadas por viadutos e vegetação.
2. **Reflexos Especulares e Retrorreflexão:** As placas brasileiras atuais (padrão Mercosul estabelecido pelas Resoluções CONTRAN 780/2019 e 969/2022) utilizam películas retrorrefletivas de microprismas. Essas películas devolvem a luz incidente diretamente para a fonte com alto ganho óptico. Se a iluminação ativa for mal calibrada, a placa satura completamente o sensor (estouro de brancos), destruindo os caracteres.
3. **Ofuscamento por Faróis (*Blooming / Smear*):** Faróis de veículos com tecnologia LED ou Xenon apontados diretamente para as câmeras causam vazamento de carga elétrica entre pixels adjacentes e superexposição severa em sensores de baixa qualidade.
4. **Perturbações Meteorológicas:** Chuva torrencial gerando spray de água e poças reflexivas, neblina densa, poeira de suspensão e vibrações mecânicas no pórtico induzidas por rajadas de vento e passagem contínua de carretas bitrem.
5. **Heterogeneidade de Placas em Circulação:** Convivência simultânea de dois padrões legais principais no Brasil:
   * **Placas Padrão Mercosul (Resolução CONTRAN 780/2019):** Dimensões de 400 mm x 130 mm (veículos de quatro ou mais rodas) e 200 mm x 170 mm (motocicletas); fundo branco retrorrefletivo, faixa azul superior de 30 mm, combinação de 4 letras e 3 números alfanuméricos em relevo (ex: ABC1D23) com cor dos caracteres variando conforme a categoria (preto para particular, vermelho para comercial, azul para oficial, etc.), contendo ondas holográficas e marca d'água.
   * **Placas Padrão Pré-Mercosul (Cinzas):** Fundo cinza metálico retrorrefletivo, caracteres pretos estampados no formato alfanumérico LLL-NNNN (3 letras e 4 números).

---

## 3. Fundamentação Teórica sobre os Fundamentos de Imagens Digitais

Para embasar todas as decisões de projeto, revisitam-se os princípios físico-matemáticos que regem a conversão do mundo contínuo luminoso na representação computacional discreta.

### 3.1 Estrutura Matricial da Imagem e Representação Matemática
Uma imagem contínua no mundo real é representada pela função bidimensional de irradiância:

$$f(x, y) = i(x, y) \cdot r(x, y)$$

onde $x, y \in \mathbb{R}$ são coordenadas espaciais euclidianas no plano do sensor, $i(x, y) \in (0, \infty)$ é o componente de iluminação incidente da fonte externa, e $r(x, y) \in [0, 1]$ é o coeficiente de refletância da superfície do objeto (Gonzalez & Woods, 2018).

O processo de digitalização converte essa função contínua em uma matriz bidimensional discreta $I[m, n]$ de dimensões $M \times N$:

$$I[m, n] = \mathcal{Q}\Big( f(m \cdot \Delta x, \, n \cdot \Delta y) \Big)$$

onde:
* $m \in \{0, 1, \dots, M-1\}$ e $n \in \{0, 1, \dots, N-1\}$ são os índices discretos de linha e coluna;
* $\Delta x, \Delta y$ representam o espaçamento espacial de amostragem (passo de amostragem ou pitch dos fotossítios do sensor);
* $\mathcal{Q}(\cdot)$ é o operador de quantização que mapeia valores analógicos contínuos de tensão elétrica para um conjunto finito de níveis de intensidade discretos.

Em termos computacionais, cada elemento $I[m, n]$ é denominado **pixel** (*picture element*). Em imagens multiespectrais ou coloridas, $I[m, n]$ assume a forma de um vetor tridimensional $\mathbf{I}[m, n, c]$, onde $c \in \{0, 1, 2\}$ indexa os canais de cor.

### 3.2 Amostragem, Resolução Espacial e Teorema de Nyquist-Shannon
A **resolução espacial** define a densidade de amostras por unidade de comprimento no plano do mundo real ou no plano do sensor (ex: pixels por polegada — PPI, ou pixels por metro — PPM).

De acordo com o **Teorema de Amostragem de Nyquist-Shannon**, para que um sinal contínuo com largura de banda limitada em frequência espacial máxima $f_{\max}$ seja reconstruído ou discriminado sem ambiguidades (sem o fenômeno destrutivo do *aliasing* ou serrilhamento), a taxa de amostragem espacial $f_s$ deve satisfazer:

$$f_s \ge 2 \cdot f_{\max}$$

onde $f_{\text{Nyquist}} = \frac{f_s}{2}$ é a frequência limite de corte.

No domínio de caracteres alfanuméricos em placas veiculares, cada caractere é constituído por traços com espessura física delimitada (geralmente entre 10 mm e 12 mm de espessura de traço na placa Mercosul). Se um traço de 10 mm precisa ser distinguido de um vão de fundo de 10 mm adjacente, estamos diante de um ciclo periódico de $20\text{ mm}$ (frequência fundamental espacial de $50\text{ ciclos/metro}$). Para satisfazer a condição de Nyquist no pior caso, o sistema óptico-digital precisa registrar no mínimo 2 amostras por ciclo, isto é, 1 pixel a cada 10 mm. No entanto, para fins de classificação robusta em OCR e algoritmos morfológicos, a literatura clássica e as normas industriais de ITS exigem no mínimo de 2 a 3 pixels através da espessura do menor traço, o que traduz em uma altura vertical mínima de **20 a 30 pixels por caractere da placa**.

### 3.3 Quantização, Resolução Radiométrica e Alcance Dinâmico (Dynamic Range)
A **resolução radiométrica** (ou profundidade de bits, $b$) quantifica o número de níveis discretos de cinza ou luminosidade disponíveis para codificar a intensidade de cada amostra espacial:

$$L = 2^b$$

* Com $b = 8\text{ bits}$, temos $L = 256$ níveis $[0, 255]$;
* Com $b = 10\text{ bits}$, temos $L = 1024$ níveis $[0, 1023]$;
* Com $b = 12\text{ bits}$, temos $L = 4096$ níveis $[0, 4095]$.

O erro de quantização introduz um ruído estocástico intrínseco. O **Alcance Dinâmico** (*Dynamic Range* — DR) de um sensor é a razão entre a máxima carga de saturação mensurável do fotossítio (*Full Well Capacity*) e o piso de ruído no escuro (*Read Noise*), comumente expresso em decibéis (dB):

$$\text{DR (dB)} = 20 \log_{10}\left( \frac{V_{\max}}{\sigma_{\text{ruído}}} \right)$$

Sensores padrão de 8 bits oferecem alcance dinâmico de 48 dB a 60 dB, insuficiente para lidar simultaneamente com a carcaça escura de um caminhão à noite e os faróis altos acesos ou o brilho retrorrefletivo da placa sob iluminação estroboscópica. Sensores industriais modernos para ITS operam com conversores A/D de **12 bits** associados a tecnologias de *Wide Dynamic Range* (WDR) ou *High Dynamic Range* (HDR) com dupla ou tripla exposição simultânea no pixel, atingindo alcances dinâmicos de até **120 dB**.

### 3.4 Espaços de Cor e a Física Óptica no Espectro Eletromagnético
As cores perceptíveis pelo olho humano resultam da integração da energia espectral incidente com os três tipos de cones da retina (modelo tricromático):
1. **Espaço RGB:** Modelo aditivo primário. Os canais Red, Green e Blue possuem altíssima correlação mútua. Qualquer variação na iluminância ambiente altera simultaneamente os valores numéricos dos três canais, tornando o RGB extremamente sensível a sombras, mudanças de hora do dia e reflexos especulares.
2. **Espaço HSV (Hue, Saturation, Value):** Projeção cilíndrica do cubo RGB que desacopla a informação cromática pura (Matiz $H$ e Saturação $S$) da intensidade luminosa intrínseca (Valor $V$). Muito útil para segmentação de cores de veículos e placas sob iluminação não controlada.
3. **Escala de Cinza (Luminância $Y$):** Projeção unidimensional ponderada pela sensibilidade fotópica humana (norma ITU-R BT.601):
   $$Y = 0{,}299 R + 0{,}587 G + 0{,}114 B$$
4. **Espectro Infravermelho Próximo (Near-Infrared — NIR, ~850 nm):** Localizado logo além do espectro visível humano (380–750 nm). A utilização de iluminação ativa monocromática em 850 nm combinada com filtros ópticos passa-banda específicos na lente elimina a influência da luz visível solar diurna, faróis coloridos e distorções cromáticas noturnas.

### 3.5 Pipeline de Aquisição Física e Tecnologias de Obturador (*Shutter*)
O pipeline de captura física compreende:
$$\text{Fótons} \xrightarrow{\text{Lente/Filtro}} \text{Fotossítios CMOS} \xrightarrow{\text{Efeito Fotoelétrico}} \text{Cargas Elétricas} \xrightarrow{\text{Amplificador/ADC}} \text{Sinal Digital RAW} \xrightarrow{\text{ISP}} \text{Imagem Final}$$

Um elemento crítico para veículos a 140 km/h (aproximadamente $38{,}89\text{ m/s}$) é a arquitetura do obturador:
* **Rolling Shutter (Obturador Rolante):** Os pixels do sensor são lidos linha por linha sequencialmente ao longo do tempo. Em altas velocidades relativas, um veículo que se desloca durante a varredura sofre uma distorção geométrica severa (*rolling shutter skew*), inclinando os caracteres da placa e deformando a geometria retangular, o que degrada catastroficamente a etapa de segmentação e OCR.
* **Global Shutter (Obturador Global):** Todos os pixels de toda a matriz do sensor iniciam e encerram a integração de fótons exatamente no mesmo instante de tempo. O sinal é armazenado na memória interna de cada fotossítio e lido sem distorção espacial. É **obrigatório** em sistemas ALPR de alta precisão.

### 3.6 Formatos de Armazenamento e Compressão de Dados
* **Formatos Sem Perdas (Lossless — RAW, PNG, TIFF, BMP):** Preservam exatamente o valor de intensidade de cada pixel capturado pelo ADC. Essenciais durante a etapa de inferência em tempo real dentro da memória RAM, pois qualquer distorção matemática nas bordas é evitada.
* **Formatos Com Perdas (Lossy — JPEG, WebP, H.264, H.265):** Utilizam a Transformada Discreta de Cossenos (DCT) ou transformada de ondas (*wavelets*) em blocos espaciais (ex: blocos $8 \times 8$ no JPEG), seguidos de quantização agressiva dos coeficientes de alta frequência espacial. A compressão JPEG excessiva introduz dois artefatos destrutivos para a Visão Computacional:
  1. *Blocking Artifacts* (descontinuidade nas fronteiras entre blocos $8 \times 8$);
  2. *Ringing Artifacts* (fenômeno de Gibbs) ao redor de bordas abruptas de alto contraste, como o contorno preto dos caracteres em fundo branco.

---

## 4. Análise Técnica e Respostas Fundamentadas às Questões Críticas da Atividade

A seguir, responde-se ponto a ponto às seis perguntas estruturantes da Atividade 6, articulando a teoria matemática e física com as restrições reais de engenharia da concessão rodoviária.

### 4.1 Qual é a relevância da resolução espacial para o seu problema? É preciso uma resolução alta ou não?

**Resposta Técnica e Argumentação:**  
A resolução espacial é **crucial**, mas deve ser compreendida como **densidade de amostragem no objeto-alvo (Pixels Por Metro — PPM)** e não como simples contagem genérica de megapixels no sensor.

1. **Requisito Métrico do Caractere:**  
   A placa veicular padrão Mercosul possui largura de $400\text{ mm}$ e altura de $130\text{ mm}$. Cada caractere alfanumérico tem altura física de $65\text{ mm}$ e largura média de $30\text{ mm}$, com traço de aproximadamente $10\text{ mm}$.  
   Para garantir que o módulo de OCR (seja baseado em classificadores clássicos com descritores morfológicos/HOG ou redes neurais convolucionais como CRNN/YOLOv8-tiny) atinja acurácia $> 99{,}5\%$, a literatura e a prática industrial estabelecem uma densidade mínima de:
   * **Mínimo absoluto:** 20 pixels de altura por caractere;
   * **Recomendado para robustez sob chuva/sujeira:** 30 a 35 pixels de altura por caractere.  
   Considerando um caractere de $65\text{ mm} = 0{,}065\text{ m}$, a densidade espacial requerida no plano da pista é:
   $$\text{PPM}_{\text{alvo}} = \frac{30\text{ pixels}}{0{,}065\text{ m}} \approx 461{,}5\text{ pixels/metro}$$
   Isso significa que a placa completa ($0{,}40\text{ m}$ de largura) deve ocupar entre **180 e 220 pixels de largura** na imagem capturada.

2. **Análise de Engenharia: Resolução Global vs Enquadramento Óptico Dedicado:**  
   Em um pórtico *Free-Flow*, cada faixa de rodovia possui largura de aproximadamente $3{,}5\text{ metros}$.  
   * **Abordagem Ingênua (Câmera Única 4K/8K Panorâmica):** Se tentássemos cobrir as 3 faixas ($10{,}5\text{ m}$) com uma única câmera 4K ($3840 \times 2160$ pixels), a densidade espacial por metro seria de:
     $$\text{PPM} = \frac{3840\text{ pixels}}{10{,}5\text{ m}} \approx 365\text{ PPM}$$
     Embora teoricamente próximo do limite, haveria perda drástica de nitidez nas bordas devido à distorção radial da lente grande angular (efeito barril), profundidade de campo rasa e oclusão mútua quando um caminhão na faixa 1 tampa a visão de um veículo na faixa 2. Além disso, processar frames 4K a 60 FPS exigiria servidores com altíssimo consumo energético e custo proibitivo.
   * **Abordagem de Engenharia Adotada (Câmeras Dedicadas por Faixa):**  
     Projeta-se uma câmera dedicada para cada faixa de rolamento ($3{,}5\text{ m}$ de campo de visão horizontal — FOV).  
     Utilizando um sensor industrial com resolução de **2.3 Megapixels (Full HD / $1920 \times 1200$ pixels)**:
     $$\text{PPM} = \frac{1920\text{ pixels}}{3{,}5\text{ m}} \approx 548{,}5\text{ pixels/metro}$$
     Com $548{,}5\text{ PPM}$, a placa veicular ocupará exatamente:
     $$\text{Largura da placa em pixels} = 0{,}40\text{ m} \times 548{,}5\text{ pixels/metro} \approx 219\text{ pixels}$$
     Cada caractere terá:
     $$\text{Altura do caractere em pixels} = 0{,}065\text{ m} \times 548{,}5 \approx 35{,}6\text{ pixels}$$

**Conclusão sobre a Resolução Espacial:**  
A resolução espacial do sensor não precisa ser astronômica (sensores 4K ou 8K são inadequados e ineficientes para essa tarefa). Uma resolução de **2 a 3 Megapixels por faixa** é ideal e perfeitamente dimensionada, desde que combinada com a **distância focal correta da lente**, garantindo alta densidade de pixels no alvo ($> 500\text{ PPM}$) e taxas de quadros de 60 a 100 FPS sem sobrecarregar o hardware computacional de borda.

---

### 4.2 Qual é a relevância da profundidade de cor (resolução radiométrica) para o seu problema? É preciso uma resolução alta ou não?

**Resposta Técnica e Argumentação:**  
A profundidade de cor é de **extrema relevância na etapa de captura e conversão analógico-digital (ADC)**, embora para a inferência final do algoritmo o formato comumente manipulado seja convertido para 8 bits.

1. **A Física da Película Retrorrefletiva e dos Faróis:**  
   Em rodovias abertas, a faixa de intensidades luminosas em uma mesma cena noturna pode facilmente ultrapassar uma razão de $100.000:1$ ($100\text{ dB}$ de faixa dinâmica). Um veículo com faróis de alta potência LED gera pontos de luz intensa concentrada, enquanto a carcaça adjacente pode estar na escuridão. Simultaneamente, a película retrorrefletiva da placa, ao ser atingida pelo flash infravermelho do pórtico, reflete a luz com ganho óptico 100 vezes maior do que a lataria ao redor.

2. **Limitação de 8 bits na Captura:**  
   Se o sensor capturasse diretamente em $8\text{ bits}$ ($256$ níveis de cinza com alcance dinâmico de ~48 dB):
   * Se o tempo de exposição fosse calibrado para ler o contorno do veículo, a placa estaria totalmente saturada no valor $255$ (branco estourado, impossibilitando distinguir os caracteres);
   * Se a exposição fosse reduzida para evitar saturação da placa, qualquer sombra ou sujeira na placa cairia no nível $0$ (preto absoluto), perdendo o contraste entre caractere e fundo.

3. **A Solução de Engenharia: Captura em 10/12 bits com WDR e Mapeamento Adaptativo:**  
   O sistema adota sensores CMOS industriais com conversor A/D nativo de **12 bits** ($4096$ níveis de cinza) e funcionalidade de *Wide Dynamic Range* (WDR) multi-exposição em nível de pixel ($120\text{ dB}$).  
   Com $12\text{ bits}$, dispõe-se de uma faixa dinâmica suficiente para preservar simultaneamente os detalhes da placa retrorrefletiva sem saturação e as bordas de contraste dos caracteres em relevo.  
   No processador de sinal de imagem (*Image Signal Processor* — ISP de borda), aplica-se um algoritmo de **Equalização de Histograma Local Adaptativo (CLAHE)** e mapeamento de tons (*Tone Mapping*), que comprime a faixa dinâmica dos $12\text{ bits}$ em uma matriz final de **8 bits (256 níveis)** otimizada para o classificador OCR.

**Conclusão sobre a Resolução Radiométrica:**  
É imprescindível uma **alta resolução radiométrica na aquisição bruta (10 a 12 bits com sensor WDR de 120 dB)** para capturar a amplitude dinâmica extrema do cenário rodoviário. No entanto, para o treinamento e inferência dos modelos de Visão Computacional, essa informação é linearizada e normalizada em **8 bits**, pois 256 níveis de contraste entre o traço do caractere e o fundo da placa são matematicamente mais que suficientes para algoritmos de binarização (como Otsu adaptativo) e redes neurais convolucionais.

---

### 4.3 Ao tratar os dados do seu problema, o espaço de cor (RGB, HSV, escala de cinza) tem influência? Qual seria escolhido e por quê?

**Resposta Técnica e Argumentação:**  
O espaço de cor possui **influência determinante** na acurácia, imunidade a ruídos e no custo computacional do sistema. Para solucionar o problema com excelência técnica, implementa-se uma **arquitetura de visão dual-stream (dois fluxos especializados)**:

```
[Fluxo 1: Câmera LPR Dedicada]   -->  Iluminação NIR (850 nm)  -->  ESCALA DE CINZA (Mono)  -->  Localização + OCR
[Fluxo 2: Câmera Panorâmica]     -->  Iluminação Visível/Luz Dia -->  RGB / HSV             -->  Contexto / Auditoria
```

#### A Escolha Primária para a Leitura da Placa (LPR): Escala de Cinza sob Iluminação Infravermelha Próxima (NIR ~850 nm)
Para a tarefa essencial de **leitura dos caracteres da placa**, o espaço escolhido é **Escala de Cinza (Monocromático)** capturado diretamente no espectro infravermelho próximo (850 nm). Justificativas técnicas:
1. **Invariância à Iluminação Ambiente:** Ao iluminar a placa com flashes estroboscópicos de LED em 850 nm e equipar a lente com um filtro óptico passa-banda de 850 nm ($\pm 20\text{ nm}$), elimina-se 95% da luz visível proveniente do sol, de postes de vapor de sódio e de faróis dianteiros. O sensor monocromático "enxerga" uma cena estável e previsível, seja ao meio-dia ou à meia-noite.
2. **Eficiência Quântica e Ausência do Filtro de Bayer:** Câmeras coloridas utilizam uma matriz de filtros de cor (CFA — *Bayer Color Filter Array*), onde 50% dos pixels são verdes, 25% vermelhos e 25% azuis. Para gerar uma imagem RGB, é necessário interpolar os pixels vizinhos (*demosaicing*), processo que introduz borramento subpixel nas bordas finas dos caracteres. Um sensor monocromático dedicado não possui filtro de Bayer: 100% dos fotossítios capturam fótons úteis, aumentando a sensibilidade em baixa luz em até 300% e eliminando artefatos de interpolação.
3. **Redução Dimensional de 3x no Custo Computacional:** A leitura de caracteres depende estritamente do **gradiente espacial de luminância** (a diferença de contraste entre o caractere preto e a chapa branca). Processar 1 canal (escala de cinza de 8 bits) em vez de 3 canais (RGB de 24 bits) reduz em **66,7% o volume de memória** transferido pelo barramento PCIe da GPU/NPU e acelera drasticamente as convoluções dos modelos de detecção.

#### O Papel Secundário do RGB e HSV na Câmera de Contexto
A concessionária de rodovias necessita, por exigência do órgão regulador (ANTT) e para auditoria jurídica, gerar uma **fotografia contextual colorida** que comprove a marca, o modelo, a cor do veículo e a pista de rolamento.
* O espaço **RGB** é mantido na câmera de contexto panorâmica para visualização humana e registro probatório em processos administrativos.
* O espaço **HSV** é empregado no pré-processamento de classificação de atributos veiculares: ao isolar o canal de Matiz ($H$) e Saturação ($S$), o sistema consegue classificar com precisão se a carroceria do veículo é "prata", "branca", "preta", "vermelha" ou "azul", permanecendo imune a variações intensas de sombras (que alteram apenas o canal $V$).

---

### 4.4 Para o seu problema, é mais importante o detalhamento individual de cada pixel ou ele aceita perda de informação em troca de arquivos menores e maior rapidez no processamento?

**Resposta Técnica e Argumentação:**  
Esta pergunta aborda um dos dilemas mais clássicos de engenharia de software e hardware em sistemas embarcados. A resposta reside na **bifurcação estrita entre o pipeline de inferência em tempo real e o pipeline de armazenamento/transmissão para auditoria fiscal**:

```
[Frame Bruto RAW/YUV na RAM] ---> [Pipeline 1: SEM PERDAS (Lossless)] ---> Inferência OCR / YOLO (Latência < 50ms)
                                     |
                                     V
                             [Pipeline 2: COM PERDAS CONTROLADAS] ---> Compressão JPEG (Q=85-90) ---> Nuvem / Auditoria
```

#### 1. Na Etapa de Inferência em Tempo Real: Zero Tolerância a Perdas (Prioridade ao Detalhamento Individual do Pixel)
Durante a passagem do veículo pelo pórtico a 140 km/h, o sistema dispõe de menos de **100 milissegundos** para detectar o veículo, recortar a Região de Interesse (ROI da placa), retificar a perspectiva geométrica e submeter os caracteres ao classificador OCR.
* **Por que nenhuma perda é tolerada aqui?**  
  Algoritmos de detecção de bordas (como Sobel e Canny), técnicas de binarização adaptativa (Otsu local) e redes neurais de detecção de caracteres operam extraindo derivadas espaciais direcionais:
  $$\nabla I = \left[ \frac{\partial I}{\partial x}, \, \frac{\partial I}{\partial y} \right]^T$$
  Se a imagem for submetida a compressão JPEG com perdas antes da inferência, a quantização nos blocos $8 \times 8$ da DCT gera artefatos de altas frequências (*ringing*) ao redor das bordas dos caracteres. Isso transforma um "8" em um "B", um "D" em um "0", ou introduz falsas bordas nas ranhuras da placa.
* **Decisão:** O frame capturado pelo sensor via interface GigE Vision / USB3 Vision é descarregado diretamente na memória RAM do processador de borda como **matriz de bytes brutos (RAW / YUV422 / Gray8) sem qualquer compressão**. Todo o processamento algorítmico ocorre na fidelidade absoluta do pixel original.

#### 2. Na Etapa de Armazenamento e Transmissão para Auditoria: Aceitação de Perda Controlada
Após a placa ser reconhecida com sucesso, o sistema precisa transmitir a evidência da passagem (metadados com timestamp, placa reconhecida, velocidade, foto da placa recortada e foto panorâmica do veículo) para os servidores centrais da concessionária e para a ANTT.
* Um pórtico *Free-Flow* com 80.000 veículos/dia gerando 2 fotos de alta resolução não comprimidas por passagem consumiria:
  $$\text{Volume RAW} = 80.000 \times (2{,}3\text{ MB} + 6\text{ MB}) \approx 664\text{ Gigabytes por dia por pórtico!}$$
  Em uma malha com 20 pórticos interligados via links de dados 4G/5G ou fibra óptica, isso geraria um custo de armazenamento e tráfego de dados insustentável ($> 13\text{ Terabytes/mês}$).
* **Decisão:** Para gravação em disco e envio em rede, a imagem da placa recortada (crop de ~300x150 pixels) e a imagem de contexto são comprimidas em **JPEG com fator de qualidade $Q \ge 85$** (ou WebP com compressão com perdas ajustada). Essa compressão reduz o tamanho do arquivo em **cerca de 85% a 90%**, preservando a legibilidade visual absoluta para perícia humana e auditoria jurídica sem comprometer os custos de infraestrutura de nuvem.

---

### 4.5 Você teria controle sobre o processo de captura de dados (imagens) para seu sistema? Como montaria este processo?

**Resposta Técnica e Argumentação:**  
Sim, em um sistema de pórtico *Free-Flow* rodoviário, **o controle absoluto sobre o processo de captura de dados não é apenas possível, mas obrigatório para a viabilidade do projeto**. Em Visão Computacional industrial, delegar a qualidade da imagem ao acaso da iluminação solar ou aos faróis do tráfego resulta invariavelmente em falha operacional.

O processo de captura de dados foi projetado através da seguinte arquitetura mecatrônica e óptica:

#### 1. Geometria Física e Posicionamento no Pórtico
* **Estrutura de Sustentação:** Pórtico metálico treliçado instalado transversalmente sobre a rodovia, a uma altura livre de **5,8 metros** da pista (atendendo às normas do DNIT para passagem de cargas especiais).
* **Ângulo de Apontamento Vertical (*Pitch*):** As câmeras dedicadas de LPR são fixadas com inclinação de **$20^\circ$ a $25^\circ$** em relação à vertical. Esse ângulo foi matematicamente calculado para garantir que a placa frontal/traseira esteja perfeitamente no plano focal quando o veículo estiver a uma distância de $15$ a $22\text{ metros}$ do pórtico, minimizando a distorção trapezoidal de perspectiva e impedindo que a câmera aponte diretamente contra o ângulo de reflexão dos faróis.
* **Ângulo de Apontamento Horizontal (*Yaw*):** Alinhado paralelamente ao eixo central de cada faixa de rolamento ($\le 5^\circ$ de desvio angular), garantindo que os caracteres não sofram deformação por rotação espacial.

#### 2. Iluminação Ativa Estroboscópica Infravermelha (NIR 850 nm)
* Canhões estroboscópicos de LED de alta potência no comprimento de onda de **$850\text{ nm}$**, invisíveis ao olho humano (evitando cegueira momentânea ou distração perigosa para os motoristas à noite).
* Os LEDs operam em regime pulsado (*strobe*), disparando pulsos de luz de altíssima intensidade luminosa com duração ultracurta de **$200\text{ a }500\text{ microssegundos}$**, sincronizados no microssegundo exato com a abertura do obturador da câmera.
* Como o pulso de luz de 850 nm é ordens de grandeza mais potente na superfície da placa do que a luz ambiente ou faróis de carros, a razão sinal-ruído (SNR) resultante é perfeita.

#### 3. Mecanismo de Disparo (*Trigger*) Sincronizado
Em vez de operar a câmera em captura contínua aleatória (*free-running*), utiliza-se um sistema de disparo por evento físico:
* **Sensor de Trigger:** Em pórticos *Free-Flow*, adota-se um sensor laser de varredura (LiDAR 2D/3D industrial) ou radar Doppler montado no pórtico, com detecção volumétrica precisa de passagem do veículo.
* Quando a dianteira do veículo atinge a linha de captura calibrada ($18\text{ metros}$), a controladora de bordo envia um pulso elétrico em nível lógico TTL via optoacoplador para a porta de entrada de *Hardware Trigger* da câmera.
* A câmera inicia a exposição em menos de $10\mu s$ e emite simultaneamente um sinal de saída *Flash Out* para disparar o estroboscópio infravermelho.

#### 4. Seleção da Lente e Controle de Obturação (*Exposure Time / Shutter*)
* **Obturador:** *Global Shutter* com tempo de exposição ultrarrápido travado em:
  $$t_{\text{exposição}} \le \frac{1}{2000}\text{ segundos } (500\,\mu\text{s})$$
  A $140\text{ km/h}$ ($38{,}89\text{ m/s}$), durante uma exposição de $500\,\mu\text{s}$, o veículo desloca-se apenas:
  $$\Delta d = 38{,}89\text{ m/s} \times 0{,}0005\text{ s} \approx 0{,}0194\text{ m} = 1{,}94\text{ cm}$$
  Com a iluminação estroboscópica concentrando a maior parte da energia nos primeiros $200\,\mu\text{s}$, o deslocamento efetivo capturado é de meros **$7{,}7\text{ mm}$**, o que resulta em um borrão de movimento (*motion blur*) de menos de 1 pixel na escala PPM do alvo, gerando caracteres congelados com nitidez cirúrgica.
* **Lente:** Montagem C/CS com distância focal de **$25\text{ mm a }35\text{ mm}$** com diafragma ajustado em abertura média ($f/4$ a $f/5.6$) para garantir profundidade de campo (*Depth of Field* — DOF) profunda de $4\text{ metros}$, mantendo o foco nítido tanto na dianteira de um carro baixo quanto na placa de um caminhão elevado.
* **Filtro Óptico:** Filtro óptico passa-banda de interferência acoplado à lente, com centro em $850\text{ nm}$ e largura de banda de $\pm 25\text{ nm}$, bloqueando qualquer comprimento de onda fora do espectro infravermelho do estroboscópio.

---

### 4.6 Se você tivesse orçamento limitado, qual decisão técnica sacrificaria primeiro: resolução espacial, profundidade de cor, ou o processo de captura?

**Resposta Técnica e Argumentação:**  
Esta pergunta constitui o teste definitivo de maturidade e julgamento técnico de um engenheiro de Visão Computacional. A avaliação sistemática dos três fatores sob restrição orçamentária é detalhada a seguir:

#### 1. O que JAMAIS pode ser sacrificado: O Processo de Captura de Dados
Se o orçamento for cortado, o **processo de captura (iluminação controlada NIR e obturador Global Shutter com sincronismo de trigger) é o único componente absolutamente inegociável**.
* **Fundamentação:** Se uma câmera de 100 Megapixels de R$ 50.000,00 for instalada em um pórtico operando com obturador convencional *Rolling Shutter*, sem estroboscópio infravermelho e sem controle de iluminação, um veículo a 120 km/h à noite produzirá uma imagem com:
  1. *Motion blur* severo (caracteres borrados cobrindo vários centímetros);
  2. *Rolling shutter skew* (caracteres tombados diagonalmente);
  3. Superexposição pelos faróis (*blooming*) ou escuridão total.
* Nenhuma técnica matemática, algoritmo de restauração por desconvolução de Wiener ou modelo de Deep Learning avançado é capaz de recuperar informação que sequer foi amostrada pelos fotossítios do sensor. Se a aquisição física falha, o sistema está morto antes mesmo de a primeira linha de código ser executada (*Garbage In, Garbage Out*).

#### 2. O que se sacrificaria PRIMEIRO: A Resolução Espacial Bruta Global
Entre as opções, a decisão técnica a ser sacrificada em primeiro lugar é a **Resolução Espacial Bruta do Sensor**.
* **Justificativa Técnica de Engenharia:**  
  Muitos projetistas inexperientes acreditam que "mais megapixels equivalem a um sistema melhor", investindo em sensores 4K ($8{,}3\text{ MP}$) ou 8K ($33\text{ MP}$) de alto custo.  
  Se houver limitação orçamentária, pode-se tranquilamente reduzir o sensor para uma câmera industrial de **1.3 Megapixels ($1280 \times 1024$) ou 2.0 Megapixels ($1920 \times 1080$ Full HD)**.  
  Para compensar essa redução de pixels brutos no sensor sem perder a acurácia do OCR, basta realizar um **ajuste puramente óptico**:
  * Em vez de tentar cobrir a pista com uma lente angular, fecha-se o campo de visão (FOV) utilizando uma lente de maior distância focal (ex: de $25\text{ mm}$ para $35\text{ mm}$), enquadrando estritamente a zona de passagem do veículo.
  * Como demonstrado na Seção 4.1, um sensor de $1280 \times 1024$ com FOV fechado em $2{,}8\text{ metros}$ ainda entrega:
    $$\text{PPM} = \frac{1280}{2{,}8} \approx 457\text{ PPM}$$
    o que garante mais de $180\text{ pixels}$ de largura na placa e $30\text{ pixels}$ de altura por caractere, satisfazendo com folga o Teorema de Nyquist e a convergência do OCR.
  * Além da economia de 40% a 60% no custo da câmera, a redução de resolução reduz o tráfego de dados no barramento, permitindo utilizar processadores de borda muito mais baratos (ex: um módulo compacto NVIDIA Jetson Orin Nano em vez de caros servidores industriais com GPUs topo de linha).

#### 3. E quanto à Profundidade de Cor?
A profundidade de cor pode sofrer um sacrifício intermediário controlado: pode-se substituir um sensor de 12 bits de altíssimo custo por um sensor de **10 bits ou até 8 bits com funcionalidade de WDR nativa**, desde que a iluminação estroboscópica infravermelha tenha sua potência finamente calibrada para a distância fixa do veículo. Como o flash é a fonte de luz dominante e a placa é retrorrefletiva em uma geometria conhecida, a faixa dinâmica da placa fica contida em uma janela de intensidades previsível, mitigando a necessidade de sensores radiométricos de nível científico.

**Quadro Síntese da Decisão de Engenharia:**

| Decisão Técnica | Nível de Prioridade | Pode ser Sacrificada? | Impacto se Sacrificada | Alternativa de Engenharia |
| :--- | :---: | :---: | :--- | :--- |
| **Processo de Captura** (Iluminação IR + Shutter Global) | **Máxima (Inviolável)** | **NÃO** | Falha catastrófica: *motion blur*, reflexo de farol, ilegibilidade total. | Nenhuma. Sem captura física controlada, não há visão computacional viável. |
| **Profundidade de Cor** (12 bits WDR vs 8-10 bits) | Média | Parcialmente | Risco de saturação em placas sujas ou sob sol a pino. | Calibrar potência do estroboscópio e exposição automática rápida. |
| **Resolução Espacial** (4K vs Full HD / HD) | Menor | **SIM (Primeiro sacrifício)** | Menor campo de visão panorâmico ao redor do veículo. | Ajustar distância focal da lente (teleobjetiva) para manter PPM alto na placa. |

---

## 5. Especificação da Arquitetura do Sistema de Visão Computacional

Com base nos fundamentos apresentados, a tabela abaixo consolida a especificação de engenharia do subsistema de visão do pórtico *Free-Flow*:

| Componente | Especificação Técnica Selecionada | Justificativa Baseada nos Fundamentos |
| :--- | :--- | :--- |
| **Sensor de Imagem (LPR)** | CMOS Sony Pregius IMX392 / IMX530 (Monocromático) | *Global Shutter* nativo (elimina *skew* a 140 km/h) e altíssima sensibilidade quântica em 850 nm. |
| **Resolução do Sensor** | $1920 \times 1200$ pixels (2.3 Megapixels) por faixa | Entrega $> 540\text{ PPM}$ na pista, garantindo 35 pixels por caractere (atende Nyquist com folga). |
| **Taxa de Amostragem Temporal** | 60 a 100 FPS (com disparo por Hardware Trigger) | Permite registrar no mínimo 3 frames da placa na zona de captura ótima para redundância de leitura. |
| **Profundidade Radiométrica** | 12 bits ADC com conversão ISP para 8 bits via CLAHE | Captura faixa dinâmica de até $100\text{ dB}$ (evita saturação da película retrorrefletiva). |
| **Óptica e Lente** | Lente varifocal industrial 16–35 mm com montagem C, abertura $f/2.8-f/4$ | Profundidade de campo estendida e foco preciso na zona de passagem de 18 a 22 metros. |
| **Filtro Óptico** | Filtro passa-banda de interferência centrado em $850\text{ nm} \pm 20\text{ nm}$ | Bloqueia espectro visível (luz solar, faróis de xenônio/LED e lâmpadas de iluminação pública). |
| **Iluminação Ativa** | Estroboscópio LED Infravermelho Pulsado (850 nm, potência de 120W de pico) | Congela o movimento ($t_{\text{pulso}} \le 300\,\mu\text{s}$), invisível e seguro para os olhos dos motoristas. |
| **Hardware de Borda (Edge AI)** | IPC Fanless Industrial com acelerador NVIDIA Jetson Orin Industrial | Processamento local de inferência em barramento de memória unificada, latência $< 45\text{ ms}$. |
| **Pipeline de Imagem** | RAW (RAM) $\to$ Retificação $\to$ Binarização/OCR $\to$ JPEG ($Q=88$, armazenamento) | Separação estrita entre integridade matemática na inferência e compressão para banco de dados. |

---

## 6. Reflexão Final: Maturidade Técnica e Mentalidade de Engenharia em Visão Computacional

Na última década, o avanço extraordinário do aprendizado profundo difundiu a percepção equivocada de que algoritmos modernos de Inteligência Artificial tornam secundárias as decisões da física de captura. Essa ilusão cai por terra nos primeiros minutos de operação em ambiente industrial e rodoviário real.

Em um sistema crítico de pedágio *Free-Flow* — onde um erro de 0,5% na leitura diária representa mais de 400 veículos não tarifados ou cobrados indevidamente por dia por pórtico —, **a qualidade do dado amostrado no domínio físico é o teto absoluto do desempenho algorítmico**. O princípio universal **"Garbage In, Garbage Out" (Lixo Entra, Lixo Sai)** impõe que:

1. Se uma imagem for adquirida com **tempo de exposição excessivo**, o *motion blur* causará a convolução espacial dos caracteres da placa com uma função de espalhamento pontual (*Point Spread Function* — PSF) destrutiva. A informação de alta frequência dos traços é fisicamente perdida no momento da integração dos fotossítios. Nenhuma rede neural poderá "adivinhar" com confiabilidade jurídica se uma mancha borrada era um "8" ou um "B".
2. Se a imagem for adquirida com **sensor de obturador rolante (*rolling shutter*)**, a deformação geométrica induzida pela velocidade relativa do veículo quebrará a proporcionalidade métrica dos caracteres.
3. Se a imagem sofrer **compressão JPEG destrutiva** antes da extração de características, os artefatos de quantização da DCT introduzirão falsos gradientes que confundirão os mapas de atenção dos modelos neurais.
4. Se a **iluminação** for negligenciada, a saturação radiométrica da película retrorrefletiva Mercosul transformará a placa em um retângulo branco uniforme sem qualquer contraste.

Portanto, a verdadeira maturidade de um engenheiro de Visão Computacional não se mede pelo número de camadas de sua rede neural ou pela sofisticação de sua biblioteca de aprendizado de máquina, mas sim pela **sua capacidade de projetar o pipeline completo**: desde a propagação dos fótons na cena, a geometria da óptica, a resposta quântica do silício e a discretização matemática da matriz de pixels, até o processamento embarcado de baixo consumo e a gestão de recursos sob restrições orçamentárias do mundo real. Quando a física da aquisição é dominada com excelência, os algoritmos de visão tornam-se leves, rápidos, determinísticos e extraordinariamente robustos.

---

## 7. Referências Bibliográficas e Fontes Consultadas

1. **GONZALEZ, Rafael C.; WOODS, Richard E.** *Digital Image Processing*. 4. ed. Nova York: Pearson, 2018. 1022 p. (Obra fundamental que embasou as formulações de amostragem matricial, quantização, modelo de iluminação-refletância e compressão DCT).
2. **SZELISKI, Richard.** *Computer Vision: Algorithms and Applications*. 2. ed. Cham: Springer, 2022. 949 p. (Referência para formação de imagem em sensores CMOS, óptica de câmeras, aberrações e restauração de imagens com borrão de movimento).
3. **DU, Shan; IBRAHIM, Mahmoud; SHEHATA, Mohamed; BADAWY, Wael.** Automatic License Plate Recognition (ALPR): A State-of-the-Art Review. *IEEE Transactions on Circuits and Systems for Video Technology*, v. 23, n. 2, p. 311–325, fev. 2013. DOI: [10.1109/TCSVT.2012.2203741](https://doi.org/10.1109/TCSVT.2012.2203741). (Revisão sistemática com parametrização de densidades de pixels por caractere e robustez sob iluminação infravermelha).
4. **BRASIL. CONSELHO NACIONAL DE TRÂNSITO (CONTRAN).** *Resolução nº 780, de 26 de junho de 2019*. Dispõe sobre o novo padrão das Placas de Identificação Veicular (PIV) no âmbito do Mercosul. Brasília: Ministério da Infraestrutura, 2019. (Especificação geométrica, cores de caracteres por categoria e características retrorrefletivas de microprismas).
5. **BRASIL. CONSELHO NACIONAL DE TRÂNSITO (CONTRAN).** *Resolução nº 969, de 20 de junho de 2022*. Consolida as normas sobre o sistema de Placas de Identificação Veicular (PIV). Brasília: Ministério da Infraestrutura, 2022.
6. **SILVA, E. et al.** *Visão computacional na indústria: tendências e exemplos práticos*. São Paulo: Repositório Institucional da Universidade de São Paulo (USP), 2020. Disponível em: <https://repositorio.usp.br/directbitstream/e9aced0b-d8da-4956-b2e0-bd3bcbeceb24/Vis%C3%A3o%20computacional%20na%20ind%C3%BAstria%20tend%C3%AAncias%20e%20exemplos%20pr%C3%A1ticos.pdf>. Acesso em: 13 out. 2026.
7. **EUROPEAN MACHINE VISION ASSOCIATION (EMVA).** *EMVA Standard 1288: Standard for Characterization of Image Sensors and Cameras*. Release 4.0. Barcelona: EMVA, 2021. (Norma internacional para caracterização de alcance dinâmico, sensibilidade quântica e ruído de leitura em sensores de visão industrial).
8. **AUTOMATED IMAGING ASSOCIATION (AIA).** *GigE Vision Standard Specification*. Versão 2.1. Ann Arbor: AIA, 2018. (Padrão de transmissão de dados de imagens não comprimidas via rede Ethernet industrial para sistemas de transporte inteligente).
