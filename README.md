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
  </style>
</head>
<body class="text-slate-800 antialiased h-full flex flex-col min-h-screen">

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
        <button onclick="autoFillWholeFormAI(); toggleMobileDrawer(false);" class="w-full flex items-center space-x-3 px-3 py-2.5 rounded-lg text-xs font-semibold text-emerald-400 hover:bg-slate-800 transition">
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
      <button onclick="resetForm(); toggleMobileDrawer(false);" class="w-full flex items-center justify-center space-x-2 px-3 py-2 text-xs font-medium text-slate-400 hover:text-white bg-slate-800 rounded-lg">
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
              Penjana OPR KPM 
              <span class="bg-blue-500/30 text-blue-300 text-[10px] sm:text-xs px-2 py-0.5 rounded-full border border-blue-400/30">Edisi AI</span>
            </h1>
            <p class="text-[11px] text-slate-400 hidden md:block">Sistem Laporan Satu Halaman Standard Sekolah & Pejabat Pendidikan KPM</p>
          </div>
        </div>
      </div>

      <!-- Right Action Toolbar -->
      <div class="flex items-center space-x-2">
        <button onclick="autoFillWholeFormAI()" class="px-3 py-2 text-xs font-semibold bg-emerald-600 hover:bg-emerald-500 text-white rounded-lg shadow flex items-center gap-1.5 transition active:scale-95">
          <i data-lucide="sparkles" class="w-4 h-4"></i>
          <span class="hidden sm:inline">Jana Semua via</span> AI
        </button>

        <button onclick="window.print()" class="px-3.5 py-2 text-xs font-semibold bg-blue-600 hover:bg-blue-500 text-white rounded-lg shadow flex items-center gap-1.5 transition active:scale-95">
          <i data-lucide="printer" class="w-4 h-4"></i>
          <span class="hidden sm:inline">Cetak /</span> PDF
        </button>

        <button onclick="resetForm()" class="hidden sm:flex px-3 py-2 text-xs font-medium bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg border border-slate-700 transition" title="Set Semula Borang">
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
              <input type="text" id="input-sekolah" oninput="updateOPR()" value="SEKOLAH JENIS KEBANGSAAN PEI HUA" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <div>
              <label class="block text-xs font-medium text-slate-700 mb-1">Pilihan Logo Header</label>
              <select id="input-logo-type" onchange="toggleLogoInput(); updateOPR();" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
                <option value="kpm">Logo KPM (Standard Rasmi)</option>
                <option value="jpn">Logo JPN / PPD</option>
                <option value="custom">URL Logo Tersuai</option>
                <option value="none">Tanpa Logo</option>
              </select>
            </div>
          </div>

          <div id="custom-logo-container" class="hidden">
            <label class="block text-xs font-medium text-slate-700 mb-1">URL Logo Tersuai (Pautan Gambar)</label>
            <input type="text" id="input-logo-url" oninput="updateOPR()" placeholder="https://example.com/logo.png" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
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
                  <option value="Kurikulum">Kurikulum</option>
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
                  <option value="Peringkat Sekolah">Sekolah</option>
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
                <input type="text" id="input-masa" oninput="updateOPR()" value="8.30 PAGI - 1.00 TENGARI" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              </div>
              <!-- Location Dropdown with Custom Input Toggle -->
              <div>
                <label class="block text-xs font-medium text-slate-700 mb-1">Tempat Program</label>
                <select id="input-tempat-select" onchange="handleTempatChange(); updateOPR();" class="w-full text-xs px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none transition bg-white">
                  <option value="STEM Lab">STEM Lab</option>
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
          <div class="space-y-1">
            <div class="flex justify-between items-center">
              <label class="text-xs font-semibold text-slate-700">1. Objektif Program</label>
              <button type="button" onclick="generateAISpecific('objektif')" class="text-[11px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-1 bg-blue-50 hover:bg-blue-100 px-2 py-1 rounded-md transition active:scale-95">
                <i data-lucide="sparkles" class="w-3.5 h-3.5 text-blue-500"></i> ✦ Cadangan AI
              </button>
            </div>
            <textarea id="input-objektif" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
          </div>

          <!-- 2. Ringkasan Aktiviti -->
          <div class="space-y-1">
            <div class="flex justify-between items-center">
              <label class="text-xs font-semibold text-slate-700">2. Ringkasan Pelaksanaan / Pengisian</label>
              <button type="button" onclick="generateAISpecific('ringkasan')" class="text-[11px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-1 bg-blue-50 hover:bg-blue-100 px-2 py-1 rounded-md transition active:scale-95">
                <i data-lucide="sparkles" class="w-3.5 h-3.5 text-blue-500"></i> ✦ Cadangan AI
              </button>
            </div>
            <textarea id="input-ringkasan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
          </div>

          <!-- 3 & 4. Kekuatan & Kelemahan -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div class="space-y-1">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">3. Kekuatan (Strengths)</label>
                <button type="button" onclick="generateAISpecific('kekuatan')" class="text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-kekuatan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
            </div>
            <div class="space-y-1">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">4. Kelemahan (Weaknesses)</label>
                <button type="button" onclick="generateAISpecific('kelemahan')" class="text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-kelemahan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
            </div>
          </div>

          <!-- 5 & 6. Penambahbaikan & Impak -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div class="space-y-1">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">5. Cadangan Penambahbaikan</label>
                <button type="button" onclick="generateAISpecific('penambahbaikan')" class="text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-penambahbaikan" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
            </div>
            <div class="space-y-1">
              <div class="flex justify-between items-center">
                <label class="text-xs font-semibold text-slate-700">6. Impak Program (Outcomes)</label>
                <button type="button" onclick="generateAISpecific('impak')" class="text-[10px] text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-0.5 bg-blue-50 px-2 py-1 rounded-md transition">
                  <i data-lucide="sparkles" class="w-3 h-3"></i> AI
                </button>
              </div>
              <textarea id="input-impak" oninput="updateOPR()" rows="3" class="w-full text-xs p-2.5 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none leading-relaxed transition"></textarea>
            </div>
          </div>
        </div>

        <!-- Section 4: Dynamic Photo Manager (2 to 4 photos) -->
        <div class="p-4 sm:p-5 space-y-4">
          <div class="flex items-center justify-between">
            <h2 class="text-xs sm:text-sm font-bold text-slate-900 uppercase tracking-wide flex items-center gap-2">
              <i data-lucide="image" class="w-4 h-4 text-blue-600"></i>
              4. Gambar Dokumentasi (2 - 4 Foto)
            </h2>
            <span id="photo-count-badge" class="text-xs font-semibold bg-slate-100 text-slate-700 px-2.5 py-0.5 rounded-full border border-slate-300">4 / 4 Gambar</span>
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
            <div id="image-preview-list" class="grid grid-cols-2 sm:grid-cols-4 gap-3">
              <!-- Dynamically Populated Cards -->
            </div>
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
              <input type="text" id="input-disediakan-nama" oninput="updateOPR()" value="NURUL HASNAH BINTI ROSLI" placeholder="Nama Pegawai" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              <input type="text" id="input-disediakan-jawatan" oninput="updateOPR()" value="Penyelaras Program STEM" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <!-- Disemak Oleh -->
            <div class="space-y-2 p-3 bg-slate-50 rounded-lg border border-slate-200">
              <span class="text-xs font-bold text-slate-800 block">Disemak Oleh:</span>
              <input type="text" id="input-disemak-nama" oninput="updateOPR()" value="ZULKIFLI BIN MAHMUD" placeholder="Nama Penolong Kanan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              <input type="text" id="input-disemak-jawatan" oninput="updateOPR()" value="Penolong Kanan Pentadbiran" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
            </div>

            <!-- Disahkan Oleh -->
            <div class="space-y-2 p-3 bg-slate-50 rounded-lg border border-slate-200">
              <span class="text-xs font-bold text-slate-800 block">Disahkan Oleh:</span>
              <input type="text" id="input-disahkan-nama" oninput="updateOPR()" value="DR. AZMAN BIN SHAH" placeholder="Nama Guru Besar / Pengetua" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
              <input type="text" id="input-disahkan-jawatan" oninput="updateOPR()" value="Guru Besar Cemerlang" placeholder="Jawatan" class="w-full text-xs px-2.5 py-1.5 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 focus:outline-none transition">
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
          
          <div>
            <!-- Header KPM Standard -->
            <div class="border-b-2 border-slate-900 pb-1 mb-1 flex items-center justify-between gap-2">
              <div id="view-logo-container" class="w-10 h-10 flex items-center justify-center shrink-0">
                <img id="view-logo-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/94/Coat_of_arms_of_Malaysia.svg/300px-Coat_of_arms_of_Malaysia.svg.png" alt="Logo Header" class="max-h-full max-w-full object-contain">
              </div>

              <div class="text-center flex-1">
                <h2 id="view-sekolah" class="font-extrabold text-xs tracking-wide uppercase text-slate-900">SJKC PEI HUA</h2>
                <h3 class="font-bold text-[10px] uppercase tracking-wider text-slate-800">LAPORAN SATU HALAMAN (ONE PAGE REPORT - OPR)</h3>
                <p class="text-[8.5px] font-semibold text-slate-600 uppercase" id="view-unit-anjuran">UNIT KURIKULUM | PERINGKAT SEKOLAH</p>
              </div>

              <div class="w-10 h-10 shrink-0 opacity-0">
                <!-- Spacer for visual symmetry -->
              </div>
            </div>

            <!-- Program Title Banner -->
            <div class="bg-slate-100 border border-slate-800 p-1 text-center mb-1">
              <h1 id="view-program" class="font-bold text-[10.5px] uppercase text-slate-900 leading-tight">BENGKEL PEMERKASAAN STEM & INOVASI SAINS 2026</h1>
            </div>

            <!-- Maklumat Utama Grid / Table -->
            <table class="w-full border-collapse border border-slate-800 mb-1 text-[8.5px]">
              <tbody>
                <tr class="border-b border-slate-800">
                  <td class="bg-slate-100 font-bold p-0.5 px-1 border-r border-slate-800 w-1/6">Tarikh</td>
                  <td id="view-tarikh" class="p-0.5 px-1 border-r border-slate-800 w-2/6">6 NOVEMBER 2026</td>
                  <td class="bg-slate-100 font-bold p-0.5 px-1 border-r border-slate-800 w-1/6">Masa</td>
                  <td id="view-masa" class="p-0.5 px-1 w-2/6">8.30 PAGI - 1.00 TENGARI</td>
                </tr>
                <tr class="border-b border-slate-800">
                  <td class="bg-slate-100 font-bold p-0.5 px-1 border-r border-slate-800">Tempat</td>
                  <td id="view-tempat" class="p-0.5 px-1 border-r border-slate-800">STEM Lab</td>
                  <td class="bg-slate-100 font-bold p-0.5 px-1 border-r border-slate-800">Sasaran</td>
                  <td id="view-sasaran" class="p-0.5 px-1">60 Murid Tahun 4 & 5 serta 8 Guru Pembimbing</td>
                </tr>
                <tr>
                  <td class="bg-slate-100 font-bold p-0.5 px-1 border-r border-slate-800">Tetamu / VIP</td>
                  <td id="view-perasmi" colspan="3" class="p-0.5 px-1">En. Rosli bin Ahmad (Pegawai SISC+ PPD)</td>
                </tr>
              </tbody>
            </table>

            <!-- Content Grid: Objektif & Pengisian -->
            <div class="grid grid-cols-2 gap-2 mb-1.5">
              <!-- Objektif -->
              <div class="border border-slate-800 p-1.5 bg-white flex flex-col">
                <div class="bg-slate-800 text-white font-bold text-[8.5px] px-1 py-0.5 uppercase mb-1">1. Objektif Program</div>
                <div id="view-objektif" class="whitespace-pre-line text-slate-800 flex-1 text-[8px] leading-snug"></div>
              </div>

              <!-- Pengisian Aktiviti -->
              <div class="border border-slate-800 p-1.5 bg-white flex flex-col">
                <div class="bg-slate-800 text-white font-bold text-[8.5px] px-1 py-0.5 uppercase mb-1">2. Ringkasan Pelaksanaan / Pengisian</div>
                <div id="view-ringkasan" class="whitespace-pre-line text-slate-800 flex-1 text-[8px] leading-snug"></div>
              </div>
            </div>

            <!-- Analysis Grid: SWOT / Impak -->
            <div class="grid grid-cols-2 gap-2 mb-1.5">
              <!-- Kekuatan & Kelemahan -->
              <div class="border border-slate-800 p-1.5 space-y-1">
                <div>
                  <span class="font-bold text-slate-900 border-b border-slate-400 block pb-0.5 mb-0.5 uppercase text-[8px]">3. Kekuatan (Strengths):</span>
                  <div id="view-kekuatan" class="whitespace-pre-line text-slate-800 text-[8px]"></div>
                </div>
                <div>
                  <span class="font-bold text-slate-900 border-b border-slate-400 block pb-0.5 mb-0.5 uppercase text-[8px]">4. Kelemahan (Weaknesses):</span>
                  <div id="view-kelemahan" class="whitespace-pre-line text-slate-800 text-[8px]"></div>
                </div>
              </div>

              <!-- Penambahbaikan & Impak -->
              <div class="border border-slate-800 p-1.5 space-y-1">
                <div>
                  <span class="font-bold text-slate-900 border-b border-slate-400 block pb-0.5 mb-0.5 uppercase text-[8px]">5. Cadangan Penambahbaikan:</span>
                  <div id="view-penambahbaikan" class="whitespace-pre-line text-slate-800 text-[8px]"></div>
                </div>
                <div>
                  <span class="font-bold text-slate-900 border-b border-slate-400 block pb-0.5 mb-0.5 uppercase text-[8px]">6. Impak Program (Outcomes):</span>
                  <div id="view-impak" class="whitespace-pre-line text-slate-800 text-[8px]"></div>
                </div>
              </div>
            </div>

            <!-- Section 4: Gambar Dokumentasi Grid -->
            <div class="border border-slate-800 p-1 mb-1.5">
              <div class="bg-slate-800 text-white font-bold text-[8.5px] px-1 py-0.5 uppercase mb-1">7. Gambar Dokumentasi Aktiviti</div>
              <div id="view-photo-grid" class="grid grid-cols-2 gap-1">
                <!-- Dynamically populated images -->
              </div>
            </div>
          </div>

          <!-- Section 5: Signatures Section -->
          <div class="border-t-2 border-slate-800 pt-1 mt-auto">
            <div class="grid grid-cols-3 gap-2 text-[8px]">
              <!-- Disediakan -->
              <div class="flex flex-col justify-between h-10">
                <div>
                  <span class="font-semibold text-slate-700 block">Disediakan Oleh:</span>
                </div>
                <div>
                  <div class="border-b border-slate-400 w-3/4 mb-0.5"></div>
                  <p id="view-disediakan-nama" class="font-bold text-slate-900 uppercase text-[7.5px]"></p>
                  <p id="view-disediakan-jawatan" class="text-[7px] text-slate-600"></p>
                </div>
              </div>

              <!-- Disemak -->
              <div class="flex flex-col justify-between h-10">
                <div>
                  <span class="font-semibold text-slate-700 block">Disemak Oleh:</span>
                </div>
                <div>
                  <div class="border-b border-slate-400 w-3/4 mb-0.5"></div>
                  <p id="view-disemak-nama" class="font-bold text-slate-900 uppercase text-[7.5px]"></p>
                  <p id="view-disemak-jawatan" class="text-[7px] text-slate-600"></p>
                </div>
              </div>

              <!-- Disahkan -->
              <div class="flex flex-col justify-between h-10">
                <div>
                  <span class="font-semibold text-slate-700 block">Disahkan Oleh:</span>
                </div>
                <div>
                  <div class="border-b border-slate-400 w-3/4 mb-0.5"></div>
                  <p id="view-disahkan-nama" class="font-bold text-slate-900 uppercase text-[7.5px]"></p>
                  <p id="view-disahkan-jawatan" class="text-[7px] text-slate-600"></p>
                </div>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>

  </main>

  <!-- Notification Toast Container -->
  <div id="toast" class="no-print fixed bottom-5 right-5 z-50 bg-slate-900 text-white text-xs px-4 py-3 rounded-lg shadow-xl hidden transition-all duration-300 flex items-center space-x-2 border border-slate-700">
    <i data-lucide="check-circle-2" class="w-4 h-4 text-emerald-400"></i>
    <span id="toast-message">Mesej Notifikasi</span>
  </div>

  <!-- Application Logic JavaScript -->
  <script>
    // Initial Dynamic Photos State
    let uploadedPhotos = [
      {
        id: 1,
        url: 'https://images.unsplash.com/photo-1577896851231-70ef18881754?auto=format&fit=crop&w=600&q=80',
        caption: 'Pendaftaran & Taklimat Ringkas Program'
      },
      {
        id: 2,
        url: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=600&q=80',
        caption: 'Slot Ceramah & Penerangan Modul Utama'
      },
      {
        id: 3,
        url: 'https://images.unsplash.com/photo-1517245386807-bb43f82c33c4?auto=format&fit=crop&w=600&q=80',
        caption: 'Sesi Amali & Perbincangan Kumpulan'
      },
      {
        id: 4,
        url: 'https://images.unsplash.com/photo-1524178232363-1fb2b075b655?auto=format&fit=crop&w=600&q=80',
        caption: 'Majlis Penutupan & Penyampaian Sijil'
      }
    ];

    // Current active tab state for mobile
    let activeMobileTab = 'form';

    // Preset Header Logos
    const logoPresets = {
      kpm: "https://upload.wikimedia.org/wikipedia/commons/thumb/9/94/Coat_of_arms_of_Malaysia.svg/300px-Coat_of_arms_of_Malaysia.svg.png",
      jpn: "https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Jabatan_Pendidikan_Negeri_Selangor.png/300px-Jabatan_Pendidikan_Negeri_Selangor.png"
    };

    // Helper Selector Shortcut
    function id(elemId) {
      return document.getElementById(elemId);
    }

    // Dynamic Responsive Auto Switcher Listener
    function handleViewportResize() {
      const isDesktop = window.innerWidth >= 1024;
      const formEl = id('opr-form-container');
      const previewEl = id('preview-wrapper');

      if (!formEl || !previewEl) return;

      if (isDesktop) {
        // Desktop View: Display both side-by-side
        formEl.classList.remove('hidden');
        previewEl.classList.remove('hidden');
        previewEl.classList.add('flex');
      } else {
        // Mobile View: Respect current active tab
        switchTab(activeMobileTab);
      }
    }

    // Toggle Mobile Drawer Navigation
    function toggleMobileDrawer(open) {
      const drawer = id('mobile-drawer');
      const backdrop = id('mobile-drawer-backdrop');

      if (!drawer || !backdrop) return;

      if (open) {
        backdrop.classList.remove('hidden');
        setTimeout(() => backdrop.classList.remove('opacity-0'), 10);
        drawer.classList.remove('-translate-x-full');
      } else {
        drawer.classList.add('-translate-x-full');
        backdrop.classList.add('opacity-0');
        setTimeout(() => backdrop.classList.add('hidden'), 300);
      }
    }

    // Default Date Selector to Today's Date
    function setDefaultDateToday() {
      const dateEl = id('input-tarikh-date');
      if (!dateEl) return;
      const today = new Date();
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0');
      const day = String(today.getDate()).padStart(2, '0');
      dateEl.value = `${year}-${month}-${day}`;
    }

    // Format Date String to Formal Uppercase Malay Text
    function formatMalayDate(dateString) {
      if (!dateString) return "-";
      const parts = dateString.split("-");
      if (parts.length !== 3) return dateString;

      const day = parseInt(parts[2], 10);
      const monthIdx = parseInt(parts[1], 10) - 1;
      const year = parts[0];

      const bulanMelayu = [
        "JANUARI", "FEBRUARI", "MAC", "APRIL", "MEI", "JUN",
        "JULAI", "OGOS", "SEPTEMBER", "OKTOBER", "NOVEMBER", "DISEMBER"
      ];

      return `${day} ${bulanMelayu[monthIdx] || ''} ${year}`;
    }

    // Handle Custom Location Entry Visibility
    function handleTempatChange() {
      const selectEl = id('input-tempat-select');
      const customContainer = id('custom-tempat-container');
      if (!selectEl || !customContainer) return;
      
      if (selectEl.value === 'custom') {
        customContainer.classList.remove('hidden');
      } else {
        customContainer.classList.add('hidden');
      }
    }

    // Initialize application on load & attach event listeners
    window.onload = function() {
      lucide.createIcons();
      setDefaultDateToday();
      autoFillInitialAI();
      renderPhotoControls();
      updateOPR();
      handleViewportResize();

      // Listen to window resize events for instant responsive auto-switching
      window.addEventListener('resize', handleViewportResize);
    };

    // Switch Mobile Tabs
    function switchTab(tab) {
      activeMobileTab = tab;
      const isDesktop = window.innerWidth >= 1024;
      const formEl = id('opr-form-container');
      const previewEl = id('preview-wrapper');
      const btnForm = id('btn-tab-form');
      const btnPreview = id('btn-tab-preview');

      if (!formEl || !previewEl) return;

      if (isDesktop) return; // Keep side-by-side on desktop

      if (tab === 'form') {
        formEl.classList.remove('hidden');
        previewEl.classList.add('hidden');
        previewEl.classList.remove('flex');
        if (btnForm && btnPreview) {
          btnForm.className = "flex-1 py-2 text-center rounded-lg bg-blue-600 text-white font-semibold transition shadow-sm";
          btnPreview.className = "flex-1 py-2 text-center rounded-lg text-slate-700 hover:bg-slate-300 transition";
        }
      } else {
        formEl.classList.add('hidden');
        previewEl.classList.remove('hidden');
        previewEl.classList.add('flex');
        if (btnForm && btnPreview) {
          btnPreview.className = "flex-1 py-2 text-center rounded-lg bg-blue-600 text-white font-semibold transition shadow-sm";
          btnForm.className = "flex-1 py-2 text-center rounded-lg text-slate-700 hover:bg-slate-300 transition";
        }
      }
    }

    // Dynamic Header Logo Handling
    function toggleLogoInput() {
      const typeEl = id('input-logo-type');
      const customContainer = id('custom-logo-container');
      if (!typeEl || !customContainer) return;

      if (typeEl.value === 'custom') {
        customContainer.classList.remove('hidden');
      } else {
        customContainer.classList.add('hidden');
      }
    }

    // Sync Input Form Data with A4 Live Preview
    function updateOPR() {
      // Header & Logo
      if (id('view-sekolah') && id('input-sekolah')) {
        id('view-sekolah').innerText = id('input-sekolah').value || 'NAMA SEKOLAH / INSTITUSI';
      }
      
      const logoType = id('input-logo-type') ? id('input-logo-type').value : 'kpm';
      const logoImg = id('view-logo-img');
      const logoContainer = id('view-logo-container');

      if (logoContainer && logoImg) {
        if (logoType === 'none') {
          logoContainer.classList.add('hidden');
        } else {
          logoContainer.classList.remove('hidden');
          if (logoType === 'custom') {
            logoImg.src = (id('input-logo-url') && id('input-logo-url').value) || logoPresets.kpm;
          } else if (logoPresets[logoType]) {
            logoImg.src = logoPresets[logoType];
          }
        }
      }

      // Metadata & Units
      const unit = id('input-unit') ? id('input-unit').value : 'Kurikulum';
      const anjuran = id('input-anjuran') ? id('input-anjuran').value : 'Sekolah';
      if (id('view-unit-anjuran')) {
        id('view-unit-anjuran').innerText = `UNIT ${unit.toUpperCase()} | ${anjuran.toUpperCase()}`;
      }
      
      if (id('view-program') && id('input-program')) {
        id('view-program').innerText = id('input-program').value || 'NAMA PROGRAM / AKTIVITI';
      }
      
      // Date Processing
      const rawDate = id('input-tarikh-date') ? id('input-tarikh-date').value : '';
      if (id('view-tarikh')) {
        id('view-tarikh').innerText = formatMalayDate(rawDate);
      }

      if (id('view-masa') && id('input-masa')) {
        id('view-masa').innerText = id('input-masa').value || '-';
      }
      
      // Location Processing
      const tempatSelect = id('input-tempat-select') ? id('input-tempat-select').value : '';
      if (id('view-tempat')) {
        if (tempatSelect === 'custom') {
          id('view-tempat').innerText = (id('input-tempat-custom') && id('input-tempat-custom').value) || 'Tempat Tersuai';
        } else {
          id('view-tempat').innerText = tempatSelect || '-';
        }
      }

      if (id('view-sasaran') && id('input-sasaran')) id('view-sasaran').innerText = id('input-sasaran').value || '-';
      if (id('view-perasmi') && id('input-perasmi')) id('view-perasmi').innerText = id('input-perasmi').value || '-';

      // Section Content
      if (id('view-objektif') && id('input-objektif')) id('view-objektif').innerText = id('input-objektif').value || '-';
      if (id('view-ringkasan') && id('input-ringkasan')) id('view-ringkasan').innerText = id('input-ringkasan').value || '-';
      if (id('view-kekuatan') && id('input-kekuatan')) id('view-kekuatan').innerText = id('input-kekuatan').value || '-';
      if (id('view-kelemahan') && id('input-kelemahan')) id('view-kelemahan').innerText = id('input-kelemahan').value || '-';
      if (id('view-penambahbaikan') && id('input-penambahbaikan')) id('view-penambahbaikan').innerText = id('input-penambahbaikan').value || '-';
      if (id('view-impak') && id('input-impak')) id('view-impak').innerText = id('input-impak').value || '-';

      // Signatures
      if (id('view-disediakan-nama') && id('input-disediakan-nama')) id('view-disediakan-nama').innerText = id('input-disediakan-nama').value || 'NAMA PEGAWAI';
      if (id('view-disediakan-jawatan') && id('input-disediakan-jawatan')) id('view-disediakan-jawatan').innerText = id('input-disediakan-jawatan').value || 'JAWATAN';

      if (id('view-disemak-nama') && id('input-disemak-nama')) id('view-disemak-nama').innerText = id('input-disemak-nama').value || 'NAMA PENOLONG KANAN';
      if (id('view-disemak-jawatan') && id('input-disemak-jawatan')) id('view-disemak-jawatan').innerText = id('input-disemak-jawatan').value || 'JAWATAN';

      if (id('view-disahkan-nama') && id('input-disahkan-nama')) id('view-disahkan-nama').innerText = id('input-disahkan-nama').value || 'NAMA GURU BESAR / PENGETUA';
      if (id('view-disahkan-jawatan') && id('input-disahkan-jawatan')) id('view-disahkan-jawatan').innerText = id('input-disahkan-jawatan').value || 'JAWATAN';

      renderPhotoGridPreview();
    }

    // Photo Upload & Grid Management Controls
    function handleImageUpload(e) {
      const files = e.target.files;
      if (!files || !files.length) return;

      if (uploadedPhotos.length + files.length > 4) {
        showToast("Maksimum 4 gambar sahaja dibenarkan bagi kekal format 1 A4.");
      }

      Array.from(files).slice(0, 4 - uploadedPhotos.length).forEach(file => {
        const reader = new FileReader();
        reader.onload = function(evt) {
          uploadedPhotos.push({
            id: Date.now() + Math.random(),
            url: evt.target.result,
            caption: 'Gambar Aktiviti ' + (uploadedPhotos.length + 1)
          });
          renderPhotoControls();
          updateOPR();
        };
        reader.readAsDataURL(file);
      });
    }

    function removePhoto(index) {
      if (uploadedPhotos.length <= 2) {
        showToast("Minimum 2 gambar diperlukan untuk susun atur laporan OPR.");
        return;
      }
      uploadedPhotos.splice(index, 1);
      renderPhotoControls();
      updateOPR();
    }

    function updateCaption(index, value) {
      uploadedPhotos[index].caption = value;
      updateOPR();
    }

    function renderPhotoControls() {
      const container = id('image-preview-list');
      if (!container) return;
      if (id('photo-count-badge')) id('photo-count-badge').innerText = `${uploadedPhotos.length} / 4 Gambar`;
      container.innerHTML = '';

      uploadedPhotos.forEach((photo, idx) => {
        const card = document.createElement('div');
        card.className = "relative border border-slate-200 rounded-lg p-2 bg-slate-50 space-y-1.5 shadow-sm";
        card.innerHTML = `
          <div class="relative h-20 w-full overflow-hidden rounded bg-slate-200">
            <img src="${photo.url}" class="w-full h-full object-cover">
            <button type="button" onclick="removePhoto(${idx})" class="absolute top-1 right-1 bg-red-600 hover:bg-red-700 text-white rounded-full p-1 shadow transition">
              <i data-lucide="x" class="w-3 h-3"></i>
            </button>
          </div>
          <input type="text" value="${photo.caption}" oninput="updateCaption(${idx}, this.value)" placeholder="Keterangan..." class="w-full px-2 py-1 text-[11px] border border-slate-300 rounded focus:outline-none focus:ring-1 focus:ring-blue-500">
        `;
        container.appendChild(card);
      });
      if (window.lucide) lucide.createIcons();
    }

    function renderPhotoGridPreview() {
      const grid = id('view-photo-grid');
      if (!grid) return;
      grid.innerHTML = '';

      if (uploadedPhotos.length === 2) {
        grid.className = "grid grid-cols-2 gap-1.5";
      } else if (uploadedPhotos.length === 3) {
        grid.className = "grid grid-cols-3 gap-1.5";
      } else {
        grid.className = "grid grid-cols-2 gap-1.5";
      }

      uploadedPhotos.forEach(photo => {
        const item = document.createElement('div');
        item.className = "flex flex-col items-center border border-slate-300 p-0.5 bg-slate-50 rounded-sm";
        item.innerHTML = `
          <div class="w-full h-14 sm:h-16 overflow-hidden bg-slate-200 mb-0.5">
            <img src="${photo.url}" class="w-full h-full object-cover">
          </div>
          <span class="text-[7.5px] text-slate-700 font-medium text-center line-clamp-1">${photo.caption}</span>
        `;
        grid.appendChild(item);
      });
    }

    // Contextual Malay AI Content Generator Engine
    function generateAIContent(type, programName, unit) {
      const p = (programName && programName.trim()) ? programName.trim() : "Program / Aktiviti Sekolah";
      const u = unit || "Kurikulum";
      
      const aiTemplates = {
        objektif: [
          `1. Mendedahkan peserta kepada objektif dan modul utama ${p} secara mendalam.\n2. Meningkatkan tahap kemahiran, pengetahuan serta keyakinan murid selaras dengan fokus ${u}.\n3. Memupuk semangat kerjasama dan sahsiah terpuji dalam kalangan peserta program.`,
          `1. Memastikan sasaran pencapaian ${p} dapat direalisasikan mengikut perancangan KPM.\n2. Memperkasakan potensi murid melalui pengisian bimbingan interaktif dan amali.\n3. Membina daya tahan dan motivasi diri murid dalam menghadapi cabaran pembelajaran.`
        ],
        ringkasan: [
          `Program ${p} dimulakan dengan pendaftaran peserta dan taklimat ringkas. Diikuti slot penceramahan interaktif, sesi amali LDK berkumpulan, dan diakhiri dengan majlis penutupan serta penyampaian cenderamata.`,
          `Pelaksanaan ${p} merangkumi penyampaian strategi utama, latihan praktikal terpandu dan sesi soal jawab. Semua peserta memberikan komitmen cemerlang sepanjang program berlangsung.`
        ],
        kekuatan: [
          `• Kerjasama dan komitmen padu daripada jawatankuasa pelaksana dan guru-guru.\n• Penglibatan murid amat aktif dan responsif sepanjang sesi berlangsung.\n• Pengisian program yang padat dan menepati keperluan objektif unit ${u}.`,
          `• Penceramah / fasilitator berpengalaman luas dan berupaya menarik minat peserta.\n• Sokongan berterusan daripada barisan pentadbir dan PIBG sekolah.`
        ],
        kelemahan: [
          `• Tempoh masa bagi sesi latihan amali berkumpulan agak terhad.\n• Kawalan siar raya (*PA System*) mengalami sedikit gangguan teknikal di awal majlis.`,
          `• Ruang lokasi agak padat disebabkan penyertaan murid yang amat memberangsangkan.`
        ],
        penambahbaikan: [
          `• Memanjangkan tempoh slot latihan amali bagi pelaksanaan ${p} pada masa akan datang.\n• Melaksanakan ujian teknikal peralatan siar raya lebih awal sebelum majlis bermula.`,
          `• Memilih lokasi yang lebih luas dan selesa untuk aktiviti fizikal peserta.`
        ],
        impak: [
          `• 95% peserta menunjukkan peningkatan kefahaman dan motivasi selepas mengikuti ${p}.\n• Membudayakan sikap positif, kerja berpasukan dan kecemerlangan di peringkat sekolah.`,
          `• Berjaya mencapai objektif yang ditetapkan oleh unit ${u} secara menyeluruh.`
        ]
      };

      const options = aiTemplates[type] || [`Pengisian khas bagi ${p}`];
      return options[Math.floor(Math.random() * options.length)];
    }

    function generateAISpecific(fieldName) {
      const prog = id('input-program') ? id('input-program').value : '';
      const unit = id('input-unit') ? id('input-unit').value : '';
      const text = generateAIContent(fieldName, prog, unit);
      if (id(`input-${fieldName}`)) {
        id(`input-${fieldName}`).value = text;
        updateOPR();
        showToast(`Cadangan AI bagi "${fieldName.toUpperCase()}" telah dijana!`);
      }
    }

    function autoFillInitialAI() {
      const prog = id('input-program') ? id('input-program').value : '';
      const unit = id('input-unit') ? id('input-unit').value : '';
      if (id('input-objektif')) id('input-objektif').value = generateAIContent('objektif', prog, unit);
      if (id('input-ringkasan')) id('input-ringkasan').value = generateAIContent('ringkasan', prog, unit);
      if (id('input-kekuatan')) id('input-kekuatan').value = generateAIContent('kekuatan', prog, unit);
      if (id('input-kelemahan')) id('input-kelemahan').value = generateAIContent('kelemahan', prog, unit);
      if (id('input-penambahbaikan')) id('input-penambahbaikan').value = generateAIContent('penambahbaikan', prog, unit);
      if (id('input-impak')) id('input-impak').value = generateAIContent('impak', prog, unit);
    }

    function autoFillWholeFormAI() {
      autoFillInitialAI();
      updateOPR();
      showToast("Keseluruhan laporan telah dijana semula berasaskan Nama Program!");
    }

    function resetForm() {
      const form = id('opr-form');
      if (form) form.reset();
      setDefaultDateToday();
      handleTempatChange();
      toggleLogoInput();
      autoFillInitialAI();
      updateOPR();
      showToast("Borang telah diset semula!");
    }

    function showToast(msg) {
      const toast = id('toast');
      if (!toast) return;
      if (id('toast-message')) id('toast-message').innerText = msg;
      toast.classList.remove('hidden');
      setTimeout(() => {
        toast.classList.add('hidden');
      }, 3000);
    }
  </script>
</body>
</html>
