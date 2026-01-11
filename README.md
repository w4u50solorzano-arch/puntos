<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Puntos Rewards Bot</title>
    <!-- SDKs de Telegram y Adsgram -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://sad.adsgram.ai/js/adsgram-ad-sdk.js"></script>
    
    <style>
        :root {
            --bg: #0b0e14;
            --card: #1c212d;
            --primary: #0088cc;
            --success: #22c55e;
            --text: #ffffff;
            --text-dim: #a0aabe;
            --transition: all 0.2s ease;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            display: flex;
            justify-content: center;
            min-height: 100vh;
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
            margin-bottom: 25px;
        }

        #balance-card {
            background: linear-gradient(135deg, #0088cc, #004466);
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 8px 20px rgba(0, 136, 204, 0.2);
        }

        #puntos-balance {
            font-size: 3.5rem;
            font-weight: 800;
            display: block;
        }

        #balance-card p {
            margin: 5px 0 0;
            font-size: 0.8rem;
            letter-spacing: 2px;
            text-transform: uppercase;
            opacity: 0.9;
        }

        .action-section {
            margin-bottom: 30px;
            text-align: center;
        }

        #btn-video {
            width: 100%;
            padding: 18px;
            border-radius: 15px;
            border: none;
            background-color: #ffffff;
            color: #000;
            font-size: 1.1rem;
            font-weight: 700;
            cursor: pointer;
            transition: var(--transition);
        }

        #btn-video:active { transform: scale(0.96); background-color: #e2e2e2; }

        #btn-video:disabled {
            background-color: #2d3545;
            color: #707d93;
        }

        #status-msg {
            min-height: 24px;
            margin-top: 12px;
            font-size: 0.95rem;
        }

        h2 { font-size: 1.1rem; color: var(--text-dim); margin-bottom: 15px; }

        .card {
            background-color: var(--card);
            padding: 16px;
            border-radius: 18px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            border: 1px solid #2d3545;
            animation: fadeInUp 0.4s ease forwards;
        }

        .info h3 { margin: 0; font-size: 1rem; }
        .costo { margin: 4px 0 0; color: var(--primary); font-weight: bold; }

        .btn-canje {
            padding: 12px 18px;
            border-radius: 12px;
            border: none;
            background-color: #2d3545;
            color: #707d93;
            font-weight: bold;
            transition: var(--transition);
        }

        .btn-canje.activo {
            background-color: var(--success);
            color: white;
            cursor: pointer;
        }

        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>

    <div class="app-container">
        <header>
            <div id="balance-card">
                <span id="puntos-balance">0</span>
                <p>MIS PUNTOS</p>
            </div>
        </header>

        <main>
            <section class="action-section">
                <button id="btn-video" onclick="verVideo()">📺 VER VIDEO (+1 PUNTO)</button>
                <p id="status-msg"></p>
            </section>

            <section class="premios-section">
                <h2>Premios Disponibles</h2>
                
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
        // 1. CONFIGURACIÓN
        const tg = window.Telegram.WebApp;
        tg.expand();
        
        const MI_WHATSAPP = "584144952096";
        const BLOCK_ID = "4280"; // Reemplaza con tu ID real de Adsgram
        
        let puntos = localStorage.getItem('puntos_user') ? parseInt(localStorage.getItem('puntos_user')) : 0;
        const AdController = window.Adsgram ? window.Adsgram.init({ UnitID: bot-20776}) : null;

        // 2. ACTUALIZAR INTERFAZ
        function actualizarUI() {
            document.getElementById('puntos-balance').innerText = puntos;
            
            const premios = [
                { id: 'btn-30', coste: 30, nombre: 'Datos Fijos' },
                { id: 'btn-60', coste: 60, nombre: 'Tripletas' },
                { id: 'btn-100', coste: 100, nombre: 'Plan Premium' }
            ];

            premios.forEach(premio => {
                const boton = document.getElementById(premio.id);
                if (puntos >= premio.coste) {
                    boton.disabled = false;
                    boton.innerText = "Canjear";
                    boton.classList.add('activo');
                    boton.onclick = () => procesarCanje(premio);
                } else {
                    boton.disabled = true;
                    boton.innerText = `Faltan ${premio.coste - puntos}`;
                    boton.classList.remove('activo');
                    boton.onclick = null;
                }
            });
        }

        // 3. LOGICA DE VIDEO
        async function verVideo() {
            const btn = document.getElementById('btn-video');
            const msg = document.getElementById('status-msg');
            
            btn.disabled = true;
            msg.innerText = "Cargando video...";
            msg.style.color = "#a0aabe";

            if (!AdController) {
                msg.innerText = "Error: Adsgram no disponible";
                btn.disabled = false;
                return;
            }

            try {
                const result = await AdController.show();
                if (result.done) {
                    puntos += 1;
                    localStorage.setItem('puntos_user', puntos);
                    if (tg.HapticFeedback) tg.HapticFeedback.impactOccurred('medium');
                    actualizarUI();
                    msg.style.color = "#22c55e";
                    msg.innerText = "¡Ganaste +1 punto!";
                } else {
                    msg.innerText = "Vea el video completo.";
                }
            } catch (e) {
                msg.innerText = "No hay videos ahora.";
            } finally {
                btn.disabled = false;
                setTimeout(() => msg.innerText = "", 3000);
            }
        }

        // 4. LOGICA DE CANJE
        function procesarCanje(premio) {
            tg.showConfirm(`¿Canjear ${premio.coste} puntos por ${premio.nombre}?`, (confirmado) => {
                if (confirmado) {
                    puntos -= premio.coste;
                    localStorage.setItem('puntos_user', puntos);
                    actualizarUI();
                    
                    const user = tg.initDataUnsafe?.user?.first_name || "Usuario";
                    const msj = `Hola, canjeé ${premio.coste} puntos por ${premio.nombre}. Mi nombre: ${user}`;
                    tg.openLink(`https://wa.me/${584144952096}?text=${encodeURIComponent(msj)}`);
                }
            });
        }

        actualizarUI();
    </script>
</body>
</html>
