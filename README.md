# Convergência de anotação de preferência humana por macroárea do conhecimento

**Linha de Pesquisa:** Item selection model for human preference benchmarks based on annotation quality prediction

**Membros:** Gabriel Gravina, Ícaro Caniçali e Vitor Sabbagh

Este repositório reproduz o estudo: **Em quais macroáreas do conhecimento a anotação de preferência humana converge?**

- A **MultiPref (MP)** é a base principal, e a **HelpSteer2 (HS2)** é a base de replicação.
- Cada prompt recebe um rótulo de `subject_study` do classificador JEV. Os 53 rótulos são agrupados em 10 macroáreas.
- A variável resposta é o número de pares de anotadores que preferiram respostas opostas.
- Ela é modelada com um GLMM binomial (`gpboost`), que tem a macroárea como variável explicativa, controles das respostas e efeitos aleatórios de prompt.

## Estrutura do repositório

| arquivo | conteúdo |
|---|---|
| `Projeto_convergencia_efeito_total.ipynb` | notebook com todo o estudo, do processamento dos dados às figuras |
| `jev_multipref_train_all.csv` | rótulos do JEV para a MP: uma linha por comparação |
| `jev_helpsteer2_train_all.csv` | rótulos do JEV para a HS2: uma linha por item |
| `requirements.txt` | pacotes com as versões exatas |

As partes do notebook são:

1. **Rótulos JEV:** lê os dois .csv e agrupa os 53 rótulos em 10 macroáreas.
2. **Base de Dados:** baixa a MP e a HS2 e monta a variável resposta e as variáveis de controle.
3. **GLMM na MP:** teste de sobredispersão, LRT da macroárea, contrastes e probabilidade prevista por macroárea.
4. **GLMM na HS2:** o mesmo modelo na base de replicação.
5. **Análise dos dados:** figuras e testes de replicação da MP na HS2.
6. **Especialistas e não especialistas na MP:** o mesmo modelo, separado por grupo de anotadores.

O processamento dos dados está nas partes 1 e 2, os modelos nas partes 3 e 4, e a análise e as figuras nas partes 5 e 6.

## Dados

Todos os dados são públicos e não precisam de login. O notebook baixa tudo sozinho, então não é preciso baixar nada antes.

- **Rótulos JEV:** os dois .csv deste repositório. O notebook lê direto do GitHub (`URL_GITHUB`).
  - O classificador JEV foi rodado fora do notebook, porque precisa de uma API Key. Por isso as saídas já vão prontas nos .csv.
  - Colunas usadas: `row_index` (posição na base original + 1), `subject_study` (rótulo), `confidence` (confiança de 0 a 1) e, na MP, `subject_study_original_multipref` (rótulo original da MP).
- **MultiPref:** [allenai/multipref](https://huggingface.co/datasets/allenai/multipref) (licença ODC-BY).
  - `MultiPref-Train.jsonl`: comparações e anotações.
  - `features/multipref-features.jsonl` e `features/helpsteer2pref-features.jsonl`: atributos das respostas.
- **HelpSteer2:** [nvidia/HelpSteer2](https://huggingface.co/datasets/nvidia/HelpSteer2) (licença CC-BY-4.0).
  - `preference/preference.jsonl.gz`: preferências.

Os arquivos do Hugging Face somam cerca de 200 MB e ficam no cache do `huggingface_hub`.

## Ambiente

Use Python 3.12. O estudo foi testado com 3.12.10.

1. Clone o repositório e entre na pasta:

```bash
git clone https://github.com/icarocanicali/IA_medicao_caracterizacao.git
```

```bash
cd IA_medicao_caracterizacao
```

2. Crie um ambiente virtual:

```bash
python -m venv venv
```

3. Ative o ambiente. No Windows:

```bash
venv\Scripts\activate
```

No Linux ou no macOS:

```bash
source venv/bin/activate
```

4. Instale os pacotes:

```bash
pip install -r requirements.txt
```

## Como reproduzir

É preciso ter acesso à internet, porque o notebook baixa os dados.

**Opção 1, no Jupyter:** abra o notebook com o comando abaixo e rode todas as células (*Run > Run All Cells*).

```bash
jupyter lab Projeto_convergencia_efeito_total.ipynb
```

**Opção 2, pelo terminal:** o comando abaixo roda o notebook inteiro e salva as saídas em `executado.ipynb`.

```bash
jupyter nbconvert --to notebook --execute --output executado.ipynb Projeto_convergencia_efeito_total.ipynb
```

O notebook roda do início ao fim sem nenhuma alteração. Dois avisos aparecem e são esperados:

- **`gpboost`:** avisa várias vezes que o lbfgs "parou na primeira iteração". O aviso vem das reparametrizações do modelo, que já começam no ótimo do modelo principal.
- **`huggingface_hub`:** avisa que os downloads são feitos sem login. Esse aviso não muda nada no resultado.

## Tempo e recursos

- Só usa CPU, sem GPU.
- Foi testado num notebook com Intel Core i5-13450HX, 16 GB de RAM e Windows 11.
- Leva cerca de 1 minuto rodando no Jupyter e cerca de 2 minutos com o `nbconvert`, que inclui a abertura do kernel.
- Na primeira execução, some o tempo de download dos dados do Hugging Face (cerca de 200 MB).

## Onde está cada resultado

| seção do notebook | resultado |
|---|---|
| 1. Rótulos JEV | itens por macroárea nas duas bases |
| 2. Base de Dados | tamanho das bases e descritivas dos controles |
| 3. GLMM na MP | teste de sobredispersão, LRT da macroárea, tabela por macroárea e coeficientes dos controles |
| 4. GLMM na HS2 | LRT da macroárea, tabela por macroárea e coeficientes dos controles |
| 5.1 | figura: probabilidade prevista de discordância por macroárea, MP e HS2 |
| 5.2 | figura: razão de chances de cada macroárea contra a média, MP e HS2 |
| 5.3 | tabela de replicação (correlação, direção, interação macroárea × base) e figura MP × HS2 |
| 5.4 | figura: controles nas duas bases |
| 6.1 | modelos por grupo de anotadores (total, especialistas e não especialistas) |
| 6.2 | comparação entre os grupos (correlações e interação macroárea × tipo) |
| 6.3 | figura: razão de chances de cada macroárea por grupo de anotadores |

## Resultados esperados

Os números abaixo devem sair iguais ou muito próximos:

| resultado | valor |
|---|---|
| amostra da MP | 9.034 comparações (4.610 prompts) |
| amostra da HS2 | 4.996 itens |
| LRT da macroárea na MP (9 gl) | 57,5 (p = 4e-09) |
| LRT da macroárea na HS2 (9 gl) | 10,4 (p = 0,32) |
| correlação dos efeitos MP × HS2 | r = 0,31 |
| LRT da interação macroárea × base (9 gl) | 27,0 (p = 0,0014) |
| LRT da macroárea, só especialistas | 47,8 (p = 2,8e-07) |
| LRT da macroárea, só não especialistas | 6,9 (p = 0,65) |
| LRT da interação macroárea × tipo de anotador | 14,6 (p = 0,10) |

## Reprodutibilidade

- As versões exatas dos pacotes estão no `requirements.txt`.
- O único processo aleatório é a simulação do teste de sobredispersão. Ela usa a semente `SEMENTE = 42`, definida na primeira célula.
- O ajuste do GLMM é determinístico.

## Código de terceiros

- **Lista de frases de recusa:** é baseada no XSTest (Röttger et al., 2024), arquivo `evaluation/classify_completions_strmatch.py` de [paul-rottger/xstest](https://github.com/paul-rottger/xstest), licença CC-BY-4.0. A origem também está marcada no próprio código.
- **Atributos das respostas:** os atributos `token_len_diff`, `len_shorter`, `len_longer`, `cosine_sim` e `rouge` vêm dos arquivos de atributos da MultiPref (Allen AI).
- **Teste de sobredispersão:** segue o `testDispersion` do pacote DHARMa, do R.
- **Ajuste do GLMM:** segue o `glmer` do lme4, do R (aproximação de Laplace).
