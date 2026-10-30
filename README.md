# AC2 — Reconhecimento de Padrões: Qualidade de Vinho Tinto

Repositório da prova prática da disciplina **Reconhecimento de Padrões e Aprendizado de Máquina**.

O objetivo é analisar propriedades químicas de vinhos tintos portugueses (dataset [Wine Quality — UCI](https://archive.ics.uci.edu/ml/datasets/wine+quality)) e aplicar algoritmos de agrupamento (não supervisionado) e classificação (supervisionado) para determinar a qualidade do vinho.

O enunciado completo está em [`AC2_Reconhecimento_de_Padrões.ipynb`](AC2_Reconhecimento_de_Padrões.ipynb) e a resolução em [`work/ac2.ipynb`](work/ac2.ipynb).

## Como executar

**Opção 1 — Docker (recomendada, ambiente idêntico ao de desenvolvimento):**

```bash
docker compose up
```

Depois abra `http://localhost:8888` (token: `rec_padroes`), entre na pasta `work/` e abra o notebook `ac2.ipynb`. Execute de cima a baixo (Kernel → Restart and Run All).

**Opção 2 — Ambiente próprio:** qualquer Jupyter com Python 3.11+ e as dependências listadas em [`work/requirements.txt`](work/requirements.txt):

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r work/requirements.txt
```

O notebook carrega os dados da URL pública da UCI e, se não houver internet, usa automaticamente a cópia local em `data/winequality-red.csv`.

## Estrutura do repositório

```
AC2/
├── AC2_Reconhecimento_de_Padrões.ipynb   # enunciado da prova
├── data/                                 # dataset (tinto, branco e dicionário)
├── work/ac2.ipynb                        # resolução completa
├── docker-compose.yaml                   # ambiente Jupyter (jupyter/scipy-notebook)
└── README.md
```

## O que o notebook faz

| Etapa | Conteúdo |
|---|---|
| **1. Carga e pré-processamento** | Carga via URL com fallback local, verificação de nulos/duplicatas/faixas, criação do alvo binário (`bom` se nota ≥ 6, senão `ruim`) e normalização com `StandardScaler` |
| **Análise exploratória** | Balanceamento das classes, histogramas das 11 variáveis, boxplots por classe e mapa de calor de correlações |
| **2. K-Means** | Curva do Cotovelo e Silhueta (K de 2 a 5), escolha justificada do K, visualização dos clusters via PCA e cruzamento clusters x classes reais |
| **3. KNN e Árvore de Decisão** | Split estratificado 80/20, escolha visual do `k` e da profundidade, métricas (Acurácia, Precisão, Recall, F1), matrizes de confusão, importâncias das variáveis e validação cruzada com 5 pastas |
| **4. Rede neural MLP** | Três arquiteturas de camadas ocultas, curvas de perda (convergência), análise de sobreajuste treino x teste e matriz de confusão da melhor |
| **Conclusão** | Curva ROC/AUC dos três modelos, tabela final comparativa (teste + validação cruzada) e resposta final |

## Principais resultados

| Modelo | Acurácia (teste) | Acurácia (validação cruzada) | AUC (teste) |
|---|---|---|---|
| KNN (k = 9) | 78,4% | 72,6% | 0,855 |
| Árvore de Decisão (prof. 6) | 75,3% | 72,0% | 0,801 |
| **MLP (16, 8)** | **79,1%** | **75,4%** | 0,852 |

- O Cotovelo e a Silhueta indicaram **K = 2** no K-Means; o acordo entre clusters e classes reais foi de 58,7% — a química separa grupos, mas não reproduz sozinha a nota dos degustadores.
- A MLP de 2 camadas ocultas foi o melhor modelo: maior acurácia de teste e de validação cruzada, com sobreajuste pequeno (a versão de 3 camadas decorou o treino: 98,6% no treino x 79,1% no teste).
- Os três modelos concordam com a análise exploratória: **álcool alto, pouca acidez volátil e mais sulfatos** caracterizam os vinhos bons.

## Decisões metodológicas

- **Reprodutibilidade:** semente aleatória fixa (`random_state = 42`) em todas as etapas.
- **Sem vazamento de dados:** no agrupamento o normalizador usa todos os dados (não supervisionado); nos classificadores ele é reajustado apenas com o treino e aplicado ao teste, e a validação cruzada usa pipelines.
- **Split estratificado 80/20:** mantém a proporção bom/ruim no treino e no teste.
- **Linhas duplicadas mantidas (240):** são perfis químicos reais e plausíveis; remover seria descartar informação.
- **Dataset:** apenas o vinho tinto, como pede o enunciado (o branco fica em `data/` se quiser estender).
