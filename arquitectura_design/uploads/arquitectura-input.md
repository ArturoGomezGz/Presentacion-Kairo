# Arquitectura Integrada Kairo — Cloud Native / IA / DevOps

## Propósito del diagrama

Diagrama técnico que muestre en una sola vista las tres dimensiones de la arquitectura de **Kairo**, una herramienta de gestión de proyectos de software desarrollada en el Oracle Java Boot Camp. El diagrama debe ser similar al estilo de diagramas de arquitectura de OCI (Oracle Cloud Infrastructure): fondo blanco o gris claro, iconos rectangulares con etiquetas, flechas con dirección y etiqueta opcional.

---

## Dimensión 1 — DevOps (flujo de CI/CD)

### Actores y origen
- **Developers** (ícono de persona) hacen `git push` a GitHub

### Pipeline de CI (Integración Continua)
- **GitHub Repository** recibe el push
- Se dispara **GitHub Actions CI**
  - Job 1: `backend-test` — ejecuta `mvn test` con JUnit 5 + Mockito (8 suites)
  - Job 2: `frontend-test` — ejecuta `npm test` con React Testing Library
  - Si todos los tests pasan → dispara el CD

### Pipeline de CD (Despliegue Continuo)
- **GitHub Actions CD** (solo en rama `master`, requiere aprobación de entorno `production`)
  - Step 1: `docker build` — imagen multi-stage (Maven build → Eclipse Temurin 17 JRE)
  - Step 2: `docker push` → **OCIR** (Oracle Container Image Registry) con tag = SHA commit
  - Step 3: `kubectl set image` → actualiza el Deployment en **OKE**
  - Step 4: `kubectl rollout status` — espera confirmación (timeout 180s)
- **Rollback manual**: `workflow_dispatch` → `kubectl rollout undo`

---

## Dimensión 2 — Cloud Native (infraestructura OCI)

### Red
- **VCN** (Virtual Cloud Network) — CIDR 10.0.0.0/16
  - Subnet pública → endpoint del cluster OKE + Load Balancer
  - Subnet privada → nodos worker del cluster
  - **NAT Gateway** — permite salida a internet desde subnet privada
  - **Service Gateway** — acceso interno a servicios OCI (Object Storage, etc.)

### Cómputo
- **OKE** (Oracle Container Engine for Kubernetes)
  - Cluster Kubernetes v1.34.2
  - Node Pool: 3 nodos `VM.Standard.E3.Flex` (2 OCPU, 6 GB RAM c/u)
  - **Deployment**: `todolistapp-springboot-deployment`
    - 2 réplicas (pods) del contenedor Spring Boot
    - Topology Spread Constraints: máximo 1 pod por nodo (anti-affinity)
    - Variables de entorno inyectadas desde Kubernetes Secrets
    - Volumen montado: `/mtdrworkshop/creds` (Oracle Wallet)

### Entrada de tráfico
- **OCI Load Balancer** — recibe tráfico HTTP en puerto 80
  - Policy: IP_HASH (sticky sessions por IP)
  - Redirige a pods en puerto 8080
  - externalTrafficPolicy: Local

### Base de datos
- **Oracle Autonomous Database (ADB-ATP)**
  - Free Tier, 1 CPU, 1 TB storage
  - Workload: OLTP (Transaction Processing)
  - Conexión desde pods via **Oracle Wallet** (TLS)
  - JDBC: `oracle:thin:@{PDB}_tp?TNS_ADMIN=/mtdrworkshop/creds`
  - Backups automáticos habilitados
  - Encriptación en reposo

### Seguridad y secretos
- **OCI Vault** — gestión centralizada de secretos
- **Kubernetes Secrets** (dentro de OKE):
  - `db-wallet-secret` → archivos Oracle Wallet (cwallet.sso, tnsnames.ora)
  - `dbuser` → contraseña de base de datos
  - `frontendadmin` → credenciales de UI
- **Oracle Wallet** montado como volumen en cada pod

### Almacenamiento
- **OCI Object Storage** — almacena `deployment_config.tgz` (configuración de despliegue)

### Registry de imágenes
- **OCIR** (Oracle Container Image Registry)
  - Región: `mx-queretaro-1.ocir.io`
  - Imagen: `todolistapp-springboot` con tag SHA commit

---

## Dimensión 3 — IA (Inteligencia Artificial)

### Canal de entrada
- **Usuario final** → escribe mensaje en lenguaje natural en **Telegram**
- Ejemplo: *"muéstrame mis tareas"* o *"cambia la tarea 18 a DONE con 2 horas"*

### Bot de Telegram
- **Telegram Bot** (corriendo dentro del pod Spring Boot en OKE)
- Recibe el mensaje y lo pasa al componente NLU

### Componente NLU — NaturalLanguageRouter
- **NaturalLanguageRouter.java** (Spring Boot service)
- Realiza una llamada HTTP a la **Claude API** (Anthropic)
  - Modelo: `claude-haiku-4-5`
  - Endpoint: `POST https://api.anthropic.com/v1/messages`
  - Input: System prompt fijo (~340 tokens) + mensaje del usuario (~25 tokens)
  - Output: JSON con `{ status, command, params }` (~50 tokens)
  - max_tokens: 150
- Clasifica el mensaje en uno de 7 comandos: `/start`, `/help`, `/my_projects`, `/my_tasks`, `/task`, `/task_status`, `/comment`
- Extrae parámetros (id de tarea, nuevo status, horas trabajadas, texto de comentario)

### Ejecución del comando
- El resultado de Claude es interpretado por el **Bot Handler**
- Se ejecuta la acción correspondiente en la **base de datos** (ADB via JPA/Hibernate)
- Se responde al usuario en Telegram con el resultado

### Flujo de API key
- `ANTHROPIC_API_KEY` almacenada en **OCI Vault** / Kubernetes Secret
- Inyectada como variable de entorno `${anthropic.api.key}` en el pod

---

## Flujo completo de una interacción típica

```
Developer hace push a master en GitHub
  → GitHub Actions CI corre tests (JUnit + React)
  → Si pasan: GitHub Actions CD construye imagen Docker
  → Push imagen a OCIR (mx-queretaro-1.ocir.io)
  → kubectl actualiza Deployment en OKE
  → 2 pods actualizados con rolling update (zero-downtime)

Usuario final (developer del proyecto gestionado) escribe en Telegram:
  "cambia la tarea 25 a DONE con 3 horas"
  → Telegram Bot recibe el mensaje (pod en OKE)
  → NaturalLanguageRouter llama a Claude API (Haiku 4.5)
  → Claude devuelve: { status:"ok", command:"task_status", params:{id:25, status:"DONE", hours:3} }
  → Bot Handler actualiza tarea en Oracle ADB
  → Bot responde al usuario con confirmación en Telegram

Gestor del proyecto abre el navegador web:
  → OCI Load Balancer recibe request HTTP (puerto 80)
  → Redirige al pod Spring Boot (puerto 8080)
  → Spring Boot sirve React SPA + REST API
  → React consulta /api/v1/dashboard → datos reales desde ADB
  → Dashboard muestra KPIs actualizados en tiempo real
```

---

## Estilo visual sugerido para el diagrama

- **Estilo**: Diagrama de arquitectura tipo OCI / AWS — cajas con íconos y etiquetas
- **Fondo**: Blanco o gris muy claro (#F8F9FA)
- **Agrupaciones** (usar rectángulos con borde punteado o sombreado):
  - Grupo **"GitHub / CI-CD"** (color azul claro): GitHub Repo, GitHub Actions CI, GitHub Actions CD
  - Grupo **"OCI"** (color rojo OCI #C74634 o naranja): VCN, OKE, ADB, OCIR, Load Balancer, OCI Vault, Object Storage
  - Grupo **"IA"** (color morado): Claude API (Anthropic), NaturalLanguageRouter
  - Grupo **"Usuarios"** (color neutro): Developer (web browser), Usuario Telegram
- **Flechas**:
  - `git push` → GitHub Repo
  - GitHub Actions CD → OCIR (push imagen)
  - GitHub Actions CD → OKE (kubectl deploy)
  - OCI Load Balancer → OKE pods (tráfico web)
  - OKE pods → ADB (queries JDBC + Oracle Wallet)
  - Telegram → OKE pods (bot webhook)
  - OKE pods → Claude API (HTTP POST, NLU)
  - Claude API → OKE pods (respuesta JSON)
- **Colores de flechas**: Azul para DevOps, Naranja/Rojo para OCI, Morado para IA
