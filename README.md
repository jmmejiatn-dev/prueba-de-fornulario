<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema Técnico PRO</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
body {
  margin: 0;
  font-family: Arial;
  background: #0f172a;
  display: flex;
  justify-content: center;
}

.app {
  width: 100%;
  max-width: 430px;
  min-height: 100vh;
  background: #1e293b;
  padding: 20px;
  box-sizing: border-box;
  color: white;
}

input, select, button, textarea {
  width: 100%;
  margin: 8px 0;
  padding: 10px;
  border-radius: 8px;
  border: none;
  box-sizing: border-box;
}

button {
  background: #22c55e;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.preview-img {
  width: 100%;
  margin-top: 5px;
  border-radius: 8px;
}

.hidden {
  display: none;
}
</style>
</head>
<body>

<div class="app">

<!-- LOGIN -->
<div id="loginDiv">
<h2>LOGIN</h2>
<input type="text" id="usuario" placeholder="Usuario">
<input type="password" id="password" placeholder="Contraseña">
<button onclick="login()">Ingresar</button>
</div>

<!-- FORMULARIO -->
<div id="formDiv" class="hidden">
<form id="miFormulario" onsubmit="enviar(event)">

<h2>INFORME TÉCNICO</h2>

<input id="tarea" placeholder="TAREA" required>
<input id="fechaIngreso" type="date" required>
<input id="nombre" placeholder="NOMBRE" required>

<select id="area" required>
<option value="">Área</option>
<option>OPU</option>
<option>GIS</option>
<option>LAN</option>
<option>FIBRA</option>
</select>

<select id="ciudad" required>
<option value="">Ciudad</option>
<option>QUITO</option>
<option>GUAYAQUIL</option>
<option>AMBATO</option>
<option>CUENCA</option>
</select>

<input id="coordenadas" placeholder="Coordenadas GPS" readonly>

<h3>DATOS DE EQUIPO</h3>

<select id="equipo" required>
<option value="">Equipo</option>
</select>

<select id="modelo" required disabled>
<option value="">Modelo</option>
</select>

<select id="serie" required disabled>
<option value="">Serie</option>
</select>

<input id="direccion" placeholder="Dirección">

<label><input type="checkbox" value="Cambio" class="trabajo"> Cambio</label>
<label><input type="checkbox" value="Reubicación" class="trabajo"> Reubicación</label>

<textarea id="obs" placeholder="Observaciones"></textarea>

<h3>Adjuntar Imágenes</h3>
<input type="file" id="imagenes" multiple accept="image/*">
<div id="preview"></div>

<button type="submit">Generar PDF</button>
</form>
</div>

</div>

<script>

// LOGIN
function login() {
  if (usuario.value === "admin" && password.value === "1234") {
    loginDiv.classList.add("hidden");
    formDiv.classList.remove("hidden");
  } else {
    alert("Usuario o contraseña incorrectos ❌");
  }
}

// GPS
window.onload = function() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(function(pos) {
      coordenadas.value =
        pos.coords.latitude + ", " + pos.coords.longitude;
    });
  }

  Object.keys(baseDatos).forEach(e => {
    equipo.innerHTML += `<option value="${e}">${e.toUpperCase()}</option>`;
  });
};

// BASE DATOS
const baseDatos = {
  laptop: {
    "HP ProBook 250 G8": ["HP001", "HP002"]
  },
  otdr: {
    "EXFO MaxTester 720C": ["EX001"]
  },
  fusionadora: {
    "DVP-740": ["DVP001"]
  }
};

// DINÁMICA
equipo.addEventListener("change", function() {
  modelo.innerHTML = '<option value="">Modelo</option>';
  serie.innerHTML = '<option value="">Serie</option>';
  serie.disabled = true;

  if (!this.value) {
    modelo.disabled = true;
    return;
  }

  modelo.disabled = false;

  Object.keys(baseDatos[this.value]).forEach(m => {
    modelo.innerHTML += `<option value="${m}">${m}</option>`;
  });
});

modelo.addEventListener("change", function() {
  serie.innerHTML = '<option value="">Serie</option>';

  if (!this.value) {
    serie.disabled = true;
    return;
  }

  serie.disabled = false;

  baseDatos[equipo.value][this.value].forEach(s => {
    serie.innerHTML += `<option value="${s}">${s}</option>`;
  });
});

// VISTA PREVIA IMÁGENES
imagenes.addEventListener("change", function() {
  preview.innerHTML = "";
  Array.from(this.files).forEach(file => {
    const reader = new FileReader();
    reader.onload = function(e) {
      const img = document.createElement("img");
      img.src = e.target.result;
      img.className = "preview-img";
      preview.appendChild(img);
    };
    reader.readAsDataURL(file);
  });
});

// GENERAR PDF CON IMÁGENES
async function enviar(e) {
  e.preventDefault();

  const trabajos = [...document.querySelectorAll(".trabajo:checked")]
                    .map(t => t.value)
                    .join(", ");

  if (!trabajos) {
    alert("Seleccione tipo de trabajo");
    return;
  }

  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();

  let y = 20;

  doc.text("INFORME TECNICO", 20, y); y += 10;
  doc.text("Tarea: " + tarea.value, 20, y); y += 10;
  doc.text("Fecha: " + fechaIngreso.value, 20, y); y += 10;
  doc.text("Tecnico: " + nombre.value, 20, y); y += 10;
  doc.text("Area: " + area.value, 20, y); y += 10;
  doc.text("Ciudad: " + ciudad.value, 20, y); y += 10;
  doc.text("GPS: " + coordenadas.value, 20, y); y += 10;
  doc.text("Equipo: " + equipo.value, 20, y); y += 10;
  doc.text("Modelo: " + modelo.value, 20, y); y += 10;
  doc.text("Serie: " + serie.value, 20, y); y += 10;
  doc.text("Direccion: " + direccion.value, 20, y); y += 10;
  doc.text("Trabajo: " + trabajos, 20, y); y += 10;
  doc.text("Obs: " + obs.value, 20, y); y += 10;

  const files = imagenes.files;

  for (let i = 0; i < files.length; i++) {

    const reader = new FileReader();

    await new Promise(resolve => {
      reader.onload = function(event) {

        doc.addPage();
        doc.addImage(event.target.result, "JPEG", 15, 40, 180, 120);
        resolve();
      };
      reader.readAsDataURL(files[i]);
    });
  }

  doc.save("Informe_Tecnico.pdf");

  alert("PDF generado correctamente ✅");

  miFormulario.reset();
  modelo.disabled = true;
  serie.disabled = true;
  preview.innerHTML = "";
}

</script>

</body>
</html>
