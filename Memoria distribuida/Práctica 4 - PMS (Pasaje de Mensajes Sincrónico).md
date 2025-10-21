
## Ejercicio 1 - b

Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados. 

a) Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo. 
b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos. 
c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron.

```c
//A

//Procesos : 
// Usaria R procesos RobotExaminador, un proceso Analizador y un proceso Administrador el cual sea el intermediario

//Recursos : 
// Lo recursos que utilizaria seria una cola en el adminisytador para acumular los posibles sitios afectados y que el analizador los vaya pidiendo.

//Comunicaciones : 
//Las comunicacioens necesarias son los canales implicitos que hay por cada par de procesos

```

> Pregunta : Acá va con coordinador ? ya que si no le pongo el coordinador el proceso de robot examinador se quedara esperando a que el analizador reciba su mensaje y no podrá seguir su ejecución. O va sin coordinador ? 
```c
//B

process RobotExaminador[id: 0..R-1]{
	txt sitio;
	while (True){
		BuscarSitioAfectado(sitio);
		Admin!posibleAfectado(sitio);
	}
}

process Analizador{
	txt sitio; 
	while (True){
		Admin!pedido(); //Avisa que quiere un nuevo sitio
		Admin?analizar(sitio); //Espera la respuesta con el sitio
		analizarSitio(sitio);
		//Realiza acción .......
	}
}

process Admin{
	cola buffer; txt sitio; 
	while (True){
		do RobotExaminador[*]?posibleAfectado(sitio) -> push(buffer,sitio); //Pusheo en la cola el proximo sitio a examinar
		
		[] not isEmpty(buffer); Analizador?pedido() -> Analizador!analizar(pop(buffer)); //Le envia el sig sitio a analizar
	}
}

```

```c
process RobotExaminador[id: 0..R-1]{
	txt sitio;
	while (True){
		BuscarSitioAfectado(sitio);
		Admin!posibleAfectado(sitio);
	}
}

process Analizador{
	txt sitio; 
	while (True){
		Admin!pedido(); //Avisa que quiere un nuevo sitio
		Admin?analizar(sitio); //Espera la respuesta con el sitio
		analizarSitio(sitio);
		//Realiza acción .......
	}
}

process Admin{
	cola buffer; txt sitio; 
		do RobotExaminador[*]?posibleAfectado(sitio) -> push(buffer,sitio); //Pusheo en la cola el proximo sitio a examinar
		
		[] not isEmpty(buffer); Analizador?pedido() -> Analizador!analizar(pop(buffer)); //Le envia el sig sitio a analizar

}

```

## Ejercicio 2

En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado.

> Consulta : Acá como el único que me dice que continua su trabajo es el empleado 1, por eso ponemos el admin ? en cambio el empleado 2 y 3 no me lo dice por lo que el admin no aplica para ellos ? 
```c

process Empleado1{
	txt muestra; 
	while (True){
		PreparaMuestraAdn(muestra); 
		Admin!muestraNueva(muestra); 
	}
}

process Empleado2{
	txt muestra, set, res; 
	while (True){
		Admin!muestraArmar(); 
		Admin?envioMuestra(muestra);
		set = ArmarSetMuestra(muestra);
		Empleado3!SetAnalisis(set); 
		Empleado3?ResultadoAnalisis(res); 
		GuardarRespuesta(res); 
	}
}

process Empleado3{
	txt set, respuesta; 
	while(True){
		Empleado2?setAnalisis(set);
		respuesta = Analizar(set); 
		Empleado2!ResultadoAnalisis(respuesta); 
	}
}

process Admin{
	cola buffer; 
	txt muestra; 
	do Empleado1?muestraNueva(muestra) -> buffer.push(muestra)
	[] not isEmpty(buffer); Empleado2?muestraArmar() -> Empleado2!envioMuestra(buffer.pop); 
}
```

## Ejercicio 3 - byc

En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.

a) Considerando que P=1. 
b) Considerando que P>1. 
c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula. 

Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución

```c
//A

process Alumno[id: 0..N-1]{
	int nota; txt examen;  
	HacerExamen(examen); 
	Admin!entregaExamen(examen,id); 
	Profesor?entregaNota(nota); 
}

process Profesor{
	txt examen; int id, nota;
	for i: 1 to N{
		Admin!dameExamen(); 
		Admin?examenACorregir(examen,id); 
		nota = CorregirExamen(examen); 
		Alumno[id]!entregaNota(nota); 
	}
}

process Admin{
	cola buffer; txt examen; int id;  
	do Alumno[*]!entregarExamen(examen, id) -> buffer.push(examen,id); 
	[] not isEmpty(buffer); Profesor?dameExamen() -> Profesor!examenACorregir(buffer.pop); 
}

```

> Acá tengo problemas en como controlar en el admin que corte y mande el corte a los profesores. 
```c
//B

process Alumno[id: 0..N-1]{
	int nota; txt examen;  
	HacerExamen(examen); 
	Admin!entregaExamen(examen,id); 
	Profesor[*]?entregaNota(nota); 
}

//Como debe cortar puede cortar cuando reciba un id no valido
process Profesor[id:0..P-1]{
	txt examen; int idA, nota;
	Admin!dameExamen(id); 
	Admin?examenACorregir(examen,idA); 
	while (idA <> -1){
		nota = CorregirExamen(examen); 
		Alumno[idA]!entregaNota(nota);
		Admin!dameExamen(id); 
		Admin?examenACorregir(examen,idA); 
	}
}

process Admin{
	int cant = 0; 
	cola buffer; txt examen; int idA,idP;  
	
	do Alumno[*]!entregarExamen(examen, idA) -> buffer.push(examen,idA); 
	[] not isEmpty(buffer); Profesor?dameExamen(idP) -> Profesor[idP]!examenACorregir(buffer.pop); 
}

```

```c
//C
```

## Ejercicio 4 - c

En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. 

a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión mutua (sin importar el orden). 
b) Modifique la solución anterior para que el empleado los deje acceder según el orden de su identificador (hasta que la persona i no lo haya usado, la persona i+1 debe esperar). 
c) Modifique la solución a) para que el empleado considere el orden de llegada para dar acceso al simulador. 

Nota: cada persona usa sólo una vez el simulador.

```c
//A

process Empleado{
	int idA; 
	for i: 1 to P{
		Persona[*]?requerirUso(idA); // Espera que alguna persona le haga un signal usando el comodin
		Persona[idA]!pasa(); // Le avisa a la persona que hizo el signal que puede usar el simulador
		Persona[idA]?termineUsar(); // Espera que termine de usar el simulador para pasar a la sig persona
	} 
}

process Persona[id:0..P-1]{
	Empleado!requerirUso(id); // Anuncio mi llegada y mando mi id para que me puedan avisar cuando pasar
	Empleado?pasa();  // Espero a poder usar el simulador
	//Usa el simulador
	Empleado!termineUsar(); //Aviso que termine de usarlo
}

```
> La exclusión mutua es implícita ya que empleado no dejara pasar a otra persona hasta que no reciba el mensaje de la persona que esta usando actualmente el simulador. 

```c
//B

process Empleado{
	int idA; 
	for i: 0 to P-1{
		Persona[i]?requerirUso(); // Espera a que la persona con turno i le haga el signal
		Persona[i]!pasa(); // Le avisa a la persona que puede usar el simulador
		Persona[i]?termineUsar(); // Espera que termine de usar el simulador para pasar a la sig persona
	} 
}

process Persona[id:0..P-1]{
	Empleado!requerirUso(); // Anuncio mi llegada 
	Empleado?pasa();  // Espero a poder usar el simulador
	//Usa el simulador
	Empleado!termineUsar(); //Aviso que termine de usarlo
}

```
> No es necesario el pasaje del id a través del canal ya que como va por orden de id, se quedara esperando en el canal del proceso i esperando el mensaje. 

```c
//C

//Preguntar : No menciona que los procesos deben terminar... lo termino al admin ??

process Admin{
	cola buffer; 
	int idP; int usaron = 0;
	do usaron < P; Persona[*]?requerirUso(idP) -> buffer.push(idP);
	[] not isEmpty(buffer); Empleado?proximo() ->  Empleado!sig(buffer.pop()); usaron++;  
	od
}

process Empleado{
	int idA; 
	for i: 1 to P{
		Admin?proximo(); // Avisa al admin que necesita la proxima persona
		Admin!sig(idA); // Recibe el id de la sig persona en pasar
		
		Persona[idA]!pasa(); // Le avisa a la persona que pase
		Persona[idA]?termineUsar(); // Espera que termine de usar el simulador para pasar a la sig persona
	} 
}

process Persona[id:0..P-1]{
	Admin!requerirUso(id); // Anuncio mi llegada al admin y mando mi id para que me puedan avisar cuando pasar
	Empleado?pasa();  // Espero a poder usar el simulador
	//Usa el simulador
	Empleado!termineUsar(); //Aviso que termine de usarlo
}

```
> Para respetar el orden de llegada es necesario el uso de un admin, el cual vaya encolando las personas y los dos procesos se comuniquen con el mismo. 
## Ejercicio 5 - chequear

En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. 

Nota: cada Espectador una sólo una vez la máquina.

```c

process Espectador[id:0..E-1]{
	Admin!solicitarUso(id); // Aviso al admin de mi llegada y mando mi id para que me avisen cuando es mi turno
	Admin?pasa(); // Espero a que sea mi turno
	UsarMaquina(); 
	Admin!termineUsarla(); // Aviso que termine de usar la maquina y me retiro 
}


// Este se me hace raro : 
//opción 1

process Admin{
	cola buffer; 
	bool libre = True; 
	int usaron = 0; 
	int idA, 
		do usaron < E; libre == true; Espectador[*]?solicitarUso(idA) -> Espectador[idA]!pasa(); libre = False; 
		
		[] usaron < E; libre == False; Espectador[*]?solicitarUso(idA) -> buffer.push(idA); 
		
		[] usaron < E; isEmpty(buffer); Espectador[*]?termineUsarla() ->  libre = True; usaron++;
		
		[] usaron < E; not isEmpty(buffer); Espectador[*]?termineUsarla() -> Espectador[buffer.pop()]!pasa();  
}

//Sino :
//Opción 2 
process Admin{
	cola buffer; 
	bool libre = True; 
	int usaron = 0; 
	int idA, 
		do usaron < E; Espectador[*]?solicitarUso(idA) -> 
		if (libre){
			Espectador[idA]!pasa(); 
			libre = False; 
		}else{
			buffer.push(idA); 
		}
		
		[] usaron < E; Espectador[*]?termineUsarla() ->
		usaron++;
		if ( isEmpty(buffer) ){
			libre = True; 
		} else{
			Espectador[buffer.pop()]!pasa();
		}
}


```