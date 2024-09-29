# ferramenta_bancodeDados
Ferramenta para preenchimento automático de tabela csv de pokedex
import os
import pandas as pd
import shutil
import datetime

# Configurações de exibição para pandas
pd.set_option('display.max_columns', None)
pd.set_option('display.expand_frame_repr', False)

# Função para validar os dados de entrada
def validar_dados(dados):
    for chave, valor in dados.items():
        if not valor or (isinstance(valor, list) and not valor[0]):
            print(f"O campo {chave} está vazio.")
            return False
    if not str(dados['id'][0]).isdigit() or not str(dados['Qty.Evolution'][0]).isdigit():
        print("Os campos 'id' e 'Qty.Evolution' devem ser numéricos.")
        return False
    return True

# Função para salvar os dados em um arquivo CSV
def salvar_dados(dataframe, nome_arquivo='Pythondex.csv', diretorio='dados_pokemon', backup_dir='backups'):
    os.makedirs(diretorio, exist_ok=True)
    os.makedirs(backup_dir, exist_ok=True)
    caminho_completo = os.path.join(diretorio, nome_arquivo)
    dataframe.to_csv(caminho_completo, index=False, quoting=1)

    agora = datetime.datetime.now()
    nome_backup = f"backup_{nome_arquivo}_{agora.strftime('%Y%m%d_%H%M%S')}.csv"
    caminho_backup = os.path.join(backup_dir, nome_backup)
    shutil.copy2(caminho_completo, caminho_backup)
    return caminho_completo

# Criar DataFrame inicial com colunas definidas
dados_iniciais = {
    'name': [],
    'type': [],
    'evolutionStage': [],
    'Qty.Evolution': [],
    'id': [],
    'route': []
}
dataframe = pd.DataFrame(dados_iniciais)


# Verificar se o arquivo de dados existe antes de ler
nome_arquivo = 'Pythondex.csv'
if os.path.exists(os.path.join('dados_pokemon', nome_arquivo)):
    dataframe = pd.read_csv(os.path.join('dados_pokemon', nome_arquivo), encoding='latin1')

# Loop principal do programa
continuar = True
while continuar:
    print("\n--- Menu Principal ---")
    print("1. Visualizar Tabela")
    print("2. Adicionar Pokémon")
    print("3. Editar Pokémon")
    print("4. Salvar Dados")
    print("5. Sair")

    escolha = input("Escolha uma opção: ")

    if escolha == '1':
        print("\nTabela Atual:")
        print(dataframe.to_markdown(index=True))
    
    elif escolha == '2':
        nome_pokemon = input("Digite o nome do Pokémon: ")
        tipos_pokemon = input("Digite os tipos do Pokémon, separados por vírgula: ").split(',')
        estagio_evolucao = input("Digite o estágio de evolução do Pokémon: ")
        quantidade_evolucoes = input("Digite a quantidade de evoluções deste Pokémon: ")
        id_pokemon = input("Digite o ID do Pokémon: ")
        rotas= input("Digita as rotas, sepadas por vírgula").split(',')
        dados = {
            'name': [nome_pokemon],
            'type': [','.join(tipos_pokemon)],
            'evolutionStage': [estagio_evolucao],
            'Qty.Evolution': [int(quantidade_evolucoes) if quantidade_evolucoes.isdigit() else 0],
            'id': [int(id_pokemon) if id_pokemon.isdigit() else 0],
            'route': [','.join(rotas)]
        }

        if validar_dados(dados):
            dataframe = pd.concat([dataframe, pd.DataFrame(dados)], ignore_index=True)
            print("Pokémon adicionado com sucesso!")
        else:
            print("Dados inválidos, tente novamente.")

    elif escolha == '3':
        try:
            indices = input("Digite os índices das linhas que deseja editar, separados por vírgula: ")
            indices = list(map(int, indices.split(',')))

            for indice in indices:
                if 0 <= indice < len(dataframe):
                    print(f"Entradas atuais para o índice {indice}:")
                    print(dataframe.iloc[indice])

                    novo_nome = input("Novo nome do Pokémon (ou pressione Enter para manter): ")
                    novos_tipos = input("Novos tipos do Pokémon, separados por vírgula (ou pressione Enter para manter): ")
                    novo_estagio = input("Novo estágio de evolução (ou pressione Enter para manter): ")
                    nova_quantidade = input("Nova quantidade de evoluções (ou pressione Enter para manter): ")
                    novo_id = input("Novo ID do Pokémon (ou pressione Enter para manter): ")
                    novas_rotas = input("Novas rotas de encontro, separadas por vírgula (ou pressione Enter para manter): ")

                    # Atualiza os dados apenas se o usuário fornecer novos valores
                    if novo_nome:
                        dataframe.at[indice, 'name'] = novo_nome
                    if novos_tipos:
                        dataframe.at[indice, 'type'] = ','.join(novos_tipos.split(','))
                    if novo_estagio:
                        dataframe.at[indice, 'evolutionStage'] = novo_estagio
                    if nova_quantidade:
                        dataframe.at[indice, 'Qty.Evolution'] = int(nova_quantidade) if nova_quantidade.isdigit() else dataframe.at[indice, 'Qty.Evolution']
                    if novo_id:
                        dataframe.at[indice, 'id'] = int(novo_id) if novo_id.isdigit() else dataframe.at[indice, 'id']
                    if novas_rotas:
                        dataframe.at[indice, 'route'] = ','.join(novas_rotas.split(','))

                    print(f"Linha {indice} editada com sucesso.")
                else:
                    print(f"Índice {indice} inválido.")
        except ValueError:
            print("Por favor, digite números válidos separados por vírgula.")

    elif escolha == '4':
        salvar = input("Deseja salvar os dados? (s/n): ")
        if salvar.lower() == 's':
            nome_arquivo_salvo = salvar_dados(dataframe)
            print(f"Dados salvos em {nome_arquivo_salvo}")
            

    elif escolha == '5':
        continuar = False
    else:
        print("Opção inválida. Tente novamente.")
