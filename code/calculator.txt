;David Dieperink   
;ETML   
;Calculatrice   
;13.10.2022   
   
POSX = 0	; position en X   
POSY = 1	; position en Y   
NBONE = 2	; chiffre n 1   
NBTWO = 3	; chiffre n 2   
LENGTH = 4	; longueur des variables sur la pile   
   
	.LOC	0   
	SUB	#LENGTH, SP   
	MOVE	#10, {SP}+POSX   
	MOVE	#10, {SP}+POSY   
	MOVE	#0, {SP}+NBONE   
	MOVE	#0, {SP}+NBTWO   
   
NUMBER= H'600	; nombre entré par l'utilisateur 
NB_DIGIT= H'601	; nombre de digit entré 
NB_PIXLINE= H'602	; nombre de pixel pour la ligne 
OLD_X= H'603	; ancienne valeur de X 
OLD_Y= H'604	; ancienne valeur de Y
NB_NUMBERS= H'605	; nombre de chiffre entré
MULTIPLY_TEN_X= H'606	; permet de multiplié les chiffres par 10
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
	JUMP,EQ	SWITCH	; si c'est égal on appelle la méthode switch 
	COMP	#0, B	; compare si B vaut toujours 0   
	JUMP,NE	DRAW_NUMBERS	; si c'est pas égal on appelle DRAW_NUMBERS 
	JUMP	LOOP	; retour à la boucle de départ
   
DRAW_NUMBERS:   
	SUB	#H'50, B	; retire 50 à B pour ecrire le chiffre
	MOVE	B, NUMBER   
	CALL	DRAW_DIGIT
	ADD	#1, X	; ajoute 1 à X pour la position du chiffre suivant
	COMP	#0, NB_DIGIT
	JUMP,EQ	ADD_NUMBERS
	COMP	#1, NB_DIGIT
	JUMP,EQ	ADD_SECOND_NUMBER
	MOVE	#0, B	; réinitialisation de B   
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

ADD_NUMBERS:
	SUB	#H'30, B
	ADD	#1, NB_DIGIT
	MOVE	B,	{SP}+NBONE	; met la valeur de X dans  NBONE
	MOVE	#0, MULTIPLY_TEN_X
	MOVE	#0, A
	JUMP	MULTIPLY_TEN	; va multiplier le chiffre reçu par 10

MULTIPLY_TEN:
	ADD	#10, A
	INC	MULTIPLY_TEN_X
	COMP	MULTIPLY_TEN_X, B
	JUMP,NE	MULTIPLY_TEN
	MOVE	A, {SP}+NBONE
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

ADD_SECOND_NUMBER:
	ADD	#1, NB_DIGIT
	SUB	#H'30, B
	ADD	B, {SP}+NBONE	; ajout de la valeur de B dans le StackPointer
	MOVE	#0, B	; réinitialisation de B
	MOVE	#0, H'C07	; réinitialisation de l'adresse H'C07   
	JUMP	LOOP	; retour à la boucle de départ

DRAW_DIGIT:     
	MOVE	NUMBER, A   
	CALL	_DrawChar
	RET 
 
SWITCH: 
	MOVE	H'C07, B	; récupere le chiffre appuyé   
	COMP	#1, B	 
	JUMP,EQ	DRAW_ADD	; si b = 1 -> addition 
	COMP	#2, B 
	JUMP,EQ	DRAW_SUB	; si b = 2 -> soustraction
	COMP	#3, B
	JUMP,EQ	DRAW_MULTIPLY	; si b = 3 -> multiplication
	COMP	#4, B
	JUMP,EQ	DRAW_DIVISION	; si b = 4 -> division
	JUMP	SWITCH	; sinon on réappelle switch 

DRAW_ADD:
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
	COMP	#1, NB_NUMBERS
	JUMP,EQ	LOOP
	INC	NB_NUMBERS
	MOVE	X, OLD_X	; récupération de l'ancienne position de X
	MOVE	Y, OLD_Y	; récupération de l'ancienne position de Y
	MOVE	#20, X 
	MOVE	#8, Y 
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
	MOVE	#12, Y 
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