# Diagramas de secuencia

## a. Ingreso de nuevo cliente

```mermaid
sequenceDiagram
    title a. Ingreso de nuevo cliente

    actor ca as Cliente
    participant a as ARCA
    participant co as Cliente (Object)

    ca->>+a: get(CUIT)
    a-->>-ca: return Estado Fiscal, nombre || error
    ca->>+co: new(Nombre, Teléfono, CUIT, E-MAIL, Password)
    co-->>-ca: return ID || error
```

## b. Registro de nuevo cliente

```mermaid
sequenceDiagram
    title b. Registro de nuevo cliente

    actor ca as Cliente (Actor)
    participant co as Cliente (Object)

    ca->>+co: register(ID)
    co-->>-ca: return ok || error
```

## c. Acceso de nuevo cliente

```mermaid
sequenceDiagram
    title c. Acceso de nuevo cliente

    actor ca as Cliente (Actor)
    participant co as Cliente (Object)

    ca->>+co: login(ID, Password)
    co-->>-ca: return ok || error
```

## d. Cambio de password de cliente

```mermaid
sequenceDiagram
    title d. Cambio de password de cliente

    actor ca as Cliente (Actor)
    participant co as Cliente (Object)

    ca->>+co: login(ID, Password)
    co-->>-ca: return ok || error

    ca->>+co: get(ID)
    co-->>-ca: return(ID, Nombre, Teléfono, CUIT, E-mail, Password) || Error

    ca->>+co: set(ID, Nombre, Teléfono, CUIT, E-mail, Password)
    co-->>-ca: return ok || error
```

## e. Actualización de datos de cliente

```mermaid
sequenceDiagram
    title e. Actualización de datos de cliente

    actor ca as Cliente (Actor)
    participant c as Cliente
    participant l as Log Auditoria

    ca->>+c: login(ID.Cliente, Password)
    c-->>-ca: return ok || error

    ca->>+c: get(ID)
    c-->>-ca: return(Nombre, Teléfono, CUIT, E-mail, Password)

    ca->>+c: set(ID, Nombre, Teléfono, CUIT, E-mail, Password)
    c->>+l: new(Ticket.ID, Descripción)
    l-->>-c: return ok || error
    c-->>-ca: return ok || error

    ca->>+c: get(Nombre, Teléfono, CUIT, E-MAIL, Password)
    c-->>-ca: return nombre
```

## f. Abrir nuevo ticket

```mermaid
sequenceDiagram
    title f. Abrir nuevo ticket

    actor ca as Cliente (Actor)
    participant co as Cliente (objeto)
    participant t as Ticket
    participant l as Log Auditoria

    ca->>+co: login(ID.Cliente, Password)
    co-->>-ca: return ok || error

    ca->>+t: new(Descripción, Cliente.ID)
    t->>+l: new(Ticket.ID, Descpripción)
    l-->>-t: return ok || error
    t-->>-ca: return(Ticket.ID) || error
```

## g. Actualizar ticket

```mermaid
sequenceDiagram
    title g. Actualizar ticket

    actor a as Analista (Actor)
    participant co as Cliente (objeto)
    participant t as Ticket
    participant l as Log Auditoria

    a->>+co: login(ID.Cliente, Password)
    co-->>-a: return ok || Error

    a->>+t: set(Descripción)
    t->>+l: new(Ticket.ID, Descpripción)
    l-->>-t: return ok || Error
    t-->>-a: return ok || Error
```

## h. Consultar tickets existentes

```mermaid
sequenceDiagram
    title h. Consultar tickets existentes

    actor ca as Cliente (Actor)
    participant co as Cliente
    participant t as Ticket

    ca->>+co: login(ID.Cliente, Password)
    co-->>-ca: return ok || Error

    ca->>+t: getTicketByCliente(Cliente.ID)
    t-->>-ca: return(ID, Descpripción, Estado, Fecha_creación, Fecha_cierre) || Error
```

## i. Borrar ticket

```mermaid
sequenceDiagram
    title i. Borrar ticket

    actor a as Analista (Actor)
    participant co as Cliente (objeto)
    participant at as AnalistaTicket
    participant t as Ticket
    participant l as Log Auditoria

    a->>+co: login(ID.Cliente, Password)
    co-->>-a: return ok || Error

    a->>+at: getAnalistaByTicket(Ticket.ID)
    at-->>-a: return(Analista.ID, Ticket.ID) || Error

    a->>+t: get(ID)
    t-->>-a: return(Descripción, Estado, Fecha_creación, Fecha_cierre, Cliente.ID) || Error

    a->>+t: status(Cerrado)
    t->>+l: new(Ticket.ID, Descripción)
    l-->>-t: return Ok || Error
    t-->>-a: return

    a->>+l: getLogByTicket(ticket.ID)
    l-->>-a: return Descripción
```