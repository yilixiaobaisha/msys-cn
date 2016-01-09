# 介绍 #

最近准备为 Phoenix 系统开发编译器中，为了提高代码的可操作性和高效性，尝试不用lex或者flex，自己手工用 C 语言编写了一个词法分析代码，这里将其贴上，供大家感兴趣的朋友参考。


# 代码 #

## lex.h 文件 ##

```
#ifndef __LEXICAL_H__
#define __LEXICAL_H__

#define LEX_START           0
#define LEX_LABEL           1
#define LEX_NUMBER_INTEGER  2
#define LEX_NUMBER_FLOAT    3
#define LEX_NUMBER_PFLOAT   4
#define LEX_NUMBER_HEX      5
#define LEX_SPACE           6
#define LEX_NEWLINE         7
#define LEX_PSTRING         8
#define LEX_STRING          9
#define LEX_PCHAR          10
#define LEX_CHAR           11
#define LEX_COMMENT        12
#define LEX_COMMENT_P1     13
#define LEX_COMMENT1       14
#define LEX_COMMENT_P2     15
#define LEX_COMMENT_F2     16
#define LEX_COMMENT2       17
#define LEX_OTHERS         18
#define LEX_EOF            19

extern int  lex_state;
extern int  lex_lineno;
extern char lex_buffer[512];

extern int lex();

#endif
```

## lex.c 文件 ##

```
/*****************************************************************************
 * Lexer Design Note
 *
 *   [a-zA-Z_][a-zA-Z0-9_]*	label
 *   1
 *   [0-9]*[.][0-9]*		number(integer|float)
 *   2     3
 *   [.][0-9]*			number(pfloat|float)
 *   4  3
 *   0x[0-9a-fA-F]*		number(integer|hex)
 *   25
 *   [\t ]			space
 *   6
 *   [\n]			newline
 *   7
 *   "* "			pstring|string
 *   8  9
 *   'cb'			pchar|char
 *   10 11
 *   /  /  \n			comment(p1|1)
 *   12 13 14
 *   /  *  * /			comment(p2|f2|2)
 *   12 15 16 17
 *   .				others
 *   18
 *****************************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include "lex.h"
#include "file.h"

int  lex_state;
int  lex_lineno = 1;
char lex_buffer[512];

int lex()
{
	int i = 0;
	static char ch;

	lex_state = LEX_START;

	do {
		if(lex_state == LEX_START) {
			if((ch >= 'a' && ch <= 'z') ||
			   (ch >= 'A' && ch <= 'Z') ||
			   (ch == '_')) {
				lex_buffer[i++] = ch;
				lex_state = LEX_LABEL;
			} else if(ch >= '0' && ch <= '9') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_INTEGER;
			} else if(ch == '\t' || ch == ' ') {
				lex_state = LEX_START;
			} else if(ch == '/') {
				lex_state = LEX_COMMENT;
			} else if(ch == '\n') {
				lex_lineno++;
				lex_state = LEX_NEWLINE;
			} else if(ch == '.') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_PFLOAT;
			} else if(ch == '\"') {
				lex_state = LEX_PSTRING;
			} else if(ch == '\'') {
				lex_state = LEX_PCHAR;
			} else {
				lex_buffer[i++] = ch;
				lex_state = LEX_OTHERS;
			}
		} else if(lex_state == LEX_NEWLINE) {
			break;
		} else if(lex_state == LEX_COMMENT) {
			if(ch == '/') {
				lex_state = LEX_COMMENT_P1;
			} else if(ch == '*') {
				lex_state = LEX_COMMENT_P2;
			} else {
				lex_buffer[i++] = ch;
				lex_state = LEX_OTHERS;
			}
		} else if(lex_state == LEX_COMMENT_F2) {
			if(ch == '/') {
				lex_state = LEX_COMMENT2;
			} else {
				if(ch == '\n') {
					lex_lineno++;
				}
				lex_state = LEX_COMMENT_P2;
			}
		} else if(lex_state == LEX_COMMENT_P2) {
			if(ch == '*') {
				lex_state = LEX_COMMENT_F2;
			} else {
				/* contents of comment */
				if(ch == '\n') {
					lex_lineno++;
				}
			}
		} else if(lex_state == LEX_COMMENT_P1) {
			if(ch == '\n') {
				lex_lineno++;
				lex_state = LEX_COMMENT1;
			} else {
				/* contents of comment */
			}
		} else if(lex_state == LEX_PCHAR) {
			if(ch == '\'') {
				lex_state = LEX_CHAR;
			} else {
				if(ch == '\n') {
					lex_lineno++;
				}
				lex_buffer[i++] = ch;
				lex_state = LEX_PCHAR;
			}
		} else if(lex_state == LEX_CHAR) {
			break;
		} else if(lex_state == LEX_PSTRING) {
			if(ch == '\"') {
				lex_state = LEX_STRING;
			} else {
				if(ch == '\n') {
					lex_lineno++;
				}
				lex_buffer[i++] = ch;
				lex_state = LEX_PSTRING;
			}
		} else if(lex_state == LEX_STRING) {
			break;
		} else if(lex_state == LEX_LABEL) {
			if((ch >= 'a' && ch <= 'z') ||
			   (ch >= 'A' && ch <= 'Z') ||
			   (ch >= '0' && ch <= '9') ||
			   (ch == '_')) {
				lex_buffer[i++] = ch;
				lex_state = LEX_LABEL;
			} else {
				break;
			}
		} else if(lex_state == LEX_NUMBER_INTEGER) {
			if(ch >= '0' && ch <= '9') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_INTEGER;
			} else if(ch == '.') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_FLOAT;
			} else if(ch == 'x') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_HEX;
			} else {
				break;
			}
		} else if(lex_state == LEX_NUMBER_PFLOAT) {
			if(ch >= '0' && ch <= '9') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_FLOAT;
			} else {
				lex_state = LEX_OTHERS;
				break;
			}
		} else if(lex_state == LEX_NUMBER_FLOAT) {
			if(ch >= '0' && ch <= '9') {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_FLOAT;
			} else {
				break;
			}
		} else if(lex_state == LEX_NUMBER_HEX) {
			if((ch >= '0' && ch <= '9') ||
			   (ch >= 'a' && ch <= 'f') ||
			   (ch >= 'A' && ch <= 'F')) {
				lex_buffer[i++] = ch;
				lex_state = LEX_NUMBER_HEX;
			} else {
				break;
			}
		} else if(lex_state == LEX_COMMENT1) {
			break;
		} else if(lex_state == LEX_COMMENT2) {
			break;
		} else if(lex_state == LEX_OTHERS) {
			break;
		} else {
			printf("error in lexer\n");
			break;
		}

		// read next char
		ch = fgetc(file_input);

		// check EOF
		if((file_input != stdin) && feof(file_input)) {
			lex_state = LEX_EOF;
			break;
		} else if(ch == '') {
			lex_state = LEX_EOF;
			break;
		}
	} while(1);

	lex_buffer[i] = '\0';
	return lex_state;
}
```