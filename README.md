<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>メモミ</title>
<style>
    /* 全体設定・モダンホワイトUI */
    :root {
        --primary-color: #ffd700;
        --bg-color: #ffffff;
        --sidebar-bg: #fafafa;
        --text-color: #333333;
        --text-light: #888888;
        --border-color: #eaeaea;
        --error-color: #ff3b30;
        --hover-bg: #f0f0f0;
    }
    body, html {
        margin: 0; padding: 0; height: 100%; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        background-color: var(--bg-color); color: var(--text-color); overflow: hidden;
    }
    * { box-sizing: border-box; }
    button { cursor: pointer; border: none; outline: none; transition: 0.2s; font-family: inherit; }
    input { outline: none; border: 1px solid var(--border-color); padding: 12px; border-radius: 8px; font-size: 16px; width: 100%; margin-bottom: 12px; background: #fff; }
    
    /* ボタンデザイン (シンプルな黄色) */
    .btn { background: var(--primary-color); color: #000; padding: 12px; border-radius: 8px; width: 100%; font-size: 16px; font-weight: bold; margin-top: 10px; }
    .btn-secondary { background: #fff; color: #333; border: 1px solid var(--border-color); }
    .btn:hover { opacity: 0.8; }
    
    /* ログイン画面 */
    #auth-screen {
        display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh;
        background: var(--bg-color); position: absolute; top: 0; left: 0; width: 100%; z-index: 100;
    }
    .auth-box {
        text-align: center; max-width: 320px; width: 100%; padding: 40px 30px; border-radius: 16px;
        border: 1px solid var(--border-color); background: #fff;
    }
    .auth-icon { width: 80px; height: 80px; background: #000; border-radius: 16px; margin: 0 auto 20px; position: relative; display: flex; flex-direction: column; justify-content: center; align-items: center; gap: 4px; padding: 10px; }
    .auth-icon div { background: var(--primary-color); height: 12px; width: 100%; border-radius: 6px; }
    .auth-icon div:nth-child(n+2) { background: #333; }
    .error-msg { color: var(--error-color); font-size: 14px; margin-bottom: 10px; display: none; }
    .step { display: none; }
    .step.active { display: block; animation: fadeIn 0.3s; }

    /* メイン画面 */
    #app-screen { display: none; height: 100vh; display: flex; }
    
    /* サイドバー */
    .sidebar { width: 320px; background: var(--sidebar-bg); border-right: 1px solid var(--border-color); display: flex; flex-direction: column; }
    .sidebar-header { padding: 20px; display: flex; justify-content: space-between; align-items: center; }
    .sidebar-header h2 { margin: 0; font-size: 20px; font-weight: 600; }
    .new-note-btn { background: var(--primary-color); color: #000; width: 32px; height: 32px; border-radius: 50%; font-size: 20px; display: flex; align-items: center; justify-content: center; font-weight: bold; }
    
    .note-list-container { flex: 1; overflow-y: auto; padding: 0 10px 10px 10px; }
    .list-group-title { font-size: 12px; color: var(--text-light); margin: 20px 10px 8px 10px; font-weight: bold; }
    .note-item { padding: 12px 16px; border-radius: 8px; margin-bottom: 4px; cursor: pointer; background: transparent; transition: 0.1s; border-left: 4px solid transparent; }
    .note-item:hover { background: var(--hover-bg); }
    .note-item.active { background: var(--primary-color); color: #000; }
    .note-item.active .note-item-date, .note-item.active .note-item-preview { color: #333; }
    .note-item-title { font-size: 15px; font-weight: 600; margin-bottom: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-item-preview { font-size: 13px; color: var(--text-light); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-item-date { font-size: 12px; color: var(--text-light); margin-top: 4px; }
    
    .account-section { padding: 15px; border-top: 1px solid var(--border-color); display: flex; align-items: center; justify-content: space-between; background: var(--sidebar-bg); cursor: pointer; }
    .account-info { display: flex; flex-direction: column; }
    .account-name { font-weight: bold; font-size: 14px; }
    .account-label { font-size: 12px; color: var(--text-light); }

    /* メインエディタ */
    .editor-container { flex: 1; display: flex; flex-direction: column; background: #fff; }
    .toolbar { padding: 15px 30px; border-bottom: 1px solid var(--border-color); display: flex; justify-content: flex-end; align-items: center; gap: 10px; }
    .toolbar-btn { background: #fff; border: 1px solid var(--border-color); padding: 8px 16px; border-radius: 20px; font-size: 14px; font-weight: 500; display: flex; align-items: center; gap: 6px; }
    .toolbar-btn:hover { background: var(--hover-bg); }
    
    .editor-area { flex: 1; padding: 40px 60px; font-size: 16px; line-height: 1.8; outline: none; overflow-y: auto; }
    .editor-area img, .editor-area video { max-width: 100%; border-radius: 8px; margin: 10px 0; }
    
    /* 右クリックメニュー (コンテキストメニュー) */
    #context-menu { display: none; position: absolute; background: #fff; border: 1px solid var(--border-color); border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); z-index: 1000; min-width: 160px; overflow: hidden; }
    .context-item { padding: 12px 16px; font-size: 14px; cursor: pointer; display: flex; align-items: center; justify-content: space-between; }
    .context-item:hover { background: var(--hover-bg); }
    .context-item.danger { color: var(--error-color); border-top: 1px solid var(--border-color); }
    .color-picker { display: flex; gap: 5px; padding: 8px 16px; border-bottom: 1px solid var(--border-color); }
    .color-circle { width: 20px; height: 20px; border-radius: 50%; cursor: pointer; border: 1px solid var(--border-color); }

    /* モーダル (アカウント情報) */
    .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255,255,255,0.8); z-index: 200; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
    .modal-content { background: #fff; padding: 40px; border-radius: 16px; width: 340px; text-align: center; border: 1px solid var(--border-color); box-shadow: 0 10px 30px rgba(0,0,0,0.05); }
    .modal-content h3 { margin-top: 0; }

    @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
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
            <button class="btn" onclick="goToStep('step-id', 'login')">ログイン</button>
            <button class="btn btn-secondary" onclick="goToStep('step-id', 'register')">新規作成</button>
        </div>

        <div id="step-id" class="step">
            <p id="id-title" style="font-size: 14px; margin-bottom: 10px;">アカウント名を入力</p>
            <input type="text" id="account-id" placeholder="5文字以上">
            <p class="error-msg" id="id-error">5文字以上で入力してください</p>
            <button class="btn" onclick="validateId()">次へ</button>
            <button class="btn btn-secondary" onclick="goToStep('step-choice')">戻る</button>
        </div>

        <div id="step-pass" class="step">
            <p style="font-size: 14px; margin-bottom: 10px;">パスワードを入力</p>
            <input type="password" id="account-pass" placeholder="英数字を含む5文字以上">
            <p class="error-msg" id="pass-error">パスワードが間違っています</p>
            <button class="btn" onclick="submitAuth()">完了</button>
            <button class="btn btn-secondary" onclick="goToStep('step-id')">戻る</button>
        </div>
    </div>
</div>

<div id="app-screen">
    <div class="sidebar">
        <div class="sidebar-header">
            <h2>メモ</h2>
            <button class="new-note-btn" onclick="createNewNote()">+</button>
        </div>
        <div class="note-list-container" id="note-list-container">
            </div>
        <div class="account-section" onclick="showAccountInfo()">
            <div class="account-info">
                <span class="account-name" id="display-account-name">@user</span>
                <span class="account-label">ゆいきちアカウント</span>
            </div>
            <span style="color: var(--text-light); font-size: 20px;">...</span>
        </div>
    </div>

    <div class="editor-container">
        <div class="toolbar">
            <button class="toolbar-btn" onclick="systemShare()">共有</button>
            <button class="toolbar-btn" onclick="document.getElementById('file-upload').click()">ファイル・画像を追加</button>
            <input type="file" id="file-upload" style="display: none;" accept="*/*" onchange="handleFileUpload(event)">
        </div>
        <div class="editor-area" id="editor-area" contenteditable="true" oninput="saveCurrentNote()"></div>
    </div>
</div>

<div id="context-menu">
    <div class="color-picker">
        <div class="color-circle" style="background: transparent;" onclick="changeColor('')"></div>
        <div class="color-circle" style="background: #ffcccc;" onclick="changeColor('#ffcccc')"></div>
        <div class="color-circle" style="background: #ccffcc;" onclick="changeColor('#ccffcc')"></div>
        <div class="color-circle" style="background: #ccccff;" onclick="changeColor('#ccccff')"></div>
    </div>
    <div class="context-item" onclick="togglePin()" id="menu-pin-text">ピン留め</div>
    <div class="context-item danger" onclick="deleteNoteAction()">削除</div>
</div>

<div class="modal-overlay" id="account-modal">
    <div class="modal-content">
        <h3>アカウント情報</h3>
        <p style="font-size: 14px; color: var(--text-light); text-align: left; margin-bottom: 5px;">アカウント名</p>
        <p style="font-weight: bold; font-size: 18px; text-align: left; margin-top: 0;" id="info-id"></p>
        <p style="font-size: 14px; color: var(--text-light); text-align: left; margin-bottom: 5px;">パスワード</p>
        <p style="font-weight: bold; font-size: 18px; text-align: left; margin-top: 0;" id="info-pass"></p>
        <button class="btn" style="background: var(--error-color); color: #fff; margin-top: 20px;" onclick="logout()">ログアウト</button>
        <button class="btn btn-secondary" onclick="closeModal()">閉じる</button>
    </div>
</div>

<script>
    /* 状態管理 */
    let authMode = ''; 
    let currentId = '';
    let currentUser = null; 
    let notes = []; 
    let currentNoteId = null; 
    let contextMenuTargetId = null;

    /* --- 認証ロジック --- */
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
            document.getElementById('id-error').innerText = "既に存在します";
            document.getElementById('id-error').style.display = 'block';
            return;
        }
        if (authMode === 'login' && !accounts[idInput]) {
            document.getElementById('id-error').innerText = "見つかりません";
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
            const regex = /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{5,}$/;
            if(!regex.test(passInput)) {
                passError.innerText = "文字と数字を含む5桁以上にしてください";
                passError.style.display = 'block';
                return;
            }
            const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts')) || {};
            accounts[currentId] = { password: passInput, notes: [] };
            localStorage.setItem('yuikichi_accounts', JSON.stringify(accounts));
            loginSuccess(currentId);
        } else {
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
        document.getElementById('display-account-name').innerText = '@' + id;
        loadNotes();
    }
    function logout() {
        currentUser = null; currentNoteId = null;
        document.getElementById('editor-area').innerHTML = '';
        document.getElementById('app-screen').style.display = 'none';
        document.getElementById('auth-screen').style.display = 'flex';
        document.getElementById('account-id').value = '';
        document.getElementById('account-pass').value = '';
        closeModal(); goToStep('step-choice');
    }
    function showAccountInfo() {
        const accounts = JSON.parse(localStorage.getItem('yuikichi_accounts'));
        document.getElementById('info-id').innerText = currentUser;
        document.getElementById('info-pass').innerText = accounts[currentUser].password;
        document.getElementById('account-modal').style.display = 'flex';
    }
    function closeModal() { document.getElementById('account-modal').style.display = 'none'; }

    /* --- メモロジック --- */
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

    // HTMLからプレーンテキストを抽出してタイトルとプレビューを分ける
    function extractTextData(htmlContent) {
        const temp = document.createElement('div');
        temp.innerHTML = htmlContent;
        const text = temp.innerText.trim();
        const lines = text.split('\n').filter(line => line.trim() !== '');
        const title = lines.length > 0 ? lines[0] : '新規メモ';
        const preview = lines.length > 1 ? lines.slice(1).join(' ').substring(0, 30) : '追加テキストなし';
        return { title, preview };
    }

    // 日付カテゴリ判定
    function getDateCategory(timestamp) {
        const now = new Date();
        const date = new Date(timestamp);
        const diffDays = Math.floor((now - date) / (1000 * 60 * 60 * 24));
        if (diffDays === 0) return '今日';
        if (diffDays === 1) return '昨日';
        if (diffDays <= 7) return '過去7日間';
        if (diffDays <= 30) return '過去30日間';
        return date.getFullYear() + '年';
    }

    function renderNoteList() {
        const container = document.getElementById('note-list-container');
        container.innerHTML = '';
        
        notes.sort((a, b) => b.updatedAt - a.updatedAt);

        const groups = { 'ピン留め': [] };
        notes.forEach(note => {
            if (note.isPinned) {
                groups['ピン留め'].push(note);
            } else {
                const cat = getDateCategory(note.updatedAt);
                if (!groups[cat]) groups[cat] = [];
                groups[cat].push(note);
            }
        });

        for (const [category, groupNotes] of Object.entries(groups)) {
            if (groupNotes.length === 0) continue;

            const titleEl = document.createElement('div');
            titleEl.className = 'list-group-title';
            titleEl.innerText = category;
            container.appendChild(titleEl);

            groupNotes.forEach(note => {
                const { title, preview } = extractTextData(note.content);
                const dateStr = new Date(note.updatedAt).toLocaleTimeString('ja-JP', { hour: '2-digit', minute: '2-digit' });

                const div = document.createElement('div');
                div.className = `note-item ${note.id === currentNoteId ? 'active' : ''}`;
                if(note.color && note.id !== currentNoteId) div.style.backgroundColor = note.color;
                if(note.isPinned) div.style.borderLeftColor = 'var(--text-color)';
                
                div.onclick = () => selectNote(note.id);
                div.oncontextmenu = (e) => showContextMenu(e, note.id);

                div.innerHTML = `
                    <div class="note-item-title">${title}</div>
                    <div class="note-item-preview">${preview}</div>
                    <div class="note-item-date">${dateStr}</div>
                `;
                container.appendChild(div);
            });
        }
    }

    function createNewNote() {
        const newNote = {
            id: Date.now().toString(),
            content: '',
            updatedAt: Date.now(),
            isPinned: false,
            color: ''
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
        editor.innerHTML = note ? note.content : '';
        editor.focus();
        renderNoteList();
    }

    function saveCurrentNote() {
        if(!currentNoteId) return;
        const noteIndex = notes.findIndex(n => n.id === currentNoteId);
        if(noteIndex > -1) {
            notes[noteIndex].content = document.getElementById('editor-area').innerHTML;
            notes[noteIndex].updatedAt = Date.now();
            saveNotesToDB();
            renderNoteList();
        }
    }

    /* --- コンテキストメニュー (右クリック) --- */
    function showContextMenu(e, noteId) {
        e.preventDefault();
        contextMenuTargetId = noteId;
        const menu = document.getElementById('context-menu');
        const note = notes.find(n => n.id === noteId);
        
        document.getElementById('menu-pin-text').innerText = note.isPinned ? 'ピン留め解除' : 'ピン留め';
        
        menu.style.display = 'block';
        menu.style.left = `${e.pageX}px`;
        menu.style.top = `${e.pageY}px`;
    }

    document.addEventListener('click', () => {
        document.getElementById('context-menu').style.display = 'none';
    });

    function togglePin() {
        if(!contextMenuTargetId) return;
        const note = notes.find(n => n.id === contextMenuTargetId);
        note.isPinned = !note.isPinned;
        saveNotesToDB(); renderNoteList();
    }

    function deleteNoteAction() {
        if(!contextMenuTargetId) return;
        notes = notes.filter(n => n.id !== contextMenuTargetId);
        saveNotesToDB();
        if(currentNoteId === contextMenuTargetId) {
            currentNoteId = null;
            document.getElementById('editor-area').innerHTML = '';
            if(notes.length > 0) selectNote(notes[0].id);
        }
        renderNoteList();
    }

    function changeColor(colorHex) {
        if(!contextMenuTargetId) return;
        const note = notes.find(n => n.id === contextMenuTargetId);
        note.color = colorHex;
        saveNotesToDB(); renderNoteList();
    }

    /* --- 各種ファイル・メディア対応 --- */
    function handleFileUpload(event) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const result = e.target.result;
            const editor = document.getElementById('editor-area');
            editor.focus();

            if (file.type.startsWith('image/')) {
                document.execCommand('insertHTML', false, `<br><img src="${result}" alt="image"><br>`);
            } else if (file.type.startsWith('video/')) {
                document.execCommand('insertHTML', false, `<br><video src="${result}" controls></video><br>`);
            } else {
                // テキスト系ファイル
                const textContent = result.replace(/\n/g, '<br>');
                document.execCommand('insertHTML', false, `<br>--- ファイルプレビュー開始 ---<br>${textContent}<br>--- 終了 ---<br>`);
            }
            saveCurrentNote();
        };

        if (file.type.startsWith('image/') || file.type.startsWith('video/')) {
            reader.readAsDataURL(file); // 画像・動画はBase64で埋め込み
        } else {
            reader.readAsText(file); // その他はテキストとして読み込み
        }
        event.target.value = '';
    }

    /* --- システム共有 --- */
    function systemShare() {
        if(!currentNoteId) return;
        const note = notes.find(n => n.id === currentNoteId);
        const { title, preview } = extractTextData(note.content);

        if (navigator.share) {
            navigator.share({
                title: title,
                text: tempTextExtract(note.content)
            }).catch(console.error);
        } else {
            alert('お使いのブラウザはシステム共有機能に対応していません。');
        }
    }
    
    function tempTextExtract(html) {
        const temp = document.createElement('div');
        temp.innerHTML = html;
        return temp.innerText;
    }
</script>
</body>
</html>
