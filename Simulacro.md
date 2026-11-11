# Simulacro-parcial-OBJ
Simulacro para presentar 

EJERCICIO 1:

a. Falso. El método en realidad no falla, sino que devuelve un texto ("Error: no hay masa salarial"). Como devuelve un string en vez de tirar una excepción (un throw), el programa sigue de largo y no corta la ejecución donde debería, dejando todo inconsistente  

b. Falso. No hay nada de polimorfismo. En vez de usar clases distintas, metieron una variable tipo y llenaron todo de ifs (como if (tipo == "uber")). Para que haya polimorfismo en serio, hay que hacer que cada sector sea una subclase distinta y que todas entiendan los mismos mensajes   

c. Verdadero. Conceptualmente, decidir quién va a la lucha es una regla que le pertenece al sindicato. Ojo, como la fábrica es la dueña de la colección de trabajadores, la organización le tiene que mandar un mensaje (por ejemplo proceso.trabajadoresEnLucha()) para que los filtre, y así no romper el encapsulamiento

d. Falso. Si querés agregar otra medida de fuerza, como todo está hardcodeado con if / else if evaluando un string, te obliga a modificar el código viejo agregando más condiciones. Esto rompe el principio Abierto/Cerrado. Hay que usar polimorfismo y hacer que las medidas sean objetos separados

e. Falso. Pasa lo mismo que en el punto A, cuando no hay adherentes el método de la organización simplemente devuelve un string "No hay adherentes para la medida" Al no lanzar un error de verdad, la Sociedad no se entera del problema y su forEach sigue recorriendo el resto de las fábricas como si nada pasara.  

EJERCICIO 2:

CODE: 

class Trabajador {
  var property salario
  var property horasSemanales
  var property valorNuevoPorHora
  var property estaSindicalizado

  method valorProducido() = horasSemanales * valorNuevoPorHora

  method desafiliar() { 
    estaSindicalizado = false 
  }

  method reducirHoras(cant) {
    if (horasSemanales < cant) {
      horasSemanales = 0
    } else {
      horasSemanales = horasSemanales - cant
    }
  }

  method aumentarSalario(porc) {
    salario = salario + (salario * (porc / 100))
  }
}

class ProcesoProductivo {
  const trabajadores = []

  method agregarTrabajador(t) { 
    trabajadores.add(t) 
  }

  method masaSalarial() = trabajadores.sum({ t => t.salario() })

  method valorNuevoTotal() = trabajadores.sum({ t => t.valorProducido() })

  method plusvalia() = self.valorNuevoTotal() - self.masaSalarial()

  method tasaExplotacion() {
    if (self.masaSalarial() == 0) {
      throw new DomainException(message = "Sin masa salarial no es posible calcular la tasa de explotación.")
    }
    return self.plusvalia() / self.masaSalarial()
  }

  method salarioPromedio() {
    if (trabajadores.isEmpty()) {
      return 0
    } else {
      return self.masaSalarial() / trabajadores.size()
    }
  }

  method sindicalizados() = trabajadores.filter({ t => t.estaSindicalizado() })

  method vanguardia() {
    const promedio = self.salarioPromedio()
    return trabajadores.filter({ t => t.salario() < promedio })
  }

  method umbralConflicto() = 1.5

  method estaAlBordeDeUnConflicto() {
    if (trabajadores.isEmpty()) {
      return false
    }
    
    const mitad = trabajadores.size() / 2
    const hayMasaCritica = self.sindicalizados().size() > mitad or self.vanguardia().size() > mitad
    
    return self.tasaExplotacion() > self.umbralConflicto() and hayMasaCritica
  }

  method trabajadoresEnLucha() {
    const losSindicalizados = self.sindicalizados()
    
    if (losSindicalizados.isEmpty()) {
      const laVanguardia = self.vanguardia()
      
      if (laVanguardia.isEmpty()) {
        throw new DomainException(message = "No hay masa laboral para sostener la medida.")
      } else {
        return laVanguardia
      }
    } else {
      return losSindicalizados
    }
  }
}

class Uberizacion inherits ProcesoProductivo {
  const comision = 0.20

  override method agregarTrabajador(t) {
    t.desafiliar()
    super(t)
  }

  override method valorNuevoTotal() = super() * (1 + comision)
}

class SectorBancario inherits ProcesoProductivo {
  const bonoConvenio = 500

  override method masaSalarial() = 
    super() + (self.sindicalizados().size() * bonoConvenio)

  override method umbralConflicto() = 1.0
}

class OrganizacionObrera {
  var estrategia

  method aplicarMedida(proceso) {
    estrategia.ejecutar(proceso)
  }
}

class Huelga {
  var horasParo

  method ejecutar(proceso) {
    const adherentes = proceso.trabajadoresEnLucha()
    adherentes.forEach({ t => t.reducirHoras(horasParo) })
  }
}

class NegociacionSalarial {
  var porcentajeAumento

  method ejecutar(proceso) {
    const adherentes = proceso.trabajadoresEnLucha()
    adherentes.forEach({ t => t.aumentarSalario(porcentajeAumento) })
  }
}

class Sociedad {
  const procesos = []

  method agregarProceso(p) { 
    procesos.add(p) 
  }

  method jornadaDeLucha(organizacion) {
    procesos.forEach({ p => organizacion.aplicarMedida(p) })
  }
}


-DIAGRAMA:
<img width="884" height="667" alt="image" src="https://github.com/user-attachments/assets/357fa550-100c-4b8c-9244-e5977983d07f" />


EJERCICIO 3:

Simplemente hay que crear una nueva clase CallCenter que herede de ProcesoProductivo. Adentro de esa clase, le hacemos un override al método plusvalia() para restarle al total el costo de supervisión multiplicado por el tamaño de la vanguardia

agregamos:

Herencia: Nos salva de repetir código,  Heredamos gratis toda la lógica que ya estaba en ProcesoProductivo (el manejo de listas, cómo sacar la masa salarial, etc.) y solo nos preocupamos por escribir lo que cambia específicamente para el Call Center

Polimorfismo: Como el CallCenter entiende los mismos mensajes que los demás sectores, lo podemos mandar directamente a la colección de la Sociedad y el sistema lo usa sin enterarse de que es un sector nuevo. Así no tenemos que ir a modificar código viejo ni agregar ifs, cumpliendo a pleno con el Principio Abierto/Cerrado

EJERCICIO 4:

describe "Pruebas del parcial - Class Struggle" {
  
  test "Punto A - La tasa de explotacion falla tirando excepcion si la masa salarial es cero" {
    // Armo un proceso sin trabajadores para forzar que la masa salarial de 0
    const fabricaVacia = new ProcesoProductivo()
    
    assert.throwsExceptionWithMessage(
      "Sin masa salarial no es posible calcular la tasa de explotación.", 
      { fabricaVacia.tasaExplotacion() }
    )
  }

  test "Uberizacion - Al agregar un trabajador, lo desafilia automaticamente del sindicato" {
    const plataformaUber = new Uberizacion()
    const operario = new Trabajador(
      salario = 1500, 
      horasSemanales = 40, 
      valorNuevoPorHora = 120, 
      estaSindicalizado = true
    )
    
    plataformaUber.agregarTrabajador(operario)
    
    // Verifico que el booleano haya cambiado a false
    assert.notThat(operario.estaSindicalizado())
  }
  
  test "Punto E - Aplicar una medida corta la ejecucion si no hay adherentes" {
    const fabricaSinGente = new ProcesoProductivo()
    const paroDeOchoHoras = new Huelga(horasParo = 8)
    const elSindicato = new OrganizacionObrera(estrategia = paroDeOchoHoras)
    
    assert.throwsExceptionWithMessage(
      "No hay masa laboral para sostener la medida.", 
      { elSindicato.aplicarMedida(fabricaSinGente) }
    )
  }
}
