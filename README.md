# Super-Resolução de Faces com Real-ESRGAN

Este repositório documenta um experimento ponta a ponta de validação do fluxo de treinamento da arquitetura Real-ESRGAN para a restauração de imagens faciais degradadas, etapa inicial para o desenvolvimento do Trabalho de Conclusão de Curso (TCC).

O modelo base e os scripts de treinamento foram bifurcados e adaptados do repositório da dissertação de mestrado de Cristiano Lima Oliveira.

---

## 1. Base de Dados e Preparação do Pipeline
* **Dataset Utilizado:** [Flickr-Faces-HQ (FFHQ) Small] (Amostra de 500 imagens).
* **Justificativa:** Dataset de referência para geração e restauração de rostos humanos, mantendo o escopo da pesquisa final.
* **Pipeline de Degradação (Simulação Real-World):** 
  Para avaliar a capacidade de restauração da rede, os dados originais foram submetidos a um processo de degradação sintética usando OpenCV:
  1. **Gabarito (High Resolution - HR):** Imagens padronizadas para a resolução de `128x128` usando interpolação bicúbica.
  2. **Entrada Degradada (Low Resolution - LR):** Aplicação de desfoque gaussiano (kernel 5x5) e subamostragem (downscale) para `64x64` usando interpolação linear.

As imagens foram divididas em conjuntos de Treino (80%), Validação (10%) e Teste (10%).

---

## 2. Arquitetura e Adaptação de Metaparâmetros
A arquitetura utilizada atua como um *Upscaler de 2x* (elevando as entradas de 64x64 para 128x128). O repositório original estava configurado para um domínio de dados específico de 160x160 (OCR). 

Para adequar o modelo à natureza espacial do FFHQ (potências de 2), o arquivo `config.py` foi modificado, sobrescrevendo os parâmetros estruturais:
* **Resolução Base (HIGH_RES):** Alterada de 160 para 128.
* **Tamanho do Batch (Batch Size):** 4
* **Épocas (Epochs):** 10 (Teste piloto rápido para validação de convergência matemática).
* **Otimizador e Learning Rate:** Adam ($LR = 1 \times 10^{-4}$).
* **Ambiente de Treinamento:** Google Colab (Aceleração por GPU NVIDIA T4).

---

## 3. Métricas e Resultados do Treinamento

O foco deste experimento foi comprovar o fluxo de dados (*DataLoader*), a passagem pela rede sem quebras de dimensão em tensores e a convergência das funções de perda (*Losses*).

### ⚡ Desempenho Computacional
* **Tempo Total de Treinamento (10 épocas):** ~8 minutos (utilizando a GPU NVIDIA T4).

### 📊 Avaliação Quantitativa (Métricas de Reconstrução)
Após a conclusão das 10 épocas do teste piloto, foram calculadas as métricas de qualidade de imagem sobre o conjunto de teste:

| Métrica | Valor Obtido | Descrição |
| :--- | :---: | :--- |
| **PSNR** | `8.12 dB` | *Peak Signal-to-Noise Ratio* (Relação Sinal-Ruído de Pico) |
| **SSIM** | `0.0959` | *Structural Similarity Index Measure* (Similaridade Estrutural) |

> **Análise das Métricas:** Em modelos de super-resolução baseados em GANs, o PSNR e o SSIM tendem a apresentar valores reduzidos nos estágios iniciais (poucas épocas), uma vez que a perda adversarial prioriza a criação de texturas e nitidez percebida (Perceptual Loss) em detrimento da fidelidade matemática estrita de pixel a pixel (MSE). Espera-se a elevação expressiva dessas métricas no treinamento de longa duração.

---

## 4. Análise de Convergência das Losses

O acompanhamento dos gráficos de perda confirma que o pipeline aprendeu corretamente:

* **Gerador (`l_g_percep`):** A Perda Perceptual (VGG) apresentou uma queda acentuada logo nas primeiras iterações (reduzindo de `~25` para a faixa entre `10–13`), comprovando que o gerador aprendeu rapidamente a reconstruir formas e feições faciais coerentes.
* **Equilíbrio da GAN (`l_d_real` / `l_d_fake`):** O discriminador manteve um comportamento estável sem entrar em colapso de modo (*mode collapse*), mantendo a competição ativa e forçando a evolução contínua do gerador.

---

## 5. Evolução Visual do Experimento

*(Entrada Degradada LR -> Restauração pela GAN -> Original HR)*

![Comparativo Real-ESRGAN](caminho_para_sua_imagem_comparativa.jpg)
