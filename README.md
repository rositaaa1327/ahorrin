<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Desafío Ahorro Aleatorio</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f0f8ff;
      color: #333;
      text-align: center;
      padding: 20px;
    }
    h1 {
      color: #1a73e8;
    }
    .contenedor-principal {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .info {
      margin-top: 20px;
      background-color: #e3f2fd;
      border: 2px solid #1a73e8;
      border-radius: 10px;
      padding: 15px;
      max-width: 300px;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
      gap: 10px;
      margin-top: 30px;
      max-width: 700px;
    }
    .amount {
      padding: 10px;
      background-color: #fff;
      border-radius: 8px;
      border: 2px solid #1a73e8;
      cursor: pointer;
      transition: transform 0.2s, background-color 0.2s;
    }
    .amount.tachado {
      text-decoration: line-through;
      background-color: #bbdefb;
      color: #0d47a1;
      border-color: #0d47a1;
    }
    .amount:hover {
      transform: scale(1.05);
    }
    .reset-button {
      margin-top: 20px;
      padding: 10px 20px;
      background-color: #f44336;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.2s;
    }
    .reset-button:hover {
      background-color: #d32f2f;
    }
    #pantalla-inicio {
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      background: #ffffffdd;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      z-index: 100;
    }
    #pantalla-inicio input {
      margin: 10px;
      padding: 10px;
      font-size: 16px;
      border-radius: 8px;
      border: 1px solid #1a73e8;
    }
    #pantalla-inicio button {
      padding: 10px 20px;
      font-size: 16px;
      background: #1a73e8;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div id="pantalla-inicio">
    <h2>Crear Plan de Ahorro</h2>
    <input type="text" id="nombre-plan" placeholder="Nombre del plan" />
    <input type="number" id="meta-ahorro" placeholder="Meta de ahorro ($)" />
    <button onclick="iniciarPlan()">Comenzar</button>
  </div>

  <h1 id="titulo-plan">🎯 Desafío de Ahorro Aleatorio</h1>
  <p id="meta-texto">Tacha montos hasta alcanzar tu meta</p>
  <div class="contenedor-principal">
    <div class="grid" id="amounts"></div>
    <div class="info">
      <p><strong>Total acumulado:</strong></p>
      <p id="total-ahorrado">$0</p>
      <p><small id="meta-meta"></small></p>
      <button class="reset-button" onclick="reiniciarAhorro()">🔄 Reiniciar Ahorro</button>
    </div>
  </div>

  <script>
    function generarMontosMultiplo5(total, cantidad) {
      const opciones = [5,10,15,20,25,30,35,40,45,50];
      let montos = [];

      while (montos.length < cantidad) {
        let restante = total - montos.reduce((a,b) => a + b, 0);
        const posibles = opciones.filter(v => v <= restante);
        if (posibles.length === 0) break;
        const elegido = posibles[Math.floor(Math.random() * posibles.length)];
        montos.push(elegido);
      }

      while (montos.length < cantidad) montos.push(5);
      while (montos.length > cantidad) montos.pop();

      let sumaActual = montos.reduce((a,b)=>a+b,0);
      let dif = total - sumaActual;
      for (let i = 0; i < montos.length && dif !== 0; i++) {
        let nuevo = montos[i] + (dif >= 5 ? 5 : -5);
        if (nuevo >= 5 && nuevo <= 50) {
          montos[i] = nuevo;
          dif += (dif >= 5 ? -5 : 5);
        }
      }
      return montos.sort(() => Math.random() - 0.5);
    }

    const contenedor = document.getElementById('amounts');
    const totalDisplay = document.getElementById('total-ahorrado');
    const tituloPlan = document.getElementById('titulo-plan');
    const metaTexto = document.getElementById('meta-meta');
    const pantallaInicio = document.getElementById('pantalla-inicio');

    let nombrePlan = localStorage.getItem('nombre-plan') || '';
    let meta = parseInt(localStorage.getItem('meta-plan')) || 4000;
    let montos = JSON.parse(localStorage.getItem('montos-aleatorios'));
    let progreso = JSON.parse(localStorage.getItem('progreso-aleatorio')) || [];

    function iniciarPlan() {
      nombrePlan = document.getElementById('nombre-plan').value.trim();
      meta = parseInt(document.getElementById('meta-ahorro').value);
      if (!nombrePlan || isNaN(meta) || meta <= 0) {
        alert("Por favor completa todos los campos correctamente.");
        return;
      }
      localStorage.setItem('nombre-plan', nombrePlan);
      localStorage.setItem('meta-plan', meta);
      montos = generarMontosMultiplo5(meta, 100);
      localStorage.setItem('montos-aleatorios', JSON.stringify(montos));
      localStorage.setItem('progreso-aleatorio', JSON.stringify([]));
      pantallaInicio.style.display = 'none';
      actualizarTitulo();
      renderizarMontos();
      actualizarTotal();
    }

    function actualizarTitulo() {
      tituloPlan.textContent = `🎯 ${nombrePlan}`;
      metaTexto.textContent = `(Meta: $${meta})`;
    }

    function actualizarTotal() {
      const total = progreso.reduce((acc, i) => acc + montos[i], 0);
      totalDisplay.textContent = `$${total}`;
    }

    function renderizarMontos() {
      contenedor.innerHTML = '';
      montos.forEach((monto, index) => {
        const div = document.createElement('div');
        div.className = 'amount';
        div.textContent = `$${monto}`;
        if (progreso.includes(index)) {
          div.classList.add('tachado');
        }
        div.addEventListener('click', () => {
          div.classList.toggle('tachado');
          if (progreso.includes(index)) {
            progreso = progreso.filter(i => i !== index);
          } else {
            progreso.push(index);
          }
          localStorage.setItem('progreso-aleatorio', JSON.stringify(progreso));
          actualizarTotal();
        });
        contenedor.appendChild(div);
      });
    }

    function reiniciarAhorro() {
      if (confirm("¿Estás seguro de reiniciar el ahorro? Se perderá el progreso actual.")) {
        localStorage.removeItem('montos-aleatorios');
        localStorage.removeItem('progreso-aleatorio');
        montos = generarMontosMultiplo5(meta, 100);
        progreso = [];
        localStorage.setItem('montos-aleatorios', JSON.stringify(montos));
        renderizarMontos();
        actualizarTotal();
      }
    }

    if (nombrePlan && meta) {
      pantallaInicio.style.display = 'none';
      actualizarTitulo();
      if (!montos) {
        montos = generarMontosMultiplo5(meta, 100);
        localStorage.setItem('montos-aleatorios', JSON.stringify(montos));
      }
      renderizarMontos();
      actualizarTotal();
    }
  </script>
</body>
</html>
