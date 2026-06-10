# Att-CadastroPIF
// Estrutura para salvar os dados do cliente
// Tenta abrir o arquivo para leitura e escrita binaria
// Se nao existir, cria do zero inicializando 100 posicoes
// Menu principal do sistema
// Opcão 1: Cadastrar
// Move o ponteiro para a posicao que o usuario escolheu
// Ativa a conta
// Volta para a posicao certa para sobrescrever o registro vazio
// Opcao 2: Consultar
// Garante que a busca comece do inicio do arquivo
// Opcao 3: Atualizar Saldo
// Opcao 4: Encerrar Conta (Remocao logica)
// Opcao 5: Listar todos os clientes ativos
// Opcao 6: Executar o rewind manual
// Opcao 7: Sair do programa
// Fecha o arquivo com seguranca

#include <stdio.h>
#include <stdlib.h>

struct Cliente {
    int conta;
    char nome[50];
    float saldo;
    int ativo;
};

int main(void) {
    FILE *arq;
    struct Cliente cliente;
    
    int opcao = 0;
    int posicao;
    int contaBusca;
    int encontrou;
    float novoSaldo;

    arq = fopen("contas.dat", "r+b");

    if (arq == NULL) {
        arq = fopen("contas.dat", "w+b");
        if (arq == NULL) {
            printf("Erro catastrofico ao criar o arquivo!\n");
            return 1;
        }

        cliente.conta = 0;
        cliente.saldo = 0.0;
        cliente.ativo = 0;

        for (int i = 0; i < 100; i++) {
            fwrite(&cliente, sizeof(struct Cliente), 1, arq);
        }
    }

    while (opcao != 7) {
        printf("\n===== MENU DE CONTAS =====\n");
        printf("1 - Cadastrar cliente\n");
        printf("2 - Consultar cliente\n");
        printf("3 - Atualizar saldo\n");
        printf("4 - Encerrar conta\n");
        printf("5 - Listar clientes\n");
        printf("6 - Rewind (Resetar ponteiro)\n");
        printf("7 - Sair\n");
        printf("Escolha uma opcao: ");
        scanf("%d", &opcao);

        if (opcao == 1) {
            printf("Digite a posicao desejada (0 a 99): ");
            scanf("%d", &posicao);

            fseek(arq, posicao * sizeof(struct Cliente), SEEK_SET);
            fread(&cliente, sizeof(struct Cliente), 1, arq);

            if (cliente.ativo == 1) {
                printf("Erro: Esta posicao ja esta ocupada!\n");
            } else {
                printf("Numero da conta: ");
                scanf("%d", &cliente.conta);
                printf("Nome do titular: ");
                scanf("%s", cliente.nome);
                printf("Saldo inicial: ");
                scanf("%f", &cliente.saldo);
                
                cliente.ativo = 1;

                fseek(arq, posicao * sizeof(struct Cliente), SEEK_SET);
                fwrite(&cliente, sizeof(struct Cliente), 1, arq);

                printf("Cliente cadastrado com sucesso na posicao %d!\n", posicao);
            }
        }

        else if (opcao == 2) {
            printf("Digite o numero da conta para busca: ");
            scanf("%d", &contaBusca);

            rewind(arq);
            encontrou = 0;

            while (fread(&cliente, sizeof(struct Cliente), 1, arq) == 1) {
                if (cliente.ativo == 1 && cliente.conta == contaBusca) {
                    printf("\n--- Cliente Encontrado ---\n");
                    printf("Conta: %d\n", cliente.conta);
                    printf("Nome: %s\n", cliente.nome);
                    printf("Saldo: R$ %.2f\n", cliente.saldo);
                    encontrou = 1;
                    break;
                }
            }

            if (encontrou == 0) {
                printf("Conta %d nao foi encontrada.\n", contaBusca);
            }
        }

        else if (opcao == 3) {
            printf("Digite a conta que deseja alterar: ");
            scanf("%d", &contaBusca);

            rewind(arq);
            encontrou = 0;
            posicao = 0;

            while (fread(&cliente, sizeof(struct Cliente), 1, arq) == 1) {
                if (cliente.ativo == 1 && cliente.conta == contaBusca) {
                    printf("Saldo atual: R$ %.2f\n", cliente.saldo);
                    printf("Digite o novo saldo: ");
                    scanf("%f", &novoSaldo);

                    cliente.saldo = novoSaldo;

                    fseek(arq, posicao * sizeof(struct Cliente), SEEK_SET);
                    fwrite(&cliente, sizeof(struct Cliente), 1, arq);

                    encontrou = 1;
                    printf("Saldo updated com sucesso!\n");
                    break;
                }
                posicao++;
            }

            if (encontrou == 0) {
                printf("Conta nao encontrada para atualizacao.\n");
            }
        }

        else if (opcao == 4) {
            printf("Digite a conta que deseja fechar: ");
            scanf("%d", &contaBusca);

            rewind(arq);
            encontrou = 0;
            posicao = 0;

            while (fread(&cliente, sizeof(struct Cliente), 1, arq) == 1) {
                if (cliente.ativo == 1 && cliente.conta == contaBusca) {
                    cliente.ativo = 0;

                    fseek(arq, posicao * sizeof(struct Cliente), SEEK_SET);
                    fwrite(&cliente, sizeof(struct Cliente), 1, arq);

                    encontrou = 1;
                    printf("Conta encerrada com sucesso!\n");
                    break;
                }
                posicao++;
            }

            if (encontrou == 0) {
                printf("Conta nao encontrada.\n");
            }
        }

        else if (opcao == 5) {
            printf("\n--- LISTAGEM DE CLIENTES ATIVOS ---\n");
            rewind(arq);
            encontrou = 0;
            posicao = 0;

            while (fread(&cliente, sizeof(struct Cliente), 1, arq) == 1) {
                if (cliente.ativo == 1) {
                    printf("[%d] Conta: %d | Nome: %s | Saldo: R$ %.2f\n", posicao, cliente.conta, cliente.nome, cliente.saldo);
                    encontrou = 1;
                }
                posicao++;
            }

            if (encontrou == 0) {
                printf("Nenhum cliente cadastrado no momento.\n");
            }
        }

        else if (opcao == 6) {
            rewind(arq);
            printf("Ponteiro do arquivo resetado para o inicio (Posicao 0)!\n");
        }

        else if (opcao == 7) {
            printf("Fechando arquivos e finalizando o programa...\n");
        }

        else {
            printf("Opcao incorreta! Escolha um numero de 1 a 7.\n");
        }
    }

    fclose(arq);
    return 0;
}
