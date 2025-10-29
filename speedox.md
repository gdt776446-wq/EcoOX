# Speed OX
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>생태천 OX 스피드 퀴즈</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script> <!-- Tone.js 추가 --><link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <style>
        /* 물결 효과를 위한 배경 스타일 */
        #water-background {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #a0d8ef; /* 기본 물 색상 */
            z-index: -1; 
        }

        /* 물결 흐름 효과 (반복되는 선형 그라데이션 애니메이션) */
        #water-background::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 200%;
            height: 200%;
            background: repeating-linear-gradient(
                -45deg,
                rgba(255, 255, 255, 0.2) 0%,
                rgba(255, 255, 255, 0.2) 5%,
                transparent 5%,
                transparent 10%
            );
            animation: flow 40s linear infinite;
            opacity: 0.7;
        }

        @keyframes flow {
            0% { transform: translate(0, 0); }
            100% { transform: translate(-50%, -50%); }
        }

        /* 물고기 스타일 및 애니메이션 */
        .fish {
            position: absolute;
            font-size: 3rem; 
            text-shadow: 0 0 5px rgba(255, 255, 255, 0.5);
            animation-iteration-count: infinite;
            animation-timing-function: linear;
            cursor: default;
            pointer-events: none;
        }

        /* 물고기 개별 애니메이션 정의 */
        @keyframes swim1 {
            0% { left: 100%; top: 20%; transform: scaleX(1); }
            49% { left: -10%; top: 40%; transform: scaleX(1); }
            50% { left: -10%; top: 40%; transform: scaleX(-1); }
            99% { left: 100%; top: 80%; transform: scaleX(-1); }
            100% { left: 100%; top: 20%; transform: scaleX(1); }
        }
        @keyframes swim2 {
            0% { left: 10%; top: 90%; transform: scaleX(-1); }
            49% { left: 90%; top: 10%; transform: scaleX(-1); }
            50% { left: 90%; top: 10%; transform: scaleX(1); }
            99% { left: 10%; top: 50%; transform: scaleX(1); }
            100% { left: 10%; top: 90%; transform: scaleX(-1); }
        }
        @keyframes swim3 {
            0% { left: 50%; top: 0%; transform: scaleX(1); }
            25% { left: 80%; top: 30%; transform: scaleX(1); }
            50% { left: 20%; top: 70%; transform: scaleX(-1); }
            75% { left: 70%; top: 90%; transform: scaleX(-1); }
            100% { left: 50%; top: 0%; transform: scaleX(1); }
        }
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: transparent;
        }
        .card {
            background-color: rgba(255, 255, 255, 0.92);
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 10px 20px -3px rgba(0, 0, 0, 0.15);
            transition: all 0.3s ease-in-out;
            width: 100%;
            max-width: 42rem; 
            position: relative;
            z-index: 10;
            backdrop-filter: blur(2px);
        }
        .btn-choice {
            transition: all 0.2s ease-in-out;
            position: relative; /* 이펙트 생성을 위해 필요 */
            overflow: hidden; /* 이펙트가 버튼 밖으로 나가지 않게 */
        }
        .btn-choice:hover {
            transform: scale(1.05);
            box-shadow: 0 0 15px rgba(0, 200, 255, 0.6); /* 호버 시 빛나는 효과 */
        }
        /* 정답 버튼 클릭 시 확장 애니메이션 */
        .btn-choice.correct-flash {
            animation: correctBtnFlash 0.3s ease-out forwards;
        }
        @keyframes correctBtnFlash {
            0% { transform: scale(1); box-shadow: 0 0 15px rgba(0, 255, 0, 0.6); }
            50% { transform: scale(1.08); box-shadow: 0 0 30px rgba(0, 255, 0, 1); }
            100% { transform: scale(1); box-shadow: 0 0 15px rgba(0, 200, 255, 0.6); }
        }

        .rank-table th, .rank-table td {
            padding: 0.75rem;
            text-align: center;
        }
        .rank-table tr:nth-child(even) {
            background-color: #f9fafb;
        }
        .modal {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        /* 이펙트 컨테이너 - 모든 동적 이펙트가 여기에 생성됨 */
        #effect-container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none; /* 클릭 방지 */
            z-index: 999; /* 퀴즈 콘텐츠 위에 표시 */
            overflow: hidden; /* 이펙트가 화면 밖으로 넘치지 않도록 */
        }

        /* 정답 시 반짝이는 입자 */
        .sparkle {
            position: absolute;
            background-color: white;
            border-radius: 50%;
            animation: sparkle-anim 0.6s forwards;
            opacity: 0;
        }

        @keyframes sparkle-anim {
            0% { transform: translate(-50%, -50%) scale(0); opacity: 1; }
            50% { transform: translate(-50%, -50%) scale(1); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(0); opacity: 0; }
        }

        /* 오답 시 연기 효과 */
        .smoke {
            position: absolute;
            background-color: rgba(100, 100, 100, 0.7); /* 더 진하게 */
            border-radius: 50%;
            animation: smoke-anim 0.8s forwards;
            filter: blur(8px); /* 더 흐릿하게 */
            opacity: 0;
        }

        @keyframes smoke-anim {
            0% { transform: translate(-50%, -50%) scale(0); opacity: 0.7; }
            50% { transform: translate(-50%, -50%) scale(2); opacity: 0.4; } /* 더 크게 퍼짐 */
            100% { transform: translate(-50%, -50%) scale(3); opacity: 0; } /* 최종 크기 증가 */
        }

        /* 전체 화면 이펙트 - 물결/파동 (정답) */
        .ripple {
            position: absolute;
            background-color: rgba(0, 255, 0, 0.4); /* 더 진하게 */
            border-radius: 50%;
            animation: ripple-anim 0.7s ease-out forwards; /* 애니메이션 시간 증가 */
            transform: translate(-50%, -50%) scale(0);
            opacity: 1;
            z-index: -1; /* 다른 이펙트 아래에 깔리도록 */
        }

        @keyframes ripple-anim {
            0% { transform: translate(-50%, -50%) scale(0); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(6); opacity: 0; } /* 훨씬 더 크게 퍼짐 */
        }

        /* 전체 화면 이펙트 - 깨지는 효과 (오답) */
        #shatter-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 998; /* 이펙트 컨테이너 아래, 퀴즈 콘텐츠 위에 */
            opacity: 0;
            background-color: transparent;
            transition: opacity 0.1s linear; /* 투명도 전환 */
        }

        .shatter-image {
            position: absolute;
            width: 100%;
            height: 100%;
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path fill="none" stroke="red" stroke-width="0.5" d="M50 0 L55 10 L50 20 L45 10 Z M50 0 L40 10 L50 20 L60 10 Z M50 50 L50 0 M50 50 L0 50 M50 50 L100 50 M50 50 L50 100 M50 50 L15 15 M50 50 L85 15 M50 50 L15 85 M50 50 L85 85" /></svg>'); /* 깨진 유리 SVG (간단화) */
            background-size: cover;
            opacity: 0.6;
            animation: shatter-image-anim 0.4s ease-out forwards;
            filter: blur(1px); /* 약간의 블러 */
        }
        @keyframes shatter-image-anim {
            0% { transform: scale(0.8) rotate(0deg); opacity: 0; }
            50% { transform: scale(1.05) rotate(2deg); opacity: 0.8; }
            100% { transform: scale(1) rotate(0deg); opacity: 0; }
        }

        /* 화면 왜곡 효과 (keyframes로 적용) */
        @keyframes distort-screen {
            0% { filter: contrast(1) hue-rotate(0deg); transform: perspective(1000px) rotateX(0deg); }
            20% { filter: contrast(1.2) hue-rotate(10deg); transform: perspective(1000px) rotateX(2deg); }
            40% { filter: contrast(1) hue-rotate(0deg); transform: perspective(1000px) rotateX(0deg); }
            60% { filter: contrast(1.2) hue-rotate(-10deg); transform: perspective(1000px) rotateX(-2deg); }
            100% { filter: contrast(1) hue-rotate(0deg); transform: perspective(1000px) rotateX(0deg); }
        }
        .distort-active {
            animation: distort-screen 0.4s ease-out forwards;
        }

        /* 게임 종료 폭죽 효과 */
        .firework {
            position: absolute;
            background-color: transparent;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            animation: firework-anim 1s ease-out forwards;
            opacity: 0;
            box-shadow: 0 0 5px 2px var(--firework-color);
        }

        @keyframes firework-anim {
            0% { transform: translate(-50%, -50%) scale(0); opacity: 1; background-color: var(--firework-color); }
            20% { opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(2); opacity: 0; background-color: transparent; box-shadow: none; }
        }

        /* 빛나는 카드 효과 */
        .card:hover {
            box-shadow: 0 10px 30px -3px rgba(0, 200, 255, 0.4), 0 4px 10px -2px rgba(0, 200, 255, 0.2);
        }
        
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <!-- 이펙트가 동적으로 생성될 컨테이너 --><div id="effect-container"></div>
    <!-- 오답 시 화면 전체 깨짐 효과를 위한 오버레이 --><div id="shatter-overlay"></div>

    <!-- 물 배경 및 물고기 애니메이션 --><div id="water-background">
        <!-- 물고기 이모지: 🐠 (열대어), 🐡 (복어), 🐟 (일반 물고기) --><div class="fish" style="animation: swim1 25s -5s infinite linear;">🐠</div>
        <div class="fish" style="animation: swim2 30s -10s infinite linear; font-size: 2rem;">🐟</div>
        <div class="fish" style="animation: swim3 45s -20s infinite linear; color: #ff6600;">🐡</div>
        <div class="fish" style="animation: swim1 20s -15s infinite linear; font-size: 4rem; color: #ffffff;">🐟</div>
    </div>
    <!-- // 물 배경 및 물고기 애니메이션 --><div id="app-container" class="container mx-auto flex justify-center">
        
        <!-- 시작 화면 --><div id="start-screen" class="card text-center">
            <h1 class="text-4xl font-bold text-green-700 mb-2">생태천 OX 스피드 퀴즈</h1>
            <p class="text-gray-600 mb-6">생태천에 대해 얼마나 알고 계신가요? 당신의 지식과 순발력을 테스트해보세요!</p>
            <div class="space-y-4">
                <input type="text" id="userId" placeholder="아이디 (예: 홍길동)" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500">
                <input type="tel" id="userPhone" placeholder="전화번호 (예: 010-1234-5678)" class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500">
                <button id="start-btn" class="w-full bg-green-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-green-700 transition-colors duration-300 text-lg">퀴즈 시작!</button>
            </div>
             <p id="start-error" class="text-red-500 mt-4 text-sm h-5"></p>
             <div class="mt-6 text-right">
                <button id="admin-mode-btn" class="text-sm text-gray-500 hover:text-gray-700 underline">관리자 모드</button>
             </div>
        </div>

        <!-- 퀴즈 화면 --><div id="quiz-screen" class="card hidden">
            <div class="flex justify-between items-center mb-6">
                <div id="question-counter" class="text-xl font-bold text-gray-700">1 / 10</div>
                <div id="timer" class="text-2xl font-bold text-green-600">0.00초</div>
            </div>
            <div class="bg-gray-100 p-6 rounded-lg min-h-[150px] flex items-center justify-center mb-6">
                <p id="question-text" class="text-2xl font-bold text-center text-gray-800"></p>
            </div>
            <div class="grid grid-cols-2 gap-4">
                <button data-answer="true" id="o-btn" class="btn-choice text-7xl font-bold bg-blue-500 text-white rounded-lg py-10 hover:bg-blue-600 shadow-xl">O</button>
                <button data-answer="false" id="x-btn" class="btn-choice text-7xl font-bold bg-red-500 text-white rounded-lg py-10 hover:bg-red-600 shadow-xl">X</button>
            </div>
        </div>

        <!-- 결과 화면 --><div id="result-screen" class="card text-center hidden">
            <h2 class="text-3xl font-bold text-green-700 mb-4">퀴즈 완료!</h2>
            <div class="bg-gray-100 p-6 rounded-lg mb-6 space-y-2">
                <p class="text-lg text-gray-600">정답 개수: <span id="score" class="font-bold text-2xl text-blue-600"></span></p>
                <p class="text-lg text-gray-600">소요 시간: <span id="time-taken" class="font-bold text-2xl text-red-600"></span></p>
            </div>

            <h3 class="text-2xl font-bold text-gray-800 mb-4">명예의 전당 (TOP 10)</h3>
            <div class="overflow-x-auto">
                <table id="rank-table" class="rank-table w-full text-sm text-left text-gray-500 rounded-lg">
                    <thead class="text-xs text-gray-700 uppercase bg-gray-200">
                        <tr>
                            <th scope="col">순위</th>
                            <th scope="col">아이디</th>
                            <th scope="col">정답</th>
                            <th scope="col">기록(초)</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- 순위가 여기에 동적으로 추가됩니다 --></tbody>
                </table>
            </div>

            <button id="restart-btn" class="mt-8 w-full bg-green-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-green-700 transition-colors duration-300 text-lg">처음으로</button>
        </div>

        <!-- 관리자 화면 --><div id="admin-screen" class="card text-center hidden">
            <h2 class="text-3xl font-bold text-blue-700 mb-4">관리자 페이지 - 전체 기록</h2>
            <div class="overflow-x-auto">
                <table id="admin-rank-table" class="rank-table w-full text-sm text-left text-gray-500 rounded-lg">
                    <thead class="text-xs text-gray-700 uppercase bg-blue-100">
                        <tr>
                            <th scope="col">순위</th>
                            <th scope="col">아이디</th>
                            <th scope="col">전화번호</th>
                            <th scope="col">정답</th>
                            <th scope="col">기록(초)</th>
                        </tr>
                    </thead>
                    <tbody id="admin-rank-body">
                        <!-- 관리자용 순위가 여기에 동적으로 추가됩니다 --></tbody>
                </table>
            </div>
            <button id="admin-back-btn" class="mt-8 w-full bg-gray-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-gray-700">처음으로</button>
        </div>
    </div>

    <!-- 관리자 로그인 모달 --><div id="admin-login-modal" class="modal hidden">
        <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-sm text-center">
            <h3 class="text-xl font-bold mb-4">관리자 로그인</h3>
            <input type="password" id="admin-password" placeholder="비밀번호" class="w-full p-3 border border-gray-300 rounded-lg mb-2">
            <p id="admin-error" class="text-red-500 text-sm h-5 mb-2"></p>
            <div class="flex gap-2">
                <button id="admin-login-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700">로그인</button>
                <button id="admin-close-btn" class="w-full bg-gray-300 text-gray-800 font-bold py-2 rounded-lg hover:bg-gray-400">닫기</button>
            </div>
        </div>
    </div>

    <script>
        const quizData = [
            { question: "생태하천은 오염된 물을 스스로 정화하는 능력이 있다.", answer: true },
            { question: "모든 하천의 물고기는 1급수에서만 살 수 있다.", answer: false },
            { question: "수질 정화를 위해 하천 바닥을 모두 콘크리트로 덮는 것이 좋다.", answer: false },
            { question: "버드나무와 갈대는 수질 정화에 도움을 주는 대표적인 수생식물이다.", answer: true },
            { question: "생태하천 복원 사업은 동식물의 서식지를 파괴하는 주된 원인이다.", answer: false },
            { question: "빗물이 하천으로 바로 유입되면 수질 오염의 원인이 될 수 있다.", answer: true },
            { question: "하천에 사는 잠자리의 애벌레는 물 밖 풀숲에서 생활한다.", answer: false },
            { question: "생활하수를 정화 없이 하천으로 흘려보내는 것은 생태계를 풍요롭게 한다.", answer: false },
            { question: "생태하천의 돌과 자갈은 미생물이 살 공간을 제공하여 물을 깨끗하게 한다.", answer: true },
            { question: "생태하천에서는 어떠한 경우에도 낚시나 물놀이가 금지되어 있다.", answer: false }
        ];

        // 화면 요소
        const startScreen = document.getElementById('start-screen');
        const quizScreen = document.getElementById('quiz-screen');
        const resultScreen = document.getElementById('result-screen');
        const adminScreen = document.getElementById('admin-screen');
        const adminLoginModal = document.getElementById('admin-login-modal');
        const effectContainer = document.getElementById('effect-container'); // 이펙트 컨테이너
        const shatterOverlay = document.getElementById('shatter-overlay'); // 깨짐 오버레이
        
        // 버튼
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const choiceBtns = document.querySelectorAll('.btn-choice');
        const oBtn = document.getElementById('o-btn');
        const xBtn = document.getElementById('x-btn');
        const adminModeBtn = document.getElementById('admin-mode-btn');
        const adminLoginBtn = document.getElementById('admin-login-btn');
        const adminCloseBtn = document.getElementById('admin-close-btn');
        const adminBackBtn = document.getElementById('admin-back-btn');
        
        // 입력 및 표시 요소
        const userIdInput = document.getElementById('userId');
        const userPhoneInput = document.getElementById('userPhone');
        const startError = document.getElementById('start-error');
        const questionCounter = document.getElementById('question-counter');
        const timerDisplay = document.getElementById('timer');
        const questionText = document.getElementById('question-text');
        const scoreDisplay = document.getElementById('score');
        const timeTakenDisplay = document.getElementById('time-taken');
        const rankTableBody = document.querySelector('#rank-table tbody');
        const adminPasswordInput = document.getElementById('admin-password');
        const adminError = document.getElementById('admin-error');
        const adminRankBody = document.getElementById('admin-rank-body');

        let currentQuestionIndex = 0;
        let score = 0;
        let timerInterval;
        let startTime;

        // Tone.js 사운드 정의
        let correctSound;
        let incorrectSound;
        let gameEndSound;

        // 사운드 초기화 (사용자 상호작용 후 호출되어야 오디오 컨텍스트가 활성화됨)
        function initSounds() {
            correctSound = new Tone.PluckSynth().toDestination();
            incorrectSound = new Tone.MembraneSynth({
                pitchDecay: 0.05,
                octaves: 1,
                oscillator: { type: "square" }
            }).toDestination();
            gameEndSound = new Tone.Synth({
                oscillator: { type: "triangle" },
                envelope: {
                    attack: 0.05,
                    decay: 0.2,
                    sustain: 0.1,
                    release: 0.3
                }
            }).toDestination();
        }

        // --- 이펙트 함수들 ---

        // 버튼 주변에 스파클 이펙트 생성 (정답)
        function createSparkleEffect(x, y) {
            for (let i = 0; i < 20; i++) { // 개수 증가
                const sparkle = document.createElement('div');
                sparkle.className = 'sparkle';
                const size = Math.random() * 12 + 4; // 4px to 16px, 크기 증가
                sparkle.style.width = `${size}px`;
                sparkle.style.height = `${size}px`;
                sparkle.style.left = `${x + (Math.random() - 0.5) * 80}px`; // 더 넓게 퍼짐
                sparkle.style.top = `${y + (Math.random() - 0.5) * 80}px`; // 더 넓게 퍼짐
                effectContainer.appendChild(sparkle);
                sparkle.addEventListener('animationend', () => sparkle.remove());
            }
        }

        // 버튼 주변에 연기 이펙트 생성 (오답)
        function createSmokeEffect(x, y) {
            for (let i = 0; i < 8; i++) { // 개수 증가
                const smoke = document.createElement('div');
                smoke.className = 'smoke';
                const size = Math.random() * 40 + 20; // 20px to 60px, 크기 증가
                smoke.style.width = `${size}px`;
                smoke.style.height = `${size}px`;
                smoke.style.left = `${x + (Math.random() - 0.5) * 60}px`; // 더 넓게 퍼짐
                smoke.style.top = `${y + (Math.random() - 0.5) * 60}px`; // 더 넓게 퍼짐
                effectContainer.appendChild(smoke);
                smoke.addEventListener('animationend', () => smoke.remove());
            }
        }

        // 전체 화면 리플/파동 이펙트 (정답)
        function createRippleEffect(color) {
            const ripple = document.createElement('div');
            ripple.className = 'ripple';
            ripple.style.backgroundColor = color;
            ripple.style.left = `50%`;
            ripple.style.top = `50%`;
            effectContainer.appendChild(ripple);
            ripple.addEventListener('animationend', () => ripple.remove());
        }

        // 전체 화면 깨지는 효과 (오답)
        function createShatterScreenEffect() {
            shatterOverlay.innerHTML = ''; // 기존 깨짐 효과 초기화
            shatterOverlay.style.opacity = 1;
            shatterOverlay.style.backgroundColor = 'rgba(255, 0, 0, 0.2)'; // 붉은 플래시

            const shatterImage = document.createElement('div');
            shatterImage.className = 'shatter-image';
            shatterOverlay.appendChild(shatterImage);

            // 화면 왜곡 효과 활성화
            document.body.classList.add('distort-active');

            setTimeout(() => {
                shatterOverlay.style.opacity = 0;
                document.body.classList.remove('distort-active');
            }, 500); // 0.5초 후 초기화
        }

        // 게임 종료 폭죽 효과
        function createFireworksEffect() {
            const colors = ['#ff0000', '#00ff00', '#0000ff', '#ffff00', '#ff00ff', '#00ffff'];
            for (let i = 0; i < 50; i++) { // 폭죽 개수 증가
                const firework = document.createElement('div');
                firework.className = 'firework';
                const color = colors[Math.floor(Math.random() * colors.length)];
                firework.style.setProperty('--firework-color', color);
                firework.style.left = `${Math.random() * 100}%`;
                firework.style.top = `${Math.random() * 100}%`;
                
                const size = Math.random() * 20 + 10; // 크기 증가
                firework.style.width = `${size}px`;
                firework.style.height = `${size}px`;

                firework.style.animationDelay = `${Math.random() * 0.8}s`; // 딜레이 범위 증가

                effectContainer.appendChild(firework);
                firework.addEventListener('animationend', () => firework.remove());
            }
        }


        // 게임 시작
        function startGame() {
            if (!correctSound) {
                Tone.start();
                initSounds();
            }

            const userId = userIdInput.value.trim();
            const userPhone = userPhoneInput.value.trim();
            const phoneRegex = /^\d{2,4}-\d{3,4}-\d{4}$/; 

            if (!userId || !userPhone) {
                startError.textContent = '아이디와 전화번호를 모두 입력해주세요.';
                return;
            }
            if (!phoneRegex.test(userPhone)) {
                startError.textContent = '올바른 전화번호 형식(예: 010-1234-5678)을 입력해주세요.';
                return;
            }
            
            startError.textContent = '';
            currentQuestionIndex = 0;
            score = 0;

            startScreen.classList.add('hidden');
            quizScreen.classList.remove('hidden');
            resultScreen.classList.add('hidden');
            adminScreen.classList.add('hidden');

            displayQuestion();
            startTimer();
        }

        // 문제 표시
        function displayQuestion() {
            if (currentQuestionIndex < quizData.length) {
                const currentQuestion = quizData[currentQuestionIndex];
                questionText.textContent = currentQuestion.question;
                questionCounter.textContent = `${currentQuestionIndex + 1} / ${quizData.length}`;
            } else {
                endGame();
            }
        }

        // 답변 선택
        function selectAnswer(e) {
            const selectedAnswer = e.target.dataset.answer === 'true';
            const correctAnswer = quizData[currentQuestionIndex].answer;
            const now = Tone.now();

            const rect = e.target.getBoundingClientRect();
            const centerX = rect.left + rect.width / 2;
            const centerY = rect.top + rect.height / 2;

            if (selectedAnswer === correctAnswer) {
                score++;
                createSparkleEffect(centerX, centerY);
                createRippleEffect('rgba(0, 255, 0, 0.4)'); // 더 진한 초록색 파동
                e.target.classList.add('correct-flash'); // 버튼 자체 애니메이션
                e.target.addEventListener('animationend', () => {
                    e.target.classList.remove('correct-flash');
                }, { once: true });
                correctSound.triggerAttackRelease("C5", "8n", now); 
            } else {
                createSmokeEffect(centerX, centerY);
                createShatterScreenEffect(); // 화면 전체 깨지는 효과
                incorrectSound.triggerAttackRelease("G2", "16n", now, 0.8);
            }

            currentQuestionIndex++;
            displayQuestion();
        }

        // 타이머 시작
        function startTimer() {
            startTime = Date.now();
            timerInterval = setInterval(() => {
                const elapsedTime = (Date.now() - startTime) / 1000;
                timerDisplay.textContent = `${elapsedTime.toFixed(2)}초`;
            }, 10);
        }

        // 게임 종료
        function endGame() {
            clearInterval(timerInterval);
            const totalTime = ((Date.now() - startTime) / 1000).toFixed(2);

            const now = Tone.now();
            gameEndSound.triggerAttackRelease("C5", "8n", now);
            gameEndSound.triggerAttackRelease("E5", "8n", now + 0.1);
            gameEndSound.triggerAttackRelease("G5", "4n", now + 0.2);

            createFireworksEffect();

            quizScreen.classList.add('hidden');
            resultScreen.classList.remove('hidden');

            scoreDisplay.textContent = `${score} / ${quizData.length}`;
            timeTakenDisplay.textContent = `${totalTime}초`;

            saveRecord(totalTime);
            displayRankings();
        }

        // 기록 저장
        function saveRecord(time) {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            const newRecord = {
                id: userIdInput.value.trim(),
                phone: userPhoneInput.value.trim(),
                score: score,
                time: parseFloat(time)
            };
            
            rankings.push(newRecord);
            rankings.sort((a, b) => {
                if (b.score !== a.score) {
                    return b.score - a.score;
                }
                return a.time - b.time;
            });

            localStorage.setItem('ecoQuizRankings', JSON.stringify(rankings));
        }

        // 일반 사용자용 순위 표시 (상위 10명)
        function displayRankings() {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            rankTableBody.innerHTML = '';

            if (rankings.length === 0) {
                rankTableBody.innerHTML = '<tr><td colspan="4">아직 등록된 기록이 없습니다.</td></tr>';
                return;
            }

            const displayCount = Math.min(rankings.length, 10);

            for (let i = 0; i < displayCount; i++) {
                const rank = rankings[i];
                const tr = document.createElement('tr');
                tr.className = (rank.id === userIdInput.value.trim() && rank.time.toFixed(2) === timeTakenDisplay.textContent.replace('초', '')) ? 'bg-yellow-200 font-bold' : '';
                tr.innerHTML = `
                    <td class="font-bold">${i + 1}</td>
                    <td>${rank.id}</td>
                    <td>${rank.score}</td>
                    <td>${rank.time.toFixed(2)}</td>
                `;
                rankTableBody.appendChild(tr);
            }
        }
        
        // 처음으로 돌아가기
        function restartGame() {
            resultScreen.classList.add('hidden');
            adminScreen.classList.add('hidden');
            startScreen.classList.remove('hidden');
            userIdInput.value = '';
            userPhoneInput.value = '';
        }

        // --- 관리자 기능 ---
        
        function showAdminLogin() {
            adminPasswordInput.value = '';
            adminError.textContent = '';
            adminLoginModal.classList.remove('hidden');
        }

        function handleAdminLogin() {
            if (adminPasswordInput.value === '1212') {
                adminLoginModal.classList.add('hidden');
                startScreen.classList.add('hidden');
                resultScreen.classList.add('hidden');
                quizScreen.classList.add('hidden');
                adminScreen.classList.remove('hidden');
                displayAdminRankings();
            } else {
                adminError.textContent = '비밀번호가 올바르지 않습니다.';
            }
        }

        function displayAdminRankings() {
            const rankings = JSON.parse(localStorage.getItem('ecoQuizRankings')) || [];
            adminRankBody.innerHTML = '';

            if (rankings.length === 0) {
                adminRankBody.innerHTML = '<tr><td colspan="5">아직 등록된 기록이 없습니다.</td></tr>';
                return;
            }

            rankings.forEach((rank, index) => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="font-bold">${index + 1}</td>
                    <td>${rank.id}</td>
                    <td>${rank.phone}</td>
                    <td>${rank.score}</td>
                    <td>${rank.time.toFixed(2)}</td>
                `;
                adminRankBody.appendChild(tr);
            });
        }

        // 이벤트 리스너 연결
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', restartGame);
        choiceBtns.forEach(button => {
            button.addEventListener('click', (e) => {
                if (Tone.context.state !== 'running') {
                    Tone.start();
                    if (!correctSound) {
                        initSounds();
                    }
                }
                selectAnswer(e);
            });
        });

        adminModeBtn.addEventListener('click', showAdminLogin);
        adminCloseBtn.addEventListener('click', () => adminLoginModal.classList.add('hidden'));
        adminLoginBtn.addEventListener('click', handleAdminLogin);
        adminBackBtn.addEventListener('click', restartGame);
        adminPasswordInput.addEventListener('keyup', (e) => {
            if(e.key === 'Enter') handleAdminLogin();
        })

    </script>
</body>
</html>

