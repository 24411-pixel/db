<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DesignAI - 제품 디자이너를 위한 아이디어 파트너</title>
    <style>
        :root {
            --primary: #2563eb;
            --bg: #f8fafc;
            --card: #ffffff;
            --text: #1e293b;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
        }
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #e2e8f0;
            padding-bottom: 20px;
            margin-bottom: 30px;
        }
        .card {
            background: var(--card);
            padding: 24px;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            margin-bottom: 20px;
        }
        .hidden { display: none !important; }
        input, textarea, select {
            width: 100%;
            padding: 10px;
            margin: 8px 0 16px 0;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            box-sizing: border-box;
        }
        button {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover { opacity: 0.9; }
        .btn-secondary { background-color: #64748b; }
        .btn-speak { background-color: #10b981; }
        .idea-item {
            border-left: 4px solid var(--primary);
            padding-left: 15px;
            margin-bottom: 15px;
            background: #f1f5f9;
            padding: 10px;
            border-radius: 0 6px 6px 0;
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>💡 DesignAI</h1>
        <div id="auth-status"></div>
    </header>

    <div id="auth-section" class="card">
        <h3>🔒 디자이너 로그인</h3>
        <p>인공지능 가이드 및 아이디어 저장을 위해 회원가입 후 로그인해주세요.</p>
        <input type="email" id="auth-email" placeholder="이메일 주소 (예: designer@email.com)">
        <input type="password" id="auth-password" placeholder="비밀번호 (6자리 이상)">
        <button id="btn-login">로그인</button>
        <button id="btn-signup" class="btn-secondary">회원가입</button>
    </div>

    <div id="main-section" class="hidden">
        <div class="card">
            <h3>🎨 AI 제품 디자인 컨셉 Generator</h3>
            
            <label for="product-type"><b>개발할 제품군</b></label>
            <input type="text" id="product-type" placeholder="예: 친환경 텀블러, 스마트 워치, 1인 가구용 공기청정기">

            <label for="design-concept"><b>디자인 핵심 키워드/방향성</b></label>
            <textarea id="design-concept" rows="3" placeholder="예: 미니멀리즘, 생분해성 소재 활용, CMF 강조"></textarea>

            <button id="btn-generate">AI 디자인 컨셉 생성</button>
        </div>

        <div id="result-card" class="card hidden">
            <h3>✨ AI 제안 디자인 컨셉</h3>
            <p id="ai-response-text" style="white-space: pre-wrap; line-height: 1.6;"></p>
            <div style="margin-top: 15px;">
                <button id="btn-tts" class="btn-speak">🔊 오디오로 듣기 (TTS)</button>
                <button id="btn-save" class="btn-secondary">📁 내 아이디어 보관함에 저장</button>
            </div>
        </div>

        <div class="card">
            <h3>📁 저장된 디자인 아이디어 아카이브</h3>
            <div id="idea-history-list">
                <p style="color: #94a3b8;">저장된 아이디어가 없습니다.</p>
            </div>
        </div>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";
    import { getFirestore, collection, addDoc, query, where, onSnapshot } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

    // 🔗 사용자가 제공한 파이어베이스 정보 적용 완료!
    const firebaseConfig = {
        apiKey: "AIzaSyAs4HREbH7G8U-i2-DCOvM_hD0mX1Uq8M",
        authDomain: "aiweb-2c933.firebaseapp.com",
        projectId: "aiweb-2c933",
        storageBucket: "aiweb-2c933.firebasestorage.app",
        messagingSenderId: "1071279768652",
        appId: "1:1071279768652:web:dd88e5d6ff73df7212fe57"
    };

    // ⚠️ [필수] 구글 AI 스튜디오에서 발급받은 Gemini API 키를 여기에 넣어주세요!
    const GEMINI_API_KEY = "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";

    // Firebase 초기화
    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    let currentUser = null;
    let latestAIResult = "";

    const authSection = document.getElementById('auth-section');
    const mainSection = document.getElementById('main-section');
    const authStatus = document.getElementById('auth-status');

    // --- [1] Firebase Authentication (인증 시스템) ---
    document.getElementById('btn-signup').addEventListener('click', async () => {
        const email = document.getElementById('auth-email').value;
        const password = document.getElementById('auth-password').value;
        if(!email || !password) { alert('이메일과 비밀번호를 입력해주세요.'); return; }
        try {
            await createUserWithEmailAndPassword(auth, email, password);
            alert('회원가입이 완료되었습니다! 자동으로 로그인 처리됩니다.');
        } catch (error) {
            console.error(error);
            alert('회원가입 실패: ' + error.message);
        }
    });

    document.getElementById('btn-login').addEventListener('click', async () => {
        const email = document.getElementById('auth-email').value;
        const password = document.getElementById('auth-password').value;
        if(!email || !password) { alert('이메일과 비밀번호를 입력해주세요.'); return; }
        try {
            await signInWithEmailAndPassword(auth, email, password);
        } catch (error) {
            console.error(error);
            alert('로그인 실패: 계정 정보를 다시 확인하거나 Firebase Authentication 설정을 점검하세요.\n(' + error.message + ')');
        }
    });

    onAuthStateChanged(auth, (user) => {
        if (user) {
            currentUser = user;
            authSection.classList.add('hidden');
            mainSection.classList.remove('hidden');
            authStatus.innerHTML = `<span>디자이너: <b>${user.email}</b></span> <button id="btn-logout" class="btn-secondary" style="padding:5px 10px; margin-left:10px;">로그아웃</button>`;
            document.getElementById('btn-logout').addEventListener('click', () => signOut(auth));
            loadSavedIdeas(user.uid);
        } else {
            currentUser = null;
            authSection.classList.remove('hidden');
            mainSection.classList.add('hidden');
            authStatus.innerHTML = '';
        }
    });

    // --- [2] 생성형 AI 연동 (Gemini API) ---
    document.getElementById('btn-generate').addEventListener('click', async () => {
        const productType = document.getElementById('product-type').value;
        const designConcept = document.getElementById('design-concept').value;

        if(!productType || !designConcept) {
            alert('제품군과 디자인 방향성을 모두 적어주세요!');
            return;
        }

        document.getElementById('btn-generate').innerText = "AI 아이디어 스케칭 중...";
        
        const prompt = `너는 세계적인 제품 디자이너야. 제품군: [${productType}], 컨셉 방향성: [${designConcept}]에 부합하는 혁신적이고 심미적인 제품 디자인 컨셉 제안서를 작성해줘. 제안서에는 1. 제품 디자인 컨셉 명칭, 2. 형태 및 구조적 특징, 3. 추천 CMF(컬러, 소재, 마감)가 꼭 포함되게 전문적이고 매끄러운 한국어로 작성해줘.`;

        try {
            const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_API_KEY}`, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
            });

            if (!response.ok) {
                const errorData = await response.json();
                console.error("Gemini API Error Detail:", errorData);
                throw new Error(`상태코드 ${response.status}: ${errorData.error?.message || '알 수 없는 API 오류'}`);
            }

            const data = await response.json();
            latestAIResult = data.candidates[0].content.parts[0].text;
            
            document.getElementById('ai-response-text').innerText = latestAIResult;
            document.getElementById('result-card').classList.remove('hidden');
        } catch (error) {
            console.error("AI 에러 내용:", error);
            alert(`AI 연동 중 에러가 발생했습니다.\n\n[원인]: ${error.message}\n\n상단의 GEMINI_API_KEY 변수에 유효한 키가 들어갔는지 확인하세요.`);
        } finally {
            document.getElementById('btn-generate').innerText = "AI 디자인 컨셉 생성";
        }
    });

    // --- [3] TTS (Text-to-Speech) 기능 ---
    document.getElementById('btn-tts').addEventListener('click', () => {
        if (!latestAIResult) return;
        window.speechSynthesis.cancel(); 
        const utterance = new SpeechSynthesisUtterance(latestAIResult);
        utterance.lang = 'ko-KR';
        utterance.rate = 1.0;
        window.speechSynthesis.speak(utterance);
    });

    // --- [4] Firebase Firestore 데이터 아카이빙 연동 ---
    document.getElementById('btn-save').addEventListener('click', async () => {
        if (!currentUser || !latestAIResult) return;
        const productType = document.getElementById('product-type').value;

        try {
            await addDoc(collection(db, "design_ideas"), {
                userId: currentUser.uid,
                productType: productType,
                ideaText: latestAIResult,
                createdAt: new Date()
            });
            alert('아이디어 아카이브에 성공적으로 저장되었습니다!');
        } catch (e) {
            console.error("Firestore 저장 실패:", e);
            alert("저장 실패: Firestore 보안 규칙이 '테스트 모드'로 되어 있는지 콘솔에서 확인하세요.");
        }
    });

    function loadSavedIdeas(userId) {
        const q = query(collection(db, "design_ideas"), where("userId", "==", userId));
        onSnapshot(q, (querySnapshot) => {
            const listContainer = document.getElementById('idea-history-list');
            listContainer.innerHTML = '';

            if(querySnapshot.empty) {
                listContainer.innerHTML = '<p style="color: #94a3b8;">저장된 아이디어가 없습니다.</p>';
                return;
            }

            querySnapshot.forEach((doc) => {
                const data = doc.data();
                const itemDiv = document.createElement('div');
                itemDiv.className = 'idea-item';
                itemDiv.innerHTML = `
                    <strong>📦 제품군: ${data.productType}</strong>
                    <p style="font-size: 14px; color: #475569; white-space: pre-wrap;">${data.ideaText}</p>
                `;
                listContainer.appendChild(itemDiv);
            });
        }, (error) => {
            console.error("Firestore 조회 실패:", error);
        });
    }
</script>
</body>
</html>
