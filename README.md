# kotomanacard
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KOTOHANA - 心身相談＆市販薬事前チェックサポート</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- html2pdf.js for generating PDF cards -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700&family=Zen+Maru+Gothic:wght@400;500;700&display=swap" rel="stylesheet">
    
    <style>
        body {
            font-family: 'Zen Maru Gothic', 'Plus Jakarta Sans', sans-serif;
        }
        /* Custom scrollbar for better chat experience */
        #chat-container::-webkit-scrollbar {
            width: 5px;
        }
        #chat-container::-webkit-scrollbar-track {
            background: transparent;
        }
        #chat-container::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 10px;
        }
        /* Touch friendly tap highlights */
        * {
            -webkit-tap-highlight-color: transparent;
        }
        /* Print styles */
        @media print {
            body {
                background: #ffffff !important;
                padding: 0 !important;
                margin: 0 !important;
            }
            body > *:not(#printable-card-area) {
                display: none !important;
            }
            #printable-card-area, #printable-card-area * {
                visibility: visible !important;
            }
            #printable-card-area {
                position: absolute !important;
                left: 0 !important;
                top: 0 !important;
                width: 100% !important;
                max-height: none !important;
                overflow: visible !important;
                margin: 0 !important;
                padding: 0 !important;
                border: none !important;
                box-shadow: none !important;
            }
            #pdf-card {
                border: 2px solid #0d9488 !important;
                box-shadow: none !important;
                width: 100% !important;
                page-break-inside: avoid;
            }
            .no-print {
                display: none !important;
            }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-teal-50 via-slate-50 to-emerald-50 text-slate-800 min-h-screen flex items-center justify-center p-0 sm:p-4">

    <div class="w-full sm:max-w-md bg-white/95 backdrop-blur-md sm:rounded-3xl shadow-2xl overflow-hidden flex flex-col h-[100dvh] sm:h-[94vh] sm:max-h-[820px] border-0 sm:border border-teal-100">
        
        <!-- Header Section -->
        <div id="main-header" class="bg-gradient-to-r from-teal-700 via-teal-600 to-emerald-600 text-white p-3 sm:p-4 shadow-md flex-shrink-0 relative transition-all duration-300">
            <div class="flex items-center justify-between">
                <h1 class="font-bold text-sm sm:text-lg tracking-wide flex items-center gap-1.5 sm:gap-2">
                    <span id="header-icon" class="text-lg sm:text-xl">🌱</span>
                    <span id="header-title">KOTOHANA</span>
                    <span id="header-badge" class="text-[9px] sm:text-[10px] bg-white/20 backdrop-blur-md text-white font-medium px-2 py-0.5 rounded-full border border-white/20">安心の匿名サポート</span>
                </h1>
                <span id="header-subtitle" class="text-[10px] sm:text-xs text-teal-100 font-medium">心と体の相談メモ</span>
            </div>
            <p id="header-desc" class="text-[10px] sm:text-[11px] opacity-90 mt-1 sm:mt-1.5 leading-relaxed">伝えにくい不調やけが、悩みをカードに整理して周囲や専門家へ共有できます</p>
        </div>

        <!-- Safety Disclaimer Banner -->
        <div id="safety-banner" class="bg-amber-50/90 border-b border-amber-200/60 text-amber-900 px-3 py-1.5 sm:px-3.5 sm:py-2 text-[10px] sm:text-[11px] leading-snug sm:leading-relaxed flex items-start gap-1.5 sm:gap-2 flex-shrink-0">
            <span class="text-amber-600 font-bold text-xs sm:text-sm">💡</span>
            <div>
                このアプリは診断をするものではありません。整理したメモ（KOTOHANAカード）を保護者、保健室の先生、ドラッグストアの薬剤師さんへスムーズに提示するためのツールです。
            </div>
        </div>

        <!-- Progress Header & Back Control -->
        <div id="step-nav" class="bg-slate-100/80 px-3 py-1.5 border-b border-slate-200 flex items-center justify-between text-[10px] sm:text-[11px] font-bold text-slate-600 flex-shrink-0">
            <button id="back-btn" onclick="goBackStep()" class="hidden text-teal-800 hover:text-teal-900 bg-white px-2.5 py-1 rounded-lg border border-slate-200 shadow-sm active:scale-95 flex items-center gap-1">
                <span>↩️</span>
                <span>1つ前に戻る</span>
            </button>
            <div class="flex items-center gap-2 ml-auto">
                <span id="step-indicator" class="text-slate-500">準備中...</span>
            </div>
        </div>

        <!-- Chat Container -->
        <div id="chat-container" class="flex-1 p-3 sm:p-4 overflow-y-auto space-y-3 sm:space-y-4 bg-slate-50/50"></div>

        <!-- Input / Dynamic Controls Area -->
        <div id="input-container" class="p-2.5 sm:p-3.5 bg-white border-t border-slate-100 flex-shrink-0 shadow-lg max-h-[50dvh] overflow-y-auto"></div>
    </div>

    <script>
        // XSSサニタイズ処理
        function escapeHtml(str) {
            if (typeof str !== 'string') return str;
            return str
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;')
                .replace(/'/g, '&#039;');
        }

        // グローバル状態管理
        const state = {
            mode: 'physical',   // 'physical' or 'mental'
            birthdate: '',
            calculatedAge: '',
            ageGroup: '',
            height: '160',
            weight: '50',
            pregnancy: 'いいえ',
            redFlags: [],
            symptom: '',
            onset: '',
            severity: '7',
            impact: '',
            currentMeds: '',
            allergies: '',
            history: '',
            doping: '',
            expectations: '',
            finalNotes: '',
            destination: '',
            isEmergency: false,
            geminiAdvice: ''
        };

        // 🔑 バックグラウンド連携用 API キー
        let geminiApiKey = 'AQ.Ab8RN6K_4fj9jQdz-cjznf2qbVSCjnds7AHNcz4Q0jpnflq-HQ';
        let currentStep = 0;
        let conversationHistory = [];

        const chatContainer = document.getElementById('chat-container');
        const inputContainer = document.getElementById('input-container');
        const stepIndicator = document.getElementById('step-indicator');
        const backBtn = document.getElementById('back-btn');

        window.onload = function() {
            startInitialFlow();
        };

        function startInitialFlow() {
            currentStep = 0;
            conversationHistory = [];
            state.mode = 'physical';
            state.isEmergency = false;
            setThemeStyle('physical');
            if (chatContainer) chatContainer.innerHTML = '';
            addBotMessage("こんにちは。KOTOHANAへようこそ。ここは完全匿名で利用できる安心な場所です。\n\nまず最初に、**どのようなご相談**ですか？");
            updateStepIndicator("相談カテゴリーの選択");

            if (inputContainer) {
                inputContainer.innerHTML = `
                    <div class="space-y-2">
                        <button type="button" onclick="selectMode('physical')" class="w-full py-3 px-3.5 bg-gradient-to-r from-teal-50 to-emerald-50 hover:from-teal-100 hover:to-emerald-100 text-teal-900 border border-teal-200 rounded-2xl font-bold text-xs sm:text-sm text-left flex items-center justify-between shadow-sm active:scale-98 cursor-pointer">
                            <div class="flex items-center gap-2">
                                <span class="text-lg">🩺</span>
                                <div>
                                    <div>体の不調・けが・痛みの相談</div>
                                    <div class="text-[9.5px] sm:text-[10px] text-teal-700 font-normal">頭痛、腹痛、ねんざ、創傷、風邪症状など</div>
                                </div>
                            </div>
                            <span class="text-teal-600">→</span>
                        </button>
                        <button type="button" onclick="selectMode('mental')" class="w-full py-3 px-3.5 bg-gradient-to-r from-purple-50 via-fuchsia-50 to-rose-50 hover:from-purple-100 hover:to-fuchsia-100 text-purple-950 border border-purple-200 rounded-2xl font-bold text-xs sm:text-sm text-left flex items-center justify-between shadow-sm active:scale-98 cursor-pointer">
                            <div class="flex items-center gap-2">
                                <span class="text-lg">🌙</span>
                                <div>
                                    <div>心の相談・気分の不調（専用モード）</div>
                                    <div class="text-[9.5px] sm:text-[10px] text-purple-700 font-normal">気持ちの整理・不安・疲れ・学校生活の悩み</div>
                                </div>
                            </div>
                            <span class="text-purple-600">→</span>
                        </button>
                    </div>
                `;
            }
        }

        function updateStepIndicator(text) {
            if (stepIndicator) stepIndicator.innerText = text;
            if (backBtn) {
                if (currentStep > 0) backBtn.classList.remove('hidden');
                else backBtn.classList.add('hidden');
            }
        }

        function setThemeStyle(mode) {
            const header = document.getElementById('main-header');
            const icon = document.getElementById('header-icon');
            const subtitle = document.getElementById('header-subtitle');
            const desc = document.getElementById('header-desc');

            if (!header || !icon || !subtitle || !desc) return;

            if (mode === 'mental') {
                header.className = "bg-gradient-to-r from-purple-900 via-purple-800 to-rose-900 text-white p-3 sm:p-4 shadow-md flex-shrink-0 relative transition-all duration-300";
                icon.innerText = "🌙";
                subtitle.innerText = "心の相談モード";
                subtitle.className = "text-[10px] sm:text-xs text-purple-200 font-medium";
                desc.innerText = "不安・悲しみ・言葉にしにくい気持ちを静かに整理します";
            } else {
                header.className = "bg-gradient-to-r from-teal-700 via-teal-600 to-emerald-600 text-white p-3 sm:p-4 shadow-md flex-shrink-0 relative transition-all duration-300";
                icon.innerText = "🌱";
                subtitle.innerText = "心と体の相談メモ";
                subtitle.className = "text-[10px] sm:text-xs text-teal-100 font-medium";
                desc.innerText = "伝えにくい不調やけが、悩みをカードに整理して周囲や専門家へ共有できます";
            }
        }

        function selectMode(mode) {
            state.mode = mode;
            setThemeStyle(mode);
            saveStepState();
            currentStep = 1;
            if (mode === 'physical') {
                addUserMessage("🩺 体の不調・けが・痛みの相談");
                checkRedFlagsFirst();
            } else {
                addUserMessage("🌙 心の相談モードを選択しました");
                askMentalQuestions();
            }
        }

        function checkRedFlagsFirst() {
            saveStepState();
            addBotMessage("最初に、緊急性の高い症状がないか安全確認を行います。\n当てはまる項目があるかチェックしてください。");
            updateStepIndicator("🚨 緊急安全チェック");
            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <div class="bg-amber-50 p-2 rounded-xl border border-amber-200 text-[10.5px] text-amber-900">
                        以下の**緊急の症状**に当てはまるものはありますか？（複数選択可）
                    </div>
                    <div class="grid grid-cols-1 gap-1.5 max-h-[160px] overflow-y-auto p-1 bg-slate-50 border border-slate-200 rounded-xl">
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="息苦しさ・激しい呼吸困難" class="accent-rose-600 rounded w-4 h-4">
                            <span>🫁 息苦しさや呼吸が困難</span>
                        </label>
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="意識が遠のく・呼びかけへの反応が悪い" class="accent-rose-600 rounded w-4 h-4">
                            <span>🧠 意識がおかしい・反応が悪い</span>
                        </label>
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="突然の激しい頭痛" class="accent-rose-600 rounded w-4 h-4">
                            <span>💥 突然のバットで殴られたような激しい頭痛</span>
                        </label>
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="強い胸の痛み・締め付け感" class="accent-rose-600 rounded w-4 h-4">
                            <span>💔 強い胸の痛みや締め付け</span>
                        </label>
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="大量の出血" class="accent-rose-600 rounded w-4 h-4">
                            <span>🩸 止まらない大量の出血</span>
                        </label>
                        <label class="flex items-center gap-2 p-2 bg-white rounded-lg border border-slate-200 text-[11px] font-bold text-slate-800 cursor-pointer">
                            <input type="checkbox" name="redflag" value="重いアレルギー反応（アナフィラキシー）" class="accent-rose-600 rounded w-4 h-4">
                            <span>⚠️ 全身の蕁麻疹・腫れ・アレルギー反応</span>
                        </label>
                    </div>
                    <div class="flex gap-2">
                        <button type="button" onclick="submitRedFlags(false)" class="flex-1 py-2.5 bg-teal-700 hover:bg-teal-800 text-white font-bold rounded-xl text-xs shadow-sm active:scale-98">
                            該当なし（通常の問診へ進む）
                        </button>
                        <button type="button" onclick="submitRedFlags(true)" class="py-2.5 px-3 bg-rose-700 hover:bg-rose-800 text-white font-bold rounded-xl text-xs shadow-sm active:scale-98">
                            ⚠️ 該当あり
                        </button>
                    </div>
                </div>
            `;
        }

        function submitRedFlags(hasFlags) {
            if (hasFlags) {
                const checked = document.querySelectorAll('input[name="redflag"]:checked');
                let selected = [];
                checked.forEach(c => selected.push(c.value));
                state.redFlags = selected;
                if (selected.length === 0) selected.push("緊急性が疑われる重篤な症状");
                addUserMessage("⚠️ 危険症状あり: " + selected.join('、'));
                
                state.isEmergency = true;
                addBotMessage("🚨 **【緊急メッセージ】**\n選択された症状は、命にかかわる緊急性が高い可能性があります。\n通常の問診を中断します。今すぐ119番通報をするか、近くの大人・先生・保健室へ助けを求めてください！");
                updateStepIndicator("🚨 救急要請を最優先してください");
                
                inputContainer.innerHTML = `
                    <div class="bg-rose-50 border-2 border-rose-500 rounded-2xl p-3.5 space-y-3 text-rose-950">
                        <div class="font-black text-xs text-rose-800">今すぐ行える緊急対応</div>
                        <div class="space-y-1.5 text-[11px] font-bold">
                            <div class="bg-white p-2 rounded-xl border border-rose-200 flex justify-between items-center">
                                <span>🚑 救急車を呼ぶ (119番通報)</span>
                                <a href="tel:119" class="bg-rose-600 text-white px-3 py-1 rounded-lg font-black text-xs">今すぐ発信</a>
                            </div>
                            <div class="bg-white p-2 rounded-xl border border-rose-200">
                                🏫 近くにいる保護者・先生・保健室の先生へ直接伝える
                            </div>
                            <div class="bg-white p-2 rounded-xl border border-rose-200">
                                📞 救急安心センター事業: <a href="tel:#7119" class="text-rose-700 underline font-black">#7119</a>
                            </div>
                        </div>
                        <button type="button" onclick="startInitialFlow()" class="w-full py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl border border-slate-300">最初に戻る</button>
                    </div>
                `;
            } else {
                addUserMessage("✅ 緊急危険症状なし");
                currentStep = 2;
                askPhysicalStep();
            }
        }

        function askPhysicalStep() {
            saveStepState();
            if (currentStep === 2) {
                addBotMessage("生年月日（または年齢）を教えてください。");
                updateStepIndicator("年齢の確認");
                renderBirthdateControl(false);
            } else if (currentStep === 3) {
                addBotMessage("体格の目安（身長・体重）および妊娠の可能性について教えてください。");
                updateStepIndicator("体格・身体情報の確認");
                renderBodyMetricsControl();
            } else if (currentStep === 4) {
                addBotMessage("一番気になっている**症状や、けがの場所、痛み**を教えてください。（複数選択可）");
                updateStepIndicator("症状・不調部位の確認");
                renderSymptomControl();
            } else if (currentStep === 5) {
                addBotMessage("その症状やけがは、いつ頃から始まりましたか？");
                updateStepIndicator("発症時期の確認");
                renderOnsetControl(false);
            } else if (currentStep === 6) {
                addBotMessage("つらさや痛みの程度をスライダーで選んでください（1〜10）。");
                updateStepIndicator("つらさレベルの確認");
                renderSeverityControl(false);
            } else if (currentStep === 7) {
                addBotMessage("💊 **安全チェック**です。\n現在**服用している薬**（市販薬含む）や、薬・食品の**アレルギー**、持病はありますか？");
                updateStepIndicator("服用薬・アレルギー確認");
                renderSafetyHistoryControl();
            } else if (currentStep === 8) {
                addBotMessage("部活動や競技の公式大会など、**アンチ・ドーピング**（使用禁止薬物）の確認が必要ですか？");
                updateStepIndicator("アンチ・ドーピング確認");
                renderDopingControl();
            } else if (currentStep === 9) {
                addBotMessage("保健室やドラッグストアで特に希望する配慮・対応があれば教えてください。");
                updateStepIndicator("配慮事項の確認");
                renderExpectationsControl(false);
            } else if (currentStep === 10) {
                addBotMessage("最後に、カードを見せる相手へ**伝えたいひとことメッセージ**があればご記入ください。");
                updateStepIndicator("最終メッセージの入力");
                renderFinalNotesControl(false);
            } else if (currentStep === 11) {
                askDestination(false);
            }
        }

        function askMentalQuestions() {
            saveStepState();
            if (currentStep === 1) {
                addBotMessage("【ステップ①：安全確認】\n今、あなたの身の安全は確保できていますか？自傷の危険や、緊急で助けが必要な状況ではないことを確認します。");
                updateStepIndicator("① 安全確認");
                inputContainer.innerHTML = `
                    <div class="space-y-2">
                        <div class="bg-purple-50 border border-purple-200 p-2 rounded-xl text-[10.5px] text-purple-950">
                            🛡️ まず今の状態を確認させてください
                        </div>
                        <div class="grid grid-cols-1 gap-1.5">
                            <button onclick="handleMentalSafety('safe')" class="py-2.5 px-3 bg-purple-900 hover:bg-purple-950 text-white font-bold rounded-xl text-xs active:scale-98 shadow-sm text-left flex items-center justify-between">
                                <span>今すぐ命の危険はありません（気持ちを整理する）</span>
                                <span>→</span>
                            </button>
                            <button onclick="handleMentalSafety('uncertain')" class="py-2.5 px-3 bg-indigo-50 hover:bg-indigo-100 text-indigo-900 font-bold rounded-xl text-xs border border-indigo-200 active:scale-98 shadow-sm text-left flex items-center justify-between">
                                <span>よくわからない・今は答えにくい</span>
                                <span>💬</span>
                            </button>
                            <button onclick="handleMentalSafety('emergency')" class="py-2.5 px-3 bg-rose-700 hover:bg-rose-800 text-white font-bold rounded-xl text-xs active:scale-98 shadow-sm text-left flex items-center justify-between">
                                <span>⚠️ 自分を傷つけそう・緊急の助けが必要</span>
                                <span>🚨</span>
                            </button>
                        </div>
                    </div>
                `;
            } else if (currentStep === 2) {
                addBotMessage("生年月日（または年齢区分）を教えてください。");
                updateStepIndicator("年齢の確認");
                renderBirthdateControl(true);
            } else if (currentStep === 3) {
                addBotMessage("【ステップ③：気持ちの整理】\n言葉にしにくい今の感情や気分の状態を、当てはまるものから選んでみてください。（複数選択可）");
                updateStepIndicator("③ 気持ちの整理");
                inputContainer.innerHTML = `
                    <div class="space-y-2">
                        <div class="grid grid-cols-2 gap-1.5 max-h-[160px] overflow-y-auto p-1 bg-purple-50/50 border border-purple-200 rounded-xl">
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="😰 不安・緊張が強い" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>😰 不安・緊張が強い</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="😢 わけもなく悲しい・涙が出る" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>😢 悲しい・涙が出る</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="🥱 疲れ切っている・力が出ない" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>🥱 疲れ切っている</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="🌀 イライラする・モヤモヤする" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>🌀 イライラ・モヤモヤ</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="😶 何も感じない・無気力" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>😶 何も感じない</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-2 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_feel" value="💬 誰かに聞いてほしい" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>💬 話を聞いてほしい</span>
                            </label>
                        </div>
                        <div class="flex gap-1.5">
                            <input type="text" id="customFeelInput" placeholder="その他の感情（自由記入）" class="flex-1 px-2.5 py-1.5 text-xs border border-purple-200 rounded-xl outline-none focus:border-purple-600 bg-white">
                            <button onclick="submitMentalFeeling()" class="bg-purple-800 hover:bg-purple-900 text-white px-3 py-1.5 rounded-xl text-xs font-bold transition shadow-sm whitespace-nowrap active:scale-95">次へ</button>
                        </div>
                    </div>
                `;
            } else if (currentStep === 4) {
                addBotMessage("【ステップ④：期間の確認】\nそのつらい気持ちやモヤモヤは、いつ頃から続いていますか？");
                updateStepIndicator("④ 期間の確認");
                renderOnsetControl(true);
            } else if (currentStep === 5) {
                addBotMessage("【ステップ⑤：生活への影響】\n困っている生活への影響（学校、勉強、睡眠、食事など）を教えてください。（複数選択可）");
                updateStepIndicator("⑤ 生活への影響");
                inputContainer.innerHTML = `
                    <div class="space-y-2">
                        <div class="grid grid-cols-2 gap-1.5 max-h-[150px] overflow-y-auto p-1 bg-purple-50/50 border border-purple-200 rounded-xl">
                            <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_impact" value="💤 夜寝られない・朝起きられない" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>💤 睡眠に響いている</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_impact" value="🏫 学校に行きづらい・休みたい" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>🏫 学校へ行きづらい</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_impact" value="✏️ 勉強や作業に集中できない" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>✏️ 集中できない</span>
                            </label>
                            <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-purple-100 text-[10px] sm:text-[11px] font-bold text-purple-950 cursor-pointer">
                                <input type="checkbox" name="mental_impact" value="🍚 食欲がない・食べ過ぎる" class="accent-purple-700 rounded w-3.5 h-3.5">
                                <span>🍚 食生活の不調</span>
                            </label>
                        </div>
                        <div class="flex gap-1.5">
                            <input type="text" id="customImpactInput" placeholder="その他の生活の困りごと" class="flex-1 px-2.5 py-1.5 text-xs border border-purple-200 rounded-xl outline-none focus:border-purple-600 bg-white">
                            <button onclick="submitMentalImpact()" class="bg-purple-800 hover:bg-purple-900 text-white px-3 py-1.5 rounded-xl text-xs font-bold transition shadow-sm whitespace-nowrap active:scale-95">次へ</button>
                        </div>
                    </div>
                `;
            } else if (currentStep === 6) {
                addBotMessage("【ステップ⑥：必要な助け】\n周りの大人（保護者、先生、カウンセラー等）に一番してほしい対応を選んでみましょう。");
                updateStepIndicator("⑥ 必要な助け");
                renderExpectationsControl(true);
            } else if (currentStep === 7) {
                addBotMessage("【ステップ⑦：相談メモの作成】\n最後に、カードを見せる相手へ伝えたいこと（または一言メモ）を入力してください。");
                updateStepIndicator("⑦ 最終メモ");
                renderFinalNotesControl(true);
            } else if (currentStep === 8) {
                askDestination(true);
            }
        }

        function handleMentalSafety(type) {
            if (type === 'emergency') {
                addUserMessage("⚠️ 自傷の恐れ・緊急の相談要請あり");
                addBotMessage("🚨 **【緊急の相談窓口のご案内】**\n1人で抱え込まず、今すぐ以下の専門窓口や周囲へ助けを求めてください。");
                updateStepIndicator("🚨 緊急心のホットライン");
                inputContainer.innerHTML = `
                    <div class="bg-rose-50 border-2 border-rose-500 rounded-2xl p-3.5 space-y-3 text-rose-950">
                        <div class="font-black text-xs text-rose-800">すぐ繋がる電話・チャット相談窓口</div>
                        <div class="space-y-1.5 text-[11px] font-bold">
                            <div class="bg-white p-2 rounded-xl border border-rose-200">
                                📞 こころの健康相談統一ダイヤル: <a href="tel:0570064556" class="text-rose-700 underline font-black">0570-064-556</a><br>
                                <span class="text-[9.5px] font-normal text-slate-600">（対応時間：都道府県・時間帯により異なる/厚労省案内）</span>
                            </div>
                            <div class="bg-white p-2 rounded-xl border border-rose-200">
                                📞 24時間子供SOSダイヤル: <a href="tel:0120078310" class="text-rose-700 underline font-black">0120-0-78310</a><br>
                                <span class="text-[9.5px] font-normal text-slate-600">（24時間365日対応・通話料無料/文科省案内）</span>
                            </div>
                            <div class="bg-white p-2 rounded-xl border border-rose-200">
                                💬 厚生労働省 SNS相談（LINE/チャット「まもろうよ こころ」）
                            </div>
                        </div>
                        <button onclick="startInitialFlow()" class="w-full py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl border border-slate-300">最初に戻る</button>
                    </div>
                `;
            } else if (type === 'uncertain') {
                addUserMessage("よくわからない・今は答えにくい");
                currentStep = 2;
                askMentalQuestions();
            } else {
                addUserMessage("今すぐ命の危険はありません（気持ちの整理へ進む）");
                currentStep = 2;
                askMentalQuestions();
            }
        }

        function submitMentalFeeling() {
            const checkedBoxes = document.querySelectorAll('input[name="mental_feel"]:checked');
            let list = [];
            checkedBoxes.forEach(b => list.push(b.value));
            const custom = document.getElementById('customFeelInput');
            if (custom && custom.value.trim()) list.push(custom.value.trim());

            if (list.length === 0) list.push("ことばにしづらいモヤモヤ・不調");
            state.symptom = list.join('、');
            addUserMessage("・感情/状態: " + state.symptom);
            currentStep = 4;
            askMentalQuestions();
        }

        function submitMentalImpact() {
            const checkedBoxes = document.querySelectorAll('input[name="mental_impact"]:checked');
            let list = [];
            checkedBoxes.forEach(b => list.push(b.value));
            const custom = document.getElementById('customImpactInput');
            if (custom && custom.value.trim()) list.push(custom.value.trim());

            if (list.length === 0) list.push("特になし（日常生活への大きな支障は未申告）");
            state.impact = list.join('、');
            addUserMessage("・生活への影響: " + state.impact);
            currentStep = 6;
            askMentalQuestions();
        }

        function renderBirthdateControl(isMental = false) {
            inputContainer.innerHTML = `
                <div class="flex gap-2 items-center">
                    <input type="date" id="birthdateInput" class="flex-1 px-3 py-2 text-xs border border-slate-300 rounded-xl outline-none focus:border-teal-600 bg-white shadow-sm">
                    <button onclick="submitBirthdate(${isMental})" class="bg-teal-700 hover:bg-teal-800 text-white px-4 py-2 rounded-xl text-xs font-bold transition shadow-sm active:scale-95">決定</button>
                </div>
            `;
        }

        function submitBirthdate(isMental = false) {
            const input = document.getElementById('birthdateInput');
            if (!input || !input.value) {
                alert('生年月日を選択してください。');
                return;
            }
            state.birthdate = input.value;
            const age = calculateAge(input.value);
            if (age !== null) {
                state.calculatedAge = age;
                state.ageGroup = age < 15 ? `${age}歳 (15歳未満)` : `${age}歳 (15歳以上・高校生/成人)`;
                addUserMessage(`${input.value}（${state.calculatedAge}歳）`);
            } else {
                state.calculatedAge = '';
                state.ageGroup = input.value;
                addUserMessage(input.value);
            }
            currentStep++;
            if (isMental) askMentalQuestions(); else askPhysicalStep();
        }

        function renderBodyMetricsControl() {
            let heightOptions = '';
            for (let h = 120; h <= 200; h++) heightOptions += `<option value="${h}" ${h === 160 ? 'selected' : ''}>${h} cm</option>`;
            let weightOptions = '';
            for (let w = 30; w <= 120; w++) weightOptions += `<option value="${w}" ${w === 50 ? 'selected' : ''}>${w} kg</option>`;

            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <div class="grid grid-cols-2 gap-2">
                        <div class="bg-slate-50 p-1.5 rounded-xl border border-slate-200 text-center">
                            <label class="text-[9px] text-slate-500 font-bold block mb-0.5">身長</label>
                            <select id="heightSelect" class="w-full bg-white border border-slate-300 rounded-lg p-1 text-xs font-bold text-teal-800 text-center outline-none">
                                ${heightOptions}
                            </select>
                        </div>
                        <div class="bg-slate-50 p-1.5 rounded-xl border border-slate-200 text-center">
                            <label class="text-[9px] text-slate-500 font-bold block mb-0.5">体重</label>
                            <select id="weightSelect" class="w-full bg-white border border-slate-300 rounded-lg p-1 text-xs font-bold text-teal-800 text-center outline-none">
                                ${weightOptions}
                            </select>
                        </div>
                    </div>
                    <div class="bg-slate-50 p-1.5 rounded-xl border border-slate-200 flex items-center justify-between text-xs font-bold">
                        <span class="text-slate-600 text-[10px]">妊娠の可能性（※市販薬の成分確認）</span>
                        <select id="pregnancySelect" class="bg-white border border-slate-300 rounded-lg p-1 text-xs font-bold text-teal-800 outline-none">
                            <option value="いいえ">いいえ / 該当しない</option>
                            <option value="あり・可能性あり">あり / 可能性あり</option>
                        </select>
                    </div>
                    <button onclick="submitBodyMetrics()" class="w-full py-2 bg-teal-700 hover:bg-teal-800 text-white text-xs font-bold rounded-xl shadow-sm transition active:scale-98">決定して進む</button>
                </div>
            `;
        }

        function submitBodyMetrics() {
            state.height = document.getElementById('heightSelect').value;
            state.weight = document.getElementById('weightSelect').value;
            state.pregnancy = document.getElementById('pregnancySelect').value;
            addUserMessage(`身長: ${state.height}cm / 体重: ${state.weight}kg / 妊娠の可能性: ${state.pregnancy}`);
            currentStep++;
            askPhysicalStep();
        }

        function renderSymptomControl() {
            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <div class="grid grid-cols-2 gap-1.5 max-h-[160px] overflow-y-auto p-1 bg-slate-50 border border-slate-200 rounded-xl">
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="頭痛・めまい" class="accent-teal-700 rounded w-3.5 h-3.5">
                            <span>🤕 頭痛・めまい</span>
                        </label>
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="腹痛・胃痛" class="accent-teal-700 rounded w-3.5 h-3.5">
                            <span>😣 腹痛・胃痛</span>
                        </label>
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="発熱・寒気・のど痛" class="accent-teal-700 rounded w-3.5 h-3.5">
                            <span>😷 発熱・のど痛</span>
                        </label>
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="擦り傷・切り傷" class="accent-amber-600 rounded w-3.5 h-3.5">
                            <span>🩹 擦り傷・切り傷</span>
                        </label>
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="打撲・捻挫・痛む箇所" class="accent-amber-600 rounded w-3.5 h-3.5">
                            <span>🦴 打撲・捻挫</span>
                        </label>
                        <label class="flex items-center gap-1.5 p-1.5 bg-white rounded-lg border border-slate-200 text-[10px] sm:text-[11px] font-medium text-slate-700 cursor-pointer">
                            <input type="checkbox" name="symptom_chip" value="やけど（火傷）" class="accent-amber-600 rounded w-3.5 h-3.5">
                            <span>🔥 やけど</span>
                        </label>
                    </div>
                    <div class="flex gap-1.5">
                        <input type="text" id="customSymptomInput" placeholder="詳細（例：右足首の激痛）" class="flex-1 px-2.5 py-1.5 text-xs border border-slate-300 rounded-xl outline-none focus:border-teal-600 bg-white">
                        <button onclick="submitMultipleSymptoms()" class="bg-teal-700 hover:bg-teal-800 text-white px-3 py-1.5 rounded-xl text-xs font-bold transition shadow-sm whitespace-nowrap active:scale-95">確定</button>
                    </div>
                </div>
            `;
        }

        function submitMultipleSymptoms() {
            const checked = document.querySelectorAll('input[name="symptom_chip"]:checked');
            let list = [];
            checked.forEach(b => list.push(b.value));
            const custom = document.getElementById('customSymptomInput');
            if (custom && custom.value.trim()) list.push(custom.value.trim());

            if (list.length === 0) {
                alert('症状を選択するか入力してください。');
                return;
            }
            state.symptom = list.join('、');
            addUserMessage(state.symptom);
            currentStep++;
            askPhysicalStep();
        }

        function renderOnsetControl(isMental = false) {
            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <div class="grid grid-cols-2 gap-1.5">
                        <button onclick="handleOptionSelect('onset', '今日・さっきから', ${isMental})" class="py-2 px-1.5 bg-teal-50 hover:bg-teal-100 text-teal-800 text-[10px] font-bold rounded-lg border border-teal-200">🏃‍♂️ 今日・さっきから</button>
                        <button onclick="handleOptionSelect('onset', '昨日の夜から', ${isMental})" class="py-2 px-1.5 bg-teal-50 hover:bg-teal-100 text-teal-800 text-[10px] font-bold rounded-lg border border-teal-200">🌙 昨日の夜から</button>
                        <button onclick="handleOptionSelect('onset', '数日前から続いて居る', ${isMental})" class="py-2 px-1.5 bg-teal-50 hover:bg-teal-100 text-teal-800 text-[10px] font-bold rounded-lg border border-teal-200">🗓️ 数日前から</button>
                        <button onclick="handleOptionSelect('onset', '1週間以上前からずっと', ${isMental})" class="py-2 px-1.5 bg-teal-50 hover:bg-teal-100 text-teal-800 text-[10px] font-bold rounded-lg border border-teal-200">⏳ 1週間以上前から</button>
                    </div>
                    <div class="flex gap-1.5">
                        <input type="text" id="customOnset" placeholder="自由入力（例：テスト前日頃から）" class="flex-1 px-2.5 py-1.5 text-xs border border-slate-300 rounded-lg outline-none focus:border-teal-600">
                        <button onclick="submitCustomText('onset', 'customOnset', ${isMental})" class="bg-teal-700 text-white px-3 py-1.5 rounded-lg text-xs font-bold active:scale-95">決定</button>
                    </div>
                </div>
            `;
        }

        function renderSeverityControl(isMental = false) {
            inputContainer.innerHTML = `
                <div class="space-y-2.5 bg-teal-50/50 p-2.5 rounded-2xl border border-teal-100">
                    <div class="flex items-center gap-2">
                        <span class="text-[10px] font-bold text-slate-500">1 (軽)</span>
                        <input type="range" id="severityRange" min="1" max="10" value="7" class="flex-1 accent-teal-700 h-2 bg-slate-200 rounded-lg cursor-pointer" oninput="document.getElementById('sevVal').innerText = this.value">
                        <span class="text-xs font-bold text-teal-800 bg-white px-2 py-0.5 rounded-md border border-teal-200"><span id="sevVal">7</span>/10</span>
                    </div>
                    <button onclick="handleOptionSelect('severity', document.getElementById('severityRange').value, ${isMental})" class="w-full py-2 bg-teal-700 hover:bg-teal-800 text-white text-xs font-bold rounded-xl shadow-sm active:scale-98">決定</button>
                </div>
            `;
        }

        function renderSafetyHistoryControl() {
            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <div class="bg-slate-50 p-2 rounded-xl border border-slate-200 space-y-1.5">
                        <input type="text" id="currentMedsInput" placeholder="1. 現在服用中の薬（例：抗ヒスタミン薬など）" class="w-full p-2 text-xs border border-slate-300 rounded-lg outline-none focus:border-teal-600 bg-white">
                        <input type="text" id="allergiesInput" placeholder="2. アレルギー（例：湿布かぶれ、卵など）" class="w-full p-2 text-xs border border-slate-300 rounded-lg outline-none focus:border-teal-600 bg-white">
                        <input type="text" id="historyInput" placeholder="3. 持病・過去の副作用（例：喘息、胃潰瘍など）" class="w-full p-2 text-xs border border-slate-300 rounded-lg outline-none focus:border-teal-600 bg-white">
                    </div>
                    <div class="flex gap-1.5">
                        <button onclick="submitSafetyHistoryNone()" class="w-1/3 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl border border-slate-300 active:scale-95">
                            特になし
                        </button>
                        <button onclick="submitSafetyHistory()" class="w-2/3 py-2 bg-teal-700 hover:bg-teal-800 text-white text-xs font-bold rounded-xl shadow-sm active:scale-98">
                            確認して決定
                        </button>
                    </div>
                </div>
            `;
        }

        function submitSafetyHistoryNone() {
            state.currentMeds = 'なし';
            state.allergies = 'なし';
            state.history = 'なし';
            addUserMessage('・服用薬: なし\n・アレルギー: なし\n・持病・副作用歴: なし');
            currentStep++;
            askPhysicalStep();
        }

        function submitSafetyHistory() {
            state.currentMeds = document.getElementById('currentMedsInput').value.trim() || 'なし';
            state.allergies = document.getElementById('allergiesInput').value.trim() || 'なし';
            state.history = document.getElementById('historyInput').value.trim() || 'なし';

            addUserMessage(`・服用薬: ${state.currentMeds}\n・アレルギー: ${state.allergies}\n・持病・副作用歴: ${state.history}`);
            currentStep++;
            askPhysicalStep();
        }

        function renderDopingControl() {
            inputContainer.innerHTML = `
                <div class="space-y-1.5">
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-1.5">
                        <button onclick="handleOptionSelect('doping', 'なし（特になし）')" class="py-2 px-2.5 bg-teal-50 hover:bg-teal-100 text-teal-800 text-[11px] font-bold rounded-xl border border-teal-200 text-left active:scale-98">
                            ✅ なし（特に考慮不要）
                        </button>
                        <button onclick="handleOptionSelect('doping', 'あり（公式大会・部活等でドーピング考慮が必要）')" class="py-2 px-2.5 bg-amber-50 hover:bg-amber-100 text-amber-900 text-[11px] font-bold rounded-xl border border-amber-200 text-left active:scale-98">
                            🏅 あり（大会出場・ドーピング確認が必要）
                        </button>
                    </div>
                    <div class="flex gap-1.5 pt-0.5">
                        <input type="text" id="customDoping" placeholder="詳細メモ（例：来週陸上の県大会あり）" class="flex-1 px-2.5 py-1.5 text-xs border border-slate-300 rounded-xl outline-none focus:border-teal-600">
                        <button onclick="submitCustomText('doping', 'customDoping')" class="bg-teal-700 text-white px-3 py-1.5 rounded-xl text-xs font-bold active:scale-95">決定</button>
                    </div>
                </div>
            `;
        }

        function renderExpectationsControl(isMental = false) {
            inputContainer.innerHTML = `
                <div class="space-y-1.5">
                    <input type="text" id="expectInput" placeholder="例：体育・部活を見学したい、静かな部屋で休みたい等" class="w-full p-2 text-xs border border-slate-300 rounded-xl outline-none focus:border-teal-600 bg-white">
                    <div class="flex gap-1.5">
                        <button onclick="document.getElementById('expectInput').value='特になし'; submitCustomText('expectations', 'expectInput', ${isMental});" class="w-1/3 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl border border-slate-300 active:scale-95">
                            特になし
                        </button>
                        <button onclick="submitCustomText('expectations', 'expectInput', ${isMental})" class="w-2/3 py-2 bg-teal-700 hover:bg-teal-800 text-white text-xs font-bold rounded-xl shadow-sm active:scale-98">
                            入力内容で決定
                        </button>
                    </div>
                </div>
            `;
        }

        function renderFinalNotesControl(isMental = false) {
            inputContainer.innerHTML = `
                <div class="space-y-2">
                    <textarea id="finalNotesInput" rows="2" placeholder="相手へ伝えたいこと（空欄でも可）" class="w-full p-2 text-xs border border-slate-300 rounded-xl outline-none focus:border-teal-600 bg-white resize-none"></textarea>
                    <div class="flex gap-1.5">
                        <button onclick="document.getElementById('finalNotesInput').value='特になし'; submitCustomText('finalNotes', 'finalNotesInput', ${isMental});" class="w-1/3 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl border border-slate-300 active:scale-95">
                            特になし
                        </button>
                        <button onclick="submitCustomText('finalNotes', 'finalNotesInput', ${isMental})" class="w-2/3 py-2 bg-teal-700 hover:bg-teal-800 text-white text-xs font-bold rounded-xl shadow-sm active:scale-98">
                            カードを作成する
                        </button>
                    </div>
                </div>
            `;
        }

        function handleOptionSelect(key, val, isMental = false) {
            state[key] = val;
            addUserMessage(val);
            currentStep++;
            if (isMental) askMentalQuestions(); else askPhysicalStep();
        }

        function submitCustomText(key, inputId, isMental = false) {
            const input = document.getElementById(inputId);
            let val = '特になし';
            if (input && input.value.trim() !== '') val = input.value.trim();
            state[key] = val;
            addUserMessage(val);
            currentStep++;
            if (isMental) askMentalQuestions(); else askPhysicalStep();
        }

        function saveStepState() {
            conversationHistory.push({
                step: currentStep,
                state: JSON.parse(JSON.stringify(state)),
                chatHtml: chatContainer.innerHTML
            });
        }

        function goBackStep() {
            if (conversationHistory.length > 0) {
                const prev = conversationHistory.pop();
                currentStep = prev.step;
                Object.assign(state, prev.state);
                chatContainer.innerHTML = prev.chatHtml;
                chatContainer.scrollTop = chatContainer.scrollHeight;

                if (state.mode === 'mental') {
                    askMentalQuestions();
                } else if (currentStep === 1) {
                    checkRedFlagsFirst();
                } else if (currentStep === 0) {
                    startInitialFlow();
                } else {
                    askPhysicalStep();
                }
            }
        }

        function askDestination(isMental = false) {
            addBotMessage("ご回答ありがとうございました！\n完成した**「KOTOHANA相談カード」**を誰に見せますか？");
            updateStepIndicator("カード宛先の選択");

            if (isMental) {
                inputContainer.innerHTML = `
                    <div class="flex flex-col gap-2">
                        <button onclick="selectDestination('保護者の方へ')" class="py-2.5 px-3.5 bg-emerald-50 hover:bg-emerald-100 text-emerald-800 font-bold rounded-xl text-xs border border-emerald-200 text-left transition flex items-center gap-2">👨‍👩‍👦 保護者の方へ</button>
                        <button onclick="selectDestination('学校の先生・スクールカウンセラーへ')" class="py-2.5 px-3.5 bg-emerald-50 hover:bg-emerald-100 text-emerald-800 font-bold rounded-xl text-xs border border-emerald-200 text-left transition flex items-center gap-2">🏫 学校の先生・スクールカウンセラーへ</button>
                        <button onclick="selectDestination('保健室の先生へ')" class="py-2.5 px-3.5 bg-emerald-50 hover:bg-emerald-100 text-emerald-800 font-bold rounded-xl text-xs border border-emerald-200 text-left transition flex items-center gap-2">🏥 保健室の先生へ</button>
                    </div>
                `;
            } else {
                inputContainer.innerHTML = `
                    <div class="flex flex-col gap-2">
                        <button onclick="selectDestination('ドラッグストアの薬剤師・登録販売者様へ')" class="py-2.5 px-3.5 bg-teal-50 hover:bg-teal-100 text-teal-800 font-bold rounded-xl text-xs border border-teal-200 text-left transition flex items-center gap-2">💊 ドラッグストアの薬剤師・登録販売者様へ</button>
                        <button onclick="selectDestination('保護者の方へ')" class="py-2.5 px-3.5 bg-teal-50 hover:bg-teal-100 text-teal-800 font-bold rounded-xl text-xs border border-teal-200 text-left transition flex items-center gap-2">👨‍👩‍👦 保護者の方へ</button>
                        <button onclick="selectDestination('学校の保健室の先生へ')" class="py-2.5 px-3.5 bg-teal-50 hover:bg-teal-100 text-teal-800 font-bold rounded-xl text-xs border border-teal-200 text-left transition flex items-center gap-2">🏫 学校の保健室の先生へ</button>
                    </div>
                `;
            }
        }

        function selectDestination(dest) {
            state.destination = dest;
            if (!state.finalNotes || state.finalNotes === '特になし') {
                if (dest.includes('薬剤師')) {
                    state.finalNotes = '店頭で言葉でうまく説明するのが少し苦手なためカードを作成しました。体調・年齢に合う市販薬のご確認をお願いいたします。';
                } else {
                    state.finalNotes = '自分の体調や気持ちを直接話すのが難しかったためカードに書きました。無理のない形で相談に乗ってもらえると嬉しいです。';
                }
            }
            addUserMessage(`${dest}`);
            generatePDFCard();
        }

        function getDrugRecommendations(symptomText, age, historyText, dopingText, currentMeds, allergies, pregnancy) {
            const symptom = (symptomText || '').toLowerCase();
            const history = (historyText || '').toLowerCase();
            const meds = (currentMeds || '').toLowerCase();
            const allergy = (allergies || '').toLowerCase();
            const doping = (dopingText || '').toLowerCase();
            const ageNum = parseInt(age, 10) || 15;

            let warnings = [];

            if (ageNum < 15) {
                warnings.push("⚠️ 15歳未満：ライ症候群リスクのあるアスピリン・ロキソプロフェン等は避ける（小児用アセトアミノフェン等を推奨）");
            }
            if (pregnancy && pregnancy.includes('あり')) {
                warnings.push("⚠️ 妊娠の可能性：NSAIDs（イブプロフェン等）の服用は避け、事前に必ず医師・薬剤師にご相談ください");
            }
            if (doping && doping.includes('あり')) {
                warnings.push("🏅 アンチ・ドーピング：エフェドリンやマオウ含有成分（総合感冒薬・一部漢方等）の服用に注意してください");
            }
            if (meds !== 'なし') {
                warnings.push(`💊 併用確認：現在服用中の「${escapeHtml(currentMeds)}」との重複成分・相互作用を店頭でご確認ください`);
            }
            if (allergy !== 'なし') {
                warnings.push(`⚠️ アレルギー確認：「${escapeHtml(allergies)}」に対する過敏症成分が含まれていないかご確認ください`);
            }

            if (warnings.length === 0) {
                warnings.push("✅ 特筆すべき即時警告事項なし。店頭の薬剤師・登録販売者様と一緒に症状に最適な薬をご確認ください。");
            }

            let category = "一般的な症状に応じたOTC医薬品（第2類・第3類医薬品）";
            let ingredients = "症状に合わせた標準有効成分（アセトアミノフェン、整腸菌、消炎成分等）";

            if (symptom.includes('頭痛') || symptom.includes('熱')) {
                category = "解熱鎮痛薬 / 総合感冒薬";
                ingredients = ageNum < 15 ? "アセトアミノフェン（小児用処方）" : "ロキソプロフェン、イブプロフェン、アセトアミノフェン";
            } else if (symptom.includes('腹痛') || symptom.includes('胃')) {
                category = "胃腸薬 / 胃腸鎮痛鎮痙薬";
                ingredients = "ファモチジン（H2ブロッカー）、ブチルスコポラミン、健胃生薬";
            } else if (symptom.includes('傷') || symptom.includes('擦り傷')) {
                category = "外用殺菌消毒薬 / 創傷被覆材";
                ingredients = "ベンザルコニウム塩化物、ハイドロコロイド素材";
            } else if (symptom.includes('捻挫') || symptom.includes('打撲')) {
                category = "外用消炎鎮痛薬（湿布・パップ剤）";
                ingredients = "フェルビナク、ジクロフェナク、サリチル酸グリコール";
            }

            return {
                category: category,
                ingredients: ingredients,
                warnings: warnings.join('<br>')
            };
        }

        async function fetchGeminiAdvice(promptText) {
            const apiKey = geminiApiKey || "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;
            const systemPrompt = "あなたは温かく親身な医療コミュニケーションの専門相談員です。診断は行わず、ユーザーがドラッグストアの薬剤師やスクールカウンセラーに相談する際に役立つ「配慮事項」や「伝え方のコツ」を要約して100文字以内で優しく箇条書きでアドバイスしてください。";

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: promptText }] }],
                        systemInstruction: { parts: [{ text: systemPrompt }] }
                    })
                });
                const result = await response.json();
                return result.candidates?.[0]?.content?.parts?.[0]?.text || null;
            } catch (err) {
                console.error("Gemini API Error:", err);
                return "・店頭や保健室でカードを提示する際は、一番つらい部分（" + (state.symptom || '症状') + "）を優先的にお伝えください。\n・薬剤師や先生に質問されたら、カードに記載の服薬・アレルギー情報をご確認いただくとスムーズです。";
            }
        }

        async function generatePDFCard() {
            addBotMessage("ご入力内容を整理してカードを作成しています...");
            updateStepIndicator("カード完成・プレビュー");

            const prompt = `相談モード: ${state.mode}, 症状/悩み: ${state.symptom}, 発症: ${state.onset}, つらさ: ${state.severity}/10, 年齢: ${state.ageGroup}, 服用薬: ${state.currentMeds}, アレルギー: ${state.allergies}, 配慮希望: ${state.expectations}`;
            const aiRes = await fetchGeminiAdvice(prompt);
            state.geminiAdvice = aiRes || "・カードを提示する際は、一番つらい部分を優先的にお伝えください。\n・必要に応じて市販薬や休養のアドバイスを求めてみましょう。";

            let cardContentHtml = '';

            if (state.mode === 'mental') {
                cardContentHtml = `
                    <div id="pdf-card" class="bg-white border-2 border-purple-600 p-3.5 sm:p-4 rounded-2xl text-slate-800 font-sans shadow-md text-xs space-y-3 relative">
                        <div class="border-b-2 border-purple-600 pb-2 flex justify-between items-start">
                            <div>
                                <div class="flex items-center gap-1.5">
                                    <span class="text-base">🌙</span>
                                    <h2 class="font-bold text-sm text-purple-900 tracking-wide">KOTOHANA 心の相談メモ</h2>
                                </div>
                                <div class="text-[11px] font-bold text-purple-700 mt-0.5">【宛先】 ${escapeHtml(state.destination)}</div>
                            </div>
                            <span class="text-[8px] bg-purple-50 text-purple-700 border border-purple-200 px-2 py-0.5 rounded-full font-bold">完全匿名</span>
                        </div>

                        <div class="bg-slate-50 p-2 rounded-xl border border-slate-100 text-[10.5px]">
                            <div><b>年齢区分:</b> ${escapeHtml(state.birthdate ? `${state.birthdate} (${state.ageGroup})` : state.ageGroup)}</div>
                        </div>

                        <div class="space-y-1 text-[10.5px]">
                            <div class="font-bold text-purple-800 border-b border-purple-100 pb-0.5">■ 気になる心の状態・悩み</div>
                            <div class="font-bold text-slate-900 bg-purple-50/50 p-2 rounded-lg border border-purple-100">${escapeHtml(state.symptom || '未記入')}</div>
                            <div><b>・継続期間:</b> ${escapeHtml(state.onset || '未記入')}</div>
                        </div>

                        <div class="bg-indigo-50 p-2 rounded-xl border border-indigo-200 text-[10px]">
                            <div class="font-bold text-indigo-900 flex items-center gap-1 mb-0.5">💡 相談サポート助言:</div>
                            <div class="text-indigo-950 whitespace-pre-wrap">${escapeHtml(state.geminiAdvice)}</div>
                        </div>

                        <div class="space-y-1 text-[10.5px]">
                            <div class="font-bold text-purple-800 border-b border-purple-100 pb-0.5">■ 日常生活への影響</div>
                            <div>${escapeHtml(state.impact || '特になし')}</div>
                        </div>

                        <div class="space-y-1 text-[10.5px]">
                            <div class="font-bold text-purple-800 border-b border-purple-100 pb-0.5">■ 希望するサポート・配慮</div>
                            <div>${escapeHtml(state.expectations || '特になし')}</div>
                        </div>

                        <div class="bg-purple-50/80 p-2.5 rounded-xl border border-purple-200 text-[10.5px]">
                            <div class="font-bold text-purple-900 mb-0.5">💬 伝えたいメッセージ:</div>
                            <div class="text-slate-700 italic">"${escapeHtml(state.finalNotes)}"</div>
                        </div>
                    </div>
                `;
            } else {
                const recs = getDrugRecommendations(state.symptom, state.calculatedAge || '15', state.history, state.doping, state.currentMeds, state.allergies, state.pregnancy);

                // 妊娠の可能性が「あり」の場合のみ表示するロジック
                const isPregnancyPresent = state.pregnancy && (state.pregnancy.includes('あり') || state.pregnancy.includes('可能性'));
                const pregnancyItemHtml = isPregnancyPresent 
                    ? `<div><b>・妊娠の可能性:</b> <span class="font-bold text-rose-700">${escapeHtml(state.pregnancy)}</span></div>` 
                    : '';

                const severityHtml = state.severity ? `<div><b>・つらさレベル:</b> <span class="font-bold text-teal-700 text-xs">${escapeHtml(state.severity)} / 10</span></div>` : '';

                cardContentHtml = `
                    <div id="pdf-card" class="bg-white border-2 border-teal-600 p-3.5 sm:p-4 rounded-2xl text-slate-800 font-sans shadow-md text-xs space-y-3 relative">
                        <div class="border-b-2 border-teal-600 pb-2 flex justify-between items-start">
                            <div>
                                <div class="flex items-center gap-1.5">
                                    <span class="text-base">🌱</span>
                                    <h2 class="font-bold text-sm text-teal-900 tracking-wide">KOTOHANA 医療・市販薬相談カード</h2>
                                </div>
                                <div class="text-[11px] font-bold text-teal-700 mt-0.5">【宛先】 ${escapeHtml(state.destination)}</div>
                            </div>
                            <span class="text-[8px] bg-teal-50 text-teal-700 border border-teal-200 px-2 py-0.5 rounded-full font-bold">完全匿名</span>
                        </div>

                        <div class="grid grid-cols-2 gap-1.5 bg-slate-50 p-2 rounded-xl border border-slate-100 text-[10px]">
                            <div><b>年齢区分:</b> ${escapeHtml(state.birthdate ? `${state.birthdate} (${state.ageGroup})` : state.ageGroup)}</div>
                            <div><b>体格目安:</b> 身長 ${escapeHtml(state.height)}cm / 体重 ${escapeHtml(state.weight)}kg</div>
                        </div>

                        <div class="space-y-1 text-[10.5px]">
                            <div class="font-bold text-teal-800 border-b border-teal-100 pb-0.5">■ 症状・経過</div>
                            <div><b>・症状・部位:</b> <span class="font-bold text-slate-900">${escapeHtml(state.symptom)}</span></div>
                            <div><b>・いつから:</b> ${escapeHtml(state.onset)}</div>
                            ${severityHtml}
                        </div>

                        <div class="bg-indigo-50 p-2 rounded-xl border border-indigo-200 text-[10px]">
                            <div class="font-bold text-indigo-900 flex items-center gap-1 mb-0.5">💡 受診・店頭相談サポート助言:</div>
                            <div class="text-indigo-950 whitespace-pre-wrap">${escapeHtml(state.geminiAdvice)}</div>
                        </div>

                        <div class="bg-emerald-50/90 border border-emerald-300 rounded-xl p-2.5 text-[10px] space-y-1">
                            <div class="font-bold text-emerald-900 border-b border-emerald-200 pb-0.5 flex justify-between">
                                <span>💊 店頭薬剤師・登録販売者様への確認シート</span>
                                <span class="text-[8px] bg-emerald-200 text-emerald-900 px-1 rounded">要確認</span>
                            </div>
                            <div><b>・分類目安:</b> ${recs.category}</div>
                            <div><b>・候補成分:</b> ${recs.ingredients}</div>
                            <div class="bg-white/90 p-1.5 rounded-lg border border-emerald-200 text-[9.5px]">
                                <b>・事前チェック警告事項:</b><br>
                                <span class="text-emerald-900 font-medium">${recs.warnings}</span>
                            </div>
                        </div>

                        <div class="space-y-1 text-[10px]">
                            <div class="font-bold text-teal-800 border-b border-teal-100 pb-0.5">■ 患者安全情報（事前申告）</div>
                            <div><b>・現在服用中の薬:</b> ${escapeHtml(state.currentMeds)}</div>
                            <div><b>・アレルギー:</b> ${escapeHtml(state.allergies)}</div>
                            <div><b>・持病・副作用歴:</b> ${escapeHtml(state.history)}</div>
                            ${pregnancyItemHtml}
                            <div><b>・アンチドーピング考慮:</b> <span class="font-bold text-amber-800">${escapeHtml(state.doping)}</span></div>
                            <div><b>・周囲への配慮希望:</b> ${escapeHtml(state.expectations)}</div>
                        </div>

                        <div class="bg-teal-50/70 p-2 rounded-xl border border-teal-100 text-[10.5px]">
                            <div class="font-bold text-teal-900 mb-0.5">💬 メッセージ:</div>
                            <div class="text-slate-700 italic">"${escapeHtml(state.finalNotes)}"</div>
                        </div>
                    </div>
                `;
            }

            inputContainer.innerHTML = `
                <div id="printable-card-area" class="max-h-[280px] sm:max-h-[340px] overflow-y-auto p-1 bg-slate-50 border border-slate-200 rounded-2xl">
                    ${cardContentHtml}
                </div>

                <div class="space-y-1.5 mt-2 no-print">
                    <button onclick="downloadPDF()" class="w-full py-2.5 bg-teal-700 hover:bg-teal-800 text-white font-bold rounded-xl text-xs shadow-md transition flex items-center justify-center gap-2 active:scale-98">
                        📄 KOTOHANAカード（PDF）を保存
                    </button>
                    <button onclick="printCard()" class="w-full py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold rounded-xl text-xs transition flex items-center justify-center gap-2 border border-slate-300 active:scale-98">
                        🖨️ 直接印刷 / ブラウザ保存
                    </button>
                    <button onclick="startInitialFlow()" class="w-full py-1.5 bg-white text-slate-500 font-medium rounded-xl text-[10px] text-center border border-slate-200">
                        🔄 別の内容で最初からやり直す
                    </button>
                </div>
            `;
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        window.downloadPDF = function() {
            const element = document.getElementById('pdf-card');
            const opt = {
                margin:       6,
                filename:     'KOTOHANA相談カード.pdf',
                image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2 },
                jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
            };
            html2pdf().from(element).set(opt).save();
        };

        window.printCard = function() {
            const cardElement = document.getElementById('pdf-card');
            if (!cardElement) return;

            const cardHtmlContent = cardElement.outerHTML;
            const printWindow = window.open('', '_blank', 'width=800,height=900');
            
            if (printWindow) {
                printWindow.document.open();
                printWindow.document.write(`
                    <!DOCTYPE html>
                    <html lang="ja">
                    <head>
                        <meta charset="UTF-8">
                        <title>KOTOHANAカード 印刷</title>
                        <script src="https://cdn.tailwindcss.com"><\/script>
                        <link href="https://fonts.googleapis.com/css2?family=Zen+Maru+Gothic:wght@400;500;700&display=swap" rel="stylesheet">
                        <style>
                            body { font-family: 'Zen Maru Gothic', sans-serif; padding: 20px; background: #ffffff; }
                            @media print { body { padding: 0; } .no-print { display: none !important; } }
                        </style>
                    </head>
                    <body class="flex flex-col items-center">
                        <div class="no-print w-full max-w-xl mb-4 flex justify-between items-center bg-teal-50 p-3 rounded-xl border border-teal-200">
                            <span class="text-xs text-teal-800 font-bold">🖨️ 印刷プレビュー</span>
                            <button onclick="window.print()" class="bg-teal-700 text-white font-bold px-4 py-1.5 rounded-lg text-xs">今すぐ印刷する</button>
                        </div>
                        <div class="w-full max-w-xl">${cardHtmlContent}</div>
                        <script>
                            window.onload = function() { setTimeout(function() { window.print(); }, 500); };
                        <\/script>
                    </body>
                    </html>
                `);
                printWindow.document.close();
            } else {
                window.print();
            }
        };

        function addBotMessage(text) {
            const div = document.createElement('div');
            div.className = "flex justify-start transition-all duration-300 transform translate-y-1";
            div.innerHTML = `
                <div class="flex items-start gap-2 max-w-[88%]">
                    <div class="w-7 h-7 rounded-full bg-teal-100 border border-teal-200 flex items-center justify-center text-xs flex-shrink-0 mt-0.5">🌱</div>
                    <div class="p-3.5 rounded-2xl rounded-tl-sm text-xs sm:text-sm bg-white text-slate-800 border border-teal-100 shadow-sm leading-relaxed whitespace-pre-wrap">${escapeHtml(text)}</div>
                </div>
            `;
            chatContainer.appendChild(div);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function addUserMessage(text) {
            const div = document.createElement('div');
            div.className = "flex justify-end transition-all duration-300 transform translate-y-1";
            div.innerHTML = `
                <div class="max-w-[85%] p-3.5 rounded-2xl rounded-tr-sm text-xs sm:text-sm bg-gradient-to-r from-teal-700 to-emerald-700 text-white shadow-md leading-relaxed whitespace-pre-wrap font-medium">${escapeHtml(text)}</div>
            `;
            chatContainer.appendChild(div);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function calculateAge(birthdateStr) {
            const birthDate = new Date(birthdateStr);
            if (isNaN(birthDate.getTime())) return null;
            const today = new Date();
            let age = today.getFullYear() - birthDate.getFullYear();
            const m = today.getMonth() - birthDate.getMonth();
            if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) age--;
            return age;
        }
    </script>
</body>
</html>
