<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Energy Battle - Cuộc chiến năng lượng</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts for modern typography -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&family=Orbitron:wght@500;700;900&display=swap" rel="stylesheet">
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: radial-gradient(circle at center, #0b1528 0%, #030712 100%);
            color: #f3f4f6;
            overflow-x: hidden;
        }
        .font-orbitron {
            font-family: 'Orbitron', sans-serif;
        }
        /* Custom Glowing and Energy Animations */
        .glow-blue {
            box-shadow: 0 0 20px rgba(59, 130, 246, 0.6);
        }
        .glow-yellow {
            box-shadow: 0 0 20px rgba(234, 179, 8, 0.6);
        }
        .glow-green {
            box-shadow: 0 0 20px rgba(34, 197, 94, 0.6);
        }
        .pulse-slow {
            animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: .8; transform: scale(1.03); }
        }
        /* Shake effect for hits */
        .shake {
            animation: shake 0.5s;
        }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }
        /* Beam laser animation */
        .energy-beam {
            background: linear-gradient(90deg, #3b82f6 0%, #eab308 50%, #ef4444 100%);
            background-size: 200% 100%;
            animation: moveBeam 2s linear infinite;
        }
        @keyframes moveBeam {
            0% { background-position: 0% 50%; }
            100% { background-position: 200% 50%; }
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between">

    <!-- Firebase Script Import Module -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot, collection, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global state and initialization parameters
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'energy-battle-1v1';
        let firebaseConfig = {};
        let db = null;
        let auth = null;
        let currentUser = null;

        if (typeof __firebase_config !== 'undefined') {
            try {
                firebaseConfig = JSON.parse(__firebase_config);
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
            } catch (e) {
                console.warn("Firebase initialization skipped or failed: using local mode fallback.", e);
            }
        }

        // Expose Firebase components for application usage
        window.db = db;
        window.auth = auth;
        window.appId = appId;
        window.signInAnonymously = signInAnonymously;
        window.signInWithCustomToken = signInWithCustomToken;
    </script>

    <script>
        // Web Audio Synthesizer Class for sound effects without external files
        class SoundSynth {
            constructor() {
                this.ctx = null;
            }
            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
            }
            playCorrect() {
                this.init();
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(523.25, this.ctx.currentTime); // C5
                osc.frequency.exponentialRampToValueAtTime(880, this.ctx.currentTime + 0.15); // A5
                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.3);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.3);
            }
            playIncorrect() {
                this.init();
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(220, this.ctx.currentTime); // A3
                osc.frequency.linearRampToValueAtTime(110, this.ctx.currentTime + 0.25); // A2
                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.3);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.3);
            }
            playTug() {
                this.init();
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(440, this.ctx.currentTime);
                osc.frequency.linearRampToValueAtTime(600, this.ctx.currentTime + 0.1);
                gain.gain.setValueAtTime(0.1, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.15);
            }
            playWin() {
                this.init();
                if (!this.ctx) return;
                const notes = [261.63, 329.63, 392.00, 523.25]; // C major chord
                notes.forEach((freq, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime + idx * 0.12);
                    gain.gain.setValueAtTime(0.1, this.ctx.currentTime + idx * 0.12);
                    gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + idx * 0.12 + 0.4);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(this.ctx.currentTime + idx * 0.12);
                    osc.stop(this.ctx.currentTime + idx * 0.12 + 0.4);
                });
            }
        }
        const sound = new SoundSynth();
    </script>

    <!-- Header Section -->
    <header class="p-4 bg-gray-900 border-b border-gray-800 shadow-md">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div class="flex items-center space-x-3 cursor-pointer" onclick="goToHome()">
                <div class="p-2 bg-yellow-500 rounded-lg shadow-lg text-gray-950 font-bold flex items-center justify-center">
                    <i class="fa-solid fa-bolt text-xl animate-bounce"></i>
                </div>
                <div>
                    <h1 class="font-orbitron text-xl md:text-2xl font-black tracking-wider text-yellow-400">ENERGY BATTLE</h1>
                    <p class="text-xs text-blue-400 font-semibold tracking-widest">CUỘC CHIẾN NĂNG LƯỢNG 1V1</p>
                </div>
            </div>
            <div class="flex items-center space-x-2">
                <button onclick="showInfoModal()" class="px-3 py-1.5 bg-gray-800 hover:bg-gray-700 transition text-sm rounded-lg text-gray-300 flex items-center gap-1.5">
                    <i class="fa-solid fa-circle-info"></i> Hướng dẫn
                </button>
                <div id="authStatusIndicator" class="text-xs bg-red-900/40 text-red-400 px-3 py-1.5 rounded-lg border border-red-800 flex items-center gap-1.5">
                    <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse"></span> Ngoại tuyến (Local Only)
                </div>
            </div>
        </div>
    </header>

    <!-- Custom Modal for Notifications and Game Rules -->
    <div id="customModal" class="hidden fixed inset-0 z-50 bg-black/80 flex items-center justify-center p-4">
        <div class="bg-gray-900 border border-gray-800 rounded-2xl max-w-md w-full p-6 text-center shadow-2xl relative overflow-hidden">
            <div class="absolute -top-10 -left-10 w-32 h-32 bg-yellow-500/10 rounded-full filter blur-xl"></div>
            <div class="absolute -bottom-10 -right-10 w-32 h-32 bg-blue-500/10 rounded-full filter blur-xl"></div>
            <div class="mx-auto w-16 h-16 bg-yellow-500/20 text-yellow-400 rounded-full flex items-center justify-center text-3xl mb-4">
                <i id="modalIcon" class="fa-solid fa-circle-exclamation"></i>
            </div>
            <h3 id="modalTitle" class="text-xl font-bold font-orbitron text-white mb-2">Thông báo</h3>
            <p id="modalBody" class="text-gray-400 text-sm mb-6">Nội dung thông báo...</p>
            <button onclick="closeModal()" class="w-full py-3 bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-gray-950 font-bold rounded-xl shadow-lg transition duration-200">Đóng</button>
        </div>
    </div>

    <!-- MAIN APP WRAPPER -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 py-6 flex flex-col justify-center">

        <!-- SCREEN: HOME / SELECTION -->
        <section id="screenHome" class="space-y-8 animate-fade-in max-w-4xl mx-auto w-full">
            <div class="text-center space-y-4">
                <span class="px-4 py-1.5 bg-yellow-500/10 text-yellow-400 text-xs font-bold uppercase tracking-widest rounded-full border border-yellow-500/30">Học Tập Gamification</span>
                <h2 class="text-3xl md:text-5xl font-extrabold tracking-tight text-white leading-tight">
                    Đấu Trường Tri Thức <br class="hidden md:block">
                    <span class="text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 via-emerald-400 to-blue-500">Năng Lượng Tái Tạo</span>
                </h2>
                <p class="text-gray-400 max-w-xl mx-auto text-sm md:text-base">
                    Trò chơi đối kháng trực tiếp giúp bạn làm chủ các kiến thức năng lượng: Mặt trời, Gió, Nước, và các tài nguyên hóa thạch. Đẩy lùi đối thủ để giành chiến thắng!
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 pt-4">
                <!-- Offline Split Screen Game Card -->
                <div class="bg-gray-900/60 border border-gray-800 rounded-2xl p-6 hover:border-blue-500/40 transition duration-300 hover:shadow-lg hover:shadow-blue-900/10 flex flex-col justify-between group">
                    <div class="space-y-4">
                        <div class="w-12 h-12 rounded-xl bg-gradient-to-br from-blue-500 to-indigo-600 flex items-center justify-center text-xl text-white shadow-lg shadow-blue-500/20">
                            <i class="fa-solid fa-gamepad group-hover:rotate-12 transition duration-300"></i>
                        </div>
                        <h3 class="text-xl font-bold text-white font-orbitron">Chơi Offline 1v1</h3>
                        <p class="text-gray-400 text-sm leading-relaxed">
                            Hai người chơi đối kháng trực diện trên cùng một màn hình (Chia đôi giao diện). Hoàn hảo khi học nhóm trực tiếp hoặc sử dụng trên cùng máy tính/máy tính bảng.
                        </p>
                    </div>
                    <button onclick="startLocalSetup()" class="mt-6 w-full py-3 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-500 hover:to-indigo-500 text-white font-bold rounded-xl transition duration-200 shadow-md">
                        Chơi Ngay (Không cần mạng)
                    </button>
                </div>

                <!-- Online Real-Time Game Card -->
                <div class="bg-gray-900/60 border border-gray-800 rounded-2xl p-6 hover:border-yellow-500/40 transition duration-300 hover:shadow-lg hover:shadow-yellow-900/10 flex flex-col justify-between group relative overflow-hidden">
                    <div class="absolute top-3 right-3">
                        <span class="px-2.5 py-0.5 bg-yellow-500/20 text-yellow-400 border border-yellow-500/30 text-[10px] font-bold rounded-full uppercase">Real-Time Cloud</span>
                    </div>
                    <div class="space-y-4">
                        <div class="w-12 h-12 rounded-xl bg-gradient-to-br from-yellow-500 to-amber-600 flex items-center justify-center text-xl text-gray-950 shadow-lg shadow-yellow-500/20">
                            <i class="fa-solid fa-wifi group-hover:pulse-slow"></i>
                        </div>
                        <h3 class="text-xl font-bold text-white font-orbitron">Phòng Đấu Trực Tuyến</h3>
                        <p class="text-gray-400 text-sm leading-relaxed">
                            Giáo viên tạo phòng, nhận Mã Phòng (Code). Học sinh tham gia thi đấu từ các thiết bị riêng biệt. Đồng bộ điểm số, thứ hạng và trạng thái trận đấu tức thời.
                        </p>
                    </div>
                    <div class="mt-6 grid grid-cols-2 gap-3">
                        <button onclick="startOnlineSetupCreate()" class="py-3 bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-gray-950 font-bold rounded-xl transition duration-200">
                            Tạo Phòng Đấu
                        </button>
                        <button onclick="startOnlineSetupJoin()" class="py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded-xl transition duration-200 border border-gray-700">
                            Vào Phòng Chơi
                        </button>
                    </div>
                </div>
            </div>

            <!-- Teacher Generator and Resource Management Section -->
            <div class="bg-gray-900/40 border border-gray-800 rounded-2xl p-6 mt-4">
                <div class="flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
                    <div class="flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-full bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 flex items-center justify-center text-xl shrink-0">
                            <i class="fa-solid fa-wand-magic-sparkles"></i>
                        </div>
                        <div>
                            <h4 class="text-base font-bold text-white">Công cụ Giáo viên: AI Tạo Câu Hỏi Tự Động</h4>
                            <p class="text-sm text-gray-400">Tạo mới bộ câu hỏi trắc nghiệm năng lượng theo chủ đề mong muốn bằng trí tuệ nhân tạo Gemini.</p>
                        </div>
                    </div>
                    <button onclick="openAiGenerator()" class="w-full md:w-auto px-5 py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white font-semibold rounded-xl text-sm transition duration-200 shadow-md">
                        Tạo câu hỏi với AI
                    </button>
                </div>
            </div>
        </section>

        <!-- SCREEN: LOCAL SETUP -->
        <section id="screenLocalSetup" class="hidden max-w-md mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-6 space-y-6">
            <h3 class="text-xl font-bold font-orbitron text-white text-center">Cài đặt người chơi Offline</h3>
            
            <div class="space-y-4">
                <div>
                    <label class="block text-sm text-blue-400 font-semibold mb-2">Người chơi 1 (Phía Trái):</label>
                    <input id="localPlayer1Name" type="text" value="Chiến binh Lửa" class="w-full bg-gray-950 border border-gray-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-blue-500">
                </div>
                <div>
                    <label class="block text-sm text-yellow-400 font-semibold mb-2">Người chơi 2 (Phía Phải):</label>
                    <input id="localPlayer2Name" type="text" value="Chiến binh Sét" class="w-full bg-gray-950 border border-gray-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-yellow-500">
                </div>
            </div>

            <div class="bg-gray-950 p-4 rounded-xl border border-gray-800 space-y-2 text-xs text-gray-400">
                <p class="font-bold text-gray-300">⌨️ Cách thức thi đấu trên một thiết bị:</p>
                <p><b class="text-blue-400">Người chơi 1:</b> Sử dụng chuột click trực tiếp các lựa chọn bên trái của mình.</p>
                <p><b class="text-yellow-400">Người chơi 2:</b> Sử dụng chuột click các lựa chọn bên phải màn hình.</p>
            </div>

            <div class="flex gap-3">
                <button onclick="goToHome()" class="w-1/3 py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded-xl transition">Quay Lại</button>
                <button onclick="launchLocalGame()" class="w-2/3 py-3 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-500 hover:to-indigo-500 text-white font-bold rounded-xl transition shadow-lg">BẮT ĐẦU ĐẤU</button>
            </div>
        </section>

        <!-- SCREEN: ONLINE SETUP (CREATE ROOM) -->
        <section id="screenOnlineSetupCreate" class="hidden max-w-md mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-6 space-y-6 text-center">
            <h3 class="text-xl font-bold font-orbitron text-yellow-400">Tạo phòng thi đấu mới</h3>
            <p class="text-sm text-gray-400">Chia sẻ mã phòng dưới đây với các học sinh của bạn để tham gia so tài.</p>
            
            <div class="p-4 bg-gray-950 border border-gray-800 rounded-2xl">
                <span class="block text-xs uppercase tracking-wider text-gray-500 font-bold mb-1">Mã phòng thi đấu</span>
                <span id="createdRoomCode" class="text-4xl font-black font-orbitron tracking-widest text-white block">------</span>
            </div>

            <div class="space-y-3 text-left">
                <h4 class="text-xs font-bold text-gray-500 uppercase">Trạng thái kết nối</h4>
                <div class="space-y-2">
                    <div class="flex justify-between text-sm bg-gray-950 p-3 rounded-xl border border-gray-800">
                        <span class="text-blue-400 font-semibold"><i class="fa-solid fa-user"></i> Người chơi A:</span>
                        <span id="p1JoinStatus" class="text-gray-500 italic">Đang chờ...</span>
                    </div>
                    <div class="flex justify-between text-sm bg-gray-950 p-3 rounded-xl border border-gray-800">
                        <span class="text-yellow-400 font-semibold"><i class="fa-solid fa-user"></i> Người chơi B:</span>
                        <span id="p2JoinStatus" class="text-gray-500 italic">Đang chờ...</span>
                    </div>
                </div>
            </div>

            <div class="flex gap-3 pt-2">
                <button onclick="goToHome()" class="w-1/3 py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded-xl transition">Hủy</button>
                <button id="btnStartOnlineGame" disabled onclick="launchOnlineGameFromHost()" class="w-2/3 py-3 bg-gradient-to-r from-yellow-500 to-amber-600 text-gray-950 font-black rounded-xl transition shadow-lg disabled:opacity-50 disabled:cursor-not-allowed">
                    BẮT ĐẦU CHƠI
                </button>
            </div>
        </section>

        <!-- SCREEN: ONLINE SETUP (JOIN ROOM) -->
        <section id="screenOnlineSetupJoin" class="hidden max-w-md mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-6 space-y-6">
            <h3 class="text-xl font-bold font-orbitron text-white text-center">Tham gia phòng đấu</h3>
            
            <div class="space-y-4">
                <div>
                    <label class="block text-xs uppercase tracking-wider text-gray-500 font-bold mb-2">Mã Phòng gồm 4 chữ số:</label>
                    <input id="inputJoinCode" type="number" placeholder="Ví dụ: 8295" class="w-full bg-gray-950 border border-gray-800 rounded-xl px-4 py-3 text-center text-2xl font-black tracking-widest text-yellow-400 focus:outline-none focus:border-yellow-500">
                </div>
                <div>
                    <label class="block text-xs uppercase tracking-wider text-gray-500 font-bold mb-2">Tên của bạn:</label>
                    <input id="inputJoinName" type="text" placeholder="Nhập tên của bạn..." class="w-full bg-gray-950 border border-gray-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-yellow-500">
                </div>
            </div>

            <div class="flex gap-3">
                <button onclick="goToHome()" class="w-1/3 py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded-xl transition">Quay Lại</button>
                <button onclick="joinRoomAction()" class="w-2/3 py-3 bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-gray-950 font-bold rounded-xl transition shadow-lg">VÀO PHÒNG</button>
            </div>
        </section>

        <!-- SCREEN: ONLINE WAITING (CLIENTS) -->
        <section id="screenOnlineWaiting" class="hidden max-w-md mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-6 space-y-6 text-center">
            <div class="w-20 h-20 mx-auto rounded-full bg-yellow-500/10 flex items-center justify-center text-4xl text-yellow-400 animate-spin">
                <i class="fa-solid fa-circle-notch"></i>
            </div>
            <h3 class="text-xl font-bold font-orbitron text-white">Chờ giáo viên bắt đầu...</h3>
            <p class="text-gray-400 text-sm">Bạn đã tham gia phòng thành công. Hãy chuẩn bị tinh thần thi đấu đối kháng trực diện!</p>
            <div class="p-3 bg-gray-950 rounded-xl border border-gray-800">
                <p class="text-xs text-gray-500">Mã phòng của bạn</p>
                <p id="lblMyJoinedCode" class="text-xl font-bold font-orbitron text-yellow-400">- - - -</p>
            </div>
        </section>

        <!-- SCREEN: AI QUIZ GENERATOR TOOL -->
        <section id="screenAiGenerator" class="hidden max-w-2xl mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-6 space-y-6">
            <div class="flex items-center justify-between border-b border-gray-800 pb-4">
                <h3 class="text-xl font-bold font-orbitron text-white flex items-center gap-2">
                    <i class="fa-solid fa-wand-magic-sparkles text-emerald-400"></i> AI Thiết Kế Câu Hỏi
                </h3>
                <button onclick="closeAiGenerator()" class="text-gray-400 hover:text-white"><i class="fa-solid fa-xmark text-lg"></i></button>
            </div>

            <div class="space-y-4">
                <p class="text-sm text-gray-400 leading-relaxed">
                    Bạn muốn tùy chỉnh bộ đề thi đấu cho phù hợp với bài giảng trên lớp? Hãy mô tả chủ đề và AI Gemini sẽ tự động sinh ra bộ 10 câu hỏi chất lượng cao.
                </p>

                <div>
                    <label class="block text-xs uppercase tracking-wider text-gray-500 font-bold mb-2">Chủ đề chi tiết (Ví dụ: "Năng lượng gió và mặt trời ở Việt Nam", "Tiết kiệm năng lượng gia đình"):</label>
                    <textarea id="aiTopicInput" rows="3" class="w-full bg-gray-950 border border-gray-800 rounded-xl p-4 text-white focus:outline-none focus:border-emerald-500 text-sm" placeholder="Nhập chủ đề giáo dục năng lượng của bạn tại đây..."></textarea>
                </div>

                <div id="aiLoadingIndicator" class="hidden p-4 bg-emerald-900/10 border border-emerald-800/30 rounded-xl flex items-center gap-3">
                    <i class="fa-solid fa-circle-notch animate-spin text-emerald-400 text-xl"></i>
                    <p class="text-xs text-emerald-300">AI đang phân tích và thiết lập các câu hỏi tối ưu... (Thao tác này mất từ 3-5 giây)</p>
                </div>

                <!-- Custom Question Sets list -->
                <div class="space-y-2">
                    <h4 class="text-xs font-bold text-gray-500 uppercase">Danh sách bộ câu hỏi đang có:</h4>
                    <div class="max-h-36 overflow-y-auto space-y-1 bg-gray-950 p-2 rounded-xl border border-gray-800 text-xs text-gray-300" id="currentQuestionSetsList">
                        <!-- Dynamic content -->
                    </div>
                </div>
            </div>

            <div class="flex gap-3 justify-end pt-2 border-t border-gray-800">
                <button onclick="closeAiGenerator()" class="px-5 py-2.5 bg-gray-800 hover:bg-gray-700 text-white rounded-xl text-sm font-semibold transition">Quay Lại</button>
                <button onclick="generateCustomQuizWithAi()" class="px-5 py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white rounded-xl text-sm font-semibold transition flex items-center gap-1.5 shadow-md">
                    <i class="fa-solid fa-gears"></i> Tạo ngay
                </button>
            </div>
        </section>

        <!-- SCREEN: LOCAL SPLIT-SCREEN GAMEPLAY -->
        <section id="screenLocalGame" class="hidden flex-col gap-6 w-full animate-fade-in">
            <!-- Dynamic Battle Tug of War Visualization bar -->
            <div class="bg-gray-900 border border-gray-800 rounded-2xl p-6 shadow-xl relative overflow-hidden">
                <div class="absolute -top-10 left-1/2 -translate-x-1/2 w-40 h-40 bg-blue-500/10 rounded-full filter blur-2xl"></div>
                <div class="flex justify-between items-center mb-4">
                    <div class="text-left">
                        <span id="localP1Display" class="text-base md:text-lg font-black font-orbitron text-blue-400">P1. FIRE BATTLE</span>
                        <div class="text-xs text-gray-500">Nhấn lựa chọn phía TRÁI của bạn</div>
                    </div>
                    <!-- Central energy gauge indicator -->
                    <div class="text-center">
                        <span class="text-xs uppercase tracking-widest text-gray-500 font-bold block">Chênh lệch lực lượng</span>
                        <div id="localTugIndicator" class="text-2xl font-black font-orbitron text-yellow-400">CÂN BẰNG</div>
                    </div>
                    <div class="text-right">
                        <span id="localP2Display" class="text-base md:text-lg font-black font-orbitron text-red-400">P2. THUNDER BATTLE</span>
                        <div class="text-xs text-gray-500">Nhấn lựa chọn phía PHẢI của bạn</div>
                    </div>
                </div>

                <!-- Interactive Visual Tug of War Line -->
                <div class="relative h-8 bg-gray-950 rounded-full border border-gray-800 overflow-hidden flex items-center px-1">
                    <!-- Left Blue safe indicator -->
                    <div class="absolute left-0 top-0 bottom-0 w-5 bg-blue-500/40 animate-pulse"></div>
                    <!-- Right Red safe indicator -->
                    <div class="absolute right-0 top-0 bottom-0 w-5 bg-red-500/40 animate-pulse"></div>
                    
                    <!-- Sliding center laser energy beam representing balance -->
                    <div id="localTugBar" class="h-5 bg-gradient-to-r from-blue-500 via-yellow-400 to-red-500 rounded-full w-10 absolute left-1/2 -translate-x-1/2 transition-all duration-500 shadow-md">
                        <!-- Tiny pulsing energy core -->
                        <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-3 h-3 bg-white rounded-full pulse-slow shadow-lg"></div>
                    </div>
                    
                    <!-- Center Divider -->
                    <div class="absolute left-1/2 top-0 bottom-0 w-0.5 bg-gray-700/50"></div>
                </div>
            </div>

            <!-- Two Dual Screens (Split) layout for competitive interaction -->
            <div id="localSplitArena" class="grid grid-cols-1 md:grid-cols-2 gap-6 items-stretch">
                <!-- Player 1 Screen Panel (LEFT) -->
                <div id="localPanelP1" class="bg-gray-900 border-2 border-blue-500/30 rounded-2xl p-6 flex flex-col justify-between space-y-6 relative transition duration-300">
                    <div class="flex justify-between items-center border-b border-gray-800 pb-3">
                        <span class="text-xs uppercase tracking-widest font-black text-blue-400"><i class="fa-solid fa-user-shield"></i> PHÍA TRÁI</span>
                        <span id="localP1AnswerStatus" class="px-2.5 py-1 bg-gray-800 text-[10px] font-bold text-gray-400 rounded-full uppercase">ĐANG ĐỢI</span>
                    </div>

                    <div class="space-y-4 flex-grow">
                        <div class="min-h-[80px] bg-gray-950 p-4 rounded-xl border border-gray-800 flex items-center justify-center text-center">
                            <p id="localP1QuestionText" class="text-sm md:text-base font-semibold leading-relaxed text-white">Câu hỏi đang tải...</p>
                        </div>

                        <!-- Player 1 option lists -->
                        <div class="grid grid-cols-1 gap-2.5" id="localP1OptionsList">
                            <!-- Dynamic answers generated -->
                        </div>
                    </div>
                </div>

                <!-- Player 2 Screen Panel (RIGHT) -->
                <div id="localPanelP2" class="bg-gray-900 border-2 border-red-500/30 rounded-2xl p-6 flex flex-col justify-between space-y-6 relative transition duration-300">
                    <div class="flex justify-between items-center border-b border-gray-800 pb-3">
                        <span class="text-xs uppercase tracking-widest font-black text-red-400"><i class="fa-solid fa-user-shield"></i> PHÍA PHẢI</span>
                        <span id="localP2AnswerStatus" class="px-2.5 py-1 bg-gray-800 text-[10px] font-bold text-gray-400 rounded-full uppercase">ĐANG ĐỢI</span>
                    </div>

                    <div class="space-y-4 flex-grow">
                        <div class="min-h-[80px] bg-gray-950 p-4 rounded-xl border border-gray-800 flex items-center justify-center text-center">
                            <p id="localP2QuestionText" class="text-sm md:text-base font-semibold leading-relaxed text-white">Câu hỏi đang tải...</p>
                        </div>

                        <!-- Player 2 option lists -->
                        <div class="grid grid-cols-1 gap-2.5" id="localP2OptionsList">
                            <!-- Dynamic answers generated -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Match Options bottom -->
            <div class="flex justify-between items-center bg-gray-900 border border-gray-800 px-6 py-4 rounded-xl">
                <span id="localMatchRoundInfo" class="text-xs text-gray-400 font-bold uppercase tracking-widest">Tiến độ: Câu 1 / 10</span>
                <button onclick="goToHome()" class="px-4 py-2 bg-gray-800 hover:bg-gray-700 text-sm font-semibold rounded-lg text-white transition"><i class="fa-solid fa-arrow-right-from-bracket"></i> Thoát</button>
            </div>
        </section>

        <!-- SCREEN: ONLINE GAMEPLAY ARENA -->
        <section id="screenOnlineGame" class="hidden flex-col gap-6 w-full animate-fade-in">
            <!-- Dynamic Battle Tug of War Visualization bar for Online game -->
            <div class="bg-gray-900 border border-gray-800 rounded-2xl p-6 shadow-xl relative overflow-hidden">
                <div class="absolute -top-10 left-1/2 -translate-x-1/2 w-40 h-40 bg-amber-500/10 rounded-full filter blur-2xl"></div>
                
                <div class="flex justify-between items-center mb-4">
                    <div class="text-left">
                        <span id="onlineP1Display" class="text-base md:text-lg font-black font-orbitron text-blue-400">P1. NGƯỜI CHƠI A</span>
                        <div id="onlineP1Score" class="text-xs text-blue-300">Trả lời đúng: 0</div>
                    </div>
                    <!-- Central status -->
                    <div class="text-center">
                        <span class="text-xs uppercase tracking-widest text-gray-500 font-bold block">TÌNH TRẠNG TRANH CHẤP</span>
                        <div id="onlineTugIndicator" class="text-2xl font-black font-orbitron text-yellow-400">CÂN BẰNG</div>
                    </div>
                    <div class="text-right">
                        <span id="onlineP2Display" class="text-base md:text-lg font-black font-orbitron text-red-400">P2. NGƯỜI CHƠI B</span>
                        <div id="onlineP2Score" class="text-xs text-red-300">Trả lời đúng: 0</div>
                    </div>
                </div>

                <!-- Laser Tug of War sync line -->
                <div class="relative h-8 bg-gray-950 rounded-full border border-gray-800 overflow-hidden flex items-center px-1">
                    <div class="absolute left-0 top-0 bottom-0 w-5 bg-blue-500/40 animate-pulse"></div>
                    <div class="absolute right-0 top-0 bottom-0 w-5 bg-red-500/40 animate-pulse"></div>
                    
                    <div id="onlineTugBar" class="h-5 bg-gradient-to-r from-blue-500 via-yellow-400 to-red-500 rounded-full w-10 absolute left-1/2 -translate-x-1/2 transition-all duration-500 shadow-md">
                        <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-3 h-3 bg-white rounded-full pulse-slow shadow-lg"></div>
                    </div>
                    
                    <div class="absolute left-1/2 top-0 bottom-0 w-0.5 bg-gray-700/50"></div>
                </div>
            </div>

            <!-- Single Player Client panel for active player answers (online) -->
            <div id="onlineActionPanel" class="max-w-xl mx-auto w-full bg-gray-900 border-2 border-yellow-500/30 rounded-2xl p-6 space-y-6">
                <div class="flex justify-between items-center border-b border-gray-800 pb-3">
                    <span id="onlineMyIdentityHeader" class="text-xs uppercase tracking-widest font-black text-yellow-400"><i class="fa-solid fa-gamepad"></i> VÙNG TRẢ LỜI CỦA BẠN</span>
                    <span id="onlineMyAnswerStatus" class="px-2.5 py-1 bg-yellow-500/20 text-[10px] font-bold text-yellow-400 rounded-full uppercase">SẴN SÀNG</span>
                </div>

                <div class="space-y-4">
                    <div class="min-h-[100px] bg-gray-950 p-4 rounded-xl border border-gray-800 flex items-center justify-center text-center">
                        <p id="onlineQuestionText" class="text-base font-semibold leading-relaxed text-white">Câu hỏi đang đồng bộ từ máy chủ...</p>
                    </div>

                    <!-- Options -->
                    <div class="grid grid-cols-1 gap-2.5" id="onlineOptionsList">
                        <!-- Dynamic generated -->
                    </div>
                </div>
            </div>

            <div class="flex justify-between items-center max-w-xl mx-auto w-full bg-gray-900 border border-gray-800 px-6 py-4 rounded-xl">
                <span id="onlineMatchRoundInfo" class="text-xs text-gray-400 font-bold uppercase tracking-widest">Tiến độ: Đang chờ máy chủ...</span>
                <button onclick="goToHome()" class="px-4 py-2 bg-gray-800 hover:bg-gray-700 text-sm font-semibold rounded-lg text-white transition">Thoát</button>
            </div>
        </section>

        <!-- SCREEN: WINNER REVEAL -->
        <section id="screenWinner" class="hidden max-w-lg mx-auto w-full bg-gray-900 border border-gray-800 rounded-2xl p-8 space-y-6 text-center relative overflow-hidden">
            <div class="absolute -top-10 left-1/2 -translate-x-1/2 w-64 h-64 bg-yellow-500/10 rounded-full filter blur-3xl"></div>
            
            <div class="mx-auto w-24 h-24 bg-gradient-to-tr from-yellow-500 to-amber-600 rounded-full flex items-center justify-center text-5xl text-gray-950 shadow-lg shadow-yellow-500/20 animate-bounce">
                <i class="fa-solid fa-trophy"></i>
            </div>

            <div class="space-y-2">
                <span class="px-3 py-1 bg-yellow-500/10 border border-yellow-500/30 rounded-full text-[10px] font-bold uppercase tracking-widest text-yellow-400">KẾT QUẢ CUỐI CÙNG</span>
                <h3 id="lblWinnerTitle" class="text-2xl md:text-3xl font-black font-orbitron text-white">CHIẾN BINH THẮNG TRẬN!</h3>
                <p id="lblWinnerSub" class="text-gray-400 text-sm max-w-sm mx-auto">Chúc mừng bạn đã giành thắng lợi tuyệt đối trong cuộc đua kéo co năng lượng tri thức!</p>
            </div>

            <!-- Score breakdown overview -->
            <div class="p-4 bg-gray-950 rounded-xl border border-gray-800 space-y-2 text-sm text-left">
                <div class="flex justify-between text-blue-400">
                    <span>Người chơi A:</span>
                    <span id="lblBreakdownP1" class="font-bold">0 câu đúng</span>
                </div>
                <div class="flex justify-between text-red-400">
                    <span>Người chơi B:</span>
                    <span id="lblBreakdownP2" class="font-bold">0 câu đúng</span>
                </div>
            </div>

            <button onclick="goToHome()" class="w-full py-3.5 bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-gray-950 font-extrabold rounded-xl shadow-lg transition duration-200 uppercase tracking-wider text-sm">Quay Lại Trang Chủ</button>
        </section>

    </main>

    <!-- Footer Area -->
    <footer class="p-4 bg-gray-950 border-t border-gray-900 text-center text-xs text-gray-600">
        <div class="max-w-7xl mx-auto flex flex-col md:flex-row justify-between items-center gap-2">
            <p>&copy; 2026 Energy Battle. Thiết kế thông tin năng lượng THCS.</p>
            <p>Xây dựng dựa trên các tiêu chí tương tác nâng cao.</p>
        </div>
    </footer>

    <!-- MAIN INTERACTIVE APP LOGIC & ENGINE SCRIPTS -->
    <script>
        // Core default 10 integrated energy questions
        const DEFAULT_QUESTIONS = [
            {
                q: "Năng lượng trong thức ăn, than đá, dầu mỏ, khí đốt tự nhiên bắt nguồn từ đâu?",
                options: ["Gió", "Điện", "Nước chảy", "Mặt trời"],
                correct: 3
            },
            {
                q: "Người dân thường sử dụng năng lượng mặt trời vào việc nào?",
                options: ["Làm quay cọn nước", "Phơi khô thóc, sấy chuối", "Giã gạo", "Chạy thuyền buồm"],
                correct: 1
            },
            {
                q: "Ưu điểm của sấy chuối bằng năng lượng mặt trời?",
                options: ["Tốn chi phí hơn", "Tiết kiệm và bảo vệ môi trường", "Gây ô nhiễm hơn", "Nhanh hơn bình thường"],
                correct: 1
            },
            {
                q: "Hoạt động sử dụng năng lượng gió?",
                options: ["Làm nóng nước", "Chạy thủy điện", "Rê thóc", "Vận chuyển gỗ"],
                correct: 2
            },
            {
                q: "Vì sao gió giúp loại bỏ thóc lép?",
                options: ["Làm thóc biến mất", "Làm thóc nặng hơn", "Thổi bay thóc lép nhẹ hơn", "Cuốn cả hai loại"],
                correct: 2
            },
            {
                q: "Năng lượng nước chảy dùng để làm gì ở miền núi?",
                options: ["Phơi đồ", "Giám sát", "Sản xuất muối", "Quay cọn nước"],
                correct: 3
            },
            {
                q: "Thuyền buồm đi ngược gió thường làm gì?",
                options: ["Đổ nước", "Căng buồm", "Thả neo", "Hạ buồm"],
                correct: 3
            },
            {
                q: "Nguồn năng lượng giúp bè gỗ trôi sông?",
                options: ["Mặt trời", "Than đá", "Điện", "Nước chảy"],
                correct: 3
            },
            {
                q: "Điện từ năng lượng mặt trời có ưu điểm gì?",
                options: ["Không ô nhiễm, vô tận", "Nhanh cạn kiệt", "Độc hại hơn", "Dễ cháy nổ"],
                correct: 0
            },
            {
                q: "Tỉnh nào có điện gió lớn ở Việt Nam?",
                options: ["Lạng Sơn", "Bạc Liêu", "Điện Biên", "Hà Nội"],
                correct: 1
            }
        ];

        // Active active questions database
        let activeQuestions = [...DEFAULT_QUESTIONS];
        
        // Local game state variables
        let localState = {
            active: false,
            p1Name: "Người chơi A",
            p2Name: "Người chơi B",
            currentQIdx: 0,
            p1Score: 0,
            p2Score: 0,
            p1Answered: null, // index selected
            p2Answered: null, // index selected
            tugValue: 0, // -5 (P1 fully winning) to +5 (P2 fully winning)
        };

        // Firebase / Online state variables
        let onlineState = {
            active: false,
            role: null, // 'host' or 'p1' or 'p2'
            roomCode: null,
            roomDocId: null,
            p1Name: "",
            p2Name: "",
            myId: "",
            joinedAs: null, // 'player1' or 'player2'
            unsubscribeRoom: null,
            lastSyncedQIdx: -1
        };

        // Custom storage of question sets
        let questionSets = [
            { id: "default", name: "Bộ đề mặc định (10 câu Năng lượng THCS)", data: DEFAULT_QUESTIONS }
        ];

        window.addEventListener('load', () => {
            // Check if Firebase was successfully connected and try to authenticate
            const authStatusDiv = document.getElementById('authStatusIndicator');
            if (window.db && window.auth) {
                authStatusDiv.className = "text-xs bg-emerald-900/40 text-emerald-400 px-3 py-1.5 rounded-lg border border-emerald-800 flex items-center gap-1.5";
                authStatusDiv.innerHTML = `<span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span> Trực tuyến (Sẵn sàng Multiplayer)`;
                
                // Trigger anonymous sign in to ensure we have credentials for firestore
                window.signInAnonymously(window.auth).catch(err => {
                    console.error("Sign in anonymously failed:", err);
                });
            } else {
                authStatusDiv.className = "text-xs bg-gray-800 text-gray-400 px-3 py-1.5 rounded-lg border border-gray-700 flex items-center gap-1.5";
                authStatusDiv.innerHTML = `<span class="w-2 h-2 rounded-full bg-gray-500"></span> Ngoại tuyến (Local Match Only)`;
            }
            renderQuestionSetsList();
        });

        // SCREEN NAVIGATIONS
        function hideAllScreens() {
            document.getElementById('screenHome').classList.add('hidden');
            document.getElementById('screenLocalSetup').classList.add('hidden');
            document.getElementById('screenOnlineSetupCreate').classList.add('hidden');
            document.getElementById('screenOnlineSetupJoin').classList.add('hidden');
            document.getElementById('screenOnlineWaiting').classList.add('hidden');
            document.getElementById('screenAiGenerator').classList.add('hidden');
            document.getElementById('screenLocalGame').classList.add('hidden');
            document.getElementById('screenOnlineGame').classList.add('hidden');
            document.getElementById('screenWinner').classList.add('hidden');
        }

        function goToHome() {
            hideAllScreens();
            document.getElementById('screenHome').classList.remove('hidden');
            // Cleanup any active listeners
            if (onlineState.unsubscribeRoom) {
                onlineState.unsubscribeRoom();
                onlineState.unsubscribeRoom = null;
            }
            localState.active = false;
            onlineState.active = false;
        }

        // LOCAL MULTIPLAYER LAUNCH
        function startLocalSetup() {
            hideAllScreens();
            document.getElementById('screenLocalSetup').classList.remove('hidden');
        }

        function launchLocalGame() {
            localState.p1Name = document.getElementById('localPlayer1Name').value || "Người chơi A";
            localState.p2Name = document.getElementById('localPlayer2Name').value || "Người chơi B";
            localState.active = true;
            localState.currentQIdx = 0;
            localState.p1Score = 0;
            localState.p2Score = 0;
            localState.p1Answered = null;
            localState.p2Answered = null;
            localState.tugValue = 0;

            document.getElementById('localP1Display').innerText = localState.p1Name.toUpperCase();
            document.getElementById('localP2Display').innerText = localState.p2Name.toUpperCase();

            hideAllScreens();
            document.getElementById('screenLocalGame').classList.remove('hidden');
            
            sound.init();
            renderLocalQuestion();
            updateLocalTugUI();
        }

        // LOCAL QUESTION RENDER & LOGIC
        function renderLocalQuestion() {
            const q = activeQuestions[localState.currentQIdx];
            document.getElementById('localMatchRoundInfo').innerText = `TIẾN ĐỘ: CÂU ${localState.currentQIdx + 1} / ${activeQuestions.length}`;
            
            // P1 Section render
            document.getElementById('localP1QuestionText').innerText = q.q;
            document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-blue-500/20 text-[10px] font-bold text-blue-400 rounded-full uppercase";
            document.getElementById('localP1AnswerStatus').innerText = "ĐANG SUY NGHĨ";
            
            let p1OptionsHTML = '';
            q.options.forEach((opt, idx) => {
                p1OptionsHTML += `
                    <button onclick="localSelectAnswer(1, ${idx})" id="local_p1_opt_${idx}" class="w-full text-left p-3.5 bg-gray-950 hover:bg-gray-800 border border-gray-800 hover:border-blue-500/50 rounded-xl text-sm transition text-gray-200">
                        <span class="inline-block w-6 h-6 rounded bg-blue-500/20 text-blue-400 font-bold text-center leading-6 text-xs mr-3">${String.fromCharCode(65 + idx)}</span>
                        ${opt}
                    </button>
                `;
            });
            document.getElementById('localP1OptionsList').innerHTML = p1OptionsHTML;

            // P2 Section render
            document.getElementById('localP2QuestionText').innerText = q.q;
            document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-red-500/20 text-[10px] font-bold text-red-400 rounded-full uppercase";
            document.getElementById('localP2AnswerStatus').innerText = "ĐANG SUY NGHĨ";
            
            let p2OptionsHTML = '';
            q.options.forEach((opt, idx) => {
                p2OptionsHTML += `
                    <button onclick="localSelectAnswer(2, ${idx})" id="local_p2_opt_${idx}" class="w-full text-left p-3.5 bg-gray-950 hover:bg-gray-800 border border-gray-800 hover:border-red-500/50 rounded-xl text-sm transition text-gray-200">
                        <span class="inline-block w-6 h-6 rounded bg-red-500/20 text-red-400 font-bold text-center leading-6 text-xs mr-3">${String.fromCharCode(65 + idx)}</span>
                        ${opt}
                    </button>
                `;
            });
            document.getElementById('localP2OptionsList').innerHTML = p2OptionsHTML;

            // Reset selected states
            localState.p1Answered = null;
            localState.p2Answered = null;

            // Restore styling opacity
            document.getElementById('localPanelP1').classList.remove('opacity-50');
            document.getElementById('localPanelP2').classList.remove('opacity-50');
        }

        function localSelectAnswer(player, index) {
            const q = activeQuestions[localState.currentQIdx];
            if (player === 1) {
                if (localState.p1Answered !== null) return; // already answered
                localState.p1Answered = index;
                sound.playTug();
                document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-blue-500 text-[10px] font-bold text-white rounded-full uppercase animate-pulse";
                document.getElementById('localP1AnswerStatus').innerText = "ĐÃ CHỌN";
                
                // highlight option selected
                q.options.forEach((_, idx) => {
                    const btn = document.getElementById(`local_p1_opt_${idx}`);
                    if (idx === index) {
                        btn.className = "w-full text-left p-3.5 bg-blue-950/40 border-2 border-blue-500 rounded-xl text-sm transition text-white font-bold";
                    } else {
                        btn.className = "w-full text-left p-3.5 bg-gray-950 border border-gray-900 rounded-xl text-sm transition text-gray-600 cursor-not-allowed";
                        btn.disabled = true;
                    }
                });
            } else {
                if (localState.p2Answered !== null) return; // already answered
                localState.p2Answered = index;
                sound.playTug();
                document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-red-500 text-[10px] font-bold text-white rounded-full uppercase animate-pulse";
                document.getElementById('localP2AnswerStatus').innerText = "ĐÃ CHỌN";
                
                // highlight option selected
                q.options.forEach((_, idx) => {
                    const btn = document.getElementById(`local_p2_opt_${idx}`);
                    if (idx === index) {
                        btn.className = "w-full text-left p-3.5 bg-red-950/40 border-2 border-red-500 rounded-xl text-sm transition text-white font-bold";
                    } else {
                        btn.className = "w-full text-left p-3.5 bg-gray-950 border border-gray-900 rounded-xl text-sm transition text-gray-600 cursor-not-allowed";
                        btn.disabled = true;
                    }
                });
            }

            // Check if both have responded
            if (localState.p1Answered !== null && localState.p2Answered !== null) {
                setTimeout(revealLocalAnswers, 1200);
            }
        }

        function revealLocalAnswers() {
            const q = activeQuestions[localState.currentQIdx];
            const correctIdx = q.correct;
            
            const p1Correct = (localState.p1Answered === correctIdx);
            const p2Correct = (localState.p2Answered === correctIdx);

            // Play response chime sounds
            if (p1Correct || p2Correct) {
                sound.playCorrect();
            } else {
                sound.playIncorrect();
            }

            // Highlight green for actual correct option, red for incorrect chosen
            q.options.forEach((_, idx) => {
                const btn1 = document.getElementById(`local_p1_opt_${idx}`);
                const btn2 = document.getElementById(`local_p2_opt_${idx}`);

                if (idx === correctIdx) {
                    btn1.className = "w-full text-left p-3.5 bg-emerald-950/50 border-2 border-emerald-500 text-emerald-300 rounded-xl text-sm font-black";
                    btn2.className = "w-full text-left p-3.5 bg-emerald-950/50 border-2 border-emerald-500 text-emerald-300 rounded-xl text-sm font-black";
                } else {
                    if (idx === localState.p1Answered) {
                        btn1.className = "w-full text-left p-3.5 bg-red-950/50 border-2 border-red-500 text-red-400 rounded-xl text-sm line-through";
                    }
                    if (idx === localState.p2Answered) {
                        btn2.className = "w-full text-left p-3.5 bg-red-950/50 border-2 border-red-500 text-red-400 rounded-xl text-sm line-through";
                    }
                }
            });

            // Calculate Tug of War movements
            // Player 1 Correct moves slider LEFT (-1)
            // Player 2 Correct moves slider RIGHT (+1)
            if (p1Correct && !p2Correct) {
                localState.tugValue -= 1;
                localState.p1Score += 1;
                document.getElementById('localPanelP1').classList.add('glow-blue');
                document.getElementById('localP1AnswerStatus').innerText = "CHÍNH XÁC +1";
                document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-emerald-500 text-[10px] font-bold text-white rounded-full uppercase";
                document.getElementById('localP2AnswerStatus').innerText = "SAI RỒI";
                document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-red-900 text-[10px] font-bold text-red-400 rounded-full uppercase";
            } else if (!p1Correct && p2Correct) {
                localState.tugValue += 1;
                localState.p2Score += 1;
                document.getElementById('localPanelP2').classList.add('glow-green');
                document.getElementById('localP2AnswerStatus').innerText = "CHÍNH XÁC +1";
                document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-emerald-500 text-[10px] font-bold text-white rounded-full uppercase";
                document.getElementById('localP1AnswerStatus').innerText = "SAI RỒI";
                document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-red-900 text-[10px] font-bold text-red-400 rounded-full uppercase";
            } else if (p1Correct && p2Correct) {
                localState.p1Score += 1;
                localState.p2Score += 1;
                document.getElementById('localP1AnswerStatus').innerText = "CHÍNH XÁC";
                document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-emerald-500 text-[10px] font-bold text-white rounded-full uppercase";
                document.getElementById('localP2AnswerStatus').innerText = "CHÍNH XÁC";
                document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-emerald-500 text-[10px] font-bold text-white rounded-full uppercase";
            } else {
                document.getElementById('localP1AnswerStatus').innerText = "CẢ HAI ĐỀU SAI";
                document.getElementById('localP1AnswerStatus').className = "px-2.5 py-1 bg-red-900 text-[10px] font-bold text-red-400 rounded-full uppercase";
                document.getElementById('localP2AnswerStatus').innerText = "CẢ HAI ĐỀU SAI";
                document.getElementById('localP2AnswerStatus').className = "px-2.5 py-1 bg-red-900 text-[10px] font-bold text-red-400 rounded-full uppercase";
            }

            updateLocalTugUI();

            // Check extreme win conditions (-5 or +5)
            if (localState.tugValue <= -5) {
                setTimeout(() => finishLocalGame(1), 2000);
            } else if (localState.tugValue >= 5) {
                setTimeout(() => finishLocalGame(2), 2000);
            } else {
                // Next question logic
                localState.currentQIdx++;
                if (localState.currentQIdx >= activeQuestions.length) {
                    // Out of questions, evaluate winner by tug value placement
                    setTimeout(() => {
                        if (localState.tugValue < 0) {
                            finishLocalGame(1);
                        } else if (localState.tugValue > 0) {
                            finishLocalGame(2);
                        } else {
                            finishLocalGame(0); // Draw
                        }
                    }, 2000);
                } else {
                    setTimeout(() => {
                        document.getElementById('localPanelP1').classList.remove('glow-blue');
                        document.getElementById('localPanelP2').classList.remove('glow-green');
                        renderLocalQuestion();
                    }, 3000);
                }
            }
        }

        function updateLocalTugUI() {
            const bar = document.getElementById('localTugBar');
            const ind = document.getElementById('localTugIndicator');
            
            // Value ranges -5 to 5. Compute percentage
            // -5 is leftmost (0%), 0 is center (50%), +5 is rightmost (100%)
            const percent = 50 + (localState.tugValue * 10);
            bar.style.left = `calc(${percent}% - 20px)`;

            // text alert indicator update
            if (localState.tugValue < 0) {
                ind.innerText = `🔥 LỢI THẾ ${localState.p1Name.toUpperCase()} (+${Math.abs(localState.tugValue)})`;
                ind.className = "text-xl font-black font-orbitron text-blue-400";
            } else if (localState.tugValue > 0) {
                ind.innerText = `⚡ LỢI THẾ ${localState.p2Name.toUpperCase()} (+${localState.tugValue})`;
                ind.className = "text-xl font-black font-orbitron text-red-400";
            } else {
                ind.innerText = "CÂN BẰNG";
                ind.className = "text-xl font-black font-orbitron text-yellow-400";
            }
        }

        function finishLocalGame(winnerCode) {
            hideAllScreens();
            document.getElementById('screenWinner').classList.remove('hidden');
            sound.playWin();

            const title = document.getElementById('lblWinnerTitle');
            const sub = document.getElementById('lblWinnerSub');
            
            if (winnerCode === 1) {
                title.innerText = `🏆 ${localState.p1Name.toUpperCase()} CHIẾN THẮNG!`;
                sub.innerText = `${localState.p1Name} đã giành thắng lợi bằng cách áp đảo hoàn toàn năng lượng đối thủ!`;
            } else if (winnerCode === 2) {
                title.innerText = `🏆 ${localState.p2Name.toUpperCase()} CHIẾN THẮNG!`;
                sub.innerText = `${localState.p2Name} đã chứng tỏ kiến thức xuất sắc và đẩy lùi hoàn toàn đối phương!`;
            } else {
                title.innerText = "🥊 KẾT QUẢ HÒA!";
                sub.innerText = "Trận đấu cân sức không phân thắng bại! Cả hai kỳ phùng địch thủ đều có nguồn năng lượng tương đồng.";
            }

            document.getElementById('lblBreakdownP1').innerText = `${localState.p1Score} / ${activeQuestions.length} câu đúng`;
            document.getElementById('lblBreakdownP2').innerText = `${localState.p2Score} / ${activeQuestions.length} câu đúng`;
        }

        // ONLINE MULTIPLAYER ENGINE AND LOGIC (FIRESTORE Sync based on guidelines)
        async function startOnlineSetupCreate() {
            if (!window.db || !window.auth) {
                showModal("Lỗi kết nối", "Hệ thống cơ sở dữ liệu trực tuyến không khả dụng hoặc bị chặn. Hãy trải nghiệm chế độ offline cực mượt!", "fa-circle-exclamation");
                return;
            }

            // Ensure we are signed in first
            if (!window.auth.currentUser) {
                try {
                    await window.signInAnonymously(window.auth);
                } catch (e) {
                    showModal("Lỗi xác thực", "Không thể thiết lập quyền truy cập mạng. Vui lòng kiểm tra lại.", "fa-triangle-exclamation");
                    return;
                }
            }

            hideAllScreens();
            document.getElementById('screenOnlineSetupCreate').classList.remove('hidden');

            // Generate unique 4 digit room code
            const roomCode = Math.floor(1000 + Math.random() * 9000).toString();
            document.getElementById('createdRoomCode').innerText = roomCode;

            onlineState.active = true;
            onlineState.role = 'host';
            onlineState.roomCode = roomCode;
            
            // Sync status variables to defaults
            document.getElementById('p1JoinStatus').innerText = "Đang đợi...";
            document.getElementById('p1JoinStatus').className = "text-gray-500 italic";
            document.getElementById('p2JoinStatus').innerText = "Đang đợi...";
            document.getElementById('p2JoinStatus').className = "text-gray-500 italic";
            document.getElementById('btnStartOnlineGame').disabled = true;

            // Strict Guideline Path Rule 1: /artifacts/${appId}/public/data/rooms
            try {
                const roomDocRef = doc(window.db, 'artifacts', window.appId, 'public', 'data', 'rooms', roomCode);
                await setDoc(roomDocRef, {
                    code: roomCode,
                    status: 'waiting',
                    player1: { id: '', name: '', score: 0, answered: -1 },
                    player2: { id: '', name: '', score: 0, answered: -1 },
                    tugValue: 0,
                    currentQIdx: 0,
                    questions: activeQuestions,
                    createdAt: Date.now()
                });

                // Listen to room document changes
                onlineState.unsubscribeRoom = onSnapshot(roomDocRef, (snapshot) => {
                    if (snapshot.exists()) {
                        const data = snapshot.data();
                        handleHostRoomSync(data);
                    }
                }, (error) => {
                    console.error("Room listener error: ", error);
                });

            } catch (err) {
                console.error("Error creating Firestore room:", err);
                showModal("Không thể tạo phòng", "Lỗi dịch vụ Firebase Cloud. Vui lòng thử lại.", "fa-circle-xmark");
                goToHome();
            }
        }

        function startOnlineSetupJoin() {
            if (!window.db || !window.auth) {
                showModal("Lỗi kết nối", "Ứng dụng đang ngoại tuyến. Hãy sử dụng chế độ Chơi Offline 1v1 tiện lợi!", "fa-circle-exclamation");
                return;
            }
            hideAllScreens();
            document.getElementById('screenOnlineSetupJoin').classList.remove('hidden');
        }

        async function joinRoomAction() {
            const joinCode = document.getElementById('inputJoinCode').value.trim();
            const joinName = document.getElementById('inputJoinName').value.trim();

            if (!joinCode || !joinName) {
                showModal("Thiếu thông tin", "Vui lòng nhập đầy đủ Mã Phòng và Tên người chơi để tham chiến.", "fa-circle-info");
                return;
            }

            // Authentication state check
            if (!window.auth.currentUser) {
                try {
                    await window.signInAnonymously(window.auth);
                } catch (e) {
                    showModal("Lỗi mạng", "Kết nối xác thực thất bại. Vui lòng thử lại.", "fa-wifi");
                    return;
                }
            }

            const myId = window.auth.currentUser.uid;

            try {
                // Strict Guideline Path Rule 1
                const roomDocRef = doc(window.db, 'artifacts', window.appId, 'public', 'data', 'rooms', joinCode);
                const roomSnap = await getDoc(roomDocRef);

                if (!roomSnap.exists()) {
                    showModal("Sai mã phòng", "Phòng đấu không tồn tại hoặc đã hết hạn.", "fa-ban");
                    return;
                }

                const roomData = roomSnap.data();

                if (roomData.status !== 'waiting') {
                    showModal("Không thể tham gia", "Phòng đấu này đã bắt đầu hoặc kết thúc.", "fa-circle-xmark");
                    return;
                }

                let joinedAs = null;
                let updatedPlayer1 = { ...roomData.player1 };
                let updatedPlayer2 = { ...roomData.player2 };

                // Assign to free slots
                if (!roomData.player1.id) {
                    joinedAs = 'player1';
                    updatedPlayer1 = { id: myId, name: joinName, score: 0, answered: -1 };
                } else if (!roomData.player2.id) {
                    joinedAs = 'player2';
                    updatedPlayer2 = { id: myId, name: joinName, score: 0, answered: -1 };
                } else {
                    showModal("Phòng đã đầy", "Phòng đã có đủ 2 đối thủ tham gia đấu trí.", "fa-users");
                    return;
                }

                await updateDoc(roomDocRef, {
                    player1: updatedPlayer1,
                    player2: updatedPlayer2
                });

                // Establish local client values
                onlineState.active = true;
                onlineState.role = joinedAs === 'player1' ? 'p1' : 'p2';
                onlineState.roomCode = joinCode;
                onlineState.joinedAs = joinedAs;
                onlineState.myId = myId;

                hideAllScreens();
                document.getElementById('screenOnlineWaiting').classList.remove('hidden');
                document.getElementById('lblMyJoinedCode').innerText = joinCode;

                // Sync snapshot as player client
                onlineState.unsubscribeRoom = onSnapshot(roomDocRef, (snapshot) => {
                    if (snapshot.exists()) {
                        const data = snapshot.data();
                        handleClientRoomSync(data);
                    }
                }, (error) => {
                    console.error("Client listener error: ", error);
                });

            } catch (err) {
                console.error("Join room error:", err);
                showModal("Lỗi kết nối", "Xử lý tham gia gặp sự cố. Thử lại sau ít phút.", "fa-circle-exclamation");
            }
        }

        // SYNC HANDLER: Host view
        function handleHostRoomSync(roomData) {
            // Check connected clients
            const p1Connected = !!roomData.player1.id;
            const p2Connected = !!roomData.player2.id;

            if (p1Connected) {
                document.getElementById('p1JoinStatus').innerText = `SẴN SÀNG: ${roomData.player1.name}`;
                document.getElementById('p1JoinStatus').className = "text-blue-400 font-bold";
            }
            if (p2Connected) {
                document.getElementById('p2JoinStatus').innerText = `SẴN SÀNG: ${roomData.player2.name}`;
                document.getElementById('p2JoinStatus').className = "text-yellow-400 font-bold";
            }

            // Enable start button if both are present
            if (p1Connected && p2Connected) {
                document.getElementById('btnStartOnlineGame').disabled = false;
            }

            // If game is playing and this is host, sync active questions and show scoreboard
            if (roomData.status === 'playing') {
                // If it transitioned to playing, launch host tracking
                if (!onlineState.activeGameLaunched) {
                    onlineState.activeGameLaunched = true;
                    launchOnlineGameArena(roomData);
                }
                
                // Track mutual answers
                checkAndProcessOnlineAnswers(roomData);
            }
        }

        // SYNC HANDLER: Client view
        function handleClientRoomSync(roomData) {
            if (roomData.status === 'playing') {
                if (document.getElementById('screenOnlineWaiting').classList.contains('hidden') === false) {
                    hideAllScreens();
                    document.getElementById('screenOnlineGame').classList.remove('hidden');
                    
                    document.getElementById('onlineP1Display').innerText = roomData.player1.name.toUpperCase();
                    document.getElementById('onlineP2Display').innerText = roomData.player2.name.toUpperCase();
                }

                // If new question, render it
                if (roomData.currentQIdx !== onlineState.lastSyncedQIdx) {
                    onlineState.lastSyncedQIdx = roomData.currentQIdx;
                    renderOnlineQuestionForClient(roomData);
                }

                // Update Tug representation
                updateOnlineTugUI(roomData.tugValue, roomData.player1.name, roomData.player2.name);
                
                document.getElementById('onlineP1Score').innerText = `Trả lời đúng: ${roomData.player1.score}`;
                document.getElementById('onlineP2Score').innerText = `Trả lời đúng: ${roomData.player2.score}`;
            }

            if (roomData.status === 'ended') {
                finishOnlineGameResult(roomData);
            }
        }

        function launchOnlineGameFromHost() {
            // Update room state to 'playing'
            const roomDocRef = doc(window.db, 'artifacts', window.appId, 'public', 'data', 'rooms', onlineState.roomCode);
            updateDoc(roomDocRef, {
                status: 'playing',
                currentQIdx: 0,
                tugValue: 0
            });
        }

        function launchOnlineGameArena(roomData) {
            hideAllScreens();
            document.getElementById('screenOnlineGame').classList.remove('hidden');
            
            document.getElementById('onlineP1Display').innerText = roomData.player1.name.toUpperCase();
            document.getElementById('onlineP2Display').innerText = roomData.player2.name.toUpperCase();
            
            updateOnlineTugUI(roomData.tugValue, roomData.player1.name, roomData.player2.name);
        }

        function renderOnlineQuestionForClient(roomData) {
            const qIdx = roomData.currentQIdx;
            const qList = roomData.questions || activeQuestions;
            
            if (qIdx >= qList.length) return; // safety
            
            const q = qList[qIdx];
            document.getElementById('onlineMatchRoundInfo').innerText = `TIẾN ĐỘ: CÂU ${qIdx + 1} / ${qList.length}`;
            document.getElementById('onlineQuestionText').innerText = q.q;

            // Reset My status
            document.getElementById('onlineMyAnswerStatus').innerText = "ĐANG SUY NGHĨ";
            document.getElementById('onlineMyAnswerStatus').className = "px-2.5 py-1 bg-yellow-500/20 text-[10px] font-bold text-yellow-400 rounded-full uppercase";

            let optionsHTML = '';
            q.options.forEach((opt, idx) => {
                optionsHTML += `
                    <button onclick="onlineSubmitAnswer(${idx}, ${qIdx})" id="online_opt_${idx}" class="w-full text-left p-3.5 bg-gray-950 hover:bg-gray-800 border border-gray-800 hover:border-yellow-500/50 rounded-xl text-sm transition text-gray-200">
                        <span class="inline-block w-6 h-6 rounded bg-yellow-500/20 text-yellow-400 font-bold text-center leading-6 text-xs mr-3">${String.fromCharCode(65 + idx)}</span>
                        ${opt}
                    </button>
                `;
            });
            document.getElementById('onlineOptionsList').innerHTML = optionsHTML;
        }

        async function onlineSubmitAnswer(choiceIdx, qIdx) {
            // Update my status in firestore
            const roomDocRef = doc(window.db, 'artifacts', window.appId, 'public', 'data', 'rooms', onlineState.roomCode);
            
            sound.playTug();

            // Disable all options immediately to prevent dual click
            const options = document.getElementById('onlineOptionsList').getElementsByTagName('button');
            for (let i = 0; i < options.length; i++) {
                options[i].disabled = true;
                if (i === choiceIdx) {
                    options[i].className = "w-full text-left p-3.5 bg-yellow-950/40 border-2 border-yellow-500 rounded-xl text-sm transition text-white font-bold";
                } else {
                    options[i].className = "w-full text-left p-3.5 bg-gray-950 border border-gray-900 rounded-xl text-sm transition text-gray-600";
                }
            }

            document.getElementById('onlineMyAnswerStatus').innerText = "ĐÃ ĐĂNG KÝ PHƯƠNG ÁN";
            document.getElementById('onlineMyAnswerStatus').className = "px-2.5 py-1 bg-yellow-500 text-[10px] font-bold text-white rounded-full uppercase animate-pulse";

            const updatePayload = {};
            if (onlineState.joinedAs === 'player1') {
                updatePayload['player1.answered'] = choiceIdx;
            } else {
                updatePayload['player2.answered'] = choiceIdx;
            }

            try {
                await updateDoc(roomDocRef, updatePayload);
            } catch (e) {
                console.error("Answer upload failed", e);
            }
        }

        // Evaluation logic (called by Host's synchronization loop)
        async function checkAndProcessOnlineAnswers(roomData) {
            const p1Answered = roomData.player1.answered;
            const p2Answered = roomData.player2.answered;

            // Proceed if both players locked in their selections
            if (p1Answered !== -1 && p2Answered !== -1) {
                const qIdx = roomData.currentQIdx;
                const qList = roomData.questions;
                const correctIdx = qList[qIdx].correct;

                const p1Correct = (p1Answered === correctIdx);
                const p2Correct = (p2Answered === correctIdx);

                let updatedP1Score = roomData.player1.score;
                let updatedP2Score = roomData.player2.score;
                let updatedTug = roomData.tugValue;

                if (p1Correct) updatedP1Score++;
                if (p2Correct) updatedP2Score++;

                if (p1Correct && !p2Correct) {
                    updatedTug -= 1; // Pull left
                } else if (!p1Correct && p2Correct) {
                    updatedTug += 1; // Pull right
                }

                // Check finish conditions
                let nextStatus = 'playing';
                let nextQIdx = qIdx + 1;

                if (updatedTug <= -5 || updatedTug >= 5 || nextQIdx >= qList.length) {
                    nextStatus = 'ended';
                }

                // Reset answer variables for next round
                const roomDocRef = doc(window.db, 'artifacts', window.appId, 'public', 'data', 'rooms', onlineState.roomCode);
                
                // Wait briefly before publishing result to allow user visual comfort
                setTimeout(async () => {
                    await updateDoc(roomDocRef, {
                        status: nextStatus,
                        currentQIdx: nextQIdx,
                        tugValue: updatedTug,
                        'player1.score': updatedP1Score,
                        'player2.score': updatedP2Score,
                        'player1.answered': -1,
                        'player2.answered': -1
                    });
                }, 2000);
            }
        }

        function updateOnlineTugUI(tugValue, p1Name, p2Name) {
            const bar = document.getElementById('onlineTugBar');
            const ind = document.getElementById('onlineTugIndicator');
            
            const percent = 50 + (tugValue * 10);
            bar.style.left = `calc(${percent}% - 20px)`;

            if (tugValue < 0) {
                ind.innerText = `🔥 LỢI THẾ ${p1Name.toUpperCase()} (+${Math.abs(tugValue)})`;
                ind.className = "text-xl font-orbitron font-black text-blue-400";
            } else if (tugValue > 0) {
                ind.innerText = `⚡ LỢI THẾ ${p2Name.toUpperCase()} (+${tugValue})`;
                ind.className = "text-xl font-orbitron font-black text-red-400";
            } else {
                ind.innerText = "CÂN BẰNG";
                ind.className = "text-xl font-orbitron font-black text-yellow-400";
            }
        }

        function finishOnlineGameResult(roomData) {
            hideAllScreens();
            document.getElementById('screenWinner').classList.remove('hidden');
            sound.playWin();

            const title = document.getElementById('lblWinnerTitle');
            const sub = document.getElementById('lblWinnerSub');
            
            const p1Name = roomData.player1.name;
            const p2Name = roomData.player2.name;

            if (roomData.tugValue < 0) {
                title.innerText = `🏆 ${p1Name.toUpperCase()} CHIẾN THẮNG!`;
                sub.innerText = `${p1Name} đã áp đảo hoàn toàn bằng kiến thức và kéo phao năng lượng về sân nhà!`;
            } else if (roomData.tugValue > 0) {
                title.innerText = `🏆 ${p2Name.toUpperCase()} CHIẾN THẮNG!`;
                sub.innerText = `${p2Name} đã thể hiện tốc độ và sự sắc bén đẩy lùi đối phương ngoạn mục!`;
            } else {
                title.innerText = "🥊 TRANH TÀI BẤT PHÂN THẮNG BẠI!";
                sub.innerText = "Hai bên quá xuất sắc, cân bằng thế trận đến những câu hỏi cuối cùng.";
            }

            document.getElementById('lblBreakdownP1').innerText = `${p1Name}: ${roomData.player1.score} câu đúng`;
            document.getElementById('lblBreakdownP2').innerText = `${p2Name}: ${roomData.player2.score} câu đúng`;

            if (onlineState.unsubscribeRoom) {
                onlineState.unsubscribeRoom();
                onlineState.unsubscribeRoom = null;
            }
        }

        // AI CUSTOM QUIZ GENERATOR FUNCTION (Using Google Gemini API directly)
        function openAiGenerator() {
            hideAllScreens();
            document.getElementById('screenAiGenerator').classList.remove('hidden');
        }

        function closeAiGenerator() {
            goToHome();
        }

        function renderQuestionSetsList() {
            const listContainer = document.getElementById('currentQuestionSetsList');
            let listHTML = '';
            questionSets.forEach((set, index) => {
                listHTML += `
                    <div class="flex justify-between items-center p-2.5 bg-gray-900 border border-gray-800 rounded-lg hover:border-emerald-500/30 transition">
                        <div>
                            <span class="font-semibold text-white">${set.name}</span>
                            <span class="block text-[10px] text-gray-500">Mã định danh: ${set.id} | ${set.data.length} câu</span>
                        </div>
                        <button onclick="selectQuestionSet(${index})" class="px-2.5 py-1 bg-emerald-500/10 hover:bg-emerald-500 text-emerald-400 hover:text-white border border-emerald-500/30 text-[10px] font-bold rounded transition">
                            Kích hoạt
                        </button>
                    </div>
                `;
            });
            listContainer.innerHTML = listHTML;
        }

        function selectQuestionSet(index) {
            const selectedSet = questionSets[index];
            activeQuestions = [...selectedSet.data];
            showModal("Kích hoạt đề mới", `Đã thiết lập sử dụng: "${selectedSet.name}" thành công!`, "fa-circle-check");
        }

        async function generateCustomQuizWithAi() {
            const topic = document.getElementById('aiTopicInput').value.trim();
            if (!topic) {
                showModal("Thiếu thông tin", "Hãy nhập chủ đề bạn muốn để AI có định hướng tạo câu hỏi.", "fa-keyboard");
                return;
            }

            document.getElementById('aiLoadingIndicator').classList.remove('hidden');

            const systemPrompt = `Bạn là một chuyên gia thiết kế trò chơi giáo dục và giáo viên THCS. 
Hãy thiết kế đúng 10 câu hỏi trắc nghiệm chủ đề học tập về tài nguyên năng lượng dựa theo chủ đề của người dùng.
Mỗi câu hỏi phải có cấu trúc chính xác sau đây dưới dạng JSON:
{
  "q": "Câu hỏi tiếng Việt?",
  "options": ["Phương án A", "Phương án B", "Phương án C", "Phương án D"],
  "correct": 0, 1, 2 hoặc 3 (chỉ số của đáp án đúng nhất)
}

Yêu cầu xuất ra định dạng JSON thuần túy (Mảng 10 phần tử), không chứa bất cứ văn bản markdown hay chú giải bên ngoài nào cả.`;

            const userQuery = `Hãy tạo 10 câu hỏi trắc nghiệm tiếng Việt liên quan đến chủ đề năng lượng: "${topic}".`;
            const apiKey = ""; // Leave blank for dynamic insertion by system runtime
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "ARRAY",
                        items: {
                            type: "OBJECT",
                            properties: {
                                "q": { "type": "STRING" },
                                "options": {
                                    "type": "ARRAY",
                                    "items": { "type": "STRING" }
                                },
                                "correct": { "type": "INTEGER" }
                            },
                            "required": ["q", "options", "correct"]
                        }
                    }
                },
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                }
            };

            // Implement exponential backoff for resilience
            let response = null;
            let delay = 1000;
            for (let attempt = 0; attempt < 3; attempt++) {
                try {
                    response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    if (response.ok) break;
                } catch (e) {
                    await new Promise(res => setTimeout(res, delay));
                    delay *= 2;
                }
            }

            document.getElementById('aiLoadingIndicator').classList.add('hidden');

            if (response && response.ok) {
                try {
                    const result = await response.json();
                    const jsonText = result.candidates[0].content.parts[0].text;
                    const parsedQuiz = JSON.parse(jsonText);

                    if (Array.isArray(parsedQuiz) && parsedQuiz.length > 0) {
                        const newId = "custom-" + Date.now();
                        const newName = `Bộ câu hỏi AI: ${topic.substring(0, 30)}...`;
                        
                        questionSets.push({
                            id: newId,
                            name: newName,
                            data: parsedQuiz
                        });

                        // Set as currently active quiz database
                        activeQuestions = [...parsedQuiz];
                        
                        renderQuestionSetsList();
                        showModal("Thành công!", `Gemini đã soạn thảo thành công 10 câu hỏi cho chủ đề "${topic}". Đã tự động kích hoạt bộ đề này!`, "fa-wand-magic-sparkles");
                    } else {
                        showModal("Lỗi phân tích", "Định dạng câu hỏi do AI tạo ra chưa đồng bộ. Vui lòng thử lại.", "fa-triangle-exclamation");
                    }
                } catch (e) {
                    console.error("Parse generated quiz error:", e);
                    showModal("Phân tích thất bại", "Cấu trúc phản hồi không hợp lệ. Vui lòng thử lại.", "fa-circle-exclamation");
                }
            } else {
                showModal("Không kết nối được AI", "Không nhận được phản hồi từ hệ thống AI Gemini. Hãy kiểm tra kết nối mạng của bạn.", "fa-triangle-exclamation");
            }
        }

        // DIALOG HELPERS
        function showModal(title, body, iconClass = "fa-circle-info") {
            document.getElementById('modalTitle').innerText = title;
            document.getElementById('modalBody').innerText = body;
            document.getElementById('modalIcon').className = `fa-solid ${iconClass}`;
            document.getElementById('customModal').classList.remove('hidden');
        }

        function closeModal() {
            document.getElementById('customModal').classList.add('hidden');
        }

        function showInfoModal() {
            showModal(
                "Energy Battle là gì?",
                "Đây là trò chơi tương tác đối kháng 1v1 về năng lượng. " +
                "Mỗi câu trả lời đúng của bạn sẽ kéo/đẩy dây chuyền năng lượng về phía bạn. " +
                "Đẩy hoàn toàn đối phương ra khỏi vạch giới hạn (đạt mức chênh lệch 5 điểm) để chiến thắng tuyệt đối. " +
                "Bạn có thể đấu split-screen offline trên 1 máy, hoặc thiết lập phòng đấu mạng cloud 1v1 đồng bộ real-time cực vui!",
                "fa-circle-info"
            );
        }
    </script>
</body>
</html>
