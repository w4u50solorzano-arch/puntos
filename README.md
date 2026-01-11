<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <!-- Verificación de Monetag -->
    <meta name="monetag" content="db8f471cb82be6057fc6b2d199208382">
    <title>Puntos Rewards Bot</title>
    
    <!-- SDK de Telegram y Confetti -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

    <!-- SCRIPT DE MONETAG (REEMPLAZA ESTO) -->
    <!-- Pega aquí el script que te da Monetag para tu Zone ID -->
    <script src="https://alwingulla.com/88/pfe/current/tag.min.js?z=XXXXXXX" data-zone="XXXXXXX" async data-cfasync="false"></script>

    <style>
        :root {
            --bg: #0b0e14;
            --card: #1c212d;
            --primary: #0088cc;
            --success: #22c55e;
            --text: #ffffff;
            --text-dim: #a0aabe;
            --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            display: flex;
            justify-content: center;
            min-height: 100vh;
            overflow-x: hidden;
            -webkit-tap-highlight-color: transparent;
        }

        .app-container {
            width: 100%;
            max-width: 400px;
            padding: 20px;
            box-sizing: border-box;
        }

        header {
            text-align: center;
            margin-bottom: 30px;
        }

        #balance-card {
            background: linear-gradient(135deg, #0088cc, #004466);
            padding: 30px;
            border-radius: 24px;
            box-shadow: 0 10px 25px rgba(0, 136, 204, 0.3);
            transition: var(--transition);
        }

        #balance-card.pulse {
            transform: scale(1.05);
            box-shadow: 0 0 40px rgba(34, 197, 94, 0.5);
        }

        #puntos-balance {
            font-size: 4rem;
            font-weight: 900;
            display: block;
        }

        #balance-card p {
            margin: 10px 0 0;
            font-size: 0.85rem;
            letter-spacing: 3px;
            font-weight: bold;
            text-transform: uppercase;
            opacity: 0.8;
        }

        .action-section {
            margin-bottom: 35px;
            position: relative;
            text-align: center;
        }

        #btn-video {
            width: 100%;
            padding: 20px;
            border-radius: 18px;
            border: none;
            background-color: #ffffff;
            color: #000;
            font-size: 1.2rem;
            font-weight: 800;
            cursor: pointer;
            box-shadow: 0 6px 0 #cccccc;
            transition: all 0.1s;
        }

        #btn-video:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #cccccc;
        }

        .plus-one {
            position: absolute;
            color: var(--success);
            font-weight: bold;
            font-size: 2.5rem;
            pointer-events: none;
            animation: floatUp 1s ease-out forwards;
            left: 50%;
            z-index: 10;
        }

        @keyframes floatUp {
            0% { opacity: 0; transform: translate(-50%, 0); }
            100% { opacity: 0; transform: translate(-50%, -120px); }
        }

        .card {
            background-color: var(--card);
            padding: 18px;
            border-radius: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            border: 1px solid #2d3545;
        }

        .btn-canje {
            padding: 12px 20px;
            border-radius: 14px;
            border: none;
            background-color: #2d3545;
            color: #707d93;
            font-weight: 800;
        }

        .btn-canje.activo {
            background-color: var(--success);
            color: white;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div class="app-container">
        <header>
            <div id="balance-card">
                <span id="puntos-balance">0</span>
                <p>Puntos Acumulados</p>
            </div>
        </header>

        <main>
            <section class="action-section" id="video-container">
                <button id="btn-video" onclick="mostrarAnuncioMonetag()">
                    <span>📺</span> VER PUBLICIDAD (+1)
                </button>
                <p id="status-msg"></p>
            </section>

            <section class="premios-section">
                <h2 style="color: var(--text-dim); font-size: 1.1rem; margin-bottom: 15px;">Canjear Premios</h2>
                <div class="card"><div class="info"><h3>Datos Fijos</h3><p style="color:var(--primary)">30 Puntos</p></div><button class="btn-canje" id="btn-30" disabled>Bloqueado</button></div>
                <div class="card"><div class="info"><h3>Tripletas</h3><p style="color:var(--primary)">60 Puntos</p></div><button class="btn-canje" id="btn-60" disabled>Bloqueado</button></div>
                <div class="card"><div class="info"><h3>Plan Premium</h3><p style="color:var(--primary)">100 Puntos</p></div><button class="btn-canje" id="btn-100" disabled>Bloqueado</button></div>
            </section>
        </main>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        
        const MI_WHATSAPP = "584144952096";
        let puntosActuales = localStorage.getItem('puntos_user') ? parseInt(localStorage.getItem('puntos_user')) : 0;

        // --- FUNCIÓN PARA MONETAG ---
        function mostrarAnuncioMonetag() {
            const msg = document.getElementById('status-msg');
            msg.innerText = "Abriendo publicidad...";

            // 1. Llamamos al anuncio de Monetag
            // Si usas un Rewarded Interstitial de Monetag, el script suele activarse solo o mediante una función.
            // Si usas un "Direct Link", usa la línea de abajo:
            // window.open("TU_DIRECT_LINK_AQUI", "_blank");

            // Simulación de éxito de Monetag (Ya que Monetag no siempre devuelve si se vio el video)
            // Agregamos el punto inmediatamente o tras un pequeño delay
            setTimeout(() => {
                sumarPunto();
                msg.innerText = "¡Punto sumado!";
                setTimeout(() => msg.innerText = "", 2000);
            }, 2000); 
        }

        function sumarPunto() {
            puntosActuales += 1;
            localStorage.setItem('puntos_user', puntosActuales);
            
            // Animaciones
            celebrarEfecto();
            actualizarUI(true);
        }

        function celebrarEfecto() {
            if (tg.HapticFeedback) tg.HapticFeedback.notificationOccurred('success');
            confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
            
            const container = document.getElementById('video-container');
            const plusOne = document.createElement('div');
            plusOne.className = 'plus-one';
            plusOne.innerText = '+1';
            container.appendChild(plusOne);
            setTimeout(() => plusOne.remove(), 1000);

            const card = document.getElementById('balance-card');
            card.classList.add('pulse');
            setTimeout(() => card.classList.remove('pulse'), 500);
        }

        function animarContador(objetivo) {
            const el = document.getElementById('puntos-balance');
            const inicio = parseInt(el.innerText);
            const duracion = 800;
            let tiempoInicio = null;

            function paso(timestamp) {
                if (!tiempoInicio) tiempoInicio = timestamp;
                const progreso = Math.min((timestamp - tiempoInicio) / duracion, 1);
                const valorActual = Math.floor(progreso * (objetivo - inicio) + inicio);
                el.innerText = valorActual;
                if (progreso < 1) window.requestAnimationFrame(paso);
            }
            window.requestAnimationFrame(paso);
        }

        function actualizarUI(animar = false) {
            if (animar) animarContador(puntosActuales);
            else document.getElementById('puntos-balance').innerText = puntosActuales;
            
            const premios = [
                { id: 'btn-30', coste: 30, nom: 'Datos Fijos' },
                { id: 'btn-60', coste: 60, nom: 'Tripletas' },
                { id: 'btn-100', coste: 100, nom: 'Plan Premium' }
            ];

            premios.forEach(p => {
                const b = document.getElementById(p.id);
                if (puntosActuales >= p.coste) {
                    b.disabled = false;
                    b.innerText = "Canjear";
                    b.classList.add('activo');
                    b.onclick = () => {
                        tg.showConfirm(`¿Canjear ${p.coste} pts?`, (confirm) => {
                            if(confirm) {
                                puntosActuales -= p.coste;
                                localStorage.setItem('puntos_user', puntosActuales);
                                actualizarUI(true);
                                tg.openLink(`https://wa.me/${MI_WHATSAPP}?text=Canje: ${p.nom}`);
                            }
                        });
                    };
                } else {
                    b.disabled = true;
                    b.innerText = `Faltan ${p.coste - puntosActuales}`;
                    b.classList.remove('activo');
                }
            });
        }

        actualizarUI();
    </script>
</body>
</html>
