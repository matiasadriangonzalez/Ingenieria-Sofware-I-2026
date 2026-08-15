
# Diagramas de casos de uso — Sistema de Gestión de Biblioteca 

### Lectores (Estudiante y Docente)

```mermaid
graph LR
    Estudiante["Estudiante"]
    Docente["Docente"]
    Lector["Lector"]
    Estudiante -.->|generaliza| Lector
    Docente -.->|generaliza| Lector

    subgraph Sistema1 ["Sistema de Gestión de Biblioteca - Lectores"]
        UC1((Buscar libros))
        UC1_Ext((Búsqueda avanzada))
        UC2((Reservar libros))
        UC3((Consultar préstamos))

        UC1_Ext -. "&lt;&lt;extend&gt;&gt;" .-> UC1
    end

    Lector -->|"busca"| UC1
    Lector -->|"reserva"| UC2
    Lector -->|"consulta"| UC3
```

### Bibliotecario

```mermaid
graph LR
    subgraph Sistema2 ["Sistema de Gestión de Biblioteca - Bibliotecario"]
        UC0((Registrar préstamo))
        UC1((Administrar devoluciones))
        UC2((Calcular multas))
        UC3((Gestionar usuarios))
        UC4((Administrar inventario de libros))

        UC2 -. "&lt;&lt;extend&gt;&gt;" .-> UC1
    end

    Bibliotecario["Bibliotecario"]

    Bibliotecario -->|"registra"| UC0
    Bibliotecario -->|"administra"| UC1
    Bibliotecario -->|"gestiona"| UC3
    Bibliotecario -->|"administra"| UC4
```

### Administrador del Sistema

```mermaid
graph LR
    subgraph Sistema3 ["Sistema de Gestión de Biblioteca - Administrador"]
        UC1((Configurar el sistema))
        UC2((Mantenimiento del sistema))
    end

    Actor["Administrador del Sistema"]

    Actor -->|"configura"| UC1
    Actor -->|"realiza"| UC2
```



## Diagrama de Clases: Sistema de GestiÃ³n de Biblioteca

```mermaid
classDiagram
    %% Superclase Usuario
    class Usuario {
        <<abstract>>
        #id_usuario
        #nombre
        #email
        #password
        #estado_cuenta
        +login()
        +logout()
    }

    %% Subclases de Usuario
    class Lector {
        #max_prestamos_permitidos
        #dias_prestamo
        +buscarLibro(criterio)
        +reservarLibro(libro)
        +consultarPrestamosActivos()
        +consultarHistorial()
    }
    
    class Estudiante {
        -matricula
        -carrera
    }
    
    class Docente {
        -legajo
        -departamento
    }
    
    class Bibliotecario {
        +registrarPrestamo(lector, ejemplar)
        +registrarDevolucion(ejemplar)
        +gestionarAltaLibro(libro)
        +gestionarBajaEjemplar(ejemplar)
        +calcularMulta(prestamo)
    }
    
    class Administrador {
        +gestionarUsuarios()
        +configurarParametrosSistema()
        +realizarMantenimiento()
    }

    %% Relaciones de Herencia
    Usuario <|-- Lector
    Usuario <|-- Bibliotecario
    Usuario <|-- Administrador
    Lector <|-- Estudiante
    Lector <|-- Docente

    %% Clases del Dominio de Biblioteca
    class Catalogo {
        +buscarPorTitulo(titulo)
        +buscarPorAutor(autor)
        +buscarPorISBN(isbn)
    }

    class Libro {
        -ISBN
        -titulo
        -autor
        -editorial
        -anio_publicacion
        +obtenerEjemplaresDisponibles()
    }

    class Ejemplar {
        -id_ejemplar
        -ubicacion_fisica
        -estado_conservacion
        -estado_disponibilidad
        +cambiarEstado(nuevo_estado)
    }

    class Prestamo {
        -id_prestamo
        -fecha_prestamo
        -fecha_vencimiento
        -fecha_devolucion
        -estado
        +calcularDiasAtraso()
        +finalizarPrestamo()
    }

    class Reserva {
        -id_reserva
        -fecha_reserva
        -estado
        +completarReserva()
        +cancelarReserva()
    }

    class Multa {
        -id_multa
        -monto
        -fecha_emision
        -estado_pago
        +calcularMonto(dias_atraso)
        +registrarPago()
    }

    %% Relaciones / Asociaciones
    Catalogo "1" *-- "0..*" Libro : contiene
    Libro "1" *-- "1..*" Ejemplar : posee
    
    Lector "1" -- "0..*" Prestamo : solicita
    Ejemplar "1" -- "0..*" Prestamo : es prestado en
    
    Lector "1" -- "0..*" Reserva : realiza
    Libro "1" -- "0..*" Reserva : tiene reservas de
    
    Prestamo "1" -- "0..1" Multa : puede generar
    Lector "1" -- "0..*" Multa : acumula
```


# Diagrama de secuencia — Reserva de un libro por parte de un estudiante 

```mermaid
sequenceDiagram
    title Reserva de un libro por parte de un estudiante

    actor es as Estudiante
    participant cat as Catalogo
    participant li as Libro
    participant re as Reserva

    es->>+cat: buscarPorTitulo(criterio)
    cat-->>-es: return listaLibros

    es->>+li: obtenerEjemplaresDisponibles()
    li-->>-es: return cantidadDisponibles

    alt cantidadDisponibles > 0
        es->>+re: new(Libro.ID, Lector.ID)
        re-->>-es: return Reserva.ID, fechaReserva
    else cantidadDisponibles = 0
        cat-->>es: return mensaje("sin ejemplares disponibles para reservar")
    end
```


# Diagrama de secuencia — Devolución de un libro con cálculo de multa 

```mermaid
sequenceDiagram
    title Devolución de un libro con cálculo de multa

    actor b as Bibliotecario
    participant p as Prestamo
    participant m as Multa
    participant e as Ejemplar

    b->>+p: registrarDevolucion(fechaDevolucionReal)
    p->>p: calcularDiasAtraso()

    alt diasAtraso > 0
        p->>+m: new(Prestamo.ID)
        m->>m: calcularMonto(diasAtraso)
        m-->>-p: return Multa.ID, monto
    end

    p->>+e: cambiarEstado(Disponible)
    e-->>-p: return ok
    p->>p: finalizarPrestamo()
    p-->>-b: return ok, monto Multa || error
```

# Diagrama de actividad — Préstamo de un libro

```mermaid
flowchart TD
    Inicio((Inicio))
    A[Estudiante/Docente solicita préstamo]
    B[Bibliotecario busca el libro]
    C{¿Ejemplar disponible?}
    D[Ofrecer reserva]
    E[Consultar multas pendientes del Lector]
    F{¿Usuario habilitado?<br/>sin multas pendientes}
    G[Rechazar préstamo]
    H[Registrar préstamo]
    I[Ejemplar.cambiarEstado a Prestado]
    J[Entregar libro al usuario]
    Fin((Fin))

    Inicio --> A --> B --> C
    C -- No --> D --> Fin
    C -- Sí --> E --> F
    F -- No --> G --> Fin
    F -- Sí --> H --> I --> J --> Fin
```


## Diagrama de Estados: Ciclo de vida de un Libro (Ejemplar)
```mermaid
stateDiagram-v2
    [*] --> Disponible : Alta de Ejemplar
    
    Disponible --> Reservado : reservarLibro()
    Reservado --> Disponible : cancelarReserva() / Vencimiento
    
    Reservado --> Prestado : registrarPrestamo()
    Disponible --> Prestado : registrarPrestamo()
    
    Prestado --> Disponible : registrarDevolucion() [Buen estado]
    Prestado --> Mantenimiento : registrarDevolucion() [Dañado]
    Prestado --> Extraviado : Reportado como perdido
    
    Mantenimiento --> Disponible : Reparación exitosa
    Mantenimiento --> Baja : Daño irreparable (gestionarBajaEjemplar)
    
    Extraviado --> Baja : Confirmación de pérdida (gestionarBajaEjemplar)
    Disponible --> Baja : Retiro por antigüedad (gestionarBajaEjemplar)
    
    Baja --> [*]
```

