[[Axe 1]]
02/10/2022
****

0 = Absence de courant électrique 
1 = Présence de courant

La mémoire d'un ordinateur est un tableau dont chaque case contient un octet 
Chaque case de la mémoire est repérée par son **adresse**, elle représente sa position dans le tableau.


Pour organiser les valeurs dans ce tableau, on passe par l'un des deux concepts suivants: 
- Le petit boutisme (Little-Endian)
	*Celui qui nous intéresse le plus* 
- Le grand boutisme (Big-Endian)


Processus de stockage: 
1. Si u n'est pas (au total) un multiple de 8, on rajoute des 0 jusqu’à ce qu'il le soit (padding)
	11111 00000000 11111111 
	= 00011111 00000000 11111111 
	
2. On découpe le tout en parties divisant le total par 8 
	u1 = 00011111, u2= 00000000, u3 = 11111111