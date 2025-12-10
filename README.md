# JobMatcher AI - Sistema de Recomendación de Empleo Inteligente

Bienvenido a JobMatcher AI, una plataforma avanzada que utiliza Inteligencia Artificial Generativa (RAG) para conectar candidatos con ofertas de empleo ideales. A diferencia de los buscadores tradicionales basados en palabras clave, JobMatcher "entiende" tu perfil y te recomienda ofertas basándose en afinidad semántica, cultura y requisitos técnicos.

## 🏗 Arquitectura del Sistema

El sistema sigue una arquitectura de microservicios dividida en tres capas principales:

1.  **Capa de Datos (ELK Stack)**:
    *   **Elasticsearch**: Motor de búsqueda vectorial y almacenamiento de ofertas.
    *   **Kibana**: Visualización de datos y gestión de índices.
    *   **Logstash/Metricbeat**: Ingesta de logs y monitorización (opcional).
2.  **Backend (Python/FastAPI)**:
    *   API REST que gestiona la lógica de negocio.
    *   Motor RAG con **Grok-4 Fast (xAI)** como LLM.
    *   Generación de embeddings con `sentence-transformers`.
3.  **Frontend (React)**:
    *   Interfaz de usuario moderna y responsiva.
    *   Chatbot integrado con historial y persistencia de sesión.

---

## 🛠 Guía de Despliegue (Deployment)

El proyecto está organizado en **dos repositorios distintos** para desacoplar la infraestructura del código de aplicación:

1.  **`repo-devops`**: Contiene la definición de infraestructura (ELK Stack) y configuraciones.
2.  **`repo-backend`**: Contiene el código fuente de la API, el Chatbot y el Frontend.

### 1. Despliegue de Infraestructura

Este repositorio gestiona la base de datos y herramientas de monitorización. Se asume la siguiente estructura:
```
/g3r12025devops
├── elasticsearch
│   └── config
│       └── elasticsearch.yml  <-- Configuración personalizada
├── kibana
│   └── config
│       └── kibana.yml
└── docker-compose.yml (o scripts de despliegue)
```

#### Paso 1.1: Crear Red Compartida
Es crucial crear una red externa para que los contenedores de ambos repositorios se comuniquen.
```bash
docker network create jobmatcher-network
```

#### Paso 1.2: Levantar ELK Stack
Desde la raíz de `repo-devops`, levanta los servicios. Si usas scripts individuales o `docker run`, asegúrate de montar los volúmenes de configuración correctamente.

**Ejemplo para Elasticsearch:**
```bash
docker run -d \
  --name elasticsearch \
  --net jobmatcher-network \
  -p 9200:9200 \
  -v $(pwd)/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v es_data:/usr/share/elasticsearch/data \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
  -e "xpack.security.enabled=true" \
  -e "ELASTIC_PASSWORD=changeme" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0
```

---

### 2. Despliegue de Aplicación (Desde `g3r12025backend`)

Este repositorio contiene la lógica de negocio.

#### Paso 2.1: Configuración
Crea un archivo `.env` en la raíz del backend basándote en el ejemplo. Asegúrate de que `ES_HOST` apunte al nombre del contenedor definido en el paso anterior (e.g., `http://elasticsearch:9200`).

```env
ES_HOST=http://elasticsearch:9200
OPENROUTER_API_KEY=...
```

#### Paso 2.2: Construcción y Ejecución del Backend
Desde `g3r12025backend`:

```bash
# Construir la imagen
docker build -t jobmatcher-backend -f backend/Dockerfile .

# Ejecutar conectando a la red 'jobmatcher-network'
docker run -d \
  --name jobmatcher_backend \
  --net jobmatcher-network \
  -p 8000:8000 \
  --env-file .env \
  -v $(pwd)/model_cache:/app/model_cache \
  jobmatcher-backend
```

### 3. Despliegue del Frontend (Desde `g3r12025-frontend`)

El código del frontend se encuentra en el submódulo/carpeta `g3reto12025-frontend`.

#### Construcción para Producción
1. Navega a la carpeta del frontend:
   ```bash
   cd g3reto12025-frontend/frontend
   ```
2. Instala dependencias y construye:
   ```bash
   npm install
   npm run build
   ```
3. El resultado estará en `dist/` o `build/`.

#### Ejecución en Desarrollo
```bash
cd g3reto12025-frontend/frontend
npm start
```

---

## 🧪 Uso y Testing

1.  **Ingesta de Datos**:
    ```bash
    docker exec -it jobmatcher_backend python regenerate_embeddings.py
    ```
2.  **Verificación**: `http://localhost:8000/docs`
3.  **Chat**: Entra al frontend y prueba el recomendador.

## 📂 Estructura del Proyecto

*   **`g3r12025backend`** (Este repositorio):
    *   `/backend`: Código fuente Python (FastAPI).
    *   `/g3reto12025-frontend/frontend`: Código fuente React.
*   **`g3r12025devops`** (Externo):
    *   `/elasticsearch`: Configuración de base de datos.
    *   `/kibana`: Configuración de visualización.

---
**Autores**: Equipo AI-Somorrostro
