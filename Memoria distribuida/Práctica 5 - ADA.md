 
### Ejercicio 1

Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 

a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad. 
b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.

Respuesta : 
```c
//A

Procedure Puente is
	TASK Acceso IS
		ENTRY AccesoAuto;
		ENTRY AccesoCamioneta;
		ENTRY AccesoCamion;
		ENTRY Salir (peso: IN integer);
	End Acceso;

	TASK TYPE Auto;
	vecAutos: array (1..A) of Auto;
	TASK BODY Auto IS
	BEGIN
		Acceso.AccesoAuto;
		Acceso.Salir(1);
	END Auto;
	
	TASK TYPE Camioneta;
	vecCamionetas: array (1..B) of Camioneta;
	TASK BODY Camioneta IS
	BEGIN
		Acceso.AccesoCamioneta;
		Acceso.Salir(2);
	END Camioneta;

	TASK TYPE Camion;
	vecCamiones: array (1..C) of Camion;
	TASK BODY Camion IS
	BEGIN
		Acceso.AccesoCamion;
		Acceso.Salir(3);
	END Camion;

	TASK BODY Acceso IS
		peso: integer := 0;
	BEGIN
		LOOP
			SELECT 
				WHEN (peso < 5) => 
					ACCEPT AccesoAuto DO
						peso:= peso + 1;
					END AccesoAuto;
			OR
				WHEN (peso < 4) => 
					ACCEPT AccesoCamioneta DO
						peso:= peso + 2;
					END AccesoCamioneta;
			OR
				WHEN (peso < 3)  => 
					ACCEPT AccesoCamion DO
						peso:= peso + 3;
					END AccesoCamion;
			OR
                ACCEPT Salir (pesoSalida: IN integer) DO
                    peso:= peso - pesoSalida;
                END Salir;
            END SELECT;	
		END LOOP;
	END Acceso;
Begin
	null;
End Puente;

```

```c
//B

Procedure Puente is
	TASK Acceso IS
		ENTRY AccesoAuto;
		ENTRY AccesoCamioneta;
		ENTRY AccesoCamion;
		ENTRY Salir (peso: IN integer);
	End Acceso;

	TASK TYPE Auto;
	vecAutos: array (1..A) of Auto;
	TASK BODY Auto IS
	BEGIN
		Acceso.AccesoAuto;
		Acceso.Salir(1);
	END Auto;
	
	TASK TYPE Camioneta;
	vecCamionetas: array (1..B) of Camioneta;
	TASK BODY Camioneta IS
	BEGIN
		Acceso.AccesoCamioneta;
		Acceso.Salir(2);
	END Camioneta;

	TASK TYPE Camion;
	vecCamiones: array (1..C) of Camion;
	TASK BODY Camion IS
	BEGIN
		Acceso.AccesoCamion;
		Acceso.Salir(3);
	END Camion;

	TASK BODY Acceso IS
		peso: integer := 0;
	BEGIN
		LOOP
			SELECT 
				WHEN (AccesoCamion´count = 0) and (peso < 5) => 
					ACCEPT AccesoAuto DO
						peso:= peso + 1;
					END AccesoAuto;
			OR
				WHEN (AccesoCamion´count = 0) and (peso < 4)  => 
					ACCEPT AccesoCamioneta DO
						peso:= peso + 2;
					END AccesoCamioneta;
			OR
				WHEN (peso < 3)  => 
					ACCEPT AccesoCamion DO
						peso:= peso + 3;
					END AccesoCamion;
			OR
                ACCEPT Salir (pesoSalida: IN integer) DO
                    peso:= peso - pesoSalida;
                END Salir;
            END SELECT;	
		END LOOP;
	END Acceso;
Begin
	null;
End Puente;
```

### Ejercicio 2

Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada. 

a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atendidos. 
b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago. 
c) Implemente una solución donde los clientes se retiran si no son atendidos inmediatamente. 
d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más y se retiran si no son atendidos inmediatamente.

Respuesta : 

```c
//A

Procedure Banco is 
	
	TASK empleado is
		ENTRY realizarPago(comprobante : OUT txt);
	END empleado; 
	
	TASK Type cliente; 
	vecClientes: array (1..N) of Cliente;
		comprobante : txt; 
	TASK BODY cliente IS 
	BEGIN
		empleado.realizarPago(comprobante); 
		retirarse(); 
	END cliente; 
	
	TASK BODY empleado IS
	BEGIN
		LOOP
			ACCEPT realizarPago (comprobante : out txt) do
				comprobante = realizarCobro(); 
			END realizarPago; 
		END LOOP; 
	END empleado; 

Begin
	null; 
end; 
```

```c
//B 


Procedure Banco is 
	
	TASK empleado is
		ENTRY realizarPago(comprobante : OUT txt);
	END empleado; 
	
	TASK Type cliente; 
	vecClientes: array (1..N) of Cliente;
		comprobante : txt; 
	TASK BODY cliente IS 
	BEGIN
		SELECT		
            empleado.realizarPago(comprobante); 
        OR DELAY 600.0
            NULL;
        END SELECT;
		retirarse(); 
	END cliente; 
	
	TASK BODY empleado IS
	BEGIN
		LOOP
			ACCEPT realizarPago (comprobante : out txt) do
				comprobante = realizarCobro(); 
			END realizarPago; 
		END LOOP; 
	END empleado; 

Begin
	null; 
end; 

```

```c
//C

Procedure Banco is 
	
	TASK empleado is
		ENTRY realizarPago(comprobante : OUT txt);
	END empleado; 
	
	TASK Type cliente; 
	vecClientes: array (1..N) of Cliente;
		comprobante : txt; 
	TASK BODY cliente IS 
	BEGIN
		SELECT		
            empleado.realizarPago(comprobante); 
        END SELECT;
	END cliente; 
	
	TASK BODY empleado IS
	BEGIN
		LOOP
			ACCEPT realizarPago (comprobante : out txt) do
				comprobante = realizarCobro(); 
			END realizarPago; 
		END LOOP; 
	END empleado; 

Begin
	null; 
end; 

```

```c
// D

Procedure Banco is 
	
	TASK empleado is
		ENTRY realizarPago(comprobante : OUT txt);
	END empleado; 
	
	
	TASK Type cliente; 
		comprobante : txt; 
	TASK BODY cliente IS 
	BEGIN
		SELECT		
            empleado.realizarPago(comprobante); 
            retirarse();
        OR  DELAY 600.0
            SELECT 
	            empleado.realizarPago(comprobante); 
	        OR 
		        retirarse(); 
            END SELECT; 
        END SELECT; 
	END cliente; 
	
	vecClientes: array (1..N) of Cliente;
	
	TASK BODY empleado IS
	BEGIN
		LOOP
			ACCEPT realizarPago (comprobante : out txt) do
				comprobante = realizarCobro(); 
			END realizarPago; 
		END LOOP; 
	END empleado; 

Begin
	null; 
end; 
```

### Ejercicio 3 

Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se comunican continuamente. Se requiere modelar su funcionamiento considerando las siguientes condiciones: 
- La central siempre comienza su ejecución tomando una señal del proceso 1; luego toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos. 
- Los procesos periféricos envían señales continuamente a la central. La señal del proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y vuelve a mandarla (no se deshecha).

Respuesta : 

> Falta modelar lo de los 3 minutos. 
```c
procedure sistema IS

	TASK central; 
	BEGIN
	 ENTRY
	END; 
	
	TASK central;  
	BEGIN
		ENTRY recibirSeñal1(señal : in txt);
		ENTRY recibirSeñal2(señal : in txt);
	END; 
	
	TASK periferico1; 
	TASK BODY periferico1 is;
		señal : txt; 
	BEGIN
		LOOP
			señal = GenerarSeñal(); 
			SELECT
				central.recibirSeñal1(señal);
			OR
				DELAY 120.0
				NULL; 
			END SELECT; 
		END LOOP; 
	END; 
	
	TASK periferico2; 
	TASK BODY periferico2 is;
		señal : txt; 
	BEGIN
		LOOP
			señal = GenerarSeñal(); 
			SELECT
				central.recibirSeñal2(señal);
			OR
				DELAY 60.0
				central.recibirSeñal2(señal);
			END SELECT; 
		END LOOP; 
	END; 
	
	TASK BODY central is; 
	BEGIN
		ACCEPT recibirSeñal1(señal : in txt)
				procesarDatos(señal);
		END recibirSeñal;
		LOOP
			SELECT
				ACCEPT recibirSeñal1(señal: in txt) DO
					procesarDatos(señal);
				END recibirSeñal1; 
			OR
				ACCEPT recibirSeñal2(señal: in txt) DO
					procesarDatos(señal);
				END recibirSeñal2; 
				
			END SELECT;
		END LOOP; 
	END; 

BEGIN
 NULL; 
END; 

```

### Ejercicio 4

En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos. Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras.

Respuesta : 

```c
procedure clinica IS

TASK medico;
BEGIN
	ENTRY solicitudAtencionCliente(pedido: IN TXT);
	ENTRY solicitudAtencionEnfermera(pedido: IN TXT);
END;  

TASK buzon_consultorio
BEGIN
	ENTRY dejarNotaEnfermera(nota : IN TXT)
	ENTRY revisarNotas(nota : IN OUT TXT)
END; 

TASK TYPE enfermera;
TASK BODY enfermera IS;
	nota : TXT; 
	pedido : TXT; 
BEGIN
	LOOP 
		peticion = generarPeticion();
		SELECT 
			medico.solicitudAtencionEnfermera(peticion); 
		ELSE
			nota = generarNota(); 
			buzon_consultorio.dejarNotaEnfermera(nota); 
		END SELECT; 
	END LOOP;
END;

vectorEnfermeras : array (1..E) of enfermera; 

TASK TYPE persona;
TASK BODY persona IS;  
	pedido : txt; 
	Cantidad : int;
	continuar : boolean;
begin
	cantidad = 0;
	pedido = GenerarPedido(); 
	continuar = True; 
	while (cantidad < 3) and (continuar) LOOP
		SELECT
			medico.solicitudAtencionCliente(pedido);
			continuar = False; 
		OR 
			DELAY 300.0
			cantidad++;
			IF (cantidad < 3) THEN
			BEGIN
				DELAY 600.0
			END IF;
		END SELECT; 
	END LOOP;
end; 

vectorPersonas : array (1..P) of persona;

TASK BODY medico IS;
	nota : txt; 
BEGIN
	LOOP
		SELECT
			ACCEPT solicitudAtencionCliente(pedido: IN TXT) DO
				procesarPedido();
			END solicitudAtencionCliente;
		OR 
			WHEN (solicitudAtencionCliente´count = 0) => 
				ACCEPT solicitudAtencionEnfermera(pedido: IN TXT) DO
					procesarPedido();
				END solicitudAtencionEnfermera;
		ELSE
			SELECT
				buzon_consultorio.revisarNotas(nota)
				ProcesarNota(nota);
			ELSE
				null;
			END SELECT;
			
		END SELECT; 
	END LOOP; 
END; 

TASK BODY buzon_consultorio IS
	notas : cola;
BEGIN
	LOOP
		SELECT 
			ACCEPT dejarNotaEnfermera(nota : IN TXT) DO
				cola.push(nota); 
			END dejarNotaEnfermera;
		OR 
			WHEN (notIsEmpty(cola)) ; =>
				SELECT
					ACCEPT revisarNotas(nota : IN OUT TXT) DO
						nota = notas.pop();
					END revisarNotas; 
				OR 
					null; 
				END SELECT; 
	END LOOP; 
END; 


BEGIN
	NULL;
END; 
```

### Ejercicio 5 

En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos. Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento).

Respuesta : 

```c
procedure carreras IS

TASK servidor; 
BEGIN
	ENTRY enviarDocumento(documento : IN TXT,noEstaListo : OUT BOOLEAN);
END; 
TASK TYPE usuario;

TASK BODY usuario IS
	vectorUsuarios : array (1..U) of usuario;
	documento : txt; 
	noEstaListo : boolean;
	resultado : booelan;
BEGIN
	noEstaListo = True;  
	EscribirDocumento(documento); 
	while (noEstaListo) LOOP
		SELECT
			servidor.enviarDocumento(documento,noEstaListo); 
			if (noEstaListo) THEN
				EscribirDocumento(documento); 
			END IF; 
		OR
			DELAY 120.0
			DELAY 60.0
		END SELECT; 
	END LOOP;
END; 

TASK BODY servidor IS;
BEGIN
	LOOP
		ACCEPT enviarDocumento(documento : IN TXT,noEstaListo : OUT BOOLEAN) DO
			noEstaListo = ProcesarDocumentoVerErrores(documento); 
		END enviarDocumento;
	END LOOP; 
END; 


BEGIN
	null;
END carreras; 
```


### Ejercicio 6 

En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto. Nota: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.

Respuesta : 

```c
Procedure Playa IS 
	TASK Administrador IS
		ENTRY CalcularGanador (idEquipo: IN integer; monto: IN integer);
		ENTRY IdentificarGanador (ganador: OUT integer);
	END Administrador;

	TASK TYPE Equipo IS 
		Entry RecibirID (id: IN integer);
		ENTRY Llegada;
		ENTRY Empezar;
		ENTRY SumarMonedas (monto: IN integer);
	END Equipo;
	vecEquipos = array(1..5) of Equipo;
	TASK BODY Equipo IS
		montoTotal: integer:= 0;
		idEquipo: integer;
	BEGIN
		ACCEPT RecibirID (id: IN integer) DO
			idEquipo:= id;
		END RecibirID;
		FOR i IN 1..4 LOOP
			ACCEPT Llegada;
		END LOOP;
		FOR i IN 1..4 LOOP
			ACCEPT Empezar;
		END LOOP;
		FOR i IN 1..4 LOOP
			ACCEPT SumarMonedas(monto: IN integer) DO
				montoTotal:= montoTotal + monto;
			END SumarMonedas;
		END LOOP;
		Administrador.CalcularGanador(idEquipo, montoTotal);
	END Equipo;

	TASK TYPE Persona;
	vecPersonas = array(1..20) of Persona;
	TASK BODY Persona IS
		equipo: integer:= ObtenerNumEquipo();
		monto: integer:= 0;
		ganador: integer;
	BEGIN
		Equipo(equipo).Llegada;
		Equipo(equipo).Empezar;
		FOR i 1..15 LOOP
			monto:= monto + Moneda();
		END LOOP;
		Equipo(equipo).SumarMonedas(monto);
		Administrador.IdentificarGanador(ganador);
	END Persona;

	TASK BODY Administrador IS
		max: integer:= -1;
		maxEquipo: integer;
	BEGIN
		FOR i IN 1..5 LOOP
			ACCEPT CalcularGanador (idEquipo: IN integer; monto: IN integer) DO
				IF (monto > max) THEN
					max:= monto;
					maxEquipo:= idEquipo;
				END IF;
			END CalcularGanador;
		END LOOP;
		FOR i IN 1..20 LOOP
			ACCEPT IdentificarGanador (ganador: OUT integer) DO
				ganador:= maxEquipo;
			END IdentificarGanador;
		END LOOP;
	END Administrador;
Begin
	FOR i IN 1..5 LOOP 
		vecEquipos(i).RecibirID(i);
	END LOOP;
End Playa;
```


### Ejercicio 7 

Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de 100 mil números). Para ello, existe un Coordinador que determina el momento en que se debe realizar el cálculo de este promedio y que, además, se queda con el resultado. Nota: maximizar la concurrencia; este cálculo se hace una sola vez.

Respuesta  : 

```c

procedure vector IS

TASK TYPE worker;

TASK coordinador;
BEGIN
	ENTRY resultadoDelWorker(suma : IN INT);
END;

TASK BODY worker IS;
	vectorNumeros : array (1..100000) of INT; 
	i : INT; 
	res : int; 
BEGIN
	res = 0;
	inicializarVector(vectorNumeros);
	for i IN 1..100000 LOOP
		res += vectorNumeros[i]; 
	END LOOP; 
	coordinador.resultadoDelWorker(res);
END worker;

TASK BODY coordinador IS;
	i: int;
	res : INT; 
	promedio : REAL; 
BEGIN
	res = 0; 
	for i IN 1..10 LOOP
		ACCEPT resultadoDelWorker(suma : IN INT) DO
			res += suma; 
		END  resultadoDelWorker; 
	END LOOP;
	promedio := (res/1000000);
END coordinador;

vectorWorkers : array (1..10) of worker;
BEGIN
	null;
END; 

```

### Ejercicio 8 

Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria

```c
procedure huellas IS
	
TASK TYPE servidor; 

TASK especialista;
BEGIN
	ENTRY huellaMasSimilar(similitud : IN INT, codigo : IN INT)
	ENTRY peticionHuella(huella : OUT TXT)
	ENTRY fin
END especialista;  

TASK BODY especialista IS; 
	test : txt; 
	similitud: int; 
	codigo : int; 
	codMayor : int; 
	simiMayor : int; 
BEGIN
	LOOP
		codMayor := -1;
		simiMayor := -1; 
		tomarImagenHuella(test);
		for i IN 1..16 LOOP
			SELECT
				ACCEPT peticionHuella(huella : OUT TXT) DO
					huella := test; 
				END peticionHuella;
			OR
				ACCEPT huellaMasSimilar(similitud : IN INT, codigo : IN INT) DO
					if (simiMayor < similitud) THEN
						codMayor:= codigo;
						simiMayor:= similitud; 
					END IF; 
				END huellaMasSimilar;			
			END SELECT; 
		END;
		FOR i IN 1..8 LOOP
				ACCEPT Fin;
		END LOOP; 
	END LOOP; 
END especialista; 

TASK BODY servidor IS
	codigo : int; 
	valor : int; 
	test : txt; 
BEGIN
	LOOP
		especialista.peticionHuella(test);
		Buscar(test, código, valor);
		especialista.huellaMasSimilar(valor,codigo);
		especialista.fin 
	END LOOP; 
END servidor; 
	
vectorServidores : array (1..8) of servidores; 
BEGIN
	NUll; 
END huellas; 
```

### Ejercicio 9 

Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3 camiones. Hay P personas que hacen reclamos continuamente hasta que uno de los camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a que llegue un camión; si no pasa, vuelve a hacer el reclamo y a esperar a lo sumo 15 minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte los residuos. Sólo cuando un camión llega, es cuando deja de hacer reclamos y se retira. Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos ha hecho sin ser atendido. Nota: maximizar la concurrencia.

Respuesta  : 

```c
procedure Limpieza IS

TASK TYPE camion; 
TASK TYPE persona; 
BEGIN
	ENTRY tuId(ID : IN INT); 
	ACCEPT reclamoTomado; 
END persona; 

TASK empresa; 
BEGIN
	ENTRY HacerReclamo(miId : IN INT); 
	ENTRY solicitarTrabajo(id : OUT INT);
END empresa; 

TASK BODY empresa IS; 
	vecContador : array (1..P) of INT; 
	codMax : int; 
	valorMax : int; 
	reclamos : int; 
BEGIN
	reclamos := 0; 
	for i in 1..P do
		vectorContador[i] := 0;
	END FOR; 
	LOOP
		SELECT 
			ACCEPT HacerReclamo(miId : IN INT) DO;
				IF (vectorContador[miId] = 0) THEN
					reclamos++;
				END IF; 
				vectorContador[miId] += 1;
			END HacerReclamo; 
		OR
			WHEN(reclamos > 0) =>
				ACCEPT solicitarTrabajo(ID : OUT INT); DO
					codMax : -1; 
					valorMax : -1;
					for i IN 1..P LOOP
						IF (vectorContador[i] > valorMax) THEN
							codMax := i; 
							valorMax := vectorContador[i]; 
						END IF; 
					END LOOP;
				vectorContador[codMax]:= -1; 
				reclamos--;
				ID :=  codMax; 
				END solicitarTrabajo; 
		END SELECT; 
	END LOOP;
END; 

TASK BODY camion IS; 
	id: int;
BEGIN
	LOOP
		empresa.solicitarTrabajo(ID);
		vectorPersonas[id].reclamoTomado;
	END LOOP; 
END; 

TASK BODY persona IS; 
	miId : int; 
	seguir : boolean; 
BEGIN
	seguir := True; 
	ACCEPT tuId(ID : IN INT) DO ; 
		miId := ID; 
	END tuId; 
	WHILE (seguir) LOOP 
		empresa.HacerReclamo(miId);
		SELECT
			ACCEPT reclamoTomado DO
				seguir:= False; 
			END reclamoTomado; 
		OR
			DELAY 900.0
		END SELECT; 
	END LOOP; 
END persona; 


BEGIN
	for i in 1..P THEN
		persona.tuId(i);
	END FOR; 
END limpieza; 


```