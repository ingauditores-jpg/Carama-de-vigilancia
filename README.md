<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SentryCam AI - Centinela Autónomo v5.2</title>
    <!-- Tailwind CSS para diseño premium militar -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons para iconografía táctica -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&family=Share+Tech+Mono&display=swap');
        
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: radial-gradient(circle at top, #111118 0%, #010103 100%);
        }
        
        .mono-font {
            font-family: 'Share Tech Mono', monospace;
        }

        @keyframes scan-pulse {
            0%, 100% {
                border-color: rgba(239, 68, 68, 0.4);
                box-shadow: inset 0 0 15px rgba(239, 68, 68, 0.2);
            }
            50% {
                border-color: rgba(239, 68, 68, 1);
                box-shadow: inset 0 0 30px rgba(239, 68, 68, 0.5), 0 0 15px rgba(239, 68, 68, 0.3);
            }
        }

        .scanning-active {
            animation: scan-pulse 1.5s infinite;
        }

        @keyframes radar-sweep {
            0% { top: 0%; }
            50% { top: 100%; }
            100% { top: 0%; }
        }

        .radar-line {
            height: 2px;
            background: linear-gradient(90deg, rgba(239,68,68,0) 0%, rgba(239,68,68,0.8) 50%, rgba(239,68,68,0) 100%);
            animation: radar-sweep 4s infinite linear;
        }

        .bg-grid {
            background-size: 35px 35px;
            background-image: linear-gradient(to right, rgba(255, 255, 255, 0.01) 1px, transparent 1px),
                              linear-gradient(to bottom, rgba(255, 255, 255, 0.01) 1px, transparent 1px);
        }

        .custom-scroll::-webkit-scrollbar {
            width: 4px;
        }
        .custom-scroll::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.2);
        }
        .custom-scroll::-webkit-scrollbar-thumb {
            background: rgba(239,68,68,0.2);
            border-radius: 10px;
        }
        .custom-scroll::-webkit-scrollbar-thumb:hover {
            background: rgba(239,68,68,0.5);
        }
    </style>
</head>
<body class="min-h-screen text-zinc-100 flex flex-col justify-between bg-grid relative overflow-x-hidden antialiased">

    <!-- Pantalla de Activación Inicial (Garantiza permisos de cámara sin bloqueos de navegador) -->
    <div id="startOverlay" class="fixed inset-0 bg-zinc-950/95 z-50 flex flex-col items-center justify-center p-6 text-center transition-all duration-500">
        <div class="max-w-md space-y-6">
            <div class="w-20 h-20 bg-red-600/10 rounded-full border border-red-500/30 flex items-center justify-center mx-auto animate-pulse">
                <i data-lucide="shield-alert" class="w-10 h-10 text-red-500"></i>
            </div>
            <div class="space-y-2">
                <h2 class="text-2xl font-bold tracking-wider uppercase text-white">SentryCam AI Pro</h2>
                <p class="text-zinc-400 text-sm">Inicia el centinela óptico autónomo de seguridad. Autoriza el acceso a la cámara trasera para el análisis de movimiento en tiempo real.</p>
            </div>
            <button id="btnStartEngine" class="w-full py-4 bg-red-600 hover:bg-red-700 text-white font-bold uppercase rounded-xl transition-all shadow-lg shadow-red-900/40 flex items-center justify-center gap-2">
                <i data-lucide="power" class="w-5 h-5"></i>
                Iniciar Centinela Óptico
            </button>
        </div>
    </div>

    <!-- Encabezado Táctico -->
    <header class="border-b border-zinc-900 bg-zinc-950/80 backdrop-blur-md sticky top-0 z-40 px-4 py-3 sm:px-6">
        <div class="max-w-7xl mx-auto flex flex-col sm:flex-row justify-between items-center gap-3">
            <div class="flex items-center gap-3">
                <div class="p-2 bg-red-600/10 rounded-xl text-red-500 border border-red-500/20 shadow-lg">
                    <i data-lucide="eye" class="w-6 h-6 animate-pulse"></i>
                </div>
                <div>
                    <h1 class="text-md font-bold tracking-wider text-white flex items-center gap-1.5 uppercase">
                        Sentry<span class="text-red-500 font-extrabold text-xs px-2 py-0.5 rounded bg-red-500/10 border border-red-500/30">CAM ACTIVE v5.2</span>
                    </h1>
                    <p class="text-[9px] text-zinc-500 tracking-widest uppercase font-semibold">Vigilancia Autónoma con Autodiagnóstico Integrado</p>
                </div>
            </div>
            
            <div class="flex items-center gap-3">
                <div class="flex items-center gap-2 bg-zinc-900/80 px-3 py-1.5 rounded-lg border border-zinc-800 text-xs text-zinc-400">
                    <span class="w-2 h-2 rounded-full bg-red-500 animate-ping"></span>
                    <span id="engineStatusText">INICIANDO IA...</span>
                </div>
                <span id="systemTime" class="mono-font text-zinc-400 text-sm bg-zinc-900 px-3 py-1.5 rounded-lg border border-zinc-800">12:00:00 AM</span>
            </div>
        </div>
    </header>

    <!-- Banner de Advertencia de Diagnóstico Completo -->
    <div id="telegramWarningBanner" class="bg-amber-950/40 border-b border-amber-900/50 px-4 py-2.5 text-center text-xs text-amber-300 flex items-center justify-center gap-2 transition-all duration-300">
        <i data-lucide="alert-triangle" class="w-4 h-4 text-amber-400 animate-bounce"></i>
        <span id="warningBannerText">Inicia el sistema ingresando tu **Chat ID de Telegram** para activar los envíos automáticos de capturas.</span>
    </div>

    <!-- Contenido Principal -->
    <main class="max-w-7xl w-full mx-auto p-4 sm:p-6 flex-grow grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <!-- COLUMNA MONITOR (8 Columnas) -->
        <section class="lg:col-span-8 space-y-6">
            
            <!-- Contenedor del Monitor Central -->
            <div id="monitorContainer" class="relative rounded-2xl border border-zinc-850 bg-zinc-950 overflow-hidden shadow-2xl transition-all duration-300 scanning-active">
                
                <!-- Línea de radar -->
                <div class="radar-line absolute w-full left-0 z-10 pointer-events-none"></div>

                <!-- Superposiciones de estado superior izquierdo -->
                <div class="absolute top-4 left-4 z-20 flex flex-wrap gap-2 pointer-events-none">
                    <div id="statusBadge" class="flex items-center gap-2 bg-red-600/20 border border-red-500/30 text-red-400 px-3 py-1.5 rounded-full text-xs font-bold backdrop-blur-md shadow-md">
                        <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse"></span>
                        <span id="statusText">MODO AUTÓNOMO ACTIVO</span>
                    </div>
                    <div id="cooldownIndicator" class="hidden flex items-center gap-1.5 bg-zinc-900/90 border border-zinc-800 text-yellow-500 px-2.5 py-1.5 rounded-full text-[10px] font-bold tracking-wider uppercase backdrop-blur-md">
                        <i data-lucide="clock" class="w-3.5 h-3.5 animate-spin"></i>
                        <span>ESPERA AUTOMÁTICA (<span id="cooldownSeconds">0</span>s)</span>
                    </div>
                </div>

                <!-- Botón para activar cámara -->
                <div class="absolute bottom-4 left-4 z-20 flex gap-2">
                    <button id="forceCameraBtn" class="px-3 py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded-lg shadow-lg flex items-center gap-1.5 transition-all">
                        <i data-lucide="camera" class="w-4 h-4"></i>
                        <span>Activar Cámara de Seguridad</span>
                    </button>
                </div>

                <!-- Controles del Monitor -->
                <div class="absolute top-4 right-4 z-20 flex gap-2">
                    <button id="toggleEngineBtn" class="px-3 py-1.5 bg-red-600 hover:bg-red-700 text-white text-xs font-bold rounded-lg shadow-lg flex items-center gap-1.5 transition-all">
                        <i data-lucide="power" class="w-4 h-4"></i>
                        <span id="toggleEngineText">Pausar Centinela</span>
                    </button>
                    <button id="fullscreenBtn" class="p-2 bg-zinc-900/80 border border-zinc-800 text-zinc-400 hover:text-white hover:bg-zinc-850 rounded-lg backdrop-blur-md transition-colors" title="Pantalla Completa">
                        <i data-lucide="maximize-2" class="w-4 h-4"></i>
                    </button>
                </div>

                <!-- Feed del Sensor -->
                <div class="relative w-full aspect-video flex items-center justify-center bg-black overflow-hidden">
                    <video id="webcam" class="absolute inset-0 w-full h-full object-cover hidden" autoplay playsinline muted></video>
                    <canvas id="staticCanvas" class="absolute inset-0 w-full h-full object-cover block opacity-25"></canvas>
                    <canvas id="trackingCanvas" class="absolute inset-0 w-full h-full object-cover z-10"></canvas>

                    <!-- Elementos tácticos en el Feed -->
                    <div class="absolute inset-0 pointer-events-none p-4 flex flex-col justify-between z-10 select-none">
                        <div class="flex justify-between text-zinc-500 font-mono text-[9px]">
                            <div>AUTO_VIGILANCIA: CONFIG_CHECK</div>
                            <div>SENTRY_CAM_SECURE_MODE</div>
                        </div>
                        <div class="flex justify-between items-end text-zinc-500 font-mono text-[9px]">
                            <div>SISTEMA AUTÓNOMO INTELIGENTE</div>
                            <div>BATTERY_PASS_THROUGH</div>
                        </div>
                    </div>

                    <!-- Pantalla de Error o Carga Táctica -->
                    <div id="cameraLoadingScreen" class="absolute inset-0 bg-zinc-950 flex flex-col items-center justify-center gap-3 z-30 transition-opacity duration-500">
                        <div class="animate-spin text-red-500">
                            <i data-lucide="loader-2" class="w-8 h-8"></i>
                        </div>
                        <p class="text-zinc-400 text-xs tracking-widest uppercase text-center px-4">Iniciando algoritmo de análisis visual autónomo...</p>
                    </div>
                </div>

                <!-- Consola inferior del monitor -->
                <div class="bg-zinc-900/90 border-t border-zinc-800/80 px-4 py-3 flex flex-wrap gap-4 items-center justify-between text-xs mono-font text-zinc-400">
                    <div class="flex items-center gap-2">
                        <span class="text-red-500 animate-pulse">● ANALIZANDO FLUJO ÓPTICO</span>
                        <span class="text-zinc-700">|</span>
                        <span>VARIACIÓN SENSOR: <span id="currentDiffVal" class="text-white">0.0%</span></span>
                    </div>
                    <div class="flex items-center gap-2">
                        <span>ESTADO TELEGRAM: </span>
                        <span id="telegramStatusLabel" class="text-red-400 font-bold">DESCONECTADO</span>
                    </div>
                </div>
            </div>

            <!-- Panel de Autodiagnóstico Interactivo (EXPLICACIÓN PASO A PASO EN PANTALLA) -->
            <div class="bg-zinc-900/50 border border-zinc-800/80 rounded-xl p-5 space-y-3 shadow-lg">
                <h3 class="text-xs font-bold uppercase text-zinc-300 tracking-wider flex items-center gap-2">
                    <i data-lucide="activity" class="text-red-500 w-4 h-4"></i>
                    Diagnóstico y Solución de Problemas de Seguridad
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs">
                    <div class="bg-zinc-950/50 p-3 rounded-lg border border-zinc-850 space-y-2">
                        <span class="font-bold text-red-400 flex items-center gap-1">
                            <span class="w-1.5 h-1.5 bg-red-500 rounded-full animate-ping"></span>
                            1. ¿La cámara trasera no se activa?
                        </span>
                        <p class="text-zinc-400 text-[11px] leading-relaxed">
                            Los navegadores bloquean la cámara si abres el archivo localmente como <code class="bg-zinc-900 px-1 py-0.5 rounded text-red-300 font-mono">file:///...</code>. 
                            <strong>Solución:</strong> Sube el archivo a un servidor web seguro con <strong class="text-white">HTTPS</strong> (como GitHub Pages o un servidor local en tu PC). El botón "Forzar Cámara" intentará re-evaluar los permisos.
                        </p>
                    </div>
                    <div class="bg-zinc-950/50 p-3 rounded-lg border border-zinc-850 space-y-2">
                        <span class="font-bold text-red-400 flex items-center gap-1">
                            <span class="w-1.5 h-1.5 bg-red-500 rounded-full animate-ping"></span>
                            2. ¿Error "chat not found" en la Bitácora?
                        </span>
                        <p class="text-zinc-400 text-[11px] leading-relaxed">
                            Tu Token ya está configurado. Para recibir alertas, <strong>debes buscar a tu bot en Telegram y presionar "INICIAR" (/start)</strong>. Si no has iniciado conversación previa, Telegram no le permitirá mandarte mensajes.
                        </p>
                    </div>
                </div>
            </div>

            <!-- Visualizadores de IA -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                
                <!-- Sensor de movimiento diferencial -->
                <div class="bg-zinc-900/40 border border-zinc-800/60 rounded-xl p-4 flex flex-col gap-3">
                    <div class="flex items-center justify-between">
                        <h3 class="text-xs font-bold tracking-wider text-zinc-400 uppercase flex items-center gap-1.5">
                            <i data-lucide="eye" class="w-3.5 h-3.5 text-red-500"></i>
                            Matriz de Diferencia Digital
                        </h3>
                        <span class="text-[9px] bg-red-950 text-red-400 border border-red-900/30 px-1.5 py-0.5 rounded uppercase font-bold">Mapa de Calor</span>
                    </div>
                    <div class="relative aspect-video rounded-lg overflow-hidden border border-zinc-850 bg-zinc-950">
                        <canvas id="analysisCanvas" class="w-full h-full object-cover"></canvas>
                    </div>
                </div>

                <!-- Última Captura Enviada -->
                <div class="bg-zinc-900/40 border border-zinc-800/60 rounded-xl p-4 flex flex-col justify-between gap-4">
                    <div class="flex items-center justify-between">
                        <h3 class="text-xs font-bold tracking-wider text-zinc-400 uppercase flex items-center gap-1.5">
                            <i data-lucide="image" class="w-3.5 h-3.5 text-zinc-500"></i>
                            Captura Procesada y Enviada
                        </h3>
                        <span id="sendProgressBadge" class="text-[9px] bg-zinc-800 text-zinc-400 px-1.5 py-0.5 rounded uppercase font-bold">Inactivo</span>
                    </div>
                    
                    <div class="relative flex-grow aspect-video bg-zinc-950 rounded-lg overflow-hidden border border-zinc-850 flex items-center justify-center">
                        <img id="lastCapturedPreview" src="" class="hidden w-full h-full object-cover">
                        <div id="noCapturePlaceholder" class="text-center p-4 flex flex-col items-center gap-1.5">
                            <i data-lucide="camera-off" class="w-8 h-8 text-zinc-700"></i>
                            <span class="text-[11px] text-zinc-500 uppercase tracking-wider">Esperando detección para captura automática...</span>
                        </div>
                        <div id="sendingSpinner" class="hidden absolute inset-0 bg-black/80 flex flex-col items-center justify-center gap-2 z-30 font-semibold">
                            <div class="animate-spin text-red-500"><i data-lucide="loader-2" class="w-6 h-6"></i></div>
                            <span class="text-[10px] text-zinc-300 tracking-wider font-mono">TRANSMITIENDO CAPTURA A TELEGRAM...</span>
                        </div>
                    </div>
                </div>

            </div>

        </section>

        <!-- CONFIGURACIÓN DE TELEGRAM (4 Columnas) -->
        <section class="lg:col-span-4 space-y-6">
            
            <!-- CONFIGURACIÓN DE TELEGRAM -->
            <div class="bg-zinc-900/60 border border-zinc-850 rounded-xl p-5 shadow-lg space-y-4">
                <h2 class="text-sm font-bold uppercase text-white tracking-wider flex items-center gap-2 border-b border-zinc-800 pb-3">
                    <i data-lucide="send" class="w-4 h-4 text-blue-400"></i>
                    Parámetros de Telegram
                </h2>

                <div class="space-y-3">
                    <div class="space-y-1">
                        <label class="text-[10px] text-zinc-400 uppercase tracking-wider block font-bold">Token de Bot (Sincronizado)</label>
                        <!-- Token proporcionado pre-rellenado y bloqueado para evitar fallos de escritura -->
                        <input type="text" id="telegramToken" value="8845128487:AAHQLpwetv0CMOapSSM7ppnvdGNanNAqhX8" readonly
                               class="w-full bg-zinc-950/80 border border-zinc-800/50 rounded px-2.5 py-1.5 text-xs text-zinc-500 cursor-not-allowed mono-font select-all">
                        <p class="text-[9px] text-emerald-500 font-semibold flex items-center gap-1">
                            <i data-lucide="check" class="w-3.5 h-3.5"></i> Token completo cargado con éxito.
                        </p>
                    </div>
                    <div class="space-y-1">
                        <label class="text-[10px] text-zinc-400 uppercase tracking-wider block font-bold">Tu Chat ID de Telegram</label>
                        <input type="text" id="telegramChatId" placeholder="Ingresa tu Chat ID (Ej: 987654321)" 
                               class="w-full bg-zinc-950 border border-zinc-800 rounded px-2.5 py-1.5 text-xs text-zinc-200 placeholder-zinc-700 focus:outline-none focus:border-red-500 font-bold mono-font">
                        <p class="text-[9px] text-zinc-500">Este ID es único y numérico. El sistema lo guardará automáticamente en el almacenamiento local.</p>
                    </div>

                    <!-- Lista de control dinámica interactiva para el usuario -->
                    <div class="p-3 bg-zinc-950/60 border border-zinc-850 rounded-lg space-y-2 text-[11px] text-zinc-400">
                        <span class="font-bold text-zinc-300 block uppercase text-[9px] tracking-wider">Lista de Activación:</span>
                        <label class="flex items-start gap-2 cursor-pointer">
                            <input type="checkbox" id="checkStart" class="mt-0.5 accent-red-500 rounded">
                            <span>Busqué al Bot en Telegram y presioné **INICIAR** (/start)</span>
                        </label>
                        <label class="flex items-start gap-2 cursor-pointer">
                            <input type="checkbox" id="checkId" class="mt-0.5 accent-red-500 rounded">
                            <span>Mi Chat ID tiene sólo números (obtenido de @userinfobot)</span>
                        </label>
                    </div>
                    
                    <button id="saveTelegramConfigBtn" class="w-full py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs uppercase rounded transition-colors flex items-center justify-center gap-1.5 shadow-md">
                        <i data-lucide="check-circle" class="w-4 h-4"></i> Guardar y Validar Bot
                    </button>
                    
                    <p class="text-[9px] text-zinc-500 leading-relaxed pt-1 border-t border-zinc-800/60 mt-2">
                        Para obtener tu Chat ID personal, búscalo enviando cualquier mensaje a la cuenta <a href="https://t.me/userinfobot" target="_blank" class="text-blue-400 hover:underline text-semibold">@userinfobot</a> en Telegram.
                    </p>
                </div>
            </div>

            <!-- PARÁMETROS DEL ALGORITMO -->
            <div class="bg-zinc-900/60 border border-zinc-850 rounded-xl p-5 shadow-lg space-y-4">
                <h2 class="text-sm font-bold uppercase text-white tracking-wider flex items-center gap-2 border-b border-zinc-800 pb-3">
                    <i data-lucide="sliders" class="w-4 h-4 text-red-500"></i>
                    Ajustes de Calibración
                </h2>

                <!-- Selector de entrada rápida -->
                <div class="space-y-1.5">
                    <label class="text-[10px] text-zinc-400 block uppercase font-bold">Origen de Captura</label>
                    <div class="grid grid-cols-2 gap-2">
                        <button id="sourceRealBtn" class="py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-zinc-950 border-zinc-800 text-zinc-400 hover:text-white">
                            <i data-lucide="video" class="w-3.5 h-3.5"></i> Cámara Física
                        </button>
                        <button id="sourceSimBtn" class="py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-red-500/10 border-red-500/30 text-red-400 font-bold">
                            <i data-lucide="cpu" class="w-3.5 h-3.5"></i> Simulador
                        </button>
                    </div>
                </div>

                <!-- Sensibilidad -->
                <div class="space-y-1.5 pt-2">
                    <div class="flex justify-between items-center text-xs">
                        <span class="text-zinc-400 font-bold uppercase text-[10px]">Sensibilidad del Sensor:</span>
                        <span id="sliderSensVal" class="text-red-400 font-bold mono-font">30</span>
                    </div>
                    <input type="range" id="sensibilidad" min="5" max="100" value="30" 
                           class="w-full accent-red-500 bg-zinc-800 h-1.5 rounded-lg appearance-none cursor-pointer">
                </div>

                <!-- Cooldown -->
                <div class="space-y-1.5 pt-2">
                    <div class="flex justify-between items-center text-xs">
                        <span class="text-zinc-400 font-bold uppercase text-[10px]">Intervalo de Reportes:</span>
                        <span id="cooldownText" class="text-red-400 font-bold mono-font">10 segundos</span>
                    </div>
                    <input type="range" id="cooldownSlider" min="3" max="60" value="10" 
                           class="w-full accent-red-500 bg-zinc-800 h-1.5 rounded-lg appearance-none cursor-pointer">
                </div>

                <!-- Alarma Local -->
                <div class="flex items-center justify-between text-xs bg-zinc-950/40 p-2.5 rounded-lg border border-zinc-800/40">
                    <span class="text-zinc-300 flex items-center gap-1.5">
                        <i data-lucide="volume-2" class="w-4 h-4 text-zinc-500"></i>
                        Sirena de Alerta Local
                    </span>
                    <button id="soundToggle" class="w-10 h-5 bg-zinc-800 rounded-full relative p-0.5 transition-colors duration-200 focus:outline-none">
                        <div class="w-4 h-4 bg-zinc-400 rounded-full transform translate-x-0 transition-transform duration-200" id="soundToggleDot"></div>
                    </button>
                </div>
            </div>

            <!-- BITÁCORA DE TRANSMISIÓN -->
            <div class="bg-zinc-900/60 border border-zinc-850 rounded-xl p-5 shadow-lg flex flex-col gap-3">
                <div class="flex items-center justify-between border-b border-zinc-800 pb-2">
                    <h2 class="text-sm font-bold uppercase text-white tracking-wider flex items-center gap-2">
                        <i data-lucide="activity" class="w-4 h-4 text-red-500"></i>
                        Bitácora de IA
                    </h2>
                    <span id="totalAlertsVal" class="text-[10px] bg-zinc-800 px-2 py-0.5 rounded text-zinc-400 font-bold">0 Eventos</span>
                </div>
                <div id="logsContainer" class="space-y-2 max-h-[140px] overflow-y-auto custom-scroll pr-1">
                    <div id="noLogsMsg" class="text-center py-6 text-zinc-500 text-xs">
                        <i data-lucide="terminal" class="w-6 h-6 mx-auto mb-2 text-zinc-600"></i>
                        Sistema listo para recibir parámetros.
                    </div>
                </div>
            </div>

        </section>

    </main>

    <!-- Pie de página -->
    <footer class="border-t border-zinc-900 bg-zinc-950/80 backdrop-blur-md px-6 py-4 mt-6">
        <div class="max-w-7xl mx-auto flex flex-col sm:flex-row justify-between items-center gap-2 text-xs text-zinc-500 text-center sm:text-left">
            <div>
                <p>© 2026 SentryCam Autonomous Security.</p>
                <p class="text-[10px] text-zinc-600">Procesamiento diferencial de fotogramas directo en la GPU/CPU local del dispositivo.</p>
            </div>
        </div>
    </footer>

    <!-- Script de Control Inteligente -->
    <script>
        lucide.createIcons();

        // Referencias HTML de Lienzos y Video
        const video = document.getElementById('webcam');
        const staticCanvas = document.getElementById('staticCanvas');
        const analysisCanvas = document.getElementById('analysisCanvas');
        const trackingCanvas = document.getElementById('trackingCanvas');
        
        const staticCtx = staticCanvas.getContext('2d');
        const analysisCtx = analysisCanvas.getContext('2d', { willReadFrequently: true });
        const trackingCtx = trackingCanvas.getContext('2d');

        // Elementos de UI
        const systemTime = document.getElementById('systemTime');
        const statusBadge = document.getElementById('statusBadge');
        const statusText = document.getElementById('statusText');
        const currentDiffVal = document.getElementById('currentDiffVal');
        const engineStatusText = document.getElementById('engineStatusText');
        const cameraLoadingScreen = document.getElementById('cameraLoadingScreen');
        const loadingSpinner = document.getElementById('loadingSpinner');
        const loadingText = document.getElementById('loadingText');
        const monitorContainer = document.getElementById('monitorContainer');
        const camResolution = document.getElementById('camResolution');
        const cooldownIndicator = document.getElementById('cooldownIndicator');
        const cooldownSeconds = document.getElementById('cooldownSeconds');
        const telegramWarningBanner = document.getElementById('telegramWarningBanner');
        const warningBannerText = document.getElementById('warningBannerText');
        const telegramStatusLabel = document.getElementById('telegramStatusLabel');
        const sendProgressBadge = document.getElementById('sendProgressBadge');
        const lastCapturedPreview = document.getElementById('lastCapturedPreview');
        const noCapturePlaceholder = document.getElementById('noCapturePlaceholder');
        const sendingSpinner = document.getElementById('sendingSpinner');
        const startOverlay = document.getElementById('startOverlay');
        const btnStartEngine = document.getElementById('btnStartEngine');

        // Checkboxes del asistente
        const checkStart = document.getElementById('checkStart');
        const checkId = document.getElementById('checkId');

        // Controles de Configuración
        const sliderSensibilidad = document.getElementById('sensibilidad');
        const sliderSensVal = document.getElementById('sliderSensVal');
        const cooldownSlider = document.getElementById('cooldownSlider');
        const cooldownText = document.getElementById('cooldownText');
        const soundToggle = document.getElementById('soundToggle');
        const soundToggleDot = document.getElementById('soundToggleDot');
        const toggleEngineBtn = document.getElementById('toggleEngineBtn');
        const toggleEngineText = document.getElementById('toggleEngineText');
        const forceCameraBtn = document.getElementById('forceCameraBtn');

        // Botones de fuente
        const sourceRealBtn = document.getElementById('sourceRealBtn');
        const sourceSimBtn = document.getElementById('sourceSimBtn');

        // Telegram Bot Inputs
        const telegramToken = document.getElementById('telegramToken');
        const telegramChatId = document.getElementById('telegramChatId');
        const saveTelegramConfigBtn = document.getElementById('saveTelegramConfigBtn');

        // Logs
        const logsContainer = document.getElementById('logsContainer');
        const noLogsMsg = document.getElementById('noLogsMsg');
        const totalAlertsVal = document.getElementById('totalAlertsVal');

        // Estados Operacionales de la IA
        let isEngineRunning = true;
        let isUsingRealCamera = false;
        let isAudioOn = false;
        let stream = null;
        let fotoAnterior = null;
        let totalAlerts = 0;
        let simulationAngle = 0;
        
        let isInCooldown = false;
        let cooldownTimeRemaining = 0;
        let cooldownInterval = null;

        let telegramConfig = {
            token: "8845128487:AAHQLpwetv0CMOapSSM7ppnvdGNanNAqhX8", // Inyectado directamente
            chatId: ""
        };

        // Inicializar Audio Context
        let audioCtx = null;
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function playBeep(frequency = 1000, duration = 0.2) {
            if (!isAudioOn) return;
            try {
                initAudio();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(frequency, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + duration);
            } catch (e) {
                console.warn(e);
            }
        }

        function updateClock() {
            const now = new Date();
            systemTime.innerText = now.toLocaleTimeString();
        }
        setInterval(updateClock, 1000);
        updateClock();

        // Conversor robusto de Base64 DataURL a archivo Blob
        function dataURLtoBlob(dataurl) {
            try {
                var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
                    bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
                while(n--){
                    u8arr[n] = bstr.charCodeAt(n);
                }
                return new Blob([u8arr], {type:mime});
            } catch (err) {
                console.error("Fallo al convertir Base64 a Blob:", err);
                return null;
            }
        }

        // 1. CARGA DE CONFIGURACIÓN GUARDADA (LocalStorage)
        function loadSavedCredentials() {
            const savedChatId = localStorage.getItem('sentry_telegram_chat_id');
            if (savedChatId) {
                telegramChatId.value = savedChatId;
                telegramConfig.chatId = savedChatId;
                
                checkStart.checked = true;
                checkId.checked = true;

                // Actualizar badges visuales de sincronización
                telegramWarningBanner.classList.add('hidden');
                telegramStatusLabel.innerText = "SOCIOCONECTADO";
                telegramStatusLabel.className = "text-emerald-400 font-bold";
                addLog("Credenciales de Telegram recuperadas automáticamente.");
            }
        }

        // 2. GESTIÓN ÓPTICA DE CÁMARA TRASERA
        async function setVideoSource(useReal) {
            isUsingRealCamera = useReal;
            cameraLoadingScreen.style.opacity = '1';
            cameraLoadingScreen.style.pointerEvents = 'auto';
            loadingSpinner.classList.remove('hidden');
            loadingText.innerHTML = "Iniciando algoritmo de análisis visual autónomo...";

            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
            }

            if (useReal) {
                // Verificar si el navegador corre bajo contexto seguro HTTPS
                if (window.location.protocol !== 'https:' && window.location.hostname !== 'localhost' && window.location.hostname !== '127.0.0.1') {
                    loadingSpinner.classList.add('hidden');
                    loadingText.innerHTML = `
                        <span class="text-red-500 font-bold block mb-2">⚠️ PROTOCOLO NO SEGURO DETECTADO</span>
                        Los navegadores móviles exigen conexiones seguras <span class="text-white bg-red-900/40 px-1.5 py-0.5 rounded font-mono">HTTPS</span> para activar la cámara física. 
                        Sube este archivo a un servidor seguro o usa el <span class="text-yellow-400 font-bold cursor-pointer underline" onclick="setVideoSource(false)">Simulador</span> para probar las alertas de Telegram.
                    `;
                    addLog("Fallo de cámara: No se cuenta con conexión HTTPS segura.");
                    return;
                }

                sourceRealBtn.className = "py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-red-500/10 border-red-500/30 text-red-400 font-bold";
                sourceSimBtn.className = "py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-zinc-950 border-zinc-800 text-zinc-400 hover:text-white";
                video.classList.remove('hidden');
                staticCanvas.classList.add('hidden');

                try {
                    stream = await navigator.mediaDevices.getUserMedia({
                        video: { facingMode: "environment", width: { ideal: 640 }, height: { ideal: 480 } },
                        audio: false
                    });
                    video.srcObject = stream;
                    video.onloadedmetadata = () => {
                        analysisCanvas.width = 160;
                        analysisCanvas.height = 120;
                        trackingCanvas.width = video.videoWidth;
                        trackingCanvas.height = video.videoHeight;
                        
                        camResolution.innerText = `RESOLUCIÓN: ${video.videoWidth}x${video.videoHeight} (REAL)`;
                        cameraLoadingScreen.style.opacity = '0';
                        cameraLoadingScreen.style.pointerEvents = 'none';
                        engineStatusText.innerText = "MONITOREO REAL";
                        addLog("Lente óptico trasero iniciado con éxito.");
                    };
                } catch (err) {
                    console.error("No se pudo iniciar cámara física:", err);
                    loadingSpinner.classList.add('hidden');
                    loadingText.innerHTML = `
                        <span class="text-red-500 font-bold block mb-2">⚠️ CÁMARA NO ENCONTRADA O BLOQUEADA</span>
                        Asegúrate de conceder el permiso de acceso a la cámara o que tu teléfono tenga una cámara trasera habilitada.
                    `;
                    addLog("La cámara física falló o fue rechazada. Iniciando simulador...");
                    setTimeout(() => setVideoSource(false), 2000);
                }
            } else {
                sourceSimBtn.className = "py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-red-500/10 border-red-500/30 text-red-400 font-bold";
                sourceRealBtn.className = "py-1.5 px-3 text-[10px] font-bold rounded border transition-all duration-200 flex items-center justify-center gap-1 bg-zinc-950 border-zinc-800 text-zinc-400 hover:text-white";
                video.classList.add('hidden');
                staticCanvas.classList.remove('hidden');

                staticCanvas.width = 640;
                staticCanvas.height = 480;
                analysisCanvas.width = 160;
                analysisCanvas.height = 120;
                trackingCanvas.width = 640;
                trackingCanvas.height = 480;
                camResolution.innerText = "RESOLUCIÓN: 640x480 (SIMULADOR)";
                
                setTimeout(() => {
                    cameraLoadingScreen.style.opacity = '0';
                    cameraLoadingScreen.style.pointerEvents = 'none';
                    engineStatusText.innerText = "MUESTRA SIMULADA";
                }, 400);
            }
        }

        // Generador de estática y objetivo para el simulador de pruebas
        function renderSimulatedFeed() {
            if (isUsingRealCamera) return;

            staticCtx.fillStyle = '#060609';
            staticCtx.fillRect(0, 0, staticCanvas.width, staticCanvas.height);

            const imgData = staticCtx.getImageData(0, 0, staticCanvas.width, staticCanvas.height);
            const data = imgData.data;
            for (let i = 0; i < data.length; i += 4) {
                const noise = (Math.random() - 0.5) * 6;
                data[i] = Math.max(0, Math.min(255, 10 + noise));
                data[i+1] = Math.max(0, Math.min(255, 12 + noise));
                data[i+2] = Math.max(0, Math.min(255, 16 + noise));
            }
            staticCtx.putImageData(imgData, 0, 0);

            staticCtx.strokeStyle = 'rgba(239, 68, 68, 0.03)';
            staticCtx.lineWidth = 1;
            for (let i = 0; i < staticCanvas.width; i += 80) {
                staticCtx.beginPath(); staticCtx.moveTo(i, 0); staticCtx.lineTo(i, staticCanvas.height); staticCtx.stroke();
            }
            for (let j = 0; j < staticCanvas.height; j += 60) {
                staticCtx.beginPath(); staticCtx.moveTo(0, j); staticCtx.lineTo(staticCanvas.width, j); staticCtx.stroke();
            }

            // Esfera de calor simulada
            simulationAngle += 0.02;
            const isMovingNow = (Math.sin(simulationAngle * 0.5) > -0.4); 
            if (isMovingNow) {
                const intruderX = staticCanvas.width / 2 + Math.cos(simulationAngle) * (staticCanvas.width * 0.35);
                const intruderY = staticCanvas.height / 2 + Math.sin(simulationAngle * 2.5) * (staticCanvas.height * 0.2);
                
                const grad = staticCtx.createRadialGradient(intruderX, intruderY, 4, intruderX, intruderY, 30);
                grad.addColorStop(0, 'rgba(255, 255, 255, 0.85)');
                grad.addColorStop(0.3, 'rgba(255, 255, 255, 0.2)');
                grad.addColorStop(1, 'rgba(255, 255, 255, 0)');
                
                staticCtx.fillStyle = grad;
                staticCtx.beginPath();
                staticCtx.arc(intruderX, intruderY, 30, 0, Math.PI * 2);
                staticCtx.fill();
            }
        }

        // 3. ANÁLISIS DE DIFERENCIA Y RASTREO TÁCTICO DE IA
        function runAutonomousAnalysis() {
            if (!isEngineRunning) return;

            const threshold = parseInt(sliderSensibilidad.value);
            const areaCriticaDeDisparo = 0.045; // 4.5% de cambio mínimo

            let sourceImageElement = null;
            if (isUsingRealCamera) {
                if (video.readyState === video.HAVE_ENOUGH_DATA) {
                    sourceImageElement = video;
                }
            } else {
                sourceImageElement = staticCanvas;
            }

            trackingCtx.clearRect(0, 0, trackingCanvas.width, trackingCanvas.height);

            if (sourceImageElement) {
                analysisCtx.drawImage(sourceImageElement, 0, 0, analysisCanvas.width, analysisCanvas.height);
                const fotogramaActual = analysisCtx.getImageData(0, 0, analysisCanvas.width, analysisCanvas.height);
                const pixeles = fotogramaActual.data;

                const diffMap = analysisCtx.createImageData(analysisCanvas.width, analysisCanvas.height);
                const diffData = diffMap.data;

                if (fotoAnterior) {
                    let pixelesCambiados = 0;
                    const totalPixeles = pixeles.length / 4;

                    let minX = analysisCanvas.width;
                    let maxX = 0;
                    let minY = analysisCanvas.height;
                    let maxY = 0;

                    for (let i = 0; i < pixeles.length; i += 4) {
                        const indexPixel = i / 4;
                        const pixelX = indexPixel % analysisCanvas.width;
                        const pixelY = Math.floor(indexPixel / analysisCanvas.width);

                        const lum1 = (pixeles[i] + pixeles[i+1] + pixeles[i+2]) / 3;
                        const lum2 = (fotoAnterior[i] + fotoAnterior[i+1] + fotoAnterior[i+2]) / 3;

                        if (Math.abs(lum1 - lum2) > threshold) {
                            pixelesCambiados++;
                            
                            diffData[i] = 239;     // R
                            diffData[i+1] = 68;    // G
                            diffData[i+2] = 68;    // B
                            diffData[i+3] = 255;   // Opaco

                            if (pixelX < minX) minX = pixelX;
                            if (pixelX > maxX) maxX = pixelX;
                            if (pixelY < minY) minY = pixelY;
                            if (pixelY > maxY) maxY = pixelY;
                        } else {
                            diffData[i] = 10;
                            diffData[i+1] = 10;
                            diffData[i+2] = 15;
                            diffData[i+3] = 120;
                        }
                    }

                    analysisCtx.putImageData(diffMap, 0, 0);

                    const ratioModificado = pixelesCambiados / totalPixeles;
                    const porcentajePantalla = (ratioModificado * 100).toFixed(1);
                    currentDiffVal.innerText = `${porcentajePantalla}%`;

                    // Dibujar recuadro de enfoque de movimiento
                    if (pixelesCambiados > 12 && maxX > minX && maxY > minY) {
                        const scaleX = trackingCanvas.width / analysisCanvas.width;
                        const scaleY = trackingCanvas.height / analysisCanvas.height;

                        const boxX = minX * scaleX;
                        const boxY = minY * scaleY;
                        const boxWidth = (maxX - minX) * scaleX;
                        const boxHeight = (maxY - minY) * scaleY;

                        trackingCtx.strokeStyle = '#ef4444';
                        trackingCtx.lineWidth = 2.5;
                        trackingCtx.shadowColor = '#ef4444';
                        trackingCtx.shadowBlur = 8;
                        trackingCtx.strokeRect(boxX, boxY, boxWidth, boxHeight);
                        trackingCtx.shadowBlur = 0;

                        trackingCtx.fillStyle = '#ef4444';
                        trackingCtx.font = "bold 10px Share Tech Mono";
                        trackingCtx.fillText(`IA LOCK: ${(ratioModificado * 100).toFixed(1)}%`, boxX, boxY - 6);
                    }

                    // Disparador del envío autónomo si sobrepasa el área crítica
                    if (ratioModificado > areaCriticaDeDisparo) {
                        triggerAutonomousCapture(porcentajePantalla);
                    }
                }
                fotoAnterior = pixeles;
            }
        }

        // 4. CAPTURA Y TRANSMISIÓN ULTRA RESILIENTE A TELEGRAM
        async function triggerAutonomousCapture(porcentajeModificado) {
            if (isInCooldown || !isEngineRunning) return;

            // Bloquear ráfagas innecesarias de fotos
            startCooldown();

            addLog(`Gatillo de movimiento detectado: ${porcentajeModificado}%`);
            playBeep(1100, 0.25);

            // Cambiar barra de progreso visual
            sendingSpinner.classList.remove('hidden');
            sendProgressBadge.innerText = "PROCESANDO FOTO...";
            sendProgressBadge.className = "text-[9px] bg-yellow-500/20 text-yellow-500 border border-yellow-500/30 px-1.5 py-0.5 rounded uppercase font-bold animate-pulse";

            // Congelar el fotograma actual
            let captureSource = isUsingRealCamera ? video : staticCanvas;
            const tempCanvas = document.createElement('canvas');
            tempCanvas.width = 640;
            tempCanvas.height = 480;
            const tempCtx = tempCanvas.getContext('2d');
            tempCtx.drawImage(captureSource, 0, 0, tempCanvas.width, tempCanvas.height);
            
            // Colocar marca de agua de SentryCam
            tempCtx.font = "bold 14px Share Tech Mono";
            tempCtx.fillStyle = "rgba(220, 38, 38, 0.9)";
            tempCtx.fillText(`SENTRYCAM SECURE - ALERTA AUTÓNOMA`, 20, 35);
            tempCtx.fillStyle = "rgba(255, 255, 255, 0.7)";
            tempCtx.fillText(`FECHA: ${new Date().toLocaleDateString()}`, 20, 55);
            tempCtx.fillText(`HORA: ${new Date().toLocaleTimeString()}`, 20, 75);
            tempCtx.fillText(`DIFERENCIA SENSOR: ${porcentajeModificado}%`, 20, 95);

            // Mostrar previsualización
            const dataUrl = tempCanvas.toDataURL('image/jpeg');
            lastCapturedPreview.src = dataUrl;
            lastCapturedPreview.classList.remove('hidden');
            noCapturePlaceholder.classList.add('hidden');

            const captionText = `🚨 *SENTRYCAM - MOVIMIENTO DETECTADO*\n\n*Fecha:* ${new Date().toLocaleDateString()}\n*Hora:* ${new Date().toLocaleTimeString()}\n*Sensor:* ${porcentajeModificado}%\n\n_Vigilancia en vivo de forma autónoma._`;

            // Envío a Telegram
            if (telegramConfig.token && telegramConfig.chatId) {
                const blob = dataURLtoBlob(dataUrl);
                if (!blob) {
                    addLog("Fallo de memoria al procesar imagen. Usando respaldo de texto...");
                    await sendTelegramFallbackMessage(captionText);
                    sendingSpinner.classList.add('hidden');
                    return;
                }

                const formData = new FormData();
                formData.append('chat_id', telegramConfig.chatId);
                formData.append('caption', captionText);
                formData.append('parse_mode', 'Markdown');
                formData.append('photo', blob, 'alerta_cam.jpg');

                try {
                    const response = await fetch(`https://api.telegram.org/bot${telegramConfig.token}/sendPhoto`, {
                        method: 'POST',
                        body: formData
                    });

                    const resJson = await response.json();

                    if (response.ok && resJson.ok) {
                        sendProgressBadge.innerText = "FOTO TRANSMITIDA";
                        sendProgressBadge.className = "text-[9px] bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 px-1.5 py-0.5 rounded uppercase font-bold";
                        addLog("Captura de foto transmitida de forma exitosa a Telegram.");
                    } else {
                        const errorMsg = resJson.description || "Error de red";
                        throw new Error(errorMsg);
                    }
                } catch (err) {
                    console.error("Fallo de subida de imagen:", err);
                    addLog(`Error en foto: ${err.message}. Intentando respaldo de texto...`);
                    
                    // Fallback inmediato a texto si la foto falla
                    const textSent = await sendTelegramFallbackMessage(captionText);
                    if (textSent) {
                        sendProgressBadge.innerText = "TEXTO ENVIADO";
                        sendProgressBadge.className = "text-[9px] bg-yellow-500/20 text-yellow-500 border border-yellow-500/30 px-1.5 py-0.5 rounded uppercase font-bold";
                    } else {
                        sendProgressBadge.innerText = "FALLO DE RED";
                        sendProgressBadge.className = "text-[9px] bg-red-500/20 text-red-400 border border-red-500/30 px-1.5 py-0.5 rounded uppercase font-bold";
                    }
                } finally {
                    sendingSpinner.classList.add('hidden');
                }
            } else {
                sendingSpinner.classList.add('hidden');
                sendProgressBadge.innerText = "SIN CONFIGURAR";
                sendProgressBadge.className = "text-[9px] bg-zinc-850 text-zinc-400 border border-zinc-700 px-1.5 py-0.5 rounded uppercase font-bold";
                addLog("Alerta visualizada localmente (Telegram sin configurar).");
            }
        }

        // Respaldo por texto si falla la transmisión de imagen pesada en redes lentas
        async function sendTelegramFallbackMessage(text) {
            if (!telegramConfig.token || !telegramConfig.chatId) return false;
            try {
                const response = await fetch(`https://api.telegram.org/bot${telegramConfig.token}/sendMessage`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        chat_id: telegramConfig.chatId,
                        text: text,
                        parse_mode: 'Markdown'
                    })
                });
                const resJson = await response.json();
                if (response.ok && resJson.ok) {
                    addLog("Reporte de respaldo por texto enviado a Telegram.");
                    return true;
                } else {
                    addLog(`Error en Telegram: "${resJson.description}". (¿Le diste /start en Telegram?)`);
                    return false;
                }
            } catch (err) {
                console.error(err);
                addLog("Error de red crítico. Revisa la conectividad WiFi/Datos.");
                return false;
            }
        }

        // TEMPORIZADOR DE COOLDOWN DE REPORTES
        function startCooldown() {
            isInCooldown = true;
            cooldownTimeRemaining = parseInt(cooldownSlider.value);
            cooldownSeconds.innerText = cooldownTimeRemaining;
            cooldownIndicator.classList.remove('hidden');

            cooldownInterval = setInterval(() => {
                cooldownTimeRemaining--;
                cooldownSeconds.innerText = cooldownTimeRemaining;

                if (cooldownTimeRemaining <= 0) {
                    clearInterval(cooldownInterval);
                    isInCooldown = false;
                    cooldownIndicator.classList.add('hidden');
                    addLog("Sensor listo para nueva captura autónoma.");
                }
            }, 1000);
        }

        // Bucle infinito
        function mainLoop() {
            renderSimulatedFeed();
            runAutonomousAnalysis();
            requestAnimationFrame(mainLoop);
        }

        // REGISTRO DE BITÁCORA
        function addLog(message) {
            if (noLogsMsg) {
                noLogsMsg.classList.add('hidden');
            }

            const now = new Date();
            const timeStr = now.toLocaleTimeString();

            const logItem = document.createElement('div');
            logItem.className = "flex items-center justify-between text-[11px] bg-zinc-950/80 border border-zinc-800/60 p-2 rounded-lg text-zinc-300 mono-font hover:bg-zinc-900 transition-colors animate-fade-in";
            logItem.innerHTML = `
                <div class="flex items-center gap-1.5">
                    <span class="w-1.5 h-1.5 bg-red-500 rounded-full animate-ping"></span>
                    <span class="text-zinc-500">${timeStr}</span>
                    <span class="text-left flex-grow pl-2">${message}</span>
                </div>
            `;

            logsContainer.insertBefore(logItem, logsContainer.firstChild);
            totalAlerts++;
            totalAlertsVal.innerText = `${totalAlerts} Eventos`;
        }

        // VALIDACIÓN Y GUARDADO DE CONFIGURACIÓN
        saveTelegramConfigBtn.addEventListener('click', async () => {
            const chatIdVal = telegramChatId.value.trim();

            if (!chatIdVal) {
                alert("Por favor, ingresa tu Chat ID personal para poder sincronizar.");
                return;
            }

            if (!checkStart.checked || !checkId.checked) {
                alert("Por favor, lee y marca las casillas de verificación del asistente para confirmar que seguiste los pasos necesarios en Telegram.");
                return;
            }

            // Guardar configuración en memoria y en LocalStorage para persistencia
            telegramConfig.chatId = chatIdVal;
            localStorage.setItem('sentry_telegram_chat_id', chatIdVal);

            // Actualizar interfaz visual
            telegramWarningBanner.classList.add('hidden');
            telegramStatusLabel.innerText = "BOT CONFIGURADO Y CONECTADO";
            telegramStatusLabel.className = "text-emerald-400 font-bold";

            addLog("Módulo de Telegram conectado e ID guardado.");
            playBeep(800, 0.15);

            // Enviar mensaje de prueba inmediato
            const sent = await sendTelegramFallbackMessage("🔔 *SentryCam Autónomo:* Sincronización realizada con éxito. Tu teléfono ya está reportando alertas de seguridad aquí.");
            if (sent) {
                alert("¡Sincronización exitosa! Revisa tu Telegram, acabas de recibir un mensaje de confirmación.");
            } else {
                alert("No se pudo enviar el mensaje de prueba. Asegúrate de haber buscado a tu bot en Telegram y presionar el botón 'INICIAR' (/start) primero.");
            }
        });

        // Evento de clic inicial de la pantalla de bienvenida
        btnStartEngine.addEventListener('click', () => {
            initAudio();
            startOverlay.style.opacity = '0';
            setTimeout(() => {
                startOverlay.classList.add('hidden');
            }, 500);

            // Intentar abrir la cámara inmediatamente tras interactuar
            setVideoSource(true);
        });

        // Botón forzado de cámara trasera
        forceCameraBtn.addEventListener('click', () => {
            initAudio();
            setVideoSource(true);
        });

        // Detener / Reanudar el escáner de IA
        toggleEngineBtn.addEventListener('click', () => {
            isEngineRunning = !isEngineRunning;
            initAudio();
            if (isEngineRunning) {
                toggleEngineBtn.className = "px-3 py-1.5 bg-red-600 hover:bg-red-700 text-white text-xs font-bold rounded-lg shadow-lg flex items-center gap-1.5 transition-all";
                toggleEngineText.innerText = "Pausar Centinela";
                statusBadge.className = "flex items-center gap-2 bg-red-600/25 border border-red-500/40 text-red-400 px-3 py-1.5 rounded-full text-xs font-bold backdrop-blur-md shadow-md";
                statusText.innerText = "MODO AUTÓNOMO ACTIVO";
                monitorContainer.classList.add('scanning-active');
                engineStatusText.innerText = isUsingRealCamera ? "MONITOREO REAL" : "MUESTRA SIMULADA";
                playBeep(900, 0.15);
            } else {
                toggleEngineBtn.className = "px-3 py-1.5 bg-zinc-850 hover:bg-zinc-700 text-zinc-300 text-xs font-bold rounded-lg shadow-lg flex items-center gap-1.5 transition-all";
                toggleEngineText.innerText = "Reanudar Centinela";
                statusBadge.className = "flex items-center gap-2 bg-zinc-900/80 border border-zinc-800 text-zinc-500 px-3 py-1.5 rounded-full text-xs font-bold backdrop-blur-md shadow-md";
                statusText.innerText = "ESCÁNER DETENIDO";
                monitorContainer.classList.remove('scanning-active');
                engineStatusText.innerText = "PAUSADO";
                playBeep(300, 0.2);
            }
        });

        sliderSensibilidad.addEventListener('input', () => {
            sliderSensVal.innerText = sliderSensibilidad.value;
        });

        cooldownSlider.addEventListener('input', () => {
            cooldownText.innerText = `${cooldownSlider.value} segundos`;
        });

        sourceRealBtn.addEventListener('click', () => { initAudio(); setVideoSource(true); });
        sourceSimBtn.addEventListener('click', () => { initAudio(); setVideoSource(false); });

        soundToggle.addEventListener('click', () => {
            initAudio();
            isAudioOn = !isAudioOn;
            if (isAudioOn) {
                soundToggle.className = "w-10 h-5 bg-red-600 rounded-full relative p-0.5 transition-colors duration-200 focus:outline-none";
                soundToggleDot.className = "w-4 h-4 bg-white rounded-full transform translate-x-5 transition-transform duration-200";
                playBeep(800, 0.1);
            } else {
                soundToggle.className = "w-10 h-5 bg-zinc-800 rounded-full relative p-0.5 transition-colors duration-200 focus:outline-none";
                soundToggleDot.className = "w-4 h-4 bg-zinc-400 rounded-full transform translate-x-0 transition-transform duration-200";
            }
        });

        // Pantalla completa
        fullscreenBtn.addEventListener('click', () => {
            if (!document.fullscreenElement) {
                monitorContainer.requestFullscreen();
            } else {
                document.exitFullscreen();
            }
        });

        // Inicialización
        window.onload = function() {
            loadSavedCredentials();
            setVideoSource(false); // Iniciar con el simulador de respaldo por defecto
            mainLoop();
        };
    </script>
</body>
</html>

```
