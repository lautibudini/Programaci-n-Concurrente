
### Ejercicio 1 - Semáforos

a) En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona. Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible. 

b) Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t.

```c
//A

bool libre = True; 
cola c; 
sem mutex = 1; 

sem espera([P] 0); 

process Persona[id: 1..P]{
int idProx;
	
	P(mutex)
	if (not libre){        
		cola.push(id);     //Me encolo 
		V(mutex);
		P(espera[id]);     //Me duermo hasta que me llamen
	}else{
		libre = False;     //La ocupo
		V(mutex);
	}
	
	UsarTerminal();
	
	P(mutex);
	if( c.isEmpty){       //Si no hay nadie esperando
		libre = True;     //La dejo libre así el proximo pasa directo
	}else{
		c.pop(idProx);     //Desencolo el proximo
		V(espera[idProx]); //Despierto al que sigue
	}
	V(mutex);
}
```


> Nota : Tener en cuenta que la terminal cuando la liberamos de pushearla a la cola antes de fijarnos el siguiente así no cometemos el error de olvidarnos de hacerlo al evaluar si hay siguiente o no. 
```c
//B

cola terminalesLibres; //Se inicializa con las terminales libres
cola espera;

sem vectorEspera([P] 0 )
sem mutexTerminales = 1;
sem mutexEspera = 1;

int contadorLibres = T;
sem mutexContador;

process personas[id: 1..P]{
int terminal; 
int idSiguientes; 

	P(mutexContador);
	if(contadorLibres == 0){ //Si no hay una terminal libre
		espera.push(id);     //Me encolo
		V(mutexContador);
		P(vectorEspera[id]); //Me duermo
	}else{
		contadorLibre--;  //Decremento la cantidad
		V(mutexContador);		
	}
	
	P(mutexTerminales);
	terminalesLibres.pop(terminal); //Me guardo la proxima terminal libre
	V(mutexTerminales);
	
	UsarTerminal(terminal);  
	
	P(mutexTerminales);
	terminalesLibres.push(terminal); //Pongo la terminal en la cola de libres
	V(mutexTerminales);
	
	P(mutexContador);
	if ( espera.isEmpty()){
		contadorLibre++;  //Incremento ya que no le puede pasar mi terminal a nadie
	}else{
		espera.pop(idSiguiente);      //Saco a la siguiente persona esperando
		V(vectorEspera[idSiguiente]); //Lo despierto
	}
	V(mutexContador)

}
```

### Ejercicio 2 - Semáforos

Implemente una solución para el siguiente problema. Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno. Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la función de validación. Nota: maximizar la concurrencia.

```c

cola transacciones; //Se cargan las transacciones
sem mutexTrans = 1; 

int vectorContador([10] 0) //Para acumular los resultados
sem mutexPosVector([10] 1); 
//Tenemos un vector de semaforos ya que solo debemos verificar que más de un worker
//no acceda a la misma POSICIÓN del vector. Los dos pueden acceder a distintas en
//el mismo tiempo.

int ultimoTramo = 0;       //Para saber cuando teminaron todos
sem mutexUltimoTramo = 1; 

process workers[id: 1..7]{
int actual;
int resultado; 

	P(mutexTrans);
	while (not transacciones.isEmpty() ){ //Si todavia no se procesaron todos
		transacciones.pop (actual);       //Me guardo el que proceso
		V(mutexTrans);
		
		resultado = Validar(actual);
		
		P(mutexPosVector[resultado]);
		vectorContador[resultado]++; 
		V(mutexPosVector[resultado]);
	
		P(mutexTrans); 
	}
	V(mutexTrans); 
	
	//Cuando esta acá es porque ya no hay transacciones por procesar
	P(mutexUltimoTramo);
	ultimoTramo ++;
	if (ultimoTramo == 7 ){  //Si soy el ultimo
		V(mutexUltimoTramo); //Puede hacerse solo al final ya que es el ultimo
		for i: 0 to 9{
			print(v[i]);
		}
	}else{
		V(mutexUltimoTramo); 
	}
	}
	
	//En este caso como solo el ultimo informa puedo hacer a lo ultimo solo un 
	//V(mutexUltimoTramo) ya que no retrasa nada ser el ultimo. 
}

```

### Ejercicio 3 - Semáforos
 

Implemente una solución para el siguiente problema. Se debe simular el uso de una máquina expendedora de gaseosas con capacidad para 100 latas por parte de U usuarios. Además, existe un repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma completa. Luego de la recarga, saca una botella y se retira. Nota: maximizar la concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.


> Nota : No hay un mutex de cantDeLatas xq en este caso hay una SOLA PERSONA EN LA MAQUINA, tampoco el repositor ya que entra solo si no hay latas y el otro espera.

```c

int cantLatas  = 100; //Se inicia con 100 latas

cola espera;
//sem mutexEspera = 1; No lo usamos ya que cola se usa en comun con libre
//el cual tiene su semaforo, por ende no debemos hacer uso de esto 

bool libre = true; 
sem mutexLibre = 1; 

sem vectorEspera([U] 0);

sem necesitaRecarga = 0; 

sem yaHayRecarga = 0; 


process persona[id: 0..U-1]{
Lata lata;
int idSig; 

	P(mutexLibre);
	if( libre ) {       //Si esta libre la maquina
		libre = False;  //La marco como en uso
		V(mutexLibre);
	}else{
		espera.push(id); //Me encolo 
		V(mutexLibre);
		P(vectorEspera[id]); //Espero que sea mi turno
	}
	
	
	if (cantLatas == 0){     //Si no hay latas
		V(necesitaRecarga);  //Le avisa al repositor para recargarla
		P(yaHayRecarga);     // Espera a que ya termine de cargar la maquina
	}
	
	cantLatas--; //Retira la lata
	
	P(mutexEspera);
	if (espera.isEmpty()){      //Si no hay gente esperando 
		libre = True;           //Dejo libre la maquina
	}else{
		espera.pop(idSig);      //Saco la siguiente persona
		V(vectorEspera[idSig]); //Le aviso que pase
	}
	V(mutexEspera); 
}

process repositor{
Lata lata; 
	while(true){
		P(necesitoRecarga); //Va a esperar que lo necesiten
		
		cantLatas = 100; //Las recarga
		
		V(yaHayCarga); //Le avisa que ya cargo la maquina
	}
}

```

### Ejercicio 4 - Monitores

Resolver el siguiente problema. En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.

```c

Monitor colaAcceso{

colaOrdenada c;
cond terminaronDeUsarla;
cond quieroVotar;
cond vectorEspera[N] //Hago un vector de espera ya que no solo es orden de llegada

	procedure llegue(int in id, txt int tipo){
		c.insertarConPrioridad(id,tipo);
		signal(quieroVotar); //Aviso que llegue
		wait(vectorEspera[id]); //Espera hasta que sea su turno
	}
	
	procedure sig(){
	int id;
		if (c.isEmpty()){
			wait(quieroVotar); //Me duermo hasta que alguien me despierte
		}
		c.pop(id);
		signal(vectorEspera[id]); //Despierto al proximo
		wait(terminaronDeUsarla); //Se duerme esperando que terminen de usarla
	}
	
	procedure termine(){
		signal(terminaronDeUsarla); //Aviso que termine de usar la maquina
	}
}

process Persona[id:0..N-1]{
int tipo;
	colaAcceso.llegue(id,tipo);
	Votar();
	colaAcceso.termine();
}

process Autoridad{
	for i: 0 ..N-1{
		colaAcceso.sig();
	}
	
}

```
### Ejercicio 5 - Monitores

Resolver el siguiente problema. En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor conoce previamente a qué equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del grupo debe conocer la cantidad de ejemplares vendidos por el grupo. Nota: maximizar la concurrencia.

```c

Monitor Equipos[id: 1..5]{
	cond espera; 
	int cantLlego = 0;
	int cantVendidosTot = 0; 
	cond esperarVuelta;
	
	procedure llegue(){
		cantLlego++;
		if (cantLlego == 4){
			signal_all(espera); //Los despierto a los demas
		}else{
			wait(espera); //Me duermo hasta que lleguen todos
		}
		
	}
	
	procedure termine(int in misVenidos, int out totVendidos){
		cantVendidosTot+= misVendidos; 
		cantLlego--;
		if (cantLlego > 0){ //Si no soy el ultimo que volvio
			wait(esperarVuelta); //Me duermo hasta que vuelvan todos
		}else{
			signal_all(esperarVuelta); //Despierto a los que estaban dormidos
		}
		totVendidos = cantVendidosTot;
	}
}

process Vendedor[id : 1..20]{
int numero; 
int totVendidos;
int misVendidos; 
	Equipo[numero].llegue();
	misVendidos = Trabajar();
	Equipo[numero].termine(misVendidos, totVendidos);
	//Conoce el total vendido del grupo
}
```

### Ejercicio 6 - Monitores

Resolver el siguiente problema. En una montaña hay 30 escaladores que en una parte de la subida deben utilizar un único paso de a uno a la vez y de acuerdo con el orden de llegada al mismo. Nota: sólo se pueden utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez el paso


```c

Monitor Paso{
	cond espera; 
	int genteEsperando = 0; 
	cond quieroPasar;
	bool libre = True
	
	procedure esperarPaso(int in id){
		if (not libre){
			genteEsperando++;
			wait(esperaPasar); //Espera a poder pasar
		}else{
			libre = false;
		}
	}
	
	procedure termineDePasar(){
		if (genteEsperando == 0){ //Si no hay alguien en la cola
			libre = True;
		}else{
			genteEsperando--;
			signal(esperaPasar);  //Le aviso al sig para que pasar
		}
	}
}

process escaladores[id: 0..29]{
	Paso.esperarPaso(id);
	//Pasa
	Paso.termineDePasar();
}
```