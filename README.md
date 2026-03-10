<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>メモミ</title>
<style>
    /* iOS風 モダン変数 */
    :root {
        --primary: #FFCC00; /* メモミの黄色 */
        --bg-main: #F2F2F7;
        --bg-sheet: #FFFFFF;
        --text-main: #000000;
        --text-sec: #8A8A8E;
        --separator: #C6C6C8;
        --danger: #FF3B30;
        --blue: #007AFF;
    }

    body, html {
        margin: 0; padding: 0; height: 100%;
        font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", "Segoe UI", sans-serif;
        background-color: var(--bg-main); color: var(--text-main);
        overflow: hidden; -webkit-tap-highlight-color: transparent;
        -webkit-user-select: none; user-select: none; /* 長押し時のデフォルト選択を防ぐ */
    }
    * { box-sizing: border-box; }
    button { cursor: pointer; border: none; outline: none; background: transparent; font-family: inherit; padding: 0; }
    input { outline: none; border: none; background: transparent; font-family: inherit; }

    /* --- アイコン (SVG) --- */
    .icon { width: 24px; height: 24px; fill: var(--primary); }
    .icon-sm { width: 18px; height: 18px; fill: var(--text-sec); }

    /* --- 画面コンテナ --- */
    .screen {
        display: none; height: 100%; width: 100%; position: absolute; top: 0; left: 0;
        flex-direction: column; background: var(--bg-main);
        transition: transform 0.4s cubic-bezier(0.32, 0.72, 0, 1);
    }
    .screen.active { display: flex; transform: translateX(0); z-index: 10; }
    .screen.hidden-left { transform: translateX(-30%); z-index: 5; }
    .screen.hidden-right { transform: translateX(100%); z-index: 20; }

    /* --- ナビゲーションバー (ヘッダー) --- */
    .nav-bar {
        display: flex; justify-content: space-between; align-items: flex-end;
        padding: 10px 16px 12px 16px; min-height: 80px;
        background: rgba(242, 242, 247, 0.85); backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px);
        position: fixed; top: 0; left: 0; right: 0; z-index: 50;
        border-bottom: 0.5px solid rgba(0,0,0,0.1);
    }
    .nav-left, .nav-right { display: flex; align-items: center; gap: 16px; }
    .nav-btn { color: var(--primary); font-size: 17px; font-weight: 400; display: flex; align-items: center; gap: 4px; }
    
    /* --- ホーム画面 --- */
    #home-screen { padding-top: 80px; }
    .large-title { font-size: 34px; font-weight: 700; padding: 8px 16px; margin: 0; letter-spacing: -0.5px; }
    
    .search-container { padding: 4px 16px 16px 16px; }
    .search-box {
        background: #E3E3E8; border-radius: 10px; padding: 8px 12px;
        display: flex; align-items: center; gap: 8px;
    }
    .search-box input { width: 100%; font-size: 17px; color: var(--text-main); }
    .search-box input::placeholder { color: var(--text-sec); }

    .note-list-container { flex: 1; overflow-y: auto; padding: 0 16px 100px 16px; }
    .list-group { background: var(--bg-sheet); border-radius: 10px; margin-bottom: 24px; overflow: hidden; }
    .group-title { font-size: 20px; font-weight: 600; padding: 16px 16px 8px 16px; margin: 0; }
    
    .note-item {
        padding: 12px 16px; border-bottom: 0.5px solid var(--separator);
        display: flex; flex-direction: column; gap: 2px; transition: background 0.2s;
    }
    .note-item:last-child { border-bottom: none; }
    .note-item:active { background: #E5E5EA; }
    
    .note-title { font-size: 16px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-preview-row { display: flex; align-items: center; gap: 8px; font-size: 15px; }
    .note-date { color: var(--text-sec); flex-shrink: 0; }
    .note-preview { color: var(--text-sec); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

    /* ツールバー (画面下部) */
    .bottom-toolbar {
        position: fixed; bottom: 0; left: 0; right: 0; height: 80px;
        background: rgba(242, 242, 247, 0.85); backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px); border-top: 0.5px solid rgba(0,0,0,0.1);
        display: flex; justify-content: space-between; align-items: center; padding: 0 20px 20px 20px; z-index: 50;
    }
    .note-count { font-size: 13px; color: var(--text-main); font-weight: 500; }

    /* --- エディタ画面 --- */
    #editor-screen { background: var(--bg-sheet); padding-top: 80px; }
    .editor-nav { background: rgba(255, 255, 255, 0.85); }
    .editor-area {
        flex: 1; padding: 16px 20px 150px 20px; font-size: 17px; line-height: 1.5;
        outline: none; overflow-y: auto; -webkit-user-select: text; user-select: text;
    }
    .editor-area img, .editor-area video { max-width: 100%; border-radius: 12px; margin: 12px 0; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
    
    /* リッチテキストキーボードツールバー */
    .keyboard-toolbar {
        position: fixed; bottom: 0; left: 0; right: 0; background: #F2F2F7;
        border-top: 0.5px solid var(--separator); padding: 10px 16px;
        display: flex; justify-content: space-between; align-items: center;
        overflow-x: auto; z-index: 60; padding-bottom: max(10px, env(safe-area-inset-bottom));
    }
    .kb-btn { padding: 8px 12px; font-size: 16px; font-weight: 600; color: var(--text-main); }
    .kb-btn:active { opacity: 0.5; }

    /* --- アクションメニュー (長押し・詳細) --- */
    .overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.2); z-index: 100; opacity: 0; transition: opacity 0.3s; }
    .action-sheet {
        position: fixed; bottom: -100%; left: 10px; right: 10px;
        z-index: 101; transition: bottom 0.4s cubic-bezier(0.32, 0.72, 0, 1);
        display: flex; flex-direction: column; gap: 8px; padding-bottom: 20px;
    }
    .action-group { background: rgba(255,255,255,0.95); backdrop-filter: blur(20px); border-radius: 14px; overflow: hidden; }
    .action-item {
        padding: 16px 20px; font-size: 17px; text-align: center;
        border-bottom: 0.5px solid rgba(0,0,0,0.1); color: var(--blue); font-weight: 400;
        display: flex; justify-content: space-between; align-items: center;
    }
    .action-item:last-child { border-bottom: none; }
    .action-item:active { background: rgba(0,0,0,0.05); }
    .action-item.danger { color: var(--danger); }
    .action-item.cancel { font-weight: 600; justify-content: center; }

    /* カラーパレット */
    .color-row { display: flex; justify-content: space-between; padding: 16px 20px; background: rgba(255,255,255,0.95); border-radius: 14px; margin-bottom: 8px; }
    .color-dot { width: 32px; height: 32px; border-radius: 50%; border: 1px solid rgba(0,0,0,0.1); transition: transform 0.2s; }
    .color-dot:active { transform: scale(1.2); }

    /* 選択アニメーション */
    .haptic-pop { animation: pop 0.2s ease-out; }
    @keyframes pop { 0% { transform: scale(1); } 50% { transform: scale(0.95); } 100% { transform: scale(1); } }
</style>
</head>
<body>

<svg style="display:none;">
    <symbol id="icon-back" viewBox="0 0 24 24"><path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6 1.41-1.41z"/></symbol>
    <symbol id="icon-compose" viewBox="0 0 24 24"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></symbol>
    <symbol id="icon-share" viewBox="0 0 24 24"><path d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.05-4.11c.54.5 1.25.81 2.04.81 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.04 9.81C7.5 9.31 6.79 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.79 0 1.5-.31 2.04-.81l7.12 4.16c-.05.21-.08.43-.08.65 0 1.61 1.31 2.92 2.92 2.92 1.61 0 2.92-1.31 2.92-2.92s-1.31-2.92-2.92-2.92z"/></symbol>
    <symbol id="icon-more" viewBox="0 0 24 24"><path d="M12 8c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zm0 2c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0 6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2z"/></symbol>
    <symbol id="icon-camera" viewBox="0 0 24 24"><path d="M9 2L7.17 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2h-3.17L15 2H9zm3 15c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5z"/></symbol>
    <symbol id="icon-search" viewBox="0 0 24 24"><path d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></symbol>
</svg>

<div id="home-screen" class="screen active">
    <div class="nav-bar">
        <div class="nav-left"></div>
        <div class="nav-right">
            <button class="nav-btn" onclick="document.getElementById('file-import').click()">読込</button>
            <input type="file" id="file-import" style="display: none;" accept="*/*" onchange="handleFileUpload(event, true)">
        </div>
    </div>
    
    <h1 class="large-title">メモ</h1>
    
    <div class="search-container">
        <div class="search-box">
            <svg class="icon-sm"><use href="#icon-search"></use></svg>
            <input type="text" id="search-input" placeholder="検索" oninput="filterNotes()">
        </div>
    </div>

    <div class="note-list-container" id="note-list">
        </div>

    <div class="bottom-toolbar">
        <div style="width: 24px;"></div> <span class="note-count" id="note-count-text">0 件のメモ</span>
        <button onclick="createNewNote()"><svg class="icon"><use href="#icon-compose"></use></svg></button>
    </div>
</div>

<div id="editor-screen" class="screen hidden-right">
    <div class="nav-bar editor-nav">
        <div class="nav-left">
            <button class="nav-btn" onclick="backToHome()">
                <svg class="icon"><use href="#icon-back"></use></svg> メモ
            </button>
        </div>
        <div class="nav-right">
            <button class="nav-btn" onclick="systemShare()"><svg class="icon"><use href="#icon-share"></use></svg></button>
            <button class="nav-btn" onclick="openActionMenu()"><svg class="icon"><use href="#icon-more"></use></svg></button>
        </div>
    </div>

    <div class="editor-area" id="editor-area" contenteditable="true" oninput="saveCurrentNote()"></div>

    <div class="keyboard-toolbar" id="keyboard-toolbar">
        <button class="kb-btn" style="font-weight: 800;" onmousedown="formatText(event, 'bold')">B</button>
        <button class="kb-btn" style="font-style: italic;" onmousedown="formatText(event, 'italic')">I</button>
        <button class="kb-btn" style="text-decoration: underline;" onmousedown="formatText(event, 'underline')">U</button>
        <button class="kb-btn" onmousedown="formatText(event, 'insertUnorderedList')">・リスト</button>
        <button class="kb-btn" onmousedown="applyHighlight(event)">マーカー</button>
        <button class="kb-btn" onclick="document.getElementById('file-upload').click()">
            <svg class="icon" style="width:20px; height:20px;"><use href="#icon-camera"></use></svg>
        </button>
        <input type="file" id="file-upload" style="display: none;" accept="image/*,video/*" onchange="handleFileUpload(event, false)">
    </div>
</div>

<div class="overlay" id="action-overlay" onclick="closeActionMenu()"></div>
<div class="action-sheet" id="action-sheet">
    <div class="color-row" id="color-palette">
        <div class="color-dot" style="background: #FFFFFF;" onclick="changeColor('')"></div>
        <div class="color-dot" style="background: #FFF59D;" onclick="changeColor('#FFF59D')"></div> <div class="color-dot" style="background: #FFCDD2;" onclick="changeColor('#FFCDD2')"></div> <div class="color-dot" style="background: #C8E6C9;" onclick="changeColor('#C8E6C9')"></div> <div class="color-dot" style="background: #BBDEFB;" onclick="changeColor('#BBDEFB')"></div> <div class="color-dot" style="background: #E1BEE7;" onclick="changeColor('#E1BEE7')"></div> </div>

    <div class="action-group">
        <div class="action-item" onclick="togglePin()">
            <span id="sheet-pin-text">ピン留め</span>
        </div>
        <div class="action-item" onclick="copyNoteText()">
            <span>テキストをコピー</span>
        </div>
    </div>
    
    <div class="action-group">
        <div class="action-item danger" onclick="deleteNote()">
            <span>削除</span>
        </div>
    </div>

    <div class="action-group" style="background: #fff;">
        <div class="action-item cancel" onclick="closeActionMenu()">
            キャンセル
        </div>
    </div>
</div>

<script>
    /* --- データ管理 --- */
    let notes = JSON.parse(localStorage.getItem('apple_memomi_notes')) || [];
    let currentNoteId = null;
    let targetNoteId = null; // 長押しで選択されたメモID

    window.onload = () => { renderNoteList(); };

    /* --- 画面遷移 (スライドアニメーション) --- */
    function showScreen(screenId) {
        const home = document.getElementById('home-screen');
        const editor = document.getElementById('editor-screen');
        
        if(screenId === 'editor-screen') {
            home.classList.remove('active');
            home.classList.add('hidden-left');
            editor.classList.remove('hidden-right');
            editor.classList.add('active');
        } else {
            editor.classList.remove('active');
            editor.classList.add('hidden-right');
            home.classList.remove('hidden-left');
            home.classList.add('active');
        }
    }

    function backToHome() {
        saveCurrentNote();
        currentNoteId = null;
        document.getElementById('search-input').value = '';
        renderNoteList();
        showScreen('home-screen');
    }

    /* --- メモロジック --- */
    function saveToDB() { localStorage.setItem('apple_memomi_notes', JSON.stringify(notes)); }

    function extractTextData(htmlContent) {
        const temp = document.createElement('div');
        temp.innerHTML = htmlContent;
        const text = temp.innerText.trim();
        const lines = text.split('\n').filter(line => line.trim() !== '');
        
        let title = '新規メモ';
        let preview = '追加テキストなし';
        
        if (lines.length > 0) title = lines[0];
        if (lines.length > 1) preview = lines.slice(1).join(' ').substring(0, 40);
        else if (htmlContent.includes('<img')) preview = '画像';
        else if (htmlContent.includes('<video')) preview = '動画';

        return { title, preview, plainText: text };
    }

    function getDateCategory(timestamp) {
        const now = new Date();
        const date = new Date(timestamp);
        const diffDays = Math.floor((now.setHours(0,0,0,0) - date.setHours(0,0,0,0)) / (1000 * 60 * 60 * 24));
        if (diffDays === 0) return '今日';
        if (diffDays === 1) return '昨日';
        if (diffDays <= 7) return '過去7日間';
        if (diffDays <= 30) return '過去30日間';
        return date.getFullYear() + '年';
    }

    /* メモリスト描画 */
    function renderNoteList(filterText = '') {
        const container = document.getElementById('note-list');
        container.innerHTML = '';
        
        const filteredNotes = notes.filter(note => {
            if (!filterText) return true;
            const temp = document.createElement('div');
            temp.innerHTML = note.content;
            return temp.innerText.toLowerCase().includes(filterText.toLowerCase());
        });

        document.getElementById('note-count-text').innerText = `${filteredNotes.length} 件のメモ`;

        filteredNotes.sort((a, b) => b.updatedAt - a.updatedAt);
        const groups = { 'ピン留め': [] };
        
        filteredNotes.forEach(note => {
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

            const groupEl = document.createElement('div');
            groupEl.className = 'list-group';
            
            if (category !== '今日' && Object.keys(groups).length > 1) {
                const titleEl = document.createElement('h3');
                titleEl.className = 'group-title';
                titleEl.innerText = category;
                container.appendChild(titleEl);
            }

            groupNotes.forEach(note => {
                const { title, preview } = extractTextData(note.content);
                const dateObj = new Date(note.updatedAt);
                const dateStr = dateObj.toLocaleTimeString('ja-JP', { hour: '2-digit', minute: '2-digit' });

                const div = document.createElement('div');
                div.className = 'note-item';
                if(note.color) div.style.backgroundColor = note.color;
                
                // 長押し (Haptic Touch / 3D Touch風)
                let pressTimer;
                const startPress = (e) => {
                    pressTimer = setTimeout(() => {
                        div.classList.add('haptic-pop');
                        if('vibrate' in navigator) navigator.vibrate(50); // 触覚フィードバック
                        setTimeout(() => div.classList.remove('haptic-pop'), 200);
                        openActionMenu(note.id);
                    }, 500);
                };
                const cancelPress = () => clearTimeout(pressTimer);

                div.addEventListener('touchstart', startPress);
                div.addEventListener('touchend', cancelPress);
                div.addEventListener('touchmove', cancelPress);
                // PC右クリック対応
                div.addEventListener('contextmenu', (e) => {
                    e.preventDefault();
                    openActionMenu(note.id);
                });
                
                // タップで開く
                div.onclick = (e) => {
                    if(!e.defaultPrevented) openNote(note.id);
                };

                div.innerHTML = `
                    <div class="note-title">${title}</div>
                    <div class="note-preview-row">
                        <span class="note-date">${dateStr}</span>
                        <span class="note-preview">${preview}</span>
                    </div>
                `;
                groupEl.appendChild(div);
            });
            container.appendChild(groupEl);
        }
    }

    function filterNotes() {
        const text = document.getElementById('search-input').value;
        renderNoteList(text);
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
        saveToDB();
        openNote(newNote.id);
    }

    function openNote(id) {
        currentNoteId = id;
        const note = notes.find(n => n.id === id);
        const editor = document.getElementById('editor-area');
        editor.innerHTML = note ? note.content : '';
        editor.style.backgroundColor = note.color || '#fff';
        showScreen('editor-screen');
        setTimeout(() => editor.focus(), 400); // スライド後にフォーカス
    }

    function saveCurrentNote() {
        if(!currentNoteId) return;
        const noteIndex = notes.findIndex(n => n.id === currentNoteId);
        if(noteIndex > -1) {
            notes[noteIndex].content = document.getElementById('editor-area').innerHTML;
            notes[noteIndex].updatedAt = Date.now();
            saveToDB();
        }
    }

    /* --- リッチテキスト・キーボード機能 --- */
    function formatText(e, command, value = null) {
        e.preventDefault(); // フォーカスを失わないようにする
        document.execCommand(command, false, value);
        saveCurrentNote();
    }

    function applyHighlight(e) {
        e.preventDefault();
        formatText(e, 'hiliteColor', '#FFCC00'); // Apple風の黄色マーカー
    }

    /* --- メディア・ファイル読込 --- */
    function handleFileUpload(event, isImport) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const result = e.target.result;
            if (isImport) createNewNote();
            
            const editor = document.getElementById('editor-area');
            editor.focus();

            if (file.type.startsWith('image/')) {
                document.execCommand('insertHTML', false, `<br><img src="${result}" alt="image"><br>`);
            } else if (file.type.startsWith('video/')) {
                document.execCommand('insertHTML', false, `<br><video src="${result}" controls></video><br>`);
            } else {
                const textContent = result.replace(/\n/g, '<br>');
                document.execCommand('insertHTML', false, `<div>${textContent}</div>`);
            }
            saveCurrentNote();
            if(isImport) renderNoteList();
        };

        if (file.type.startsWith('image/') || file.type.startsWith('video/')) {
            reader.readAsDataURL(file); 
        } else {
            reader.readAsText(file); 
        }
        event.target.value = '';
    }

    /* --- アクションシート (長押し・詳細メニュー) --- */
    function openActionMenu(noteId = currentNoteId) {
        targetNoteId = noteId;
        const note = notes.find(n => n.id === targetNoteId);
        if(!note) return;

        document.getElementById('sheet-pin-text').innerText = note.isPinned ? 'ピン留めを解除' : 'ピン留め';

        const overlay = document.getElementById('action-overlay');
        const sheet = document.getElementById('action-sheet');
        
        overlay.style.display = 'block';
        requestAnimationFrame(() => {
            overlay.style.opacity = '1';
            sheet.style.bottom = '10px';
        });

        // 端末のキーボードを隠す
        document.getElementById('editor-area').blur();
        document.getElementById('search-input').blur();
    }

    function closeActionMenu() {
        const overlay = document.getElementById('action-overlay');
        const sheet = document.getElementById('action-sheet');
        
        overlay.style.opacity = '0';
        sheet.style.bottom = '-100%';
        
        setTimeout(() => {
            overlay.style.display = 'none';
        }, 400);
    }

    function togglePin() {
        const note = notes.find(n => n.id === targetNoteId);
        note.isPinned = !note.isPinned;
        saveToDB(); renderNoteList(); closeActionMenu();
    }

    function deleteNote() {
        notes = notes.filter(n => n.id !== targetNoteId);
        saveToDB(); 
        closeActionMenu();
        if(currentNoteId === targetNoteId) {
            backToHome();
        } else {
            renderNoteList();
        }
    }

    function changeColor(hex) {
        const note = notes.find(n => n.id === targetNoteId);
        note.color = hex;
        saveToDB();
        
        if(currentNoteId === targetNoteId) {
            document.getElementById('editor-area').style.backgroundColor = hex || '#fff';
        }
        renderNoteList(); closeActionMenu();
    }

    function copyNoteText() {
        const note = notes.find(n => n.id === targetNoteId);
        const { plainText } = extractTextData(note.content);
        navigator.clipboard.writeText(plainText).then(() => {
            alert('コピーしました');
            closeActionMenu();
        });
    }

    /* --- システム共有 --- */
    function systemShare() {
        if(!currentNoteId) return;
        const note = notes.find(n => n.id === currentNoteId);
        const { title, plainText } = extractTextData(note.content);

        if (navigator.share) {
            navigator.share({ title: title, text: plainText }).catch(console.error);
        } else {
            alert('お使いの環境は共有機能に対応していません。');
        }
    }
</script>
</body>
</html>
