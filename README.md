<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>El Tesoro de la Librería</title>
    <style>
        :root { --madera: #8B4513; --papel: #FDF5E6; --verde: #2ECC71; --rojo: #E74C3C; }
        body { background: var(--madera); font-family: 'Comic Sans MS', cursive, sans-serif; text-align: center; color: #333; margin: 0; padding: 10px; }
        .tablero { background: var(--papel); max-width: 600px; margin: auto; padding: 20px; border-radius: 15px; box-shadow: 0 0 20px rgba(0,0,0,0.5); }
        .grilla { display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin-top: 20px; }
        .boton-opcion { padding: 15px; border: 3px solid var(--madera); border-radius: 10px; cursor: pointer; font-weight: bold; background: white; transition: 0.2s; }
        .boton-opcion:hover { background: #ffeaa7; }
        .btn-juego { padding: 15px 30px; font-size: 1.2em; cursor: pointer; border-radius: 10px; border: none; background: var(--verde); color: white; margin: 10px; }
        #mensaje { font-size: 1.3em; margin: 20px 0; height: 60px; }
        .oculto { display: none; }
    </style>
</head>
<body>

<div class="tablero">
    <h1>📚 Tesoro de la Librería</h1>
    <div id="info">Tiempo: <span id="timer">0</span>s | Aciertos: <span id="score">0</span>/10</div>
    <div id="mensaje">¡Ayuda a organizar la librería y encuentra el tesoro!</div>
    
    <button id="btn-iniciar" class="btn-juego" onclick="iniciarJuego()">COMENZAR AVENTURA</button>
    
    <div id="area-juego" class="oculto">
        <h2 id="pregunta"></h2>
        <div class="grilla" id="opciones"></div>
    </div>

    <div id="area-final" class="oculto">
        <div id="resultado-final" style="font-size: 2em;"></div>
        <button class="btn-juego" onclick="location.reload()">REINICIAR</button>
    </div>
</div>

<script>
    const baseDatos = [
        {p: "3 libros a C$12 cada uno. ¿Total?", r: [30, 36, 40], c: 36},
        {p: "45 libros y vendes 18. ¿Quedan?", r: [25, 27, 30], c: 27},
        {p: "5 libros al mes. ¿En 1 año?", r: [50, 60, 70], c: 60},
        {p: "8 estantes con 12 libros. ¿Total?", r: [96, 84, 100], c: 96},
        {p: "100 marcadores cuestan C$200. ¿Costo c/u?", r: [1, 2, 5], c: 2},
        // Nuevos problemas añadidos
        {p: "Lápiz cuesta C$5, compras 4. ¿Total?", r: [15, 20, 25], c: 20},
        {p: "Tajador C$8, compras 3. ¿Total?", r: [16, 24, 32], c: 24},
        {p: "Hojas de colores C$40, compras 2. ¿Total?", r: [70, 80, 90], c: 80},
        {p: "Marcador C$15, compras 3. ¿Total?", r: [30, 45, 60], c: 45},
        {p: "Borrador C$7, compras 5. ¿Total?", r: [28, 35, 42], c: 35},
        {p: "Un libro de C$50 y un lápiz de C$5. ¿Total?", r: [50, 55, 60], c: 55},
        {p: "Si tienes C$100 y gastas C$30 en hojas. ¿Quedan?", r: [60, 70, 80], c: 70},
        {p: "Borrador a C$10, compras 6. ¿Total?", r: [40, 50, 60], c: 60},
        {p: "Tajador a C$5, compras 10. ¿Total?", r: [40, 50, 60], c: 50},
        {p: "Dos marcadores a C$20 c/u. ¿Total?", r: [30, 40, 50], c: 40}
    ];

    let problemasJuego = [], indice = 0, aciertos = 0, tiempo = 0, crono;
    const ctx = new (window.AudioContext || window.webkitAudioContext)();

    function sonido(freq) {
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.connect(gain); gain.connect(ctx.destination);
        osc.frequency.value = freq;
        osc.start(); gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3);
        osc.stop(ctx.currentTime + 0.3);
    }

    function iniciarJuego() {
        document.getElementById('btn-iniciar').classList.add('oculto');
        document.getElementById('area-juego').classList.remove('oculto');
        crono = setInterval(() => { tiempo++; document.getElementById('timer').innerText = tiempo; }, 1000);
        
        // Mezclar y seleccionar 10 aleatorios
        problemasJuego = baseDatos.sort(() => Math.random() - 0.5).slice(0, 10);
        cargarProblema();
    }

    function cargarProblema() {
        const p = problemasJuego[indice];
        document.getElementById('pregunta').innerText = p.p;
        const cont = document.getElementById('opciones');
        cont.innerHTML = '';
        p.r.forEach(op => {
            cont.innerHTML += `<button class="boton-opcion" onclick="verificar(${op})">${op}</button>`;
        });
    }

    function verificar(op) {
        if (op === problemasJuego[indice].c) {
            sonido(600);
            aciertos++;
            document.getElementById('score').innerText = aciertos;
            document.getElementById('mensaje').innerText = "¡Correcto! ¡Un paso más!";
            indice++;
            if (indice < problemasJuego.length) cargarProblema();
            else finalizar();
        } else {
            sonido(150);
            document.getElementById('mensaje').innerText = "¡Intenta de nuevo!";
        }
    }

    function finalizar() {
        clearInterval(crono);
        document.getElementById('area-juego').classList.add('oculto');
        document.getElementById('area-final').classList.remove('oculto');
        const pts = (aciertos / problemasJuego.length) * 100;
        let msg = pts >= 75 ? "¡Eres un Gran Librero! ¡Excelente trabajo! 🏆" : "¡Sigue practicando, puedes mejorar! 💪";
        document.getElementById('resultado-final').innerHTML = `Puntaje: ${pts}%<br>Tiempo: ${tiempo}s<br>${msg}`;
    }
</script>
</body>
</html>
