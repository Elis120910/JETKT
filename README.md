# JETKT
<!-- index.html -->
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>JEKT</title>
  <link rel="stylesheet" href="style.css" />
  <link rel="manifest" href="manifest.json" />
  <meta name="theme-color" content="#2e7d32" />
  <link rel="icon" href="icono_jekt_512.png" />
</head>
<body>
  <header>
    <img src="icono_jekt_512.png" alt="Logo reciclaje">
    <h1>JEKT</h1>
    <p>Convierte desechos en recompensas 🌱</p>
  </header>

  <main>
    <div class="card">
      <h2>📦 Registrar Desechable</h2>
      <label for="tipo">Tipo de producto</label>
      <select id="tipo">
        <option>Botella plástica</option>
        <option>Vidrio</option>
        <option>Cartón</option>
        <option>Lata</option>
      </select>

      <label for="cantidad">Cantidad</label>
      <input type="number" id="cantidad" placeholder="Ej. 3">

      <button onclick="registrarProducto()">Registrar</button>
      <p class="puntos">Total de puntos: <span id="puntos">0</span></p>
    </div>

    <div class="card">
      <h2>📍 Tu ubicación actual</h2>
      <p id="ubicacion">Cargando ubicación...</p>
    </div>

    <div class="card">
      <h2>♻️ Centros cercanos</h2>
      <ul id="centros-cercanos">
        <li>Cargando centros...</li>
      </ul>
    </div>
  </main>

  <footer>
    © 2025 JEKT - Todos los derechos reservados
  </footer>

  <script src="script.js"></script>
  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js')
        .then(() => console.log("✔️ Service Worker registrado"))
        .catch(err => console.log("❌ Error con el Service Worker", err));
    }
  </script>
</body>
</html>

<!-- style.css -->
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', sans-serif;
}

body {
  background: #e8f5e9;
  color: #333;
}

header {
  background-color: #2e7d32;
  color: white;
  text-align: center;
  padding: 1.5rem 1rem;
}

header img {
  width: 60px;
  margin-bottom: 10px;
}

main {
  padding: 1rem;
}

.card {
  background: white;
  border-radius: 12px;
  padding: 1rem;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.2rem;
}

.card h2 {
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
  color: #2e7d32;
}

label {
  display: block;
  margin-top: 0.5rem;
  font-size: 0.9rem;
}

input, select, button {
  width: 100%;
  padding: 0.6rem;
  margin-top: 0.3rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 1rem;
}

button {
  background-color: #43a047;
  color: white;
  border: none;
  margin-top: 1rem;
  cursor: pointer;
}

button:hover {
  background-color: #388e3c;
}

.puntos {
  font-weight: bold;
  color: #1b5e20;
  margin-top: 0.5rem;
}

footer {
  text-align: center;
  font-size: 0.8rem;
  padding: 1rem;
  color: #666;
}

@media (min-width: 600px) {
  .card {
    max-width: 500px;
    margin: 0 auto 1.2rem;
  }
}

<!-- script.js -->
let totalPuntos = 0;

const centros = [
  { nombre: "Punto Reciclaje San Borja", lat: -12.100, lon: -77.000 },
  { nombre: "EcoCentro Miraflores", lat: -12.120, lon: -77.030 },
  { nombre: "Recicla Chorrillos", lat: -12.200, lon: -77.010 },
  { nombre: "Centro Ambiental Surco", lat: -12.150, lon: -76.980 }
];

function registrarProducto() {
  const tipo = document.getElementById('tipo').value;
  const cantidad = parseInt(document.getElementById('cantidad').value);

  if (!cantidad || cantidad <= 0) {
    alert('Por favor, ingresa una cantidad válida.');
    return;
  }

  const puntosGanados = cantidad * 5;
  totalPuntos += puntosGanados;
  document.getElementById('puntos').textContent = totalPuntos;

  alert(`Registraste ${cantidad} unidad(es) de ${tipo} y ganaste ${puntosGanados} puntos.`);
}

function obtenerUbicacion() {
  const ubicacionEl = document.getElementById('ubicacion');
  const listaCentros = document.getElementById('centros-cercanos');

  if (!navigator.geolocation) {
    ubicacionEl.textContent = 'La geolocalización no está disponible.';
    return;
  }

  navigator.geolocation.getCurrentPosition(
    (position) => {
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;
      ubicacionEl.textContent = `Latitud: ${lat.toFixed(5)}, Longitud: ${lon.toFixed(5)}`;

      const cercanos = centros.map(c => {
        const distancia = calcularDistancia(lat, lon, c.lat, c.lon);
        return { ...c, distancia };
      }).sort((a, b) => a.distancia - b.distancia).slice(0, 3);

      listaCentros.innerHTML = "";
      cercanos.forEach(c => {
        listaCentros.innerHTML += `<li>${c.nombre} - ${c.distancia.toFixed(2)} km</li>`;
      });
    },
    () => {
      ubicacionEl.textContent = 'No se pudo obtener tu ubicación.';
      listaCentros.innerHTML = '<li>No se pudieron cargar los centros cercanos.</li>';
    }
  );
}

function calcularDistancia(lat1, lon1, lat2, lon2) {
  const R = 6371;
  const dLat = gradosARadianes(lat2 - lat1);
  const dLon = gradosARadianes(lon2 - lon1);
  const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
            Math.cos(gradosARadianes(lat1)) * Math.cos(gradosARadianes(lat2)) *
            Math.sin(dLon / 2) * Math.sin(dLon / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
}

function gradosARadianes(grados) {
  return grados * (Math.PI / 180);
}

window.onload = obtenerUbicacion;

<!-- manifest.json -->
{
  "name": "JEKT",
  "short_name": "JEKT",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#e8f5e9",
  "theme_color": "#2e7d32",
  "icons": [
    {
      "src": "icono_jekt_512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}

<!-- service-worker.js -->
self.addEventListener('install', function(e) {
  e.waitUntil(
    caches.open('jekt-v1').then(function(cache) {
      return cache.addAll([
        'index.html',
        'style.css',
        'script.js',
        'manifest.json',
        'icono_jekt_512.png'
      ]);
    })
  );
});

self.addEventListener('fetch', function(e) {
  e.respondWith(
    caches.match(e.request).then(function(response) {
      return response || fetch(e.request);
    })
  );
});
