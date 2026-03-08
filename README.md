<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>メモミ</title>
<style>
    /* 全体設定・モダンUI */
    :root {
        --primary-color: #ffd700; /* アイコンの黄色をイメージ */
        --bg-color: #f5f5f7;
        --sidebar-bg: #f5f5f5;
        --text-color: #1d1d1f;
        --border-color: #d2d2d7;
        --error-color: #ff3b30;
    }
    body, html {
        margin: 0; padding: 0; height: 100%; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        background-color: var(--bg-color); color: var(--text-color); overflow: hidden;
    }
    * { box-sizing: border-box; }
    button { cursor: pointer; border: none; outline: none; transition: 0.2s; }
    input { outline: none; border: 1px solid var(--border-color); padding: 10px; border-radius: 8px; font-size: 16px; width: 100%; margin-bottom: 10px; }
    
    /* ログイン画面 (ゆいきちアカウント) */
    #auth-screen {
        display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh;
        background: white; position: absolute; top: 0; left: 0; width: 100%; z-index: 100;
    }
    .auth-box {
        text-align: center; max-width: 320px; width: 100%; padding: 30px; border-radius: 16px;
        box-shadow: 0 4px 24px rgba(0,0,0,0.1); background: white;
    }
    .auth-icon { width: 80px; height: 80px; background: #333; border-radius: 16px; margin: 0 auto 20px; position: relative; display: flex; flex-direction: column; justify-content: center; align-items: center; gap: 4px; padding: 10px; }
    .auth-icon div { background: var(--primary-color); height: 12px; width: 100%; border-radius: 6px; }
    .auth-icon div:nth-child(n+2) { background: #555; }
    
    .btn { background: #007aff; color: white; padding: 12px; border-radius: 8px; width: 100%; font-size: 16px; font-weight: bold; margin-top: 10px; }
    .btn-secondary { background: #e5e5ea; color: black; }
    .btn:hover { opacity: 0.8; }
    .error-msg { color: var(--error-color); font-size: 14px; margin-bottom: 10px; display: none; }
    .step { display: none; }
    .step.active { display: block; animation: fadeIn 0.3s; }

    /* メイン画面 (Appleメモ風) */
    #app-screen { display: none; height: 100vh; display: flex; }
    
    /* サイドバー */
    .sidebar { width: 300px; background: var(--sidebar-bg); border-right: 1px solid var(--border-color); display: flex; flex-direction: column; }
    .sidebar-header { padding: 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border-color); }
    .sidebar-header h2 { margin: 0; font-size: 18px; }
    .new-note-btn { background: none; color: #007aff; font-size: 24px; padding: 0 10px; }
    .note-list { flex: 1; overflow-y: auto; padding: 10px; }
    .note-item { padding: 12px; border-radius: 8px; margin-bottom: 5px; cursor: pointer; background: transparent; }
    .note-item:hover { background: #e5e5ea; }
    .note-item.active { background: #d1d1d6; font-weight: bold; }
    .note-item-title { font-size: 15px; margin-bottom: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-item-date { font-size: 12px; color: #86868b; }

    /* メインエディタ */
    .editor-container { flex: 1; display: flex; flex-direction: column; background: white; }
    .toolbar { padding: 10px 20px; border-bottom: 1px solid var(--border-color); display: flex; justify-content: space-between; align-items: center; }
    .toolbar-left { display: flex; gap: 10px; }
    .toolbar-right { display: flex; gap: 10px; }
    .editor-area { flex: 1; padding: 40px; font-size: 16px; line-height: 1.6; outline: none; border: none; resize: none; width: 100%; }

    /* モーダル (アカウント情報) */
    .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.4); z-index: 200; align-items: center; justify-content: center; }
    .modal-content { background: white; padding: 30px; border-radius: 16px; width: 300px; text-align: center; }

    @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
</style>
</head>
<body>

<div id="auth-screen">
    <div class="auth-box">
        <div class="auth-icon">
            <div></div><div></div><div></div><div></div>
        </div>
        <h2>ゆいきちアカウント</h2>
        
        <div id="step-choice" class="step active">
            <p>メモミへようこそ</p>
            <button class="btn" onclick="goToStep('step-id', 'login')">ログイン</button>
            <button class="btn btn-secondary" onclick="goToStep('step-id', 'register')">新しいアカウントを作成</button>
        </div>

        <div id="step-id" class="step">
            <p id="id-title">アカウント名を入力</p>
            <input type="text" id="account-id" placeholder="5文字以上のアカウント名">
            <p class="error-msg" id="id-error">5文字以上で入力してください</p>
            <button class="btn" onclick="validateId()">次へ</button>
            <button class="btn btn-secondary" style="margin-top: 10px;" onclick="goToStep('step-choice')">戻る</button>
        </div>

        <div id="step-pass" class="step">
            <p>パスワードを入力</p>
            <input type="password" id="account-pass" placeholder="英数字を含む5文字以上">
            <p class="error-msg" id="pass-error">パスワードが間違っています</p>
            <button class="btn" onclick="submitAuth()">完了</button>
            <button class="btn btn-secondary" style="margin-top: 10px;" onclick="goToStep('step-id')">戻る</button>
        </div>
    </div>
</div>

<div id="app-screen">
    <div class="sidebar">
        <div class="sidebar-header">
            <h2>メモミ</h2>
            <button class="new-note-btn" onclick="createNewNote()">📝</button>
        </div>
        <div class="note-list" id="note-list">
            </div>
    </div>

    <div class="editor-container">
        <div class="toolbar">
            <div class="toolbar-left">
                <button class="btn btn-secondary" onclick="document.getElementById('file-upload').click()" style="padding: 6px 12px; font-size: 14px;">📄 ファイルをテキストで挿入</button>
                <input type="file" id="file-upload" style="display: none;" accept=".txt,.csv,.json,.md,.html" onchange="handleFileUpload(event)">
            </div>
            <div class="toolbar-right">
                <button class="btn btn-secondary" style="padding: 6px 12px; font-size: 14px;" onclick="showAccountInfo()">👤 アカウント</button>
            </div>
        </div>
        <textarea class="editor-area" id="editor-area" placeholder="ここにメモを入力..." oninput="saveCurrentNote()"></textarea>
    </div>
</div>

<div class="modal-overlay" id="account-modal">
    <div class="modal-content">
        <h3>ゆいきちアカウント情報</h3>
        <p><strong>アカウント名:</strong> <span id="info-id"></span></p>
        <p><strong>パスワード:</strong> <span id="info-pass"></span></p>
        <button class="btn" onclick="logout()" style="background: var(--error-color);">ログアウト</button>
        <button class="btn btn-secondary" onclick="closeModal()">閉じる</button>
    </div>
</div>

<script>
    /* --- 状態管理 --- */
    let authMode = ''; // 'login' or 'register'
    let currentId = '';
    let currentUser = null; // ログイン中のユーザーID
    let notes = []; // メモデータ
    let currentNoteId = null; // 編集中のメモID

    /* --- 認証ロジック (ゆいきちアカウント) --- */
    function goToStep(stepId, mode = null) {
        if(mode) authMode = mode;
        document.querySelectorAll('.step').forEach(el => el.classList.remove('active'));
        document.getElementById(stepId).classList.add('active');
        hideErrors();
    }

    function hideErrors() {
        document.getElementById('id-error').style.display = 'none';
        document.getElementById('pass-error').style.display = 'none';
    }

    function validateId() {
        const idInput = document.getElementById('account-id').value.trim();
        if(idInput.length < 5) {
            document.getElementById('id-error').innerText = "5文字以上で入力してください";
            document.getElementById('id-error').style.display = 'block';
            return;
        }
        
        const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts')) || {};
        if (authMode === 'register' && accounts[idInput]) {
            document.getElementById('id-error').innerText = "このアカウントは既に存在します";
            document.getElementById('id-error').style.display = 'block';
            return;
        }
        if (authMode === 'login' && !accounts[idInput]) {
            document.getElementById('id-error').innerText = "アカウントが見つかりません";
            document.getElementById('id-error').style.display = 'block';
            return;
        }

        currentId = idInput;
        goToStep('step-pass');
    }

    function submitAuth() {
        const passInput = document.getElementById('account-pass').value;
        const passError = document.getElementById('pass-error');
        
        if(authMode === 'register') {
            // 英数字混合で5文字以上かチェック
            const regex = /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{5,}$/;
            if(!regex.test(passInput)) {
                passError.innerText = "パスワードは文字と数字を合わせて5桁以上にしてください";
                passError.style.display = 'block';
                return;
            }
            // 登録処理
            const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts')) || {};
            accounts[currentId] = { password: passInput, notes: [] };
            localStorage.setItem('yuikichi_accounts', JSON.stringify(accounts));
            loginSuccess(currentId);
        } else {
            // ログイン処理
            const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts')) || {};
            if(accounts[currentId].password !== passInput) {
                passError.innerText = "パスワードが間違っています";
                passError.style.display = 'block';
                return;
            }
            loginSuccess(currentId);
        }
    }

    function loginSuccess(id) {
        currentUser = id;
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('app-screen').style.display = 'flex';
        loadNotes();
    }

    function logout() {
        currentUser = null;
        currentNoteId = null;
        document.getElementById('editor-area').value = '';
        document.getElementById('app-screen').style.display = 'none';
        document.getElementById('auth-screen').style.display = 'flex';
        document.getElementById('account-id').value = '';
        document.getElementById('account-pass').value = '';
        closeModal();
        goToStep('step-choice');
    }

    /* --- アカウント情報 --- */
    function showAccountInfo() {
        const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts'));
        document.getElementById('info-id').innerText = currentUser;
        document.getElementById('info-pass').innerText = accounts[currentUser].password;
        document.getElementById('account-modal').style.display = 'flex';
    }

    function closeModal() {
        document.getElementById('account-modal').style.display = 'none';
    }

    /* --- メモのロジック --- */
    function loadNotes() {
        const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts'));
        notes = accounts[currentUser].notes || [];
        renderNoteList();
        if(notes.length > 0) {
            selectNote(notes[0].id);
        } else {
            createNewNote();
        }
    }

    function saveNotesToDB() {
        const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts'));
        accounts[currentUser].notes = notes;
        localStorage.setItem('yuikichi_accounts', JSON.stringify(accounts));
    }

    function renderNoteList() {
        const listEl = document.getElementById('note-list');
        listEl.innerHTML = '';
        
        // 更新日時でソート
        notes.sort((a, b) => b.updatedAt - a.updatedAt);

        notes.forEach(note => {
            const div = document.createElement('div');
            div.className = `note-item ${note.id === currentNoteId ? 'active' : ''}`;
            div.onclick = () => selectNote(note.id);
            
            const title = note.content.split('\n')[0].trim() || '新規メモ';
            const date = new Date(note.updatedAt).toLocaleString('ja-JP', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' });

            div.innerHTML = `
                <div class="note-item-title">${title}</div>
                <div class="note-item-date">${date}</div>
            `;
            listEl.appendChild(div);
        });
    }

    function createNewNote() {
        const newNote = {
            id: Date.now().toString(),
            content: '',
            updatedAt: Date.now()
        };
        notes.unshift(newNote);
        saveNotesToDB();
        renderNoteList();
        selectNote(newNote.id);
    }

    function selectNote(id) {
        currentNoteId = id;
        const note = notes.find(n => n.id === id);
        const editor = document.getElementById('editor-area');
        editor.value = note ? note.content : '';
        editor.focus();
        renderNoteList();
    }

    function saveCurrentNote() {
        if(!currentNoteId) return;
        const noteIndex = notes.findIndex(n => n.id === currentNoteId);
        if(noteIndex > -1) {
            notes[noteIndex].content = document.getElementById('editor-area').value;
            notes[noteIndex].updatedAt = Date.now();
            saveNotesToDB();
            renderNoteList();
        }
    }

    /* --- ファイル読み込み機能 --- */
    function handleFileUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const textContent = e.target.result;
            const editor = document.getElementById('editor-area');
            
            // 現在のカーソル位置、または末尾にテキストを挿入
            const cursorPos = editor.selectionStart;
            const textBefore = editor.value.substring(0, cursorPos);
            const textAfter = editor.value.substring(cursorPos, editor.value.length);
            
            editor.value = textBefore + "\n--- ファイルプレビュー開始 ---\n" + textContent + "\n--- ファイルプレビュー終了 ---\n" + textAfter;
            
            // 保存処理
            saveCurrentNote();
        };
        // テキストとしてファイルを読み込む
        reader.readAsText(file);
        // inputをリセットして同じファイルを再度選べるようにする
        event.target.value = '';
    }
</script>
</body>
</html>
