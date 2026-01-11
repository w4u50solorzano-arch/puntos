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

    <!-- SCRIPT DE MONETAG (Rewarded Interstitial) -->
    <!-- Este es el script necesario para que la función show_10088762 funcione -->
    <script src="https://alwingulla.com/88/pfe/current/tag.min.js?z=10088762" data-zone="10088762" async data-cfasync="false"></script>

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
            transform: scale(1.08);
            box-shadow: 0 0 40px rgba(34, 197, 94, 0.5);
        }

        #puntos-balance {
            font-size: 4rem;
            font-weight: 900;
            display: block;
            line-height: 1;
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

        #btn-video:disabled {
            background-color: #2d3545;
            color: #707d93;
            box-shadow: none;
            transform: translateY(4px);
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
                <p style="margin:10px 0 0; letter-spacing:2px; font-weight:bold; opacity:0.8;">MIS PUNTOS</p>
            </div>
        </header>

        <main>
            <section class="action-section" id="video-container">
                <button id="btn-video" onclick="lanzarPublicidad()">
                    📺 VER ANUNCIO (+1)
                </button>
                <p id="status-msg" style="margin-top:15px; height:20px;"></p>
            </section>

            <section class="premios-section">
                <h2 style="color: var(--text-dim); font-size: 1.1rem; margin-bottom: 15px;">Canjear Premios</h2>
                <div class="card"><div class="info"><h3>Datos Fijos</h3><p style="color:var(--primary); font-weight:bold; margin:5px 0 0;">30 Puntos</p></div><button class="btn-canje" id="btn-30" disabled>Faltan 30</button></div>
                <div class="card"><div class="info"><h3>Tripletas</h3><p style="color:var(--primary); font-weight:bold; margin:5px 0 0;">60 Puntos</p></div><button class="btn-canje" id="btn-60" disabled>Faltan 60</button></div>
                <div class="card"><div class="info"><h3>Plan Premium</h3><p style="color:var(--primary); font-weight:bold; margin:5px 0 0;">100 Puntos</p></div><button class="btn-canje" id="btn-100" disabled>Faltan 100</button></div>
            </section>
        </main>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        
        const MI_WHATSAPP = "584144952096";
        let puntosActuales = localStorage.getItem('puntos_user') ? parseInt(localStorage.getItem('puntos_user')) : 0;

        // --- INTEGRACIÓN CON MONETAG ---
        function lanzarPublicidad() {
            const btn = document.getElementById('btn-video');
            const msg = document.getElementById('status-msg');
            
            btn.disabled = true;
            msg.innerText = "Cargando anuncio...";

            // Verificamos si la función de Monetag existe
            if (typeof show_10088762 === 'function') {
                show_10088762().then(() => {
                    // ESTO SE EJECUTA CUANDO EL USUARIO TERMINA DE VER EL ANUNCIO
                    puntosActuales += 1;
                    localStorage.setItem('puntos_user', puntosActuales);
                    
                    celebrarLogro();
                    actualizarInterfaz(true);
                    
                    msg.style.color = "#22c55e";
                    msg.innerText = "¡Punto sumado con éxito!";
                    btn.disabled = false;
                    setTimeout(() => msg.innerText = "", 3000);
                }).catch(() => {
                    msg.style.color = "#ef4444";
                    msg.innerText = "Error al mostrar anuncio.";
                    btn.disabled = false;
                });
            } else {
                msg.innerText = "Anuncio no listo. Intenta de nuevo.";
                btn.disabled = false;
            }
        }

        // --- EFECTOS VISUALES ---
        function celebrarLogro() {
            if (tg.HapticFeedback) tg.HapticFeedback.notificationOccurred('success');
            
            confetti({
                particleCount: 150,
                spread: 70,
                origin: { y: 0.6 },
                colors: ['#0088cc', '#22c55e', '#ffffff']
            });

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

        // Contador animado
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

        function actualizarInterfaz(animar = false) {
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
                        tg.showConfirm(`¿Confirmas el canje de ${p.coste} puntos?`, (confirm) => {
                            if(confirm) {
                                puntosActuales -= p.coste;
                                localStorage.setItem('puntos_user', puntosActuales);
                                actualizarInterfaz(true);
                                const user = tg.initDataUnsafe?.user?.first_name || "Usuario";
                                tg.openLink(`https://wa.me/${MI_WHATSAPP}?text=Canje: ${p.nom} - De: ${user}`);
                            }
                        });
                    };
                } else {
                    b.disabled = true;
                    b.innerText = `Faltan ${p.coste - puntosActuales}`;
                    b.classList.remove('activo');
                    b.onclick = null;
                }
            });
        }

        // Iniciar App
        actualizarInterfaz(false);
    </script>
</body>
</html>
