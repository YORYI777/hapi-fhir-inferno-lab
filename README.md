# Laboratorio: Despliegue de Servidor HAPI FHIR y Auditoría de Interoperabilidad con Inferno

**Autor:** Andres Felipe Sanchez Acuña  
**Perfil de GitHub:** [@Andres24xd](https://github.com/andres24xd)  

---

## 📄 Descripción del Proyecto
Este repositorio contiene la documentación y los pasos necesarios para desplegar un servidor médico HAPI FHIR utilizando Kubernetes (Minikube) y validar su cumplimiento con el estándar internacional de interoperabilidad en salud (HL7 FHIR) mediante la herramienta de auditoría Inferno de la ONC.

## 🛠️ Requisitos Previos
Para ejecutar este laboratorio, asegúrate de tener instalados y configurados los siguientes componentes:
* **Docker Desktop** (con al menos 6GB de memoria RAM asignada en su configuración).
* **Minikube** y **kubectl**.
* Herramienta **curl** para peticiones HTTP.

## 📂 Estructura del Proyecto
```text
.
├── fhir-minikube-starter/
│   └── scripts/start.sh
├── inferno-template/
│   └── run.sh
└── README.md
```

---

## 🚀 Paso 1: Despliegue del Clúster y Servidor
Para garantizar un rendimiento óptimo del servidor Java HAPI FHIR, inicializamos Minikube con 5 GB de memoria RAM.

1. **Iniciar Minikube:**
```bash
minikube start --driver=docker --memory=5120 --cpus=2
```

2. **Desplegar la infraestructura (HAPI + Base de datos):**
```bash
cd fhir-minikube-starter/scripts
./start.sh
```

3. **Exponer el servidor (Port-Forward):**
```bash
kubectl port-forward service/nginx-lb 8080:80 -n fhir-demo
```

## 🚀🏥 Paso 2: Operaciones CRUD (Estándar FHIR)
Con el servidor expuesto, interactuamos con la API FHIR para crear, leer, actualizar y eliminar recursos de tipo Patient.

4. **Crear un Paciente (POST):**
```bash
curl -i -X POST "[http://127.0.0.1:8080/fhir/Patient](http://127.0.0.1:8080/fhir/Patient)" \
  -H "Content-Type: application/fhir+json" \
  -d '{
        "resourceType": "Patient",
        "text": {
          "status": "generated",
          "div": "<div xmlns=\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml)\">Paciente: Nombre paciente</div>"
        },
        "name": [{"family": "apellido", "given": ["nombre"]}]
      }'
```

5. **Consultar el Paciente (GET):**
```bash
curl -i -X GET "[http://127.0.0.1:8080/fhir/Patient/1002](http://127.0.0.1:8080/fhir/Patient/1002)"
```

6. **Actualizar el Paciente (PUT):**
Agregamos un número de teléfono al recurso existente.
```bash
curl -i -X PUT "[http://127.0.0.1:8080/fhir/Patient/1002](http://127.0.0.1:8080/fhir/Patient/1002)" \
  -H "Content-Type: application/fhir+json" \
  -d '{
        "resourceType": "Patient",
        "id": "1002",
        "text": {"status": "generated", "div": "<div xmlns=\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml)\">Paciente: Nombre paciente</div>"},
        "name": [{"family": "apellido", "given": ["nombre"]}],
        "telecom": [{"system": "phone", "value": "300-1234567"}]
      }'
```

7. **Eliminar el Paciente (DELETE):**
```bash
curl -i -X DELETE "[http://127.0.0.1:8080/fhir/Patient/1002](http://127.0.0.1:8080/fhir/Patient/1002)"
```

## 🛡️ Paso 3: Auditoría de Interoperabilidad con Inferno
Para certificar que el servidor cumple con los estándares, utilizamos Inferno Framework.

8. **Iniciamos el contenedor de Inferno:**
Navega a la carpeta de inferno y levanta el entorno web:
```bash
cd inferno-template
./run.sh
```

9. **Ejecuta la validación:**
* Abre un navegador y dirígete a `http://localhost`.
* Selecciona el test suite de FHIR y haz clic en **Run All Tests**.
* **FHIR Endpoint URL:** Ingresa `http://host.docker.internal:8080/fhir` (Esta ruta es vital para que el contenedor de Docker pueda comunicarse con el clúster de Minikube).
* **Patient ID:** Ingresa el ID del paciente creado en el Paso 2 (ej. 1002).

10. **Resultados:**
Al finalizar el escaneo, todas las pruebas de validación de capacidades y estructura del recurso Patient finalizarán en estado Passed (Verde), demostrando un 100% de cumplimiento técnico.
