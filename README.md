```
    function renderPortalGallery() {
        const gallery = getDB(STORAGE_KEYS.GALLERY, DEFAULT_GALLERY);
        const grid = document.getElementById('portalGalleryGrid');
        if (!grid) return;

        grid.innerHTML = gallery.map(item => {
            const mediaHtml = item.url
                ? `<img src="${item.url}" alt="${item.title}" onerror="this.outerHTML=\`${getGalleryIllustrationSVG(item.svgType || 'lab').replace(/"/g, '&quot;')}\`" class="w-full h-44 object-cover block">`
                : getGalleryIllustrationSVG(item.svgType || 'lab');

            return `
                <div class="bg-white rounded-2xl border border-slate-200 overflow-hidden shadow-sm hover:shadow-md transition flex flex-col">
                    ${mediaHtml}
                    <div class="p-4 flex-1 flex flex-col justify-between">
                        <p class="text-xs sm:text-sm font-bold text-slate-800 leading-snug">${item.title}</p>
                        <span class="text-[11px] text-slate-400 mt-2 inline-flex items-center gap-1">
                            <i data-lucide="camera" class="w-3 h-3"></i> Dokumentasi CBT Sekolah
                        </span>
                    </div>
                </div>
            `;
        }).join('');
    }

    function openInstallModal(platform) {
        const modal = document.getElementById('installPlatformModal');
        const title = document.getElementById('installPlatformTitle');
        const content = document.getElementById('installPlatformContent');

        if (platform === 'android') {
            title.innerText = 'Instalasi Aplikasi di HP / Tablet Android';
            content.innerHTML = `
                <p>Pasang <strong>CBT-PRO</strong> di perangkat Android agar berjalan dalam mode layar penuh tanpa bilah navigasi browser:</p>
                <ol class="list-decimal pl-5 space-y-2 text-xs text-slate-700">
                    <li>Buka link portal ujian ini melalui <strong>Google Chrome</strong> di HP Android Anda.</li>
                    <li>Ketuk ikon menu <strong>Titik Tiga (⋮)</strong> di pojok kanan atas layar.</li>
                    <li>Pilih <strong>"Tambahkan ke Layar Utama"</strong> atau <strong>"Install Aplikasi"</strong>.</li>
                    <li>Konfirmasi dengan menekan <strong>Install / Tambah</strong>. Ikon aplikasi CBT akan muncul di layar utama HP Anda.</li>
                </ol>
            `;
        } else if (platform === 'ios') {
            title.innerText = 'Instalasi Aplikasi di iPhone / iPad (iOS)';
            content.innerHTML = `
                <p>Untuk memasang aplikasi ujian pada perangkat iPhone atau iPad:</p>
                <ol class="list-decimal pl-5 space-y-2 text-xs text-slate-700">
                    <li>Buka halaman ujian ini menggunakan browser bawaan <strong>Safari</strong>.</li>
                    <li>Ketuk tombol <strong>Bagikan (Share)</strong> di bagian bawah Safari.</li>
                    <li>Gulir menu ke bawah, lalu ketuk <strong>"Tambah ke Layar Utama" (Add to Home Screen)</strong>.</li>
                    <li>Ketuk <strong>Tambah</strong> di pojok kanan atas. Aplikasi siap dibuka langsung dari Home Screen.</li>
                </ol>
            `;
        } else {
            title.innerText = 'Instalasi Aplikasi di Komputer (Windows / Mac)';
            content.innerHTML = `
                <p>Untuk menjalankan aplikasi ujian secara mandiri di komputer laboratorium atau laptop:</p>
                <ol class="list-decimal pl-5 space-y-2 text-xs text-slate-700">
                    <li>Buka link aplikasi ini menggunakan <strong>Google Chrome</strong> atau <strong>Microsoft Edge</strong>.</li>
                    <li>Klik ikon <strong>Install App</strong> di ujung kanan bilah alamat URL.</li>
                    <li>Atau klik tombol <strong>Unduh File .HTML</strong> di bawah untuk menjalankan aplikasi secara lokal tanpa internet.</li>
                </ol>
            `;
        }

        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function closeInstallModal() {
        const modal = document.getElementById('installPlatformModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
    }

    function downloadStandaloneApp() {
        const htmlDoc = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;
        const blob = new Blob([htmlDoc], { type: 'text/html;charset=utf-8' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'CBT-PRO-Aplikasi-Ujian.html';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('File aplikasi mandiri (.HTML) berhasil diunduh!', 'success');
    }

    // =================================================================
    // H. AUTENTIKASI 3 PERAN (SISWA KE DASHBOARD, PENGAWAS, ADMIN)
    // =================================================================
    function openLoginModal(role) {
        const modal = document.getElementById('loginModal');
        const roleField = document.getElementById('loginRoleField');
        const badge = document.getElementById('loginModalBadge');
        const title = document.getElementById('loginModalTitle');
        const desc = document.getElementById('loginModalDesc');
        const userLabel = document.getElementById('loginUsernameLabel');
        const submitBtn = document.getElementById('loginSubmitButton');

        document.getElementById('loginUsernameField').value = '';
        document.getElementById('loginPasswordField').value = '';
        roleField.value = role;

        if (role === 'admin') {
            badge.className = 'px-3 py-1 rounded-full bg-indigo-100 text-indigo-700 text-xs font-bold uppercase';
            badge.innerText = 'Administrator';
            title.innerText = 'Login Administrator';
            desc.innerText = 'Masukkan Username dan Password Administrator sekolah.';
            userLabel.innerText = 'Username Administrator';
            submitBtn.className = 'w-full py-3.5 px-6 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-bold text-sm transition shadow-lg mt-2';
        } else if (role === 'pengawas') {
            badge.className = 'px-3 py-1 rounded-full bg-amber-100 text-amber-800 text-xs font-bold uppercase';
            badge.innerText = 'Pengawas Ruang';
            title.innerText = 'Login Dashboard Pengawas';
            desc.innerText = 'Masukkan Username dan Password Pengawas Ruang yang terdaftar.';
            userLabel.innerText = 'Username Pengawas';
            submitBtn.className = 'w-full py-3.5 px-6 rounded-xl bg-amber-500 hover:bg-amber-600 text-white font-bold text-sm transition shadow-lg mt-2';
        } else {
            badge.className = 'px-3 py-1 rounded-full bg-blue-100 text-blue-700 text-xs font-bold uppercase';
            badge.innerText = 'Peserta Ujian';
            title.innerText = 'Login Dashboard Siswa';
            desc.innerText = 'Masukkan NISN/Username dan Password Kartu Login. Token dimasukkan saat Anda siap mulai ujian.';
            userLabel.innerText = 'NISN / Username Peserta';
            submitBtn.className = 'w-full py-3.5 px-6 rounded-xl bg-blue-600 hover:bg-blue-700 text-white font-bold text-sm transition shadow-lg mt-2';
        }

        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function closeLoginModal() {
        const modal = document.getElementById('loginModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
    }

    function processLogin(e) {
        e.preventDefault();
        const role = document.getElementById('loginRoleField').value;
        const userInput = document.getElementById('loginUsernameField').value.trim();
        const passInput = document.getElementById('loginPasswordField').value.trim();
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);

        if (role === 'admin') {
            const validUser = settings.adminUsername || 'admin';
            const validPass = settings.adminPassword || 'admin123';
            if (userInput === validUser && passInput === validPass) {
                activeSession = { role: 'admin', user: { name: 'Administrator', username: validUser } };
                recordSystemLog('admin', 'Administrator', validUser, 'Pusat', 'LOGIN', 'Administrator masuk ke panel kendali utama.');
                closeLoginModal();
                showToast('Berhasil masuk sebagai Administrator.', 'success');
                navigateTo('admin');
            } else {
                showToast('Username atau Password Administrator tidak valid!', 'error');
            }
            return;
        }

        if (role === 'pengawas') {
            const proctors = getDB(STORAGE_KEYS.PROCTORS, DEFAULT_PROCTORS);
            const foundProctor = proctors.find(p => p.username === userInput && p.password === passInput);
            if (foundProctor) {
                activeSession = { role: 'pengawas', user: foundProctor };
                recordSystemLog('pengawas', foundProctor.name, foundProctor.username, foundProctor.room, 'LOGIN', `Pengawas masuk ke Dashboard Pengawas (${foundProctor.room}).`);
                closeLoginModal();
                showToast(`Selamat bertugas, ${foundProctor.name}!`, 'success');
                navigateTo('pengawas');
            } else {
                showToast('Username atau Password Pengawas tidak terdaftar!', 'error');
            }
            return;
        }

        if (role === 'siswa') {
            const students = getDB(STORAGE_KEYS.STUDENTS, []);
            const student = students.find(s =>
                (s.username.toLowerCase() === userInput.toLowerCase() || s.nisn === userInput) &&
                s.password === passInput
            );

            if (!student) {
                showToast('NISN/Username atau Password tidak cocok dengan Kartu Login!', 'error');
                return;
            }

            const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
            const prev = liveMap[student.id] || {};
            liveMap[student.id] = {
                ...prev,
                status: prev.status === 'Selesai' ? 'Selesai' : 'Di Dashboard',
                lastLoginAt: new Date().toLocaleTimeString('id-ID'),
                updatedAt: new Date().toLocaleTimeString('id-ID')
            };
            setDB(STORAGE_KEYS.LIVE_STATUS, liveMap);

            activeSession = { role: 'siswa', user: student };
            recordSystemLog('siswa', student.name, student.nisn, `${student.className} • ${student.room}`, 'LOGIN', 'Siswa masuk ke Dashboard Peserta Ujian.');
            closeLoginModal();
            showToast(`Selamat datang, ${student.name}! Silakan pilih mata pelajaran ujian.`, 'success');
            navigateTo('siswa-dashboard');
        }
    }

    function logoutSession() {
        if (activeSession) {
            if (activeSession.role === 'siswa') {
                const s = activeSession.user;
                const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
                if (liveMap[s.id] && liveMap[s.id].status === 'Di Dashboard') {
                    liveMap[s.id].status = 'Keluar / Offline';
                    liveMap[s.id].updatedAt = new Date().toLocaleTimeString('id-ID');
                    setDB(STORAGE_KEYS.LIVE_STATUS, liveMap);
                }
                recordSystemLog('siswa', s.name, s.nisn, `${s.className} • ${s.room}`, 'LOGOUT', 'Siswa keluar (logout) dari Dashboard Siswa.');
            } else if (activeSession.role === 'pengawas') {
                const p = activeSession.user;
                recordSystemLog('pengawas', p.name, p.username, p.room, 'LOGOUT', `Pengawas keluar (logout) dari sistem.`);
            } else if (activeSession.role === 'admin') {
                recordSystemLog('admin', 'Administrator', activeSession.user.username || 'admin', 'Pusat', 'LOGOUT', 'Administrator keluar (logout) dari sistem.');
            }
        }
        activeSession = null;
        sessionStorage.removeItem(SESSION_STATE_KEY);
        showToast('Anda telah keluar dari sistem.', 'info');
        navigateTo('portal');
    }

    // =================================================================
    // I. DASHBOARD SISWA: PROFIL KARTU, PILIH MAPEL & INPUT TOKEN
    // =================================================================
    function switchStudentDashTab(tabName) {
        currentStudentDashTab = tabName;
        ['ujian', 'profil', 'panduan'].forEach(t => {
            document.getElementById(`stuDashTab-${t}`).classList.add('hidden');
            const btn = document.getElementById(`stuTabBtn-${t}`);
            btn.className = 'px-4 py-2.5 rounded-xl text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100 transition flex items-center gap-2';
        });
        document.getElementById(`stuDashTab-${tabName}`).classList.remove('hidden');
        const activeBtn = document.getElementById(`stuTabBtn-${tabName}`);
        activeBtn.className = 'px-4 py-2.5 rounded-xl text-xs sm:text-sm font-bold bg-blue-600 text-white transition flex items-center gap-2';
        saveSessionState();
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function renderStudentDashboard() {
        if (!activeSession || activeSession.role !== 'siswa') return;
        const s = activeSession.user;
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const myLive = liveMap[s.id] || {};
        const completedPkgs = myLive.completedPackages || {};

        document.getElementById('stuDashWelcomeName').innerText = `Selamat Datang, ${s.name}`;
        document.getElementById('stuDashWelcomeSub').innerText = `NISN: ${s.nisn} • Kelas ${s.className} • ${s.room}`;
        document.getElementById('stuSideName').innerText = s.name;
        document.getElementById('stuSideNisn').innerText = s.nisn;
        document.getElementById('stuSideClass').innerText = s.className;
        document.getElementById('stuSideRoom').innerText = s.room;

        document.getElementById('stuCardSchoolName').innerText = settings.schoolName;
        document.getElementById('stuProfName').innerText = s.name;
        document.getElementById('stuProfNisn').innerText = s.nisn;
        document.getElementById('stuProfClass').innerText = s.className;
        document.getElementById('stuProfRoom').innerText = s.room;
        document.getElementById('stuProfUser').innerText = s.username;
        document.getElementById('stuProfLoginTime').innerText = `Aktif (Login pukul ${myLive.lastLoginAt || new Date().toLocaleTimeString('id-ID')})`;

        const relevantPackages = packages.filter(pkg => {
            if (!pkg.targetGroup || pkg.targetGroup === 'ALL') return true;
            return pkg.targetGroup === s.className || pkg.targetGroup === s.room;
        });

        document.getElementById('stuAssignedCountBadge').innerText = `${relevantPackages.length} Mata Pelajaran`;

        const listContainer = document.getElementById('studentAssignmentsList');
        if (relevantPackages.length === 0) {
            listContainer.innerHTML = `
                <div class="bg-white rounded-2xl border border-slate-200 p-8 text-center space-y-2">
                    <div class="w-12 h-12 rounded-2xl bg-slate-100 text-slate-400 flex items-center justify-center mx-auto">
                        <i data-lucide="calendar-clock" class="w-6 h-6"></i>
                    </div>
                    <h4 class="font-bold text-slate-800 text-sm">Belum Ada Penugasan Ujian</h4>
                    <p class="text-xs text-slate-500">Saat ini belum ada mata pelajaran yang dijadwalkan untuk kelas Anda. Silakan menunggu informasi dari Pengawas / Administrator.</p>
                </div>
            `;
        } else {
            listContainer.innerHTML = relevantPackages.map(pkg => {
                const isDone = !!completedPkgs[pkg.id];
                const isAssigned = pkg.status === 'ASSIGNED';
                const qCount = (pkg.questions || []).length;

                let statusBadge = '';
                let actionButton = '';

                if (isDone) {
                    const res = completedPkgs[pkg.id];
                    statusBadge = `<span class="px-3 py-1 rounded-full bg-emerald-100 text-emerald-700 text-xs font-bold inline-flex items-center gap-1"><i data-lucide="check-circle2" class="w-3.5 h-3.5"></i> Selesai Dikerjakan</span>`;
                    actionButton = `
                        <div class="text-right">
                            <span class="text-xs font-bold text-emerald-700 block">Terkirim (${res.finishedAt || '-'})</span>
                            ${settings.showScoreToStudent ? `<span class="text-xs font-extrabold text-blue-700">Nilai: ${res.score}</span>` : ''}
                        </div>
                    `;
                } else if (isAssigned && qCount > 0) {
                    statusBadge = `<span class="px-3 py-1 rounded-full bg-blue-100 text-blue-700 text-xs font-bold inline-flex items-center gap-1.5"><span class="w-2 h-2 rounded-full bg-blue-600 animate-ping"></span> Siap Dikerjakan (Assigned)</span>`;
                    actionButton = `
                        <button onclick="openStudentTokenModal('${pkg.id}')" class="px-5 py-2.5 rounded-xl bg-blue-600 hover:bg-blue-700 text-white text-xs sm:text-sm font-bold transition shadow-sm flex items-center gap-2">
                            <i data-lucide="key-round" class="w-4 h-4"></i> Mulai Ujian &amp; Input Token
                        </button>
                    `;
                } else {
                    statusBadge = `<span class="px-3 py-1 rounded-full bg-amber-100 text-amber-800 text-xs font-bold inline-flex items-center gap-1"><i data-lucide="clock" class="w-3.5 h-3.5"></i> Terjadwal • Menunggu Assign</span>`;
                    actionButton = `
                        <button disabled class="px-4 py-2.5 rounded-xl bg-slate-100 text-slate-400 text-xs font-bold cursor-not-allowed flex items-center gap-1.5">
                            <i data-lucide="lock" class="w-3.5 h-3.5"></i> Menunggu Dibuka Admin
                        </button>
                    `;
                }

                return `
                    <div class="bg-white rounded-2xl border ${isAssigned && !isDone ? 'border-blue-300 shadow-sm' : 'border-slate-200'} p-5 transition space-y-4">
                        <div class="flex flex-col sm:flex-row sm:items-start justify-between gap-3">
                            <div class="space-y-1 min-w-0">
                                <span class="text-[11px] font-bold uppercase tracking-wider text-slate-400 block truncate">${pkg.examTitle}</span>
                                <h4 class="text-base sm:text-lg font-extrabold text-slate-900 break-words">${pkg.subjectName}</h4>
                                <div class="flex flex-wrap items-center gap-3 text-xs text-slate-600 pt-1">
                                    <span class="inline-flex items-center gap-1"><i data-lucide="calendar" class="w-3.5 h-3.5 text-blue-600"></i> ${pkg.scheduleDate || 'Sesuai Jadwal'}</span>
                                    <span class="inline-flex items-center gap-1"><i data-lucide="clock" class="w-3.5 h-3.5 text-amber-600"></i> ${pkg.scheduleTime || '-'}</span>
                                    <span class="inline-flex items-center gap-1"><i data-lucide="timer" class="w-3.5 h-3.5 text-indigo-600"></i> Durasi: <strong>${pkg.durationMinutes} Menit</strong></span>
                                    <span class="inline-flex items-center gap-1"><i data-lucide="file-question" class="w-3.5 h-3.5 text-emerald-600"></i> <strong>${qCount} Soal</strong></span>
                                </div>
                            </div>
                            <div class="shrink-0">
                                ${statusBadge}
                            </div>
                        </div>
                        <div class="pt-3 border-t border-slate-100 flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                            <span class="text-[11px] text-slate-500">
                                Target Peserta: <strong>${pkg.targetGroup === 'ALL' ? 'Semua Kelas & Ruangan' : pkg.targetGroup}</strong>
                            </span>
                            <div class="shrink-0">
                                ${actionButton}
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        const historyContainer = document.getElementById('stuMyHistoryList');
        const doneKeys = Object.keys(completedPkgs);
        if (doneKeys.length === 0) {
            historyContainer.innerHTML = `<div class="p-6 text-center text-slate-400 bg-slate-50 rounded-2xl">Belum ada ujian yang diselesaikan.</div>`;
        } else {
            historyContainer.innerHTML = doneKeys.map(k => {
                const item = completedPkgs[k];
                return `
                    <div class="p-3.5 rounded-xl bg-emerald-50/60 border border-emerald-200 flex items-center justify-between gap-2">
                        <div class="min-w-0">
                            <div class="font-bold text-slate-900 truncate">${item.subjectName}</div>
                            <div class="text-[11px] text-slate-500">Selesai: ${item.finishedAt} • Benar ${item.correct} / Salah ${item.wrong}</div>
                        </div>
                        ${settings.showScoreToStudent ? `<span class="px-3 py-1 rounded-lg bg-emerald-600 text-white font-extrabold text-sm shrink-0">${item.score}</span>` : `<span class="text-xs font-bold text-emerald-700 shrink-0">Terkirim</span>`}
                    </div>
                `;
            }).join('');
        }
    }

    function openStudentTokenModal(pkgId) {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkg = packages.find(p => p.id === pkgId);
        if (!pkg) return;

        if (pkg.status !== 'ASSIGNED') {
            showToast('Mata pelajaran ini masih menunggu di-assign oleh Administrator.', 'warning');
            return;
        }
        if (!pkg.questions || pkg.questions.length === 0) {
            showToast('Soal pada paket mata pelajaran ini belum tersedia.', 'warning');
            return;
        }

        document.getElementById('tokenModalPackageId').value = pkg.id;
        document.getElementById('tokenModalExamTitle').innerText = pkg.subjectName;
        document.getElementById('tokenModalExamMeta').innerText = `${pkg.examTitle} • Durasi: ${pkg.durationMinutes} Menit • ${pkg.questions.length} Soal`;
        document.getElementById('studentExamTokenInput').value = '';

        const modal = document.getElementById('studentTokenEntryModal');
        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
        setTimeout(() => document.getElementById('studentExamTokenInput').focus(), 100);
    }

    function closeStudentTokenModal() {
        const modal = document.getElementById('studentTokenEntryModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
    }

    function confirmStartExamWithToken(e) {
        e.preventDefault();
        const pkgId = document.getElementById('tokenModalPackageId').value;
        const enteredToken = document.getElementById('studentExamTokenInput').value.trim().toUpperCase();
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);

        if (!enteredToken || enteredToken !== settings.activeToken.toUpperCase()) {
            showToast('Token Ujian tidak cocok! Periksa kembali token aktif dari Pengawas Ruangan.', 'error');
            return;
        }

        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkg = packages.find(p => p.id === pkgId);
        if (!pkg || pkg.status !== 'ASSIGNED') {
            showToast('Paket ujian sudah tidak aktif atau belum di-assign.', 'error');
            closeStudentTokenModal();
            return;
        }

        closeStudentTokenModal();
        beginStudentExam(activeSession.user, pkg);
    }

    // =================================================================
    // J. DASHBOARD ADMINISTRATOR: 7 MENU TERPISAH
    // =================================================================
    function switchAdminMenu(menuKey) {
        currentAdminMenu = menuKey;
        const menus = ['monitoring', 'soal', 'assign', 'logs', 'nilai', 'kartu', 'setelan'];

        menus.forEach(m => {
            document.getElementById(`adminSection-${m}`).classList.add('hidden');
            const btn = document.getElementById(`adminNav-${m}`);
            btn.className = 'admin-sidebar-btn flex items-center gap-3 px-3.5 py-2.5 rounded-xl text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100 transition whitespace-nowrap';
        });

        document.getElementById(`adminSection-${menuKey}`).classList.remove('hidden');
        const activeBtn = document.getElementById(`adminNav-${menuKey}`);
        activeBtn.className = 'admin-sidebar-btn flex items-center gap-3 px-3.5 py-2.5 rounded-xl text-xs sm:text-sm font-bold bg-indigo-600 text-white transition whitespace-nowrap';

        saveSessionState();
        syncAllViews();
    }

    function populateFilterDropdowns() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const classes = [...new Set(students.map(s => s.className).filter(Boolean))];
        const rooms = [...new Set(students.map(s => s.room).filter(Boolean))];

        const monFilter = document.getElementById('monRoomFilter');
        const currentMonVal = monFilter.value;
        monFilter.innerHTML = `<option value="">Semua Kelas & Ruangan</option>` +
            classes.map(c => `<option value="class:${c}">Kelas: ${c}</option>`).join('') +
            rooms.map(r => `<option value="room:${r}">Ruangan: ${r}</option>`).join('');
        monFilter.value = currentMonVal;

        const nClass = document.getElementById('nilaiClassFilter');
        const curNClass = nClass.value;
        nClass.innerHTML = `<option value="">Semua Kelas</option>` + classes.map(c => `<option value="${c}">${c}</option>`).join('');
        nClass.value = curNClass;

        const nRoom = document.getElementById('nilaiRoomFilter');
        const curNRoom = nRoom.value;
        nRoom.innerHTML = `<option value="">Semua Ruangan</option>` + rooms.map(r => `<option value="${r}">${r}</option>`).join('');
        nRoom.value = curNRoom;

        const assignTarget = document.getElementById('assignTargetGroup');
        const curAssignTarget = assignTarget.value;
        assignTarget.innerHTML = `<option value="ALL">Semua Kelas & Ruangan</option>` +
            classes.map(c => `<option value="${c}">Khusus Kelas: ${c}</option>`).join('') +
            rooms.map(r => `<option value="${r}">Khusus Ruangan: ${r}</option>`).join('');
        if (curAssignTarget) assignTarget.value = curAssignTarget;
    }

    function renderAdminMonitoring() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const search = (document.getElementById('monSearchInput')?.value || '').toLowerCase();
        const filterVal = document.getElementById('monRoomFilter')?.value || '';

        let activeCount = 0, doneCount = 0, lockedCount = 0;
        students.forEach(s => {
            const st = liveMap[s.id]?.status;
            if (st === 'Mengerjakan' || st === 'Di Dashboard') activeCount++;
            else if (st === 'Selesai') doneCount++;
            else if (st === 'TERKUNCI') lockedCount++;
        });

        document.getElementById('monStatTotal').innerText = students.length;
        document.getElementById('monStatActive').innerText = activeCount;
        document.getElementById('monStatDone').innerText = doneCount;
        document.getElementById('monStatLocked').innerText = lockedCount;

        const filtered = students.filter(s => {
            const matchText = s.name.toLowerCase().includes(search) || s.nisn.includes(search);
            let matchGroup = true;
            if (filterVal.startsWith('class:')) matchGroup = s.className === filterVal.replace('class:', '');
            if (filterVal.startsWith('room:')) matchGroup = s.room === filterVal.replace('room:', '');
            return matchText && matchGroup;
        });

        const tbody = document.getElementById('adminMonitoringTbody');
        if (filtered.length === 0) {
            tbody.innerHTML = `<tr><td colspan="7" class="py-8 text-center text-slate-400 text-xs">Tidak ada data peserta yang sesuai.</td></tr>`;
            return;
        }

        tbody.innerHTML = filtered.map(s => {
            const st = liveMap[s.id] || { status: 'Belum Masuk', progress: '-', currentSubject: '-', violations: 0 };
            let badge = `<span class="px-2.5 py-1 rounded-full bg-slate-100 text-slate-600 text-[11px] font-bold">Belum Masuk</span>`;
            if (st.status === 'Di Dashboard') {
                badge = `<span class="px-2.5 py-1 rounded-full bg-indigo-100 text-indigo-700 text-[11px] font-bold">Di Dashboard</span>`;
            } else if (st.status === 'Mengerjakan') {
                badge = `<span class="px-2.5 py-1 rounded-full bg-blue-100 text-blue-700 text-[11px] font-bold animate-pulse">Mengerjakan</span>`;
            } else if (st.status === 'TERKUNCI') {
                badge = `<span class="px-2.5 py-1 rounded-full bg-red-600 text-white text-[11px] font-bold animate-bounce">TERKUNCI + SIRINE</span>`;
            } else if (st.status === 'Selesai') {
                badge = `<span class="px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-700 text-[11px] font-bold">Selesai</span>`;
            } else if (st.status === 'Keluar / Offline') {
                badge = `<span class="px-2.5 py-1 rounded-full bg-slate-200 text-slate-700 text-[11px] font-bold">Keluar / Offline</span>`;
            }

            return `
                <tr class="hover:bg-slate-50 transition ${st.status === 'TERKUNCI' ? 'bg-red-50/70' : ''}">
                    <td class="py-3 px-4">
                        <div class="font-bold text-slate-900">${s.name}</div>
                        <div class="text-[11px] font-mono text-slate-500">NISN: ${s.nisn}</div>
                    </td>
                    <td class="py-3 px-4 text-xs text-slate-700">${s.className} • <strong>${s.room}</strong></td>
                    <td class="py-3 px-4 text-xs font-semibold text-slate-700">${st.currentSubject || '-'}</td>
                    <td class="py-3 px-4">${badge}</td>
                    <td class="py-3 px-4 font-mono text-xs font-semibold">${st.progress || '-'}</td>
                    <td class="py-3 px-4">
                        <span class="px-2 py-0.5 rounded-full text-xs font-bold ${st.violations > 0 ? 'bg-red-100 text-red-700' : 'bg-slate-100 text-slate-600'}">
                            ${st.violations || 0}x
                        </span>
                    </td>
                    <td class="py-3 px-4 text-right space-x-1 whitespace-nowrap">
                        ${st.status === 'TERKUNCI' ? `
                            <button onclick="remoteUnlockStudentById('${s.id}')" class="px-2.5 py-1.5 rounded-lg bg-amber-500 hover:bg-amber-600 text-white text-[11px] font-bold transition">
                                Matikan Sirine &amp; Buka Kunci
                            </button>
                        ` : ''}
                        <button onclick="resetStudentExamSession('${s.id}')" class="px-2.5 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-700 text-[11px] font-bold transition">
                            Reset Sesi
                        </button>
                    </td>
                </tr>
            `;
        }).join('');
    }

    function remoteUnlockStudentById(studentId) {
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const stu = students.find(s => s.id === studentId);
        if (liveMap[studentId]) {
            liveMap[studentId].status = 'Mengerjakan';
            liveMap[studentId].remoteUnlocked = true;
            setDB(STORAGE_KEYS.LIVE_STATUS, liveMap);

            const actorName = activeSession?.role === 'pengawas' ? `Pengawas (${activeSession.user.name})` : 'Administrator';
            recordSystemLog(
                activeSession?.role || 'admin',
                stu ? stu.name : studentId,
                stu ? stu.nisn : '-',
                stu ? `${stu.className} • ${stu.room}` : '-',
                'UNLOCK',
                `Kunci layar & sirine dibuka jarak jauh oleh ${actorName}.`
            );

            syncAllViews();
            showToast('Sirine dimatikan & kunci layar peserta dibuka jarak jauh!', 'success');
        }
    }

    function resetStudentExamSession(studentId) {
        showConfirmDialog('Reset Sesi Ujian Peserta?', 'Status pengerjaan dan riwayat selesai peserta ini akan direset agar dapat mengikuti ujian kembali.', () => {
            const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
            delete liveMap[studentId];
            setDB(STORAGE_KEYS.LIVE_STATUS, liveMap);
            syncAllViews();
            showToast('Sesi peserta berhasil direset.', 'info');
        });
    }

    function renderAdminPackageControls() {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const selector = document.getElementById('adminPackageSelector');
        if (!selector) return;

        if (packages.length === 0) {
            selector.innerHTML = `<option value="">-- Belum Ada Paket Mata Pelajaran --</option>`;
            return;
        }

        if (!packages.some(p => p.id === selectedAdminPkgId)) {
            selectedAdminPkgId = packages[0].id;
        }

        selector.innerHTML = packages.map(p => `
            <option value="${p.id}">${p.subjectName} (${(p.questions || []).length} Soal) — [${p.status === 'ASSIGNED' ? 'DI-ASSIGN' : 'MENUNGGU ASSIGN'}]</option>
        `).join('');
        selector.value = selectedAdminPkgId;
    }

    function selectAdminQuestionPackage(pkgId) {
        selectedAdminPkgId = pkgId;
        renderAdminQuestionList();
    }

    function createNewSubjectPackage(e) {
        e.preventDefault();
        const subjectName = document.getElementById('newPkgSubjectName').value.trim();
        if (!subjectName) return;

        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const newPkg = {
            id: 'pkg_' + Date.now(),
            examTitle: settings.examTitle || 'Ujian Berbasis Komputer',
            subjectName,
            scheduleDate: new Date().toISOString().slice(0, 10),
            scheduleTime: '08:00 - 09:30 WITA',
            durationMinutes: 45,
            targetGroup: 'ALL',
            status: 'WAITING',
            shuffleQuestions: true,
            shuffleOptions: true,
            questions: []
        };
        packages.push(newPkg);
        setDB(STORAGE_KEYS.PACKAGES, packages);
        selectedAdminPkgId = newPkg.id;
        selectedProctorPkgId = newPkg.id;
        document.getElementById('newPkgSubjectName').value = '';
        syncAllViews();
        showToast(`Paket Mata Pelajaran "${subjectName}" berhasil dibuat! Silakan isi atau upload soal.`, 'success');
    }

    function handleAddManualQuestion(e, sourceRole = 'admin') {
        e.preventDefault();
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const targetPkgId = sourceRole === 'pengawas' ? selectedProctorPkgId : selectedAdminPkgId;
        const pkgIdx = packages.findIndex(p => p.id === targetPkgId);

        if (pkgIdx === -1) {
            showToast('Pilih atau buat paket mata pelajaran terlebih dahulu!', 'warning');
            return;
        }

        const prefix = sourceRole === 'pengawas' ? 'pQ' : 'mQ';
        const optE = document.getElementById(`${prefix}OptE`).value.trim();
        const options = {
            A: document.getElementById(`${prefix}OptA`).value.trim(),
            B: document.getElementById(`${prefix}OptB`).value.trim(),
            C: document.getElementById(`${prefix}OptC`).value.trim(),
            D: document.getElementById(`${prefix}OptD`).value.trim()
        };
        if (optE) options.E = optE;

        const ans = document.getElementById(`${prefix}Answer`).value;
        if (ans === 'E' && !optE) {
            showToast('Opsi E kosong, silakan isi Opsi E atau pilih kunci A-D.', 'warning');
            return;
        }

        packages[pkgIdx].questions.push({
            id: 'q_' + Date.now(),
            question: document.getElementById(`${prefix}Text`).value.trim(),
            options,
            answer: ans,
            points: parseInt(document.getElementById(`${prefix}Points`).value, 10) || 10
        });

        setDB(STORAGE_KEYS.PACKAGES, packages);
        e.target.reset();
        document.getElementById(`${prefix}Points`).value = 10;
        syncAllViews();
        showToast(`Soal berhasil ditambahkan ke ${packages[pkgIdx].subjectName}!`, 'success');
    }

    function parseExtractedTextToQuestions(rawText) {
        const normalized = rawText.replace(/\r\n/g, '\n').replace(/\r/g, '\n');
        const parsedList = [];

        const lines = normalized.split('\n').map(l => l.trim()).filter(Boolean);
        lines.forEach((line, idx) => {
            if (line.includes('|')) {
                const parts = line.split('|').map(p => p.trim());
                if (parts.length >= 6) {
                    const hasE = parts.length >= 7 && ['A', 'B', 'C', 'D', 'E'].includes(parts[6].toUpperCase());
                    const ans = hasE ? parts[6].toUpperCase() : parts[5].toUpperCase();
                    if (['A', 'B', 'C', 'D', 'E'].includes(ans)) {
                        const opts = { A: parts[1], B: parts[2], C: parts[3], D: parts[4] };
                        if (hasE && parts[5]) opts.E = parts[5];
                        parsedList.push({
                            id: `q_imp_${Date.now()}_${idx}`,
                            question: parts[0],
                            options: opts,
                            answer: ans,
                            points: 10
                        });
                    }
                }
            }
        });

        if (parsedList.length > 0) return parsedList;

        const blocks = normalized.split(/(?=\n?\s*\d+[\.\)]\s+)/g).map(b => b.trim()).filter(Boolean);
        blocks.forEach((block, idx) => {
            const keyMatch = block.match(/(?:Kunci|Jawaban)\s*[:=\-]\s*([A-Ea-e])/i);
            const optAMatch = block.match(/(?:^|\n|\s)A[\.\)]\s*([\s\S]*?)(?=(?:\n|\s)B[\.\)]\s*)/i);
            const optBMatch = block.match(/(?:^|\n|\s)B[\.\)]\s*([\s\S]*?)(?=(?:\n|\s)C[\.\)]\s*)/i);
            const optCMatch = block.match(/(?:^|\n|\s)C[\.\)]\s*([\s\S]*?)(?=(?:\n|\s)D[\.\)]\s*)/i);
            const optDMatch = block.match(/(?:^|\n|\s)D[\.\)]\s*([\s\S]*?)(?=(?:(?:\n|\s)E[\.\)]\s*|(?:Kunci|Jawaban)\s*[:=\-]|$))/i);
            const optEMatch = block.match(/(?:^|\n|\s)E[\.\)]\s*([\s\S]*?)(?=(?:(?:Kunci|Jawaban)\s*[:=\-]|$))/i);

            if (optAMatch && optBMatch && optCMatch && optDMatch) {
                const firstOptIdx = block.search(/(?:^|\n|\s)A[\.\)]\s*/i);
                let qText = block.substring(0, firstOptIdx).replace(/^\d+[\.\)]\s*/, '').trim();
                const opts = {
                    A: optAMatch[1].trim(),
                    B: optBMatch[1].trim(),
                    C: optCMatch[1].trim(),
                    D: optDMatch[1].trim()
                };
                if (optEMatch && optEMatch[1].trim()) {
                    opts.E = optEMatch[1].trim();
                }
                const ans = keyMatch ? keyMatch[1].toUpperCase() : 'A';
                if (qText) {
                    parsedList.push({
                        id: `q_doc_${Date.now()}_${idx}`,
                        question: qText,
                        options: opts,
                        answer: ans,
                        points: 10
                    });
                }
            }
        });

        return parsedList;
    }

    async function handleDocumentQuestionUpload(event, sourceRole = 'admin') {
        const file = event.target.files[0];
        if (!file) return;

        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const targetPkgId = sourceRole === 'pengawas' ? selectedProctorPkgId : selectedAdminPkgId;
        const pkgIdx = packages.findIndex(p => p.id === targetPkgId);

        if (pkgIdx === -1) {
            showToast('Pilih paket mata pelajaran tujuan terlebih dahulu!', 'warning');
            event.target.value = '';
            return;
        }

        const statusBox = document.getElementById('uploadParseStatus');
        if (statusBox && sourceRole === 'admin') {
            statusBox.classList.remove('hidden');
            statusBox.innerText = `Sedang membaca file "${file.name}"...`;
        }

        const ext = file.name.split('.').pop().toLowerCase();
        let extractedText = '';

        try {
            if (ext === 'docx') {
                if (typeof mammoth === 'undefined') {
                    throw new Error('Library Mammoth.js tidak tersedia (mode offline). Gunakan file .TXT atau input manual.');
                }
                const arrayBuffer = await file.arrayBuffer();
                const result = await mammoth.extractRawText({ arrayBuffer });
                extractedText = result.value;
            } else if (ext === 'pdf') {
                if (typeof pdfjsLib === 'undefined') {
                    throw new Error('Library PDF.js tidak tersedia (mode offline). Gunakan file .TXT atau input manual.');
                }
                const arrayBuffer = await file.arrayBuffer();
                const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
                for (let p = 1; p <= pdf.numPages; p++) {
                    const page = await pdf.getPage(p);
                    const content = await page.getTextContent();
                    const pageText = content.items.map(item => item.str).join(' ');
                    extractedText += '\n' + pageText;
                }
            } else {
                extractedText = await file.text();
            }

            const newQuestions = parseExtractedTextToQuestions(extractedText);
            if (newQuestions.length > 0) {
                packages[pkgIdx].questions = [...(packages[pkgIdx].questions || []), ...newQuestions];
                setDB(STORAGE_KEYS.PACKAGES, packages);
                syncAllViews();
                showToast(`Berhasil mengimpor ${newQuestions.length} soal ke "${packages[pkgIdx].subjectName}"!`, 'success');
                if (statusBox && sourceRole === 'admin') {
                    statusBox.innerText = `Sukses: ${newQuestions.length} soal ditambahkan ke ${packages[pkgIdx].subjectName}.`;
                }
            } else {
                showToast('Tidak ditemukan pola soal yang valid dalam file. Unduh contoh format untuk panduan.', 'warning');
                if (statusBox && sourceRole === 'admin') {
                    statusBox.innerText = 'Format soal tidak dikenali. Pastikan menggunakan penomoran 1., opsi A-D, dan Kunci: A.';
                }
            }
        } catch (err) {
            showToast(err.message || 'Gagal memproses file dokumen.', 'error');
            if (statusBox && sourceRole === 'admin') statusBox.innerText = `Error: ${err.message}`;
        } finally {
            event.target.value = '';
        }
    }

    function downloadSampleQuestionFormat() {
        const sample = [
            "1. Ibu kota negara Indonesia adalah...",
            "A. Bandung",
            "B. Jakarta",
            "C. Surabaya",
            "D. Medan",
            "E. Makassar",
            "Kunci: B",
            "",
            "2. Hasil dari 25 x 4 adalah...",
            "A. 80",
            "B. 90",
            "C. 100",
            "D. 110",
            "Kunci: C"
        ].join('\n');

        const blob = new Blob([sample], { type: 'text/plain;charset=utf-8' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'Contoh_Format_Soal_CBT.txt';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('Contoh format soal berhasil diunduh!', 'success');
    }

    function renderAdminQuestionList() {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkg = packages.find(p => p.id === selectedAdminPkgId);
        const questions = pkg ? (pkg.questions || []) : [];
        const container = document.getElementById('adminSoalListContainer');
        document.getElementById('adminBankSoalCount').innerText = questions.length;

        if (questions.length === 0) {
            container.innerHTML = `<div class="text-center py-12 text-xs text-slate-400">Belum ada soal pada paket mata pelajaran ini.</div>`;
            return;
        }

        container.innerHTML = questions.map((q, idx) => {
            const optionKeys = Object.keys(q.options);
            return `
                <div class="p-4 rounded-2xl border border-slate-200 bg-slate-50/60 hover:bg-white transition space-y-2.5">
                    <div class="flex items-start justify-between gap-3">
                        <div class="text-xs sm:text-sm font-bold text-slate-900 break-words flex-1">
                            <span class="text-indigo-600 mr-1">${idx + 1}.</span> ${q.question}
                            <span class="ml-2 px-2 py-0.5 rounded bg-slate-200 text-slate-700 text-[10px] font-semibold inline-block">${q.points || 10} Poin</span>
                        </div>
                        <div class="flex items-center gap-1.5 shrink-0">
                            <button onclick="openEditQuestionModal('${pkg.id}', '${q.id}')" class="px-2.5 py-1.5 rounded-lg bg-amber-100 hover:bg-amber-200 text-amber-800 text-xs font-bold transition flex items-center gap-1">
                                <i data-lucide="edit-3" class="w-3.5 h-3.5"></i> Edit
                            </button>
                            <button onclick="deleteSingleQuestion('${pkg.id}', '${q.id}')" class="p-1.5 rounded-lg bg-red-50 hover:bg-red-100 text-red-600 transition" title="Hapus Soal">
                                <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
                            </button>
                        </div>
                    </div>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-1.5 text-xs">
                        ${optionKeys.map(k => `
                            <div class="px-2.5 py-1.5 rounded-lg border break-words ${q.answer === k ? 'bg-emerald-100 border-emerald-300 text-emerald-900 font-bold' : 'bg-white border-slate-200 text-slate-600'}">
                                <strong>${k}.</strong> ${q.options[k]}
                            </div>
                        `).join('')}
                    </div>
                </div>
            `;
        }).join('');
    }

    let editingPkgId = null;

    function openEditQuestionModal(pkgId, qId) {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkg = packages.find(p => p.id === pkgId);
        if (!pkg) return;
        const q = (pkg.questions || []).find(item => item.id === qId);
        if (!q) return;

        editingPkgId = pkgId;
        document.getElementById('editQId').value = q.id;
        document.getElementById('editQText').value = q.question;
        document.getElementById('editQOptA').value = q.options.A || '';
        document.getElementById('editQOptB').value = q.options.B || '';
        document.getElementById('editQOptC').value = q.options.C || '';
        document.getElementById('editQOptD').value = q.options.D || '';
        document.getElementById('editQOptE').value = q.options.E || '';
        document.getElementById('editQAnswer').value = q.answer;
        document.getElementById('editQPoints').value = q.points || 10;

        const modal = document.getElementById('editQuestionModal');
        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function closeEditQuestionModal() {
        const modal = document.getElementById('editQuestionModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
    }

    function saveEditedQuestion(e) {
        e.preventDefault();
        const qId = document.getElementById('editQId').value;
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkgIdx = packages.findIndex(p => p.id === editingPkgId);
        if (pkgIdx === -1) return;

        const idx = packages[pkgIdx].questions.findIndex(item => item.id === qId);
        if (idx === -1) return;

        const optE = document.getElementById('editQOptE').value.trim();
        const options = {
            A: document.getElementById('editQOptA').value.trim(),
            B: document.getElementById('editQOptB').value.trim(),
            C: document.getElementById('editQOptC').value.trim(),
            D: document.getElementById('editQOptD').value.trim()
        };
        if (optE) options.E = optE;

        const ans = document.getElementById('editQAnswer').value;
        if (ans === 'E' && !optE) {
            showToast('Opsi E kosong, pilih kunci A-D atau isi teks Opsi E.', 'warning');
            return;
        }

        packages[pkgIdx].questions[idx] = {
            ...packages[pkgIdx].questions[idx],
            question: document.getElementById('editQText').value.trim(),
            options,
            answer: ans,
            points: parseInt(document.getElementById('editQPoints').value, 10) || 10
        };

        setDB(STORAGE_KEYS.PACKAGES, packages);
        closeEditQuestionModal();
        syncAllViews();
        showToast('Perubahan soal berhasil disimpan!', 'success');
    }

    function deleteSingleQuestion(pkgId, qId) {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const pkgIdx = packages.findIndex(p => p.id === pkgId);
        if (pkgIdx === -1) return;
        packages[pkgIdx].questions = packages[pkgIdx].questions.filter(q => q.id !== qId);
        setDB(STORAGE_KEYS.PACKAGES, packages);
        syncAllViews();
        showToast('Soal dihapus dari paket.', 'info');
    }

    function confirmClearPackageQuestions() {
        showConfirmDialog('Kosongkan Soal Paket Ini?', 'Seluruh soal dalam paket mata pelajaran terpilih akan dihapus permanen.', () => {
            const packages = getDB(STORAGE_KEYS.PACKAGES, []);
            const pkgIdx = packages.findIndex(p => p.id === selectedAdminPkgId);
            if (pkgIdx !== -1) {
                packages[pkgIdx].questions = [];
                setDB(STORAGE_KEYS.PACKAGES, packages);
                syncAllViews();
                showToast('Soal dalam paket berhasil dikosongkan.', 'info');
            }
        });
    }

    function generateRandomToken() {
        const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
        let token = '';
        for (let i = 0; i < 6; i++) token += chars.charAt(Math.floor(Math.random() * chars.length));
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        settings.activeToken = token;
        setDB(STORAGE_KEYS.SETTINGS, settings);
        syncAllViews();
        showToast(`Token baru berhasil dibuat & disinkronkan: ${token}`, 'success');
    }

    function saveManualToken() {
        const val = document.getElementById('customTokenInput').value.trim().toUpperCase();
        if (val.length < 4) {
            showToast('Token minimal terdiri dari 4 karakter!', 'warning');
            return;
        }
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        settings.activeToken = val;
        setDB(STORAGE_KEYS.SETTINGS, settings);
        document.getElementById('customTokenInput').value = '';
        syncAllViews();
        showToast(`Token berhasil diubah & disinkronkan: ${val}`, 'success');
    }

    function loadSelectedPackageToAssignForm() {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const sel = document.getElementById('assignPackageSelect');
        if (!sel) return;
        const pkg = packages.find(p => p.id === sel.value) || packages[0];
        if (!pkg) return;

        document.getElementById('assignExamTitle').value = pkg.examTitle || '';
        document.getElementById('assignSubject').value = pkg.subjectName || '';
        document.getElementById('assignScheduleDate').value = pkg.scheduleDate || new Date().toISOString().slice(0, 10);
        document.getElementById('assignScheduleTime').value = pkg.scheduleTime || '08:00 - 09:30 WITA';
        document.getElementById('assignDuration').value = pkg.durationMinutes || 45;
        document.getElementById('assignTargetGroup').value = pkg.targetGroup || 'ALL';
        document.getElementById('assignStatusSelect').value = pkg.status || 'ASSIGNED';
        document.getElementById('assignShuffleQuestions').checked = !!pkg.shuffleQuestions;
        document.getElementById('assignShuffleOptions').checked = !!pkg.shuffleOptions;
    }

    function savePackageAssignment(e) {
        e.preventDefault();
        const pkgId = document.getElementById('assignPackageSelect').value;
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const idx = packages.findIndex(p => p.id === pkgId);
        if (idx === -1) {
            showToast('Pilih paket soal terlebih dahulu!', 'warning');
            return;
        }

        const newStatus = document.getElementById('assignStatusSelect').value;
        if (newStatus === 'ASSIGNED' && (!packages[idx].questions || packages[idx].questions.length === 0)) {
            showToast('Paket soal masih kosong! Tambahkan soal terlebih dahulu sebelum di-assign ke siswa.', 'warning');
            return;
        }

        packages[idx] = {
            ...packages[idx],
            examTitle: document.getElementById('assignExamTitle').value.trim(),
            subjectName: document.getElementById('assignSubject').value.trim(),
            scheduleDate: document.getElementById('assignScheduleDate').value,
            scheduleTime: document.getElementById('assignScheduleTime').value.trim(),
            durationMinutes: parseInt(document.getElementById('assignDuration').value, 10) || 45,
            targetGroup: document.getElementById('assignTargetGroup').value,
            status: newStatus,
            shuffleQuestions: document.getElementById('assignShuffleQuestions').checked,
            shuffleOptions: document.getElementById('assignShuffleOptions').checked
        };

        setDB(STORAGE_KEYS.PACKAGES, packages);
        syncAllViews();
        showToast(`Penugasan "${packages[idx].subjectName}" berhasil disimpan & disinkronkan ke Dashboard Siswa!`, 'success');
    }

    function togglePackageAssignStatus(pkgId) {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const idx = packages.findIndex(p => p.id === pkgId);
        if (idx === -1) return;

        const nextStatus = packages[idx].status === 'ASSIGNED' ? 'WAITING' : 'ASSIGNED';
        if (nextStatus === 'ASSIGNED' && (!packages[idx].questions || packages[idx].questions.length === 0)) {
            showToast('Tidak dapat meng-assign karena paket soal ini belum memiliki butir soal.', 'warning');
            return;
        }

        packages[idx].status = nextStatus;
        setDB(STORAGE_KEYS.PACKAGES, packages);
        syncAllViews();
        showToast(
            nextStatus === 'ASSIGNED'
                ? `Mata pelajaran "${packages[idx].subjectName}" kini DI-ASSIGN & siap dikerjakan siswa!`
                : `Mata pelajaran "${packages[idx].subjectName}" diubah ke status Menunggu Assign.`,
            'info'
        );
    }

    function renderAdminAssignmentsTable() {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const sel = document.getElementById('assignPackageSelect');
        if (sel) {
            const prevVal = sel.value;
            sel.innerHTML = packages.map(p => `
                <option value="${p.id}">${p.subjectName} (${(p.questions || []).length} Soal)</option>
            `).join('');
            if (prevVal && packages.some(p => p.id === prevVal)) {
                sel.value = prevVal;
            } else if (packages.length > 0) {
                sel.value = packages[0].id;
                loadSelectedPackageToAssignForm();
            }
        }

        const tbody = document.getElementById('adminAssignmentsTableBody');
        if (!tbody) return;

        if (packages.length === 0) {
            tbody.innerHTML = `<tr><td colspan="6" class="py-8 text-center text-slate-400 text-xs">Belum ada paket soal mata pelajaran. Buat paket terlebih dahulu di Menu 2.</td></tr>`;
            return;
        }

        tbody.innerHTML = packages.map(p => {
            const isAssigned = p.status === 'ASSIGNED';
            return `
                <tr class="hover:bg-slate-50 transition">
                    <td class="py-3 px-4">
                        <div class="font-bold text-slate-900">${p.subjectName}</div>
                        <div class="text-[11px] text-slate-500">${p.examTitle}</div>
                    </td>
                    <td class="py-3 px-4 text-xs">
                        <div class="font-semibold text-slate-800">${p.scheduleDate || '-'}</div>
                        <div class="text-slate-500">${p.scheduleTime || '-'} (${p.durationMinutes} mnt)</div>
                    </td>
                    <td class="py-3 px-4 text-xs font-bold text-indigo-700">${p.targetGroup === 'ALL' ? 'Semua Kelas' : p.targetGroup}</td>
                    <td class="py-3 px-4 font-bold text-xs">${(p.questions || []).length} Soal</td>
                    <td class="py-3 px-4">
                        ${isAssigned
                            ? `<span class="px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-700 text-[11px] font-bold">DI-ASSIGN (Aktif)</span>`
                            : `<span class="px-2.5 py-1 rounded-full bg-amber-100 text-amber-800 text-[11px] font-bold">Menunggu Assign</span>`
                        }
                    </td>
                    <td class="py-3 px-4 text-right space-x-1 whitespace-nowrap">
                        <button onclick="togglePackageAssignStatus('${p.id}')" class="px-3 py-1.5 rounded-lg ${isAssigned ? 'bg-amber-100 hover:bg-amber-200 text-amber-900' : 'bg-blue-600 hover:bg-blue-700 text-white'} text-[11px] font-bold transition">
                            ${isAssigned ? 'Tunda / Set Menunggu' : 'Assign Sekarang'}
                        </button>
                        <button onclick="document.getElementById('assignPackageSelect').value='${p.id}'; loadSelectedPackageToAssignForm();" class="px-2.5 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-700 text-[11px] font-bold transition">
                            Edit Jadwal
                        </button>
                    </td>
                </tr>
            `;
        }).join('');
    }

    function renderAdminLogsTable() {
        const logs = getDB(STORAGE_KEYS.LOGS, []);
        const search = (document.getElementById('logSearchInput')?.value || '').toLowerCase();
        const roleFilter = document.getElementById('logRoleFilter')?.value || '';
        const typeFilter = document.getElementById('logTypeFilter')?.value || '';

        const filtered = logs.filter(item => {
            const matchText =
                (item.userName || '').toLowerCase().includes(search) ||
                (item.userIdentifier || '').toLowerCase().includes(search) ||
                (item.detail || '').toLowerCase().includes(search);
            const matchRole = !roleFilter || item.role === roleFilter;
            const matchType = !typeFilter || item.category === typeFilter;
            return matchText && matchRole && matchType;
        });

        const tbody = document.getElementById('adminLogsTableBody');
        if (!tbody) return;

        if (filtered.length === 0) {
            tbody.innerHTML = `<tr><td colspan="5" class="py-8 text-center text-slate-400 text-xs">Belum ada catatan aktivitas masuk/keluar yang sesuai filter.</td></tr>`;
            return;
        }

        tbody.innerHTML = filtered.map(item => {
            let roleBadge = `<span class="px-2 py-0.5 rounded bg-slate-100 text-slate-700 text-[10px] font-bold uppercase">${item.role}</span>`;
            if (item.role === 'siswa') roleBadge = `<span class="px-2 py-0.5 rounded bg-blue-100 text-blue-700 text-[10px] font-bold uppercase">Siswa</span>`;
            else if (item.role === 'pengawas') roleBadge = `<span class="px-2 py-0.5 rounded bg-amber-100 text-amber-800 text-[10px] font-bold uppercase">Pengawas</span>`;
            else if (item.role === 'admin') roleBadge = `<span class="px-2 py-0.5 rounded bg-indigo-100 text-indigo-700 text-[10px] font-bold uppercase">Admin</span>`;

            let catBadge = `<span class="px-2 py-0.5 rounded-full bg-slate-100 text-slate-700 text-[10px] font-bold">${item.category}</span>`;
            if (item.category === 'LOGIN') catBadge = `<span class="px-2 py-0.5 rounded-full bg-emerald-100 text-emerald-700 text-[10px] font-bold">MASUK (LOGIN)</span>`;
            else if (item.category === 'LOGOUT') catBadge = `<span class="px-2 py-0.5 rounded-full bg-slate-200 text-slate-700 text-[10px] font-bold">KELUAR (LOGOUT)</span>`;
            else if (item.category === 'EXAM_START') catBadge = `<span class="px-2 py-0.5 rounded-full bg-blue-100 text-blue-700 text-[10px] font-bold">MULAI UJIAN</span>`;
            else if (item.category === 'EXAM_FINISH') catBadge = `<span class="px-2 py-0.5 rounded-full bg-indigo-100 text-indigo-700 text-[10px] font-bold">SELESAI UJIAN</span>`;
            else if (item.category === 'VIOLATION') catBadge = `<span class="px-2 py-0.5 rounded-full bg-red-600 text-white text-[10px] font-bold">PELANGGARAN + SIRINE</span>`;
            else if (item.category === 'UNLOCK') catBadge = `<span class="px-2 py-0.5 rounded-full bg-amber-100 text-amber-800 text-[10px] font-bold">BUKA KUNCI</span>`;

            return `
                <tr class="hover:bg-slate-50 transition ${item.category === 'VIOLATION' ? 'bg-red-50/50' : ''}">
                    <td class="py-2.5 px-4 font-mono text-xs text-slate-600 whitespace-nowrap">${item.timestamp}</td>
                    <td class="py-2.5 px-4">${roleBadge}</td>
                    <td class="py-2.5 px-4">
                        <div class="font-bold text-slate-900">${item.userName}</div>
                        <div class="text-[11px] font-mono text-slate-500">${item.userIdentifier} • ${item.roomOrClass}</div>
                    </td>
                    <td class="py-2.5 px-4 whitespace-nowrap">${catBadge}</td>
                    <td class="py-2.5 px-4 text-xs text-slate-700">${item.detail}</td>
                </tr>
            `;
        }).join('');
    }

    function exportActivityLogsExcel() {
        const logs = getDB(STORAGE_KEYS.LOGS, []);
        if (logs.length === 0) {
            showToast('Belum ada data log aktivitas untuk diunduh.', 'warning');
            return;
        }
        const rows = logs.map((l, idx) => ({
            'No': idx + 1,
            'Waktu': l.timestamp,
            'Peran': l.role.toUpperCase(),
            'Nama Pengguna': l.userName,
            'ID / NISN': l.userIdentifier,
            'Kelas / Ruangan': l.roomOrClass,
            'Kategori': l.category,
            'Keterangan': l.detail
        }));

        const fileName = `Log_Aktivitas_CBT_${new Date().toISOString().slice(0, 10)}`;
        if (typeof XLSX !== 'undefined') {
            const ws = XLSX.utils.json_to_sheet(rows);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Log Masuk Keluar');
            XLSX.writeFile(wb, `${fileName}.xlsx`);
            showToast('Log aktivitas masuk/keluar (.XLSX) berhasil diunduh!', 'success');
        }
    }

    function confirmClearActivityLogs() {
        showConfirmDialog('Bersihkan Seluruh Log Aktivitas?', 'Semua riwayat log masuk/keluar dan pelanggaran akan dihapus permanen.', () => {
            setDB(STORAGE_KEYS.LOGS, []);
            renderAdminLogsTable();
            showToast('Log aktivitas berhasil dibersihkan.', 'info');
        });
    }

    function renderAdminNilaiTable() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const search = (document.getElementById('nilaiSearchInput')?.value || '').toLowerCase();
        const classFilter = document.getElementById('nilaiClassFilter')?.value || '';
        const roomFilter = document.getElementById('nilaiRoomFilter')?.value || '';

        const filtered = students.filter(s => {
            const matchText = s.name.toLowerCase().includes(search) || s.nisn.includes(search);
            const matchClass = !classFilter || s.className === classFilter;
            const matchRoom = !roomFilter || s.room === roomFilter;
            return matchText && matchClass && matchRoom;
        });

        const tbody = document.getElementById('adminNilaiTbody');
        if (filtered.length === 0) {
            tbody.innerHTML = `<tr><td colspan="10" class="py-8 text-center text-slate-400 text-xs">Tidak ada data peserta yang ditemukan.</td></tr>`;
            return;
        }

        tbody.innerHTML = filtered.map((s, idx) => {
            const st = liveMap[s.id] || { status: 'Belum Ujian', currentSubject: '-', correct: '-', wrong: '-', score: '-', violations: 0 };
            return `
                <tr class="hover:bg-slate-50">
                    <td class="py-3 px-3 text-slate-500">${idx + 1}</td>
                    <td class="py-3 px-3 font-bold text-slate-900">${s.name}</td>
                    <td class="py-3 px-3 font-mono text-xs">${s.nisn}</td>
                    <td class="py-3 px-3 text-xs">${s.className} / ${s.room}</td>
                    <td class="py-3 px-3 text-xs font-semibold text-slate-700">${st.currentSubject || '-'}</td>
                    <td class="py-3 px-3">
                        <span class="px-2 py-0.5 rounded-full text-[11px] font-bold ${st.status === 'Selesai' ? 'bg-emerald-100 text-emerald-700' : 'bg-slate-100 text-slate-600'}">
                            ${st.status}
                        </span>
                    </td>
                    <td class="py-3 px-3 font-bold text-emerald-600">${st.correct ?? '-'}</td>
                    <td class="py-3 px-3 font-bold text-red-500">${st.wrong ?? '-'}</td>
                    <td class="py-3 px-3 font-extrabold text-indigo-700 text-sm">${st.score ?? '-'}</td>
                    <td class="py-3 px-3 text-xs font-bold ${st.violations > 0 ? 'text-red-600' : 'text-slate-400'}">${st.violations || 0}x</td>
                </tr>
            `;
        }).join('');
    }

    function downloadExcelResults() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});

        const rows = students.map((s, idx) => {
            const st = liveMap[s.id] || {};
            return {
                'No': idx + 1,
                'Nama Lengkap': s.name,
                'NISN': s.nisn,
                'Kelas': s.className,
                'Ruangan': s.room,
                'Mata Pelajaran': st.currentSubject || '-',
                'Status Ujian': st.status || 'Belum Ujian',
                'Jumlah Benar': st.correct ?? 0,
                'Jumlah Salah': st.wrong ?? 0,
                'Nilai Akhir': st.score ?? 0,
                'Pelanggaran (Kali)': st.violations ?? 0,
                'Waktu Selesai': st.finishedAt || '-'
            };
        });

        const fileName = `Rekap_Nilai_CBT_${new Date().toISOString().slice(0, 10)}`;

        if (typeof XLSX !== 'undefined') {
            const ws = XLSX.utils.json_to_sheet(rows);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Hasil Ujian');
            XLSX.writeFile(wb, `${fileName}.xlsx`);
            showToast('File Rekap Nilai (.XLSX) berhasil diunduh!', 'success');
        }
    }

    function generateRandomStudentPassword() {
        return String(Math.floor(10000 + Math.random() * 90000));
    }

    function toggleShowStudentPasswords() {
        showStudentPasswords = !showStudentPasswords;
        const btn = document.getElementById('btnTogglePassVis');
        btn.innerHTML = showStudentPasswords
            ? `<i data-lucide="eye-off" class="w-3.5 h-3.5"></i> <span>Sembunyikan Password</span>`
            : `<i data-lucide="eye" class="w-3.5 h-3.5"></i> <span>Tampilkan Password</span>`;
        renderAdminStudentAccountTable();
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function handleAddManualStudent(e) {
        e.preventDefault();
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const name = document.getElementById('kStuName').value.trim();
        const nisn = document.getElementById('kStuNisn').value.trim();
        const className = document.getElementById('kStuClass').value.trim();
        const room = document.getElementById('kStuRoom').value.trim();
        const customUser = document.getElementById('kStuUser').value.trim();
        const customPass = document.getElementById('kStuPass').value.trim();

        if (students.some(s => s.nisn === nisn)) {
            showToast('NISN tersebut sudah terdaftar dalam sistem!', 'warning');
            return;
        }

        const username = customUser || nisn;
        const password = customPass || generateRandomStudentPassword();

        students.push({
            id: 'stu_' + Date.now(),
            name,
            nisn,
            className,
            room,
            username,
            password
        });

        setDB(STORAGE_KEYS.STUDENTS, students);
        e.target.reset();
        syncAllViews();
        showToast(`Akun siswa ${name} (${username}) siap digunakan!`, 'success');
    }

    function downloadStudentExcelTemplate() {
        const templateData = [
            { Nama: 'Contoh Siswa 1', NISN: '0089910001', Kelas: 'XII IPA 1', Ruangan: 'Ruang 01' },
            { Nama: 'Contoh Siswa 2', NISN: '0089910002', Kelas: 'XII IPS 1', Ruangan: 'Ruang 02' }
        ];
        if (typeof XLSX !== 'undefined') {
            const ws = XLSX.utils.json_to_sheet(templateData);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Data_Siswa');
            XLSX.writeFile(wb, 'Template_Import_Siswa_CBT.xlsx');
            showToast('Template Excel siswa berhasil diunduh!', 'success');
        }
    }

    function handleStudentExcelUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        if (typeof XLSX === 'undefined') {
            showToast('Library SheetJS tidak tersedia saat offline.', 'error');
            return;
        }

        const reader = new FileReader();
        reader.onload = (e) => {
            try {
                const data = new Uint8Array(e.target.result);
                const wb = XLSX.read(data, { type: 'array' });
                const sheet = wb.Sheets[wb.SheetNames[0]];
                const jsonRows = XLSX.utils.sheet_to_json(sheet);

                const existingStudents = getDB(STORAGE_KEYS.STUDENTS, []);
                let added = 0;

                jsonRows.forEach((row, idx) => {
                    const name = String(row.Nama || row.nama || row['Nama Lengkap'] || '').trim();
                    const nisn = String(row.NISN || row.nisn || '').trim();
                    const className = String(row.Kelas || row.kelas || 'Umum').trim();
                    const room = String(row.Ruangan || row.ruangan || row.Ruang || 'Ruang 01').trim();

                    if (name && nisn && !existingStudents.some(s => s.nisn === nisn)) {
                        existingStudents.push({
                            id: `stu_imp_${Date.now()}_${idx}`,
                            name,
                            nisn,
                            className,
                            room,
                            username: nisn,
                            password: generateRandomStudentPassword()
                        });
                        added++;
                    }
                });

                setDB(STORAGE_KEYS.STUDENTS, existingStudents);
                syncAllViews();
                showToast(`Berhasil mengimpor ${added} akun siswa siap digunakan!`, 'success');
            } catch (err) {
                showToast('Gagal membaca file Excel. Pastikan format kolom sesuai template.', 'error');
            } finally {
                event.target.value = '';
            }
        };
        reader.readAsArrayBuffer(file);
    }

    function renderAdminStudentAccountTable() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const search = (document.getElementById('accountSearchInput')?.value || '').toLowerCase();
        document.getElementById('studentAccountCount').innerText = students.length;

        const filtered = students.filter(s =>
            s.name.toLowerCase().includes(search) ||
            s.nisn.toLowerCase().includes(search) ||
            s.className.toLowerCase().includes(search) ||
            s.room.toLowerCase().includes(search)
        );

        const tbody = document.getElementById('adminStudentAccountTbody');
        if (!tbody) return;

        if (filtered.length === 0) {
            tbody.innerHTML = `<tr><td colspan="5" class="py-8 text-center text-slate-400 text-xs">Belum ada akun siswa terdaftar.</td></tr>`;
            return;
        }

        tbody.innerHTML = filtered.map(s => `
            <tr class="hover:bg-slate-50 transition">
                <td class="py-2.5 px-3">
                    <div class="font-bold text-slate-900">${s.name}</div>
                    <div class="text-[11px] font-mono text-slate-500">NISN: ${s.nisn}</div>
                </td>
                <td class="py-2.5 px-3 text-slate-700">${s.className} • <strong>${s.room}</strong></td>
                <td class="py-2.5 px-3 font-mono font-bold text-slate-800">${s.username}</td>
                <td class="py-2.5 px-3 font-mono font-bold text-blue-700">
                    ${showStudentPasswords ? s.password : '•••••'}
                </td>
                <td class="py-2.5 px-3 text-right space-x-1 whitespace-nowrap">
                    <button onclick="copyTextToClipboard('Username: ${s.username} | Password: ${s.password}', 'Akun ${s.name} disalin!')" class="px-2 py-1 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-700 text-[11px] font-bold" title="Salin Akun">
                        Salin
                    </button>
                    <button onclick="openEditStudentModal('${s.id}')" class="px-2 py-1 rounded-lg bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-[11px] font-bold">
                        Edit / Reset
                    </button>
                    <button onclick="deleteSingleStudent('${s.id}')" class="px-2 py-1 rounded-lg bg-red-50 hover:bg-red-100 text-red-600 text-[11px] font-bold">
                        Hapus
                    </button>
                </td>
            </tr>
        `).join('');
    }

    function openEditStudentModal(stuId) {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const stu = students.find(s => s.id === stuId);
        if (!stu) return;

        document.getElementById('editStuId').value = stu.id;
        document.getElementById('editStuName').value = stu.name;
        document.getElementById('editStuNisn').value = stu.nisn;
        document.getElementById('editStuClass').value = stu.className;
        document.getElementById('editStuRoom').value = stu.room;
        document.getElementById('editStuUser').value = stu.username;
        document.getElementById('editStuPass').value = stu.password;

        const modal = document.getElementById('editStudentModal');
        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function closeEditStudentModal() {
        const modal = document.getElementById('editStudentModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
    }

    function generateRandomStudentPassInModal() {
        document.getElementById('editStuPass').value = generateRandomStudentPassword();
    }

    function saveEditedStudent(e) {
        e.preventDefault();
        const id = document.getElementById('editStuId').value;
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const idx = students.findIndex(s => s.id === id);
        if (idx === -1) return;

        students[idx] = {
            ...students[idx],
            name: document.getElementById('editStuName').value.trim(),
            nisn: document.getElementById('editStuNisn').value.trim(),
            className: document.getElementById('editStuClass').value.trim(),
            room: document.getElementById('editStuRoom').value.trim(),
            username: document.getElementById('editStuUser').value.trim(),
            password: document.getElementById('editStuPass').value.trim()
        };

        setDB(STORAGE_KEYS.STUDENTS, students);
        closeEditStudentModal();
        syncAllViews();
        showToast(`Data & password akun ${students[idx].name} berhasil diperbarui!`, 'success');
    }

    function deleteSingleStudent(stuId) {
        showConfirmDialog('Hapus Akun Siswa?', 'Akun dan kartu login peserta ini akan dihapus dari sistem.', () => {
            const students = getDB(STORAGE_KEYS.STUDENTS, []).filter(s => s.id !== stuId);
            setDB(STORAGE_KEYS.STUDENTS, students);
            syncAllViews();
            showToast('Akun siswa berhasil dihapus.', 'info');
        });
    }

    function confirmClearAllStudents() {
        showConfirmDialog('Kosongkan Seluruh Akun Siswa?', 'Semua data akun dan kartu login peserta akan dihapus permanen.', () => {
            setDB(STORAGE_KEYS.STUDENTS, []);
            syncAllViews();
            showToast('Daftar akun siswa telah dikosongkan.', 'info');
        });
    }

    function exportStudentAccountsExcel() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        if (students.length === 0) {
            showToast('Belum ada data akun siswa untuk diunduh.', 'warning');
            return;
        }
        const rows = students.map((s, idx) => ({
            'No': idx + 1,
            'Nama Lengkap': s.name,
            'NISN': s.nisn,
            'Kelas': s.className,
            'Ruangan': s.room,
            'Username': s.username,
            'Password': s.password
        }));

        if (typeof XLSX !== 'undefined') {
            const ws = XLSX.utils.json_to_sheet(rows);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Akun_Siswa');
            XLSX.writeFile(wb, `Daftar_Akun_Login_Siswa_${new Date().toISOString().slice(0, 10)}.xlsx`);
            showToast('Daftar akun login siswa (.XLSX) berhasil diunduh!', 'success');
        }
    }

    function renderAdminLoginCards() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const container = document.getElementById('printCardsContainer');
        document.getElementById('cardTotalCount').innerText = students.length;

        if (students.length === 0) {
            container.innerHTML = `<div class="col-span-2 text-center py-8 text-xs text-slate-400">Belum ada kartu peserta. Tambahkan atau import data siswa di atas.</div>`;
            return;
        }

        container.innerHTML = students.map(s => `
            <div class="login-card-item border-2 border-slate-300 rounded-2xl p-4 bg-white shadow-sm flex flex-col justify-between space-y-3">
                <div class="flex items-center justify-between border-b border-slate-200 pb-2.5 gap-2">
                    <div class="flex items-center gap-2 min-w-0">
                        <div class="w-8 h-8 rounded-lg bg-slate-900 text-white flex items-center justify-center font-extrabold text-xs shrink-0">CBT</div>
                        <div class="min-w-0">
                            <div class="text-[11px] font-extrabold text-slate-900 uppercase leading-tight truncate">${settings.schoolName}</div>
                            <div class="text-[10px] text-slate-500">KARTU LOGIN PESERTA UJIAN</div>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-2 gap-2 text-xs">
                    <div class="col-span-2">
                        <span class="text-[10px] text-slate-400 block">Nama Peserta</span>
                        <span class="font-extrabold text-slate-900 text-sm">${s.name}</span>
                    </div>
                    <div>
                        <span class="text-[10px] text-slate-400 block">NISN</span>
                        <span class="font-mono font-bold text-slate-800">${s.nisn}</span>
                    </div>
                    <div>
                        <span class="text-[10px] text-slate-400 block">Kelas / Ruangan</span>
                        <span class="font-bold text-indigo-700">${s.className} • ${s.room}</span>
                    </div>
                    <div class="bg-slate-50 p-2 rounded-lg border border-slate-200">
                        <span class="text-[10px] text-slate-400 block">Username</span>
                        <span class="font-mono font-extrabold text-slate-900">${s.username}</span>
                    </div>
                    <div class="bg-blue-50 p-2 rounded-lg border border-blue-200">
                        <span class="text-[10px] text-blue-600 block">Password</span>
                        <span class="font-mono font-extrabold text-blue-900">${s.password}</span>
                    </div>
                </div>

                <div class="flex items-center justify-between pt-2 border-t border-slate-100 text-[10px] text-slate-500">
                    <span>Sesi: ${settings.examTitle}</span>
                </div>
            </div>
        `).join('');
    }

    function renderAdminSettingsForms() {
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const proctors = getDB(STORAGE_KEYS.PROCTORS, DEFAULT_PROCTORS);

        document.getElementById('setSchoolName').value = settings.schoolName || '';
        document.getElementById('setSchoolSubtitle').value = settings.schoolSubtitle || '';
        document.getElementById('setAdminUsername').value = settings.adminUsername || 'admin';
        document.getElementById('setAdminPassword').value = settings.adminPassword || 'admin123';
        document.getElementById('setUnlockPin').value = settings.unlockPin || '123';
        document.getElementById('setRequireFullscreen').checked = !!settings.requireFullscreen;
        document.getElementById('setDetectTabSwitch').checked = !!settings.detectTabSwitch;
        document.getElementById('setEnableSiren').checked = settings.enableSiren !== false;
        document.getElementById('setShowScoreToStudent').checked = !!settings.showScoreToStudent;

        const procContainer = document.getElementById('proctorListContainer');
        procContainer.innerHTML = proctors.map(p => `
            <div class="flex items-center justify-between p-2.5 rounded-xl bg-slate-50 border border-slate-200 text-xs gap-2">
                <div class="min-w-0">
                    <div class="font-bold text-slate-900 truncate">${p.name} <span class="text-amber-700">(${p.room})</span></div>
                    <div class="text-[11px] font-mono text-slate-500">User: ${p.username} • Pass: ${p.password}</div>
                </div>
                <button onclick="deleteProctor('${p.id}')" class="px-2.5 py-1 rounded-lg bg-red-50 hover:bg-red-100 text-red-600 font-bold shrink-0">Hapus</button>
            </div>
        `).join('');
    }

    function saveSchoolIdentity() {
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        settings.schoolName = document.getElementById('setSchoolName').value.trim() || DEFAULT_SETTINGS.schoolName;
        settings.schoolSubtitle = document.getElementById('setSchoolSubtitle').value.trim() || DEFAULT_SETTINGS.schoolSubtitle;
        settings.adminUsername = document.getElementById('setAdminUsername').value.trim() || 'admin';
        settings.adminPassword = document.getElementById('setAdminPassword').value.trim() || 'admin123';
        setDB(STORAGE_KEYS.SETTINGS, settings);
        syncAllViews();
        showToast('Identitas sekolah & kredensial administrator berhasil disimpan!', 'success');
    }

    function saveSecuritySettings() {
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const pin = document.getElementById('setUnlockPin').value.trim();
        if (!pin) {
            showToast('PIN pembuka kunci tidak boleh kosong!', 'warning');
            return;
        }
        settings.unlockPin = pin;
        settings.requireFullscreen = document.getElementById('setRequireFullscreen').checked;
        settings.detectTabSwitch = document.getElementById('setDetectTabSwitch').checked;
        settings.enableSiren = document.getElementById('setEnableSiren').checked;
        settings.showScoreToStudent = document.getElementById('setShowScoreToStudent').checked;
        setDB(STORAGE_KEYS.SETTINGS, settings);
        syncAllViews();
        showToast('Setelan keamanan, sirine & PIN berhasil disimpan!', 'success');
    }

    function handleAddProctor(e) {
        e.preventDefault();
        const proctors = getDB(STORAGE_KEYS.PROCTORS, []);
        proctors.push({
            id: 'proc_' + Date.now(),
            name: document.getElementById('procName').value.trim(),
            room: document.getElementById('procRoom').value.trim(),
            username: document.getElementById('procUser').value.trim(),
            password: document.getElementById('procPass').value.trim()
        });
        setDB(STORAGE_KEYS.PROCTORS, proctors);
        e.target.reset();
        syncAllViews();
        showToast('Akun Pengawas Ruangan berhasil ditambahkan!', 'success');
    }

    function deleteProctor(procId) {
        const proctors = getDB(STORAGE_KEYS.PROCTORS, []).filter(p => p.id !== procId);
        setDB(STORAGE_KEYS.PROCTORS, proctors);
        syncAllViews();
        showToast('Akun pengawas dihapus.', 'info');
    }

    function handleAddGalleryItem(e) {
        e.preventDefault();
        const gallery = getDB(STORAGE_KEYS.GALLERY, DEFAULT_GALLERY);
        const title = document.getElementById('galTitle').value.trim();
        const url = document.getElementById('galUrl').value.trim();
        const types = ['lab', 'proctor', 'student'];
        gallery.unshift({
            id: 'gal_' + Date.now(),
            title,
            url,
            svgType: types[gallery.length % types.length]
        });
        setDB(STORAGE_KEYS.GALLERY, gallery);
        e.target.reset();
        syncAllViews();
        showToast('Foto dokumentasi berhasil ditambahkan ke Galeri Portal!', 'success');
    }

    function exportDatabaseJSON() {
        const payload = {
            settings: getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS),
            packages: getDB(STORAGE_KEYS.PACKAGES, []),
            students: getDB(STORAGE_KEYS.STUDENTS, []),
            proctors: getDB(STORAGE_KEYS.PROCTORS, []),
            liveStatus: getDB(STORAGE_KEYS.LIVE_STATUS, {}),
            logs: getDB(STORAGE_KEYS.LOGS, []),
            gallery: getDB(STORAGE_KEYS.GALLERY, [])
        };
        const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `Backup_CBT_PRO_${new Date().toISOString().slice(0, 10)}.json`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('Cadangan database JSON berhasil diunduh!', 'success');
    }

    function importDatabaseJSON(event) {
        const file = event.target.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = (e) => {
            try {
                const data = JSON.parse(e.target.result);
                if (data.settings) setDB(STORAGE_KEYS.SETTINGS, data.settings);
                if (data.packages) setDB(STORAGE_KEYS.PACKAGES, data.packages);
                if (data.students) setDB(STORAGE_KEYS.STUDENTS, data.students);
                if (data.proctors) setDB(STORAGE_KEYS.PROCTORS, data.proctors);
                if (data.liveStatus) setDB(STORAGE_KEYS.LIVE_STATUS, data.liveStatus);
                if (data.logs) setDB(STORAGE_KEYS.LOGS, data.logs);
                if (data.gallery) setDB(STORAGE_KEYS.GALLERY, data.gallery);
                syncAllViews();
                showToast('Database berhasil dipulihkan dari file backup!', 'success');
            } catch (err) {
                showToast('File backup JSON tidak valid.', 'error');
            } finally {
                event.target.value = '';
            }
        };
        reader.readAsText(file);
    }

    function confirmCleanStartEmpty() {
        showConfirmDialog('Mulai Kosong (Hapus Data Contoh)?', 'Seluruh paket soal awal, akun siswa contoh, log, dan progres ujian akan dikosongkan sehingga Anda dapat mengisi data sekolah dari awal.', () => {
            setDB(STORAGE_KEYS.PACKAGES, []);
            setDB(STORAGE_KEYS.STUDENTS, []);
            setDB(STORAGE_KEYS.LIVE_STATUS, {});
            setDB(STORAGE_KEYS.LOGS, []);
            syncAllViews();
            showToast('Sistem berhasil dikosongkan dan siap diisi data baru.', 'success');
        });
    }

    function confirmFactoryReset() {
        showConfirmDialog('Reset Data ke Pengaturan Awal?', 'Semua data yang telah Anda ubah akan dikembalikan ke data awal bawaan sistem.', () => {
            Object.values(STORAGE_KEYS).forEach(k => localStorage.removeItem(k));
            initializeAppDatabase();
            syncAllViews();
            showToast('Sistem berhasil dikembalikan ke pengaturan awal.', 'info');
        });
    }

    // =================================================================
    // K. DASHBOARD PENGAWAS RUANG: 3 MENU TERSTRUKTUR
    // =================================================================
    function switchProctorMenu(menuName) {
        currentProctorMenu = menuName;
        ['kehadiran', 'pelanggaran', 'soal'].forEach(m => {
            document.getElementById(`proctorSection-${m}`).classList.add('hidden');
            const btn = document.getElementById(`procMenuBtn-${m}`);
            btn.className = 'px-4 py-2.5 rounded-xl text-xs sm:text-sm font-semibold text-slate-600 hover:bg-slate-100 transition flex items-center gap-2';
        });
        document.getElementById(`proctorSection-${menuName}`).classList.remove('hidden');
        const activeBtn = document.getElementById(`procMenuBtn-${menuName}`);
        activeBtn.className = 'px-4 py-2.5 rounded-xl text-xs sm:text-sm font-bold bg-amber-500 text-white transition flex items-center gap-2';
        saveSessionState();
        syncAllViews();
    }

    function renderProctorMonitoring() {
        if (!activeSession || activeSession.role !== 'pengawas') return;
        const proctor = activeSession.user;
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const logs = getDB(STORAGE_KEYS.LOGS, []);

        const roomStudents = proctor.room.toLowerCase().includes('semua')
            ? students
            : students.filter(s => s.room.toLowerCase() === proctor.room.toLowerCase());

        const container = document.getElementById('proctorStudentCards');
        if (roomStudents.length === 0) {
            container.innerHTML = `<div class="bg-slate-50 rounded-2xl border border-slate-200 p-8 text-center text-xs text-slate-400">Tidak ada peserta terdaftar di ruangan tugas Anda (${proctor.room}).</div>`;
        } else {
            container.innerHTML = roomStudents.map(s => {
                const st = liveMap[s.id] || { status: 'Belum Masuk', progress: '-', currentSubject: '-', violations: 0 };
                const isLocked = st.status === 'TERKUNCI';

                let badge = `<span class="px-2.5 py-1 rounded-full bg-slate-100 text-slate-600 text-[11px] font-bold">Belum Masuk</span>`;
                if (st.status === 'Di Dashboard') badge = `<span class="px-2.5 py-1 rounded-full bg-indigo-100 text-indigo-700 text-[11px] font-bold">Masuk Dashboard</span>`;
                else if (st.status === 'Mengerjakan') badge = `<span class="px-2.5 py-1 rounded-full bg-blue-100 text-blue-700 text-[11px] font-bold">Sedang Ujian</span>`;
                else if (isLocked) badge = `<span class="px-2.5 py-1 rounded-full bg-red-600 text-white text-[11px] font-bold animate-bounce">TERKUNCI + SIRINE</span>`;
                else if (st.status === 'Selesai') badge = `<span class="px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-700 text-[11px] font-bold">Selesai Ujian</span>`;
                else if (st.status === 'Keluar / Offline') badge = `<span class="px-2.5 py-1 rounded-full bg-slate-200 text-slate-700 text-[11px] font-bold">Keluar Sistem</span>`;

                return `
                    <div class="rounded-2xl border ${isLocked ? 'border-red-400 bg-red-50/70' : 'border-slate-200 bg-white'} p-4 flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                        <div class="min-w-0">
                            <div class="flex flex-wrap items-center gap-2">
                                <h4 class="font-bold text-slate-900 text-sm">${s.name}</h4>
                                ${badge}
                            </div>
                            <div class="text-xs text-slate-500 mt-1">
                                NISN: <span class="font-mono">${s.nisn}</span> • ${s.className} • <strong>${s.room}</strong> • Mapel: <strong>${st.currentSubject || '-'}</strong>
                            </div>
                        </div>
                        <div class="flex items-center gap-3 shrink-0">
                            <span class="text-xs font-mono font-bold text-slate-700">Progres: ${st.progress || '-'}</span>
                            ${isLocked ? `
                                <button onclick="remoteUnlockStudentById('${s.id}')" class="px-3 py-2 rounded-xl bg-amber-400 hover:bg-amber-300 text-slate-950 text-xs font-extrabold transition shadow-sm">
                                    Matikan Sirine &amp; Buka Kunci
                                </button>
                            ` : ''}
                        </div>
                    </div>
                `;
            }).join('');
        }

        const studentInOutLogs = logs.filter(l => l.role === 'siswa').slice(0, 25);
        const inOutBox = document.getElementById('proctorInOutLogList');
        if (studentInOutLogs.length === 0) {
            inOutBox.innerHTML = `<div class="text-center py-6 text-slate-400">Belum ada aktivitas masuk/keluar siswa.</div>`;
        } else {
            inOutBox.innerHTML = studentInOutLogs.map(l => `
                <div class="p-2.5 rounded-xl border border-slate-100 bg-slate-50 space-y-0.5">
                    <div class="flex justify-between gap-2 text-[10px] font-bold">
                        <span class="${l.category === 'VIOLATION' ? 'text-red-600' : l.category === 'LOGIN' ? 'text-emerald-700' : 'text-blue-700'}">${l.category}</span>
                        <span class="text-slate-400 font-mono">${l.timestamp}</span>
                    </div>
                    <div class="font-bold text-slate-800">${l.userName} (${l.userIdentifier})</div>
                    <div class="text-[11px] text-slate-600">${l.detail}</div>
                </div>
            `).join('');
        }
    }

    function renderProctorViolationsView() {
        const students = getDB(STORAGE_KEYS.STUDENTS, []);
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const logs = getDB(STORAGE_KEYS.LOGS, []);

        const lockedStudents = students.filter(s => liveMap[s.id]?.status === 'TERKUNCI');
        const pill = document.getElementById('proctorViolationCountPill');
        if (pill) pill.innerText = lockedStudents.length;

        const activeBox = document.getElementById('proctorActiveLockedBox');
        if (lockedStudents.length === 0) {
            activeBox.innerHTML = `
                <div class="bg-emerald-50 border border-emerald-200 rounded-2xl p-4 text-xs text-emerald-800 font-semibold flex items-center gap-2">
                    <i data-lucide="shield-check" class="w-4 h-4 text-emerald-600 shrink-0"></i> Tidak ada layar peserta yang sedang terkunci saat ini. Kondisi ruangan kondusif.
                </div>
            `;
        } else {
            activeBox.innerHTML = lockedStudents.map(s => {
                const st = liveMap[s.id];
                return `
                    <div class="bg-red-600 text-white rounded-2xl p-5 shadow-lg flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                        <div>
                            <span class="px-2.5 py-0.5 rounded-full bg-white/20 text-[10px] font-extrabold uppercase tracking-wider">SIRINE ALARM SEDANG BERBUNYI</span>
                            <h4 class="text-lg font-extrabold mt-1">${s.name} (NISN: ${s.nisn})</h4>
                            <p class="text-xs text-red-100">${s.className} • ${s.room} • Total Pelanggaran: <strong>${st.violations || 1}x</strong></p>
                        </div>
                        <button onclick="remoteUnlockStudentById('${s.id}')" class="px-5 py-3 rounded-xl bg-amber-400 hover:bg-amber-300 text-slate-950 font-extrabold text-xs sm:text-sm transition shadow shrink-0">
                            Matikan Sirine &amp; Buka Kunci Jarak Jauh
                        </button>
                    </div>
                `;
            }).join('');
        }

        const vioLogs = logs.filter(l => l.category === 'VIOLATION' || l.category === 'UNLOCK');
        const tbody = document.getElementById('proctorViolationLogsTbody');
        if (vioLogs.length === 0) {
            tbody.innerHTML = `<tr><td colspan="3" class="py-8 text-center text-slate-400 text-xs">Belum ada catatan pelanggaran.</td></tr>`;
        } else {
            tbody.innerHTML = vioLogs.map(v => `
                <tr class="hover:bg-slate-50">
                    <td class="py-3 px-4 font-mono text-xs text-slate-600 whitespace-nowrap">${v.timestamp}</td>
                    <td class="py-3 px-4">
                        <div class="font-bold text-slate-900">${v.userName}</div>
                        <div class="text-[11px] font-mono text-slate-500">${v.userIdentifier} • ${v.roomOrClass}</div>
                    </td>
                    <td class="py-3 px-4 text-xs ${v.category === 'VIOLATION' ? 'text-red-700 font-bold' : 'text-emerald-700 font-semibold'}">${v.detail}</td>
                </tr>
            `).join('');
        }
    }

    function selectProctorQuestionPackage(pkgId) {
        selectedProctorPkgId = pkgId;
        renderProctorSoalView();
    }

    function renderProctorSoalView() {
        const packages = getDB(STORAGE_KEYS.PACKAGES, []);
        const sel = document.getElementById('proctorPackageSelector');
        if (!sel) return;

        if (packages.length === 0) {
            sel.innerHTML = `<option value="">-- Belum Ada Paket Mapel --</option>`;
            document.getElementById('proctorPkgSoalCount').innerText = '0';
            document.getElementById('proctorSoalListContainer').innerHTML = `<div class="text-center py-10 text-xs text-slate-400">Administrator belum membuat paket mata pelajaran.</div>`;
            return;
        }

        if (!packages.some(p => p.id === selectedProctorPkgId)) {
            selectedProctorPkgId = packages[0].id;
        }

        sel.innerHTML = packages.map(p => `<option value="${p.id}">${p.subjectName} (${(p.questions || []).length} Soal)</option>`).join('');
        sel.value = selectedProctorPkgId;

        const pkg = packages.find(p => p.id === selectedProctorPkgId);
        const questions = pkg ? (pkg.questions || []) : [];
        document.getElementById('proctorPkgSoalCount').innerText = questions.length;

        const container = document.getElementById('proctorSoalListContainer');
        if (questions.length === 0) {
            container.innerHTML = `<div class="text-center py-10 text-xs text-slate-400">Paket soal masih kosong. Upload file Word/PDF atau input soal di samping.</div>`;
        } else {
            container.innerHTML = questions.map((q, idx) => `
                <div class="p-3.5 rounded-2xl border border-slate-200 bg-slate-50/70 space-y-1.5 text-xs">
                    <div class="font-bold text-slate-900 break-words"><span class="text-amber-700">${idx + 1}.</span> ${q.question}</div>
                    <div class="text-[11px] text-emerald-700 font-bold">Kunci Jawaban: Opsi ${q.answer} (${q.points || 10} Poin)</div>
                </div>
            `).join('');
        }
    }

    // =================================================================
    // L. MESIN UJIAN SISWA, PENGACAKAN & ANTI-CHEAT LOCKDOWN + SIRINE
    // =================================================================
    function shuffleArray(arr) {
        const copy = [...arr];
        for (let i = copy.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [copy[i], copy[j]] = [copy[j], copy[i]];
        }
        return copy;
    }

    function beginStudentExam(student, pkg) {
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);

        let preparedQuestions = (pkg.questions || []).map(q => ({
            ...q,
            options: { ...q.options }
        }));

        if (pkg.shuffleQuestions) {
            preparedQuestions = shuffleArray(preparedQuestions);
        }

        if (pkg.shuffleOptions) {
            preparedQuestions = preparedQuestions.map(q => {
                const keys = Object.keys(q.options);
                const originalCorrectText = q.options[q.answer];
                const shuffledTexts = shuffleArray(keys.map(k => q.options[k]));
                const newOptions = {};
                let newCorrectKey = 'A';
                keys.forEach((k, i) => {
                    newOptions[k] = shuffledTexts[i];
                    if (shuffledTexts[i] === originalCorrectText) {
                        newCorrectKey = k;
                    }
                });
                return { ...q, options: newOptions, answer: newCorrectKey };
            });
        }

        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const prevData = liveMap[student.id] || {};
        examSession = {
            isRunning: true,
            isLocked: false,
            packageId: pkg.id,
            packageSubject: pkg.subjectName,
            packageTitle: pkg.examTitle,
            questions: preparedQuestions,
            answers: {},
            doubts: {},
            currentIdx: 0,
            remainingSeconds: (parseInt(pkg.durationMinutes, 10) || 45) * 60,
            timerId: null,
            violations: prevData.violations || 0
        };

        document.getElementById('studentExamTitle').innerText = `${pkg.subjectName} — ${pkg.examTitle}`;
        document.getElementById('studentExamName').innerText = student.name;
        document.getElementById('studentExamNisn').innerText = student.nisn;
        document.getElementById('studentExamRoom').innerText = `${student.className} • ${student.room}`;
        document.getElementById('examTotalNum').innerText = preparedQuestions.length;

        updateStudentLiveRecord('Mengerjakan');
        recordSystemLog('siswa', student.name, student.nisn, `${student.className} • ${student.room}`, 'EXAM_START', `Memulai pengerjaan ujian: ${pkg.subjectName}.`);

        navigateTo('siswa');
        renderCurrentExamQuestion();
        startExamCountdown();

        if (settings.requireFullscreen && document.documentElement.requestFullscreen) {
            document.documentElement.requestFullscreen().catch(() => {});
        }
    }

    function updateStudentLiveRecord(statusOverride = null, extraFields = {}) {
        if (!activeSession || activeSession.role !== 'siswa') return;
        const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
        const prev = liveMap[activeSession.user.id] || {};
        const answeredCount = Object.keys(examSession.answers).length;
        const totalCount = examSession.questions.length;

        liveMap[activeSession.user.id] = {
            ...prev,
            status: statusOverride || prev.status || 'Mengerjakan',
            currentSubject: examSession.packageSubject || prev.currentSubject || '-',
            progress: `${answeredCount}/${totalCount}`,
            violations: examSession.violations,
            updatedAt: new Date().toLocaleTimeString('id-ID'),
            ...extraFields
        };
        setDB(STORAGE_KEYS.LIVE_STATUS, liveMap);
    }

    function startExamCountdown() {
        if (examSession.timerId) clearInterval(examSession.timerId);
        updateTimerDisplay();

        examSession.timerId = setInterval(() => {
            if (!examSession.isRunning) return;
            examSession.remainingSeconds--;
            updateTimerDisplay();

            if (examSession.remainingSeconds <= 0) {
                clearInterval(examSession.timerId);
                showToast('Waktu ujian telah habis! Jawaban dikirim otomatis.', 'warning');
                finalizeAndSubmitExam();
            }
        }, 1000);
    }

    function updateTimerDisplay() {
        const mins = Math.floor(Math.max(0, examSession.remainingSeconds) / 60);
        const secs = Math.max(0, examSession.remainingSeconds) % 60;
        const text = `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
        const el = document.getElementById('studentTimerDisplay');
        el.innerText = text;
        if (examSession.remainingSeconds <= 300) {
            el.classList.remove('text-amber-400');
            el.classList.add('text-red-400', 'animate-pulse');
        }
    }

    function renderCurrentExamQuestion() {
        const idx = examSession.currentIdx;
        const q = examSession.questions[idx];
        if (!q) return;

        document.getElementById('examCurrentNum').innerText = idx + 1;
        document.getElementById('examQuestionBody').innerText = q.question;

        const isDoubt = !!examSession.doubts[q.id];
        const doubtBtn = document.getElementById('doubtToggleBtn');
        doubtBtn.className = isDoubt
            ? 'px-3.5 py-1.5 rounded-xl border border-amber-500 bg-amber-400 text-slate-950 text-xs font-extrabold transition flex items-center gap-1.5'
            : 'px-3.5 py-1.5 rounded-xl border border-amber-300 bg-amber-50 hover:bg-amber-100 text-amber-800 text-xs font-bold transition flex items-center gap-1.5';

        const selectedAnswer = examSession.answers[q.id];
        const optionKeys = Object.keys(q.options);

        document.getElementById('examOptionsList').innerHTML = optionKeys.map(k => {
            const isSelected = selectedAnswer === k;
            return `
                <div onclick="selectExamOption('${q.id}', '${k}')" class="p-4 rounded-2xl border-2 cursor-pointer transition flex items-start gap-3.5 ${isSelected ? 'border-blue-600 bg-blue-50/70' : 'border-slate-200 hover:border-blue-300 bg-white'}">
                    <div class="w-8 h-8 rounded-xl flex items-center justify-center font-extrabold text-xs shrink-0 ${isSelected ? 'bg-blue-600 text-white' : 'bg-slate-100 text-slate-700'}">
                        ${k}
                    </div>
                    <div class="text-sm font-medium text-slate-800 pt-1 leading-relaxed break-words flex-1">${q.options[k]}</div>
                </div>
            `;
        }).join('');

        document.getElementById('btnPrevQuestion').disabled = idx === 0;
        const nextBtn = document.getElementById('btnNextQuestion');
        if (idx === examSession.questions.length - 1) {
            nextBtn.innerHTML = `Selesai Ujian <i data-lucide="check-circle" class="w-4 h-4"></i>`;
            nextBtn.className = 'px-5 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-xs sm:text-sm font-bold transition flex items-center gap-2';
            nextBtn.onclick = confirmSubmitExam;
        } else {
            nextBtn.innerHTML = `Selanjutnya <i data-lucide="chevron-right" class="w-4 h-4"></i>`;
            nextBtn.className = 'px-5 py-2.5 rounded-xl bg-blue-600 hover:bg-blue-700 text-white text-xs sm:text-sm font-bold transition flex items-center gap-2';
            nextBtn.onclick = goNextQuestion;
        }

        renderExamNavigationGrid();
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function renderExamNavigationGrid() {
        const grid = document.getElementById('examNumberGrid');
        grid.innerHTML = examSession.questions.map((q, i) => {
            const isCurrent = i === examSession.currentIdx;
            const isAnswered = !!examSession.answers[q.id];
            const isDoubt = !!examSession.doubts[q.id];

            let cls = 'bg-slate-100 text-slate-700 border-slate-300';
            if (isDoubt) cls = 'bg-amber-400 text-slate-950 border-amber-500 font-extrabold';
            else if (isAnswered) cls = 'bg-blue-600 text-white border-blue-600 font-bold';

            const ring = isCurrent ? 'ring-2 ring-slate-900 ring-offset-2' : '';

            return `
                <button onclick="jumpToQuestion(${i})" class="h-10 rounded-xl border text-xs transition ${cls} ${ring}">
                    ${i + 1}
                </button>
            `;
        }).join('');
    }

    function selectExamOption(qId, optKey) {
        examSession.answers[qId] = optKey;
        updateStudentLiveRecord('Mengerjakan');
        renderCurrentExamQuestion();
    }

    function toggleDoubtQuestion() {
        const q = examSession.questions[examSession.currentIdx];
        if (!q) return;
        examSession.doubts[q.id] = !examSession.doubts[q.id];
        renderCurrentExamQuestion();
    }

    function goPrevQuestion() {
        if (examSession.currentIdx > 0) {
            examSession.currentIdx--;
            renderCurrentExamQuestion();
        }
    }

    function goNextQuestion() {
        if (examSession.currentIdx < examSession.questions.length - 1) {
            examSession.currentIdx++;
            renderCurrentExamQuestion();
        }
    }

    function jumpToQuestion(idx) {
        examSession.currentIdx = idx;
        renderCurrentExamQuestion();
    }

    function changeExamFontSize(size) {
        const body = document.getElementById('examQuestionBody');
        body.classList.remove('text-sm', 'text-base', 'text-lg', 'text-xl');
        if (size === 'sm') body.classList.add('text-sm');
        else if (size === 'lg') body.classList.add('text-xl');
        else body.classList.add('text-base');
    }

    function confirmSubmitExam() {
        const answered = Object.keys(examSession.answers).length;
        const total = examSession.questions.length;
        const msg = answered < total
            ? `Anda baru menjawab ${answered} dari ${total} soal. Yakin ingin mengakhiri dan mengirim jawaban sekarang?`
            : `Seluruh ${total} soal telah terjawab. Kirim jawaban ujian Anda sekarang?`;

        showConfirmDialog('Kirim Jawaban Ujian?', msg, finalizeAndSubmitExam);
    }

    function finalizeAndSubmitExam() {
        if (!examSession.isRunning) return;
        examSession.isRunning = false;
        examSession.isLocked = false;
        stopSirenAlarm();
        if (examSession.timerId) clearInterval(examSession.timerId);

        let totalPoints = 0;
        let earnedPoints = 0;
        let correctCount = 0;
        let wrongCount = 0;

        examSession.questions.forEach(q => {
            const pts = q.points || 10;
            totalPoints += pts;
            if (examSession.answers[q.id] === q.answer) {
                earnedPoints += pts;
                correctCount++;
            } else {
                wrongCount++;
            }
        });

        const finalScore = totalPoints > 0 ? Math.round((earnedPoints / totalPoints) * 100) : 0;
        const finishedAt = new Date().toLocaleTimeString('id-ID');

        if (activeSession?.role === 'siswa') {
            const s = activeSession.user;
            const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
            const prev = liveMap[s.id] || {};
            const completedPackages = prev.completedPackages || {};

            completedPackages[examSession.packageId] = {
                subjectName: examSession.packageSubject,
                examTitle: examSession.packageTitle,
                correct: correctCount,
                wrong: wrongCount,
                score: finalScore,
                finishedAt
            };

            updateStudentLiveRecord('Selesai', {
                correct: correctCount,
                wrong: wrongCount,
                score: finalScore,
                finishedAt,
                completedPackages
            });

            recordSystemLog(
                'siswa',
                s.name,
                s.nisn,
                `${s.className} • ${s.room}`,
                'EXAM_FINISH',
                `Menyelesaikan ujian ${examSession.packageSubject} (Benar: ${correctCount}, Salah: ${wrongCount}, Nilai: ${finalScore}).`
            );
        }

        if (document.fullscreenElement && document.exitFullscreen) {
            document.exitFullscreen().catch(() => {});
        }

        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        const scoreBox = document.getElementById('studentScoreSummaryBox');
        if (settings.showScoreToStudent) {
            scoreBox.classList.remove('hidden');
            document.getElementById('studentFinalScoreText').innerText = finalScore;
            document.getElementById('studentFinalDetailText').innerText = `Benar: ${correctCount} • Salah: ${wrongCount}`;
        } else {
            scoreBox.classList.add('hidden');
        }

        const modal = document.getElementById('examFinishedModal');
        modal.classList.remove('hidden');
        modal.classList.add('flex');
        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function closeExamFinishedModal() {
        const modal = document.getElementById('examFinishedModal');
        modal.classList.add('hidden');
        modal.classList.remove('flex');
        if (activeSession && activeSession.role === 'siswa') {
            navigateTo('siswa-dashboard');
        } else {
            navigateTo('portal');
        }
    }

    // =================================================================
    // M. SISTEM KEAMANAN ANTI-CURANG + SIRINE OTOMATIS
    // =================================================================
    let lastViolationTime = 0;

    function triggerAntiCheatLock(reasonText) {
        if (!examSession.isRunning) return;
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        if (!settings.detectTabSwitch) return;

        const now = Date.now();
        if (now - lastViolationTime < 1500 && examSession.isLocked) return;
        lastViolationTime = now;

        if (!examSession.isLocked) {
            examSession.violations++;
        }
        examSession.isLocked = true;

        startSirenAlarm();

        const student = activeSession?.user || { name: 'Peserta', nisn: '-', className: '-', room: '-' };
        document.getElementById('lockReasonText').innerText = reasonText;
        document.getElementById('lockStudentName').innerText = student.name;
        document.getElementById('lockStudentMeta').innerText = `${student.nisn} • ${student.className} (${student.room})`;
        document.getElementById('lockExamSubject').innerText = examSession.packageSubject || '-';
        document.getElementById('lockViolationBadge').innerText = `${examSession.violations} Kali`;
        document.getElementById('unlockPinField').value = '';

        const overlay = document.getElementById('antiCheatLockOverlay');
        overlay.classList.remove('hidden');
        overlay.classList.add('flex');

        updateStudentLiveRecord('TERKUNCI', { remoteUnlocked: false });
        recordSystemLog(
            'siswa',
            student.name,
            student.nisn,
            `${student.className} • ${student.room}`,
            'VIOLATION',
            `Pelanggaran ke-${examSession.violations} pada ${examSession.packageSubject}: ${reasonText} (Sirine Alarm Aktif)`
        );

        if (typeof lucide !== 'undefined') lucide.createIcons();
    }

    function verifyPinUnlock() {
        const inputPin = document.getElementById('unlockPinField').value.trim();
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);

        if (inputPin === settings.unlockPin) {
            examSession.isLocked = false;
            stopSirenAlarm();

            const overlay = document.getElementById('antiCheatLockOverlay');
            overlay.classList.add('hidden');
            overlay.classList.remove('flex');
            updateStudentLiveRecord('Mengerjakan');

            const student = activeSession?.user;
            if (student) {
                recordSystemLog(
                    'siswa',
                    student.name,
                    student.nisn,
                    `${student.className} • ${student.room}`,
                    'UNLOCK',
                    `Sirine dimatikan & kunci layar dibuka menggunakan PIN Pengawas.`
                );
            }

            if (settings.requireFullscreen && document.documentElement.requestFullscreen) {
                document.documentElement.requestFullscreen().catch(() => {});
            }
            showToast('Sirine dimatikan & kunci layar berhasil dibuka oleh Pengawas!', 'success');
        } else {
            showToast('Kode PIN Pengawas salah! Sirine tetap menyala.', 'error');
            document.getElementById('unlockPinField').value = '';
        }
    }

    document.addEventListener('visibilitychange', () => {
        if (document.hidden && examSession.isRunning) {
            triggerAntiCheatLock('Terdeteksi berpindah tab browser atau meminimalkan aplikasi ujian!');
        }
    });

    window.addEventListener('blur', () => {
        if (examSession.isRunning) {
            triggerAntiCheatLock('Fokus layar ujian hilang (membuka jendela/aplikasi lain atau split-screen)!');
        }
    });

    document.addEventListener('fullscreenchange', () => {
        const settings = getDB(STORAGE_KEYS.SETTINGS, DEFAULT_SETTINGS);
        if (!document.fullscreenElement && examSession.isRunning && settings.requireFullscreen) {
            triggerAntiCheatLock('Peserta keluar dari mode Layar Penuh (Fullscreen)!');
        }
    });

    document.addEventListener('contextmenu', (e) => {
        if (examSession.isRunning) e.preventDefault();
    });

    document.addEventListener('keydown', (e) => {
        if (!examSession.isRunning) return;
        if ((e.ctrlKey || e.metaKey) && ['c', 'v', 'x', 'u', 's', 'p', 'a'].includes(e.key.toLowerCase())) {
            e.preventDefault();
            showToast('Pintasan keyboard dinonaktifkan selama ujian!', 'warning');
        }
        if (e.key === 'F12') e.preventDefault();
    });

    window.addEventListener('beforeunload', (e) => {
        if (examSession.isRunning) {
            e.preventDefault();
            e.returnValue = '';
        }
    });

    // =================================================================
    // N. SINKRONISASI REAL-TIME ANTAR TAB & INISIALISASI APLIKASI
    // =================================================================
    window.addEventListener('storage', (e) => {
        if (Object.values(STORAGE_KEYS).includes(e.key)) {
            if (examSession.isRunning && examSession.isLocked && activeSession?.role === 'siswa') {
                const liveMap = getDB(STORAGE_KEYS.LIVE_STATUS, {});
                const myStatus = liveMap[activeSession.user.id];
                if (myStatus && myStatus.status === 'Mengerjakan' && myStatus.remoteUnlocked) {
                    examSession.isLocked = false;
                    stopSirenAlarm();
                    const overlay = document.getElementById('antiCheatLockOverlay');
                    overlay.classList.add('hidden');
                    overlay.classList.remove('flex');
                    showToast('Pengawas telah mematikan sirine & membuka kunci layar Anda dari jarak jauh!', 'success');
                }
            }
            syncAllViews();
        }
    });

    setInterval(() => {
        if (activeSession && !examSession.isRunning) {
            syncAllViews();
        }
    }, 4000);

    document.addEventListener('DOMContentLoaded', () => {
        initializeAppDatabase();

        // Pulihkan sesi login jika halaman di-refresh dari Dashboard Siswa, Pengawas, atau Admin
        const restored = restoreSessionState();
        if (!restored) {
            navigateTo('portal');
            const params = new URLSearchParams(window.location.search);
            const roleParam = params.get('role');
            if (roleParam && ['siswa', 'pengawas', 'admin'].includes(roleParam)) {
                openLoginModal(roleParam);
            }
        }
    });
</script>

```
