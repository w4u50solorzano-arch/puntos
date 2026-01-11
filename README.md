<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <!-- Meta de verificación Monetag -->
    <meta name="monetag" content="db8f471cb82be6057fc6b2d199208382">
    <title>Puntos Rewards Bot</title>
    
    <!-- SDK de Telegram y Librería de Confeti -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

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

        /* --- TARJETA DE BALANCE --- */
        #balance-card {
            background: linear-gradient(135deg, #0088cc, #004466);
            padding: 30px;
            border-radius: 24px;
            box-shadow: 0 10px 25px rgba(0, 136, 204, 0.3);
            transition: var(--transition);
        }

        #balance-card.pulse {
            transform: scale(1.1);
            box-shadow: 0 0 40px rgba(34, 197, 94, 0.5);
        }

        #puntos-balance {
            font-size: 4rem;
            font-weight: 900;
            display: block;
            line-height: 1;
        }

        /* --- BOTÓN PRINCIPAL --- */
        .action-section {
            margin-bottom: 35px;
            position: relative;
            text-align: center;
        }

        #btn-ganar {
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

        #btn-ganar:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #cccccc;
        }

        #status-msg {
            margin-top: 15px;
            height: 20px;
            font-size: 0.9rem;
            font-weight: 500;
        }

        /* --- PREMIOS --- */
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

        .info h3 { margin: 0; font-size: 1.1rem; }
        .costo { margin: 5px 0 0; color: var(--primary); font-weight: bold; }

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
                <p style="margin:10px 0 0; letter-spacing:2px; font-weight:bold; opacity:0.8;">PUNTOS TOTALES</p>
            </div>
        </header>

        <main>
            <section class="action-section">
                <button id="btn-ganar" onclick="abrirAnuncio()">
                    📺 VER PUBLICIDAD (+1)
                </button>
                <div id="status-msg"></div>
            </section>

            <section class="premios-section">
                <h2 style="color: var(--text-dim); font-size: 1.1rem; margin-bottom: 15px;">Canjear Premios</h2>
                
                <div class="card">
                    <div class="info">
                        <h3>Datos Fijos</h3>
                        <p class="costo">30 Puntos</p>
                    </div>
                    <button class="btn-canje" id="btn-30" disabled>Bloqueado</button>
                </div>

                <div class="card">
                    <div class="info">
                        <h3>Tripletas</h3>
                        <p class="costo">60 Puntos</p>
                    </div>
                    <button class="btn-canje" id="btn-60" disabled>Bloqueado</button>
                </div>

                <div class="card">
                    <div class="info">
                        <h3>Plan Premium</h3>
                        <p class="costo">100 Puntos</p>
                    </div>
                    <button class="btn-canje" id="btn-100" disabled>Bloqueado</button>
                </div>
            </section>
        </main>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        
        // --- CONFIGURACIÓN ---
        const DIRECT_LINK = "https://otieu.com/4/10431827";
        const MI_WHATSAPP = "584144952096";
        
        let puntos = localStorage.getItem('puntos_user') ? parseInt(localStorage.getItem('puntos_user')) : 0;

        // --- LÓGICA DE PUBLICIDAD ---
        function abrirAnuncio() {
            const msg = document.getElementById('status-msg');
            
            // 1. Abrir el link de Monetag de forma oficial en Telegram
            tg.openLink(DIRECT_LINK);

            msg.innerHTML = "<span style='color: #a0aabe'>Verificando anuncio...</span>";

            // 2. Simulamos un pequeño tiempo de espera para sumar el punto
            setTimeout(() => {
                puntos += 1;
                localStorage.setItem('puntos_user', puntos);
                
                celebrarExito();
                actualizarUI(true);
                
                msg.innerHTML = "<span style='color: #22c55e'>¡Punto sumado correctamente!</span>";
                setTimeout(() => msg.innerHTML = "", 3000);
            }, 3500); // 3.5 segundos después de hacer clic
        }

        // --- EFECTOS VISUALES ---
        function celebrarExito() {
            // Vibración
            if (tg.HapticFeedback) tg.HapticFeedback.notificationOccurred('success');
            
            // Confeti
            confetti({
                particleCount: 150,
                spread: 70,
                origin: { y: 0.6 },
                colors: ['#0088cc', '#22c55e', '#ffffff']
            });

            // Animación tarjeta
            const card = document.getElementById('balance-card');
            card.classList.add('pulse');
            setTimeout(() => card.classList.remove('pulse'), 500);
        }

        // Contador de puntos animado
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
            if (animar) animarContador(puntos);
            else document.getElementById('puntos-balance').innerText = puntos;
            
            const premios = [
                { id: 'btn-30', coste: 30, nom: 'Datos Fijos' },
                { id: 'btn-60', coste: 60, nom: 'Tripletas' },
                { id: 'btn-100', coste: 100, nom: 'Plan Premium' }
            ];

            premios.forEach(p => {
                const b = document.getElementById(p.id);
                if (puntos >= p.coste) {
                    b.disabled = false;
                    b.innerText = "Canjear";
                    b.classList.add('activo');
                    b.onclick = () => {
                        tg.showConfirm(`¿Canjear ${p.coste} puntos?`, (confirm) => {
                            if(confirm) {
                                puntos -= p.coste;
                                localStorage.setItem('puntos_user', puntos);
                                actualizarUI(true);
                                const user = tg.initDataUnsafe?.user?.first_name || "Usuario";
                                tg.openLink(`https://wa.me/${MI_WHATSAPP}?text=Canje: ${p.nom} - De: ${user}`);
                            }
                        });
                    };
                } else {
                    b.disabled = true;
                    b.innerText = `Faltan ${p.coste - puntos}`;
                    b.classList.remove('activo');
                    b.onclick = null;
                }
            });
        }

        // Iniciar
        actualizarUI();
    </script>
</body>
</html>
