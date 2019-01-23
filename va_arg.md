// on utilise un pointer sur char (ou u8int) pour parcourir un octet par un octet
// et on profite ainsi de la puissance de sizeof sans avoir a faire de divisions sur la taille
// en octets de l'untité
 
char *list;

// récupère le début de aliste d'arguments en se basant sur l'adresse d'un argument connus
// cela fonctionne car les arguments sont poussés à l'envers sur la stack lors de l'appel d'une fonction en C
// le premier argument est poussé en dernier, cela permet a la callee de récupérer les arguments dans l'ordre lors du dépilement

#define va_start(list, from) list = ((char *) &from) + sizeof(from)

/*

void caller() {
	push list
	...
	push b
	push a
	call callee
}

void callee (char a, int b, ...) {
	pop a
	pop b
	pop list
	...
}

*/

// pour recuperer un argument, il suffit de prendre X bits (taille du type de l'argument demandé) du haut de la stack
// ensuite on incrémente d'autant le haut de la stack pour qu'il pointe vers le suivant

// conversion au type, passage par un pointer car list est l'adresse du haut de la stack
*((type *) list)

// passage a l'argument suivant
list += sizeof(type)

#define va_arg(list, type) \
	list += sizeof(type), \
	*((type *) list - sizeof(type))

// pour clore la liste d'arguments, il suffit de l'assignée à nulle
// pour signaler la fin de l'utilisation et éviter des utilisations hasardeuses

#define va_end(list) list = NULL


dans la librairie C, il n'est pas possible de prendre moins de 4 octets (un int)
le compilateur ne connaissant pas la taille des arguments futurs, et pour permettre un passage
d'arguments en arguments plus simple (4 octets par 4 octets), les char sont des int et float des doubles (4 & 8 vs 2 & 6)

