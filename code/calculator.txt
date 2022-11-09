;David Dieperink   
;ETML   
;Calculatrice   
;13.10.2022   
   
POSX = 0	; position en X   
POSY = 1	; position en Y   
NBONE = 2	; chiffre n 1   
NBTWO = 3	; chiffre n 2
RESULT = 4	; résultat
LENGTH = 5	; longueur des variables sur la pile   
   
	.LOC	0   
	SUB	#LENGTH, SP   
	MOVE	#10, {SP}+POSX   
	MOVE	#10, {SP}+POSY   
	MOVE	#0, {SP}+NBONE   
	MOVE	#0, {SP}+NBTWO
	MOVE	#0, {SP}+RESULT 
   
NUMBER= H'600	; nombre entré par l'utilisateur 
NB_DIGIT= H'601	; nombre de digit entré 
NB_PIXLINE= H'602	; nombre de pixel pour la ligne 
OLD_X= H'603	; ancienne valeur de X 
OLD_Y= H'604	; ancienne valeur de Y
NB_NUMBERS= H'605	; nombre de chiffre entré
MULTIPLY_TEN_X= H'606	; permet de multiplié les chiffres par 10
OPERATOR_VALUE= H'607	; garde en mémoir quelle opération doit être faite
MULTIPLY_RES_INC= H'608	; permet de faire le résultat de la multiplication
DIVISION_RES_INC= H'609	; permet de faire le résultat de la division
BOOL_RESULT_DRAWED= H'610	; permet de savoir si le résultat à été écrit

MOVE	#0, NB_PIXLINE
MOVE	#0, B   
MOVE	#1, X	; position X à 1   
MOVE	#1, Y	; position Y à 1   
   
LOOP:
	MOVE	#0, A	; réinitialisation de A
	COMP	#0, NB_PIXLINE 
	JUMP,EQ	DRAW_LINE 
	MOVE	H'C07, B	; récupere le chiffre appuyé   
	COMP	#2, NB_DIGIT	; compare si l'utilisateur à entré 2 digit 
	JUMP,EQ	SWITCH_DRAW_SIGN	; si c'est égal on appelle la méthode switch 
	COMP	#0, B	; compare si B vaut toujours 0   
	JUMP,NE	DRAW_NUMBERS	; si c'est pas égal on appelle DRAW_NUMBERS
	COMP	#0, {SP}+RESULT
	JUMP,NE DRAW_RESULT
	JUMP	LOOP	; retour à la boucle de départ
   
DRAW_NUMBERS:   
	SUB	#H'50, B	; retire 50 à B pour ecrire le chiffre
	MOVE	B, NUMBER   
	CALL	DRAW_DIGIT
	ADD	#1, X	; ajoute 1 à X pour la position du chiffre suivant
	COMP	#0, NB_DIGIT
	JUMP,EQ	ADD_FIRST_NUMBER
	COMP	#1, NB_DIGIT
	JUMP,EQ	ADD_SECOND_NUMBER
	MOVE	#0, B	; réinitialisation de B   
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

DRAW_RESULT:
	COMP	#0, BOOL_RESULT_DRAWED	; si le résultat à déjà été écrit on retourne dans LOOP
	JUMP,NE	LOOP
	MOVE	#22, X
	MOVE	#15, Y
	MOVE	{SP}+RESULT, A	; déplace le résultat dans A pour pouvoir l'écrire
	CALL _DrawHexaByte	; ECRIT LE RESULTAT EN HEXADECIMAL PAS EN DECIMAL !!!
	INC	BOOL_RESULT_DRAWED
	JUMP LOOP

DRAW_DIGIT:
	MOVE	NUMBER, A   
	CALL	_DrawChar
	RET 

ADD_FIRST_NUMBER:
	SUB	#H'30, B
	ADD	#1, NB_DIGIT
	COMP	#0,OPERATOR_VALUE
	JUMP,EQ	ADD_NBONE_FIRSTDIGIT	; ajout du premier digit du premier nombre dans le stackpointer NBONE
	COMP	#0,OPERATOR_VALUE
	JUMP,NE	ADD_NBTWO_FIRSTDIGIT	; ajout du premier digit du deuxième nombre dans le stackpointer NBONE

ADD_SECOND_NUMBER:
	ADD	#1, NB_DIGIT
	SUB	#H'30, B
	COMP	#0,OPERATOR_VALUE
	JUMP,EQ	ADD_NBONE_SECDIGIT	; ajout du deuxième digit du premier nombre dans le stackpointer NBONE
	COMP	#0,OPERATOR_VALUE
	JUMP,NE	ADD_NBTWO_SECDIGIT	; ajout du deuxième digit du deuxième nombre dans le stackpointer NBONE

ADD_NBONE_FIRSTDIGIT:
	MOVE	B,	{SP}+NBONE	; met la valeur de X dans NBONE
	MOVE	#0, MULTIPLY_TEN_X
	MOVE	#0, A
	JUMP	MULTIPLY_TEN	; va multiplier le chiffre reçu par 10

ADD_NBTWO_FIRSTDIGIT:
	MOVE	B,	{SP}+NBTWO	; met la valeur de X dans NBTWO
	MOVE	#0, MULTIPLY_TEN_X
	MOVE	#0, A
	JUMP	MULTIPLY_TEN	; va multiplier le chiffre reçu par 10

ADD_NBONE_SECDIGIT:
	ADD	B, {SP}+NBONE	; ajout de la valeur de B dans le StackPointer
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

ADD_NBTWO_SECDIGIT:
	ADD	B, {SP}+NBTWO	; ajout de la valeur de B dans le StackPointer
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07
	COMP	#1,	OPERATOR_VALUE
	JUMP,EQ	STORE_ADD_RESULT	; stock le résultat de l'addition
	COMP	#2,	OPERATOR_VALUE
	JUMP,EQ	STORE_SUB_RESULT	; stock le résultat de la soustraction
	COMP	#3,	OPERATOR_VALUE
	JUMP,EQ	STORE_MULTIPLY_RESULT	; stock le résultat de la multiplication
	COMP	#4,	OPERATOR_VALUE
	JUMP,EQ	STORE_DIVISION_RESULT	; stock le résultat de la division
	JUMP	LOOP	; retour à la boucle de départ

MULTIPLY_TEN:
	ADD	#10, A
	INC	MULTIPLY_TEN_X
	COMP	MULTIPLY_TEN_X, B
	JUMP,NE	MULTIPLY_TEN
	COMP	#0,OPERATOR_VALUE
	JUMP,EQ RESULT_MULTIPLY_TEN_NBONE
	COMP	#0,OPERATOR_VALUE
	JUMP,NE RESULT_MULTIPLY_TEN_NBTWO

RESULT_MULTIPLY_TEN_NBONE:
	MOVE	A, {SP}+NBONE
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

RESULT_MULTIPLY_TEN_NBTWO:
	MOVE	A, {SP}+NBTWO
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

STORE_ADD_RESULT:
	MOVE	{SP}+NBONE, A
	ADD	{SP}+NBTWO, A
	MOVE	A, {SP}+RESULT
	JUMP	LOOP	; retour à la boucle de départ

STORE_SUB_RESULT:
	MOVE	{SP}+NBONE, A
	SUB	{SP}+NBTWO,A 
	MOVE	A, {SP}+RESULT
	JUMP	LOOP	; retour à la boucle de départ

STORE_MULTIPLY_RESULT:
	MOVE	#0, MULTIPLY_RES_INC
	MOVE	#0, A
	MOVE	{SP}+NBTWO, B
	JUMP	MULTIPLY_RESULT
	
MULTIPLY_RESULT:
	ADD	{SP}+NBONE, A
	INC	MULTIPLY_RES_INC
	COMP	MULTIPLY_RES_INC, B
	JUMP,NE	MULTIPLY_RESULT
	MOVE	A, {SP}+RESULT	; stocke le résultat
	MOVE	#0, MULTIPLY_RES_INC
	JUMP	LOOP	; retour à la boucle de départ

STORE_DIVISION_RESULT:
	MOVE	#0, DIVISION_RES_INC
	MOVE	#0, A
	JUMP	DIVISION_RESULT

DIVISION_RESULT:
	ADD	{SP}+NBTWO, A
	INC	DIVISION_RES_INC
	COMP	{SP}+NBONE, A
	JUMP,NE	DIVISION_RESULT
	MOVE	DIVISION_RES_INC, B
	MOVE	B, {SP}+RESULT	; stocke le résultat
	MOVE	#0, DIVISION_RES_INC
	JUMP	LOOP	; retour à la boucle de départ

SWITCH_DRAW_SIGN: 
	MOVE	H'C07, B	; récupere le chiffre appuyé   
	COMP	#1, B	 
	JUMP,EQ	DRAW_ADD	; si b = 1 -> addition 
	COMP	#2, B 
	JUMP,EQ	DRAW_SUB	; si b = 2 -> soustraction
	COMP	#3, B
	JUMP,EQ	DRAW_MULTIPLY	; si b = 3 -> multiplication
	COMP	#4, B
	JUMP,EQ	DRAW_DIVISION	; si b = 4 -> division
	COMP	#0, {SP}+RESULT
	JUMP,EQ	SWITCH_DRAW_SIGN	; sinon on réappelle switch
	MOVE	#0, NB_DIGIT
	JUMP 	LOOP	; retour à la boucle de départ

DRAW_ADD:
	MOVE	#1, OPERATOR_VALUE
	COMP	#1, NB_NUMBERS
	JUMP,EQ	LOOP
	INC	NB_NUMBERS
	MOVE	X, OLD_X	; récupération de l'ancienne position de X
	MOVE	Y, OLD_Y	; récupération de l'ancienne position de Y
	MOVE	#20, X 
	MOVE	#7, Y 
	INC	X 
	CALL	_SetPixel 
	INC	Y 
	CALL	_SetPixel 
	INC	Y 
	CALL	_SetPixel 
	DEC	X 
	DEC	Y 
	CALL	_SetPixel 
	ADD	#2, X 
	CALL	_SetPixel 
	MOVE	OLD_X, X 
	ADD	#1, X 
	MOVE	OLD_Y, Y 
	MOVE	#0, H'C07 
	MOVE	#0, NB_DIGIT 
	JUMP	LOOP	; retour à la boucle de départ

DRAW_SUB:
	MOVE	#2, OPERATOR_VALUE
	COMP	#1, NB_NUMBERS
	JUMP,EQ	LOOP
	INC	NB_NUMBERS
	MOVE	X, OLD_X	; récupération de l'ancienne position de X
	MOVE	Y, OLD_Y	; récupération de l'ancienne position de Y
	MOVE	#20, X 
	MOVE	#8, Y 
	CALL	_SetPixel 
	INC	X 
	CALL	_SetPixel 
	INC	X 
	CALL	_SetPixel 
	MOVE	OLD_X, X 
	ADD	#1, X 
	MOVE	OLD_Y, Y 
	MOVE	#0, H'C07 
	MOVE	#0, NB_DIGIT 
	JUMP	LOOP	; retour à la boucle de départ

DRAW_MULTIPLY:
	MOVE	#3, OPERATOR_VALUE
	COMP	#1, NB_NUMBERS
	JUMP,EQ	LOOP
	INC	NB_NUMBERS
	MOVE	X, OLD_X	; récupération de l'ancienne position de X
	MOVE	Y, OLD_Y	; récupération de l'ancienne position de Y
	MOVE	#20, X 
	MOVE	#7, Y 
	CALL	_SetPixel 
	INC	X
	INC X 
	CALL	_SetPixel
	DEC	X
	INC Y
	CALL	_SetPixel
	INC	Y
	DEC X
	CALL	_SetPixel
	INC X
	INC X
	CALL	_SetPixel
	MOVE	OLD_X, X 
	ADD	#1, X 
	MOVE	OLD_Y, Y 
	MOVE	#0, H'C07 
	MOVE	#0, NB_DIGIT 
	JUMP	LOOP	; retour à la boucle de départ

DRAW_DIVISION:
	MOVE	#4, OPERATOR_VALUE
	COMP	#1, NB_NUMBERS
	JUMP,EQ	LOOP
	INC	NB_NUMBERS
	MOVE	X, OLD_X	; récupération de l'ancienne position de X
	MOVE	Y, OLD_Y	; récupération de l'ancienne position de Y
	MOVE	#22, X 
	MOVE	#7, Y 
	CALL	_SetPixel
	INC	Y
	DEC	X
	CALL	_SetPixel
	INC	Y
	DEC	X
	CALL	_SetPixel
	MOVE	OLD_X, X 
	ADD	#1, X 
	MOVE	OLD_Y, Y 
	MOVE	#0, H'C07 
	MOVE	#0, NB_DIGIT 
	JUMP	LOOP	; retour à la boucle de départ

DRAW_LINE:
	MOVE	#14, Y 
	MOVE	#2, X 
	COMP	#0, NB_PIXLINE 
	JUMP,EQ	FORLOOP 
 
FORLOOP: 
	CALL	_SetPixel	; appel de la méthode _SetPixel pour placer des pixels
	INC	X 
	INC NB_PIXLINE 
	COMP	#29, NB_PIXLINE 
	JUMP,NE	FORLOOP 
	MOVE	#3, X 
	MOVE	#1, Y 
	JUMP	LOOP	; retour à la boucle de départ

END: 