 
### Ejercicio 1 

Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes situaciones: 

a. Existe un único empleado, el cual atiende por orden de llegada. 
b. Ídem a) pero considerando que hay 2 empleados para atender, ¿Qué debe modificarse en la solución anterior? 
c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?

```c
//A
chan atencion(int)

process Cliente[id:1..N]{
	send atencion(id)
}

process Empleado{
int idA;
	while (true){
		recive atencion(idA) 
		//Atiende al cliente
	}
}

```

```c
//B
chan atencion(int)

process Cliente[id:1..N]{
	send atencion(id)
}

process Empleado[Id:1..2{
int idA;
	while (true){
		recive atencion(idA) 
		//Atiende al cliente
	}
}
```

```c
//C
chan atencion(int)
chan pedido(int)
chan siguiente[3](int)

process Cliente[id:1..N]{
	send atencion(id)
}

process Empleado[Id:1..2{
int idA;
	while (true){
		send pedido(id) //El empleado pide el sig al coordinador y le aviso quien soy 	
		receive siguiente[id](idA) //Espero el sig (que me avisen) en MI canal con MI ID 
		if (idA == "vacio"){
			Delay(900)
		}else{
			//Atiende al cliente
		}
	}
}

process Administrador{
int idA,id;
	receive pedido(idA) //Se  demora en el canal esperando solicitud del empleado
	if (Empty(atencion)){  //Si el canal del cliente esta vacio
		id = "vacio"       //Setea el id con vacio 
	}else{
		receive atencion(id) //Recibo el id y lo seteo en la variable id 
	}
	send siguiente[idA](id)  //Envio al empleado quiere laburar la informaación cual sea el caso en el que estamos
}


```

### Ejercicio 2

Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia.

> La resta de la persona en caja (El cual lo hace el admin) se realiza recién cuando la persona termina de ser atendida ???

```c
chan espera(int)
chan respuesta(txt)

chan aviso()
chan pedidoCliente(int)
chan avisoDeCaja(int)

int vectorContador([5] 0)
chan numeroCaja[P](int)

chan esperarAtencion[5](int)
chan esperarComprobante[P](txt)

process cliente[id: 1..P]{
int cola; txt comprobante; 
	send aviso() //Se hace un signal para poder avisarle al admin
	send viso(id) //Se le pide una caja al admin
	receive numeroCaja[id](cola) //Espera el numero de cola
	
	//Se comunica directo con la caja a partir de ahora
	send esperarAtencion[cola](id); //Le manda a la caja y espera
	receive esperarComprobante[id](comprobante);
	//Se retira
}

process caja[id:1..5]{
int idProx; txt comprobante; 
	receive esperarAtencion[id](idProx) //Espera el proximo cliente
	comprobante = GenerarComprobante(id);
	send esperarComprobante[idProx](comprobante)
	
	//Ahora debemos comunicarnos con el administrador
	send aviso() //Se hace un signal para poder avisarle al admin
	send avisoCaja(id); //Le mandamos nuestro id para que nos reste uno en la cant de personas esperando en caja 
}

process administrador{
int idAux, idC, colaMin
	receive aviso() //Se depierta 
	//Ahora debe fijarse quien es el que me desperto 
	if ( isEmpty(pedidoCliente) ){ //Si el cliente no mando nada es de la caja
		receive avisoDeCaja(idAux);
		vectorContador[idAux] --; //Decrementa la cantidad de clientes en la cola de la caja
	}else{ //Es un pedido de un cliente para una caja
		receive aviso(idC);
		colaMin = Min(vectorContador); //El numero de cola con menor cant de gente
		vectorContador[colaMin] ++; //Sumo uno para el nuevo cliente
		send numeroCaja[P](colaMin); //Le envia al cliente el numero de caja donde ir 
	}

}
```

### Ejercicio 3

Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que: 
- Cada cliente realiza un pedido y luego espera a que se lo entreguen. 
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto). 
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente. 
Nota: maximizar la concurrencia.

> Esta solución esta mal : 
```c
chan RecibirPedido(txt, int)
chan RecibirPedidoVendedor(txt, int)

chan esperaComida[C](txt)
chan pedidosACocinar(txt,int)

process cocinero[id:0..1]{
txt pedido; int idA; 
txt comida;

	while (True)
		receive hayPedido(pedido,idA); //Espera que haya algun pedido que se lo manda algun vendedor
		comida = Cocinar(pedido);     
		send esperaComida[idA](comida) ; //Le envia la comida al cliente
}

process vendedor[id:0..2]{
txt pedido; int idA; 
	while (True){
		if (isEmpty (RecibirPedidoVendedor)){
			ReponerPack();
		}else{
			receive RecibirPedidoVendedor(pedido,idA); //Recibe el pedido de algun cliente
			send hayPedido(pedido,idA); //Se los manda a los cocineros
		}

		
	}
}

process cliente[id:0..C-1]{
	txt pedido = GenerarPedido();      //Genera el pedido 
	send RecibirPedido(pedido,id);     //avisar que hay pedido
	receive esperaComida[id](comida);  //Espera que le entreguen su comida
}

process coordinador{
txt pedido; int idA; 
	while (True){
		receive RecibirPedido(pedido,idA); //Recibe el pedido del cliente
		send RecibirPedidoVendedor(pedido,idA);       //Se lo envia a los vendedores
	}
}

```

```c
chan RecibirPedido(txt, int)
chan RecibirPedidoVendedor(txt, int)

chan esperaComida[C](txt)
chan pedidosACocinar(txt,int)

process cocinero[id:0..1]{
txt pedido; int idA; 
txt comida;

	while (True)
		receive hayPedido(pedido,idA); //Espera que haya algun pedido que se lo manda algun vendedor
		comida = Cocinar(pedido);     
		send esperaComida[idA](comida) ; //Le envia la comida al cliente
}

process cliente[id:0..C-1]{
	txt pedido = GenerarPedido();      //Genera el pedido 
	send RecibirPedido(pedido,id);     //avisar que hay pedido
	receive esperaComida[id](comida);  //Espera que le entreguen su comida
}

process vendedor[id:0..2]{
txt pedido; int idA; 
	while (True){
		send recibirSolicitudVendedor(); //Le avisa al coordinador q esta libre
		receive esperaAccion(); //Espera que el coordinador le avise 
		
		if (isEmpty (RecibirPedidoVendedor)){
			ReponerPack();
		}else{
			receive RecibirPedidoVendedor(pedido,idA); //Recibe el pedido del cliente
			send hayPedido(pedido,idA); //Se los manda a los cocineros
		}
		
	}
}


process coordinador{
txt pedido; int idA; 
	while (True){
		receive recibirSolicitudVendedor(); //Se queda esperando que un vendedor pueda atender un pedido
		
		if (not isEmpty(RecibirPedido)){ //Si el cliente solicito algo
		
			receive RecibirPedido(pedido,idA); //Recibe el pedido del cliente
			send RecibirPedidoVendedor(pedido,idA);       //Se lo envia a los vendedores
		}
		send esperaAccion();//Le envia un signal al vendedor ya que hay una acción a realizar
	}
}
```
> Acá lo que hago es en lugar de mandar algo para avisar que no hay cliente, es mandar un signal y si esta vacío el canal donde se envían los pedidos es que debe reponer el vendedor. 
### Ejercicio 4

Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación. 

a) Implemente una solución para el problema descrito. 
b) Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla. 

Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente


```c
//A 
//Acá hacemos uso del if no deterministico ya que no nos aclara si alguien tiene o no prioridad entonces puede ejecutarse cualquiera que sea verdadero. 

queue cabinasLibres(1,2,3,4,5,6,7,8,9,10);
chan avisoAccion();
chan UsoDeMaquina(int);
chan liberoMaquina(int, int);
chan ticket[N](txt); 
chan UsaMaquina[N](int); 

process empelado{
int numMaquina, idA, idB; txt comprobante; 
	for i : 1 to N {
		receive avisoAccion(); //Espera a que pase algo
		if (not isEmpty(UsoDeMaquina)){ 
			//Alguien quiere usar la maquina
			receive UsoDeMaquina(idA); 
			if (cabinasLibre.isEmpty()){
				//No hay cabinas libres
				//Espero a que alguien quiera pagar
				receive liberoMaquina(numMaquina, idB); 
				cabinasLibres.push(numMaquina); //Pushea la cabina como libre
				comprobante = Cobrar(idA); //Le cobra
				send ticket[idA](comprobante);//Le envia el comprobante al cliente
			}
			//Le asigna una cabina
			cabinasLibres.pop(numMaquina); 
			send usaMaquina[idA](numeroMaquina);
		[] (not empty(liberoMaquina))
		//}else{
			//esperamos que alguien quiera pagar
			receive liberoMaquina(numMaquina, idA); 
			cabinasLibres.push(numMaquina); //Pushea la cabina como libre
			comprobante = Cobrar(idA); //Le cobra
			send ticket[idA](comprobante);//Le envia el comprobante al cliente
		}
	}
}

process cliente[id:0..N-1]{
int numeroMaquina; txt comprobante; 
	send UsoDeMaquina(id); //Le mando que quiero usar la maquina
	send avisoAccion(id); 
	receive usaMaquina[id](numeroMaquina); //Espera a que sea su turno
	
	UsarMaquina(numeroMaquina); 
	
	send liberoMaquina(numeroMaquina,id); //Avisa que libero la maquina y quiere pagar
	send avisoAccion(id); 
	receive ticket[id](comprobante);
}
```

```c
//B

//Lo que debe hacerse acá es la misma logica que en el punto A pero debe reemplazarse el if no deterministico por un if-else comun, priorizando así el pago antes de el uso de cabinas

queue cabinasLibres(1,2,3,4,5,6,7,8,9,10);
chan avisoAccion();
chan UsoDeMaquina(int);
chan liberoMaquina(int, int);
chan ticket[N](txt); 
chan UsaMaquina[N](int); 

process empelado{
int numMaquina, idA, idB; txt comprobante; 
	for i : 1 to N {
		receive avisoAccion(); //Espera a que pase algo
		if (not isEmpty(liberoMaquina)){ 
			receive liberoMaquina(numMaquina, idA); 
			cabinasLibres.push(numMaquina); //Pushea la cabina como libre
			comprobante = Cobrar(idA); //Le cobra
			send ticket[idA](comprobante);//Le envia el comprobante al cliente
			
		} else {
			receive UsoDeMaquina(idA); 
			if (cabinasLibre.isEmpty()){
				//No hay cabinas libres
				//Espero a que alguien quiera pagar
				receive liberoMaquina(numMaquina, idB); 
				cabinasLibres.push(numMaquina); //Pushea la cabina como libre
				comprobante = Cobrar(idA); //Le cobra
				send ticket[idA](comprobante);//Le envia el comprobante al cliente
			}
			//Le asigna una cabina
			cabinasLibres.pop(numMaquina); 
			send usaMaquina[idA](numeroMaquina);
		}
	}

process cliente[id:0..N-1]{
int numeroMaquina; txt comprobante; 
	send UsoDeMaquina(id); //Le mando que quiero usar la maquina
	send avisoAccion(id); 
	receive usaMaquina[id](numeroMaquina); //Espera a que sea su turno
	
	UsarMaquina(numeroMaquina); 
	
	send liberoMaquina(numeroMaquina,id); //Avisa que libero la maquina y quiere pagar
	send avisoAccion(id); 
	receive ticket[id](comprobante);
}
```

### Ejercicio 5 

Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada. 

a) Implemente una solución para el problema descrito. 
b) Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos. 
c) Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución. 
d) Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución. 
e) Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo. 

Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.

```c
//A

//Acá lo que hice fue que los empleados reciban lo que imprimieron. ???? 

chan solicitudImpresion(int, txt); 
chan resultado[N](txt); 

process impresora[id:0..2]{
int idAux; txt Domumento, impresion; 
	while (True){
		receive solicitudImpresion(idAux,documento); 
		impresion = Imprimir(documento);
		send resultado[idAux](documento); 
	}
}


process administrativo[id:0..N-1]{
txt documento, resultado; 
	while (True){
		GenerarDocumento(documento);
		send solicitudImpresion(id,documento); 
		receive resultado[id](resultado); 
		// Analiza el resultado
	}
}
```

```c
//B

//Acá lo que cambia es que se debe crear un nuevo canal para que pueda comunicarse solo con el director y saber cuando es el que manda la solicitud y tenerlo como prioridad. 
//Como recibe de los dos lados debemos tener un canal para el signal así no queda en loop hasta que uno le mande

chan solicitudImpresion(int, txt); 
chan solicitudDirector(txt);
chan resultado[N](txt); 
chan resultadoDirector(txt); 

chan hayPedido(); 

process impresora[id:0..2]{
int idAux; txt Domumento, impresion; 
	while (True){
		receive hayPedido(); 
		if (not isEmpty(solicitudDirector)){
			receive solicitudDirector(documento); 
			impresion = Imprimir(documento);
			send resultadoDirector(documento);
		}else{
			receive solicitudImpresion(idAux,documento); 
			impresion = Imprimir(documento);
			send resultado[idAux](documento); 
		}
	}
}


process administrativo[id:0..N-1]{
txt documento, resultado; 
	while (True){
		GenerarDocumento(documento);
		send solicitudImpresion(id,documento); 
		send hayPedido();
		receive resultado[id](resultado); 
		// Analiza el resultado
	}
}

process director{
txt documento, resultado; 
	while (True){ 
		GenerarDocumento(documento);
		send solicitudDirector(documento); 
		send hayPedido();
		receive resultadoDirector(resultado); 
		// Analiza el resultado
	}
}
```

```c
//C

```

```c
//D
```

```c
//E
```
