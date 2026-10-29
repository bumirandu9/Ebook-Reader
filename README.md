# Ebook-Reader
New One
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="theme-color" content="#1a1a2e" />
  <title>BacaBuku - EPUB & PDF Reader (Tanpa Iklan)</title>
  <style>
    :root {
      --bg: #f8f9fa;
      --text: #212529;
      --toolbar: #ffffff;
      --border: #dee2e6;
      --accent: #0d6efd;
      --accent-hover: #0b5ed7;
      --shadow: 0 2px 12px rgba(0,0,0,0.08);
      --reader-bg: #ffffff;
    }
    [data-theme="dark"] {
      --bg: #121212;
      --text: #e9ecef;
      --toolbar: #1e1e1e;
      --border: #333;
      --accent: #4dabf7;
      --accent-hover: #339af0;
      --shadow: 0 2px 12px rgba(0,0,0,0.4);
      --reader-bg: #1a1a1a;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
      background: var(--bg);
      color: var(--text);
      height: 100vh;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    /* ===== Toolbar ===== */
    #toolbar {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 8px 12px;
      background: var(--toolbar);
      border-bottom: 1px solid var(--border);
      box-shadow: var(--shadow);
      z-index: 100;
      flex-wrap: wrap;
      min-height: 52px;
    }
    .btn {
      background: var(--accent);
      color: white;
      border: none;
      border-radius: 8px;
      padding: 8px 14px;
      font-size: 14px;
      cursor: pointer;
      display: inline-flex;
      align-items: center;
      gap: 6px;
      transition: background 0.15s;
      white-space: nowrap;
    }
    .btn:hover { background: var(--accent-hover); }
    .btn:disabled { opacity: 0.5; cursor: not-allowed; }
    .btn-ghost {
      background: transparent;
      color: var(--text);
      border: 1px solid var(--border);
    }
    .btn-ghost:hover { background: var(--border); }
    #fileName {
      flex: 1;
      font-size: 14px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      min-width: 80px;
    }
    .controls { display: flex; gap: 6px; align-items: center; }
    /* ===== Drop Zone ===== */
    #dropZone {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 20px;
      padding: 40px 20px;
      text-align: center;
      border: 3px dashed var(--border);
      margin: 20px;
      border-radius: 16px;
      transition: border-color 0.2s, background 0.2s;
    }
    #dropZone.dragover {
      border-color: var(--accent);
      background: rgba(13, 110, 253, 0.05);
    }
    #dropZone h1 { font-size: 1.8rem; font-weight: 700; }
    #dropZone p { color: #6c757d; max-width: 400px; line-height: 1.5; }
    [data-theme="dark"] #dropZone p { color: #adb5bd; }
    .icon-book { font-size: 64px; opacity: 0.7; }
    /* ===== Reader Area ===== */
    #readerArea {
      flex: 1;
      display: none;
      position: relative;
      overflow: hidden;
      background: var(--reader-bg);
    }
    #viewer {
      width: 100%;
      height: 100%;
      overflow: auto;
    }
    /* PDF canvas */
    #pdfContainer {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 16px;
      padding: 20px;
    }
    #pdfContainer canvas {
      max-width: 100%;
      box-shadow: var(--shadow);
      background: white;
    }
    /* EPUB */
    #epubViewer {
      width: 100%;
      height: 100%;
    }
    /* ===== Bottom bar (page nav) ===== */
    #bottomBar {
      display: none;
      align-items: center;
      justify-content: center;
      gap: 16px;
      padding: 10px;
      background: var(--toolbar);
      border-top: 1px solid var(--border);
    }
    #pageInfo { font-size: 14px; min-width: 100px; text-align: center; }
    /* ===== TOC Sidebar ===== */
    #tocSidebar {
      position: fixed;
      top: 0; left: 0; bottom: 0;
      width: 280px;
      max-width: 85vw;
      background: var(--toolbar);
      border-right: 1px solid var(--border);
      box-shadow: var(--shadow);
      transform: translateX(-100%);
      transition: transform 0.25s ease;
      z-index: 200;
      display: flex;
      flex-direction: column;
    }
    #tocSidebar.open { transform: translateX(0); }
    #tocHeader {
      padding: 16px;
      font-weight: 600;
      border-bottom: 1px solid var(--border);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    #tocList {
      flex: 1;
      overflow-y: auto;
      padding: 8px 0;
    }
    #tocList a {
      display: block;
      padding: 10px 16px;
      color: var(--text);
      text-decoration: none;
      font-size: 14px;
      border-left: 3px solid transparent;
    }
    #tocList a:hover { background: var(--border); }
    #tocList a.active { border-left-color: var(--accent); background: rgba(13,110,253,0.08); }
    #overlay {
      position: fixed; inset: 0;
      background: rgba(0,0,0,0.4);
      z-index: 150;
      display: none;
    }
    #overlay.show { display: block; }
    /* Loading */
    #loading {
      display: none;
      position: absolute;
      inset: 0;
      background: var(--bg);
      z-index: 50;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      gap: 12px;
    }
    .spinner {
      width: 40px; height: 40px;
      border: 3px solid var(--border);
      border-top-color: var(--accent);
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }
    /* Font size controls for EPUB */
    .font-controls { display: none; gap: 4px; }
    .font-controls.show { display: flex; }
  </style>
</head>
<body>
  <!-- Toolbar -->
  <div id="toolbar">
    <button class="btn" id="btnOpen" title="Buka file">📂 Buka File</button>
    <input type="file" id="fileInput" accept=".epub,.pdf,application/epub+zip,application/pdf" hidden />
    <span id="fileName">Belum ada file</span>
    <div class="controls">
      <button class="btn btn-ghost" id="btnToc" title="Daftar Isi" style="display:none">📑 Daftar Isi</button>
      <div class="font-controls" id="fontControls">
        <button class="btn btn-ghost" id="btnFontMinus" title="Perkecil font">A-</button>
        <button class="btn btn-ghost" id="btnFontPlus" title="Perbesar font">A+</button>
      </div>
      <button class="btn btn-ghost" id="btnTheme" title="Mode gelap/terang">🌙</button>
      <button class="btn btn-ghost" id="btnClose" title="Tutup buku" style="display:none">✕</button>
    </div>
  </div>

  <!-- Drop Zone -->
  <div id="dropZone">
    <div class="icon-book">📖</div>
    <h1>BacaBuku</h1>
    <p>Buka file <strong>EPUB</strong> atau <strong>PDF</strong> dengan drag & drop, atau klik tombol di bawah.<br>
    100% offline • Tanpa iklan • Tanpa akun</p>
    <button class="btn" id="btnOpenMain">Pilih File EPUB / PDF</button>
  </div>

  <!-- Reader -->
  <div id="readerArea">
    <div id="loading">
      <div class="spinner"></div>
      <div>Memuat buku...</div>
    </div>
    <div id="viewer">
      <div id="pdfContainer"></div>
      <div id="epubViewer"></div>
    </div>
  </div>

  <!-- Bottom navigation -->
  <div id="bottomBar">
    <button class="btn btn-ghost" id="btnPrev">← Sebelumnya</button>
    <span id="pageInfo">1 / 1</span>
    <button class="btn btn-ghost" id="btnNext">Berikutnya →</button>
  </div>

  <!-- TOC Sidebar -->
  <div id="overlay"></div>
  <div id="tocSidebar">
    <div id="tocHeader">
      <span>Daftar Isi</span>
      <button class="btn btn-ghost" id="btnCloseToc" style="padding:4px 10px">✕</button>
    </div>
    <div id="tocList"></div>
  </div>

  <!-- Libraries from CDN (first load needs internet) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/epubjs@0.3.93/dist/epub.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.8.69/pdf.min.mjs" type="module"></script>

  <script type="module">
    // PDF.js setup
    import * as pdfjsLib from 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.8.69/pdf.min.mjs';
    pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.8.69/pdf.worker.min.mjs';

    // ===== State =====
    let currentType = null; // 'pdf' | 'epub'
    let pdfDoc = null;
    let currentPage = 1;
    let totalPages = 0;
    let epubBook = null;
    let epubRendition = null;
    let fontSize = 100; // percent

    // ===== DOM =====
    const dropZone = document.getElementById('dropZone');
    const readerArea = document.getElementById('readerArea');
    const bottomBar = document.getElementById('bottomBar');
    const fileNameEl = document.getElementById('fileName');
    const pageInfo = document.getElementById('pageInfo');
    const pdfContainer = document.getElementById('pdfContainer');
    const epubViewer = document.getElementById('epubViewer');
    const loading = document.getElementById('loading');
    const tocSidebar = document.getElementById('tocSidebar');
    const tocList = document.getElementById('tocList');
    const overlay = document.getElementById('overlay');
    const btnToc = document.getElementById('btnToc');
    const fontControls = document.getElementById('fontControls');
    const btnClose = document.getElementById('btnClose');

    // ===== Theme =====
    const savedTheme = localStorage.getItem('bacabuku-theme') || 'light';
    document.documentElement.setAttribute('data-theme', savedTheme);
    updateThemeButton();

    document.getElementById('btnTheme').addEventListener('click', () => {
      const current = document.documentElement.getAttribute('data-theme');
      const next = current === 'dark' ? 'light' : 'dark';
      document.documentElement.setAttribute('data-theme', next);
      localStorage.setItem('bacabuku-theme', next);
      updateThemeButton();
    });

    function updateThemeButton() {
      const isDark = document.documentElement.getAttribute('data-theme') === 'dark';
      document.getElementById('btnTheme').textContent = isDark ? '☀️' : '🌙';
    }

    // ===== File Opening =====
    document.getElementById('btnOpen').addEventListener('click', () => document.getElementById('fileInput').click());
    document.getElementById('btnOpenMain').addEventListener('click', () => document.getElementById('fileInput').click());

    document.getElementById('fileInput').addEventListener('change', (e) => {
      if (e.target.files[0]) openFile(e.target.files[0]);
    });

    // Drag & Drop
    ['dragenter', 'dragover'].forEach(evt => {
      dropZone.addEventListener(evt, (e) => {
        e.preventDefault();
        dropZone.classList.add('dragover');
      });
    });
    ['dragleave', 'drop'].forEach(evt => {
      dropZone.addEventListener(evt, (e) => {
        e.preventDefault();
        dropZone.classList.remove('dragover');
      });
    });
    dropZone.addEventListener('drop', (e) => {
      const file = e.dataTransfer.files[0];
      if (file) openFile(file);
    });

    // Also allow drop on whole body when reader is open
    document.body.addEventListener('dragover', (e) => e.preventDefault());
    document.body.addEventListener('drop', (e) => {
      e.preventDefault();
      const file = e.dataTransfer.files[0];
      if (file) openFile(file);
    });

    async function openFile(file) {
      const name = file.name.toLowerCase();
      if (!name.endsWith('.pdf') && !name.endsWith('.epub')) {
        alert('Hanya mendukung file .pdf dan .epub');
        return;
      }

      closeBook();
      showLoading(true);
      fileNameEl.textContent = file.name;
      dropZone.style.display = 'none';
      readerArea.style.display = 'flex';
      bottomBar.style.display = 'flex';
      btnClose.style.display = 'inline-flex';

      try {
        if (name.endsWith('.pdf')) {
          await openPDF(file);
        } else {
          await openEPUB(file);
        }
      } catch (err) {
        console.error(err);
        alert('Gagal membuka file: ' + (err.message || err));
        closeBook();
      } finally {
        showLoading(false);
      }
    }

    function showLoading(show) {
      loading.style.display = show ? 'flex' : 'none';
    }

    // ===== PDF =====
    async function openPDF(file) {
      currentType = 'pdf';
      pdfContainer.style.display = 'flex';
      epubViewer.style.display = 'none';
      btnToc.style.display = 'none';
      fontControls.classList.remove('show');

      const arrayBuffer = await file.arrayBuffer();
      pdfDoc = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
      totalPages = pdfDoc.numPages;
      currentPage = 1;
      await renderPDFPage(currentPage);
      updatePageInfo();
    }

    async function renderPDFPage(num) {
      pdfContainer.innerHTML = '';
      const page = await pdfDoc.getPage(num);
      const scale = Math.min(1.5, (window.innerWidth - 40) / page.getViewport({ scale: 1 }).width);
      const viewport = page.getViewport({ scale });
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      canvas.height = viewport.height;
      canvas.width = viewport.width;
      pdfContainer.appendChild(canvas);
      await page.render({ canvasContext: ctx, viewport }).promise;
    }

    // ===== EPUB =====
    async function openEPUB(file) {
      currentType = 'epub';
      pdfContainer.style.display = 'none';
      epubViewer.style.display = 'block';
      epubViewer.innerHTML = '';
      btnToc.style.display = 'inline-flex';
      fontControls.classList.add('show');

      const arrayBuffer = await file.arrayBuffer();
      epubBook = ePub(arrayBuffer);
      epubRendition = epubBook.renderTo('epubViewer', {
        width: '100%',
        height: '100%',
        flow: 'paginated',
        spread: 'none'
      });

      await epubRendition.display();
      await epubBook.ready;

      // Apply saved font size
      epubRendition.themes.fontSize(fontSize + '%');

      // Build TOC
      const nav = await epubBook.loaded.navigation;
      tocList.innerHTML = '';
      if (nav && nav.toc) {
        nav.toc.forEach(item => {
          const a = document.createElement('a');
          a.href = '#';
          a.textContent = item.label.trim();
          a.addEventListener('click', (e) => {
            e.preventDefault();
            epubRendition.display(item.href);
            closeToc();
          });
          tocList.appendChild(a);
        });
      }

      // Location tracking for page info
      epubRendition.on('relocated', (location) => {
        const current = location.start.displayed.page;
        const total = location.start.displayed.total;
        pageInfo.textContent = `${current} / ${total}`;
      });

      updatePageInfo();
    }

    // ===== Navigation =====
    document.getElementById('btnPrev').addEventListener('click', async () => {
      if (currentType === 'pdf') {
        if (currentPage > 1) {
          currentPage--;
          await renderPDFPage(currentPage);
          updatePageInfo();
        }
      } else if (epubRendition) {
        epubRendition.prev();
      }
    });

    document.getElementById('btnNext').addEventListener('click', async () => {
      if (currentType === 'pdf') {
        if (currentPage < totalPages) {
          currentPage++;
          await renderPDFPage(currentPage);
          updatePageInfo();
        }
      } else if (epubRendition) {
        epubRendition.next();
      }
    });

    // Keyboard
    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'PageUp') {
        document.getElementById('btnPrev').click();
      } else if (e.key === 'ArrowRight' || e.key === 'PageDown' || e.key === ' ') {
        e.preventDefault();
        document.getElementById('btnNext').click();
      } else if (e.key === 'Escape') {
        closeToc();
      }
    });

    function updatePageInfo() {
      if (currentType === 'pdf') {
        pageInfo.textContent = `${currentPage} / ${totalPages}`;
      }
    }

    // ===== Font size (EPUB) =====
    document.getElementById('btnFontPlus').addEventListener('click', () => {
      fontSize = Math.min(200, fontSize + 10);
      if (epubRendition) epubRendition.themes.fontSize(fontSize + '%');
    });
    document.getElementById('btnFontMinus').addEventListener('click', () => {
      fontSize = Math.max(70, fontSize - 10);
      if (epubRendition) epubRendition.themes.fontSize(fontSize + '%');
    });

    // ===== TOC =====
    btnToc.addEventListener('click', () => {
      tocSidebar.classList.add('open');
      overlay.classList.add('show');
    });
    document.getElementById('btnCloseToc').addEventListener('click', closeToc);
    overlay.addEventListener('click', closeToc);

    function closeToc() {
      tocSidebar.classList.remove('open');
      overlay.classList.remove('show');
    }

    // ===== Close book =====
    btnClose.addEventListener('click', closeBook);

    function closeBook() {
      if (epubRendition) {
        epubRendition.destroy();
        epubRendition = null;
      }
      if (epubBook) {
        epubBook.destroy();
        epubBook = null;
      }
      pdfDoc = null;
      currentType = null;
      pdfContainer.innerHTML = '';
      epubViewer.innerHTML = '';
      readerArea.style.display = 'none';
      bottomBar.style.display = 'none';
      dropZone.style.display = 'flex';
      fileNameEl.textContent = 'Belum ada file';
      btnToc.style.display = 'none';
      btnClose.style.display = 'none';
      fontControls.classList.remove('show');
      closeToc();
    }

    // Touch swipe for mobile (simple)
    let touchStartX = 0;
    document.getElementById('viewer').addEventListener('touchstart', (e) => {
      touchStartX = e.changedTouches[0].screenX;
    }, { passive: true });
    document.getElementById('viewer').addEventListener('touchend', (e) => {
      const diff = e.changedTouches[0].screenX - touchStartX;
      if (Math.abs(diff) > 60) {
        if (diff < 0) document.getElementById('btnNext').click();
        else document.getElementById('btnPrev').click();
      }
    }, { passive: true });
  </script>
</body>
</html>
