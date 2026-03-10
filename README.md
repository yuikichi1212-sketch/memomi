<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>メモミ</title>
<style>
    /* 全体設定・スマホ向けモダンホワイトUI */
    :root {
        --primary-color: #ffd700;
        --bg-color: #ffffff;
        --surface-color: #f8f8f8;
        --text-color: #1c1c1e;
        --text-light: #8e8e93;
        --border-color: #e5e5ea;
        --danger-color: #ff3b30;
    }
    body, html {
        margin: 0; padding: 0; height: 100%; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        background-color: var(--bg-color); color: var(--text-color); overflow: hidden;
        -webkit-tap-highlight-color: transparent;
    }
    * { box-sizing: border-box; }
    button { cursor: pointer; border: none; outline: none; background: transparent; font-family: inherit; }
    input { outline: none; border: none; background: transparent; }

    /* 共通ヘッダー */
    .header {
        display: flex; justify-content: space-between; align-items: center;
        padding: 16px 20px; border-bottom: 1px solid var(--border-color);
        background: rgba(255,255,255,0.95); backdrop-filter: blur(10px);
        position: fixed; top: 0; left: 0; right: 0; z-index: 50; height: 60px;
    }
    .header-title { font-size: 22px; font-weight: 700; }
    .header-btn { font-size: 16px; color: var(--primary-color); font-weight: 600; padding: 8px; color: #b89a00; /* 黄色の視認性調整 */ }
    
    /* 画面コンテナ */
    .screen { display: none; height: 100%; padding-top: 60px; flex-direction: column; background: var(--surface-color); }
    .screen.active { display: flex; animation: slideIn 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); }

    /* --- ホーム画面 (一覧) --- */
    .search-bar-container { padding: 12px 20px; background: var(--surface-color); }
    .search-input { width: 100%; background: #e3e3e8; padding: 10px 16px; border-radius: 10px; font-size: 16px; color: var(--text-color); }
    
    .note-list { flex: 1; overflow-y: auto; padding: 0 20px 100px 20px; }
    .list-group-title { font-size: 13px; color: var(--text-light); margin: 24px 0 8px 4px; font-weight: bold; }
    .note-item { 
        background: #fff; padding: 16px; border-radius: 12px; margin-bottom: 12px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.03); display: flex; flex-direction: column;
        border-left: 4px solid transparent; transition: 0.2s;
    }
    .note-item:active { transform: scale(0.98); }
    .note-title { font-size: 16px; font-weight: 600; margin-bottom: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-preview { font-size: 14px; color: var(--text-light); margin-bottom: 8px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .note-meta { font-size: 12px; color: #aeaeb2; display: flex; justify-content: space-between; }

    .fab {
        position: fixed; bottom: 30px; right: 30px; width: 56px; height: 56px;
        background: var(--primary-color); border-radius: 50%;
        display: flex; align-items: center; justify-content: center;
        box-shadow: 0 4px 16px rgba(255, 215, 0, 0.4); z-index: 100;
        font-size: 32px; font-weight: 300; color: #000;
    }

    /* --- エディタ画面 --- */
    #editor-screen { background: #fff; }
    .editor-area {
        flex: 1; padding: 20px; font-size: 17px; line-height: 1.6;
        outline: none; overflow-y: auto; color: var(--text-color); padding-bottom: 80px;
    }
    .editor-area img, .editor-area video { max-width: 100%; border-radius: 8px; margin: 12px 0; }
    .editor-area blockquote { border-left: 3px solid var(--primary-color); margin: 0; padding-left: 10px; color: var(--text-light); }

    /* ボトムツールバー (多機能リッチテキスト) */
    .editor-toolbar {
        position: fixed; bottom: 0; left: 0; right: 0; height: 50px;
        background: #f0f0f5; border-top: 1px solid var(--border-color);
        display: flex; align-items: center; padding: 0 10px; gap: 15px;
        overflow-x: auto; white-space: nowrap; z-index: 60;
    }
    .editor-toolbar::-webkit-scrollbar { display: none; }
    .tool-btn { font-size: 16px; font-weight: 600; color: #333; padding: 8px; border-radius: 6px; }
    .tool-btn:active { background: #d1d1d6; }
    .tool-sep { width: 1px; height: 24px; background: #c7c7cc; }

    /* --- 詳細メニュー (ボトムシート) --- */
    .overlay { display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); z-index: 200; }
    .bottom-sheet {
        position: fixed; bottom: -100%; left: 0; right: 0; background: #fff;
        border-radius: 20px 20px 0 0; padding: 24px 20px; z-index: 201;
        transition: bottom 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
    }
    .bottom-sheet.open { bottom: 0; }
    .sheet-handle { width: 40px; height: 5px; background: #d1d1d6; border-radius: 3px; margin: 0 auto 20px; }
    
    .menu-item {
        padding: 16px 0; font-size: 17px; display: flex; justify-content: space-between;
        align-items: center; border-bottom: 1px solid var(--border-color); cursor: pointer;
    }
    .menu-item:last-child { border-bottom: none; }
    .menu-item.danger { color: var(--danger-color); }
    
    .color-palette { display: flex; gap: 12px; padding: 16px 0; overflow-x: auto; border-bottom: 1px solid var(--border-color); }
    .color-circle { width: 36px; height: 36px; border-radius: 50%; border: 1px solid #d1d1d6; flex-shrink: 0; }
    .color-circle.selected { border: 2px solid var(--primary-color); }

    .meta-info { font-size: 13px; color: var(--text-light); text-align: center; margin-top: 20px; line-height: 1.8; }

    @keyframes slideIn { from { transform: translateX(20px); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
</style>
</head>
<body>

<div id="home-screen" class="screen active">
    <div class="header">
        <div class="header-title">メモミ</div>
        <button class="header-btn" onclick="document.getElementById('file-import').click()">読込</button>
        <input type="file" id="file-import" style="display: none;" accept="*/*" onchange="handleFileUpload(event, true)">
    </div>
    
    <div class="search-bar-container">
        <input type="text" class="search-input" id="search-input" placeholder="メモを検索" oninput="filterNotes()">
    </div>

    <div class="note-list" id="note-list">
        </div>

    <button class="fab" onclick="createNewNote()">+</button>
</div>

<div id="editor-screen" class="screen">
    <div class="header">
        <button class="header-btn" onclick="backToHome()">戻る</button>
        <div style="display: flex; gap: 10px;">
            <button class="header-btn" onclick="systemShare()">共有</button>
            <button class="header-btn" onclick="openMenu()">詳細</button>
        </div>
    </div>

    <div class="editor-area" id="editor-area" contenteditable="true" oninput="saveCurrentNote()"></div>

    <div class="editor-toolbar">
        <button class="tool-btn" onclick="formatText('bold')" style="font-weight: 900;">B</button>
        <button class="tool-btn" onclick="formatText('italic')" style="font-style: italic;">I</button>
        <button class="tool-btn" onclick="formatText('underline')" style="text-decoration: underline;">U</button>
        <button class="tool-btn" onclick="formatText('strikeThrough')" style="text-decoration: line-through;">S</button>
        <div class="tool-sep"></div>
        <button class="tool-btn" onclick="formatText('insertUnorderedList')">リスト</button>
        <button class="tool-btn" onclick="formatText('insertOrderedList')">番号</button>
        <button class="tool-btn" onclick="formatText('formatBlock', 'BLOCKQUOTE')">引用</button>
        <div class="tool-sep"></div>
        <button class="tool-btn" onclick="document.getElementById('file-upload').click()">写真/動画</button>
        <input type="file" id="file-upload" style="display: none;" accept="image/*,video/*" onchange="handleFileUpload(event, false)">
        <div class="tool-sep"></div>
        <button class="tool-btn" onclick="applyTextColor()">文字色</button>
        <button class="tool-btn" onclick="applyHighlight()">背景色</button>
    </div>
</div>

<div class="overlay" id="menu-overlay" onclick="closeMenu()"></div>
<div class="bottom-sheet" id="bottom-sheet">
    <div class="sheet-handle"></div>
    
    <div class="color-palette" id="color-palette">
        </div>

    <div class="menu-item" onclick="togglePin()">
        <span id="pin-text">ピン留めする</span>
    </div>
    <div class="menu-item danger" onclick="deleteNote()">
        <span>削除</span>
    </div>

    <div class="meta-info" id="meta-info">
        </div>
</div>

<script>
    /* --- データと状態 --- */
    let notes = JSON.parse(localStorage.getItem('memomi_notes')) || [];
    let currentNoteId = null;
    
    // カラーパレット定義 (モダンで淡い色)
    const colors = [
        { hex: '', name: '標準' },
        { hex: '#fff9c4', name: '黄' },
        { hex: '#f8bbd0', name: '赤' },
        { hex: '#c8e6c9', name: '緑' },
        { hex: '#bbdefb', name: '青' },
        { hex: '#e1bee7', name: '紫' },
        { hex: '#ffcc80', name: '橙' },
        { hex: '#cfd8dc', name: '灰' }
    ];

    // 初期化
    window.onload = () => {
        renderNoteList();
        generateColorPalette();
    };

    /* --- 画面遷移 --- */
    function showScreen(screenId) {
        document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
        document.getElementById(screenId).classList.add('active');
    }

    function backToHome() {
        saveCurrentNote();
        currentNoteId = null;
        document.getElementById('search-input').value = '';
        renderNoteList();
        showScreen('home-screen');
    }

    /* --- メモのロジック --- */
    function saveToDB() {
        localStorage.setItem('memomi_notes', JSON.stringify(notes));
    }

    function extractTextData(htmlContent) {
        const temp = document.createElement('div');
        temp.innerHTML = htmlContent;
        const text = temp.innerText.trim();
        const lines = text.split('\n').filter(line => line.trim() !== '');
        const title = lines.length > 0 ? lines[0] : '新規メモ';
        const preview = lines.length > 1 ? lines.slice(1).join(' ').substring(0, 40) : '追加テキストなし';
        const charCount = text.length;
        return { title, preview, charCount };
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

    function renderNoteList(filterText = '') {
        const container = document.getElementById('note-list');
        container.innerHTML = '';
        
        // 検索フィルタリング
        const filteredNotes = notes.filter(note => {
            if (!filterText) return true;
            const temp = document.createElement('div');
            temp.innerHTML = note.content;
            return temp.innerText.toLowerCase().includes(filterText.toLowerCase());
        });

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

            const titleEl = document.createElement('div');
            titleEl.className = 'list-group-title';
            titleEl.innerText = category;
            container.appendChild(titleEl);

            groupNotes.forEach(note => {
                const { title, preview } = extractTextData(note.content);
                const dateStr = new Date(note.updatedAt).toLocaleDateString('ja-JP', { month: 'numeric', day: 'numeric' });

                const div = document.createElement('div');
                div.className = 'note-item';
                if(note.color) div.style.backgroundColor = note.color;
                if(note.isPinned) div.style.borderLeftColor = '#ffb300';
                
                div.onclick = () => openNote(note.id);

                div.innerHTML = `
                    <div class="note-title">${title}</div>
                    <div class="note-preview">${preview}</div>
                    <div class="note-meta">
                        <span>${dateStr}</span>
                    </div>
                `;
                container.appendChild(div);
            });
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
            createdAt: Date.now(),
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
        // スマホでキーボードを出すためにフォーカス
        setTimeout(() => editor.focus(), 300);
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

    /* --- リッチテキスト機能 --- */
    function formatText(command, value = null) {
        document.execCommand(command, false, value);
        document.getElementById('editor-area').focus();
        saveCurrentNote();
    }

    // 文字色（簡易的に赤を指定、応用でパレット展開可能）
    function applyTextColor() {
        formatText('foreColor', '#ff3b30'); 
    }
    // 文字背景色（マーカー）
    function applyHighlight() {
        formatText('hiliteColor', '#fff59d');
    }

    /* --- ファイル・メディア対応 --- */
    function handleFileUpload(event, isImportToNew) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const result = e.target.result;
            
            if (isImportToNew) {
                // ホーム画面の「読込」から新しいメモを作る場合
                createNewNote();
            }

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
        };

        if (file.type.startsWith('image/') || file.type.startsWith('video/')) {
            reader.readAsDataURL(file); 
        } else {
            reader.readAsText(file); 
        }
        event.target.value = '';
    }

    /* --- 詳細メニュー (BottomSheet) 機能 --- */
    function openMenu() {
        const note = notes.find(n => n.id === currentNoteId);
        if(!note) return;

        // ピン状態更新
        document.getElementById('pin-text').innerText = note.isPinned ? 'ピン留めを解除' : 'ピン留めする';

        // メタ情報生成 (隠れた多機能: 文字数カウント等)
        const { charCount } = extractTextData(note.content);
        const created = new Date(note.createdAt).toLocaleString('ja-JP');
        const updated = new Date(note.updatedAt).toLocaleString('ja-JP');
        document.getElementById('meta-info').innerHTML = `
            文字数: ${charCount} 文字<br>
            作成: ${created}<br>
            更新: ${updated}
        `;

        // カラーパレットの選択状態更新
        const palette = document.getElementById('color-palette');
        Array.from(palette.children).forEach(child => {
            if(child.dataset.color === (note.color || '')) {
                child.classList.add('selected');
            } else {
                child.classList.remove('selected');
            }
        });

        document.getElementById('menu-overlay').style.display = 'block';
        // 少し遅延させてアニメーションを適用
        setTimeout(() => {
            document.getElementById('bottom-sheet').classList.add('open');
        }, 10);
    }

    function closeMenu() {
        document.getElementById('bottom-sheet').classList.remove('open');
        setTimeout(() => {
            document.getElementById('menu-overlay').style.display = 'none';
        }, 300);
    }

    function togglePin() {
        const note = notes.find(n => n.id === currentNoteId);
        note.isPinned = !note.isPinned;
        saveToDB();
        closeMenu();
    }

    function deleteNote() {
        if(confirm('このメモを削除しますか？')) {
            notes = notes.filter(n => n.id !== currentNoteId);
            saveToDB();
            closeMenu();
            backToHome();
        }
    }

    function generateColorPalette() {
        const container = document.getElementById('color-palette');
        colors.forEach(c => {
            const div = document.createElement('div');
            div.className = 'color-circle';
            div.style.backgroundColor = c.hex || '#fff';
            div.dataset.color = c.hex;
            div.onclick = () => changeNoteColor(c.hex, div);
            container.appendChild(div);
        });
    }

    function changeNoteColor(hex, element) {
        const note = notes.find(n => n.id === currentNoteId);
        note.color = hex;
        document.getElementById('editor-area').style.backgroundColor = hex || '#fff';
        saveToDB();
        
        // 選択状態の更新
        Array.from(document.getElementById('color-palette').children).forEach(c => c.classList.remove('selected'));
        element.classList.add('selected');
    }

    /* --- システム共有 --- */
    function systemShare() {
        if(!currentNoteId) return;
        const note = notes.find(n => n.id === currentNoteId);
        const { title } = extractTextData(note.content);
        
        const temp = document.createElement('div');
        temp.innerHTML = note.content;
        const plainText = temp.innerText;

        if (navigator.share) {
            navigator.share({ title: title, text: plainText }).catch(console.error);
        } else {
            alert('お使いの環境は共有機能に対応していません。');
        }
    }
</script>
</body>
</html>
