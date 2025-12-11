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

El sistema utiliza **imágenes pre-construidas** almacenadas en el Container Registry. El despliegue es extremadamente simple gracias a `docker-compose`.

### Repositorios del Proyecto

1.  **`g3r12025devops`**: Orquestación de servicios (ELK Stack, Backend, Scrapper).
2.  **`g3r12025backend`**: Código fuente del Backend (FastAPI + RAG).
3.  **`g3r12025-frontend`**: Código fuente del Frontend (React).

---

### Despliegue Completo (desde `g3r12025devops`)

1.  **Clonar el repositorio DevOps**:
    ```bash
    git clone git@github.com:ai-somorrostro/g3r12025devops.git
    cd g3r12025devops
    ```

2.  **Configurar variables de entorno**:
    Crea un archivo `.env` en la raíz con las credenciales necesarias:
    ```env
    # Elasticsearch
    ELASTIC_PASSWORD=tu_password_seguro
    
    # Backend API
    OPENROUTER_API_KEY=sk-or-v1-...
    ES_HOST=http://elasticsearch:9200
    ES_USER=elastic
    ES_PASSWORD=tu_password_seguro
    ```

3.  **Levantar todos los servicios**:
    ```bash
    docker-compose up -d
    ```
    
    Este comando automáticamente:
    - Descarga las imágenes pre-construidas del Container Registry
    - Levanta Elasticsearch y Kibana
    - Inicia el Backend y el Scrapper
    - Configura la red interna entre servicios

4.  **Verificar estado**:
    ```bash
    docker-compose ps
    ```

5.  **Acceder a los servicios**:
    - **Backend API**: `http://localhost:8000/docs`
    - **Kibana**: `http://localhost:5601`
    - **Elasticsearch**: `http://localhost:9200`

### Detener servicios

```bash
docker-compose down
```

---

### Notas Importantes

- **No se requiere construir imágenes localmente**: Las imágenes de Backend y Scrapper están en el Container Registry y se descargan automáticamente.
- **Configuraciones personalizadas**: Los archivos `elasticsearch/config/elasticsearch.yml` y `kibana/config/kibana.yml` se montan como volúmenes.
- **Persistencia de datos**: Elasticsearch usa un volumen Docker para mantener los datos entre reinicios.

---

## 📦 Arquitectura de Configuración ELK

El stack ELK se despliega utilizando **imágenes oficiales de Elastic** con configuraciones personalizadas montadas como volúmenes. Este enfoque permite separar la lógica de la infraestructura (imágenes base) de la configuración específica del proyecto.

### Estructura de Archivos de Configuración

```
g3r12025devops/
├── elasticsearch/
│   └── config/
│       └── elasticsearch.yml    # Configuración del nodo ES
├── kibana/
│   └── config/
│       └── kibana.yml            # Configuración de Kibana
├── logstash/
│   └── config/
│       └── logstash.yml          # Configuración de Logstash (opcional)
└── docker-compose.yml
```

### Estrategia de Despliegue por Componente

#### Elasticsearch
- **Imagen base**: `docker.elastic.co/elasticsearch/elasticsearch:8.15.0`
- **Configuración personalizada**: Se monta `elasticsearch/config/elasticsearch.yml` dentro del contenedor en `/usr/share/elasticsearch/config/elasticsearch.yml`
- **Propósito del yml**: Define configuración del clúster, seguridad (xpack), límites de memoria, CORS, networking entre nodos, etc.

**Ejemplo de montaje en docker-compose**:
```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
  volumes:
    - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
    - es_data:/usr/share/elasticsearch/data
  environment:
    - discovery.type=single-node
    - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
```

#### Kibana
- **Imagen base**: `docker.elastic.co/kibana/kibana:8.15.0`
- **Configuración personalizada**: Se monta `kibana/config/kibana.yml` en `/usr/share/kibana/config/kibana.yml`
- **Propósito del yml**: Define la URL de Elasticsearch, credenciales, servidor HTTP, idioma, índices por defecto, etc.

**Ejemplo de montaje**:
```yaml
kibana:
  image: docker.elastic.co/kibana/kibana:8.15.0
  volumes:
    - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```

#### Logstash (Opcional)
- **Imagen base**: `docker.elastic.co/logstash/logstash:8.15.0`
- **Configuración personalizada**: `logstash/config/logstash.yml` + pipelines en `logstash/pipeline/`
- **Propósito del yml**: Configuración de logging, monitorización, API settings, etc.

### Ventajas de Este Enfoque

1. **Separación de responsabilidades**: El código de configuración (yml) está versionado en git, mientras que las imágenes base se obtienen del registry oficial de Elastic.
2. **Actualizaciones simplificadas**: Para actualizar Elasticsearch de 8.15.0 a 8.16.0, solo se cambia la versión de la imagen en `docker-compose.yml`.
3. **Portabilidad**: Los mismos archivos yml funcionan en desarrollo, staging y producción cambiando solo las variables de entorno.
4. **Seguridad**: Secretos (passwords, certificados) se inyectan vía variables de entorno, no hardcodeados en los yml.

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
**Autores**: Samuel Rivera, Oier Cadierno y Jon Medina
