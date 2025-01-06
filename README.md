# ronoff-exercicio-3
#include <cs50.h>
#include <stdio.h>
#include <string.h>
#include <stdbool.h>

// Máximos permitidos
#define MAX_VOTERS 100
#define MAX_CANDIDATES 9

// Preferências: preferences[i][j] indica o índice do candidato j-ésimo preferido pelo eleitor i
int preferences[MAX_VOTERS][MAX_CANDIDATES];

// Representa um candidato
typedef struct
{
    string name;
    int votes;
    bool eliminated;
} candidate;

// Array de candidatos
candidate candidates[MAX_CANDIDATES];

// Contadores globais
int voter_count;
int candidate_count;

// Funções
bool vote(int voter, int rank, string name);
void tabulate(void);
bool print_winner(void);
int find_min(void);
bool is_tie(int min);
void eliminate(int min);

int main(int argc, string argv[])
{
    // Verificar número de argumentos
    if (argc < 2)
    {
        printf("Uso: runoff [candidato1 candidato2 ...]\n");
        return 1;
    }

    // Inicializar candidatos
    candidate_count = argc - 1;
    if (candidate_count > MAX_CANDIDATES)
    {
        printf("Número máximo de candidatos é %d\n", MAX_CANDIDATES);
        return 2;
    }
    for (int i = 0; i < candidate_count; i++)
    {
        candidates[i].name = argv[i + 1];
        candidates[i].votes = 0;
        candidates[i].eliminated = false;
    }

    // Obter número de eleitores
    voter_count = get_int("Número de eleitores: ");
    if (voter_count > MAX_VOTERS)
    {
        printf("Número máximo de eleitores é %d\n", MAX_VOTERS);
        return 3;
    }

    // Preencher preferências de votos
    for (int i = 0; i < voter_count; i++)
    {
        for (int j = 0; j < candidate_count; j++)
        {
            string name = get_string("Rank %d: ", j + 1);
            if (!vote(i, j, name))
            {
                printf("Voto inválido.\n");
                return 4;
            }
        }
        printf("\n");
    }

    // Repetir o processo de segundo turno até que haja um vencedor
    while (true)
    {
        tabulate();

        // Verificar se há vencedor
        if (print_winner())
        {
            break;
        }

        // Encontrar menor número de votos e verificar empate
        int min = find_min();
        if (is_tie(min))
        {
            for (int i = 0; i < candidate_count; i++)
            {
                if (!candidates[i].eliminated)
                {
                    printf("%s\n", candidates[i].name);
                }
            }
            break;
        }

        // Eliminar o candidato (ou candidatos) com menor número de votos
        eliminate(min);

        // Resetar contagem de votos
        for (int i = 0; i < candidate_count; i++)
        {
            candidates[i].votes = 0;
        }
    }
    return 0;
}

// Atualizar preferências
bool vote(int voter, int rank, string name)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (strcmp(name, candidates[i].name) == 0)
        {
            preferences[voter][rank] = i;
            return true;
        }
    }
    return false;
}

// Atualizar contagem de votos
void tabulate(void)
{
    for (int i = 0; i < voter_count; i++)
    {
        for (int j = 0; j < candidate_count; j++)
        {
            int candidate_index = preferences[i][j];
            if (!candidates[candidate_index].eliminated)
            {
                candidates[candidate_index].votes++;
                break;
            }
        }
    }
}

// Imprimir vencedor, se houver
bool print_winner(void)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (candidates[i].votes > voter_count / 2)
        {
            printf("%s\n", candidates[i].name);
            return true;
        }
    }
    return false;
}

// Encontrar menor número de votos
int find_min(void)
{
    int min = voter_count;
    for (int i = 0; i < candidate_count; i++)
    {
        if (!candidates[i].eliminated && candidates[i].votes < min)
        {
            min = candidates[i].votes;
        }
    }
    return min;
}

// Verificar empate
bool is_tie(int min)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (!candidates[i].eliminated && candidates[i].votes != min)
        {
            return false;
        }
    }
    return true;
}

// Eliminar candidato(s) com menor número de votos
void eliminate(int min)
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (candidates[i].votes == min)
        {
            candidates[i].eliminated = true;
        }
    }
}
