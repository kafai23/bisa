# bisa
2026
<!DOCTYPE html>
<html lang="id">
<head>    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HAPPY NEW YEAR 2026 - Interaktif</title>
    
    <!-- Import Font yang Keren dari Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Oswald:wght@700&family=Dancing+Script:wght@700&display=swap" rel="stylesheet">

    <style>
        /* Menghilangkan margin, padding, dan scroll bar */
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden; /* Mencegah scroll bar */
            background-color: #000; /* Latar belakang hitam */
        }

        /* Membuat canvas memenuhi seluruh layar */
        canvas {
            display: block;
            cursor: none; /* Sembunyikan kursor */
        }

        /* --- GAYA UNTUK FOOTER TEKS --- */
        #footer-text {
            position: fixed; /* Posisi tetap di layar */
            bottom: 20px; /* Jarak 20px dari bawah */
            left: 0;
            width: 100%;
            text-align: center;
            
            /* Gaya Teks */
            color: rgba(255, 255, 255, 0.6); /* Warna putih sedikit transparan */
            font-family: 'Dancing Script', cursive; /* Gunakan font yang sama untuk konsistensi */
            font-size: 28px;
            font-weight: bold;
            
            /* Efek Glow */
            text-shadow: 0 0 8px #fff, 0 0 12px #FFD700;
            
            /* Agar tidak mengganggu event mouse di canvas */
            pointer-events: none;
            z-index: 10; /* Pastikan berada di atas canvas */
        }
    </style>
</head>
<body>

    <!-- Elemen canvas tempat semua animasi digambar -->
    <canvas id="newYearCanvas"></canvas>

    <!-- Elemen Footer Teks -->
    <div id="footer-text">*jagoangroup*</div>

    <script>
        // --- SETUP AWAL ---
        const canvas = document.getElementById('newYearCanvas');
        const ctx = canvas.getContext('2d');

        // Mengatur ukuran canvas agar selalu penuh
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Array untuk menyimpan semua partikel
        let particles = [];
        let fireworks = [];

        // Timer untuk mendeteksi kapan mouse berhenti bergerak
        let mouseStopTimer;

        // --- KELAS-KELAS PARTIKEL ---

        // Partikel kecil yang berkilauan di latar belakang
        class BackgroundParticle {
            constructor() {
                this.x = Math.random() * canvas.width;
                this.y = canvas.height + Math.random() * 100; // Muncul dari bawah
                this.vx = (Math.random() - 0.5) * 0.5; // Gerakan horizontal pelan
                this.vy = -Math.random() * 2 - 1; // Gerakan ke atas
                this.size = Math.random() * 3 + 1;
                this.color = `hsl(${Math.random() * 60 + 30}, 100%, 70%)`; // Warna emas/kuning
                this.alpha = 1;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.alpha -= 0.005; // Perlahan menghilang
            }

            draw() {
                ctx.save();
                ctx.globalAlpha = this.alpha;
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
        }

        // Partikel dari ledakan kembang api
        class FireworkParticle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                const angle = Math.random() * Math.PI * 2;
                const velocity = Math.random() * 6 + 2;
                this.vx = Math.cos(angle) * velocity;
                this.vy = Math.sin(angle) * velocity;
                this.color = color;
                this.alpha = 1;
                this.decay = Math.random() * 0.02 + 0.01;
            }

            update() {
                this.vx *= 0.99; // Gesekan udara
                this.vy *= 0.99;
                this.vy += 0.05; // Gravitasi
                this.x += this.vx;
                this.y += this.vy;
                this.alpha -= this.decay;
            }

            draw() {
                ctx.save();
                ctx.globalAlpha = this.alpha;
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 20;
                ctx.shadowColor = this.color;
                ctx.fillRect(this.x, this.y, 3, 3);
                ctx.restore();
            }
        }

        // Kembang api utama
        class Firework {
            constructor() {
                this.x = Math.random() * canvas.width;
                this.targetY = Math.random() * canvas.height * 0.5; // Target ledakan di atas tengah
                this.y = canvas.height;
                this.vy = -Math.random() * 3 - 12; // Kecepatan awal ke atas
                this.exploded = false;
                this.color = `hsl(${Math.random() * 360}, 100%, 60%)`;
            }

            update() {
                if (!this.exploded) {
                    this.y += this.vy;
                    this.vy += 0.15; // Gravitasi
                    
                    // Meledak jika mencapai target atau kecepatan sudah habis
                    if (this.vy >= 0 || this.y <= this.targetY) {
                        this.explode();
                    }
                }
            }

            explode() {
                this.exploded = true;
                for (let i = 0; i < 50; i++) {
                    particles.push(new FireworkParticle(this.x, this.y, this.color));
                }
            }

            draw() {
                if (!this.exploded) {
                    ctx.save();
                    ctx.fillStyle = 'white';
                    ctx.shadowBlur = 20;
                    ctx.shadowColor = 'white';
                    ctx.fillRect(this.x, this.y, 2, 10);
                    ctx.restore();
                }
            }
        }

        // --- FUNGSI UTAMA ---

        // Membuat satu ledakan kembang api di posisi x, y (UNTUK MOUSE)
        function createFirework(x, y) {
            const particleCount = 30; // Jumlah partikel per ledakan
            const hue = Math.random() * 360; // Warna dasar acak

            for (let i = 0; i < particleCount; i++) {
                // Membuat variasi warna dalam satu ledakan
                const color = `hsl(${hue + Math.random() * 30 - 15}, 100%, 50%)`;
                particles.push(new FireworkParticle(x, y, color));
            }
        }

        // Loop animasi utama
        function animate() {
            // Membuat efek jejak dengan menutupi canvas dengan warna hitam transparan
            ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Buat partikel latar belakang baru secara terus-menerus
            if (Math.random() < 0.1) {
                particles.push(new BackgroundParticle());
            }

            // Proses dan gambar semua partikel
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.update();
                
                // Hapus partikel yang sudah tidak terlihat lagi
                if (p.alpha <= 0) {
                    particles.splice(i, 1);
                } else {
                    p.draw();
                }
            }

            // Proses dan gambar kembang api otomatis
            for (let i = fireworks.length - 1; i >= 0; i--) {
                const f = fireworks[i];
                f.update();
                f.draw();
                if (f.exploded) {
                    fireworks.splice(i, 1);
                }
            }

            // Gambar teks utama
            drawText();

            // Memanggil fungsi ini lagi untuk frame berikutnya
            requestAnimationFrame(animate);
        }

        function drawText() {
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;

            // Teks "2026"
            ctx.save();
            ctx.font = 'bold 120px Oswald';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            
            // Efek glow (bayangan)
            ctx.shadowBlur = 30;
            ctx.shadowColor = '#FFD700'; // Warna emas
            
            // Gradient untuk teks
            const gradient = ctx.createLinearGradient(0, centerY - 60, 0, centerY + 60);
            gradient.addColorStop(0, '#FFFFFF');
            gradient.addColorStop(0.5, '#FFD700');
            gradient.addColorStop(1, '#FFA500');
            ctx.fillStyle = gradient;
            
            ctx.fillText('2026', centerX, centerY);
            ctx.restore();

            // Teks "HAPPY NEW YEAR"
            ctx.save();
            ctx.font = 'bold 50px Dancing Script';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'top';
            ctx.shadowBlur = 20;
            ctx.shadowColor = '#FFFFFF';
            ctx.fillStyle = '#FFFFFF';
            ctx.fillText('Happy New Year', centerX, centerY + 80);
            ctx.restore();
        }

        // --- EVENT LISTENER UNTUK MOUSE ---

        // Fungsi yang dijalankan saat mouse bergerak
        function handleMouseMove(e) {
            createFirework(e.clientX, e.clientY);

            // Hentikan timer yang ada
            clearTimeout(mouseStopTimer);

            // Set timer baru. Jika mouse tidak bergerak dalam 100ms, timer ini akan berjalan
            mouseStopTimer = setTimeout(() => {
                // Tidak ada yang perlu dilakukan di sini, karena kita hanya berhenti membuat kembang api baru
                // Partikel yang sudah ada akan terus beranimasi sampai habis
            }, 100);
        }

        // Tambahkan event listener untuk pergerakan mouse
        canvas.addEventListener('mousemove', handleMouseMove);

        // Juga hentikan pembuatan kembang api saat mouse keluar dari area canvas
        canvas.addEventListener('mouseout', () => {
            clearTimeout(mouseStopTimer);
        });

        // --- INISIALISASI AWAL ---
        
        // Buat kembang api baru secara berkala
        setInterval(() => {
            fireworks.push(new Firework());
        }, 1500); // Ledakan baru setiap 1.5 detik

        // Mulai loop animasi
        animate();

    </script>
</body>
</html>
