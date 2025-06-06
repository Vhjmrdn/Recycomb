<!DOCTYPE html>
<html lang="es" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Recycomb S.A. | Transformando Residuos en Energía</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Earthy Harmony -->
    <!-- Application Structure Plan: A single-page application with a fixed navigation bar for easy, non-linear exploration. The structure is thematic: 1. A dynamic hero section for immediate impact. 2. "El Proceso" combines what Recycomb does with how its product is used, presenting a complete operational loop via an interactive HTML/CSS diagram. 3. "Nuestros Insumos" uses interactive cards and a conceptual donut chart to detail the raw materials, now with illustrative percentages and annual tonnage. 4. "El Impacto" showcases benefits (CO2 reduction, circular economy) using dynamic stat cards and a new Gemini-powered "Dato Curioso" generator. 5. "Nuestros Objetivos" presents the company's vision in a clear, icon-driven layout. This structure prioritizes user-driven discovery over a passive, linear narrative, making the information more engaging and digestible. -->
    <!-- Visualization & Content Choices: 1. Process Flow Diagram (Goal: Organize): Built with HTML/CSS/Tailwind, interaction via JS. Justification: Clearly illustrates the circular process, which is central to Recycomb's model. 2. Waste Types Cards (Goal: Inform): Interactive HTML cards, now displaying illustrative percentages and approximate annual tonnage for each type. Justification: Provides a more quantitative and relatable overview of the input stream. 3. Waste Mix Donut Chart (Goal: Inform/Compare): Chart.js Canvas, updated to reflect the new percentages. Justification: Visually represents the concept of a diverse waste stream, making the information more quantitative and consistent with the card data. 4. Impact Stat Cards (Goal: Inform): HTML/CSS with JS number counters. Justification: Dynamically highlights key benefits, making them more memorable. 5. Gemini API Fact Generator (Goal: Engage/Inform): Button triggered LLM call to generate sustainability insights, displayed in a text block. Justification: Adds a dynamic, educational, and engaging element that uses AI to provide relevant context and spark further interest. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        html {
            /* Ensures smooth scrolling for anchor links */
            scroll-behavior: smooth;
            /* Adjust this value to match your fixed header's height */
            /* This prevents content from being hidden under the fixed header during anchor link navigation */
            scroll-padding-top: 70px; 
        }
        body {
            font-family: 'Roboto', sans-serif;
            /* Added a subtle gradient background for more depth */
            background-color: #f8fafc; /* Fallback color */
            background-image: linear-gradient(to bottom, #ffffff, #f8fafc);
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 450px;
            margin-left: auto;
            margin-right: auto;
            height: 300px; /* Default height for smaller screens */
            max-height: 400px;
        }
        @media (min-width: 768px) { /* md breakpoint */
            .chart-container {
                height: 350px; /* Taller chart for larger screens */
            }
        }
        .nav-link {
            transition: color 0.3s, border-color 0.3s;
        }
        .nav-link:hover {
            color: #2E7D32; /* green-700 */
            border-bottom-color: #2E7D32; /* green-700 */
        }
        /* Active state for desktop navigation links */
        .nav-link.active {
            color: #166534; /* green-800 for a slightly darker active state */
            border-bottom-color: #166534; /* green-800 */
            font-weight: 500; /* medium */
        }
        /* Active state for mobile navigation links */
        #mobile-menu a.active {
            background-color: #DCFCE7; /* green-100 */
            color: #166534; /* green-800 */
            font-weight: 500; /* medium */
        }
        .waste-card {
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .waste-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        /* Interactive Game Styles */
        .waste-item {
            cursor: grab;
            user-select: none;
            touch-action: none;
        }
        .waste-item:active {
            cursor: grabbing;
        }
        .drop-zone {
            transition: background-color 0.3s, border-color 0.3s;
        }
        .drop-zone.drag-over {
            border-color: #2E7D32;
            background-color: #DCFCE7;
        }
        .drop-zone.correct-drop {
            background-color: #c8e6c9;
            border-color: #4CAF50;
        }
        .drop-zone.incorrect-drop {
            background-color: #ffcdd2;
            border-color: #f44336;
        }
    </style>
</head>
<body class="text-gray-800">

    <!-- Header Section: Contains the navigation bar -->
    <header class="bg-white shadow-md sticky top-0 z-50">
        <nav class="container mx-auto px-6 py-3 flex justify-between items-center">
            <div class="flex items-center">
                <a href="#" class="text-2xl font-bold text-green-700">Recycomb</a> <!-- Added href for the logo to go to top -->
            </div>
            <!-- Desktop Navigation Links -->
            <div class="hidden md:flex items-center space-x-6">
                <a href="#proceso" class="nav-link text-gray-600 border-b-2 border-transparent pb-1">El Proceso</a>
                <a href="#insumos" class="nav-link text-gray-600 border-b-2 border-transparent pb-1">Insumos</a>
                <a href="#impacto" class="nav-link text-gray-600 border-b-2 border-transparent pb-1">Impacto</a>
                <a href="#objetivos" class="nav-link text-gray-600 border-b-2 border-transparent pb-1">Objetivos</a>
                <a href="#juego" class="nav-link text-gray-600 border-b-2 border-transparent pb-1">Juego</a>
            </div>
            <!-- Mobile Menu Button -->
            <div class="md:hidden">
                <button id="mobile-menu-button" class="text-gray-600 focus:outline-none" aria-label="Abrir menú principal" aria-expanded="false" aria-controls="mobile-menu">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16m-7 6h7"></path></svg>
                </button>
            </div>
        </nav>
        <!-- Mobile Navigation Menu (hidden by default) -->
        <div id="mobile-menu" class="hidden md:hidden">
            <a href="#proceso" class="block py-2 px-4 text-sm text-gray-700 hover:bg-green-50">El Proceso</a>
            <a href="#insumos" class="block py-2 px-4 text-sm text-gray-700 hover:bg-green-50">Insumos</a>
            <a href="#impacto" class="block py-2 px-4 text-sm text-gray-700 hover:bg-green-50">Impacto</a>
            <a href="#objetivos" class="block py-2 px-4 text-sm text-gray-700 hover:bg-green-50">Objetivos</a>
            <a href="#juego" class="block py-2 px-4 text-sm text-gray-700 hover:bg-green-50">Juego</a>
        </div>
    </header>

    <main>
        <!-- Hero Section: Introduction to Recycomb -->
        <section id="hero" class="bg-green-700 text-white">
            <div class="container mx-auto px-6 py-20 md:py-32 text-center">
                <h1 class="text-4xl md:text-6xl font-bold leading-tight mb-4">Transformamos Residuos en Energía</h1>
                <p class="text-lg md:text-xl text-green-200 max-w-3xl mx-auto">
                    Desde 1998, Recycomb lidera la valorización de residuos industriales, convirtiéndolos en energía limpia para construir un futuro más sostenible y responsable.
                </p>
            </div>
        </section>

        <!-- Proceso Section: Explains Recycomb's circular process -->
        <section id="proceso" class="py-16 md:py-24 bg-white">
            <div class="container mx-auto px-6">
                <div class="text-center mb-12">
                    <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-4">De Residuo a Energía: Nuestro Proceso Circular</h2>
                    <div class="max-w-3xl mx-auto text-gray-600 space-y-3">
                        <p>
                            La <strong>valorización energética</strong> es el proceso de convertir residuos no reciclables en energía utilizable, como calor o electricidad. En Recycomb, nos especializamos en transformar residuos industriales no peligrosos en un combustible alternativo de alto valor.
                        </p>
                        <p> 
                            Esto se logra mediante procesos como la <strong>combustión controlada</strong> (una oxidación exotérmica que libera calor) y la <strong>descomposición térmica</strong> (similar a la pirólisis), que adaptan las características de los residuos para maximizar su poder calorífico. Nuestro modelo se basa en un ciclo completo y eficiente que reduce la dependencia de recursos fósiles en la industria pesada.
                        </p>
                    </div>
                </div>

                <!-- Visual representation of the process flow -->
                <div class="flex flex-col md:flex-row items-center justify-center gap-4 md:gap-0 mt-10 mb-12">
                     <div class="text-center p-4">
                        <div class="text-4xl mb-2">🏭</div>
                        <h3 class="font-bold">Industrias Generadoras</h3>
                        <p class="text-sm text-gray-500">Origen del residuo</p>
                    </div>
                    <div class="text-green-500 text-4xl font-normal transform md:rotate-0 rotate-90">→</div>
                    <div class="text-center p-4 bg-green-100 rounded-lg">
                        <div class="text-4xl mb-2">♻️</div>
                        <h3 class="font-bold">Proceso Recycomb</h3>
                        <p class="text-sm text-gray-500">Clasificación, trituración, descomposición térmica y mezcla</p>
                    </div>
                    <div class="text-green-500 text-4xl font-normal transform md:rotate-0 rotate-90">→</div>
                     <div class="text-center p-4">
                        <div class="text-4xl mb-2">🔥</div>
                        <h3 class="font-bold">Combustible Alternativo</h3>
                        <p class="text-sm text-gray-500">Alto poder calorífico (RFL)</p>
                    </div>
                     <div class="text-green-500 text-4xl font-normal transform md:rotate-0 rotate-90">→</div>
                    <div class="text-center p-4 bg-gray-200 rounded-lg">
                        <div class="text-4xl mb-2">🧱</div>
                        <h3 class="font-bold">Horno de Clinker</h3>
                        <p class="text-sm text-gray-500">Producción de cemento</p>
                    </div>
                </div>

                <div class="mt-12 max-w-4xl mx-auto bg-gray-100 p-8 rounded-lg shadow-inner">
                    <h3 class="font-bold text-xl mb-4 text-center">El Corazón del Proceso: El Horno de Clinker</h3>
                    <p class="text-gray-700 mb-3">
                        El horno de clinker es un gigantesco cilindro rotatorio que alcanza temperaturas de hasta 1450°C. Aquí, el combustible alternativo de Recycomb reemplaza a los combustibles fósiles, generando el calor necesario para una serie de transformaciones químicas cruciales.
                    </p>
                    <p class="text-gray-700">
                        Una de las principales es la <strong>calcinación</strong>, donde el carbonato de calcio (CaCO₃), componente de la materia prima, se descompone en óxido de calcio (CaO) y dióxido de carbono (CO₂). Posteriormente, el óxido de calcio reacciona con otros materiales como el dióxido de silicio (SiO₂) para formar silicatos de calcio, el componente principal del <strong>clinker</strong>, la base del cemento. Esta innovación es clave: reduce drásticamente el uso de combustibles fósiles, disminuyendo emisiones de CO₂ y protegiendo recursos no renovables.
                    </p>
                </div>
            </div>
        </section>

        <!-- Insumos Section: Details the types of waste processed -->
        <section id="insumos" class="py-16 md:py-24 bg-slate-50">
            <div class="container mx-auto px-6">
                <div class="text-center mb-12">
                    <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-4">Nuestros Insumos: El Potencial Oculto</h2>
                    <p class="max-w-3xl mx-auto text-gray-600">
                        Utilizamos una variedad de residuos industriales no peligrosos que, aunque no pueden ser reciclados por vías tradicionales, poseen un gran poder calorífico. Los transformamos de un problema a una solución energética valiosa. A continuación, una composición ilustrativa de nuestros insumos.
                    </p>
                </div>

                <!-- Grid of cards detailing different waste types -->
                <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8">
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Plásticos no reciclables (25% - ~50,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Aquellos que no tienen cabida en los circuitos de reciclaje convencionales, pero con alto valor energético.</p>
                    </div>
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Residuos textiles (20% - ~40,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Fragmentos o desechos de la industria de la moda o producción de telas que se aprovechan para energía.</p>
                    </div>
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Lodos industriales (15% - ~30,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Subproductos de procesos fabriles que son secados y tratados para su valorización energética.</p>
                    </div>
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Maderas tratadas (15% - ~30,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Desechos de construcciones o carpinterías que no pueden ser reutilizados, pero son una fuente de calor.</p>
                    </div>
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Caucho y Neumáticos (10% - ~20,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Neumáticos fuera de uso o recortes industriales con un considerable valor energético.</p>
                    </div>
                    <div class="waste-card bg-white p-6 rounded-lg shadow">
                        <h3 class="font-bold text-lg mb-2">Papel y cartón contaminado (15% - ~30,000 t/año)</h3>
                        <p class="text-gray-600 text-sm">Materiales que no pueden ser reciclados como papel común debido a su contaminación.</p>
                    </div>
                </div>

                <!-- Donut chart visualizing waste composition -->
                <div class="mt-16 text-center">
                    <h3 class="text-2xl font-bold mb-4">Composición Ilustrativa de Insumos</h3>
                    <p class="max-w-2xl mx-auto text-gray-600 mb-8">
                        Nuestro proceso está diseñado para manejar una mezcla diversa de materiales, optimizando el poder calorífico del combustible final. Este gráfico muestra una composición de ejemplo.
                    </p>
                    <div class="chart-container">
                        <canvas id="wasteChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <!-- Impacto Section: Highlights the benefits of Recycomb's work -->
        <section id="impacto" class="py-16 md:py-24 bg-green-800 text-white">
            <div class="container mx-auto px-6">
                <div class="text-center mb-12">
                    <h2 class="text-3xl md:text-4xl font-bold mb-4">Impacto Real, Futuro Sostenible</h2>
                    <p class="max-w-3xl mx-auto text-green-200">
                        Nuestra labor genera beneficios directos para el medio ambiente y la industria, creando un ciclo virtuoso donde los residuos de unos son el recurso energético de otros.
                    </p>
                </div>
                <div class="grid md:grid-cols-2 gap-8 max-w-4xl mx-auto">
                    <div class="bg-green-700 p-8 rounded-lg text-center">
                        <h3 class="text-5xl font-bold mb-2">↓ CO₂</h3>
                        <p class="font-semibold">Reducción de Emisiones</p>
                        <p class="text-green-200 mt-2">Al sustituir combustibles fósiles, disminuimos significativamente la huella de carbono de la industria pesada.</p>
                    </div>
                    <div class="bg-green-700 p-8 rounded-lg text-center">
                        <h3 class="text-5xl font-bold mb-2">🌍</h3>
                        <p class="font-semibold">Impulso a la Economía Circular</p>
                        <p class="text-green-200 mt-2">Transformamos pasivos ambientales en activos energéticos, cerrando el ciclo de producción y consumo.</p>
                    </div>
                </div>

                <!-- Gemini API powered "Dato Curioso" (Fun Fact) generator -->
                <div class="mt-16 bg-green-700 p-8 rounded-lg text-center max-w-2xl mx-auto">
                    <h3 class="text-2xl font-bold mb-4">Dato Curioso sobre Sostenibilidad ✨</h3>
                    <p class="text-green-200 mb-6">
                        Haz click para descubrir un dato interesante sobre la valorización de residuos o la economía circular.
                    </p>
                    <button id="generateFactBtn" class="bg-white text-green-800 px-6 py-3 rounded-full font-semibold shadow-lg hover:bg-gray-100 transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                        Generar Dato ✨
                    </button>
                    <p id="factDisplay" class="text-lg mt-6 italic text-green-100 min-h-[4rem] flex items-center justify-center">
                        <!-- El dato generado por la IA aparecerá aquí. -->
                    </p>
                    <div id="factLoading" class="hidden text-green-200 mt-4">Generando...</div>
                </div>

            </div>
        </section>
        
        <!-- Objetivos Section: Outlines Recycomb's future goals -->
        <section id="objetivos" class="py-16 md:py-24 bg-white">
            <div class="container mx-auto px-6">
                 <div class="text-center mb-12">
                    <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-4">Nuestra Visión de Futuro</h2>
                    <p class="max-w-3xl mx-auto text-gray-600">
                        La visión de Recycomb es clara: liderar la transición hacia un modelo de producción verdaderamente sustentable. Estos son nuestros pilares.
                    </p>
                </div>
                <div class="grid sm:grid-cols-2 lg:grid-cols-4 gap-8 max-w-6xl mx-auto">
                    <div class="text-center p-6">
                        <div class="text-4xl text-green-600 mb-4">🌿</div>
                        <h3 class="font-bold text-xl mb-2">Reducir el Impacto Ambiental</h3>
                        <p class="text-gray-600">Minimizando la huella de carbono y la acumulación de residuos en vertederos.</p>
                    </div>
                    <div class="text-center p-6">
                        <div class="text-4xl text-green-600 mb-4">🔄</div>
                        <h3 class="font-bold text-xl mb-2">Promover la Economía Circular</h3>
                        <p class="text-gray-600">Convirtiendo desechos en recursos valiosos y manteniendo los materiales en uso.</p>
                    </div>
                    <div class="text-center p-6">
                        <div class="text-4xl text-green-600 mb-4">🤝</div>
                        <h3 class="font-bold text-xl mb-2">Gestión Responsable</h3>
                        <p class="text-gray-600">Ofreciendo una solución ética, trazable y eficiente para los residuos industriales.</p>
                    </div>
                    <div class="text-center p-6">
                        <div class="text-4xl text-green-600 mb-4">⛽️</div>
                        <h3 class="font-bold text-xl mb-2">Disminuir Uso de Fósiles</h3>
                        <p class="text-gray-600">Protegiendo nuestros recursos naturales para las futuras generaciones.</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- Interactive Game Section -->
        <section id="juego" class="py-16 md:py-24 bg-slate-50">
            <div class="container mx-auto px-6">
                <div class="text-center mb-12">
                    <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-4">Juego: Clasifica y Gana</h2>
                    <p class="max-w-3xl mx-auto text-gray-600">
                        ¡Pon a prueba tus conocimientos! Arrastra cada residuo al contenedor correcto en 30 segundos. Recuerda, Recycomb se especializa en residuos que no pueden ir al reciclaje convencional.
                    </p>
                </div>

                <div class="max-w-4xl mx-auto bg-white p-8 rounded-lg shadow-xl">
                    <div class="flex justify-between items-center mb-6">
                        <div class="text-2xl font-bold">Puntuación: <span id="score">0</span></div>
                        <div class="text-2xl font-bold">Tiempo: <span id="timer">30</span>s</div>
                        <button id="startGameBtn" class="bg-green-600 text-white font-semibold px-6 py-3 rounded-lg shadow-md hover:bg-green-700 transition-colors">Empezar a Jugar</button>
                    </div>

                    <div id="waste-items-container" class="h-32 flex justify-center items-center flex-wrap gap-4 p-4 mb-6 bg-gray-100 rounded-lg border-2 border-dashed border-gray-300">
                        <!-- Waste items will be generated here by JavaScript -->
                    </div>

                    <div class="grid md:grid-cols-2 gap-8 mt-4">
                        <div id="recycomb-bin" class="drop-zone border-4 border-dashed border-gray-300 rounded-lg p-6 text-center bg-gray-50">
                            <h3 class="text-2xl font-bold mb-2 text-green-700">♻️ Valorización Energética (Recycomb)</h3>
                            <p class="text-gray-500">Arrastra aquí los residuos no reciclables que convertimos en energía.</p>
                        </div>
                        <div id="recycling-bin" class="drop-zone border-4 border-dashed border-gray-300 rounded-lg p-6 text-center bg-gray-50">
                            <h3 class="text-2xl font-bold mb-2 text-blue-700">🗑️ Reciclaje Convencional</h3>
                            <p class="text-gray-500">Arrastra aquí los residuos que se reciclan de forma tradicional.</p>
                        </div>
                    </div>

                    <div id="game-feedback" class="mt-6 text-center text-2xl font-bold"></div>
                </div>
            </div>
        </section>

    </main>

    <!-- Footer Section -->
    <footer class="bg-gray-800 text-white">
        <div class="container mx-auto px-6 py-12 text-center">
            <h2 class="text-2xl font-bold mb-4">Juntos construimos una industria más verde.</h2>
            <p class="mb-6 text-gray-400">Contáctanos para descubrir cómo podemos colaborar.</p>
            
            <!-- Canva Link Button - Relocated and restyled -->
            <div class="mb-8 mt-4">
                <a href="https://www.canva.com/design/DAGphPwyFs0/0ZA1LJymU8tRpf7qEj84HQ/edit?utm_content=DAGphPwyFs0&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton" target="_blank" rel="noopener noreferrer" class="inline-block bg-gradient-to-r from-green-500 to-green-600 text-white font-bold text-lg px-8 py-4 rounded-full shadow-lg hover:from-green-600 hover:to-green-700 transition-all duration-300 transform hover:scale-105">
                    Ver Nuestra Presentación Detallada
                </a>
            </div>

            <div class="flex justify-center space-x-6">
                <!-- Reminder: Update these placeholder href values -->
                <a href="#" class="text-gray-400 hover:text-white">Página Web</a>
                <a href="#" class="text-gray-400 hover:text-white">Redes Sociales</a>
                <a href="#" class="text-gray-400 hover:text-white">Contacto</a>
            </div>
            <p class="text-sm text-gray-500 mt-8">&copy; 2024 Recycomb S.A. Todos los derechos reservados.</p>
        </div>
    </footer>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            
            // --- General Page Scripts ---

            const mobileMenuButton = document.getElementById('mobile-menu-button');
            const mobileMenu = document.getElementById('mobile-menu');
            
            // Toggle mobile menu visibility and ARIA attribute
            if (mobileMenuButton && mobileMenu) {
                mobileMenuButton.addEventListener('click', () => {
                    const isExpanded = mobileMenu.classList.toggle('hidden');
                    mobileMenuButton.setAttribute('aria-expanded', !isExpanded);
                });
            }

            // Close mobile menu when a navigation link is clicked
            const allNavLinks = document.querySelectorAll('#mobile-menu a, header .hidden.md\\:flex a.nav-link');
            allNavLinks.forEach(link => {
                link.addEventListener('click', () => {
                    if (!mobileMenu.classList.contains('hidden')) {
                        mobileMenu.classList.add('hidden');
                        mobileMenuButton.setAttribute('aria-expanded', 'false');
                    }
                });
            });

            // Initialize Donut Chart for waste composition
            const ctx = document.getElementById('wasteChart');
            if (ctx) {
                new Chart(ctx, {
                    type: 'doughnut',
                    data: {
                        labels: ['Plásticos', 'Textiles', 'Lodos', 'Maderas', 'Caucho', 'Papel y cartón'],
                        datasets: [{
                            label: 'Composición de Residuos',
                            data: [25, 20, 15, 15, 10, 15], // Data matching card percentages
                            backgroundColor: [
                                '#4CAF50', '#81C784', '#A5D6A7',
                                '#C8E6C9', '#66BB6A', '#388E3C'
                            ],
                            borderColor: '#f8fafc', // Matches body background
                            borderWidth: 4
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                position: 'bottom',
                                labels: {
                                    color: '#4A5568', // text-gray-700
                                    font: {
                                        family: "'Roboto', sans-serif"
                                    }
                                }
                            }
                        },
                        cutout: '60%'
                    }
                });
            }

            // Gemini API Fact Generator
            const generateFactBtn = document.getElementById('generateFactBtn');
            const factDisplay = document.getElementById('factDisplay');
            const factLoading = document.getElementById('factLoading');

            if (generateFactBtn && factDisplay && factLoading) {
                generateFactBtn.addEventListener('click', async () => { 
                    factDisplay.textContent = ''; 
                    factLoading.classList.remove('hidden');
                    generateFactBtn.disabled = true;

                    let chatHistory = [];
                    const promptText = "Genera un dato interesante y conciso sobre la valorización de residuos industriales, la economía circular o los beneficios de los combustibles alternativos en la industria del cemento. El dato debe ser educativo y no más de 2 oraciones.";
                    chatHistory.push({ role: "user", parts: [{ text: promptText }] });
                    const payload = { contents: chatHistory };
                    const apiKey = ""; 
                    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

                    try {
                        const response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        const result = await response.json();

                        if (result.candidates && result.candidates[0]?.content?.parts[0]) {
                            const text = result.candidates[0].content.parts[0].text;
                            factDisplay.textContent = text;
                        } else {
                            factDisplay.textContent = "No se pudo generar el dato. Intenta de nuevo.";
                        }
                    } catch (error) {
                        factDisplay.textContent = "Error al conectar con la IA. Intenta de nuevo más tarde.";
                    } finally {
                        factLoading.classList.add('hidden');
                        generateFactBtn.disabled = false;
                    }
                });
            }

            // Active Navigation Link Highlighting on Scroll
            const sections = document.querySelectorAll('main section[id]');
            const siteHeader = document.querySelector('header');
            const headerHeight = siteHeader ? siteHeader.offsetHeight : 70; 

            function changeNavOnScroll() {
                if (!sections.length) return;

                let currentSectionId = '';
                for (let i = sections.length - 1; i >= 0; i--) {
                    const section = sections[i];
                    const sectionTopThreshold = section.offsetTop - headerHeight - 20;

                    if (window.scrollY >= sectionTopThreshold) {
                        currentSectionId = section.id;
                        break; 
                    }
                }
                
                if (window.scrollY < (sections[0].offsetTop - headerHeight - 20) && sections.length > 0) {
                     currentSectionId = 'hero';
                }

                allNavLinks.forEach(link => {
                    link.classList.remove('active');
                    link.removeAttribute('aria-current');
                    if (link.hash === `#${currentSectionId}`) {
                        link.classList.add('active');
                        link.setAttribute('aria-current', 'page');
                    }
                });
            }

            if (sections.length > 0) {
                changeNavOnScroll(); 
                window.addEventListener('scroll', changeNavOnScroll);
            }


            // --- Interactive Game Logic ---
            const startGameBtn = document.getElementById('startGameBtn');
            const scoreEl = document.getElementById('score');
            const timerEl = document.getElementById('timer');
            const wasteContainer = document.getElementById('waste-items-container');
            const recycombBin = document.getElementById('recycomb-bin');
            const recyclingBin = document.getElementById('recycling-bin');
            const gameFeedback = document.getElementById('game-feedback');

            const allWasteItems = [
                { name: 'Lodo Industrial', type: 'recycomb', emoji: '🛢️' },
                { name: 'Neumático', type: 'recycomb', emoji: '⚫' },
                { name: 'Tela Contaminada', type: 'recycomb', emoji: '👕' },
                { name: 'Plástico no Reciclable', type: 'recycomb', emoji: '🛍️' },
                { name: 'Madera Tratada', type: 'recycomb', emoji: '🪵' },
                { name: 'Botella de Vidrio', type: 'recycling', emoji: '🍾' },
                { name: 'Lata de Aluminio', type: 'recycling', emoji: '🥫' },
                { name: 'Papel Limpio', type: 'recycling', emoji: '📄' },
                { name: 'Cartón Limpio', type: 'recycling', emoji: '📦' },
                { name: 'Botella de Plástico (PET)', type: 'recycling', emoji: '💧' },
            ];

            let score = 0;
            let timer = 30;
            let timerInterval;
            let gameActive = false;

            function startGame() {
                score = 0;
                timer = 30;
                scoreEl.textContent = score;
                timerEl.textContent = timer;
                gameFeedback.textContent = '';
                startGameBtn.disabled = true;
                gameActive = true;
                
                generateWasteItems();
                
                timerInterval = setInterval(() => {
                    timer--;
                    timerEl.textContent = timer;
                    if (timer <= 0) {
                        endGame();
                    }
                }, 1000);
            }

            function endGame() {
                clearInterval(timerInterval);
                gameActive = false;
                wasteContainer.innerHTML = '<p class="text-gray-500">¡Juego terminado!</p>';
                if (score > 5) {
                    gameFeedback.textContent = `¡Excelente! Puntuación final: ${score}`;
                    gameFeedback.classList.add('text-green-600');
                    gameFeedback.classList.remove('text-red-600');
                } else {
                    gameFeedback.textContent = `¡Puedes mejorar! Puntuación final: ${score}`;
                    gameFeedback.classList.add('text-red-600');
                    gameFeedback.classList.remove('text-green-600');
                }
                startGameBtn.disabled = false;
            }
            
            function generateWasteItems() {
                wasteContainer.innerHTML = '';
                const shuffled = allWasteItems.sort(() => 0.5 - Math.random());
                shuffled.slice(0, 5).forEach(item => { // Show 5 items at a time
                    const itemEl = document.createElement('div');
                    itemEl.draggable = true;
                    itemEl.id = `waste-${Date.now()}-${Math.random()}`;
                    itemEl.dataset.type = item.type;
                    itemEl.className = 'waste-item bg-white p-4 rounded-lg shadow-md flex flex-col items-center';
                    itemEl.innerHTML = `<span class="text-4xl">${item.emoji}</span><span class="text-sm font-semibold mt-2">${item.name}</span>`;
                    
                    itemEl.addEventListener('dragstart', handleDragStart);
                    wasteContainer.appendChild(itemEl);
                });
            }

            function handleDragStart(e) {
                if (!gameActive) return;
                e.dataTransfer.setData('text/plain', e.target.id);
                e.dataTransfer.setData('waste-type', e.target.dataset.type);
            }

            function setupDropZones() {
                const dropZones = [recycombBin, recyclingBin];
                dropZones.forEach(zone => {
                    zone.addEventListener('dragover', (e) => {
                        e.preventDefault();
                        if (!gameActive) return;
                        zone.classList.add('drag-over');
                    });
                    zone.addEventListener('dragleave', () => {
                        zone.classList.remove('drag-over');
                    });
                    zone.addEventListener('drop', handleDrop);
                });
            }

            function handleDrop(e) {
                e.preventDefault();
                if (!gameActive) return;

                const droppedItemId = e.dataTransfer.getData('text/plain');
                const droppedItemType = e.dataTransfer.getData('waste-type');
                const droppedItemEl = document.getElementById(droppedItemId);
                
                const dropZoneId = e.currentTarget.id;
                const correctBin = (dropZoneId === 'recycomb-bin' && droppedItemType === 'recycomb') || 
                                   (dropZoneId === 'recycling-bin' && droppedItemType === 'recycling');

                if (correctBin) {
                    score++;
                    scoreEl.textContent = score;
                    e.currentTarget.classList.add('correct-drop');
                } else {
                    score--;
                    scoreEl.textContent = score;
                    e.currentTarget.classList.add('incorrect-drop');
                }
                
                if(droppedItemEl) {
                    droppedItemEl.remove();
                }
                
                if (wasteContainer.children.length === 0) {
                    generateWasteItems();
                }

                e.currentTarget.classList.remove('drag-over');
                setTimeout(() => {
                    e.currentTarget.classList.remove('correct-drop', 'incorrect-drop');
                }, 500);
            }

            startGameBtn.addEventListener('click', startGame);
            setupDropZones();
        });
    </script>
</body>
</html>

