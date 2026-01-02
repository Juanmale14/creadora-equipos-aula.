<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Herramienta para la creación de equipos en el aula</title>
    <!-- Tailwind CSS para estilos rápidos y profesionales -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Iconos Phosphor -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    <!-- SheetJS para leer Excel -->
    <script src="https://cdn.sheetjs.com/xlsx-0.20.1/package/dist/xlsx.full.min.js"></script>
    <!-- Fuentes de Google -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; }
        
        /* Estilos para arrastrar y soltar */
        .draggable-source { cursor: grab; }
        .draggable-source:active { cursor: grabbing; }
        .drop-zone { min-height: 150px; transition: all 0.2s; }
        .drop-zone.drag-over { background-color: #e0f2fe; border-color: #3b82f6; border-style: dashed; }
        
        /* Colores de tipología de alumno */
        .student-helper { background-color: #dcfce7; border: 1px solid #86efac; color: #14532d; } /* Verde */
        .student-autonomous { background-color: #e0f2fe; border: 1px solid #93c5fd; color: #1e3a8a; } /* Azul */
        .student-needs-help { background-color: #ffedd5; border: 1px solid #fdba74; color: #7c2d12; } /* Naranja */

        /* Grid del aula */
        .desk-group {
            background: white;
            border: 2px solid #e5e7eb;
            border-radius: 12px;
            padding: 12px;
            min-height: 80px; /* Reducido para parecer más una mesa */
            display: flex;
            flex-direction: row; /* Fuerza lado a lado */
            flex-wrap: wrap; /* Permite salto si son muchos, pero intentará lado a lado primero */
            align-content: center;
            justify-content: center;
            gap: 10px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            position: relative;
        }
        .desk-group::before {
            content: attr(data-label);
            position: absolute;
            top: -22px;
            left: 0;
            width: 100%;
            text-align: center;
            font-weight: bold;
            color: #6b7280;
            font-size: 0.75rem;
            text-transform: uppercase;
        }

        /* Estilos de Impresión Profesional */
        @media print {
            /* Ocultar todo lo de la app web */
            body > *:not(#print-area) { display: none !important; }
            /* Mostrar solo el área de impresión */
            #print-area { display: block !important; position: absolute; top: 0; left: 0; width: 100%; }
            body { background: white; margin: 0; padding: 0; }
            
            /* PÁGINA 1: PORTADA */
            .print-page-cover {
                height: 100vh;
                width: 100%;
                page-break-after: always;
                break-after: page; 
                display: flex;
                flex-direction: column;
                justify-content: center;
                align-items: center;
                position: relative;
            }

            /* PÁGINA 2: MAPA */
            .print-page-map {
                height: 100vh;
                width: 100%;
                page-break-before: always;
                break-before: page;
                padding: 40px;
                box-sizing: border-box;
            }

            /* Ajustes del mapa para impresión */
            .desk-group { 
                border: 2px solid #000 !important; 
                box-shadow: none !important;
                break-inside: avoid;
                margin-bottom: 20px;
            }
            /* Forzar colores de fondo en impresión */
            .student-helper { background-color: #dcfce7 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            .student-autonomous { background-color: #e0f2fe !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            .student-needs-help { background-color: #ffedd5 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            
            /* Ajuste de Grid para impresión */
            #print-map-container {
                width: 100%;
                max-height: 90vh;
                overflow: visible;
                display: flex;
                justify-content: center;
                align-items: flex-start;
            }
            #print-map-container > div {
                display: grid !important;
                gap: 20px !important;
                width: 100%;
                zoom: 0.85; 
            }
        }

        #print-area { display: none; }
        
        /* Modal de Instrucciones */
        dialog::backdrop {
            background-color: rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(2px);
        }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <!-- Encabezado -->
    <header class="bg-white border-b border-gray-200 shadow-sm z-10 no-print">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-16">
                <div class="flex items-center gap-3">
                    <i class="ph ph-users-three text-3xl text-indigo-600"></i>
                    <h1 class="text-xl font-bold text-gray-900 tracking-tight">Herramienta para la creación de equipos en el aula</h1>
                </div>
            </div>
        </div>
    </header>

    <!-- Navegación de Pestañas -->
    <nav class="bg-indigo-700 text-white shadow-md no-print">
        <div class="max-w-7xl mx-auto px-4">
            <div class="flex space-x-4">
                <button onclick="switchTab('tab-datos')" class="tab-btn px-4 py-3 font-medium hover:bg-indigo-600 border-b-4 border-transparent focus:outline-none transition-colors active-tab" id="btn-tab-datos">
                    1. DATOS
                </button>
                <button onclick="switchTab('tab-config')" class="tab-btn px-4 py-3 font-medium hover:bg-indigo-600 border-b-4 border-transparent focus:outline-none transition-colors" id="btn-tab-config">
                    2. CONFIGURACIÓN
                </button>
                <button onclick="switchTab('tab-aula')" class="tab-btn px-4 py-3 font-medium hover:bg-indigo-600 border-b-4 border-transparent focus:outline-none transition-colors" id="btn-tab-aula">
                    3. AULA
                </button>
                <button onclick="switchTab('tab-export')" class="tab-btn px-4 py-3 font-medium hover:bg-indigo-600 border-b-4 border-transparent focus:outline-none transition-colors" id="btn-tab-export">
                    4. EXPORTACIÓN
                </button>
            </div>
        </div>
    </nav>

    <!-- Contenido Principal -->
    <main class="flex-1 overflow-auto bg-gray-50 p-6 relative">
        
        <!-- PESTAÑA 1: DATOS -->
        <div id="tab-datos" class="tab-content max-w-4xl mx-auto space-y-6">
            <div class="bg-white rounded-xl shadow p-6">
                <h2 class="text-lg font-semibold text-gray-800 mb-4 border-b pb-2 flex items-center gap-2"><i class="ph ph-identification-card"></i> Información del Grupo</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Grupo</label>
                        <select id="select-grupo" class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 p-2 border">
                            <option value="">Selecciona un grupo...</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Asignatura</label>
                        <select id="select-asignatura" class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 p-2 border">
                            <option value="">Selecciona una asignatura...</option>
                            <option>Biología</option><option>Conocimiento del medio</option><option>Economía</option><option>Educación Física</option>
                            <option>Educación Plástica y Visual</option><option>Filosofía</option><option>Francés</option>
                            <option>Geografía e Historia</option><option>Inglés</option><option>Latín</option>
                            <option>Lengua</option><option>Matemáticas</option><option>Música</option>
                            <option>Religión</option><option>Tecnología</option><option>Tutoría</option><option>Otra</option>
                        </select>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow p-6">
                <h2 class="text-lg font-semibold text-gray-800 mb-4 border-b pb-2 flex items-center gap-2"><i class="ph ph-users"></i> Listado de Alumnado</h2>
                <div class="border-2 border-dashed border-gray-300 rounded-lg p-6 text-center hover:bg-gray-50 transition-colors" id="drop-area">
                    <i class="ph ph-file-arrow-up text-4xl text-gray-400 mb-2"></i>
                    <p class="text-gray-600 mb-2 font-medium">Arrastra aquí un archivo EXCEL (.xlsx, .xls) o de texto (.txt, .csv)</p>
                    <input type="file" id="file-input" class="hidden" accept=".csv,.txt,.xlsx,.xls">
                    <button onclick="document.getElementById('file-input').click()" class="text-indigo-600 font-semibold hover:underline">Seleccionar archivo</button>
                    <p class="text-xs text-gray-400 mt-2">La aplicación leerá automáticamente la primera columna.</p>
                </div>
                <div class="mt-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">O pega la lista de nombres aquí:</label>
                    <textarea id="manual-names" rows="5" class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring focus:ring-indigo-200 p-2 border" placeholder="Juan García&#10;María López&#10;..."></textarea>
                </div>
                <div class="mt-4 flex gap-3 flex-wrap">
                    <button onclick="loadExampleData()" class="text-sm bg-gray-100 hover:bg-gray-200 text-gray-700 py-2 px-4 rounded border border-gray-300 transition">
                        <i class="ph ph-magic-wand"></i> Cargar ejemplo de prueba
                    </button>
                    <button onclick="document.getElementById('instructions-modal').showModal()" class="text-sm bg-blue-50 hover:bg-blue-100 text-blue-700 py-2 px-4 rounded border border-blue-200 transition">
                        <i class="ph ph-info"></i> Instrucciones de uso
                    </button>
                </div>
            </div>

            <div class="flex justify-end pt-4">
                <button onclick="saveData()" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-lg shadow-lg transform transition hover:scale-105 flex items-center gap-2">
                    <i class="ph ph-floppy-disk"></i> Guardar Datos
                </button>
            </div>
        </div>

        <!-- PESTAÑA 2: CONFIGURACIÓN -->
        <div id="tab-config" class="hidden tab-content max-w-7xl mx-auto h-full flex flex-col">
            <div class="bg-white p-4 rounded-lg shadow mb-4 flex justify-between items-center shrink-0">
                <h2 class="text-lg font-bold text-gray-800">Tipología de Alumnas/os</h2>
                <p class="text-sm text-gray-500">Arrastra a los alumnos a su categoría correspondiente.</p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 flex-1 min-h-0">
                <!-- Helper -->
                <div class="flex flex-col bg-green-50 rounded-xl border border-green-200 shadow-sm overflow-hidden h-full">
                    <div class="bg-green-100 p-3 border-b border-green-200 font-bold text-green-800 flex justify-between items-center">
                        <span><i class="ph ph-hand-heart mr-1"></i> Capacidad de Ayudar</span>
                        <span id="count-helper" class="bg-white px-2 py-0.5 rounded text-xs font-bold shadow-sm">0</span>
                    </div>
                    <div id="zone-helper" class="p-3 overflow-y-auto flex-1 drop-zone space-y-2" ondrop="drop(event, 'helper')" ondragover="allowDrop(event)"></div>
                </div>
                <!-- Autonomous -->
                <div class="flex flex-col bg-blue-50 rounded-xl border border-blue-200 shadow-sm overflow-hidden h-full">
                    <div class="bg-blue-100 p-3 border-b border-blue-200 font-bold text-blue-800 flex justify-between items-center">
                        <span><i class="ph ph-brain mr-1"></i> Alumnado Autónomo</span>
                        <span id="count-autonomous" class="bg-white px-2 py-0.5 rounded text-xs font-bold shadow-sm">0</span>
                    </div>
                    <div id="zone-autonomous" class="p-3 overflow-y-auto flex-1 drop-zone space-y-2" ondrop="drop(event, 'autonomous')" ondragover="allowDrop(event)"></div>
                </div>
                <!-- Needs Help -->
                <div class="flex flex-col bg-orange-50 rounded-xl border border-orange-200 shadow-sm overflow-hidden h-full">
                    <div class="bg-orange-100 p-3 border-b border-orange-200 font-bold text-orange-800 flex justify-between items-center">
                        <span><i class="ph ph-lifebuoy mr-1"></i> Necesita Ayuda</span>
                        <span id="count-needs-help" class="bg-white px-2 py-0.5 rounded text-xs font-bold shadow-sm">0</span>
                    </div>
                    <div id="zone-needs-help" class="p-3 overflow-y-auto flex-1 drop-zone space-y-2" ondrop="drop(event, 'needs-help')" ondragover="allowDrop(event)"></div>
                </div>
            </div>
            
            <div class="mt-4 flex justify-end shrink-0">
                 <button onclick="switchTab('tab-aula')" class="bg-indigo-600 text-white px-6 py-2 rounded-lg hover:bg-indigo-700 transition">Siguiente: Aula <i class="ph ph-arrow-right"></i></button>
            </div>
        </div>

        <!-- PESTAÑA 3: AULA -->
        <div id="tab-aula" class="hidden tab-content max-w-full h-full flex flex-col items-center">
            <!-- Controles -->
            <div class="w-full bg-white rounded-xl shadow p-4 mb-4 shrink-0 no-print max-w-7xl">
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 items-end">
                    <div>
                        <label class="block text-sm font-semibold text-gray-700 mb-2">Agrupamiento</label>
                        <div class="flex gap-4">
                            <label class="inline-flex items-center cursor-pointer">
                                <input type="radio" name="agrupamiento" value="heterogeneo" checked class="form-radio text-indigo-600">
                                <span class="ml-2 text-sm">Heterogéneo</span>
                            </label>
                            <label class="inline-flex items-center cursor-pointer">
                                <input type="radio" name="agrupamiento" value="homogeneo" class="form-radio text-indigo-600">
                                <span class="ml-2 text-sm">Homogéneo</span>
                            </label>
                        </div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold text-gray-700 mb-1">Integrantes por mesa</label>
                        <select id="select-espacio" class="w-full rounded border-gray-300 border p-2 text-sm">
                            <option value="2">2</option>
                            <option value="3">3</option>
                            <option value="4" selected>4</option>
                            <option value="5">5</option>
                            <option value="6">6</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold text-gray-700 mb-1">Filas del Aula (Grid)</label>
                        <select id="select-grid-cols" class="w-full rounded border-gray-300 border p-2 text-sm" onchange="updateGridColumns()">
                            <option value="2">2 Grupos de ancho</option>
                            <option value="3" selected>3 Grupos de ancho</option>
                            <option value="4">4 Grupos de ancho</option>
                            <option value="5">5 Grupos de ancho</option>
                            <option value="6">6 Grupos de ancho</option>
                            <option value="7">7 Grupos de ancho</option>
                            <option value="8">8 Grupos de ancho</option>
                        </select>
                    </div>
                    <div class="flex gap-2">
                        <button onclick="distributeStudents()" class="flex-1 bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg shadow text-sm">
                            <i class="ph ph-shuffle"></i> DISTRIBUIR
                        </button>
                        <button onclick="saveDistribution()" class="bg-gray-800 hover:bg-gray-900 text-white font-medium py-2 px-4 rounded-lg shadow text-sm">
                            <i class="ph ph-floppy-disk"></i>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Mapa del Aula - Contenedor Rectangular -->
            <div id="aula-container" class="w-full max-w-6xl aspect-[4/3] bg-gray-200 rounded-xl border-8 border-gray-300 overflow-auto relative p-8 shadow-inner mx-auto">
                <div class="w-64 mx-auto bg-gray-800 text-white text-center py-1 rounded-b-lg shadow-md mb-8 text-sm font-bold uppercase tracking-widest no-print">
                    PIZARRA
                </div>
                
                <div id="aula-map" class="grid gap-8 justify-center pb-20 transition-all duration-300">
                    <div class="col-span-full text-center text-gray-500 mt-20">
                        <i class="ph ph-chalkboard-teacher text-6xl mb-2 opacity-50"></i>
                        <p>Configura las opciones y pulsa "Distribuir".</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- PESTAÑA 4: EXPORTACIÓN -->
        <div id="tab-export" class="hidden tab-content max-w-4xl mx-auto space-y-6">
            <div class="bg-white rounded-xl shadow-lg p-8 text-center">
                <div class="mb-6">
                    <i class="ph ph-file-pdf text-6xl text-red-500"></i>
                </div>
                <h2 class="text-2xl font-bold text-gray-800 mb-2">Informe Profesional</h2>
                <p class="text-gray-600 mb-8">Genera un PDF de <strong>dos páginas</strong> (Portada + Mapa) con la distribución del aula.</p>
                
                <div class="flex flex-col sm:flex-row justify-center gap-4">
                    <button onclick="printReport()" class="bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-8 rounded-lg shadow-lg flex items-center justify-center gap-2 transition hover:-translate-y-1">
                        <i class="ph ph-printer"></i> Generar PDF (2 Páginas)
                    </button>
                    
                    <button onclick="resetApp()" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-3 px-8 rounded-lg shadow-lg flex items-center justify-center gap-2 transition">
                        <i class="ph ph-trash"></i> Reiniciar Aplicación
                    </button>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow p-6">
                <h3 class="font-bold text-gray-800 mb-4">Vista Previa de Datos</h3>
                <div id="export-preview" class="text-sm text-gray-600 space-y-2">
                    <p>No hay datos guardados aún.</p>
                </div>
            </div>
        </div>
    </main>

    <!-- Pie de página (Web) -->
    <footer class="bg-white border-t border-gray-200 py-3 text-center text-sm text-gray-600 no-print z-10">
        Autor: <a href="https://blogsaverroes.juntadeandalucia.es/ellocodelamochila/" target="_blank" class="text-indigo-600 hover:text-indigo-800 font-bold transition-colors">El loco de la mochila</a>
    </footer>

    <!-- MODAL DE INSTRUCCIONES -->
    <dialog id="instructions-modal" class="rounded-xl shadow-2xl p-0 w-full max-w-2xl backdrop:bg-gray-900/50">
        <div class="bg-indigo-600 text-white p-4 flex justify-between items-center">
            <h3 class="text-lg font-bold"><i class="ph ph-info mr-2"></i> Instrucciones de Uso</h3>
            <button onclick="document.getElementById('instructions-modal').close()" class="text-white hover:text-indigo-200">
                <i class="ph ph-x text-xl"></i>
            </button>
        </div>
        <div class="p-6 space-y-4 text-gray-700 overflow-y-auto max-h-[70vh]">
            <div class="flex gap-3">
                <div class="bg-indigo-100 text-indigo-700 w-8 h-8 rounded-full flex items-center justify-center font-bold shrink-0">1</div>
                <div>
                    <h4 class="font-bold text-gray-900">Datos</h4>
                    <p class="text-sm">Selecciona el grupo y la asignatura. Pega la lista de alumnos o arrastra un archivo Excel/CSV. Pulsa "Guardar Datos" para continuar.</p>
                </div>
            </div>
            <div class="flex gap-3">
                <div class="bg-indigo-100 text-indigo-700 w-8 h-8 rounded-full flex items-center justify-center font-bold shrink-0">2</div>
                <div>
                    <h4 class="font-bold text-gray-900">Configuración</h4>
                    <p class="text-sm">Clasifica a los alumnos arrastrándolos a las cajas: Ayuda (Verde), Autónomo (Azul) o Necesita Ayuda (Naranja).</p>
                </div>
            </div>
            <div class="flex gap-3">
                <div class="bg-indigo-100 text-indigo-700 w-8 h-8 rounded-full flex items-center justify-center font-bold shrink-0">3</div>
                <div>
                    <h4 class="font-bold text-gray-900">Aula</h4>
                    <p class="text-sm">Elige cómo quieres agruparlos (Heterogéneo/Homogéneo), el tamaño de los grupos y el ancho del aula. Pulsa "Distribuir" para generar el mapa.</p>
                </div>
            </div>
            <div class="flex gap-3">
                <div class="bg-indigo-100 text-indigo-700 w-8 h-8 rounded-full flex items-center justify-center font-bold shrink-0">4</div>
                <div>
                    <h4 class="font-bold text-gray-900">Exportación</h4>
                    <p class="text-sm">Genera un PDF profesional con la portada y el mapa de la clase lista para imprimir.</p>
                </div>
            </div>
        </div>
        <div class="bg-gray-50 p-4 flex justify-end">
            <button onclick="document.getElementById('instructions-modal').close()" class="bg-indigo-600 text-white px-4 py-2 rounded hover:bg-indigo-700">Entendido</button>
        </div>
    </dialog>

    <!-- ÁREA DE IMPRESIÓN (OCULTA EN PANTALLA) -->
    <div id="print-area">
        <!-- PÁGINA 1: PORTADA -->
        <div class="print-page-cover border-b-2 border-gray-300">
            <div class="mb-8">
                <i class="ph ph-student text-6xl text-gray-400"></i>
            </div>
            <h1 class="text-4xl font-bold text-black mb-2">INFORME DE DISTRIBUCIÓN DE AULA</h1>
            <h2 class="text-2xl text-gray-600 mb-12">Planificación de Grupos Cooperativos</h2>
            
            <div class="w-full max-w-2xl border-t border-b border-gray-800 py-8 my-8 text-left grid grid-cols-2 gap-8">
                <div>
                    <p class="text-sm text-gray-500 uppercase tracking-wide">Grupo</p>
                    <p class="text-xl font-bold" id="cover-group">-</p>
                </div>
                <div>
                    <p class="text-sm text-gray-500 uppercase tracking-wide">Asignatura</p>
                    <p class="text-xl font-bold" id="cover-subject">-</p>
                </div>
                <div>
                    <p class="text-sm text-gray-500 uppercase tracking-wide">Fecha</p>
                    <p class="text-xl font-bold" id="cover-date">-</p>
                </div>
                <div>
                    <p class="text-sm text-gray-500 uppercase tracking-wide">Total Alumnado</p>
                    <p class="text-xl font-bold" id="cover-total">-</p>
                </div>
            </div>

            <div class="mt-8 flex gap-6 justify-center text-sm">
                <div class="flex items-center gap-2">
                    <div class="w-4 h-4 bg-[#dcfce7] border border-green-500"></div> Capacidad de Ayudar
                </div>
                <div class="flex items-center gap-2">
                    <div class="w-4 h-4 bg-[#e0f2fe] border border-blue-500"></div> Autónomo
                </div>
                <div class="flex items-center gap-2">
                    <div class="w-4 h-4 bg-[#ffedd5] border border-orange-500"></div> Necesita Ayuda
                </div>
            </div>
        </div>
        
        <!-- PÁGINA 2: MAPA -->
        <div class="print-page-map">
            <h3 class="text-center font-bold text-xl mb-4 uppercase border-b pb-2">Mapa de Distribución</h3>
            <div class="w-full bg-black text-white text-center py-2 font-bold mb-8 uppercase text-sm">PIZARRA / PROFESOR</div>
            <div id="print-map-container">
                <!-- El mapa se clonará aquí -->
            </div>
        </div>
    </div>

    <!-- Toast -->
    <div id="toast" class="fixed bottom-5 right-5 bg-gray-900 text-white px-6 py-3 rounded-lg shadow-xl transform translate-y-20 transition-transform duration-300 flex items-center gap-3 z-50">
        <i class="ph ph-check-circle text-green-400 text-xl"></i>
        <span id="toast-msg">Acción completada</span>
    </div>

    <script>
        // --- ESTADO ---
        const appState = {
            group: '', subject: '',
            students: [],
            distribution: []
        };

        // --- INICIALIZACIÓN ---
        document.addEventListener('DOMContentLoaded', () => {
            populateGroups();
            updateCounts();
            setupDragAndDrop();
            updateGridColumns(); // Init grid
        });

        function setupDragAndDrop() {
            const dropArea = document.getElementById('drop-area');
            const fileInput = document.getElementById('file-input');
            ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(evt => {
                dropArea.addEventListener(evt, (e) => { e.preventDefault(); e.stopPropagation(); });
            });
            dropArea.addEventListener('drop', handleDropFile);
            fileInput.addEventListener('change', (e) => handleFiles(e.target.files));
        }

        function populateGroups() {
            const select = document.getElementById('select-grupo');
            // Añadidos niveles de Primaria
            const levels = [
                '1º Primaria', '2º Primaria', '3º Primaria', '4º Primaria', '5º Primaria', '6º Primaria',
                '1º ESO', '2º ESO', '3º ESO', '4º ESO', 
                '1º BACH', '2º BACH'
            ];
            const letters = ['A', 'B', 'C', 'D'];
            levels.forEach(l => letters.forEach(le => {
                const opt = document.createElement('option');
                opt.value = `${l} ${le}`;
                opt.textContent = `${l} ${le}`;
                select.appendChild(opt);
            }));
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.tab-btn').forEach(el => {
                el.classList.remove('border-indigo-400', 'text-indigo-100');
                el.classList.add('border-transparent');
            });
            document.getElementById(tabId).classList.remove('hidden');
            const btn = document.getElementById('btn-' + tabId);
            if(btn) {
                btn.classList.remove('border-transparent');
                btn.classList.add('border-indigo-200', 'bg-indigo-800');
            }
            if(tabId === 'tab-export') updateExportPreview();
        }

        // --- DATOS ---
        function loadExampleData() {
            document.getElementById('select-grupo').value = "5º Primaria A";
            document.getElementById('select-asignatura').value = "Matemáticas";
            const names = ["Alejandro R.", "Beatriz L.", "Carlos M.", "Diana P.", "Elena N.", "Fernando A.", "Gabriela S.", "Hugo S.", "Inés A.", "Javier B.", "Karol G.", "Luis F.", "María C.", "Nacho V.", "Olga T.", "Pedro P.", "Quentin T.", "Rosa M.", "Sergio R.", "Teresa C.", "Ursula C.", "Víctor M.", "Wanda N.", "Xavi H."];
            document.getElementById('manual-names').value = names.join('\n');
            // IMPORTANTE: Guardar automáticamente para que aparezcan en Configuración
            saveData(true); 
        }

        function handleDropFile(e) { handleFiles(e.dataTransfer.files); }
        
        function handleFiles(files) {
            if (!files.length) return;
            const file = files[0];
            const reader = new FileReader();
            
            if (file.name.match(/\.(xlsx|xls)$/)) {
                reader.onload = (e) => {
                    try {
                        const data = new Uint8Array(e.target.result);
                        const wb = XLSX.read(data, {type: 'array'});
                        if (!wb.SheetNames.length) return alert("Excel vacío");
                        const ws = wb.Sheets[wb.SheetNames[0]];
                        const json = XLSX.utils.sheet_to_json(ws, {header: 1});
                        const names = json.map(r => r[0]).filter(n => n && String(n).trim());
                        
                        if(names.length) {
                            document.getElementById('manual-names').value = names.join('\n');
                            showToast(`Excel: ${names.length} nombres cargados.`);
                        } else alert("No se encontraron nombres en la 1ª columna");
                    } catch (err) { alert("Error leyendo Excel"); console.error(err); }
                };
                reader.readAsArrayBuffer(file);
            } else {
                reader.onload = (e) => {
                    document.getElementById('manual-names').value = e.target.result;
                    showToast(`Archivo de texto cargado`);
                };
                reader.readAsText(file);
            }
        }

        function saveData(isAuto = false) {
            const group = document.getElementById('select-grupo').value;
            const subject = document.getElementById('select-asignatura').value;
            const raw = document.getElementById('manual-names').value;
            
            if (!group || !subject || !raw.trim()) return alert("Faltan datos por rellenar");
            
            appState.group = group;
            appState.subject = subject;
            
            const oldMap = new Map(appState.students.map(s => [s.name, s.type]));
            const lines = raw.split('\n').filter(l => l.trim());
            
            appState.students = lines.map((name, i) => {
                const clean = name.trim().replace(/,/g, '');
                return {
                    id: `s-${i}-${Date.now()}`,
                    name: clean,
                    type: oldMap.get(clean) || 'autonomous'
                };
            });
            
            renderConfigTab();
            if(isAuto) {
                showToast("Datos de ejemplo cargados y procesados");
                // Pequeño delay para que el usuario vea que pasó algo antes de cambiar
                setTimeout(() => switchTab('tab-config'), 500);
            } else {
                showToast("Datos guardados correctamente");
                switchTab('tab-config');
            }
        }

        // --- KANBAN CONFIG ---
        function renderConfigTab() {
            ['helper', 'autonomous', 'needs-help'].forEach(type => {
                document.getElementById(`zone-${type}`).innerHTML = '';
            });
            appState.students.forEach(s => {
                const el = createCard(s);
                document.getElementById(`zone-${s.type}`).appendChild(el);
            });
            updateCounts();
        }

        function createCard(s) {
            const div = document.createElement('div');
            div.id = s.id;
            div.className = `p-2 mb-1 rounded shadow-sm bg-white border-l-4 text-sm font-medium flex justify-between items-center draggable-source cursor-grab hover:shadow-md`;
            
            if(s.type === 'helper') div.classList.add('border-green-500');
            else if(s.type === 'autonomous') div.classList.add('border-blue-500');
            else div.classList.add('border-orange-500');
            
            div.draggable = true;
            div.ondragstart = (e) => {
                e.dataTransfer.setData("text", s.id);
                e.dataTransfer.setData("origin", "config");
            };
            div.innerHTML = `<span>${s.name}</span><i class="ph-fill ph-dots-six-vertical text-gray-300"></i>`;
            return div;
        }

        function allowDrop(e) { e.preventDefault(); e.currentTarget.classList.add('drag-over'); }
        document.addEventListener('dragleave', e => {
            if(e.target.classList?.contains('drop-zone')) e.target.classList.remove('drag-over');
        });

        function drop(e, newType) {
            e.preventDefault();
            e.currentTarget.classList.remove('drag-over');
            const id = e.dataTransfer.getData("text");
            const el = document.getElementById(id);
            if(!el || e.currentTarget.contains(el)) return;
            
            e.currentTarget.appendChild(el);
            const stu = appState.students.find(s => s.id === id);
            if(stu) {
                stu.type = newType;
                el.className = el.className.replace(/border-(green|blue|orange)-500/g, '');
                if(newType === 'helper') el.classList.add('border-green-500');
                else if(newType === 'autonomous') el.classList.add('border-blue-500');
                else el.classList.add('border-orange-500');
            }
            updateCounts();
        }

        function updateCounts() {
            document.getElementById('count-helper').innerText = appState.students.filter(s => s.type === 'helper').length;
            document.getElementById('count-autonomous').innerText = appState.students.filter(s => s.type === 'autonomous').length;
            document.getElementById('count-needs-help').innerText = appState.students.filter(s => s.type === 'needs-help').length;
        }

        // --- AULA DISTRIBUCIÓN ---
        function updateGridColumns() {
            const cols = document.getElementById('select-grid-cols').value;
            const map = document.getElementById('aula-map');
            // Usamos grid-template-columns para controlar cuántos grupos caben
            map.style.gridTemplateColumns = `repeat(${cols}, minmax(0, 1fr))`;
        }

        function distributeStudents() {
            if(!appState.students.length) return alert("Sin alumnos");
            
            const groupSize = parseInt(document.getElementById('select-espacio').value);
            const mode = document.querySelector('input[name="agrupamiento"]:checked').value;
            
            let helpers = appState.students.filter(s => s.type === 'helper');
            let autonomous = appState.students.filter(s => s.type === 'autonomous');
            let needs = appState.students.filter(s => s.type === 'needs-help');
            
            [helpers, autonomous, needs].forEach(arr => arr.sort(() => Math.random() - .5));
            
            const numGroups = Math.ceil(appState.students.length / groupSize);
            let groups = Array.from({length: numGroups}, () => []);
            
            if(mode === 'heterogeneo') {
                let idx = 0;
                while(helpers.length) { groups[idx].push(helpers.pop()); idx = (idx + 1) % numGroups; }
                idx = 0;
                while(needs.length) { groups[idx].push(needs.pop()); idx = (idx + 1) % numGroups; }
                // Rellenar huecos primero en los grupos más pequeños
                while(autonomous.length) {
                    let minG = groups.reduce((m, g) => g.length < m.length ? g : m, groups[0]);
                    minG.push(autonomous.pop());
                }
            } else {
                let all = [...helpers, ...autonomous, ...needs];
                groups = [];
                while(all.length) groups.push(all.splice(0, groupSize));
            }
            
            appState.distribution = groups;
            renderClassroom();
        }

        function renderClassroom() {
            const map = document.getElementById('aula-map');
            map.innerHTML = '';
            
            appState.distribution.forEach((grp, i) => {
                const div = document.createElement('div');
                div.className = 'desk-group bg-white';
                div.dataset.gid = i;
                div.setAttribute('data-label', `Mesa ${i + 1}`);
                
                div.ondragover = e => { e.preventDefault(); div.style.backgroundColor = '#f0fdf4'; };
                div.ondragleave = e => { div.style.backgroundColor = 'white'; };
                div.ondrop = e => dropInDesk(e, i);
                
                grp.forEach(s => {
                    const badge = document.createElement('div');
                    badge.id = `seat-${s.id}`;
                    // AJUSTE CLAVE: Eliminado 'min-w' rígido y ajustado para que quepan lado a lado si es posible
                    badge.className = `px-2 py-1 text-xs font-bold rounded shadow-sm border cursor-move flex items-center justify-center select-none flex-1 min-w-[45%] h-auto`;
                    
                    if(s.type === 'helper') badge.classList.add('student-helper');
                    else if(s.type === 'autonomous') badge.classList.add('student-autonomous');
                    else badge.classList.add('student-needs-help');
                    
                    badge.innerHTML = `<span class="text-center w-full break-words leading-tight">${s.name}</span>`;
                    
                    badge.draggable = true;
                    badge.ondragstart = e => {
                        e.dataTransfer.setData("text", JSON.stringify(s));
                        e.dataTransfer.setData("origin", "classroom");
                    };
                    div.appendChild(badge);
                });
                map.appendChild(div);
            });
        }

        function dropInDesk(e, targetIdx) {
            e.preventDefault();
            e.currentTarget.style.backgroundColor = 'white';
            const origin = e.dataTransfer.getData("origin");
            if(origin !== 'classroom') return;
            
            const sData = JSON.parse(e.dataTransfer.getData("text"));
            let srcG = -1, srcI = -1;
            
            appState.distribution.forEach((g, gi) => {
                const si = g.findIndex(s => s.id === sData.id);
                if(si !== -1) { srcG = gi; srcI = si; }
            });
            
            if(srcG !== -1 && srcG !== targetIdx) {
                appState.distribution[targetIdx].push(appState.distribution[srcG].splice(srcI, 1)[0]);
                renderClassroom();
            }
        }

        function saveDistribution() { showToast("Guardado temporalmente"); switchTab('tab-export'); }

        // --- IMPRESIÓN ---
        function updateExportPreview() {
            document.getElementById('export-preview').innerHTML = `
                <p><strong>Total:</strong> ${appState.students.length} alumnos</p>
                <p><strong>Grupos:</strong> ${appState.distribution.length}</p>
            `;
        }

        function printReport() {
            if(!appState.distribution.length) return alert("Primero genera una distribución en la pestaña AULA");
            
            // Rellenar Portada
            document.getElementById('cover-group').innerText = appState.group || "Sin grupo";
            document.getElementById('cover-subject').innerText = appState.subject || "Sin asignatura";
            document.getElementById('cover-date').innerText = new Date().toLocaleDateString();
            document.getElementById('cover-total').innerText = appState.students.length;
            
            // Clonar Mapa
            const mapContainer = document.getElementById('print-map-container');
            const originalMap = document.getElementById('aula-map');
            mapContainer.innerHTML = '';
            
            const clone = originalMap.cloneNode(true);
            clone.style.gridTemplateColumns = originalMap.style.gridTemplateColumns;
            
            mapContainer.appendChild(clone);
            
            window.print();
        }

        function resetApp() {
            if(confirm("¿Estás seguro de que quieres borrar todos los datos y empezar de cero?")) {
                location.reload();
            }
        }

        function showToast(msg) {
            const t = document.getElementById('toast');
            document.getElementById('toast-msg').innerText = msg;
            t.classList.remove('translate-y-20');
            setTimeout(() => t.classList.add('translate-y-20'), 3000);
        }
    </script>
</body>
</html>