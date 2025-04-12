# 📡 GlideAjax en ServiceNow: Guía completa con ejemplo funcional

GlideAjax es la herramienta en ServiceNow para ejecutar lógica del **lado del servidor** desde el **cliente (navegador)** de forma **asíncrona** y segura.  
Se usa cuando el cliente necesita consultar o procesar algo que **solo el servidor puede hacer**.

---

## 🎯 Objetivo del ejemplo

Desde un **Client Script**, vamos a consultar el número de **incidencias abiertas** que tiene el usuario actual y mostrarlo en pantalla.

Lo haremos dividiendo bien responsabilidades:
- La lógica y consulta estará en el servidor (Script Include).
- El cliente solo pedirá la info y la mostrará.

---

## 🧠 ¿Por qué no hacerlo todo en el cliente?

Porque el cliente:
- **No tiene acceso a GlideRecord directamente.**
- **No debería tener lógica sensible.**
- **No puede acceder a datos restringidos.**

Por eso usamos GlideAjax: el cliente pide → el servidor procesa → el cliente recibe solo el dato.

---

## 🔧 Paso 1: Crear el Script Include

> Este será nuestro "endpoint" del servidor. Le pondremos nombre, método y la lógica que queremos ejecutar.

```javascript
var CountUserIncidents = Class.create();
CountUserIncidents.prototype = Object.extendsObject(AbstractAjaxProcessor, {

  /**
   * Método que cuenta las incidencias abiertas del usuario actual
   * @returns {number} cantidad de incidencias
   */
  getOpenIncidentCount: function() {
    var userID = gs.getUserID(); // sys_id del usuario actual
    var gr = new GlideRecord('incident');
    gr.addQuery('caller_id', userID);         // Incidencias creadas por ese usuario
    gr.addQuery('state', '!=', '7');           // Excluye las cerradas (estado 7)
    gr.query();

    return gr.getRowCount(); // Devuelve el número total
  }

});
```
## 🔧 Paso 2: Crear el Client Script

> Este script se podrá usar, por ejemplo, al cargar un formulario (onLoad), y pedirá al servidor el dato que necesitamos.

```javascript
function getMyOpenIncidents() {
  // Crea la instancia de GlideAjax y apunta al Script Include
  var ga = new GlideAjax('CountUserIncidents');

  // Define el nombre del método que quieres ejecutar en el servidor
  ga.addParam('sysparm_name', 'getOpenIncidentCount');

  // Ejecuta la llamada y gestiona la respuesta
  ga.getXMLAnswer(function(response) {
    var count = response.responseXML.documentElement.getAttribute("answer"); // Asegúrate de parsear la respuesta
    alert("Tienes " + count + " incidencias abiertas 🛠️");
  });
}

getMyOpenIncidents(); // Ejecutar función al cargar
```


## 🧪 ¿Qué hace internamente GlideAjax?

  1.- El navegador crea una llamada a través de GlideAjax.

  2.- Se envía al Script Include indicado (si está marcado como Client Callable).

  3.- Se ejecuta el método especificado por 'sysparm_name'.
  
  4.- El método devuelve un valor (string).
  
  5.- El cliente lo recibe dentro de la función callback de getXMLAnswer.

  

## 🧰 Buenas prácticas

  🧼 Mantén tus Script Includes limpios y bien documentados.
  
  🔐 No expongas lógica crítica en el cliente.
  
  🧩 Si necesitas devolver más de un dato, encapsúlalo en JSON.
  
  🚫 No uses GlideAjax para escribir en tablas desde cliente.



  
