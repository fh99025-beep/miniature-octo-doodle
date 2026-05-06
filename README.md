<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Manajemen Perpustakaan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- QuaggaJS untuk Scanner Barcode -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/quagga/0.12.1/quagga.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        
        /* Perbaikan CSS untuk kontainer scanner agar video memenuhi area */
        #interactive.viewport { 
            position: relative; 
            width: 100%; 
            height: 300px; 
            overflow: hidden; 
            border-radius: 0.75rem; 
            background: #000;
        }
        #interactive.viewport video, #interactive.viewport canvas { 
            width: 100%; 
            height: 100%; 
            object-fit: cover; 
            position: absolute; 
            top: 0; 
            left: 0; 
        }
        .drawingBuffer { display: none; } /* Sembunyikan canvas debug bawaan quagga */
    </style>
</head>
<body class="bg-gray-50 text-gray-900">

    <!-- Auth Wrapper -->
    <div id="login-screen" class="min-h-screen flex items-center justify-center p-4">
        <div class="bg-white p-8 rounded-2xl shadow-xl w-full max-w-md border border-gray-100">
            <div class="text-center mb-8">
                <div class="inline-flex items-center justify-center w-16 h-16 bg-blue-100 text-blue-600 rounded-full mb-4">
                    <i data-lucide="library" class="w-8 h-8"></i>
                </div>
                <h1 class="text-2xl font-bold text-gray-800">Admin Perpustakaan</h1>
                <p class="text-gray-500">Silakan masuk ke akun Anda</p>
            </div>
            <form id="login-form" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Username</label>
                    <input type="text" id="username" class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="admin">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Password</label>
                    <input type="password" id="password" class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="••••••••">
                </div>
                <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 rounded-lg transition duration-200">
                    Masuk Sekarang
                </button>
            </form>
        </div>
    </div>

    <!-- Main App Wrapper -->
    <div id="main-app" class="hidden min-h-screen flex flex-col md:flex-row">
        
        <!-- Sidebar -->
        <aside class="w-full md:w-64 bg-white border-r border-gray-200 flex-shrink-0">
            <div class="p-6">
                <div class="flex items-center gap-3 text-blue-600 mb-8">
                    <i data-lucide="book-open-check"></i>
                    <span class="font-bold text-xl tracking-tight" id="brand-name">Lib-Admin</span>
                </div>
                <nav class="space-y-1">
                    <button onclick="showPage('dashboard')" class="nav-btn active flex items-center gap-3 w-full px-4 py-2.5 rounded-lg text-sm font-medium transition-colors">
                        <i data-lucide="layout-dashboard" class="w-4 h-4"></i> Dashboard
                    </button>
                    <button onclick="showPage('katalog')" class="nav-btn flex items-center gap-3 w-full px-4 py-2.5 rounded-lg text-sm font-medium transition-colors">
                        <i data-lucide="book" class="w-4 h-4"></i> Katalog Buku
                    </button>
                    <button onclick="showPage('tambah-buku')" class="nav-btn flex items-center gap-3 w-full px-4 py-2.5 rounded-lg text-sm font-medium transition-colors">
                        <i data-lucide="plus-circle" class="w-4 h-4"></i> Tambah Buku
                    </button>
                    <button onclick="showPage('peminjam')" class="nav-btn flex items-center gap-3 w-full px-4 py-2.5 rounded-lg text-sm font-medium transition-colors">
                        <i data-lucide="users" class="w-4 h-4"></i> Data Peminjam
                    </button>
                    <button onclick="showPage('settings')" class="nav-btn flex items-center gap-3 w-full px-4 py-2.5 rounded-lg text-sm font-medium transition-colors">
                        <i data-lucide="settings" class="w-4 h-4"></i> Pengaturan
                    </button>
                </nav>
            </div>
            <div class="mt-auto p-6 border-t">
                <button onclick="logout()" class="flex items-center gap-3 w-full px-4 py-2 text-red-600 hover:bg-red-50 rounded-lg text-sm font-medium transition-colors">
                    <i data-lucide="log-out" class="w-4 h-4"></i> Keluar
                </button>
            </div>
        </aside>

        <!-- Main Content Area -->
        <main class="flex-1 overflow-y-auto p-4 md:p-8">
            
            <!-- Dashboard Page -->
            <section id="page-dashboard" class="page-content space-y-6">
                <div class="flex justify-between items-center">
                    <h2 class="text-2xl font-bold">Ringkasan Statistik</h2>
                    <span class="text-sm text-gray-500" id="current-date"></span>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                    <div class="bg-white p-6 rounded-xl border border-gray-100 shadow-sm">
                        <p class="text-gray-500 text-sm">Total Buku</p>
                        <h3 class="text-3xl font-bold text-blue-600" id="stat-books">0</h3>
                    </div>
                    <div class="bg-white p-6 rounded-xl border border-gray-100 shadow-sm">
                        <p class="text-gray-500 text-sm">Peminjam Aktif</p>
                        <h3 class="text-3xl font-bold text-green-600" id="stat-active">0</h3>
                    </div>
                    <div class="bg-white p-6 rounded-xl border border-gray-100 shadow-sm">
                        <p class="text-gray-500 text-sm">Buku Terlambat</p>
                        <h3 class="text-3xl font-bold text-red-600" id="stat-overdue">0</h3>
                    </div>
                </div>
            </section>

            <!-- Katalog Page -->
            <section id="page-katalog" class="page-content hidden space-y-6">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                    <h2 class="text-2xl font-bold">Katalog Buku</h2>
                    <div class="relative w-full sm:w-64">
                        <i data-lucide="search" class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400"></i>
                        <input type="text" id="search-buku" onkeyup="renderKatalog()" placeholder="Cari judul atau ISBN..." class="w-full pl-10 pr-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none">
                    </div>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6" id="book-grid"></div>
            </section>

            <!-- Tambah Buku / Scan Page -->
            <section id="page-tambah-buku" class="page-content hidden space-y-6">
                <h2 class="text-2xl font-bold">Registrasi Buku Baru</h2>
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-xl border shadow-sm space-y-4">
                        <h4 class="font-semibold flex items-center gap-2 border-b pb-2">
                            <i data-lucide="camera" class="w-5 h-5 text-blue-600"></i> Scan ISBN (Barcode)
                        </h4>
                        <!-- Container Kamera -->
                        <div id="interactive" class="viewport"></div>
                        
                        <div class="flex gap-2">
                            <button onclick="startScanner()" id="btn-start" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 rounded-lg font-medium transition">
                                Aktifkan Kamera
                            </button>
                            <button onclick="stopScanner()" id="btn-stop" class="flex-1 bg-gray-200 hover:bg-gray-300 text-gray-700 py-2 rounded-lg font-medium transition">
                                Matikan
                            </button>
                        </div>
                        <p class="text-xs text-gray-400 italic text-center">Pastikan barcode berada tepat di tengah kotak kamera.</p>
                    </div>

                    <form id="form-buku" class="bg-white p-6 rounded-xl border shadow-sm space-y-4">
                        <h4 class="font-semibold border-b pb-2">Data Manual Buku</h4>
                        <div class="grid grid-cols-2 gap-4">
                            <div class="col-span-2">
                                <label class="text-sm font-medium block">ISBN / Barcode</label>
                                <input type="text" id="input-isbn" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500" required placeholder="Hasil scan muncul di sini...">
                            </div>
                            <div class="col-span-2">
                                <label class="text-sm font-medium block">Judul Buku</label>
                                <input type="text" id="input-judul" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500" required placeholder="Contoh: Matematika Kelas X">
                            </div>
                            <div>
                                <label class="text-sm font-medium block">Penulis</label>
                                <input type="text" id="input-penulis" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500" placeholder="Nama penulis...">
                            </div>
                            <div>
                                <label class="text-sm font-medium block">Kategori</label>
                                <select id="input-kategori" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500">
                                    <option value="Buku Mata Pelajaran">Buku Mata Pelajaran</option>
                                    <option value="Fiksi">Fiksi</option>
                                    <option value="Sains">Sains</option>
                                    <option value="Sejarah">Sejarah</option>
                                    <option value="Teknologi">Teknologi</option>
                                </select>
                            </div>
                        </div>
                        <button type="submit" class="w-full bg-green-600 hover:bg-green-700 text-white py-3 rounded-lg font-bold transition">Simpan Buku</button>
                    </form>
                </div>
            </section>

            <!-- Data Peminjam Page -->
            <section id="page-peminjam" class="page-content hidden space-y-6">
                <div class="flex justify-between items-center">
                    <h2 class="text-2xl font-bold">Data Peminjam</h2>
                    <button onclick="toggleModal('modal-peminjam')" class="bg-blue-600 text-white px-4 py-2 rounded-lg text-sm font-medium flex items-center gap-2">
                        <i data-lucide="user-plus" class="w-4 h-4"></i> Tambah Peminjam
                    </button>
                </div>
                
                <div class="bg-white rounded-xl border shadow-sm overflow-hidden overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead class="bg-gray-50 border-b">
                            <tr>
                                <th class="px-6 py-4 text-xs font-semibold text-gray-500 uppercase">Nama Peminjam</th>
                                <th class="px-6 py-4 text-xs font-semibold text-gray-500 uppercase">Judul Buku</th>
                                <th class="px-6 py-4 text-xs font-semibold text-gray-500 uppercase">Tgl Kembali</th>
                                <th class="px-6 py-4 text-xs font-semibold text-gray-500 uppercase">Status</th>
                                <th class="px-6 py-4 text-xs font-semibold text-gray-500 uppercase text-center">Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="borrower-table-body" class="divide-y divide-gray-100"></tbody>
                    </table>
                </div>
            </section>

            <!-- Settings Page -->
            <section id="page-settings" class="page-content hidden space-y-6">
                <h2 class="text-2xl font-bold">Pengaturan Aplikasi</h2>
                <div class="bg-white p-6 rounded-xl border shadow-sm max-w-2xl">
                    <div class="space-y-6">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Nama Perpustakaan</label>
                            <input type="text" id="set-lib-name" value="Perpustakaan Digital Mandiri" class="w-full px-4 py-2 border rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                        </div>
                        <button onclick="saveSettings()" class="bg-blue-600 text-white px-6 py-2 rounded-lg font-medium">Simpan Perubahan</button>
                    </div>
                </div>
            </section>
        </main>
    </div>

    <!-- Modals & Alerts -->
    <div id="modal-peminjam" class="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50 hidden">
        <div class="bg-white rounded-xl w-full max-w-md p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-bold">Form Peminjaman</h3>
                <button onclick="toggleModal('modal-peminjam')" class="p-1 hover:bg-gray-100 rounded">
                    <i data-lucide="x" class="w-5 h-5 text-gray-400"></i>
                </button>
            </div>
            <form id="form-peminjam" class="space-y-4">
                <div>
                    <label class="text-sm font-medium block">Nama Lengkap</label>
                    <input type="text" id="p-nama" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500" required>
                </div>
                <div>
                    <label class="text-sm font-medium block">Buku yang Dipinjam</label>
                    <select id="p-buku" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500"></select>
                </div>
                <div>
                    <label class="text-sm font-medium block">Tenggat Kembali</label>
                    <input type="date" id="p-tgl" class="w-full px-4 py-2 border rounded-lg mt-1 outline-none focus:ring-2 focus:ring-blue-500" required>
                </div>
                <button type="submit" class="w-full bg-blue-600 text-white py-3 rounded-lg font-bold">Daftarkan Pinjaman</button>
            </form>
        </div>
    </div>

    <div id="custom-alert" class="fixed bottom-6 right-6 px-6 py-3 bg-gray-800 text-white rounded-lg shadow-xl translate-y-20 transition-transform duration-300 pointer-events-none z-[100]">
        <span id="alert-msg">Pesan sukses!</span>
    </div>

    <script>
        // Data Global
        let books = JSON.parse(localStorage.getItem('lib_books')) || [
            { id: 1, isbn: "97860203001", judul: "Matematika SMA Kelas X", penulis: "Tim Edukasi", kategori: "Buku Mata Pelajaran", status: "Tersedia" }
        ];

        let borrowers = JSON.parse(localStorage.getItem('lib_borrowers')) || [];
        let isScannerRunning = false;

        lucide.createIcons();

        // Auth
        document.getElementById('login-form').addEventListener('submit', (e) => {
            e.preventDefault();
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            if(u === 'admin' && p === 'admin') {
                document.getElementById('login-screen').classList.add('hidden');
                document.getElementById('main-app').classList.remove('hidden');
                initApp();
            } else {
                showAlert("Akses ditolak!", "bg-red-600");
            }
        });

        function logout() { location.reload(); }

        function initApp() {
            document.getElementById('current-date').innerText = new Date().toLocaleDateString('id-ID', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' });
            updateDashboard();
            renderKatalog();
            renderBorrowers();
            populateBookSelect();
        }

        function showPage(pageId) {
            document.querySelectorAll('.page-content').forEach(p => p.classList.add('hidden'));
            document.getElementById(`page-${pageId}`).classList.remove('hidden');
            document.querySelectorAll('.nav-btn').forEach(b => {
                b.classList.remove('bg-blue-50', 'text-blue-600');
                if(b.getAttribute('onclick').includes(pageId)) b.classList.add('bg-blue-50', 'text-blue-600');
            });
            if(pageId !== 'tambah-buku') stopScanner();
        }

        // Dashboard
        function updateDashboard() {
            document.getElementById('stat-books').innerText = books.length;
            document.getElementById('stat-active').innerText = borrowers.filter(b => b.status === 'Aktif').length;
        }

        // Katalog & Data Buku
        function renderKatalog() {
            const grid = document.getElementById('book-grid');
            const search = document.getElementById('search-buku').value.toLowerCase();
            const filtered = books.filter(b => b.judul.toLowerCase().includes(search) || b.isbn.includes(search));

            grid.innerHTML = filtered.map(b => `
                <div class="bg-white rounded-xl border p-4 shadow-sm">
                    <div class="h-40 bg-gray-100 rounded mb-4 flex items-center justify-center">
                        <i data-lucide="book" class="w-10 h-10 text-gray-400"></i>
                    </div>
                    <span class="text-[10px] font-bold uppercase px-2 py-1 rounded bg-blue-50 text-blue-600">${b.kategori}</span>
                    <h4 class="font-bold mt-2 truncate">${b.judul}</h4>
                    <p class="text-xs text-gray-500">${b.penulis}</p>
                    <div class="mt-4 flex justify-between items-center">
                        <span class="text-xs font-semibold ${b.status === 'Tersedia' ? 'text-green-600' : 'text-orange-600'}">${b.status}</span>
                        <button onclick="deleteBook(${b.id})" class="text-red-400 hover:text-red-600"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                    </div>
                </div>
            `).join('');
            lucide.createIcons();
        }

        function deleteBook(id) {
            books = books.filter(b => b.id !== id);
            saveData();
            renderKatalog();
            updateDashboard();
            showAlert("Buku dihapus");
        }

        document.getElementById('form-buku').addEventListener('submit', (e) => {
            e.preventDefault();
            const b = {
                id: Date.now(),
                isbn: document.getElementById('input-isbn').value,
                judul: document.getElementById('input-judul').value,
                penulis: document.getElementById('input-penulis').value,
                kategori: document.getElementById('input-kategori').value,
                status: "Tersedia"
            };
            books.unshift(b);
            saveData();
            renderKatalog();
            updateDashboard();
            populateBookSelect();
            e.target.reset();
            showAlert("Buku disimpan!");
        });

        // Borrower
        function renderBorrowers() {
            const body = document.getElementById('borrower-table-body');
            body.innerHTML = borrowers.map(b => `
                <tr class="hover:bg-gray-50">
                    <td class="px-6 py-4 font-medium">${b.nama}</td>
                    <td class="px-6 py-4 text-sm">${b.buku}</td>
                    <td class="px-6 py-4 text-sm">${b.tgl}</td>
                    <td class="px-6 py-4"><span class="px-2 py-1 rounded-full text-[10px] font-bold ${b.status === 'Aktif' ? 'bg-green-100 text-green-700' : 'bg-gray-100 text-gray-500'}">${b.status}</span></td>
                    <td class="px-6 py-4 text-center">
                        ${b.status === 'Aktif' ? `<button onclick="returnBook(${b.id})" class="text-blue-600 text-xs font-bold">Kembalikan</button>` : '-'}
                    </td>
                </tr>
            `).join('');
        }

        function returnBook(id) {
            const row = borrowers.find(b => b.id === id);
            if(row) {
                row.status = "Kembali";
                const bk = books.find(b => b.judul === row.buku);
                if(bk) bk.status = "Tersedia";
                saveData();
                renderBorrowers();
                renderKatalog();
                updateDashboard();
            }
        }

        document.getElementById('form-peminjam').addEventListener('submit', (e) => {
            e.preventDefault();
            const buku = document.getElementById('p-buku').value;
            const p = {
                id: Date.now(),
                nama: document.getElementById('p-nama').value,
                buku: buku,
                tgl: document.getElementById('p-tgl').value,
                status: "Aktif"
            };
            const bk = books.find(b => b.judul === buku);
            if(bk) bk.status = "Dipinjam";
            borrowers.unshift(p);
            saveData();
            renderBorrowers();
            renderKatalog();
            updateDashboard();
            toggleModal('modal-peminjam');
            e.target.reset();
            showAlert("Peminjaman dicatat");
        });

        function populateBookSelect() {
            const s = document.getElementById('p-buku');
            const av = books.filter(b => b.status === 'Tersedia');
            s.innerHTML = av.map(b => `<option value="${b.judul}">${b.judul}</option>`).join('');
        }

        // SCANNER LOGIC - Perbaikan utama ada di sini
        function startScanner() {
            if (isScannerRunning) return;

            const container = document.querySelector('#interactive');
            container.innerHTML = ""; // Bersihkan kontainer sebelum mulai

            Quagga.init({
                inputStream: {
                    name: "Live",
                    type: "LiveStream",
                    target: container,
                    constraints: {
                        facingMode: "environment", // Kamera belakang
                        width: { min: 640 },
                        height: { min: 480 },
                        aspectRatio: { min: 1, max: 2 }
                    }
                },
                locator: { patchSize: "medium", halfSample: true },
                numOfWorkers: 2,
                decoder: {
                    readers: ["ean_reader", "code_128_reader", "code_39_reader"]
                },
                locate: true
            }, function (err) {
                if (err) {
                    console.error("Gagal memulai Quagga:", err);
                    showAlert("Kamera tidak ditemukan / Izin ditolak", "bg-red-600");
                    return;
                }
                Quagga.start();
                isScannerRunning = true;
                showAlert("Pemindai Aktif", "bg-blue-600");
            });

            Quagga.onDetected((res) => {
                const code = res.codeResult.code;
                if(code) {
                    document.getElementById('input-isbn').value = code;
                    showAlert("Berhasil: " + code, "bg-green-600");
                    stopScanner();
                    
                    // Beri feedback getaran jika di HP
                    if(navigator.vibrate) navigator.vibrate(100);
                }
            });
        }

        function stopScanner() {
            if (isScannerRunning) {
                Quagga.stop();
                isScannerRunning = false;
                showAlert("Kamera Dinonaktifkan");
                // Hapus video element agar bersih
                const container = document.querySelector('#interactive');
                if(container) container.innerHTML = "";
            }
        }

        // Helpers
        function saveData() {
            localStorage.setItem('lib_books', JSON.stringify(books));
            localStorage.setItem('lib_borrowers', JSON.stringify(borrowers));
        }

        function toggleModal(id) { document.getElementById(id).classList.toggle('hidden'); }

        function showAlert(msg, color = "bg-gray-800") {
            const el = document.getElementById('custom-alert');
            document.getElementById('alert-msg').innerText = msg;
            el.className = `fixed bottom-6 right-6 px-6 py-3 text-white rounded-lg shadow-xl transition-all duration-300 z-[100] ${color}`;
            el.classList.remove('translate-y-20');
            setTimeout(() => el.classList.add('translate-y-20'), 3000);
        }

        function saveSettings() {
            const name = document.getElementById('set-lib-name').value;
            document.getElementById('brand-name').innerText = name;
            showAlert("Nama aplikasi diubah!");
        }
    </script>
</body>
</html>
