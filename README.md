# Projeto-final_-programa-o-estruturada
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <locale.h>

typedef struct {
    char nome[60];
    char artista[40];
    char genero[20];
    char status;
} Musica;

/* --------------------------------------------------------------- */
void limpa_buffer(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

/* --------------------------------------------------------------- */
void ler_string(char *s, int tam) {
    fgets(s, tam, stdin);
    s[strcspn(s, "\n")] = '\0';
}

/* --------------------------------------------------------------- */
int tamanho(FILE *arq) {
    long pos = ftell(arq);
    fseek(arq, 0, SEEK_END);
    long fim = ftell(arq);
    fseek(arq, pos, SEEK_SET);
    return (int)(fim / sizeof(Musica));
}

/* --------------------------------------------------------------- */
void cadastrar(FILE *arq) {
    Musica M;
    M.status = ' ';
    char confirma;

    printf("\n=== CADASTRAR MÚSICA ===\n");
    printf("Registro numero: %d\n", tamanho(arq) + 1);

    printf("Nome da música: ");
    ler_string(M.nome, sizeof(M.nome));

    printf("Artista: ");
    ler_string(M.artista, sizeof(M.artista));

    printf("Gênero musical: ");
    ler_string(M.genero, sizeof(M.genero));

    printf("Confirmar (s/n)? ");
    if (scanf("%c", &confirma) != 1) { limpa_buffer(); return; }
    limpa_buffer();

    if (toupper(confirma) == 'S') {
        fseek(arq, 0, SEEK_END);
        fwrite(&M, sizeof(Musica), 1, arq);
        fflush(arq);
        printf("Música cadastrada!\n");
    } else {
        printf("Cancelado.\n");
    }
}

/* --------------------------------------------------------------- */
void consultar(FILE *arq) {
    int nr;
    Musica M;

    printf("\nCodigo da música: ");
    if (scanf("%d", &nr) != 1) { limpa_buffer(); return; }
    limpa_buffer();

    int total = tamanho(arq);
    if (nr <= 0 || nr > total) {
        printf("Codigo invalido.\n");
        return;
    }

    long pos = (long)(nr - 1) * sizeof(Musica);
    fseek(arq, pos, SEEK_SET);
    fread(&M, sizeof(Musica), 1, arq);

    printf("\n=== MÚSICA %d ===\n", nr);
    if (M.status == '*') printf("STATUS: EXCLUIDA\n");

    printf("Nome:    %s\n", M.nome);
    printf("Artista: %s\n", M.artista);
    printf("Gênero:  %s\n", M.genero);
}

/* --------------------------------------------------------------- */
void gerar_arquivo_texto(FILE *arq) {
    FILE *txt = fopen("C:\\ling_c\\Musicas.txt", "w");
    Musica M;
    int i, total = tamanho(arq);
    char status_str[12];

    if (!txt) {
        printf("Erro ao criar arquivo texto.\n");
        return;
    }

    fprintf(txt, "RELATORIO DE MÚSICAS\n\n");
    fprintf(txt, "COD  %-30s %-20s %-15s STATUS\n",
            "NOME", "ARTISTA", "GENERO");
    fprintf(txt, "--------------------------------------------------------------------------\n");

    for (i = 0; i < total; i++) {
        fseek(arq, i * sizeof(Musica), SEEK_SET);
        fread(&M, sizeof(Musica), 1, arq);

        if (M.status == '*') strcpy(status_str, "EXCLUIDA");
        else strcpy(status_str, "ATIVA");

        fprintf(txt, "%03d %-30s %-20s %-15s %s\n",
                i + 1, M.nome, M.artista, M.genero, status_str);
    }

    fclose(txt);
    printf("Arquivo texto gerado com sucesso!\n");
}

/* --------------------------------------------------------------- */
void excluir(FILE *arq) {
    int nr;
    char confirma;
    Musica M;

    printf("\nCodigo da música a excluir: ");
    if (scanf("%d", &nr) != 1) { limpa_buffer(); return; }
    limpa_buffer();

    int total = tamanho(arq);
    if (nr <= 0 || nr > total) {
        printf("Codigo invalido.\n");
        return;
    }

    long pos = (long)(nr - 1) * sizeof(Musica);
    fseek(arq, pos, SEEK_SET);
    fread(&M, sizeof(Musica), 1, arq);

    if (M.status == '*') {
        printf("Registro ja excluido.\n");
        return;
    }

    printf("Confirmar exclusao (s/n)? ");
    if (scanf("%c", &confirma) != 1) { limpa_buffer(); return; }
    limpa_buffer();

    if (toupper(confirma) == 'S') {
        M.status = '*';
        fseek(arq, pos, SEEK_SET);
        fwrite(&M, sizeof(Musica), 1, arq);
        fflush(arq);
        printf("Excluída!\n");
    } else {
        printf("Cancelado.\n");
    }
}

/* --------------------------------------------------------------- */
int main(void) {
    FILE *arq = fopen("C:\\ling_c\\Musicas.dat", "r+b");
    if (!arq) arq = fopen("C:\\ling_c\\Musicas.dat", "w+b");
    if (!arq) {
        printf("Erro ao abrir ou criar arquivo.\n");
        return 1;
    }

    int op;
    do {
        printf("\n========= SISTEMA DE MÚSICAS =========\n");
        printf("1. Cadastrar música\n");
        printf("2. Consultar música\n");
        printf("3. Gerar arquivo texto\n");
        printf("4. Excluir registro\n");
        printf("5. Sair\n");
        printf("--------------------------------------\n");
        printf("Total de músicas: %d\n", tamanho(arq));
        printf("Opcao: ");

        if (scanf("%d", &op) != 1) { limpa_buffer(); continue; }
        limpa_buffer();

        switch (op) {
            case 1: cadastrar(arq); break;
            case 2: consultar(arq); break;
            case 3: gerar_arquivo_texto(arq); break;
            case 4: excluir(arq); break;
            case 5: printf("Saindo...\n"); break;
            default: printf("Opcao invalida!\n");
        }
    } while (op != 5);

    fclose(arq);
    return 0;
}
