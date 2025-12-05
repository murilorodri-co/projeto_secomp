# 📊 Análise de acidentes por alcool e fadiga — Semana vs. Finais de Semana
Este repositório contém scripts, dados e instruções completas para reproduzir a análise estatística que investigou a hipótese:

> **“Aos finais de semana, há maior incidência de acidentes associados ao consumo de álcool e à fadiga (‘condutor dormindo’).”**

O documento explica passo a passo: preparação do ambiente, limpeza dos dados, análise exploratória, teste de Qui-Quadrado, visualizações e conclusões finais.

## 📄 Descrição do dataset

O arquivo principal é:

```
accidents_2017_to_2023_portugues.csv
```

Colunas utilizadas:
- `dia_semana`
- `horario`
- `causa_acidente`
- outras colunas conforme disponibilidade

Criamos colunas adicionais:
- `final_semana` → True se sábado/domingo  
- `causa_categoria` → Álcool, Fadiga, Outras  
- `hora` → extraída de `horario`  

---

## 🔧 1. Preparação dos dados

### Principais passos:
- Remover NAs críticos
- Padronizar texto
- Categorizar causas
- Criar indicadores semana x final de semana
- Extrair hora numérica

Exemplo de funções utilizadas:

```python
df = pd.read_csv("data/accidents_2017_to_2023_portugues.csv", low_memory=False)

df["dia_semana"] = df["dia_semana"].str.strip().str.lower()
df["causa_acidente"] = df["causa_acidente"].str.strip().str.lower()

df["final_semana"] = df["dia_semana"].isin(["sábado", "domingo"])
```

Classificação da causa:

```python
def categoriza_causa(x):
    if "álcool" in x or "alcool" in x:
        return "Álcool"
    if "dormindo" in x or "fadiga" in x or "sono" in x or "sonol" in x:
        return "Fadiga"
    return "Outras"
```

---

## 🔎 2. Análise exploratória

Inclui:
- Frequência por causa
- Frequência por dia da semana
- Proporção semana x final de semana
- Histogramas de horário
- Heatmaps por hora x dia
- Gráficos de distribuição com paleta personalizada:

```python
paleta_azul = sns.color_palette("Blues", n_colors=3)
```
---

## 🔥 3. Teste estatístico — Qui-Quadrado

Tabela de contingência:

```python
tabela = pd.crosstab(df["final_semana"], df["causa_categoria"])
```

Teste:

```python
from scipy.stats import chi2_contingency
chi2, p, dof, expected = chi2_contingency(tabela)
```

Critério:
- α = 0.05  
- Se `p < 0.05`, há associação entre final de semana e causa do acidente.

O script também exporta:
- tabela observada
- tabela esperada
- gráfico de proporções

---

## 📊 4. Visualizações geradas

Principais gráficos:
- Distribuição por dia da semana
- Proporção semana x fim de semana por causa
- Gráfico de área (tendência semanal)
- Distribuição horária
- Heatmap Sábado x Domingo — **incluindo o código abaixo**:

```python
df_fds = df_causas[df_causas["dia_semana"].isin(["sábado", "domingo"])]
heatmap_data = df_fds.groupby(["dia_semana", "hora"]).size().unstack(fill_value=0)
heatmap_data = heatmap_data.reindex(["sábado", "domingo"])

plt.figure(figsize=(12,6))
sns.heatmap(
    heatmap_data,
    cmap="Blues",
    annot=True,
    fmt="d",
    linewidths=0.3,
    cbar_kws={"label": "Número de acidentes"}
)
plt.title("Mapa de calor: frequência de acidentes por álcool e fadiga (Sábado e Domingo)")
plt.xlabel("Hora do dia")
plt.ylabel("Dia da semana")
plt.tight_layout()
plt.show()
```

---

## 🧪 5. Resultados e interpretação

- Se o p-valor for significativo (<0.05), concluímos que:
  > Existe associação entre ocorrer no final de semana e a causa do acidente (álcool/fadiga).

- As visualizações reforçam os períodos de maior risco, especialmente:
  - noites de sábado  
  - madrugadas de domingo  

---

## 🧱 6. Boas práticas e metodologia

- Documentar todas as transformações  
- Salvar versões limpas dos dados  
- Manter scripts reprodutíveis  
- Validar pressupostos do Qui-Quadrado  
- Exibir tabelas de frequências esperadas  
- Manter figuras nomeadas e descritas  

---

## 📚 7. Referências utilizadas

```
CRESWELL, John W. Research design: qualitative, quantitative, and mixed methods approaches. 4. ed. Thousand Oaks: Sage, 2014.

SANTOS, João; LIMA, Mariana. Transformação digital e segurança viária: inovação em gestão de trânsito. São Paulo: Atlas, 2022.

SILVA, Pedro; PEREIRA, Ana. Comportamento de risco no trânsito: álcool e fadiga. Rio de Janeiro: Fiocruz, 2021.

SOUZA, Carla et al. Acidentes de trânsito e fatores humanos: análise estatística. Brasília: Ipea, 2020.

FIELD, Andy. Discovering statistics using IBM SPSS statistics. 5. ed. London: Sage, 2013.
```

---

## 🚀 8. Próximos passos possíveis

- Dashboard em tempo real (Streamlit/Dash)  
- Modelos de machine learning  
- Heatmaps geográficos (caso existam coordenadas)  
- Análises causais avançadas  
- Automação via pipeline (Prefect / Airflow)

---
