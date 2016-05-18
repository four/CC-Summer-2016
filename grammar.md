Copyright (c) 2015-2016, the Selfie Project authors. All rights reserved. Please see the AUTHORS file for details. Use of this source code is governed by a BSD license that can be found in the LICENSE file.

Selfie is a project of the Computational Systems Group at the Department of Computer Sciences of the University of Salzburg in Austria. For further information and code please refer to:

http://selfie.cs.uni-salzburg.at

This is the grammar of the C Star (C*) programming language.

C* is a small Turing-complete subset of C that includes dereferencing (the * operator) but excludes data structures, bitwise and Boolean operators, and many other features. C* is supposed to be close to the minimum necessary for implementing a self-compiling, single-pass, recursive-descent compiler.

Keywords: int, while, if, else, return, void

| = or
{} = zero or more of it
[] = zero or one of it
() = group


```
digit            = "0" | ... | "9" .                                              // 1, 2, 3, 4, 5, 6, 7, 8, 9

integer          = digit { digit } .                                              //3, 6, 121, 455453242, 0

letter           = "a" | ... | "z" | "A" | ... | "Z" .                            //a, b, ... ,z, A, B, ...,Z

identifier       = letter { letter | digit | "_" } .                              //a, v, s3, a_gd4 - variable name

type             = "int" [ "*" ] .                                                //int* or int

selector         = "[" expression "]" ["[" expression "]"] .                       //for 1dim and 2dim arrays               //[5],[i],[a+45],
                                                                                  //[5*6],[25%5],[x+y-34*a],[34<<67]

cast             = "(" type ")" .                                                 //(int) or (int*)

call             = identifier "(" [ expression { "," expression } ] ")" .         //f(a,b,c+d*a)

literal          = integer | "'" ascii_character "'" .                            // 45, 1, 444343, 'a', 'R'

factor           = [ cast ]                                                       //a, (int)a
                    ( [ "*" ] ( identifier [ selector ] | "(" expression ")" ) |
                      call |                                                      //(int) f(b,45), a, *a
                      literal |                                                   // (int) 34, 45, (int) 'a', 'b'
                      """ { ascii_character } """ ) .                             // 'b'

term             = factor { ( "*" | "/" | "%" ) factor } .                        //56, 56*a, 45/(int)'a'

simpleExpression = [ "-" ] term { ( "+" | "-" ) term } .                          //- 45*a/(int)'a'

shiftExpression  = simpleExpression { ( ">>" | "<<" ) simpleExpression } .

expression       = shiftExpression [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) shiftExpression ] .

while            = "while" "(" expression ")"
                             ( statement |
                               "{" { statement } "}" ) .

if               = "if" "(" expression ")"
                             ( statement |
                               "{" { statement } "}" )
                         [ "else"
                             ( statement |
                               "{" { statement } "}" ) ] .

return           = "return" [ expression ] .

statement        = ( [ "*" ] identifier [ selector ] | "*" "(" expression ")" ) "=" expression ";" |  //int a[3] = ...;  a[6] = ;                               
                  call ";" |
                    while |
                    if |
                    return ";" .

variable         = type identifier [selector]  .

procedure        = "(" [ variable { "," variable } ] ")"                //(int a[10], int b[4]); or (int a[10], int b[4]){ int c[4]; c[3] = 0;}
                    ( ";" | "{" { variable ";" } { statement } "}" ) .

cstar(R)           = { type identifier (  [ selector ] | [ "=" [ cast ] [ "-" ] literal ] ) ";" |     // int a = 3;, int a; int a[5][56]; int a[5];
                   ( "void" | type ) identifier procedure } .                                     //void a(...){...}, int a(...){...}
