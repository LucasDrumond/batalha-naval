# batalha-naval
tentativa simples de batalha naval

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>
#include <string.h>

#define N 5
#define NAVIOS 5

void inicializa(char tab[N][N]) {
     for (int i = 0; i < N; ++i)
          for (int j = 0; j < N; ++j)
                tab[i][j] = '~'; 
}

void imprime_tabuleiro_oculto(char tab[N][N]) {
     printf("  1 2 3 4 5\n");
     for (int i = 0; i < N; ++i) {
          printf("%c ", 'A' + i);
          for (int j = 0; j < N; ++j) {
                char c = tab[i][j];
                if (c == 'X') printf("X ");
                else if (c == 'O') printf("O ");
                else printf(". ");
          }
          printf("\n");
     }
}

void imprime_tabuleiro_proprio(char tab[N][N]) {
     printf("  1 2 3 4 5\n");
     for (int i = 0; i < N; ++i) {
          printf("%c ", 'A' + i);
          for (int j = 0; j < N; ++j) {
                char c = tab[i][j];
                if (c == '~') printf("~ ");
                else if (c == 'S') printf("S ");
                else if (c == 'X') printf("X ");
                else if (c == 'O') printf("O ");
          }
          printf("\n");
     }
}

void coloca_navios_aleatorio(char tab[N][N]) {
     int colocados = 0;
     while (colocados < NAVIOS) {
          int i = rand() % N;
          int j = rand() % N;
          if (tab[i][j] != 'S') {
                tab[i][j] = 'S';
                ++colocados;
          }
     }
}

int parse_coordenada(const char *s, int *li, int *co) {
     if (!s || strlen(s) == 0) return 0;
     char c = toupper(s[0]);
     if (c < 'A' || c > 'A' + N - 1) return 0;
     int row = c - 'A';
     int col = -1;
     for (int k = 1; s[k]; ++k) {
          if (isdigit((unsigned char)s[k])) {
                col = s[k] - '1';
                break;
          }
     }
     if (col < 0 || col >= N) return 0;
     *li = row;
     *co = col;
     return 1;
}

int tiro(char alvo[N][N], int li, int co) {
     if (alvo[li][co] == 'S') {
          alvo[li][co] = 'X'; 
          return 1;
     } else if (alvo[li][co] == '~') {
          alvo[li][co] = 'O'; 
          return 0;
     } else {
          return -1;
     }
}

int conta_navios_restantes(char tab[N][N]) {
     int c = 0;
     for (int i = 0; i < N; ++i)
          for (int j = 0; j < N; ++j)
                if (tab[i][j] == 'S') ++c;
     return c;
}

int main(void) {
     srand((unsigned)time(NULL));
     char tab_jogador[N][N], tab_cpu[N][N];
     inicializa(tab_jogador);
     inicializa(tab_cpu);
     coloca_navios_aleatorio(tab_jogador); 
     coloca_navios_aleatorio(tab_cpu);

     printf("Batalha Naval (versao simplificada)\n");
     printf("Tabuleiro: linhas A-E, colunas 1-5. Objetivo: afundar %d navios.\n", NAVIOS);

     char entrada[100];
     int turno = 0; 

     int cpu_tiros[N][N] = {{0}};

     while (1) {
          printf("\nSeu tabuleiro:\n");
          imprime_tabuleiro_proprio(tab_jogador);
          printf("\nTabuleiro inimigo (oculto):\n");
          imprime_tabuleiro_oculto(tab_cpu);

          if (conta_navios_restantes(tab_cpu) == 0) {
                printf("Parabéns! Você afundou todos os navios inimigos.\n");
                break;
          }
          if (conta_navios_restantes(tab_jogador) == 0) {
                printf("Você perdeu! Todos os seus navios foram afundados.\n");
                break;
          }

          if (turno == 0) {
                int li, co;
                int valido = 0;
                do {
                     printf("\nDigite coordenada para atirar (ex: A1): ");
                     if (!fgets(entrada, sizeof(entrada), stdin)) return 0;
                     entrada[strcspn(entrada, "\n")] = 0;
                     if (!parse_coordenada(entrada, &li, &co)) {
                          printf("Coordenada invalida. Use A1..E5.\n");
                          continue;
                     }
                     int res = tiro(tab_cpu, li, co);
                     if (res == -1) {
                          printf("Voce ja atirou nessa posicao. Tente outra.\n");
                          continue;
                     } else if (res == 1) {
                          printf("Acertou!\n");
                     } else {
                          printf("Agua.\n");
                     }
                     valido = 1;
                } while (!valido);
                turno = 1;
          } else {
                int li, co, res;
                do {
                     li = rand() % N;
                     co = rand() % N;
                } while (cpu_tiros[li][co]);
                cpu_tiros[li][co] = 1;
                res = tiro(tab_jogador, li, co);
                printf("\nCPU atira em %c%d: ", 'A' + li, co + 1);
                if (res == 1) printf("Acertou!\n");
                else printf("Agua.\n");
                turno = 0;
          }
     }

     printf("\nEstado final do seu tabuleiro:\n");
     imprime_tabuleiro_proprio(tab_jogador);
     printf("\nEstado final do tabuleiro inimigo:\n");
     printf("  1 2 3 4 5\n");
     for (int i = 0; i < N; ++i) {
          printf("%c ", 'A' + i);
          for (int j = 0; j < N; ++j) {
                char c = tab_cpu[i][j];
                if (c == '~') printf("~ ");
                else if (c == 'S') printf("S ");
                else if (c == 'X') printf("X ");
                else if (c == 'O') printf("O ");
          }
          printf("\n");
     }

     return 0;
}
