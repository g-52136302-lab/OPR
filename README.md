<html lang="ms" class="h-full bg-slate-100">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Sistem Penjana OPR KPM + AI</title>

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <!-- Google Fonts: Inter & Montserrat -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Montserrat:wght@600;700;800&display=swap" rel="stylesheet">

  <style>
    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background-color: #f1f5f9;
      -webkit-tap-highlight-color: transparent;
    }

    h1, h2, h3, .font-heading {
      font-family: 'Montserrat', sans-serif;
    }

    /* Strict 1-Page A4 Container Styling for Preview */
    .a4-preview {
      width: 210mm;
      height: 297mm;
      min-height: 297mm;
      max-height: 297mm;
      background: white;
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
      box-sizing: border-box;
      position: relative;
      overflow: hidden; /* Prevent spillover */
      padding: 6mm 8mm;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
    }

    /* Responsive scaling for preview on smaller desktop viewports */
    @media (max-width: 1400px) and (min-width: 1024px) {
      .a4-preview-wrapper {
        transform: scale(0.8);
        transform-origin: top center;
        margin-bottom: -50px;
      }
    }

    /* Touch target optimization */
    input, select, textarea, button {
      min-height: 38px;
    }

    input[type="checkbox"], input[type="radio"] {
      min-height: auto;
    }

    /* ABSOLUTE STRICT 1-PAGE PRINT ENGINE RULES */
    @media print {
      @page {
        size: A4 portrait;
        margin: 0mm !important;
      }
      
      html, body {
        width: 210mm !important;
        height: 297mm !important;
        max-height: 297mm !important;
        margin: 0 !important;
        padding: 0 !important;
        background: white !important;
        overflow: hidden !important;
        -webkit-print-color-adjust: exact !important;
        print-color-adjust: exact !important;
      }

      /* Hide everything in the body by default */
      body * {
        visibility: hidden !important;
      }

      /* Make ONLY the print area and its children visible and positioned at top left */
      #opr-preview-container, #opr-preview-container * {
        visibility: visible !important;
      }

      #opr-preview-container {
        position: absolute !important;
        left: 0 !important;
        top: 0 !important;
        width: 210mm !important;
        height: 297mm !important;
        margin: 0 !important;
        padding: 0 !important;
        transform: none !important;
        overflow: hidden !important;
      }

      .a4-preview {
        position: absolute !important;
        left: 0 !important;
        top: 0 !important;
        width: 210mm !important;
        height: 297mm !important;
        min-height: 297mm !important;
        max-height: 297mm !important;
        margin: 0 !important;
        padding: 6mm 8mm !important;
        border: none !important;
        box-shadow: none !important;
        box-sizing: border-box !important;
        page-break-after: avoid !important;
        page-break-before: avoid !important;
        page-break-inside: avoid !important;
        overflow: hidden !important;
        display: flex !important;
        flex-direction: column !important;
        justify-content: space-between !important;
      }

      .no-print {
        display: none !important;
      }
    }

    /* Custom Scrollbars */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #f1f5f9;
    }
    ::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 4px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #94a3b8;
    }

    /* Utility for pre-wrap text in preview */
    .whitespace-pre-line {
      white-space: pre-line;
    }
  </style>
</head>
<body class="text-slate-800 antialiased h-full flex flex-col min-h-screen">

  <!-- Custom Modal (Replaces alert/confirm) -->
  <div id="custom-modal" class="fixed inset-0 z-[100] hidden flex items-center justify-center bg-slate-900/60 backdrop-blur-sm transition-opacity">
    <div class="bg-white rounded-xl shadow-2xl p-6 max-w-sm w-full mx-4 transform transition-all">
      <div class="flex items-center gap-3 mb-3">
        <div id="modal-icon" class="p-2 bg-blue-100 text-blue-600 rounded-full">
          <i data-lucide="info" class="w-5 h-5"></i>
        </div>
        <h3 id="modal-title" class="text-lg font-bold text-slate-900">Mesej</h3>
      </div>
      <p id="modal-message" class="text-sm text-slate-600 mb-6 leading-relaxed">...</p>
      <div class="flex justify-end gap-3" id="modal-actions">
        <button id="modal-btn-cancel" class="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 hover:bg-slate-200 rounded-lg hidden transition">Batal</button>
        <button id="modal-btn-ok" class="px-4 py-2 text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 rounded-lg transition shadow-sm">OK</button>
      </div>
    </div>
  </div>

  <!-- Mobile Navigation Drawer Backdrop -->
  <div id="mobile-drawer-backdrop" onclick="toggleMobileDrawer(false)" class="no-print fixed inset-0 bg-slate-900/60 z-50 hidden transition-opacity duration-300 opacity-0"></div>

  <!-- Mobile Navigation Drawer Sidebar -->
  <aside id="mobile-drawer" class="no-print fixed top-0 left-0 bottom-0 w-4/5 max-w-xs bg-slate-900 text-white z-50 transform -translate-x-full transition-transform duration-300 ease-in-out flex flex-col justify-between p-5 shadow-2xl">
    <div class="space-y-6">
      <div class="flex items-center justify-between border-b border-slate-800 pb-4">
        <div class="flex items-center space-x-3">
          <div class="bg-blue-600 p-2 rounded-lg text-white shadow">
            <i data-lucide="file-text" class="w-5 h-5"></i>
          </div>
          <div>
            <h2 class="font-bold text-sm tracking-tight">OPR Generator</h2>
            <span class="text-[10px] text-blue-400 font-semibold uppercase tracking-wider">Edisi KPM + AI</span>
          </div>
        </div>
        <button onclick="toggleMobileDrawer(false)" class="p-2 text-slate-400 hover:text-white rounded-lg">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>

      <nav class="space-y-2">
        <button onclick="switchTab('form'); toggleMobileDrawer(false);" class="w-full flex items-center space-x-3 px-3 py-2.5 rounded-lg text-xs font-semibold bg-blue-600 text-white shadow">
          <i data-lucide="edit-3" class="w-4 h-4"></i>
          <span>Borang Input Laporan</span>
        </button>
        <button onclick="switchTab('preview'); toggleMobileDrawer(false);" class="w-full flex items-center space-x-3 px-3 py-2.5 rounded-lg text-xs font-semibold text-slate-300 hover:bg-slate-800 transition">
          <i data-lucide="eye" class="w-4 h-4"></i>
          <span>Pratonton Dokumen A4</span>
        </button>
        <button type="button" onclick="autoFillWholeFormAI(); toggleMobileDrawer(false);" class="w-full flex items-center space-x-3 px-3 py-2.5 rounded-lg text-xs font-semibold text-emerald-400 hover:bg-slate-800 transition">
          <i data-lucide="sparkles" class="w-4 h-4"></i>
          <span>Jana Kandungan via AI</span>
        </button>
        <button onclick="window.print(); toggleMobileDrawer(false);" class="w-full flex items-center space-x-3 px-3 py-2.5 rounded-lg text-xs font-semibold text-amber-400 hover:bg-slate-800 transition">
          <i data-lucide="printer" class="w-4 h-4"></i>
          <span>Cetak / Muat Turun PDF</span>
        </button>
      </nav>
    </div>

    <div class="border-t border-slate-800 pt-4 space-y-2">
      <button onclick="promptResetForm(); toggleMobileDrawer(false);" class="w-full flex items-center justify-center space-x-2 px-3 py-2 text-xs font-medium text-slate-400 hover:text-white bg-slate-800 rounded-lg">
        <i data-lucide="rotate-ccw" class="w-4 h-4"></i>
        <span>Set Semula Borang</span>
      </button>
      <p class="text-[10px] text-center text-slate-500">Standard OPR Sekolah KPM v2.5</p>
    </div>
  </aside>

  <!-- Top Navigation Bar -->
  <header class="no-print bg-slate-900 text-white border-b border-slate-800 sticky top-0 z-40 shadow-md">
    <div class="max-w-[1700px] mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
      
      <!-- Left: Mobile Hamburger & Logo -->
      <div class="flex items-center space-x-3">
        <button onclick="toggleMobileDrawer(true)" class="lg:hidden p-2 rounded-lg text-slate-300 hover:bg-slate-800 focus:outline-none" aria-label="Buka Menu Navigation">
          <i data-lucide="menu" class="w-6 h-6"></i>
        </button>

        <div class="flex items-center space-x-3">
          <div class="bg-blue-600 p-2 rounded-lg text-white shadow hidden sm:flex">
            <i data-lucide="file-text" class="w-5 h-5"></i>
          </div>
          <div>
            <h1 class="font-bold text-base sm:text-lg tracking-tight flex items-center gap-2">
              OPR 
              <span class="bg-blue-500/30 text-blue-300 text-[10px] sm:text-xs px-2 py-0.5 rounded-full border border-blue-400/30">Edisi AI</span>
            </h1>
            <p class="text-[11px] text-slate-400 hidden md:block">Sistem Laporan Satu Halaman Standard Sekolah</p>
          </div>
        </div>
      </div>

      <!-- Right Action Toolbar -->
      <div class="flex items-center space-x-2">
        <button type="button" onclick="autoFillWholeFormAI()" id="btn-ai-all" class="px-3 py-2 text-xs font-semibold bg-emerald-600 hover:bg-emerald-500 text-white rounded-lg shadow flex items-center gap-1.5 transition active:scale-95">
          <i data-lucide="sparkles" class="w-4 h-4"></i>
          <span class="hidden sm:inline">Jana Semua via</span> AI
        </button>

        <button onclick="window.print()" class="px-3.5 py-2 text-xs font-semibold bg-blue-600 hover:bg-blue-500 text-white rounded-lg shadow flex items-center gap-1.5 transition active:scale-95">
          <i data-lucide="printer" class="w-4 h-4"></i>
          <span class="hidden sm:inline">Cetak /</span> PDF
        </button>

        <button onclick="promptResetForm()" class="hidden sm:flex px-3 py-2 text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg border border-slate-700 transition" title="Set Semula Borang">
          <i data-lucide="rotate-ccw" class="w-4 h-4"></i>
        </button>
      </div>
    </div>
  </header>

  <!-- Mobile View Switcher Tabs -->
  <div class="no-print lg:hidden bg-slate-200 border-b border-slate-300 p-1.5 flex justify-center gap-2 text-xs font-medium sticky top-16 z-30 shadow-sm">
    <button id="btn-tab-form" onclick="switchTab('form')" class="flex-1 py-2 text-center rounded-lg bg-blue-600 text-white font-semibold transition shadow-sm">
      <i data-lucide="edit-3" class="w-3.5 h-3.5 inline mr-1"></i> 1. Borang Input
    </button>
    <button id="btn-tab-preview" onclick="switchTab('preview')" class="flex-1 py-2 text-center rounded-lg text-slate-700 hover:bg-slate-300 transition">
      <i data-lucide="eye" class="w-3.5 h-3.5 inline mr-1"></i> 2. Pratonton A4
    </button>
  </div>

  <!-- Main Grid Content Workspace -->
  <main class="flex-1 max-w-[1700px] w-full mx-auto p-3 sm:p-5 lg:p-6 grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">

    <!-- Left Column: Input Form -->
    <div id="opr-form-container" class="no-print lg:col-span-6 space-y-5 w-full">

      <form id="opr-form" onsubmit="event.preventDefault();" class="bg-white rounded-xl shadow-sm border border-slate-200 divide-y divide-slate-100 overflow-hidden">

        <!-- Section 1: Header & Sekolah -->
        <div class="p-4 sm:p-5 space-y-4">
          <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
            <i data-lucide="building-2" class="w-4 h-4 text-blue-600"></i>
            1. Maklumat Asas & Tetapan Header
          </h2>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div>
              <label class="block text-xs font-medium text-slate-700 mb-1">Nama Sekolah / Institusi</label>
              <input type="text" id="input-sekolah" oninput="updateOPR()" value="SEKOLAH JENIS KEBANGSAAN (CINA) PEI HUA" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <div>
              <label class="block text-xs font-medium text-slate-700 mb-1">Pautan Logo Sekolah (URL)</label>
              <input type="text" id="input-logo-url" oninput="updateOPR()" value="https://app.pandai.org/uploads/school/NBC4046.jpg" placeholder="https://example.com/logo-sekolah.png" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>
          </div>
        </div>

        <!-- Section 2: Metadata Program -->
        <div class="p-4 sm:p-5 space-y-4">
          <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
            <i data-lucide="calendar" class="w-4 h-4 text-blue-600"></i>
            2. Maklumat Program / Aktiviti
          </h2>

          <div class="space-y-3">
            <div>
              <label class="block text-xs font-medium text-slate-700 mb-1">Nama Program / Aktiviti <span class="text-red-500">*</span></label>
              <input type="text" id="input-program" oninput="updateOPR()" value="BENGKEL PEMERKASAAN STEM & INOVASI SAINS 2026" class="w-full text-xs font-semibold px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Unit / Sektor Anjuran</label>
                <select id="input-unit" onchange="updateOPR()" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
                  <option value="Kurikulum" selected>Kurikulum</option>
                  <option value="Hal Ehwal Murid (HEM)">Hal Ehwal Murid (HEM)</option>
                  <option value="Kokurikulum">Kokurikulum</option>
                  <option value="Pentadbiran">Pentadbiran</option>
                  <option value="PIBG / KSIB">PIBG / KSIB</option>
                  <option value="Formasi STEM">Formasi STEM</option>
                  <option value="Bimbingan & Kaunseling">Bimbingan & Kaunseling</option>
                </select>
              </div>
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Penganjur Utama</label>
                <select id="input-anjuran" onchange="updateOPR()" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
                  <option value="Peringkat Sekolah" selected>Sekolah</option>
                  <option value="Panitia Sains & Matematik">Panitia Mata Pelajaran</option>
                  <option value="Kelab & Persatuan">Kelab & Persatuan</option>
                  <option value="Unit Beruniform">Unit Beruniform</option>
                  <option value="Sukan & Permainan">Sukan & Permainan</option>
                  <option value="Pejabat Pendidikan Daerah (PPD)">PPD</option>
                  <option value="Jabatan Pendidikan Negeri (JPN)">JPN</option>
                </select>
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
              <!-- Calendar Date Selection -->
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Tarikh Program</label>
                <input type="date" id="input-tarikh-date" onchange="updateOPR()" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
              </div>
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Masa</label>
                <input type="text" id="input-masa" oninput="updateOPR()" value="8.30 PAGI - 1.00 TENGAH HARI" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              </div>
              <!-- Location Dropdown with Custom Input Toggle -->
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Tempat Program</label>
                <select id="input-tempat-select" onchange="handleTempatChange(); updateOPR();" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
                  <option value="STEM Lab" selected>STEM Lab</option>
                  <option value="Kelas Pemulihan">Kelas Pemulihan</option>
                  <option value="Dewan Sekolah">Dewan Sekolah</option>
                  <option value="Padang Sekolah">Padang</option>
                  <option value="Bilik Mesyuarat">Bilik Mesyuarat</option>
                  <option value="Pejabat Guru">Pejabat Guru</option>
                  <option value="custom">Lain-lain (Taip Sendiri)</option>
                </select>
              </div>
            </div>

            <div id="custom-tempat-container" class="hidden">
              <label class="block text-xs font-medium text-slate-700 mb-1">Nyatakan Tempat Tersuai</label>
              <input type="text" id="input-tempat-custom" oninput="updateOPR()" placeholder="Contoh: Pusat Sumber Sekolah" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Kumpulan Sasaran / Kehadiran</label>
                <input type="text" id="input-sasaran" oninput="updateOPR()" value="60 Murid Tahun 4 & 5 serta 8 Guru Pembimbing" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              </div>
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Tetamu Kehormat / VIP (Jika ada)</label>
                <input type="text" id="input-perasmi" oninput="updateOPR()" value="En. Rosli bin Ahmad (Pegawai SISC+ PPD)" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              </div>
            </div>
          </div>
        </div>

        <!-- Section 3: Content Inputs with Contextual AI Prompt Buttons -->
        <div class="p-4 sm:p-5 space-y-4">
          <div class="flex items-center justify-between">
            <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
              <i data-lucide="align-left" class="w-4 h-4 text-blue-600"></i>
              3. Pengisian Laporan & Ulasan Program
            </h2>
            <span class="text-[10px] text-slate-500 hidden sm:inline">Klik ✦ Cadangan AI untuk jana automatik</span>
          </div>

          <!-- 1. Objektif -->
          <div class="space-y-1 relative">
            <div class="flex justify-between items-center">
              <label class="text-xs font-semibold text-slate-700">1. Objektif Program</label>
              <button type="button" onclick="generateAISpecific('objektif')" class="ai-btn text-[11px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-1 bg-blue-50 hover:bg-blue-100 px-2 py-1 rounded-md transition active:scale-95">
                <i data-lucide="sparkles" class="w-3.5 h-3.5 text-blue-500"></i> ✦ Cadangan AI
              </button>
            </div>
            <textarea id="input-objektif" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">Meningkatkan minat murid dalam bidang Sains, Teknologi, Kejuruteraan dan Matematik.&#10;Mendedahkan murid dengan aplikasi inovasi dan teknologi terkini.&#10;Memupuk semangat kerja berpasukan dan daya saing yang sihat di kalangan peserta.</textarea>
          </div>

          <!-- 2. Ringkasan Aktiviti -->
          <div class="space-y-1 relative">
            <div class="flex justify-between items-center">
              <label class="text-xs font-semibold text-slate-700">2. Ringkasan Pelaksanaan / Pengisian</label>
              <button type="button" onclick="generateAISpecific('ringkasan')" class="ai-btn text-[11px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-1 bg-blue-50 hover:bg-blue-100 px-2 py-1 rounded-md transition active:scale-95">
                <i data-lucide="sparkles" class="w-3.5 h-3.5 text-blue-500"></i> ✦ Cadangan AI
              </button>
            </div>
            <textarea id="input-ringkasan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">Program bermula dengan taklimat ringkas mengenai kepentingan STEM di masa hadapan. Selepas itu, murid dibahagikan kepada beberapa kumpulan untuk melaksanakan aktiviti hands-on seperti pembinaan roket air dan robotik asas. Bengkel diakhiri dengan sesi pembentangan hasil ciptaan oleh setiap kumpulan dan penyampaian hadiah.</textarea>
          </div>

          <!-- 3 & 4. Kekuatan & Kelemahan -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div class="space-y-1 relative">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">3. Kekuatan (Strengths)</label>
                <button type="button" onclick="generateAISpecific('kekuatan')" class="ai-btn text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-kekuatan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">1. Penglibatan aktif dari semua murid.&#10;2. Fasilitator yang berpengetahuan luas.&#10;3. Modul aktiviti yang menarik dan interaktif.</textarea>
            </div>
            <div class="space-y-1 relative">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">4. Kelemahan (Weaknesses)</label>
                <button type="button" onclick="generateAISpecific('kelemahan')" class="ai-btn text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-kelemahan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">1. Kekangan masa untuk setiap stesen aktiviti.&#10;2. Kekurangan beberapa peralatan teknikal.&#10;3. Ruang makmal agak sempit untuk jumlah peserta yang ramai.</textarea>
            </div>
          </div>

          <!-- 5 & 6. Penambahbaikan & Impak -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div class="space-y-1 relative">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">5. Cadangan Penambahbaikan</label>
                <button type="button" onclick="generateAISpecific('penambahbaikan')" class="ai-btn text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-penambahbaikan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">1. Menambah masa bagi sesi praktikal.&#10;2. Mengadakan program ini di dewan utama untuk ruang yang lebih selesa.&#10;3. Menyediakan kelengkapan tambahan secara berkumpulan.</textarea>
            </div>
            <div class="space-y-1 relative">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">6. Impak Program (Outcomes)</label>
                <button type="button" onclick="generateAISpecific('impak')" class="ai-btn text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-impak" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition">Meningkatkan kesedaran murid terhadap kepentingan kerjaya berteraskan sains dan teknologi serta melahirkan generasi yang celik inovasi dan berfikiran kritis.</textarea>
            </div>
          </div>
        </div>

        <!-- Section 4: Dynamic Photo Manager (2 to 4 photos) -->
        <div class="p-4 sm:p-5 space-y-4">
          <div class="flex items-center justify-between">
            <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
              <i data-lucide="image" class="w-4 h-4 text-blue-600"></i>
              4. Gambar Dokumentasi (1 - 4 Foto)
            </h2>
            <span id="photo-count-badge" class="text-xs font-semibold bg-slate-100 text-slate-700 px-2.5 py-0.5 rounded-full border border-slate-300">0 / 4 Gambar</span>
          </div>

          <div class="space-y-3">
            <div class="flex items-center justify-center w-full">
              <label class="flex flex-col items-center justify-center w-full h-24 border-2 border-slate-300 border-dashed rounded-lg cursor-pointer bg-slate-50 hover:bg-slate-100 transition">
                <div class="flex flex-col items-center justify-center pt-2 pb-3">
                  <i data-lucide="upload-cloud" class="w-6 h-6 text-slate-400 mb-1"></i>
                  <p class="text-xs text-slate-600 font-medium"><span class="font-bold text-blue-600">Muat Naik Gambar</span> (PNG/JPG)</p>
                  <p class="text-[10px] text-slate-400">Pilih sehingga 4 gambar aktiviti sekolah</p>
                </div>
                <input type="file" id="input-photos" accept="image/*" multiple onchange="handleImageUpload(event)" class="hidden" />
              </label>
            </div>

            <!-- Image Previews Container Grid -->
            <div id="image-preview-list" class="grid grid-cols-2 sm:grid-cols-4 gap-3 empty:hidden">
              <!-- Dynamically Populated Cards -->
            </div>
            <button type="button" id="btn-clear-photos" onclick="clearPhotos()" class="hidden w-full py-1.5 text-[11px] font-semibold text-red-600 bg-red-50 hover:bg-red-100 rounded-md transition border border-red-200">
              <i data-lucide="trash-2" class="w-3.5 h-3.5 inline mr-1"></i> Buang Semua Gambar
            </button>
          </div>
        </div>

        <!-- Section 5: Signatures -->
        <div class="p-4 sm:p-5 space-y-4">
          <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
            <i data-lucide="pen-tool" class="w-4 h-4 text-blue-600"></i>
            5. Ruang Pengesahan & Pegawai
          </h2>

          <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
            <!-- Disediakan Oleh -->
            <div class="space-y-2 p-3 bg-slate-50 rounded-lg border border-slate-200">
              <span class="text-xs font-bold text-slate-800 block">Disediakan Oleh:</span>
              <input type="text" id="input-disediakan-nama" oninput="updateOPR()" value="LAI YIN LEE" placeholder="Nama Pegawai" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition uppercase">
              <input type="text" id="input-disediakan-jawatan" oninput="updateOPR()" value="Penyelaras Program STEM" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <!-- Disemak Oleh -->
            <div class="space-y-2 p-3 bg-slate-50 rounded-lg border border-slate-200">
              <span class="text-xs font-bold text-slate-800 block">Disemak Oleh:</span>
              <input type="text" id="input-disemak-nama" oninput="updateOPR()" value="IRENE WAY" placeholder="Nama Penolong Kanan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition uppercase">
              <input type="text" id="input-disemak-jawatan" oninput="updateOPR()" value="Penolong Kanan Pentadbiran" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <!-- Disahkan Oleh -->
            <div class="space-y-2 p-3 bg-slate-50 rounded-lg border border-slate-200">
              <span class="text-xs font-bold text-slate-800 block">Disahkan Oleh:</span>
              <input type="text" id="input-disahkan-nama" oninput="updateOPR()" value="LEOW SOOK TIN" placeholder="Nama Guru Besar / Pengetua" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition uppercase">
              <input type="text" id="input-disahkan-jawatan" oninput="updateOPR()" value="Guru Besar" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>
          </div>
        </div>

      </form>
    </div>

    <!-- Right Column: Real-time Live A4 Document Preview -->
    <div id="preview-wrapper" class="lg:col-span-6 hidden lg:flex flex-col items-center w-full">
      
      <!-- Top Action Bar above A4 Document -->
      <div class="no-print w-full flex items-center justify-between mb-3 bg-white p-3 rounded-xl border border-slate-200 shadow-sm">
        <div class="flex items-center gap-2">
          <span class="inline-block w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse"></span>
          <span class="text-xs font-semibold text-slate-700">Pratonton Masa Nyata (Tepat 1 Muka Surat A4)</span>
        </div>
        <div class="flex items-center gap-2">
          <button onclick="window.print()" class="px-3.5 py-1.5 text-xs font-bold bg-blue-600 hover:bg-blue-700 text-white rounded-lg shadow flex items-center gap-1.5 transition active:scale-95">
            <i data-lucide="printer" class="w-4 h-4"></i> Cetak OPR
          </button>
        </div>
      </div>

      <!-- Printable Document Canvas Wrapper -->
      <div id="opr-preview-container" class="w-full flex justify-center overflow-x-auto pb-8 a4-preview-wrapper">
        <div class="a4-preview border border-slate-300 text-slate-900 text-[9px] leading-tight">
          
          <div class="flex-1 flex flex-col">
            <!-- Header KPM Standard (REMOVED: "LAPORAN SATU HALAMAN...") -->
            <div class="border-b-2 border-slate-900 pb-1 mb-1.5 flex items-center justify-between gap-2 shrink-0">
              <div id="view-logo-container" class="w-[60px] h-[60px] flex items-center justify-center shrink-0">
                <img id="view-logo-img" src="https://app.pandai.org/uploads/school/NBC4046.jpg" onerror="this.src='https://placehold.co/100x100/ffffff/cccccc?text=Logo'" alt="Logo Header" class="max-h-full max-w-full object-contain">
              </div>

              <div class="text-center flex-1">
                <h2 id="view-sekolah" class="font-extrabold text-[13px] tracking-wide uppercase text-slate-900">SEKOLAH JENIS KEBANGSAAN (CINA) PEI HUA</h2>
                <p class="text-[9px] font-semibold text-slate-600 uppercase mt-0.5" id="view-unit-anjuran">UNIT KURIKULUM | PERINGKAT SEKOLAH</p>
              </div>

              <div class="w-[60px] h-[60px] shrink-0 opacity-0">
                <!-- Spacer for visual symmetry -->
              </div>
            </div>

            <!-- Program Title Banner -->
            <div class="bg-slate-100 border border-slate-800 p-1.5 text-center mb-1.5 shrink-0">
              <h1 id="view-program" class="font-bold text-[11px] uppercase text-slate-900 leading-tight">BENGKEL PEMERKASAAN STEM & INOVASI SAINS 2026</h1>
            </div>

            <!-- Maklumat Utama Grid / Table -->
            <table class="w-full border-collapse border border-slate-800 mb-2 text-[9px] shrink-0">
              <tbody>
                <tr class="border-b border-slate-800">
                  <td class="bg-slate-100 font-bold p-1 px-1.5 border-r border-slate-800 w-1/6">Tarikh</td>
                  <td id="view-tarikh" class="p-1 px-1.5 border-r border-slate-800 w-2/6 uppercase">6 NOVEMBER 2026</td>
                  <td class="bg-slate-100 font-bold p-1 px-1.5 border-r border-slate-800 w-1/6">Masa</td>
                  <td id="view-masa" class="p-1 px-1.5 w-2/6 uppercase">8.30 PAGI - 1.00 TENGAH HARI</td>
                </tr>
                <tr class="border-b border-slate-800">
                  <td class="bg-slate-100 font-bold p-1 px-1.5 border-r border-slate-800">Tempat</td>
                  <td id="view-tempat" class="p-1 px-1.5 border-r border-slate-800 uppercase">STEM LAB</td>
                  <td class="bg-slate-100 font-bold p-1 px-1.5 border-r border-slate-800">Kehadiran</td>
                  <td id="view-sasaran" class="p-1 px-1.5 uppercase">60 MURID TAHUN 4 & 5 SERTA 8 GURU PEMBIMBING</td>
                </tr>
                <tr>
                  <td class="bg-slate-100 font-bold p-1 px-1.5 border-r border-slate-800">Perasmi / VIP</td>
                  <td colspan="3" id="view-perasmi" class="p-1 px-1.5 uppercase">EN. ROSLI BIN AHMAD (PEGAWAI SISC+ PPD)</td>
                </tr>
              </tbody>
            </table>

            <!-- Content Grid 1: Objektif & Ringkasan -->
            <div class="grid grid-cols-2 gap-2 mb-2 shrink-0">
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">1. Objektif Program</div>
                <div id="view-objektif" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">2. Ringkasan Pengisian</div>
                <div id="view-ringkasan" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
            </div>

            <!-- Content Grid 2: Kekuatan & Kelemahan -->
            <div class="grid grid-cols-2 gap-2 mb-2 shrink-0">
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">3. Kekuatan</div>
                <div id="view-kekuatan" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">4. Kelemahan</div>
                <div id="view-kelemahan" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
            </div>

            <!-- Content Grid 3: Penambahbaikan & Impak -->
            <div class="grid grid-cols-2 gap-2 mb-2 shrink-0">
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">5. Cadangan Penambahbaikan</div>
                <div id="view-penambahbaikan" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
              <div class="border border-slate-800 flex flex-col h-full">
                <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase">6. Impak (Outcomes)</div>
                <div id="view-impak" class="p-1.5 whitespace-pre-line leading-[1.35] flex-1"></div>
              </div>
            </div>

            <!-- Photo Grid -->
            <div class="border border-slate-800 flex flex-col flex-1 min-h-0">
              <div class="bg-slate-100 font-bold text-[9.5px] border-b border-slate-800 p-1 px-1.5 uppercase shrink-0">
                7. Gambar Dokumentasi Aktiviti
              </div>
              <div id="view-photos-grid" class="flex-1 p-2 grid grid-cols-2 gap-2 min-h-0 place-items-center bg-slate-50/50">
                <!-- Placeholders or uploaded images injected here by JS -->
                <div class="w-full h-full border border-dashed border-slate-300 flex items-center justify-center text-slate-400">Ruang Gambar 1</div>
                <div class="w-full h-full border border-dashed border-slate-300 flex items-center justify-center text-slate-400">Ruang Gambar 2</div>
              </div>
            </div>

          </div> <!-- End Flex 1 Container -->

          <!-- Signatures (Fixed at bottom) -->
          <div class="mt-3 pt-2 border-t-2 border-slate-900 shrink-0">
            <div class="grid grid-cols-3 gap-4 text-center">
              <div>
                <p class="font-bold text-[8.5px] uppercase mb-10">Disediakan Oleh,</p>
                <div class="border-b border-slate-400 w-3/4 mx-auto mb-1"></div>
                <p id="view-disediakan-nama" class="font-bold uppercase text-[9px]">LAI YIN LEE</p>
                <p id="view-disediakan-jawatan" class="text-[8.5px] uppercase">Penyelaras Program STEM</p>
              </div>
              <div>
                <p class="font-bold text-[8.5px] uppercase mb-10">Disemak Oleh,</p>
                <div class="border-b border-slate-400 w-3/4 mx-auto mb-1"></div>
                <p id="view-disemak-nama" class="font-bold uppercase text-[9px]">IRENE WAY</p>
                <p id="view-disemak-jawatan" class="text-[8.5px] uppercase">Penolong Kanan Pentadbiran</p>
              </div>
              <div>
                <p class="font-bold text-[8.5px] uppercase mb-10">Disahkan Oleh,</p>
                <div class="border-b border-slate-400 w-3/4 mx-auto mb-1"></div>
                <p id="view-disahkan-nama" class="font-bold uppercase text-[9px]">LEOW SOOK TIN</p>
                <p id="view-disahkan-jawatan" class="text-[8.5px] uppercase">Guru Besar</p>
              </div>
            </div>
          </div>
          
        </div>
      </div>
    </div>
  </main>

  <script>
    // Initialize Lucide Icons
    lucide.createIcons();

    // Setup initial date if not set
    document.addEventListener('DOMContentLoaded', () => {
      const dateInput = document.getElementById('input-tarikh-date');
      if (!dateInput.value) {
        const today = new Date().toISOString().split('T')[0];
        dateInput.value = today;
      }
      updateOPR();
    });

    // Custom Modal System (Replaces alert and confirm)
    function showModal(title, message, isConfirm = false, onConfirm = null) {
      const modal = document.getElementById('custom-modal');
      const titleEl = document.getElementById('modal-title');
      const messageEl = document.getElementById('modal-message');
      const btnCancel = document.getElementById('modal-btn-cancel');
      const btnOk = document.getElementById('modal-btn-ok');
      const icon = document.getElementById('modal-icon');

      titleEl.textContent = title;
      messageEl.textContent = message;

      // Adjust styling based on context
      if (title.toLowerCase().includes('ralat')) {
        icon.className = 'p-2 bg-red-100 text-red-600 rounded-full';
        icon.innerHTML = '<i data-lucide="alert-triangle" class="w-5 h-5"></i>';
      } else if (isConfirm) {
        icon.className = 'p-2 bg-amber-100 text-amber-600 rounded-full';
        icon.innerHTML = '<i data-lucide="help-circle" class="w-5 h-5"></i>';
      } else {
        icon.className = 'p-2 bg-blue-100 text-blue-600 rounded-full';
        icon.innerHTML = '<i data-lucide="info" class="w-5 h-5"></i>';
      }
      lucide.createIcons();

      // Setup actions
      btnOk.onclick = () => {
        modal.classList.add('hidden');
        if (onConfirm) onConfirm();
      };

      if (isConfirm) {
        btnCancel.classList.remove('hidden');
        btnCancel.onclick = () => modal.classList.add('hidden');
      } else {
        btnCancel.classList.add('hidden');
      }

      modal.classList.remove('hidden');
    }

    // Mobile Drawer Toggle
    function toggleMobileDrawer(show) {
      const drawer = document.getElementById('mobile-drawer');
      const backdrop = document.getElementById('mobile-drawer-backdrop');
      
      if (show) {
        drawer.classList.remove('-translate-x-full');
        backdrop.classList.remove('hidden');
        setTimeout(() => backdrop.classList.remove('opacity-0'), 10);
      } else {
        drawer.classList.add('-translate-x-full');
        backdrop.classList.add('opacity-0');
        setTimeout(() => backdrop.classList.add('hidden'), 300);
      }
    }

    // Switch Tabs for Mobile View
    function switchTab(tabName) {
      const formContainer = document.getElementById('opr-form-container');
      const previewWrapper = document.getElementById('preview-wrapper');
      const btnForm = document.getElementById('btn-tab-form');
      const btnPreview = document.getElementById('btn-tab-preview');

      if (window.innerWidth >= 1024) return; // Only apply on mobile/tablet

      if (tabName === 'form') {
        formContainer.classList.remove('hidden');
        previewWrapper.classList.add('hidden');
        previewWrapper.classList.remove('flex');
        
        btnForm.classList.replace('text-slate-700', 'text-white');
        btnForm.classList.replace('hover:bg-slate-300', 'bg-blue-600');
        btnForm.classList.add('shadow-sm');
        
        btnPreview.classList.replace('text-white', 'text-slate-700');
        btnPreview.classList.replace('bg-blue-600', 'hover:bg-slate-300');
        btnPreview.classList.remove('shadow-sm');
      } else {
        formContainer.classList.add('hidden');
        previewWrapper.classList.remove('hidden');
        previewWrapper.classList.add('flex');
        
        btnPreview.classList.replace('text-slate-700', 'text-white');
        btnPreview.classList.replace('hover:bg-slate-300', 'bg-blue-600');
        btnPreview.classList.add('shadow-sm');
        
        btnForm.classList.replace('text-white', 'text-slate-700');
        btnForm.classList.replace('bg-blue-600', 'hover:bg-slate-300');
        btnForm.classList.remove('shadow-sm');
      }
    }

    // Handle Resize to ensure desktop shows both columns
    window.addEventListener('resize', () => {
      if (window.innerWidth >= 1024) {
        document.getElementById('opr-form-container').classList.remove('hidden');
        document.getElementById('preview-wrapper').classList.remove('hidden');
        document.getElementById('preview-wrapper').classList.add('flex');
      } else {
        // Revert to form view on resize down
        switchTab('form');
      }
    });

    // Formatting date helper: YYYY-MM-DD -> DD/MM/YYYY
    function formatDateMy(dateStr) {
      if (!dateStr) return "";
      const d = new Date(dateStr);
      if (isNaN(d.getTime())) return dateStr;
      
      const months = [
        "Januari", "Februari", "Mac", "April", "Mei", "Jun", 
        "Julai", "Ogos", "September", "Oktober", "November", "Disember"
      ];
      
      const day = d.getDate();
      const month = months[d.getMonth()];
      const year = d.getFullYear();
      
      return `${day} ${month} ${year}`.toUpperCase();
    }

    // Handle Custom Location Toggle
    function handleTempatChange() {
      const select = document.getElementById('input-tempat-select');
      const customContainer = document.getElementById('custom-tempat-container');
      
      if (select.value === 'custom') {
        customContainer.classList.remove('hidden');
      } else {
        customContainer.classList.add('hidden');
      }
    }

    // Main Function to Sync Form -> Preview
    function updateOPR() {
      // 1. Header & Basics
      document.getElementById('view-sekolah').textContent = document.getElementById('input-sekolah').value || "-";
      document.getElementById('view-logo-img').src = document.getElementById('input-logo-url').value;
      
      // 2. Program Details
      document.getElementById('view-program').textContent = document.getElementById('input-program').value || "NAMA PROGRAM";
      
      const unit = document.getElementById('input-unit').value;
      const anjuran = document.getElementById('input-anjuran').value;
      document.getElementById('view-unit-anjuran').textContent = `UNIT ${unit} | ${anjuran}`.toUpperCase();
      
      const rawDate = document.getElementById('input-tarikh-date').value;
      document.getElementById('view-tarikh').textContent = formatDateMy(rawDate) || "-";
      document.getElementById('view-masa').textContent = document.getElementById('input-masa').value || "-";
      
      const selectTempat = document.getElementById('input-tempat-select').value;
      if (selectTempat === 'custom') {
        document.getElementById('view-tempat').textContent = document.getElementById('input-tempat-custom').value || "-";
      } else {
        document.getElementById('view-tempat').textContent = selectTempat;
      }
      
      document.getElementById('view-sasaran').textContent = document.getElementById('input-sasaran').value || "-";
      
      const perasmiVal = document.getElementById('input-perasmi').value.trim();
      document.getElementById('view-perasmi').textContent = perasmiVal ? perasmiVal : "TIADA";
      
      // 3. Content Sections
      document.getElementById('view-objektif').textContent = document.getElementById('input-objektif').value;
      document.getElementById('view-ringkasan').textContent = document.getElementById('input-ringkasan').value;
      document.getElementById('view-kekuatan').textContent = document.getElementById('input-kekuatan').value;
      document.getElementById('view-kelemahan').textContent = document.getElementById('input-kelemahan').value;
      document.getElementById('view-penambahbaikan').textContent = document.getElementById('input-penambahbaikan').value;
      document.getElementById('view-impak').textContent = document.getElementById('input-impak').value;

      // 5. Signatures
      document.getElementById('view-disediakan-nama').textContent = document.getElementById('input-disediakan-nama').value || "NAMA";
      document.getElementById('view-disediakan-jawatan').textContent = document.getElementById('input-disediakan-jawatan').value || "JAWATAN";
      
      document.getElementById('view-disemak-nama').textContent = document.getElementById('input-disemak-nama').value || "NAMA";
      document.getElementById('view-disemak-jawatan').textContent = document.getElementById('input-disemak-jawatan').value || "JAWATAN";
      
      document.getElementById('view-disahkan-nama').textContent = document.getElementById('input-disahkan-nama').value || "NAMA";
      document.getElementById('view-disahkan-jawatan').textContent = document.getElementById('input-disahkan-jawatan').value || "JAWATAN";
    }

    // Image Upload Logic (Store base64 images)
    let uploadedPhotos = [];

    function handleImageUpload(event) {
      const files = Array.from(event.target.files);
      if (files.length === 0) return;
      
      // We only allow up to 4 photos total
      const remainingSlots = 4 - uploadedPhotos.length;
      const filesToProcess = files.slice(0, remainingSlots);

      if (files.length > remainingSlots && remainingSlots === 0) {
        showModal("Ralat Gambar", "Maksimum 4 gambar sahaja dibenarkan.");
        return;
      }

      filesToProcess.forEach(file => {
        if (!file.type.startsWith('image/')) return;

        const reader = new FileReader();
        reader.onload = (e) => {
          uploadedPhotos.push(e.target.result);
          renderPhotos();
        };
        reader.readAsDataURL(file);
      });
      
      // Reset input so same file can be selected again if removed
      event.target.value = ''; 
    }

    function removePhoto(index) {
      uploadedPhotos.splice(index, 1);
      renderPhotos();
    }

    function clearPhotos() {
      uploadedPhotos = [];
      renderPhotos();
    }

    function renderPhotos() {
      const previewList = document.getElementById('image-preview-list');
      const viewGrid = document.getElementById('view-photos-grid');
      const countBadge = document.getElementById('photo-count-badge');
      const btnClear = document.getElementById('btn-clear-photos');
      
      countBadge.textContent = `${uploadedPhotos.length} / 4 Gambar`;
      
      // Update form thumbnail preview
      previewList.innerHTML = '';
      if (uploadedPhotos.length > 0) {
        btnClear.classList.remove('hidden');
        uploadedPhotos.forEach((src, idx) => {
          previewList.innerHTML += `
            <div class="relative group aspect-[4/3] rounded-lg overflow-hidden border border-slate-200 shadow-sm">
              <img src="${src}" class="w-full h-full object-cover" />
              <button onclick="removePhoto(${idx})" class="absolute top-1 right-1 bg-red-500/90 hover:bg-red-600 text-white p-1 rounded-md opacity-0 group-hover:opacity-100 transition shadow">
                <i data-lucide="x" class="w-3 h-3"></i>
              </button>
            </div>
          `;
        });
        lucide.createIcons();
      } else {
        btnClear.classList.add('hidden');
      }

      // Update A4 Document Grid
      viewGrid.innerHTML = '';
      if (uploadedPhotos.length === 0) {
        viewGrid.innerHTML = `
          <div class="w-full h-full border border-dashed border-slate-300 flex items-center justify-center text-slate-400 text-[10px]">Ruang Gambar 1</div>
          <div class="w-full h-full border border-dashed border-slate-300 flex items-center justify-center text-slate-400 text-[10px]">Ruang Gambar 2</div>
        `;
      } else {
        // Adjust grid columns based on count to optimize A4 space
        let gridClass = 'grid-cols-2'; 
        uploadedPhotos.forEach((src) => {
          viewGrid.innerHTML += `
            <div class="w-full h-full bg-slate-200 flex items-center justify-center overflow-hidden border border-slate-300">
              <img src="${src}" class="w-full h-full object-cover" />
            </div>
          `;
        });
      }
    }

    // Reset Form
    function promptResetForm() {
      showModal("Pengesahan Tindakan", "Adakah anda pasti untuk reset semua maklumat pada borang?", true, () => {
        document.getElementById('opr-form').reset();
        
        // Reset date to today
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('input-tarikh-date').value = today;
        
        // Ensure custom tempat is hidden
        handleTempatChange();
        
        // Clear images
        clearPhotos();
        
        // Trigger update
        updateOPR();
        
        // Reset AI button state if needed
        document.querySelectorAll('.ai-btn i').forEach(icon => {
          icon.setAttribute('data-lucide', 'sparkles');
          icon.classList.remove('animate-spin');
        });
        lucide.createIcons();
      });
    }

    // --- Gemini API Integration ---
    // Note: To adhere to platform rules, the apiKey is left blank here.
    const apiKey = ""; 
    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

    // Generate specific field via AI
    async function generateAISpecific(fieldId) {
      const programName = document.getElementById('input-program').value.trim();
      const unit = document.getElementById('input-unit').value;
      const sasaran = document.getElementById('input-sasaran').value;

      if (!programName) {
        showModal("Maklumat Tidak Lengkap", "Sila masukkan 'Nama Program' dahulu sebelum menjana kandungan AI.");
        document.getElementById('input-program').focus();
        return;
      }

      // Find button icon for loading state
      const btn = event.currentTarget;
      const icon = btn.querySelector('i');
      const originalIcon = icon.getAttribute('data-lucide');
      
      icon.setAttribute('data-lucide', 'loader-2');
      icon.classList.add('animate-spin');
      lucide.createIcons();

      const systemPrompt = `Anda adalah seorang guru Malaysia yang pakar menulis Laporan Program (OPR) yang profesional dalam Bahasa Melayu formal. Tulis point-point ringkas atau satu perenggan yang padat.`;
      
      let fieldDesc = "";
      switch(fieldId) {
        case 'objektif': fieldDesc = "3 Objektif program (Senaraikan bernombor)."; break;
        case 'ringkasan': fieldDesc = "Ringkasan pengisian program (1 perenggan padat 3-4 ayat)."; break;
        case 'kekuatan': fieldDesc = "3 Kekuatan program (Senaraikan bernombor)."; break;
        case 'kelemahan': fieldDesc = "3 Kelemahan/Kekangan program (Senaraikan bernombor)."; break;
        case 'penambahbaikan': fieldDesc = "3 Cadangan penambahbaikan (Senaraikan bernombor)."; break;
        case 'impak': fieldDesc = "Impak program kepada murid/peserta (1 ayat padat)."; break;
      }

      const userPrompt = `Program: ${programName}\nSektor: ${unit}\nSasaran: ${sasaran}\n\nTolong jana maklumat untuk ruangan: ${fieldDesc}\nPastikan laras bahasa pentadbiran sekolah KPM.`;

      const payload = {
        contents: [{ parts: [{ text: userPrompt }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
      };

      try {
        const response = await fetch(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });
        
        const result = await response.json();
        if (result.candidates && result.candidates.length > 0) {
          let text = result.candidates[0].content.parts[0].text.trim();
          // Clean up markdown formatting if present
          text = text.replace(/\*\*/g, '');
          
          document.getElementById(`input-${fieldId}`).value = text;
          updateOPR();
        } else {
          showModal("Ralat AI", "Sistem tidak memulangkan sebarang teks. Sila cuba sebentar lagi.");
        }
      } catch (error) {
        console.error("AI Error:", error);
        showModal("Ralat Sambungan", "Gagal menghubungi pelayan AI. Pastikan sambungan internet anda stabil atau cuba lagi.");
      } finally {
        icon.setAttribute('data-lucide', originalIcon);
        icon.classList.remove('animate-spin');
        lucide.createIcons();
      }
    }

    // Generate Whole Form via AI (Structured JSON)
    async function autoFillWholeFormAI() {
      const programName = document.getElementById('input-program').value.trim();
      const unit = document.getElementById('input-unit').value;
      const sasaran = document.getElementById('input-sasaran').value;

      if (!programName || programName === "BENGKEL PEMERKASAAN STEM & INOVASI SAINS 2026") {
        showModal(
          "Pengesahan Maklumat", 
          "Anda menggunakan Nama Program lalai (default). Adakah anda mahu AI menjana kandungan berdasarkan nama ini? (Digalakkan untuk menukar nama program bagi hasil yang lebih spesifik).",
          true, 
          () => executeAI()
        );
      } else {
        executeAI();
      }

      async function executeAI() {
        const btnAll = document.getElementById('btn-ai-all');
        const originalHtml = btnAll.innerHTML;
        btnAll.innerHTML = `<i data-lucide="loader-2" class="w-4 h-4 animate-spin"></i> Menjana...`;
        lucide.createIcons();

        const systemPrompt = `Anda adalah pendidik di Malaysia. Jana maklumat lengkap Laporan Program Sekolah (OPR) berdasarkan nama program yang diberi. Gunakan Bahasa Melayu rasmi KPM. Pastikan jawapan ringkas, padat dan logik.`;
        
        const userPrompt = `Jana pengisian OPR untuk program berikut:\nNama Program: ${programName}\nUnit: ${unit}\nSasaran: ${sasaran}\n\nBerikan respons dalam format JSON dengan kunci: objektif, ringkasan, kekuatan, kelemahan, penambahbaikan, impak. Gunakan bullet points atau nombor (1., 2.) di mana sesuai dalam teks.`;

        const payload = {
          contents: [{ parts: [{ text: userPrompt }] }],
          systemInstruction: { parts: [{ text: systemPrompt }] },
          generationConfig: {
            responseMimeType: "application/json",
            responseSchema: {
              type: "OBJECT",
              properties: {
                "objektif": { type: "STRING" },
                "ringkasan": { type: "STRING" },
                "kekuatan": { type: "STRING" },
                "kelemahan": { type: "STRING" },
                "penambahbaikan": { type: "STRING" },
                "impak": { type: "STRING" }
              },
              required: ["objektif", "ringkasan", "kekuatan", "kelemahan", "penambahbaikan", "impak"]
            }
          }
        };

        try {
          const response = await fetch(apiUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
          });
          
          const result = await response.json();
          
          if (result.candidates && result.candidates[0].content && result.candidates[0].content.parts) {
            const jsonText = result.candidates[0].content.parts[0].text;
            const data = JSON.parse(jsonText);
            
            document.getElementById('input-objektif').value = (data.objektif || "").replace(/\*\*/g, '');
            document.getElementById('input-ringkasan').value = (data.ringkasan || "").replace(/\*\*/g, '');
            document.getElementById('input-kekuatan').value = (data.kekuatan || "").replace(/\*\*/g, '');
            document.getElementById('input-kelemahan').value = (data.kelemahan || "").replace(/\*\*/g, '');
            document.getElementById('input-penambahbaikan').value = (data.penambahbaikan || "").replace(/\*\*/g, '');
            document.getElementById('input-impak').value = (data.impak || "").replace(/\*\*/g, '');
            
            updateOPR();
          } else {
             showModal("Ralat AI", "Sistem tidak memulangkan kandungan yang sah.");
          }
        } catch (error) {
          console.error("AI Gen All Error:", error);
          showModal("Ralat Semasa Penjanaan", "Ralat berlaku semasa menghubungi servis AI. Sila pastikan anda mempunyai akses internet dan cuba lagi.");
        } finally {
          btnAll.innerHTML = originalHtml;
          lucide.createIcons();
        }
      }
    }
  </script>
</body>
</html>
