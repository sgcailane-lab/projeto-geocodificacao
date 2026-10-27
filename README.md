# Desafio de Geocodificação - Setores Censitários (CNEFE Bahia)

## Contextualização: 
O desafio faz parte da etapa de nivelamento e avaliação de conhecimento da Liga Acadêmica de Engenharia e Ciência de Dados;

## 1. Abordagem Utilizada
A solução foi desenvolvida em Python utilizando **DuckDB** para orquestrar o cruzamento direto contra o ficheiro massivo do IBGE sem o carregar integralmente para a memória RAM. A arquitetura de *matching* foi dividida em duas camadas (*Waterfall*):
- **Nível 1 (Exato):** Ligação determinística através do CEP limpo, Número da porta e Logradouro padronizado.
- **Nível 2 (Fuzzy Match):** Para os registos não localizados no Nível 1, foi aplicada a função `jaro_winkler_similarity` sobre os logradouros, forçando o processamento num ambiente delimitado (*Blocking*) por CEPs vizinhos e número idêntico. O corte de similaridade foi definido em 85%.

## 2. Passo a Passo de Execução
O código foi estruturado para ser executado no **Google Colab**
1. Fazer o upload dos ficheiros `cnefe_bahia.csv` e da base de busca para o Google Drive.
2. Abrir o ficheiro `projeto_geocodificacao.ipynb` no Google Colab.
3. Ajustar as variáveis de caminho (`caminho_busca` e `caminho_cnefe`) na primeira célula.
4. Executar todas as células (`Runtime > Run all`). O ficheiro final será guardado no Drive.

## 3. Dependências
- `duckdb` (Motor de cruzamento SQL em disco/memória)
- `pandas` (Manipulação estrutural da amostra menor)

## 4. Premissas Adotadas
- Valores nulos ou preenchimentos como "S/N" no campo de número foram uniformizados para a *string* "SN" em ambas as bases para permitir o cruzamento.
- Acentos foram removidos e todas as expressões foram convertidas em maiúsculas antes do processamento.
- Abreviações comuns de vias na base de busca (como R., AV., ROD.) foram expandidas de forma a alinharem com o padrão do CNEFE.

## 5. Estratégias de Baixa Qualidade
Foi implementado o algoritmo *Jaro-Winkler* nativo do DuckDB. Esta estratégia revelou-se eficaz para identificar erros de digitação simples em logradouros, mantendo o custo computacional sob controlo através da técnica de *Blocking* pelas raízes do CEP.

## 6. Limitações Conhecidas
- Endereços com CEPs preenchidos incorretamente com zeros ou totalmente omissos não são apanhados pelo Nível 2 (Fuzzy), pois a lógica exige o *Blocking* por proximidade de código postal para evitar falhas de memória.
- Bairros locais/informais indicados nas consultas que divergem profundamente da catalogação estrita do Censo não garantem correspondência se o logradouro também divergir severamente.
