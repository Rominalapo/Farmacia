<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Procesos - FarmaCuenca</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: #fff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h1 {
            color: #00796b;
            text-align: center;
            margin-bottom: 30px;
        }
        .grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
        }
        @media (max-width: 768px) {
            .grid { grid-template-columns: 1fr; }
        }
        .panel {
            background: #fafafa;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #e0e0e0;
        }
        .panel h2 {
            color: #004d40;
            margin-top: 0;
            border-bottom: 2px solid #00796b;
            padding-bottom: 8px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            font-weight: bold;
            margin-bottom: 8px;
        }
        input[type="range"] {
            width: 100%;
            margin-bottom: 5px;
        }
        .value-display {
            font-weight: bold;
            color: #00796b;
            float: right;
        }
        .metric {
            background: #e0f2f1;
            padding: 15px;
            border-radius: 6px;
            margin-bottom: 15px;
            border-left: 5px solid #00796b;
        }
        .metric h3 {
            margin: 0 0 5px 0;
            font-size: 0.9rem;
            color: #555;
            text-transform: uppercase;
        }
        .metric p {
            margin: 0;
            font-size: 1.8rem;
            font-weight: bold;
            color: #004d40;
        }
        .status-alert {
            padding: 15px;
            border-radius: 6px;
            font-weight: bold;
            text-align: center;
            margin-top: 20px;
        }
        .success { background-color: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
        .danger { background-color: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
        
        /* Simulación visual de fila */
        .fila-visual {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-top: 15px;
            padding: 10px;
            background: #eee;
            border-radius: 6px;
            min-height: 40px;
        }
        .cliente-icon {
            width: 25px;
            height: 25px;
            background-color: #00796b;
            border-radius: 50%;
            display: inline-block;
        }
        .cliente-icon.espera { background-color: #e53935; }
    </style>
</head>
<body>

<div class="container">
    <h1>Simulador de Impacto: Reingeniería FarmaCuenca</h1>
    
    <div class="grid">
        <div class="panel">
            <h2>Variables Modificables</h2>
            <p style="font-size: 0.9rem; color: #666;">Mueve los deslizadores para ver cómo impacta el cambio de estructura o de estrategia (Lean/BPR) en tiempo real.</p>
            
            <div class="form-group">
                <label for="tiempoCiclo">Tiempo de Atención por Cliente: 
                    <span class="value-display" id="tiempoDisplay">20 min</span>
                </label>
                <input type="range" id="tiempoCiclo" min="3" max="30" value="20">
                <small style="color: #777;">Actual (Estructura Funcional): 20 min. Objetivo Lean: 8 min.</small>
            </div>

            <div class="form-group">
                <label for="clientesDia">Clientes que llegan al día: 
                    <span class="value-display" id="clientesDisplay">40</span>
                </label>
                <input type="range" id="clientesDia" min="10" max="80" value="40">
                <small style="color: #777;">Demanda del mercado en Cuenca.</small>
            </div>

            <div class="form-group">
                <label for="horasJornada">Horas de Jornada Laboral: 
                    <span class="value-display" id="horasDisplay">8 horas</span>
                </label>
                <input type="range" id="horasJornada" min="4" max="12" value="8">
            </div>
        </div>

        <div class="panel">
            <h2>Impacto y Métricas del Proceso</h2>
            
            <div class="metric">
                <h3>Capacidad Máxima del Sistema</h3>
                <p id="capacidadMax">24 clientes/día</p>
            </div>

            <div class="metric">
                <h3>Tiempo Total Requerido de Atención</h3>
                <p id="tiempoTotalReq">800 minutos</p>
            </div>

            <div class="metric">
                <h3>Productividad del Tiempo (vs. Estado Inicial)</h3>
                <p id="cambioProductividad">0%</p>
            </div>

            <div id="estadoSistema" class="status-alert danger">
                Cargando estado...
            </div>
        </div>
    </div>

    <div class="panel" style="margin-top: 30px;">
        <h2>Simulación de Clientes Diarios vs Capacidad</h2>
        <p>Cada círculo representa un cliente de la demanda diaria. Los <span style="color:#e53935; font-weight:bold;">Rojos</span> representan clientes que causan colapso/espera excesiva por falta de capacidad del sistema.</p>
        <div class="fila-visual" id="contenedorClientes"></div>
    </div>
</div>

<script>
    // Elementos del DOM
    const inputTiempo = document.getElementById('tiempoCiclo');
    const inputClientes = document.getElementById('clientesDia');
    const inputHoras = document.getElementById('horasJornada');

    const txtTiempo = document.getElementById('tiempoDisplay');
    const txtClientes = document.getElementById('clientesDisplay');
    const txtHoras = document.getElementById('horasDisplay');

    const resCapacidad = document.getElementById('capacidadMax');
    const resTiempoTotal = document.getElementById('tiempoTotalReq');
    const resProductividad = document.getElementById('cambioProductividad');
    const divEstado = document.getElementById('estadoSistema');
    const contenedorClientes = document.getElementById('contenedorClientes');

    // Estado inicial para comparar productividad (Caso Base: 20 minutos)
    const TIEMPO_BASE = 20;

    function calcular() {
        // Obtener valores actuales
        const tiempo = parseInt(inputTiempo.value);
        const clientes = parseInt(inputClientes.value);
        const horas = parseInt(inputHoras.value);
        const minutosDisponibles = horas * 60;

        // Actualizar textos de los sliders
        txtTiempo.innerText = `${tiempo} min`;
        txtClientes.innerText = `${clientes}`;
        txtHoras.innerText = `${horas} horas`;

        // CÁLCULOS (Fórmulas del bloque 3 adaptadas)
        // 1. Capacidad Máxima = Minutos disponibles / Tiempo de ciclo
        const capacidad = Math.floor(minutosDisponibles / tiempo);
        
        // 2. Tiempo Total Requerido = Clientes * Tiempo de ciclo
        const tiempoRequerido = clientes * tiempo;

        // 3. Incremento de Productividad (Eficiencia del tiempo de ciclo respecto a los 20 min base)
        // Fórmula: ((Tiempo Base - Tiempo Actual) / Tiempo Base) * 100
        const variacionProductividad = ((TIEMPO_BASE - tiempo) / TIEMPO_BASE) * 100;

        // RENDERIZAR RESULTADOS
        resCapacidad.innerText = `${capacidad} clientes / día`;
        resTiempoTotal.innerText = `${tiempoRequerido} minutos (${(tiempoRequerido/60).toFixed(1)} horas)`;
        
        if(variacionProductividad >= 0) {
            resProductividad.innerText = `+${variacionProductividad.toFixed(0)}% de eficiencia`;
            resProductividad.style.color = "#00796b";
        } else {
            resProductividad.innerText = `${variacionProductividad.toFixed(0)}% (Más lento que el inicio)`;
            resProductividad.style.color = "#e53935";
        }

        // Alerta de estado del negocio
        if (clientes > capacidad) {
            divEstado.innerText = `⚠️ COLAPSO: La farmacia no avanza. Faltan ${tiempoRequerido - minutosDisponibles} minutos o más personal para atender a todos.`;
            divEstado.className = "status-alert danger";
        } else {
            divEstado.innerText = `✅ PROCESO EFICIENTE: El personal puede atender la demanda. Sobran ${minutosDisponibles - tiempoRequerido} minutos para otras tareas (ordenar, inventario).`;
            divEstado.className = "status-alert success";
        }

        // Render de la Fila Visual
        contenedorClientes.innerHTML = '';
        for (let i = 1; i <= clientes; i++) {
            const icono = document.createElement('div');
            icono.className = 'cliente-icon';
            if (i > capacidad) {
                icono.classList.add('espera'); // Cliente en zona de colapso
            }
            contenedorClientes.appendChild(icono);
        }
    }

    // Escuchadores de eventos para actualizar en tiempo real
    inputTiempo.addEventListener('input', calcular);
    inputClientes.addEventListener('input', calcular);
    inputHoras.addEventListener('input', calcular);

    // Ejecución inicial
    calcular();
</script>

</body>
</html>
