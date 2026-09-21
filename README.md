# Para-ti-he
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Para una Amiga Especial 🌻</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background-color: #0d0c1d;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #ffffff;
            user-select: none;
        }

        /* Tarjeta central de mensaje */
        .tarjeta-contenedor {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 20;
            width: 90%;
            max-width: 550px;
            text-align: center;
            transition: opacity 1s ease, transform 1s ease;
        }

        .tarjeta-contenedor.oculto {
            opacity: 0;
            transform: translate(-50%, -60%) scale(0.9);
            pointer-events: none;
        }

        .tarjeta {
            background: rgba(20, 15, 38, 0.85);
            padding: 40px 30px;
            border-radius: 25px;
            box-shadow: 0 10px 40px rgba(255, 204, 0, 0.25);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 221, 87, 0.3);
        }

        h1 {
            font-size: 2.2em;
            margin-bottom: 15px;
            color: #ffd166;
            text-shadow: 0 0 15px rgba(255, 209, 102, 0.5);
        }

        p {
            font-size: 1.15em;
            color: #f1f5f9;
            line-height: 1.6;
            margin-bottom: 25px;
            font-weight: 300;
        }

        .btn-caminar {
            background: linear-gradient(135deg, #ffb703, #fb8500);
            color: #000;
            border: none;
            padding: 15px 32px;
            font-size: 1.1em;
            font-weight: bold;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 4px 20px rgba(251, 133, 0, 0.4);
            transition: all 0.3s ease;
        }

        .btn-caminar:hover {
            transform: scale(1.05);
            box-shadow: 0 6px 25px rgba(251, 133, 0, 0.6);
        }

        /* Botón para regresar el mensaje */
        .btn-regresar {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 15;
            background: rgba(0, 0, 0, 0.5);
            color: #ffd166;
            border: 1px solid rgba(255, 209, 102, 0.4);
            padding: 8px 16px;
            border-radius: 20px;
            cursor: pointer;
            backdrop-filter: blur(5px);
            opacity: 0;
            transition: opacity 0.5s ease;
            pointer-events: none;
        }

        .btn-regresar.visible {
            opacity: 1;
            pointer-events: auto;
        }

        /* Panel de depuración y prueba de audio */
        .panel-audio {
            position: absolute;
            bottom: 20px;
            left: 20px;
            z-index: 30;
            background: rgba(10, 10, 20, 0.85);
            border: 1px solid rgba(255, 209, 102, 0.3);
            border-radius: 15px;
            padding: 15px;
            max-width: 320px;
            font-size: 0.85em;
            backdrop-filter: blur(10px);
        }

        .panel-audio h3 {
            color: #ffd166;
            margin-bottom: 8px;
            font-size: 1em;
        }

        .estado-audio {
            margin-bottom: 10px;
            padding: 6px;
            border-radius: 6px;
            background: rgba(255, 255, 255, 0.05);
            word-break: break-all;
        }

        .btn-audio-test {
            background: #2196f3;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 6px;
            cursor: pointer;
            margin-right: 5px;
            margin-top: 5px;
        }

        .input-file-custom {
            display: none;
        }

        .lbl-file {
            display: inline-block;
            background: #4caf50;
            color: white;
            padding: 6px 12px;
            border-radius: 6px;
            cursor: pointer;
            margin-top: 5px;
        }

        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
        }
    </style>
</head>
<body>

    <!-- Audio del sistema -->
    <audio id="musicaFondo" src="cancion.mp3" loop preload="auto"></audio>

    <!-- Tarjeta Principal -->
    <div class="tarjeta-contenedor" id="tarjeta">
        <div class="tarjeta">
            <h1>Flores Amarillas Para Tu </h1>
            <p>
                La verdad eres una amiga increíble y muy especial. Nunca dudes de lo valiosa que eres ni de la luz que le aportas a quienes te rodean. ¡Disfruta la caminata!
            </p>
            <button class="btn-caminar" id="btnIniciar">Comenzar el recorrido </button>
        </div>
    </div>

    <button class="btn-regresar" id="btnRegresar">Detenerse a mirar 📜</button>

    <!-- Panel de diagnóstico de audio -->
    <div class="panel-audio">
        <h3>🔧 Prueba de Sonido</h3>
        <div class="estado-audio" id="estadoAudio">Estado: Esperando interacción...</div>
        
        <button class="btn-audio-test" id="btnForzarPlay">▶ Probar Audio</button>
        
        <label for="fileAudio" class="lbl-file">📁 Cargar MP3</label>
        <input type="file" id="fileAudio" class="input-file-custom" accept="audio/*">
    </div>

    <canvas id="campo"></canvas>

    <script>
        const canvas = document.getElementById('campo');
        const ctx = canvas.getContext('2d');
        const tarjeta = document.getElementById('tarjeta');
        const btnIniciar = document.getElementById('btnIniciar');
        const btnRegresar = document.getElementById('btnRegresar');
        const musica = document.getElementById('musicaFondo');
        const estadoAudio = document.getElementById('estadoAudio');
        const btnForzarPlay = document.getElementById('btnForzarPlay');
        const fileAudio = document.getElementById('fileAudio');

        let ancho, alto;
        let caminando = false;
        let avance3D = 0;
        let tiempoBamboleo = 0;

        function ajustarTamano() {
            ancho = canvas.width = window.innerWidth;
            alto = canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', ajustarTamano);
        ajustarTamano();

        // Diagnóstico de carga de audio
        musica.addEventListener('loadeddata', () => {
            estadoAudio.innerHTML = "✅ Archivo 'cancion.mp3' detectado correctamente.";
            estadoAudio.style.color = "#81c784";
        });

        musica.addEventListener('error', () => {
            estadoAudio.innerHTML = "⚠️ No se encontró 'cancion.mp3' en la carpeta. Usa el botón verde para seleccionarlo manualmente.";
            estadoAudio.style.color = "#e57373";
        });

        // Cargar archivo local manualmente si el navegador bloquea la lectura directa
        fileAudio.addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const url = URL.createObjectURL(file);
                musica.src = url;
                musica.play();
                estadoAudio.innerHTML = "✅ Audio cargado manualmente: " + file.name;
                estadoAudio.style.color = "#81c784";
            }
        });

        function reproducirMusica() {
            musica.play().then(() => {
                estadoAudio.innerHTML = "🔊 Reproduciendo audio...";
                estadoAudio.style.color = "#81c784";
            }).catch(err => {
                estadoAudio.innerHTML = "❌ Error al reproducir: " + err.message + ". Prueba cargar el archivo con el botón verde.";
                estadoAudio.style.color = "#e57373";
            });
        }

        btnForzarPlay.addEventListener('click', reproducirMusica);

        // Crear girasoles en espacio 3D
        const numGirasoles = 180;
        const girasoles = [];

        for (let i = 0; i < numGirasoles; i++) {
            girasoles.push({
                x: (Math.random() - 0.5) * 3500,
                z: Math.random() * 2000 + 100,
                tamano: Math.random() * 12 + 18,
                inclinacion: (Math.random() - 0.5) * 0.2
            });
        }

        // Lluvia de pétalos
        const numPetalos = 60;
        const petalos = [];
        for (let i = 0; i < numPetalos; i++) {
            petalos.push({
                x: Math.random() * ancho,
                y: Math.random() * alto,
                r: Math.random() * 4 + 2,
                vy: Math.random() * 1.5 + 0.5,
                vx: Math.random() * 1 - 0.5
            });
        }

        function dibujarGirasol(x, y, escala, inclinacion) {
            ctx.save();
            ctx.translate(x, y);
            ctx.scale(escala, escala);
            ctx.rotate(inclinacion);

            // Tallo
            ctx.strokeStyle = '#2d6a4f';
            ctx.lineWidth = 4;
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(0, 80);
            ctx.stroke();

            // Pétalos
            const numPetalosGirasol = 12;
            ctx.fillStyle = '#ffb703';
            for (let i = 0; i < numPetalosGirasol; i++) {
                ctx.rotate((Math.PI * 2) / numPetalosGirasol);
                ctx.beginPath();
                ctx.ellipse(0, 22, 6, 18, 0, 0, Math.PI * 2);
                ctx.fill();
            }

            // Centro
            ctx.fillStyle = '#3d2612';
            ctx.beginPath();
            ctx.arc(0, 0, 14, 0, Math.PI * 2);
            ctx.fill();

            ctx.restore();
        }

        function animar() {
            ctx.clearRect(0, 0, ancho, alto);

            // Atardecer cálido
            let gradienteFondo = ctx.createLinearGradient(0, 0, 0, alto);
            gradienteFondo.addColorStop(0, '#1d1135');
            gradienteFondo.addColorStop(0.5, '#6c234a');
            gradienteFondo.addColorStop(0.8, '#d97724');
            gradienteFondo.addColorStop(1, '#1b4332');
            ctx.fillStyle = gradienteFondo;
            ctx.fillRect(0, 0, ancho, alto);

            let offsetY = 0;
            let offsetX = 0;

            if (caminando) {
                avance3D += 6;
                tiempoBamboleo += 0.08;
                offsetY = Math.sin(tiempoBamboleo) * 12;
                offsetX = Math.cos(tiempoBamboleo * 0.5) * 8;
            }

            girasoles.sort((a, b) => b.z - a.z);

            const centroX = ancho / 2 + offsetX;
            const centroY = alto / 2 + offsetY + 50;

            girasoles.forEach(g => {
                let zActual = g.z - (caminando ? avance3D % 2000 : 0);
                if (zActual <= 10) zActual += 2000;

                const escala = 300 / zActual;
                const posX = centroX + g.x * escala;
                const posY = centroY + (alto * 0.2) * escala;

                if (posX > -100 && posX < ancho + 100 && posY > 0 && posY < alto + 150) {
                    dibujarGirasol(posX, posY, escala * (g.tamano / 15), g.inclinacion);
                }
            });

            // Pétalos
            ctx.fillStyle = '#ffd166';
            petalos.forEach(p => {
                p.y += p.vy;
                p.x += p.vx;
                if (p.y > alto) p.y = -10;
                if (p.x > ancho) p.x = 0;
                if (p.x < 0) p.x = ancho;

                ctx.beginPath();
                ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
                ctx.fill();
            });

            requestAnimationFrame(animar);
        }

        // Eventos de botones
        btnIniciar.addEventListener('click', () => {
            tarjeta.classList.add('oculto');
            btnRegresar.classList.add('visible');
            caminando = true;
            reproducirMusica();
        });

        btnRegresar.addEventListener('click', () => {
            tarjeta.classList.remove('oculto');
            btnRegresar.classList.remove('visible');
            caminando = false;
        });

        animar();
    </script>
</body>
</html>
