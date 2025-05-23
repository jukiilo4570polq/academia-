# academia-
academia 
# -*- coding: utf-8 -*-
import pandas as pd
import string
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
import nltk
import os


# Configuração mínima necessária
nltk.download(['stopwords', 'punkt'], quiet=True)


def limpar_texto(texto):
    """Limpeza básica do texto"""
    if pd.isna(texto) or not texto.strip():
        return ''
    
    texto = str(texto).lower()
    texto = texto.translate(str.maketrans('', '', string.punctuation))
    palavras = word_tokenize(texto, language='portuguese')
    stop_words = set(stopwords.words('portuguese'))
    return ' '.join([p for p in palavras if p not in stop_words and len(p) > 2])


def analisar_sentimentos(textos):
    """Análise de sentimentos com léxico simples"""
    positivos = {'bom', 'ótimo', 'excelente', 'gostei', 'recomendo'}
    negativos = {'ruim', 'péssimo', 'horrível', 'problema', 'insatisfeito'}
    
    resultados = []
    for texto in textos:
        texto_limpo = limpar_texto(texto)
        palavras = set(texto_limpo.split())
        
        pos = len(palavras & positivos)
        neg = len(palavras & negativos)
        
        if pos > neg:
            resultados.append("Positivo")
        elif neg > pos:
            resultados.append("Negativo")
        else:
            resultados.append("Neutro")
    
    return resultados


def gerar_relatorio(dados, nome_arquivo='analise_formulario.xlsx'):
    """Gera relatório simples em Excel"""
    dados_limpos = dados.copy()
    dados_limpos['Texto_Limpo'] = dados.iloc[:, 1].apply(limpar_texto)
    dados_limpos['Sentimento'] = analisar_sentimentos(dados.iloc[:, 1])
    
    # Estatísticas básicas
    stats = pd.DataFrame({
        'Métrica': ['Total', 'Positivos', 'Negativos', 'Neutros', 'Palavras Únicas'],
        'Valor': [
            len(dados),
            sum(dados_limpos['Sentimento'] == 'Positivo'),
            sum(dados_limpos['Sentimento'] == 'Negativo'),
            sum(dados_limpos['Sentimento'] == 'Neutro'),
            len(set(' '.join(dados_limpos['Texto_Limpo']).split()))
        ]
    })
    
    # Salva em uma única planilha com abas
    with pd.ExcelWriter(nome_arquivo) as writer:
        dados.to_excel(writer, sheet_name='Dados Originais', index=False)
        dados_limpos.to_excel(writer, sheet_name='Análise', index=False)
        stats.to_excel(writer, sheet_name='Resumo', index=False)
    
    print(f"Relatório gerado: {nome_arquivo}")


def main():
    print("\n=== Analisador de Formulários ===\n")
    
    # Encontra o primeiro CSV no diretório
    try:
        arquivo = next(f for f in os.listdir() if f.endswith('.csv'))
        dados = pd.read_csv(arquivo, encoding='utf-8')
        print(f"Arquivo '{arquivo}' carregado ({len(dados)} respostas)")
    except StopIteration:
        print("Nenhum arquivo CSV encontrado")
        return
    except Exception as e:
        print(f"Erro ao ler arquivo: {e}")
        return
    
    # Processa e gera relatório
    gerar_relatorio(dados)


if __name__ == "__main__":
    main()

