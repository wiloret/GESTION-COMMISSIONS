<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestion des Précommandes et Commissions</title>
    <!-- Tailwind CSS CDN -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css" rel="stylesheet">
    <!-- Google Fonts - Poppins -->
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700;800;900&display=swap" rel="stylesheet">
    <!-- Chart.js for charts -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.2.1/chart.umd.min.js"></script>
    <!-- SheetJS for Excel export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <!-- jsPDF for PDF export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <style>
        /* Custom Tailwind class for 1px border */
        .border-px {
            border-width: 1px;
        }

        /* Global body styling with direct CSS properties for maximum compatibility */
        body {
            font-family: 'Poppins', sans-serif;
            background: linear-gradient(to bottom right, #bfdbfe, #d8b4fe); /* Equivalent to from-blue-200 to-purple-300 */
            color: #4b5563; /* Equivalent to text-gray-800 */
            padding: 2rem;
            min-height: 100vh;
        }
        header {
            /* Very deep, rich header with strong gradient and shadow */
            @apply text-center mb-20 py-16 bg-gradient-to-br from-blue-900 to-purple-800 text-white rounded-b-4xl shadow-4xl;
        }
        h1 {
            /* Larger, bolder, and tighter tracking for main title with text shadow */
            @apply text-7xl font-extrabold tracking-tightest;
            text-shadow: 4px 4px 8px rgba(0,0,0,0.4);
        }
        h2 {
            /* Larger and bolder section titles */
            @apply text-4xl font-extrabold mb-8 text-blue-800;
        }
        section {
            /* Enhanced section styling with frosted glass effect, deeper shadow, and pronounced hover */
            @apply bg-white bg-opacity-80 backdrop-filter backdrop-blur-lg p-16 rounded-5xl shadow-4xl mb-16 mx-auto max-w-7xl transform transition-all duration-500 hover:scale-[1.005] hover:shadow-5xl;
            border: 1px solid rgba(255, 255, 255, 0.5); /* Subtle white border for glass effect */
        }
        .actions {
            /* Dynamic button layout with a lighter gradient background and strong shadow */
            @apply flex flex-wrap gap-10 justify-center p-12 bg-gradient-to-r from-blue-50 to-purple-50 rounded-5xl shadow-3xl mb-16;
        }
        .form-group {
            @apply mb-10; /* More space between form groups */
        }
        label {
            @apply block text-lg font-semibold text-gray-700 mb-3; /* Larger font for labels */
        }
        input[type="text"],
        input[type="number"],
        input[type="date"], /* Added style for date input */
        select,
        textarea { /* Added textarea to input styles */
            /* Prominent input styles with generous padding, strong focus, and inner shadow */
            @apply w-full p-5 border-2 border-blue-300 rounded-2xl focus:ring-5 focus:ring-blue-700 focus:border-blue-700 shadow-inner transition-all duration-300 ease-in-out;
        }
        /* Base button styles applied to all buttons via a common class */
        .btn-base {
            @apply py-5 px-12 rounded-3xl font-extrabold transition-all duration-400 ease-in-out shadow-2xl hover:shadow-3xl transform hover:scale-105 hover:-translate-y-2;
            letter-spacing: 0.08em;
            color: white; /* Ensure text is white for all gradient buttons */
        }

        /* Specific styles for buttons using direct Tailwind classes in HTML */
        .summary-totals div { /* Specificity for summary totals */
            @apply text-2xl font-bold mb-5 text-blue-900; /* Larger and bolder summary text */
        }
        .summary-totals { /* New class for the always-visible summary */
            @apply bg-blue-100 bg-opacity-70 p-12 rounded-4xl shadow-xl border border-blue-200 mb-16 mx-auto max-w-7xl;
        }

        .summary-filters { /* Class for summary section when filters are present */
            @apply bg-blue-100 bg-opacity-70 p-12 rounded-4xl shadow-xl border border-blue-200 mb-16 mx-auto max-w-7xl;
        }

        table {
            /* Table with very prominent rounded corners and shadow */
            @apply w-full border-collapse mt-12 rounded-4xl overflow-hidden shadow-3xl;
            background-color: #ffffff; /* Pure white background for better contrast with rows */
        }
        th, td {
            /* MODIFICATION ICI: Bordures fines et couleur douce pour la grille */
            @apply border border-gray-300 p-4 align-middle; /* Centered vertically, text alignment handled per cell */
            font-size: 1rem; /* Adjusted font size for better fit */
            line-height: 1.4; /* Adjusted line spacing for readability */
        }
        /* Explicit text alignment rules to ensure proper alignment based on classes */
        th.text-left, td.text-left { text-align: left; }
        th.text-right, td.text-right { text-align: right; } /* Changed from text-right-td */
        th.text-center, td.text-center { text-align: center; }

        /* Specific style for PDF link to make it more prominent if it's a link */
        td a.text-blue-500 {
            @apply font-semibold; /* Make link text bolder */
        }

        /* Class for right-aligned numeric data - now replaced by .text-right */
        /* .text-right-td {
            @apply text-right;
        } */

        tbody tr:nth-child(even) {
            background-color: #e0f2fe; /* bg-blue-50, but explicitly set for clarity */
        }
        tbody tr:hover {
            @apply bg-blue-100 transition-colors duration-300; /* Stronger hover effect */
            box-shadow: inset 0 0 0 9999px rgba(0,0,0,0.05); /* Subtle overlay on hover */
        }

        /* NEW: Style for rows with "En attente pointage" status */
        .status-pending-highlight {
            background-color: #fefcbf; /* Equivalent to bg-yellow-100 */
        }

        /* Removed chart-container styling as it's now a generic content div */
        /* .chart-container {
            @apply flex flex-wrap md:flex-nowrap gap-12 mt-12;
        } */
        .stats-table-container { /* New class for individual table containers within stats */
            @apply flex-1 min-w-full md:min-w-0 bg-white bg-opacity-70 p-10 rounded-4xl shadow-2xl;
        }
        .hidden {
            display: none;
        }
        .message-modal {
            /* Darker, more opaque overlay for modal */
            @apply fixed inset-0 bg-gray-900 bg-opacity-95 flex items-center justify-content: center; /* Changed to flex-start for top alignment */
            align-items: flex-start; /* Align to top */
            padding-top: 5vh; /* Add some padding from the top */
            z-index: 50;
        }
        .message-content {
            /* More styled modal with stronger border and shadow */
            @apply bg-white p-16 rounded-5xl shadow-4xl max-w-md w-full text-center border-t-8 border-blue-700;
            position: relative; /* For close button positioning */
        }
        .message-content h3 {
            @apply text-3xl font-bold mb-8 text-gray-900; /* Larger modal title */
        }
        .message-content p {
            @apply text-xl text-gray-700 mb-12; /* Larger modal text */
        }
        .message-content button {
            @apply bg-blue-700 text-white px-12 py-5 rounded-xl hover:bg-blue-800; /* Larger modal button */
        }
        .loading-overlay {
            @apply fixed inset-0 bg-gray-900 bg-opacity-95 flex items-center justify-content: center;
        }
        .loading-spinner {
            @apply animate-spin rounded-full h-24 w-24 border-t-4 border-b-4 border-blue-700; /* Larger spinner */
        }
        .close-modal-btn {
            position: absolute;
            top: 1rem;
            right: 1rem;
            background: none;
            border: none;
            font-size: 2rem;
            color: #4b5563;
            cursor: pointer;
            padding: 0.5rem;
            line-height: 1;
            transition: color 0.2s;
        }
        .close-modal-btn:hover {
            color: #ef4444; /* red-500 */
        }
        .preorder-form-fields .form-group {
            margin-bottom: 1.5rem; /* Space between modal form groups */
            text-align: left; /* Align labels/inputs within modal form groups */
        }
        .preorder-form-fields label {
            font-size: 1rem; /* Smaller label in modal */
            font-weight: 600;
            margin-bottom: 0.5rem;
        }
        .preorder-form-fields input[type="text"],
        .preorder-form-fields input[type="number"],
        .preorder-form-fields input[type="date"],
        .preorder-form-fields textarea {
            padding: 0.75rem; /* Smaller padding for modal inputs */
            border-radius: 0.5rem; /* Slightly less rounded for modal inputs */
            font-size: 1rem;
        }
    </style>
</head>
<body>
    <header>
        <h1>Gestion des Précommandes et Commissions</h1>
    </header>

    <section class="actions">
        <!-- Buttons with direct Tailwind classes for color and rounded corners -->
        <button id="save-file-btn" class="btn-base bg-gradient-to-r from-indigo-600 to-indigo-800 hover:from-indigo-700 hover:to-indigo-900">Sauvegarder Fichier</button>
        <input type="file" id="load-input" accept="application/json" class="hidden"/>
        <button id="load-file-btn" class="btn-base bg-gradient-to-r from-indigo-600 to-indigo-800 hover:from-indigo-700 hover:to-indigo-900">Charger Fichier</button>
        
        <!-- NEW: Export PDF Button -->
        <button id="export-pdf-btn" class="btn-base bg-red-600 hover:bg-red-700 text-white">Exporter en PDF</button>

        <!-- Navigation buttons -->
        <button id="show-form-btn" class="btn-base bg-gradient-to-r from-blue-600 to-blue-700 hover:from-blue-700 hover:to-blue-800">Nouvelle Entrée / Saisie</button>
        <button id="show-list-btn" class="btn-base bg-gradient-to-r from-purple-600 to-purple-700 hover:from-purple-700 hover:to-purple-800">Voir la Liste Détaillée</button>
        
        <button id="showStats-btn" class="btn-base bg-gradient-to-r from-purple-700 to-purple-900 hover:from-purple-800 hover:to-purple-900">Afficher Stats</button>
        <!-- NEW: Preorder Request Button -->
        <button id="request-preorder-btn" class="btn-base bg-yellow-500 hover:bg-yellow-600 text-white">Demande de Précommande</button>
        <!-- NEW: Save/Load Club Data Buttons -->
        <button id="save-club-data-btn" class="btn-base bg-gradient-to-r from-blue-400 to-blue-500 hover:from-blue-500 hover:to-blue-600">Sauvegarder Clubs</button>
        <input type="file" id="load-club-input" accept="application/json" class="hidden"/>
        <button id="load-club-data-btn" class="btn-base bg-gradient-to-r from-blue-400 to-blue-500 hover:from-blue-500 hover:to-blue-600">Charger Clubs</button>
        <!-- END NEW -->
        <div id="user-id-display" class="text-sm font-medium text-gray-600 self-center">Mode local</div>

        <!-- Moved back-to-form-btn here -->
        <button id="back-to-form-btn" class="btn-base bg-gradient-to-r from-gray-600 to-gray-700 hover:from-gray-700 hover:to-gray-800 hidden">Retour à la Saisie</button>
    </section>

    <!-- Always visible Summary section (only totals) -->
    <section class="summary-totals">
        <h2>Résumé des Précommandes</h2>
        <div>Cumul CA HT : <span id="cumul-ca" class="font-bold">0.00</span> €</div>
        <div>Cumul Commission perçue : <span id="cumul-commission-perc" class="font-bold">0.00</span> €</div>
        <div>Cumul Articles commandés : <span id="cumul-articles" class="font-bold">0</span></div>
        <div>Panier moyen (CA HT / Nombre Commandes) : <span id="panier-moyen" class="font-bold">0.00</span> €</div>
    </section>

    <!-- Main content container for switching views -->
    <div id="main-content-container">
        <!-- Form View Container (initially visible) -->
        <div id="form-view-container">
            <section id="form-section">
                <h2>Ajouter/Modifier une Entrée</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                    <div class="form-group"><label for="saison">Saison :</label><select id="saison"><option>2024/2025</option><option>2025/2026</option></select></div>
                    <div class="form-group"><label for="club">Club :</label><input type="text" id="club" placeholder="Nom du club"/></div>
                    <div class="form-group"><label for="numClient">N° Client :</label><input type="text" id="numClient" placeholder="Numéro de client"/></div>
                    <div class="form-group"><label for="numDossier">N° Dossier :</label><input type="text" id="numDossier" placeholder="Numéro de dossier"/></div>
                    <div class="form-group"><label for="departement">N° Département :</label><input type="text" id="departement" placeholder="Numéro de département"/></div>
                    <div class="form-group"><label for="numPrecommande">N° Précommande :</label><input type="text" id="numPrecommande" value="0"/></div>
                    <div class="form-group"><label for="quantiteApprox">Qté approx :</label><input type="number" id="quantiteApprox" value="0"/></div>
                    <div class="form-group"><label for="numAR">N° AR :</label><input type="text" id="numAR" placeholder="Numéro AR"/></div>
                    
                    <!-- Champ PDF AR pour URL -->
                    <div class="form-group">
                        <label for="arPdf">PDF AR (URL) :</label>
                        <input type="text" id="arPdf" placeholder="Collez l'URL du PDF ici" class="flex-1"/>
                    </div>

                    <!-- Date de livraison field -->
                    <div class="form-group">
                        <label for="dateLivraison">Date de livraison :</label>
                        <input type="date" id="dateLivraison"/>
                    </div>

                    <!-- MODIFICATION ICI: Champ "AR" avec "En attente pointage" en première option -->
                    <div class="form-group">
                        <label for="arSigne">AR :</label>
                        <select id="arSigne">
                            <option value="En attente pointage" selected>En attente pointage</option> <!-- Set as selected -->
                            <option value="En attente AR">En attente AR</option>
                            <option value="En attente signature">En attente signature</option>
                            <option value="Précommande fixée">Précommande fixée</option> <!-- NEW OPTION ADDED HERE -->
                            <option value="Signé">Signé</option>
                        </select>
                    </div>
                    <!-- FIN MODIFICATION -->

                    <div class="form-group"><label for="quantiteReelle">Qté commandée :</label><input type="number" id="quantiteReelle" value="0"/></div>
                    <div class="form-group"><label for="montantHT">Montant HT :</label><input type="number" step="0.01" id="montantHT" value="0.00"/></div>
                    <div class="form-group"><label for="taux">Taux (%) :</label><select id="taux"><option>0</option><option>6</option><option>10</option></select></div>
                    <div class="form-group"><label for="commissionPrev">Commission prévue :</label><input type="number" id="commissionPrev" readonly value="0.00"/></div>
                    <div class="form-group"><label for="commissionPercue">Commission perçue :</label><input type="number" id="commissionPercue" value="0.00"/></div>
                    <div class="form-group"><label for="mois-annee">Année :</label><select id="mois-annee"><option>2024</option><option>2025</option><option>2026</option></select></div>
                    <div class="form-group"><label for="mois-mois">Mois :</label><select id="mois-mois"><option value="01">Janvier</option><option value="02">Février</option><option value="03">Mars</option><option value="04">Avril</option><option value="05">Mai</option><option value="06">Juin</option><option value="07">Juillet</option><option value="08">Août</option><option value="09">Septembre</option><option value="10">Octobre</option><option value="11">Novembre</option><option value="12">Décembre</option></select></div>
                </div>
                <div class="form-group actions mt-8">
                    <button id="submit-btn" class="btn-base bg-gradient-to-r from-blue-700 to-blue-900 hover:from-blue-800 hover:to-blue-900">Enregistrer</button>
                    <button id="cancel-btn" class="hidden btn-base bg-gradient-to-r from-red-600 to-red-800 hover:from-red-700 hover:to-red-900">Annuler</button>
                    <!-- NEW: Duplicate Last Entry Button -->
                    <button id="duplicate-entry-btn" class="btn-base bg-gradient-to-r from-gray-500 to-gray-600 hover:from-gray-600 hover:to-gray-700">Dupliquer la dernière entrée</button>
                </div>
            </section>
        </div>

        <!-- List View Container (initially hidden) -->
        <div id="list-view-container" class="hidden">
            <section class="summary-filters">
                <h2>Filtres de la Liste</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div class="form-group">
                        <label for="filter-saison">Filtrer par saison :</label>
                        <select id="filter-saison">
                            <option value="all">Toutes</option>
                            <option value="2024/2025">2024/2025</option>
                            <option value="2025/2026">2025/2026</option>
                        </select>
                    </div>
                    <!-- Filter by Client Name -->
                    <div class="form-group">
                        <label for="filter-clubName">Filtrer par Nom du club :</label> <!-- NEW FILTER FIELD -->
                        <input type="text" id="filter-clubName" placeholder="Saisir Nom du club"/>
                    </div>
                    <!-- Filter by Client Number -->
                    <div class="form-group">
                        <label for="filter-numClient">Filtrer par N° Client :</label>
                        <input type="text" id="filter-numClient" placeholder="Saisir N° Client"/>
                    </div>
                    <!-- Filter by Dossier Status -->
                    <div class="form-group">
                        <label for="filter-dossierStatus">Filtrer par Statut Dossier :</label>
                        <select id="filter-dossierStatus">
                            <option value="all">Tous</option>
                            <option value="N/A">N/A</option>
                            <option value="⏳ En attente">En attente</option>
                            <option value="⚠️ Relancer client">Relancer client</option>
                            <option value="❌ Urgence relance">Urgence relance</option>
                            <option value="✅ Validé">Validé</option>
                            <option value="Précommande fixée">Précommande fixée</option> <!-- NEW OPTION FOR FILTER -->
                            <option value="En attente pointage">En attente pointage</option> <!-- ADDED TO FILTER -->
                            <option value="Pointée">Pointée</option> <!-- NEW OPTION FOR FILTER -->
                        </select>
                    </div>
                </div>
            </section>

            <section id="table-section">
                <h2>Liste des Entrées</h2>
                <div>
                    <table id="table-commissions">
                        <thead>
                            <tr>
                                <!-- Removed Saison column -->
                                <th class="text-left">Club</th>
                                <th class="text-right">N° Client</th>
                                <th class="text-right">N° Dossier</th>
                                <th class="text-left">Dépt</th>
                                <th class="text-left">Préco</th>
                                <th class="text-center">Qté approx</th>
                                <th class="text-left">AR</th>
                                <!-- Removed PDF AR column from table header -->
                                <th class="text-center">Date de livraison</th>
                                <th class="text-left">AR</th>
                                <th class="text-left">Statut Dossier</th>
                                <th class="text-right">Qté commandée</th>
                                <th class="text-right">Montant HT</th>
                                <th class="text-right">Commission prévue</th>
                                <th class="text-right">Commission perçue</th>
                                <th class="text-right">Écart</th>
                                <th class="text-left">Mois pointage</th>
                                <th class="text-center">Finalisée</th>
                                <th class="text-center">Actions</th>
                            </tr>
                        </thead>
                        <tbody id="commission-table">
                            <!-- Data will be loaded here by JavaScript -->
                        </tbody>
                    </table>
                </div>
            </section>
        </div>

        <!-- Stats View Container (now a direct peer to form/list containers) -->
        <section id="stats-section" class="hidden">
            <h2>Statistiques</h2>
            <!-- NEW: Season filter for stats -->
            <div class="form-group mb-8">
                <label for="filter-saison-stats">Filtrer les statistiques par saison :</label>
                <select id="filter-saison-stats">
                    <option value="all">Toutes</option>
                    <option value="2024/2025">2024/2025</option>
                    <option value="2025/2026">2025/2026</option>
                </select>
            </div>
            <div id="stats-content" class="flex flex-wrap md:flex-nowrap gap-12 mt-12">
                <!-- Content will be generated dynamically here by JavaScript -->
            </div>
            <p id="no-stats-data-message" class="text-center text-gray-600 mt-4 hidden">Aucune donnée disponible pour les statistiques.</p>
        </section>
    </div>

    <!-- Preorder Request Modal (now in-page) -->
    <div id="preorder-request-modal" class="message-modal hidden">
        <div class="message-content">
            <h3 id="preorder-modal-title">Gérer la Précommande</h3>
            <button class="close-modal-btn" onclick="closePreorderModal()">×</button>
            <div class="preorder-form-fields">
                <!-- Club Name and Client Number fields in modal -->
                <div class="form-group">
                    <label for="modalClubName">Nom du club :</label>
                    <input type="text" id="modalClubName" placeholder="Nom du club"/>
                </div>
                <div class="form-group">
                    <label for="modalClientNum">N° club :</label>
                    <input type="text" id="modalClientNum" placeholder="N° club"/>
                </div>
                <div class="form-group">
                    <label for="modalPreorderNum">N° précommande (si connu) :</label>
                    <input type="text" id="modalPreorderNum" placeholder="N° précommande"/>
                </div>
                <div class="form-group">
                    <label>Quantité prévisionnelle :</label>
                    <div class="grid grid-cols-3 gap-4">
                        <input type="number" id="modalQtyHauts" placeholder="Hauts" value="0"/>
                        <input type="number" id="modalQtyBas" placeholder="Bas" value="0"/>
                        <input type="number" id="modalQtyAcc" placeholder="Acc." value="0"/>
                    </div>
                </div>
                <div class="form-group">
                    <label for="modalDateEssayage">Date d'essayage (si prévue) :</label>
                    <input type="date" id="modalDateEssayage"/>
                </div>
                <div class="form-group">
                    <label for="modalDateDepartSouhaitee">Date départ souhaitée :</label>
                    <input type="date" id="modalDateDepartSouhaitee"/>
                </div>
                <div class="form-group">
                    <label for="modalNotesReservation">Notes pour la demande de réservation...</label>
                    <textarea id="modalNotesReservation" rows="4" placeholder="Détails supplémentaires..."></textarea>
                </div>
                <div class="form-group flex items-center">
                    <input type="checkbox" id="doNotSendEmail" class="mr-2"/>
                    <label for="doNotSendEmail" class="mb-0 text-base font-normal">Ne pas envoyer d'email (juste enregistrer la précommande)</label>
                </div>
            </div>
            <div class="actions mt-8 flex justify-end gap-4">
                <button id="cancel-preorder-modal-btn" class="btn-base bg-gray-300 text-gray-800 hover:bg-gray-400">Annuler</button>
                <button id="send-preorder-email-btn" class="btn-base bg-blue-700 text-white hover:bg-blue-800">Valider</button>
            </div>
        </div>
    </div>

    <!-- Message Modal -->
    <div id="message-modal" class="message-modal hidden">
        <div id="message-content" class="message-content">
            <h3 id="message-title"></h3>
            <p id="message-text"></p>
            <!-- Buttons will be dynamically added here -->
        </div>
    </div>

    <!-- Loading Overlay -->
    <div id="loading-overlay" class="loading-overlay hidden">
        <div class="loading-spinner"></div>
    </div>

    <script type="module">
        // Helper function to get element by ID
        const q = id => document.getElementById(id);

        // Helper function to add event listener
        const on = (id, ev, fn) => {
            const el = q(id);
            if (el) el.addEventListener(ev, fn);
        };

        // Global variables for app state
        let entries = []; // Array to store pre-order entries
        let editDocId = null; // Stores the document ID when editing an entry
        // Added defaultTaux to clubDataStore structure
        let clubDataStore = []; // Global variable to store unique club data for auto-filling: { club: '...', numClient: '...', departement: '...', numDossier: '...', defaultTaux: '...' }

        // --- UI Elements ---
        // Always visible summary elements
        const cumulCA = q('cumul-ca');
        const cumulPerc = q('cumul-commission-perc');
        const cumulArt = q('cumul-articles');
        const panier = q('panier-moyen');
        
        // Elements specific to the list view (now moved)
        const tableBody = q('commission-table');
        const filterSaison = q('filter-saison');
        const filterNumClient = q('filter-numClient');
        const filterDossierStatus = q('filter-dossierStatus');
        const showStatsBtn = q('showStats-btn');
        const statsSection = q('stats-section');
        const noStatsDataMessage = q('no-stats-data-message'); // NEW: No data message element
        const statsContentDiv = q('stats-content'); // Get the new stats content div
        const filterSaisonStats = q('filter-saison-stats'); // NEW: Season filter for stats

        // Elements specific to the form view
        const montantHTInput = q('montantHT');
        const tauxSelect = q('taux');
        const commissionPrevInput = q('commissionPrev');
        const submitBtn = q('submit-btn');
        const cancelBtn = q('cancel-btn');
        const arPdfInput = q('arPdf');
        const dateLivraisonInput = q('dateLivraison');
        const arSigneSelect = q('arSigne'); // Renamed from arSigneSelect to arSelect as per user request (but variable name remains for consistency)
        const clubInputMain = q('club'); // Main form club input
        const numClientInputMain = q('numClient'); // Main form client number input
        const numDossierInputMain = q('numDossier'); // Main form dossier number input
        const departementInputMain = q('departement'); // Main form department input
        const mainContentContainer = q('main-content-container'); // Get the main content container


        // Global utility elements
        const messageModal = q('message-modal');
        const messageTitle = q('message-title');
        const messageText = q('message-text');
        const loadingOverlay = q('loading-overlay');
        const userIdDisplay = q('user-id-display');

        // Global file operation elements
        const loadInput = q('load-input');
        const loadFileBtn = q('load-file-btn');
        const saveFileBtn = q('save-file-btn');
        // Club data file elements
        const loadClubInput = q('load-club-input');
        const loadClubDataBtn = q('load-club-data-btn');
        const saveClubDataBtn = q('save-club-data-btn');

        // View containers and navigation buttons
        const formViewContainer = q('form-view-container');
        const listViewContainer = q('list-view-container');
        const showFormBtn = q('show-form-btn');
        const showListBtn = q('show-list-btn');
        const backToFormBtn = q('back-to-form-btn'); // Now global

        // Duplicate button element
        const duplicateEntryBtn = q('duplicate-entry-btn');

        // Preorder Modal Elements
        const preorderRequestModal = q('preorder-request-modal');
        const modalPreorderNum = q('modalPreorderNum');
        const modalQtyHauts = q('modalQtyHauts');
        const modalQtyBas = q('modalQtyBas');
        const modalQtyAcc = q('modalQtyAcc');
        const modalDateEssayage = q('modalDateEssayage');
        const modalDateDepartSouhaitee = q('modalDateDepartSouhaitee');
        const modalNotesReservation = q('modalNotesReservation');
        const doNotSendEmailCheckbox = q('doNotSendEmail');
        const cancelPreorderModalBtn = q('cancel-preorder-modal-btn');
        const sendPreorderEmailBtn = q('send-preorder-email-btn');
        const requestPreorderBtn = q('request-preorder-btn');
        const modalClubName = q('modalClubName'); // NEW: Modal Club Name
        const modalClientNum = q('modalClientNum'); // NEW: Modal Client Number


        // --- Message Modal Functions ---
        /**
         * Displays a custom message modal.
         * @param {string} title - The title of the message.
         * @param {string} text - The main text content of the message.
         * @param {Array<Object>} buttons - An array of button configurations.
         * Each object should have:
         * - {string} text: The button's label.
         * - {string} className: Tailwind CSS classes for styling the button.
         * - {Function} onClick: The function to execute when the button is clicked.
         */
        window.showMessage = function(title, text, buttons = [{ text: 'OK', className: 'bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700', onClick: () => window.hideMessage() }]) {
            messageTitle.textContent = title;
            messageText.textContent = text;
            const messageContent = q('message-content');
            // Clear existing buttons first to prevent duplicates
            const existingButtons = messageContent.querySelectorAll('button');
            existingButtons.forEach(btn => btn.remove());

            // Add new buttons based on the configuration
            buttons.forEach(btnConfig => {
                const button = document.createElement('button');
                button.textContent = btnConfig.text;
                button.className = btnConfig.className;
                button.onclick = btnConfig.onClick;
                messageContent.appendChild(button);
            });
            messageModal.classList.remove('hidden');
        }

        /**
         * Hides the custom message modal (generic message modal).
         * This function does NOT hide the preorder request modal.
         */
        window.hideMessage = function() {
            messageModal.classList.add('hidden');
        }

        // --- Loading Indicator Functions ---
        /**
         * Displays the loading overlay.
         */
        function showLoading() {
            loadingOverlay.classList.remove('hidden');
        }

        /**
         * Hides the loading overlay.
         */
        function hideLoading() {
            loadingOverlay.classList.add('hidden');
        }

        // Helper to format date to DD/MM/YYYY
        function formatDate(dateString) {
            if (!dateString) return '';
            const date = new Date(dateString);
            if (isNaN(date.getTime())) return dateString; // Return original if invalid date
            const day = String(date.getDate()).padStart(2, '0');
            const month = String(date.getMonth() + 1).padStart(2, '0'); // Month is 0-indexed
            const year = date.getFullYear();
            return `${day}/${month}/${year}`;
        }

        /**
         * Calculates the dossier status based on delivery date and AR signed status.
         * @param {string} dateLivraison - The delivery date string (YYYY-MM-DD).
         * @param {string} arStatus - The AR status ('En attente pointage', 'En attente AR', 'En attente signature', 'Signé', 'Précommande fixée').
         * @param {number} commissionPercue - The perceived commission amount.
         * @returns {string} The calculated dossier status.
         */
        function getDossierStatus(dateLivraison, arStatus, commissionPercue) {
            const deliveryDate = dateLivraison ? new Date(dateLivraison) : null;
            const currentDate = new Date();
            currentDate.setHours(0, 0, 0, 0);

            // Rule 1: If commission is perceived, it's "Pointée" (highest priority)
            if (parseFloat(commissionPercue) > 0) {
                return 'Pointée';
            }

            // Rule 2: If AR is 'Signé' OR 'En attente pointage', it's "En attente pointage"
            // This covers both cases where AR is signed and waiting for pointing, or explicitly set to awaiting pointing.
            if (arStatus === 'Signé' || arStatus === 'En attente pointage') {
                return 'En attente pointage';
            }

            // Rule 3: Handle 'Précommande fixée' explicitly
            if (arStatus === 'Précommande fixée') {
                // If 'Précommande fixée' without a delivery date, it remains 'Précommande fixée'
                if (!deliveryDate || isNaN(deliveryDate.getTime())) {
                    return 'Précommande fixée';
                }
                // If 'Précommande fixée' has a date, it falls into the date-based warning logic below
            }

            // Rule 4: Date-based logic for statuses that require it
            // This applies to 'En attente AR', 'En attente signature', and 'Précommande fixée' (if it has a date)
            if (deliveryDate && !isNaN(deliveryDate.getTime())) {
                const warningDate = new Date(deliveryDate);
                warningDate.setDate(deliveryDate.getDate() - (7 * 7)); // 7 weeks before
                warningDate.setHours(0, 0, 0, 0);

                const validationDate = new Date(deliveryDate);
                validationDate.setDate(deliveryDate.getDate() - (6 * 7)); // 6 weeks before
                validationDate.setHours(0, 0, 0, 0);

                if (arStatus === 'En attente AR' || arStatus === 'En attente signature' || arStatus === 'Précommande fixée') {
                    if (currentDate < warningDate) {
                        return '⏳ En attente';
                    } else if (currentDate >= warningDate && currentDate < validationDate) {
                        return '⚠️ Relancer client';
                    } else { // currentDate >= validationDate
                        return '❌ Urgence relance';
                    }
                }
            }

            // Final Fallback: Return the AR status itself if no other specific dossier status applies
            // This catches 'En attente AR' and 'En attente signature' if no date logic applies,
            // and 'Précommande fixée' if it has no date and no commission.
            if (arStatus === 'En attente AR') return 'En attente AR';
            if (arStatus === 'En attente signature') return 'En attente signature';
            if (arStatus === 'Précommande fixée') return 'Précommande fixée'; // Explicitly return if it hasn't been caught by date logic
            
            return 'N/A'; // Default for unhandled cases
        }


        // --- Form and Table Management ---

        /**
         * Resets all form fields to their default values and clears the edit state.
         */
        function resetForm() {
            editDocId = null; // Clear the document ID for editing
            q('saison').value = '2024/2025';
            q('club').value = '';
            q('numClient').value = '';
            q('numDossier').value = '';
            q('departement').value = '';
            q('numPrecommande').value = '';
            q('quantiteApprox').value = '0';
            q('numAR').value = '';
            arPdfInput.value = ''; // Clear PDF URL field
            dateLivraisonInput.value = ''; // Clear dateLivraison field
            arSigneSelect.value = 'En attente pointage'; // Set default to 'En attente pointage'
            console.log("resetForm: arSigneSelect value set to", arSigneSelect.value); // Debugging
            q('quantiteReelle').value = '0';
            q('montantHT').value = '0.00';
            q('taux').value = '0'; // Reset taux to default
            q('commissionPrev').value = '0.00';
            q('commissionPercue').value = '0.00';
            q('mois-annee').value = new Date().getFullYear().toString(); // Set current year
            q('mois-mois').value = (new Date().getMonth() + 1).toString().padStart(2, '0'); // Set current month
            cancelBtn.classList.add('hidden'); // Hide cancel button
            submitBtn.textContent = 'Enregistrer'; // Change button text back to 'Enregistrer'
        }

        /**
         * Renders the table with the current filtered entries.
         * Applies filters based on season, client number, and dossier status.
         */
        function renderTable() {
            tableBody.innerHTML = ''; // Clear existing rows
            const seasonFilter = filterSaison.value;
            const clientFilter = filterNumClient.value.trim().toLowerCase();
            const dossierStatusFilter = filterDossierStatus.value; // Get dossier status filter value
            const clubNameFilter = q('filter-clubName').value.trim().toLowerCase(); // NEW: Get club name filter value

            let filteredEntries = entries.filter(e => {
                const matchesSeason = seasonFilter === 'all' || e.saison === seasonFilter;
                const matchesClient = clientFilter === '' || (e.numClient && e.numClient.toLowerCase().includes(clientFilter));
                const matchesClubName = clubNameFilter === '' || (e.club && e.club.toLowerCase().includes(clubNameFilter)); // NEW: Match club name
                
                // Calculate current dossier status for the entry to apply filter
                const currentDossierStatus = getDossierStatus(e.dateLivraison, e.arSigne, e.commissionPercue); // Pass commissionPercue
                
                // MODIFICATION ICI: Logique de filtrage pour "Précommandes fixée" et autres statuts
                let matchesDossierStatus = false;
                if (dossierStatusFilter === 'all') {
                    matchesDossierStatus = true;
                } else if (dossierStatusFilter === 'Précommande fixée') {
                    // If filter is "Précommande fixée", match entries where arSigne is exactly "Précommande fixée"
                    matchesDossierStatus = e.arSigne === 'Précommande fixée';
                } else {
                    // For all other specific status filters, match the calculated dossierStatus
                    matchesDossierStatus = currentDossierStatus === dossierStatusFilter;
                }

                return matchesSeason && matchesClient && matchesClubName && matchesDossierStatus; // NEW: Include club name filter
            });

            // Sort filtered entries by Club alphabetically
            filteredEntries.sort((a, b) => {
                const clubA = a.club || '';
                const clubB = b.club || '';
                return clubA.localeCompare(clubB);
            });

            // Variable to keep track of the last rendered club name
            let lastClubRendered = '';

            filteredEntries.forEach(e => {
                // Determine if club name should be displayed for this row
                const currentClub = e.club;
                const displayClubName = (currentClub !== lastClubRendered) ? currentClub : '';
                lastClubRendered = currentClub; // Update the last rendered club

                // Ensure numeric values are treated as numbers for calculation
                const montantHT = Number(e.montantHT) || 0;
                const taux = Number(e.taux) || 0;
                const commissionPercue = Number(e.commissionPercue) || 0;
                const commissionPrev = (montantHT * taux / 100);

                const prevFormatted = commissionPrev.toFixed(2);
                const percFormatted = commissionPercue.toFixed(2);
                const ecart = (commissionPercue - commissionPrev).toFixed(2);
                const color = ecart < 0 ? 'text-red-600' : (ecart > 0 ? 'text-green-600' : 'text-gray-800');

                // Add highlight class if status is "En attente pointage"
                const rowHighlightClass = (getDossierStatus(e.dateLivraison, e.arSigne, e.commissionPercue) === 'En attente pointage') ? 'status-pending-highlight' : '';

                const row = `
                    <tr class="${rowHighlightClass}">
                        <td class="text-left">${displayClubName}</td> <!-- Use displayClubName here -->
                        <td class="text-right">${e.numClient}</td>
                        <td class="text-right">${e.numDossier}</td>
                        <td class="text-left">${e.departement}</td>
                        <td class="text-left">${e.numPrecommande}</td>
                        <td class="text-center">${e.quantiteApprox}</td>
                        <td class="text-left">${e.numAR}</td>
                        <td class="text-center">${formatDate(e.dateLivraison)}</td> <!-- Date de livraison -->
                        <td class="text-left">${e.arSigne || 'N/A'}</td> <!-- AR (Signé status) -->
                        <td class="text-left">${getDossierStatus(e.dateLivraison, e.arSigne, e.commissionPercue)}</td>
                        <td class="text-right">${e.quantiteReelle}</td>
                        <td class="text-right">${montantHT.toFixed(2)}</td>
                        <td class="text-right">${prevFormatted}</td>
                        <td class="text-right">${percFormatted}</td>
                        <td class="text-right ${color}">${ecart}</td>
                        <td class="text-left">${e.mois}</td>
                        <td class="text-center">${e.finalisee}</td>
                        <td class="text-center">
                            <button onclick="window.editLine('${e.id}')" class="bg-blue-500 text-white px-2 py-1 rounded-md text-sm">✏️</button>
                            <button onclick="window.deleteLine('${e.id}')" class="bg-red-500 text-white px-2 py-1 rounded-md text-sm">🗑️</button>
                        </td>
                    </tr>
                `;
                tableBody.insertAdjacentHTML('beforeend', row);
            });
        }

        /**
         * Updates the summary statistics based on the currently filtered entries.
         */
        function updateSummary() {
            // Summary totals should now reflect the selected season from the filter
            const seasonFilter = filterSaison.value;
            let filteredEntriesForSummary = entries;

            if (seasonFilter !== 'all') {
                filteredEntriesForSummary = entries.filter(e => e.saison === seasonFilter);
            }

            let totalCA = 0;
            let totalPercCommission = 0;
            let totalArticles = 0;
            let numOrders = 0;

            filteredEntriesForSummary.forEach(e => {
                const montantHT = Number(e.montantHT) || 0;
                const commissionPercue = Number(e.commissionPercue) || 0;
                const quantiteReelle = Number(e.quantiteReelle) || 0;
                const quantiteApprox = Number(e.quantiteApprox) || 0;

                totalCA += montantHT;
                totalPercCommission += commissionPercue;
                totalArticles += (quantiteReelle > 0 ? quantiteReelle : quantiteApprox);
                numOrders++;
            });

            cumulCA.textContent = totalCA.toFixed(2);
            cumulPerc.textContent = totalPercCommission.toFixed(2);
            cumulArt.textContent = totalArticles;
            panier.textContent = (numOrders ? (totalCA / numOrders).toFixed(2) : '0.00');
        }

        /**
         * Calculates and updates the 'Commission prévue' field based on 'Montant HT' and 'Taux'.
         */
        function calculateCommissionPrev() {
            const montant = Number(montantHTInput.value) || 0;
            const taux = Number(tauxSelect.value) || 0;
            commissionPrevInput.value = (montant * taux / 100).toFixed(2);
        }

        // Event listeners for dynamic commission calculation
        on('montantHT', 'input', calculateCommissionPrev);
        on('taux', 'change', calculateCommissionPrev);

        // --- CRUD Operations (In-memory) ---

        // Handle form submission (Add/Edit)
        on('submit-btn', 'click', async (e) => {
            e.preventDefault();
            showLoading(); // Show loading indicator

            const newEntry = {
                // Generate a unique ID for new entries, or use existing for edits
                id: editDocId || crypto.randomUUID(),
                saison: q('saison').value,
                club: q('club').value,
                numClient: q('numClient').value,
                numDossier: q('numDossier').value,
                departement: q('departement').value,
                numPrecommande: q('numPrecommande').value,
                quantiteApprox: Number(q('quantiteApprox').value) || 0,
                numAR: q('numAR').value,
                arPdf: arPdfInput.value, // Get value from the arPdf text input
                dateLivraison: dateLivraisonInput.value, // Get dateLivraison value
                arSigne: arSigneSelect.value, // Get AR Signé value
                quantiteReelle: Number(q('quantiteReelle').value) || 0,
                montantHT: Number(q('montantHT').value) || 0,
                taux: Number(q('taux').value) || 0,
                commissionPrev: Number(q('commissionPrev').value) || 0, // Store calculated value
                commissionPercue: Number(q('commissionPercue').value) || 0,
                mois: q('mois-annee').value + '-' + q('mois-mois').value
            };
            // Calculate finalisee for the new entry
            newEntry.finalisee = (newEntry.commissionPercue > 0 || newEntry.arSigne === 'Signé') ? '✔️' : '';


            // Check for duplicates
            const isDuplicate = entries.some(entry =>
                entry.id !== newEntry.id && // Exclude the current entry if it's an edit
                entry.saison === newEntry.saison &&
                entry.club.toLowerCase() === newEntry.club.toLowerCase() &&
                entry.numClient.toLowerCase() === newEntry.numClient.toLowerCase() &&
                entry.numPrecommande.toLowerCase() === newEntry.numPrecommande.toLowerCase()
            );

            if (isDuplicate) {
                hideLoading();
                const proceedAnyway = await new Promise(resolve => {
                    showMessage(
                        "Doublon détecté",
                        "Une entrée avec la même Saison, Club, N° Client et N° Précommande existe déjà. Voulez-vous quand même enregistrer cette ligne ?",
                        [
                            { text: 'Oui, enregistrer', className: 'bg-yellow-600 text-white px-6 py-2 rounded-md hover:bg-yellow-700 mr-2', onClick: () => { hideMessage(); resolve(true); } },
                            { text: 'Annuler', className: 'bg-gray-300 text-gray-800 px-6 py-2 rounded-md hover:bg-gray-400', onClick: () => { hideMessage(); resolve(false); } }
                        ]
                    );
                });

                if (!proceedAnyway) {
                    return; // Stop the function if user cancels
                }
                showLoading(); // Re-show loading if user proceeds
            }

            try {
                if (editDocId) {
                    // Update existing entry in the array
                    const index = entries.findIndex(entry => entry.id === editDocId);
                    if (index !== -1) {
                        entries[index] = newEntry;
                        showMessage("Succès", "Entrée mise à jour avec succès !");
                    } else {
                        throw new Error("Entry not found for update.");
                    }
                } else {
                    // Add new entry to the array
                    entries.push(newEntry);
                    showMessage("Succès", "Nouvelle entrée ajoutée avec succès !");
                }
                // Update club data store after saving an entry, including the current taux
                addOrUpdateClubData(newEntry.club, newEntry.numClient, newEntry.departement, newEntry.numDossier, newEntry.taux);

                renderTable(); // Re-render table after modification
                updateSummary(); // Update summary totals (always based on all entries)
                resetForm(); // Reset form after successful operation
            }
            catch (error) {
                console.error("Error saving entry:", error);
                showMessage("Erreur", "Une erreur est survenue lors de l'enregistrement de l'entrée.");
            }
            finally {
                hideLoading(); // Hide loading indicator
            }
        });

        /**
         * Populates the form fields with data from a selected entry for editing.
         * @param {string} id - The ID of the entry to edit.
         */
        window.editLine = (id) => {
            // Automatically switch to form view
            showFormView(); 

            const entryToEdit = entries.find(e => e.id === id);
            if (!entryToEdit) {
                showMessage("Erreur", "Entrée non trouvée pour l'édition.");
                return;
            }

            editDocId = id; // Set the document ID for editing

            // Populate form fields
            q('saison').value = entryToEdit.saison;
            q('club').value = entryToEdit.club;
            q('numClient').value = entryToEdit.numClient;
            q('numDossier').value = entryToEdit.numDossier;
            q('departement').value = entryToEdit.departement;
            q('numPrecommande').value = entryToEdit.numPrecommande;
            q('quantiteApprox').value = entryToEdit.quantiteApprox;
            q('numAR').value = entryToEdit.numAR;
            arPdfInput.value = entryToEdit.arPdf; // Set the value of the arPdf text input
            dateLivraisonInput.value = entryToEdit.dateLivraison; // Populate dateLivraison
            arSigneSelect.value = entryToEdit.arSigne || 'En attente pointage'; // Populate AR, default to 'En attente pointage'
            console.log("editLine: arSigneSelect value set to", arSigneSelect.value); // Debugging
            q('quantiteReelle').value = entryToEdit.quantiteReelle;
            q('montantHT').value = entryToEdit.montantHT;
            q('taux').value = entryToEdit.taux;
            q('commissionPrev').value = entryToEdit.commissionPrev;
            q('commissionPercue').value = entryToEdit.commissionPercue;

            // Set month and year from the 'mois' field (e.g., "2024-07")
            const [year, month] = entryToEdit.mois.split('-');
            q('mois-annee').value = year;
            q('mois-mois').value = month;

            cancelBtn.classList.remove('hidden'); // Show cancel button
            submitBtn.textContent = 'Mettre à jour'; // Change button text to 'Mettre à jour'
            calculateCommissionPrev(); // Recalculate commission for display based on loaded values
        };

        // Handle cancel button click
        on('cancel-btn', 'click', () => {
            resetForm(); // Reset the form
            showMessage("Annulé", "Modification annulée.");
        });

        /**
         * Deletes an entry from the in-memory array after user confirmation.
         * @param {string} id - The ID of the entry to delete.
         */
        window.deleteLine = async (id) => {
            // Show confirmation modal instead of native confirm()
            const confirmDelete = await new Promise(resolve => {
                showMessage(
                    "Confirmer la suppression",
                    "Êtes-vous sûr de vouloir supprimer cette entrée ?",
                    [
                        { text: 'Oui', className: 'bg-red-600 text-white px-6 py-2 rounded-md hover:bg-red-700 mr-2', onClick: () => { hideMessage(); resolve(true); } },
                        { text: 'Non', className: 'bg-gray-300 text-gray-800 px-6 py-2 rounded-md hover:bg-gray-400', onClick: () => { hideMessage(); resolve(false); } }
                        ]
                );
            });

            if (!confirmDelete) return; // If user cancels, stop here

            showLoading(); // Show loading indicator

            try {
                // Filter out the entry to be deleted
                entries = entries.filter(entry => entry.id !== id);
                renderTable(); // Re-render table after deletion
                updateSummary(); // Update summary totals (always based on all entries)
                showMessage("Succès", "Entrée supprimée avec succès !");
            }
            catch (error) {
                console.error("Error deleting entry:", error);
                showMessage("Erreur", "Une erreur est survenue lors de la suppression de l'entrée.");
            }
            finally {
                hideLoading(); // Hide loading indicator
            }
        };

        // --- Local File Operations ---

        /**
         * Saves the current entries data to a JSON file on the user's local machine.
         */
        function saveToFile() {
            if (entries.length === 0) {
                showMessage("Sauvegarde", "Aucune donnée à sauvegarder.");
                return;
            }
            const dataStr = JSON.stringify(entries, null, 2); // Pretty print JSON
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'precommandes.json'; // Default filename
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url); // Clean up the URL object
            showMessage("Sauvegarde", "Données sauvegardées avec succès dans 'precommandes.json'.");
        }

        /**
         * Loads data from a selected JSON file and updates the application's entries.
         * Assigns new UUIDs to entries if they're missing an 'id' field.
         */
        function loadFile() {
            console.log("loadFile function triggered."); // Debugging: Confirm function call
            const file = loadInput.files[0];
            if (!file) {
                showMessage("Chargement", "Veuillez sélectionner un fichier JSON.");
                console.log("No file selected."); // Debugging: No file selected
                return;
            }

            console.log("File selected:", file); // Debugging: Log the selected file object

            const reader = new FileReader();
            reader.onload = (e) => {
                try {
                    const loadedData = JSON.parse(e.target.result);
                    if (!Array.isArray(loadedData)) {
                        throw new Error("Le fichier JSON ne contient pas un tableau valide.");
                    }
                    // Map loaded data, ensuring each item has an 'id' and numeric fields are parsed
                    entries = loadedData.map(item => {
                        const loadedEntry = {
                            id: item.id || crypto.randomUUID(), // Assign new UUID if 'id' is missing
                            ...item,
                            quantiteApprox: Number(item.quantiteApprox) || 0,
                            quantiteReelle: Number(item.quantiteReelle) || 0,
                            montantHT: Number(item.montantHT) || 0,
                            taux: Number(item.taux) || 0,
                            commissionPrev: Number(item.commissionPrev) || 0,
                            commissionPercue: Number(item.commissionPercue) || 0
                        };
                        // Recalculate finalisee for consistency
                        loadedEntry.finalisee = (loadedEntry.commissionPercue > 0 || loadedEntry.arSigne === 'Signé') ? '✔️' : '';
                        return loadedEntry;
                    });

                    // After loading entries, populate clubDataStore from them
                    clubDataStore = []; // Clear current store before populating
                    entries.forEach(entry => {
                        // Pass the taux when populating clubDataStore from loaded entries
                        addOrUpdateClubData(entry.club, entry.numClient, entry.departement, entry.numDossier, entry.taux);
                    });

                    renderTable(); // Update the table with loaded data
                    updateSummary(); // Update summary totals (always based on all entries)
                    showMessage("Chargement", "Données chargées avec succès depuis le fichier.");
                }
                catch (error) {
                    console.error("Error loading file:", error); // Debugging: Log parsing errors
                    showMessage("Erreur de chargement", `Impossible de charger le fichier : ${error.message || 'JSON invalide'}.`);
                }
            };
            reader.onerror = (error) => {
                console.error("FileReader error:", error);
                showMessage("Erreur de lecture de fichier", "Une erreur est survenue lors de la lecture du fichier.");
            };
            reader.readAsText(file);
        }

        // --- Club Data Storage and Auto-fill Functions ---
        // Global variable to store unique club data for auto-filling
        // let clubDataStore = []; // Already declared globally

        // Function to save club data to a JSON file
        function saveClubData() {
            // Clear clubDataStore and repopulate from current entries before saving
            clubDataStore = [];
            entries.forEach(entry => {
                // Pass the taux when populating clubDataStore from current entries for saving
                addOrUpdateClubData(entry.club, entry.numClient, entry.departement, entry.numDossier, entry.taux);
            });

            if (clubDataStore.length === 0) {
                showMessage("Sauvegarde Clubs", "Aucune donnée de club à sauvegarder.");
                return;
            }
            const dataStr = JSON.stringify(clubDataStore, null, 2);
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'clubs_data.json'; // Name of the file
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showMessage("Sauvegarde Clubs", "Données des clubs sauvegardées avec succès dans 'clubs_data.json'.");
        }

        /**
         * Function to load club data from a JSON file.
         * Ensures defaultTaux is handled, defaulting to '0' if not present in the file.
         */
        function loadClubData() {
            const file = loadClubInput.files[0]; // Use the specific input for club data
            if (!file) {
                showMessage("Chargement Clubs", "Veuillez sélectionner un fichier JSON de clubs.");
                return;
            }

            const reader = new FileReader();
            reader.onload = (e) => {
                try {
                    const loadedData = JSON.parse(e.target.result);
                    if (!Array.isArray(loadedData)) {
                        throw new Error("Le fichier JSON ne contient pas un tableau valide.");
                    }
                    // Ensure loaded data has necessary properties and is unique
                    clubDataStore = []; // Clear current store
                    loadedData.forEach(item => {
                        // Ensure all expected properties exist before adding, and default defaultTaux if missing
                        if (item.club && item.numClient && item.departement && item.numDossier) {
                            addOrUpdateClubData(item.club, item.numClient, item.departement, item.numDossier, item.defaultTaux || '0');
                        }
                    });
                    showMessage("Chargement Clubs", "Données des clubs chargées avec succès.");
                }
                catch (error) {
                    console.error("Error loading club data file:", error);
                    showMessage("Erreur de chargement Clubs", `Impossible de charger le fichier : ${error.message || 'JSON invalide'}.`);
                }
            };
            reader.onerror = (error) => {
                console.error("FileReader error for club data:", error);
                showMessage("Erreur de lecture de fichier Clubs", "Une erreur est survenue lors de la lecture du fichier.");
            };
            reader.readAsText(file);
        }

        /**
         * Helper function to add/update unique club data in clubDataStore.
         * Now includes defaultTaux.
         */
        function addOrUpdateClubData(club, numClient, departement, numDossier, defaultTaux) {
            // Ensure values are strings for consistent comparison
            const normalizedClub = club ? club.toString().trim().toLowerCase() : '';
            const normalizedNumClient = numClient ? numClient.toString().trim().toLowerCase() : '';
            const normalizedDepartement = departement ? departement.toString().trim().toLowerCase() : '';
            const normalizedNumDossier = numDossier ? numDossier.toString().trim().toLowerCase() : '';

            const existingIndex = clubDataStore.findIndex(data => 
                data.club.toLowerCase() === normalizedClub && 
                data.numClient.toLowerCase() === normalizedNumClient
            );

            if (existingIndex !== -1) {
                // Update existing entry if department, numDossier, or defaultTaux changed
                if (clubDataStore[existingIndex].departement.toLowerCase() !== normalizedDepartement ||
                    (clubDataStore[existingIndex].numDossier ? clubDataStore[existingIndex].numDossier.toString().toLowerCase() : '') !== normalizedNumDossier ||
                    clubDataStore[existingIndex].defaultTaux !== defaultTaux) {
                    clubDataStore[existingIndex].departement = departement;
                    clubDataStore[existingIndex].numDossier = numDossier;
                    clubDataStore[existingIndex].defaultTaux = defaultTaux; // Update defaultTaux
                }
            } else {
                // Add new unique entry
                clubDataStore.push({ club: club, numClient: numClient, departement: departement, numDossier: numDossier, defaultTaux: defaultTaux }); // Store numDossier and defaultTaux
            }
        }

        /**
         * Auto-fill logic for main form fields AND modal fields.
         * Now also fills the taux field.
         */
        window.autoFillFields = function(clubInput, numClientInput, departementInput, numDossierInput) {
            const currentClub = clubInput?.value?.trim()?.toLowerCase() || '';
            const currentNumClient = numClientInput?.value?.trim()?.toLowerCase() || '';
            const currentDepartement = departementInput?.value?.trim()?.toLowerCase() || '';
            const currentNumDossier = numDossierInput?.value?.trim()?.toLowerCase() || '';

            let foundMatch = null;

            if (currentClub && currentNumClient) {
                foundMatch = clubDataStore.find(data => 
                    data.club.toLowerCase() === currentClub && 
                    data.numClient.toLowerCase() === currentNumClient
                );
            } else if (currentClub) {
                foundMatch = clubDataStore.find(data => data.club.toLowerCase() === currentClub);
            } else if (currentNumClient) {
                foundMatch = clubDataStore.find(data => data.numClient.toLowerCase() === currentNumClient);
            } else if (currentDepartement) {
                   foundMatch = clubDataStore.find(data => data.departement.toLowerCase() === currentDepartement);
            } else if (currentNumDossier) {
                foundMatch = clubDataStore.find(data => (data.numDossier ? data.numDossier.toString().toLowerCase() : '') === currentNumDossier);
            }


            if (foundMatch) {
                // Fill other fields only if they are currently empty
                if (clubInput && !clubInput.value.trim()) clubInput.value = foundMatch.club;
                if (numClientInput && !numClientInput.value.trim()) numClientInput.value = foundMatch.numClient;
                if (departementInput && !departementInput.value.trim()) departementInput.value = foundMatch.departement;
                if (numDossierInput && !numDossierInput.value.trim()) numDossierInput.value = foundMatch.numDossier;
                
                // NEW: Auto-fill the taux field if it's the main form
                if (tauxSelect && foundMatch.defaultTaux !== undefined) {
                    tauxSelect.value = foundMatch.defaultTaux;
                    calculateCommissionPrev(); // Recalculate commission after setting taux
                }
            }
        }


        // --- Statistics and Charts ---
        // Removed chartDept and chartClubs variables as they are no longer used for Chart.js instances

        /**
         * Renders the summary tables for departmental and club statistics.
         */
        function renderStatsTables() { // Renamed from renderCharts
            statsContentDiv.innerHTML = ''; // Clear previous content

            const seasonFilter = filterSaisonStats.value; // Get season filter for stats
            let filteredEntriesForStats = entries;

            if (seasonFilter !== 'all') {
                filteredEntriesForStats = entries.filter(e => e.saison === seasonFilter);
            }

            const deptData = {};
            const clubsData = {};

            filteredEntriesForStats.forEach(e => {
                const montantHT = Number(e.montantHT) || 0;
                const quantite = (Number(e.quantiteReelle) > 0 ? Number(e.quantiteReelle) : Number(e.quantiteApprox)) || 0; // Use real quantity if available, else approximate

                if (e.departement) {
                    if (!deptData[e.departement]) {
                        deptData[e.departement] = { totalMontant: 0, totalQuantite: 0 };
                    }
                    deptData[e.departement].totalMontant += montantHT;
                    deptData[e.departement].totalQuantite += quantite;
                }
                if (e.club) {
                    if (!clubsData[e.club]) {
                        clubsData[e.club] = { totalMontant: 0, totalQuantite: 0 };
                    }
                    clubsData[e.club].totalMontant += montantHT;
                    clubsData[e.club].totalQuantite += quantite;
                }
            });

            if (filteredEntriesForStats.length === 0 || (Object.keys(deptData).length === 0 && Object.keys(clubsData).length === 0)) {
                noStatsDataMessage.classList.remove('hidden');
                return;
            } else {
                noStatsDataMessage.classList.add('hidden');
            }

            const sortedDept = Object.entries(deptData).sort(([, a], [, b]) => b.totalMontant - a.totalMontant);
            const sortedClubs = Object.entries(clubsData).sort(([, a], [, b]) => b.totalMontant - a.totalMontant);

            // Generate HTML for Department Summary Table
            let deptHtml = `<div class="stats-table-container">
                                <h3 class="text-2xl font-bold mb-6 text-blue-800 text-center">Montant HT et Quantités par Département</h3>
                                <table class="w-full border-collapse">
                                    <thead>
                                        <tr>
                                            <th class="text-left py-2 px-4 bg-blue-100 rounded-tl-lg">Département</th>
                                            <th class="text-right py-2 px-4 bg-blue-100">Montant HT (€)</th>
                                            <th class="text-right py-2 px-4 bg-blue-100 rounded-tr-lg">Quantités</th>
                                        </tr>
                                    </thead>
                                    <tbody>`;
            if (sortedDept.length > 0) {
                sortedDept.forEach(([dept, data]) => {
                    deptHtml += `<tr>
                                    <td class="text-left py-2 px-4 border-b border-gray-200">Dépt ${dept}</td>
                                    <td class="text-right py-2 px-4 border-b border-gray-200">${data.totalMontant.toFixed(2)}</td>
                                    <td class="text-right py-2 px-4 border-b border-gray-200">${data.totalQuantite}</td>
                                </tr>`;
                });
            } else {
                deptHtml += `<tr><td colspan="3" class="text-center py-4 text-gray-500">Aucune donnée de département.</td></tr>`;
            }
            deptHtml += `       </tbody>
                            </table>
                        </div>`;
            statsContentDiv.insertAdjacentHTML('beforeend', deptHtml);

            // Generate HTML for Club Summary Table
            let clubsHtml = `<div class="stats-table-container">
                                <h3 class="text-2xl font-bold mb-6 text-blue-800 text-center">Montant HT et Quantités par Club</h3>
                                <table class="w-full border-collapse">
                                    <thead>
                                        <tr>
                                            <th class="text-left py-2 px-4 bg-blue-100 rounded-tl-lg">Club</th>
                                            <th class="text-right py-2 px-4 bg-blue-100">Montant HT (€)</th>
                                            <th class="text-right py-2 px-4 bg-blue-100 rounded-tr-lg">Quantités</th>
                                        </tr>
                                    </thead>
                                    <tbody>`;
            if (sortedClubs.length > 0) {
                sortedClubs.forEach(([club, data]) => {
                    clubsHtml += `<tr>
                                    <td class="text-left py-2 px-4 border-b border-gray-200">${club}</td>
                                    <td class="text-right py-2 px-4 border-b border-gray-200">${data.totalMontant.toFixed(2)}</td>
                                    <td class="text-right py-2 px-4 border-b border-gray-200">${data.totalQuantite}</td>
                                </tr>`;
                });
            } else {
                clubsHtml += `<tr><td colspan="3" class="text-center py-4 text-gray-500">Aucune donnée de club.</td></tr>`;
            }
            clubsHtml += `       </tbody>
                            </table>
                        </div>`;
            statsContentDiv.insertAdjacentHTML('beforeend', clubsHtml);
        }

        // --- Initial Setup and Event Listeners ---
        document.addEventListener('DOMContentLoaded', () => {
            // Load data from localStorage on initial load
            const storedEntries = localStorage.getItem('preorderEntries');
            if (storedEntries) {
                entries = JSON.parse(storedEntries);
            }
            const storedClubData = localStorage.getItem('clubDataStore');
            if (storedClubData) {
                // Ensure defaultTaux is parsed correctly, providing a default if missing from old data
                clubDataStore = JSON.parse(storedClubData).map(club => ({
                    ...club,
                    defaultTaux: club.defaultTaux !== undefined ? club.defaultTaux : '0'
                }));
            }

            resetForm(); // Reset form on page load
            renderTable(); // Initial render of the table (will be empty)
            updateSummary(); // Initial update of summary (will be zeros)

            // Set initial view
            showFormView(); // Start on the form page

            // Attach navigation button listeners
            on('show-form-btn', 'click', showFormView);
            on('show-list-btn', 'click', showListView);
            on('back-to-form-btn', 'click', showFormView); // Now global

            // Event listener for season filter changes
            on('filter-saison', 'change', () => {
                renderTable(); // Re-render table based on new filter
                updateSummary(); // Update summary based on new filter
                // No need to renderCharts here, as showStatsView will handle it
            });

            // Event listener for client number filter changes
            on('filter-numClient', 'input', () => {
                renderTable(); // Re-render table based on new filter
                // No need to renderCharts here, as showStatsView will handle it
            });

            // Event listener for club name filter changes
            on('filter-clubName', 'input', () => {
                renderTable(); // Re-render table based on new filter
                // No need to renderCharts here, as showStatsView will handle it
            });

            // Event listener for dossier status filter changes
            on('filter-dossierStatus', 'change', () => {
                renderTable(); // Re-render table based on new filter
                // No need to renderCharts here, as showStatsView will handle it
            });

            // Event listeners for local file operations
            on('save-file-btn', 'click', saveToFile);
            on('load-file-btn', 'click', () => {
                console.log("Load File button clicked. Triggering hidden input click.");
                loadInput.click();
            });
            on('load-input', 'change', (event) => {
                console.log("Load Input 'change' event triggered.");
                if (event.target.files && event.target.files.length > 0) {
                    console.log("File(s) selected, calling loadFile().");
                    loadFile();
                } else {
                    console.log("No file selected in the input.");
                }
            });

            // Club data management listeners
            on('save-club-data-btn', 'click', saveClubData);
            on('load-club-data-btn', 'click', () => loadClubInput.click());
            on('load-club-input', 'change', loadClubData);

            // Auto-fill listeners for main form fields
            on('club', 'input', () => autoFillFields(clubInputMain, numClientInputMain, departementInputMain, numDossierInputMain));
            on('numClient', 'input', () => autoFillFields(clubInputMain, numClientInputMain, departementInputMain, numDossierInputMain));
            on('departement', 'input', () => autoFillFields(clubInputMain, numClientInputMain, departementInputMain, numDossierInputMain));
            on('numDossier', 'input', () => autoFillFields(clubInputMain, numClientInputMain, departementInputMain, numDossierInputMain));


            // Event listener for "Afficher Stats" button
            on('showStats-btn', 'click', showStatsView);
            // NEW: Event listener for season filter in stats section
            on('filter-saison-stats', 'change', renderStatsTables);


            // Duplicate Last Entry Button Listener
            on('duplicate-entry-btn', 'click', duplicateLastEntry);

            // Preorder Modal Buttons
            on('request-preorder-btn', 'click', openPreorderModal);
            on('cancel-preorder-modal-btn', 'click', closePreorderModal);
            on('send-preorder-email-btn', 'click', sendPreorderEmail);

            // Auto-fill listeners for preorder modal fields
            // Pass null for departementInput and numDossierInput as they are not in the modal for auto-fill based on club/client
            on('modalClubName', 'input', () => autoFillFields(modalClubName, modalClientNum, null, null));
            on('modalClientNum', 'input', () => autoFillFields(modalClubName, modalClientNum, null, null));
        });

        // Global navigation functions
        function showFormView() {
            formViewContainer.classList.remove('hidden');
            listViewContainer.classList.add('hidden');
            statsSection.classList.add('hidden'); // Hide stats section
            backToFormBtn.classList.add('hidden'); // Hide back button
            resetForm();
        }

        function showListView() {
            formViewContainer.classList.add('hidden');
            listViewContainer.classList.remove('hidden');
            statsSection.classList.add('hidden'); // Hide stats section
            backToFormBtn.classList.remove('hidden'); // Show back button
            renderTable();
        }

        function showStatsView() {
            formViewContainer.classList.add('hidden');
            listViewContainer.classList.add('hidden');
            statsSection.classList.remove('hidden'); // Show stats section
            backToFormBtn.classList.remove('hidden'); // Show back button
            renderStatsTables(); // Call the new function to render tables
        }

        // Function to duplicate the last entry
        function duplicateLastEntry() {
            if (entries.length === 0) {
                showMessage("Information", "Aucune entrée précédente à dupliquer.");
                return;
            }

            const lastEntry = entries[entries.length - 1];
            
            // Reset form first to clear any existing input or edit state
            resetForm(); 

            // Populate common fields from the last entry
            q('saison').value = lastEntry.saison;
            q('club').value = lastEntry.club;
            q('numClient').value = lastEntry.numClient;
            q('numDossier').value = lastEntry.numDossier; // Keep N° Dossier as it might be common for a club
            q('departement').value = lastEntry.departement;
            q('mois-annee').value = lastEntry.mois.split('-')[0];
            q('mois-mois').value = lastEntry.mois.split('-')[1];

            // Set the taux from the last entry if available
            if (lastEntry.taux !== undefined) {
                q('taux').value = lastEntry.taux;
                calculateCommissionPrev();
            }

            // Specific AR fields are reset by resetForm() and not duplicated
            // Set editDocId to null to ensure it's treated as a new entry
            editDocId = null;
            submitBtn.textContent = 'Enregistrer'; // Ensure button text is "Enregistrer"

            showMessage("Duplication", "Dernière entrée dupliquée. Veuillez saisir les détails du nouvel AR.");
        }

        // Preorder Modal Functions
        function openPreorderModal() {
            // Hide the main content container to make the modal appear "alone"
            mainContentContainer.classList.add('hidden'); 
            preorderRequestModal.classList.remove('hidden');
            // Pre-fill modal fields from main form's current values
            modalPreorderNum.value = '';
            modalQtyHauts.value = '0';
            modalQtyBas.value = '0';
            modalQtyAcc.value = '0';
            modalDateEssayage.value = '';
            modalDateDepartSouhaitee.value = '';
            modalNotesReservation.value = '';
            doNotSendEmailCheckbox.checked = false;

            // Pre-fill Club Name and Client Number in modal from main form
            modalClubName.value = clubInputMain.value; // Get from main form
            modalClientNum.value = numClientInputMain.value; // Get from main form

            // Trigger auto-fill for the modal fields in case club/client were already in clubDataStore
            // Note: autoFillFields for modal does not fill taux, as it's not a field in the preorder modal
            autoFillFields(modalClubName, modalClientNum, null, null);
        }

        window.closePreorderModal = function() { // Made global
            preorderRequestModal.classList.add('hidden');
            resetPreorderModalFields(); // Reset modal fields when closing
            // Show the main content container again
            mainContentContainer.classList.remove('hidden');
        }

        // NEW: Function to reset preorder modal fields
        function resetPreorderModalFields() {
            modalPreorderNum.value = '';
            modalQtyHauts.value = '0';
            modalQtyBas.value = '0';
            modalQtyAcc.value = '0';
            modalDateEssayage.value = '';
            modalDateDepartSouhaitee.value = '';
            modalNotesReservation.value = '';
            doNotSendEmailCheckbox.checked = false;
            modalClubName.value = '';
            modalClientNum.value = '';
        }

        function sendPreorderEmail() {
            const clubName = modalClubName.value || 'Non spécifié'; // Get from modal field
            const clientNum = modalClientNum.value || 'Non spécifié'; // Get from modal field
            const preorderNum = modalPreorderNum.value || 'N/A';
            const qtyHauts = modalQtyHauts.value || '0';
            const qtyBas = modalQtyBas.value || '0';
            const qtyAcc = modalQtyAcc.value || '0';
            const dateEssayage = modalDateEssayage.value ? formatDate(modalDateEssayage.value) : 'Non spécifiée';
            const dateDepartSouhaitee = modalDateDepartSouhaitee.value ? formatDate(modalDateDepartSouhaitee.value) : 'Non spécifiée';

            const notes = modalNotesReservation.value || 'Aucune note.';

            // Create a new entry in the main data based on preorder modal input
            const newPreorderEntry = {
                id: crypto.randomUUID(),
                saison: q('saison').value, // Get from main form
                club: clubName,
                numClient: clientNum,
                numDossier: q('numDossier').value, // Get from main form
                departement: q('departement').value, // Get from main form
                extraData: { // Store extra data not directly mapped to main form fields
                    qtyHauts: Number(qtyHauts),
                    qtyBas: Number(qtyBas),
                    qtyAcc: Number(qtyAcc),
                    dateEssayage: modalDateEssayage.value,
                    notesReservation: notes
                },
                numPrecommande: preorderNum,
                quantiteApprox: (Number(qtyHauts) + Number(qtyBas) + Number(qtyAcc)),
                numAR: '', // AR is not set yet for a new preorder
                arPdf: '',
                dateLivraison: modalDateDepartSouhaitee.value, // Use Date départ souhaitée as dateLivraison
                arSigne: 'Précommande fixée', // Set AR status to "Précommande fixée"
                quantiteReelle: 0,
                montantHT: 0,
                taux: 0, // Default taux for new preorder entry
                commissionPrev: 0,
                commissionPercue: 0,
                mois: q('mois-annee').value + '-' + q('mois-mois').value, // Get from main form
                preorderNotes: notes // Store notes from preorder modal
            };
            // Calculate finalisee for the new preorder entry
            newPreorderEntry.finalisee = (newPreorderEntry.commissionPercue > 0 || newPreorderEntry.arSigne === 'Signé') ? '✔️' : '';


            entries.push(newPreorderEntry); // Add to main entries array
            // Update club data store after saving an entry, using a default taux of 0 for preorders
            addOrUpdateClubData(newPreorderEntry.club, newPreorderEntry.numClient, newPreorderEntry.departement, newPreorderEntry.numDossier, '0');

            renderTable(); // Update the table
            updateSummary(); // Update summary totals

            if (doNotSendEmailCheckbox.checked) {
                showMessage("Précommande Enregistrée", "La demande de précommande a été enregistrée sans envoi d'email. Une nouvelle entrée a été ajoutée à la liste.");
                console.log("Précommande enregistrée (email non envoyé):", newPreorderEntry);
            } else {
                const subject = `Demande de Précommande - Club: ${clubName} (N° Client: ${clientNum})`;
                let body = `
Bonjour,

Veuillez trouver ci-dessous les détails pour une demande de précommande :

Club : ${clubName} (N° Client : ${clientNum})
N° Précommande (si connu) : ${preorderNum}

Quantité prévisionnelle :
  - Hauts : ${qtyHauts}
  - Bas : ${qtyBas}
  - Acc. : ${qtyAcc}
`;
                if (modalDateEssayage.value) {
                    body += `
Date d'essayage (si prévue) : ${dateEssayage}`;
                }
                if (modalDateDepartSouhaitee.value) {
                    body += `
Date départ souhaitée : ${dateDepartSouhaitee}`;
                }
                body += `

Notes pour la demande de réservation :
${notes}

Cordialement,
`;

                const mailtoLink = `mailto:aude@noret.com?subject=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}`;
                window.location.href = mailtoLink; // This will trigger the email client
                showMessage("Email Prêt", "Votre client de messagerie devrait s'ouvrir avec l'email pré-rempli. Une nouvelle entrée a été ajoutée à la liste.");
            }
            closePreorderModal(); // Close and reset modal fields
            // Automatically switch to form view and load the newly created entry for further completion
            window.editLine(newPreorderEntry.id);
        }

        // NEW: Function to export current table data to PDF
        function exportPdf() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('landscape'); // 'landscape' for wider tables

            const table = q('table-commissions');
            // Dynamically get headers from the visible table, excluding "PDF AR"
            const headers = Array.from(table.querySelectorAll('thead th'))
                                .filter(th => th.textContent.trim() !== 'PDF AR' && th.textContent.trim() !== 'Actions') // Exclude PDF AR and Actions
                                .map(th => th.textContent.trim());
            
            const body = Array.from(table.querySelectorAll('tbody tr')).map(row => {
                const rowData = Array.from(row.querySelectorAll('td')).map((td, index) => {
                    // Get all headers from the HTML table to correctly map data to original column positions
                    const allTableHeaders = Array.from(table.querySelectorAll('thead th')).map(th => th.textContent.trim());
                    const currentHeader = allTableHeaders[index];

                    // Skip "PDF AR" and "Actions" columns
                    if (currentHeader === 'PDF AR' || currentHeader === 'Actions') {
                        return null; 
                    }

                    // Handle Club column for grouped display in PDF by getting original club name
                    if (currentHeader === 'Club' && td.textContent.trim() === '') {
                        const rowElement = td.closest('tr');
                        const editButton = rowElement ? rowElement.querySelector('button[onclick*="editLine"]') : null;
                        const entryId = editButton ? editButton.onclick.toString().match(/'([^']+)'/)[1] : null;
                        const originalEntry = entries.find(e => e.id === entryId);
                        return originalEntry ? originalEntry.club : '';
                    }

                    // For "Date de livraison" and "AR" (numAR), ensure we get the correct data
                    // The `renderTable` function now correctly places `formatDate(e.dateLivraison)` and `e.arSigne`
                    // So we just need to return the text content of the cell.
                    return td.textContent.trim();
                }).filter(item => item !== null); // Filter out the nulls from excluded columns
                return rowData;
            });

            if (body.length === 0) {
                showMessage("Exportation PDF", "Aucune donnée à exporter en PDF.");
                return;
            }

            // Define column widths for better spacing
            // These widths are approximate and may need fine-tuning based on actual data
            // The number of 'auto' entries should match the number of 'headers' after filtering
            const columnWidths = [
                'auto', // Club
                'auto', // N° Client
                'auto', // N° Dossier
                'auto', // Dépt
                'auto', // Préco
                'auto', // Qté approx
                'auto', // AR (Num AR)
                'auto', // Date de livraison (index 7 in the filtered headers)
                'auto', // AR (Signé status) (index 8 in the filtered headers)
                'auto', // Statut Dossier
                'auto', // Qté commandée
                'auto', // Montant HT
                'auto', // Commission prévue
                'auto', // Commission perçue
                'auto', // Écart
                'auto', // Mois pointage
                'auto'  // Finalisée
            ];

            doc.autoTable({
                head: [headers],
                body: body,
                startY: 20,
                styles: {
                    font: 'helvetica',
                    fontSize: 8,
                    cellPadding: 2,
                    valign: 'middle',
                    overflow: 'linebreak', // Ensure text wraps within cells
                    halign: 'left' // Default text alignment
                },
                headStyles: {
                    fillColor: [55, 65, 81], // Tailwind gray-700 equivalent
                    textColor: [255, 255, 255],
                    fontStyle: 'bold',
                    halign: 'center' // Center headers
                },
                columnStyles: {
                    // Align specific columns to the right for numbers
                    // Adjust indices based on the *new* headers array (Saison, PDF AR, Actions removed)
                    1: { halign: 'right' },  // N° Client
                    2: { halign: 'right' },  // N° Dossier
                    5: { halign: 'center' }, // Qté approx
                    7: { halign: 'center' }, // Date de livraison
                    10: { halign: 'right' }, // Qté commandée
                    11: { halign: 'right' }, // Montant HT
                    12: { halign: 'right' }, // Commission prévue
                    13: { halign: 'right' }, // Commission perçue
                    14: { halign: 'right' }, // Écart
                    16: { halign: 'center' } // Finalisée
                },
                // Set column widths
                columnWidths: columnWidths,
                didParseCell: function(data) {
                    // Adjust index for Écart column (now 14)
                    if (data.column.index === 14 && parseFloat(data.cell.text[0]) < 0) {
                        data.cell.styles.textColor = [220, 38, 38]; // Tailwind red-600
                    } else if (data.column.index === 14 && parseFloat(data.cell.text[0]) > 0) {
                        data.cell.styles.textColor = [5, 150, 105]; // Tailwind green-600
                    }
                }
            });

            doc.save('Precommandes_Commissions.pdf');
            showMessage("Exportation PDF", "Données exportées avec succès en PDF.");
        }

        // Event listener for PDF export button
        on('export-pdf-btn', 'click', exportPdf);
    </script>
</body>
</html>
