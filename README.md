# Taller-Bison-y-Flex-1
# Ejemplos
## Ejemplo 1 ##
```python
/* just like Unix wc */
%{
int chars = 0;
int words = 0;
int lines = 0;
%}
%%
[a-zA-Z]+ { words++; chars += strlen(yytext); }
\n { chars++; lines++; }
. { chars++; }
%%
main(int argc, char **argv)
{
yylex();
printf("%8d%8d%8d\n", lines, words, chars);
}
```
## Como se ve en terminal ##
<img width="738" height="387" alt="imagen" src="https://github.com/user-attachments/assets/45e65446-3af7-4234-a048-635214090c96" />

## Ejemplo 2 ##
```python
/* English -> American */
%%
"colour" { printf("color"); }
"flavour" { printf("flavor"); }
"clever" { printf("smart"); }
"smart" { printf("elegant"); }
"conservative" { printf("liberal"); }
. { printf("%s", yytext); }
%%
```
## Como se ve en terminal ##
<img width="814" height="405" alt="imagen" src="https://github.com/user-attachments/assets/32074cbb-3c7f-40f9-bb02-335a3e8876dc" />

## Ejemplo 3 ##
```python
%%
"+"	{ printf("PLUS\n"); }
"-"	{ printf("MINUS\n"); }
"*"	{ printf("TIMES\n"); }
"/"	{ printf("DIVIDE\n"); }
"|"     { printf("ABS\n"); }
[0-9]+	{ printf("NUMBER %s\n", yytext); }
\n      { printf("NEWLINE\n"); }
[ \t] { }
.	{ printf("Mystery character %s\n", yytext); }
%%
```
## Como se ve en terminal ##
<img width="814" height="405" alt="imagen" src="https://github.com/user-attachments/assets/13619a23-719b-4dac-9eb1-6ffbe8d7a5df" />

## Ejemplo 4 ##
```python
%{
   enum yytokentype {
     NUMBER = 258,
     ADD = 259,
     SUB = 260,
     MUL = 261,
     DIV = 262,
     ABS = 263,
     EOL = 264 
   };

   int yylval;

%}

%%
"+"	{ return ADD; }
"-"	{ return SUB; }
"*"	{ return MUL; }
"/"	{ return DIV; }
"|"     { return ABS; }
[0-9]+	{ yylval = atoi(yytext); return NUMBER; }
\n      { return EOL; }
[ \t]   { /* ignore white space */ }
.	{ printf("Mystery character %c\n", *yytext); }
%%
main(int argc, char **argv)
{
  int tok;

  while(tok = yylex()) {
    printf("%d", tok);
    if(tok == NUMBER) printf(" = %d\n", yylval);
    else printf("\n");
  }
}


```
## Como se ve en la Terminal ##
<img width="814" height="295" alt="imagen" src="https://github.com/user-attachments/assets/40eaff56-e704-491b-8a04-c17be72b1540" />

## EJEMPLO 5 ##
```python
%{
# include "fb1-5.tab.h"
%}

%%
"+"	{ return ADD; }
"-"	{ return SUB; }
"*"	{ return MUL; }
"/"	{ return DIV; }
"|"     { return ABS; }
"("     { return OP; }
")"     { return CP; }
[0-9]+	{ yylval = atoi(yytext); return NUMBER; }

\n      { return EOL; }
"//".*  
[ \t]   { /* ignore white space */ }
.	{ yyerror("Mystery character %c\n", *yytext); }
%%
```
## Ejemplo 5 Pt2 ##
```python
%{
#  include <stdio.h>
%}

/* declare tokens */
%token NUMBER
%token ADD SUB MUL DIV ABS
%token OP CP
%token EOL

%%

calclist: /* nothing */
 | calclist exp EOL { printf("= %d\n> ", $2); }
 | calclist EOL { printf("> "); } 
 ;

exp: factor
 | exp ADD exp { $$ = $1 + $3; }
 | exp SUB factor { $$ = $1 - $3; }
 | exp ABS factor { $$ = $1 | $3; }
 ;

factor: term
 | factor MUL term { $$ = $1 * $3; }
 | factor DIV term { $$ = $1 / $3; }
 ;

term: NUMBER
 | ABS term { $$ = $2 >= 0? $2 : - $2; }
 | OP exp CP { $$ = $2; }
 ;
%%
main(int argc, char **argv)
{
  printf("> "); 
  yyparse();
}

yyerror(char *s)
{
  fprintf(stderr, "error: %s\n", s);
}
```
## Como se ve en el Terminal ##


### 2. Preguntas

#### 2.1 Manejo de comentarios

* **Pregunta:** ¿La calculadora aceptará una línea que contenga solo un comentario?
* **Respuesta:**  No, ya que el analizador requiere al menos un token válido. El escáner detecta el comentario y lo elimina, dejando la línea sin contenido para el parser.
* **Solución:** La forma más sencilla es ajustar el escáner para que genere un token especial COMMENT que el parser pueda ignorar explícitamente.

```lex
"//".*        { /* Ignorar comentario */ }
```

---
### 2.3 Operadores de nivel de bits

* **En `fb1-5.l`:**

```lex
"&"   return AND;
"|"   return OR;
```

* **En `fb1-5.y`:**

```yacc
exp: exp AND exp { $$ = $1 & $3; }
    | exp OR exp  { $$ = $1 | $3; }
```

* **Diferencia OR / ABS:**  El operador binario OR requiere dos operandos, mientras que el operador unario ABS se aplica únicamente a un solo valor.

---
#### 2.4 Reconocimiento de tokens (Ej4.l vs Flex)

* **Hallazgos:** La versión escrita manualmente no contempla todas las variantes de espacios en blanco y puede fallar ante caracteres imprevistos, mientras que Flex maneja automáticamente estos casos adicionales.

---
#### 2.5 Limitaciones de Flex

* Lenguajes que presentan dependencias de contexto complejas (por ejemplo, Python, donde la indentación es significativa).
* Lenguajes con sintaxis ambigua o cuyo análisis depende del contexto de ejecución.

---
#### 2.6 Programa de conteo de palabras en C
```c
#include <stdio.h>
#include <ctype.h>

int main() {
    char texto[1000];
    int i = 0, palabras = 0;
    int enPalabra = 0; // 0 = fuera de palabra, 1 = dentro de palabra

    printf("Ingrese un texto: ");
    fgets(texto, sizeof(texto), stdin);

    while (texto[i] != '\0') {
        if (isspace((unsigned char)texto[i])) {
            if (enPalabra) {
                palabras++;
                enPalabra = 0;
            }
        } else {
            enPalabra = 1;
        }
        i++;
    }

    if (enPalabra) {
        palabras++;
    }

    printf("Cantidad de palabras: %d\n", palabras);

    return 0;
}

```
 * **Resultados:** La versión en C ofrece mayor velocidad en archivos grandes debido a su menor sobrecarga en comparación con Flex.
 * **Dificultad:** Presenta menor legibilidad y resulta más complicada de ampliar o mantener.



