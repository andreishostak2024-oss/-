[messenger.html](https://github.com/user-attachments/files/28336160/messenger.html)
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>P2P Мессенджер</title>
<script src="https://unpkg.com/peerjs@1.5.1/dist/peerjs.min.js"></script>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; background: #0a0a0a; color: #eee; height: 100vh; display: flex; justify-content: center; align-items: center; }
#app { width: 100%; max-width: 420px; height: 100dvh; max-height: 800px; background: #121212; display: flex; flex-direction: column; border-radius: 16px; overflow: hidden; box-shadow: 0 0 50px rgba(0,0,0,0.6); margin: 8px; position: relative; }
@media(max-width:440px){#app{max-height:100dvh;margin:0;border-radius:0}}

/* Header */
.header { padding: 14px 16px; display: flex; align-items: center; gap: 10px; flex-shrink:0; background:#181818; border-bottom:1px solid #222; }
.header .back { background:none; border:none; color:#888; font-size:20px; cursor:pointer; padding:4px; }
.header h1 { font-size:17px; font-weight:700; flex:1; }
.header .badge { font-size:11px; color:#666; background:#222; padding:2px 10px; border-radius:10px; }

/* Screens */
.screen { display:none; flex-direction:column; height:100%; }
.screen.active { display:flex; }

/* Tabs */
#tabs { display:flex; background:#181818; border-bottom:1px solid #222; flex-shrink:0; }
.tab { flex:1; padding:12px; text-align:center; font-size:13px; cursor:pointer; color:#666; font-weight:600; transition:0.15s; border-bottom:2px solid transparent; }
.tab.active { color:#4CAF50; border-bottom-color:#4CAF50; }

/* Contacts */
#contacts { flex:1; overflow-y:auto; }
.contact-item { display:flex; align-items:center; gap:10px; padding:10px 14px; cursor:pointer; transition:background 0.1s; border-bottom:1px solid #1a1a1a; }
.contact-item:hover { background:#1e1e1e; }
.contact-item .avatar { width:42px; height:42px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:18px; flex-shrink:0; }
.contact-item .info { flex:1; min-width:0; }
.contact-item .name { font-weight:600; font-size:14px; }
.contact-item .status { font-size:11px; color:#666; }
.contact-item .status.online { color:#4CAF50; }
.contact-item .id-label { font-size:10px; color:#555; }
.empty-state { display:flex; flex-direction:column; align-items:center; justify-content:center; flex:1; color:#444; gap:8px; padding:40px; text-align:center; font-size:13px; }

/* Messages */
#messages { flex:1; overflow-y:auto; padding:10px 14px; display:flex; flex-direction:column; gap:4px; background:#0e0e0e; }
.msg { max-width:80%; padding:8px 14px; border-radius:16px; font-size:14px; line-height:1.4; word-wrap:break-word; }
.msg.out { align-self:flex-end; background:#1B5E20; border-bottom-right-radius:4px; }
.msg.in { align-self:flex-start; background:#262626; border-bottom-left-radius:4px; }
.msg .time { font-size:10px; opacity:0.35; text-align:right; margin-top:3px; }

#input-area { display:flex; gap:8px; padding:8px 12px; border-top:1px solid #222; background:#181818; flex-shrink:0; align-items:flex-end; }
#msg-input { flex:1; background:#222; border:1px solid #333; color:#eee; padding:10px 14px; border-radius:20px; font-size:14px; outline:none; font-family:inherit; resize:none; max-height:80px; }
#msg-input:focus { border-color:#4CAF50; }
#send-btn { width:40px; height:40px; border-radius:50%; background:#4CAF50; border:none; color:#fff; font-size:18px; cursor:pointer; flex-shrink:0; display:flex; align-items:center; justify-content:center; }

/* Call UI */
#call-overlay {
  display:none; position:absolute; top:0; left:0; width:100%; height:100%; z-index:50;
  background:linear-gradient(135deg,#0d1b0d 0%,#1a1a2e 100%);
  flex-direction:column; align-items:center; justify-content:center; gap:16px;
}
#call-overlay.show { display:flex; }
#call-overlay .call-avatar { width:80px; height:80px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:36px; }
#call-overlay .call-name { font-size:22px; font-weight:700; }
#call-overlay .call-status { font-size:14px; color:#888; }
#call-overlay .call-actions { display:flex; gap:40px; margin-top:20px; }
#call-overlay .call-btn { width:56px; height:56px; border-radius:50%; border:none; cursor:pointer; font-size:24px; display:flex; align-items:center; justify-content:center; transition:0.15s; }
#call-overlay .call-end { background:#E53935; color:#fff; }
#call-overlay .call-end:hover { background:#C62828; }
#call-overlay .call-mute { background:#333; color:#fff; }
#call-overlay .call-mute:hover { background:#444; }
#call-overlay .call-mute.active { background:#FDD835; color:#000; }

/* Profile */
#profile-bar { padding:14px 16px; display:flex; align-items:center; gap:12px; border-bottom:1px solid #222; flex-shrink:0; }
#profile-bar .my-avatar { width:44px; height:44px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:20px; cursor:pointer; transition:0.15s; }
#profile-bar .my-avatar:hover { opacity:0.8; }
#profile-bar .my-info { flex:1; min-width:0; }
#profile-bar .my-name { font-weight:600; font-size:15px; }
#profile-bar .my-id { font-size:10px; color:#555; cursor:pointer; }
#profile-bar .my-id:hover { color:#888; }

/* Buttons */
.btn-primary { padding:10px 24px; background:#4CAF50; border:none; border-radius:20px; color:#fff; font-size:14px; cursor:pointer; font-weight:600; }
.btn-secondary { padding:8px 18px; background:#222; border:none; border-radius:20px; color:#888; font-size:13px; cursor:pointer; }

/* Modal */
#modal { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.7); z-index:100; justify-content:center; align-items:center; }
#modal.show { display:flex; }
#modal-box { background:#1a1a1a; border-radius:20px; padding:24px; width:90%; max-width:340px; }
#modal-box h3 { margin-bottom:16px; font-size:18px; }
#modal-box input { width:100%; background:#222; border:1px solid #333; color:#eee; padding:12px 16px; border-radius:12px; font-size:15px; outline:none; margin-bottom:12px; }
#modal-box input:focus { border-color:#4CAF50; }
#modal-box .actions { display:flex; gap:8px; }
#modal-box button { flex:1; padding:10px; border-radius:10px; border:none; font-size:14px; cursor:pointer; font-weight:600; }
#modal-box .btn-p { background:#4CAF50; color:#fff; }
#modal-box .btn-s { background:#222; color:#888; }
#modal-box .avatars { display:flex; flex-wrap:wrap; gap:6px; margin-bottom:14px; justify-content:center; }
#modal-box .avatars span { font-size:28px; cursor:pointer; padding:4px; border-radius:8px; }
#modal-box .avatars span:hover { background:#333; }
#modal-box .avatars span.sel { background:#4CAF50; }

.notif { position:fixed; top:60px; left:50%; transform:translateX(-50%); background:#2a2a2a; padding:10px 20px; border-radius:12px; font-size:13px; z-index:200; box-shadow:0 4px 20px rgba(0,0,0,0.4); border:1px solid #333; }

.avatar-grid { display:flex; flex-wrap:wrap; gap:6px; justify-content:center; margin:8px 0 14px; }
.avatar-grid span { font-size:28px; width:40px; height:40px; display:flex; align-items:center; justify-content:center; border-radius:50%; cursor:pointer; transition:0.1s; }
.avatar-grid span:hover { background:#333; }
.avatar-grid span.sel { background:#4CAF50; }

.call-btn-accept { background:#4CAF50; color:#fff; }
.call-btn-accept:hover { background:#388E3C; }
</style>
</head>
<body>

<div id="app">
  <!-- Profile -->
  <div id="profile-bar">
    <div class="my-avatar" id="my-avatar" style="background:#4CAF50">🙋</div>
    <div class="my-info">
      <div class="my-name" id="my-name">Я</div>
      <div class="my-id" id="my-id">завантаження...</div>
    </div>
    <button class="btn-secondary" id="edit-profile-btn">✎</button>
  </div>

  <!-- Tabs -->
  <div id="tabs">
    <div class="tab active" data-tab="chats">Чати</div>
    <div class="tab" data-tab="contacts">Контакти</div>
  </div>

  <!-- Chats screen -->
  <div class="screen active" id="screen-chats">
    <div id="chat-list" style="flex:1;overflow-y:auto"></div>
  </div>

  <!-- Contacts screen -->
  <div class="screen" id="screen-contacts">
    <div id="contacts" style="flex:1;overflow-y:auto"></div>
    <div style="padding:12px;border-top:1px solid #222">
      <button class="btn-primary" id="add-contact-btn" style="width:100%">+ Додати контакт</button>
    </div>
  </div>

  <!-- Messages screen -->
  <div class="screen" id="screen-msg">
    <div class="header">
      <button class="back" id="back-btn">←</button>
      <div style="display:flex;align-items:center;gap:10px;flex:1;min-width:0">
        <div id="chat-avatar" style="width:36px;height:36px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:16px;flex-shrink:0"></div>
        <div style="min-width:0">
          <div id="chat-name" style="font-weight:600;font-size:14px"></div>
          <div id="chat-status" style="font-size:11px;color:#666"></div>
        </div>
      </div>
      <button id="call-btn" style="background:none;border:none;color:#4CAF50;font-size:20px;cursor:pointer">📞</button>
    </div>
    <div id="messages"></div>
    <div id="input-area">
      <textarea id="msg-input" placeholder="Повідомлення..." rows="1"></textarea>
      <button id="send-btn">➤</button>
    </div>
  </div>
</div>

<!-- Call overlay -->
<div id="call-overlay">
  <div class="call-avatar" id="call-avatar" style="background:#4CAF50">🙋</div>
  <div class="call-name" id="call-name">Користувач</div>
  <div class="call-status" id="call-status">Зачекайте...</div>
  <div class="call-actions">
    <button class="call-btn call-mute" id="call-mute-btn">🔊</button>
    <button class="call-btn call-end" id="call-end-btn">✕</button>
  </div>
</div>

<!-- Modal -->
<div id="modal">
  <div id="modal-box">
    <h3 id="modal-title">Назва</h3>
    <div id="modal-body"></div>
    <div class="actions" id="modal-actions"></div>
  </div>
</div>

<div id="notif" class="notif" style="display:none"></div>

<script>
const AVATARS = ['🙋','😎','🤩','🥳','🤖','👾','🦊','🐱','🐶','🐼','🐨','🦄','🦋','🐙','🦖','🐲','🦜','🐬','🦈','🐝','🌸','🌺','⭐','🔥','💎','🎮','🎸','🚀','🌈','🍕'];

let state = { name:'Я', avatar:'🙋', contacts:[], chats:{} };
let peer = null;
let myId = '';
let currentChat = null;
let connections = {}; // peerId -> DataConnection
let callInProgress = null;
let localStream = null;
let isMuted = false;
let typingTimers = {};

function load() {
  try {
    const s = localStorage.getItem('p2pmsgr');
    if (s) { state = JSON.parse(s); if(!state.contacts) state.contacts=[]; if(!state.chats) state.chats={}; }
  } catch {}
}

function save() { localStorage.setItem('p2pmsgr', JSON.stringify(state)); }

function notify(msg) {
  const el = document.getElementById('notif');
  el.textContent = msg; el.style.display = 'block';
  clearTimeout(el._t); el._t = setTimeout(() => el.style.display='none', 3000);
}

function randId() { return Math.random().toString(36).slice(2,8) + Math.random().toString(36).slice(2,4); }

function initPeer() {
  const savedId = localStorage.getItem('p2p_peerid');
  peer = new Peer(savedId || randId(), { debug:0 });
  peer.on('open', (id) => {
    myId = id;
    localStorage.setItem('p2p_peerid', id);
    document.getElementById('my-id').textContent = 'ID: ' + id;
    document.getElementById('my-id').title = 'Натисніть щоб скопіювати';
  });
  peer.on('connection', (conn) => handleConnection(conn));
  peer.on('call', (call) => handleIncomingCall(call));
  peer.on('error', (err) => { if (err.type !== 'unavailable-id') console.error(err); });
}

// Contact management
function addContact(name, peerId) {
  if (!name || !peerId) return;
  if (state.contacts.find(c => c.peerId === peerId)) { notify('Контакт вже існує'); return; }
  state.contacts.push({ id: randId(), name, peerId, avatar: AVATARS[Math.floor(Math.random()*AVATARS.length)] });
  save();
  renderContacts();
  notify('Контакт додано: ' + name);
  connectToPeer(peerId);
}

function connectToPeer(peerId) {
  if (connections[peerId]) return;
  if (peerId === myId) return;
  const conn = peer.connect(peerId, { reliable: true });
  handleConnection(conn);
}

function handleConnection(conn) {
  const pid = conn.peer;
  connections[pid] = conn;
  if (!state.chats[pid]) state.chats[pid] = [];
  conn.on('open', () => {
    conn.send({ type:'hello', name:state.name, avatar:state.avatar });
    updateUI();
  });
  conn.on('data', (data) => {
    if (data.type === 'hello') {
      if (!state.chats[pid]) state.chats[pid] = [];
      const contact = state.contacts.find(c => c.peerId === pid);
      if (!contact) {
        state.contacts.push({ id: randId(), name: data.name || pid, peerId: pid, avatar: data.avatar || '🙋', auto:true });
        save();
        renderContacts();
      }
      updateUI();
    }
    if (data.type === 'msg') {
      if (!state.chats[pid]) state.chats[pid] = [];
      state.chats[pid].push({ text: data.text, out: false, time: Date.now() });
      save();
      if (currentChat === pid) renderMessages();
      updateChatList();
      notify(data.name + ': ' + data.text);
    }
  });
  conn.on('close', () => { delete connections[pid]; updateUI(); });
}

function sendMessage() {
  const input = document.getElementById('msg-input');
  const text = input.value.trim();
  if (!text || !currentChat) return;
  if (!state.chats[currentChat]) state.chats[currentChat] = [];
  state.chats[currentChat].push({ text, out: true, time: Date.now() });
  input.value = ''; input.style.height = 'auto';
  save();
  renderMessages();
  updateChatList();
  const conn = connections[currentChat];
  if (conn && conn.open) {
    conn.send({ type:'msg', text, name:state.name });
  } else {
    notify('⚠️ Контакт не в мережі. Повідомлення збережено локально.');
  }
}

// Calls
function startCall() {
  if (!currentChat) return;
  const conn = connections[currentChat];
  if (!conn || !conn.open) { notify('⚠️ Контакт не в мережі'); return; }
  navigator.mediaDevices.getUserMedia({ audio: true, video: false }).then((stream) => {
    localStream = stream;
    const call = peer.call(currentChat, stream);
    setupCall(call, false);
  }).catch(() => notify('⚠️ Доступ до мікрофону заборонено'));
}

function handleIncomingCall(call) {
  const contact = state.contacts.find(c => c.peerId === call.peer);
  const name = contact ? contact.name : call.peer;
  if (confirm('📞 Вхідний дзвінок від ' + name + '! Прийняти?')) {
    navigator.mediaDevices.getUserMedia({ audio: true, video: false }).then((stream) => {
      localStream = stream;
      call.answer(stream);
      setupCall(call, true);
    }).catch(() => { call.close(); notify('⚠️ Немає доступу до мікрофону'); });
  } else {
    call.close();
  }
}

function setupCall(call, isIncoming) {
  callInProgress = call;
  const contact = state.contacts.find(c => c.peerId === call.peer);
  const name = contact ? contact.name : call.peer;
  const avatar = contact ? contact.avatar : '🙋';
  const color = contact ? '#4CAF50' : '#555';

  document.getElementById('call-avatar').textContent = avatar;
  document.getElementById('call-avatar').style.background = color;
  document.getElementById('call-name').textContent = name;
  document.getElementById('call-status').textContent = isIncoming ? '🔊 З'єднання...' : '🔊 Виклик...';
  document.getElementById('call-overlay').classList.add('show');

  isMuted = false;
  document.getElementById('call-mute-btn').textContent = '🔊';
  document.getElementById('call-mute-btn').classList.remove('active');

  call.on('stream', (remoteStream) => {
    document.getElementById('call-status').textContent = '🔊 Розмова';
    // Play remote audio
    const audio = document.createElement('audio');
    audio.srcObject = remoteStream;
    audio.autoplay = true;
    audio.id = 'remote-audio';
    document.body.appendChild(audio);
  });

  call.on('close', () => endCall());
}

function endCall() {
  if (localStream) { localStream.getTracks().forEach(t => t.stop()); localStream = null; }
  const audio = document.getElementById('remote-audio');
  if (audio) { audio.pause(); audio.remove(); }
  if (callInProgress) { try { callInProgress.close(); } catch {} callInProgress = null; }
  document.getElementById('call-overlay').classList.remove('show');
}

document.getElementById('call-end-btn').addEventListener('click', endCall);
document.getElementById('call-mute-btn').addEventListener('click', () => {
  if (localStream) {
    isMuted = !isMuted;
    localStream.getAudioTracks().forEach(t => t.enabled = !isMuted);
    document.getElementById('call-mute-btn').textContent = isMuted ? '🔇' : '🔊';
    document.getElementById('call-mute-btn').classList.toggle('active', isMuted);
  }
});

// UI functions
function renderContacts() {
  const el = document.getElementById('contacts');
  el.innerHTML = '';
  if (state.contacts.length === 0) {
    el.innerHTML = '<div class="empty-state"><div style="font-size:40px">📇</div>Немає контактів<br>Натисніть "Додати контакт" і введіть ID співрозмовника</div>';
    return;
  }
  for (const c of state.contacts) {
    const online = !!connections[c.peerId] && connections[c.peerId].open;
    const item = document.createElement('div');
    item.className = 'contact-item';
    item.innerHTML = `
      <div class="avatar" style="background:${online?'#4CAF50':'#333'}">${c.avatar||'🙋'}</div>
      <div class="info">
        <div class="name">${c.name}</div>
        <div class="status ${online?'online':''}">${online ? '🟢 в мережі' : '⚫ не в мережі'}</div>
        <div class="id-label">${c.peerId}</div>
      </div>
    `;
    item.addEventListener('click', () => {
      currentChat = c.peerId;
      document.getElementById('chat-avatar').textContent = c.avatar || '🙋';
      document.getElementById('chat-avatar').style.background = online ? '#4CAF50' : '#333';
      document.getElementById('chat-name').textContent = c.name;
      document.getElementById('chat-status').textContent = online ? '🟢 в мережі' : '⚫ не в мережі';
      document.getElementById('screen-chats').classList.remove('active');
      document.getElementById('screen-contacts').classList.remove('active');
      document.getElementById('screen-msg').classList.add('active');
      renderMessages();
    });
    el.appendChild(item);
  }
}

function renderChatList() {
  const el = document.getElementById('chat-list');
  el.innerHTML = '';
  const chats = Object.entries(state.chats).filter(([pid, msgs]) => msgs && msgs.length > 0);
  if (chats.length === 0) {
    el.innerHTML = '<div class="empty-state"><div style="font-size:40px">💬</div>Немає чатів<br>Додайте контакт і напишіть повідомлення</div>';
    return;
  }
  for (const [pid, msgs] of chats.sort((a,b) => (b[1][b[1].length-1]?.time||0) - (a[1][a[1].length-1]?.time||0))) {
    const contact = state.contacts.find(c => c.peerId === pid);
    if (!contact) continue;
    const last = msgs[msgs.length - 1];
    const online = !!connections[pid] && connections[pid].open;
    const item = document.createElement('div');
    item.className = 'contact-item';
    item.innerHTML = `
      <div class="avatar" style="background:${online?'#4CAF50':'#333'}">${contact.avatar||'🙋'}</div>
      <div class="info">
        <div class="name">${contact.name}</div>
        <div class="status ${online?'online':''}" style="font-size:12px">${last ? (last.out?'Ви: ':'') + last.text.slice(0,30) : ''}</div>
      </div>
      ${last ? '<div style="font-size:10px;color:#555;flex-shrink:0">'+fmtTime(last.time)+'</div>' : ''}
    `;
    item.addEventListener('click', () => {
      currentChat = pid;
      document.getElementById('chat-avatar').textContent = contact.avatar || '🙋';
      document.getElementById('chat-avatar').style.background = online ? '#4CAF50' : '#333';
      document.getElementById('chat-name').textContent = contact.name;
      document.getElementById('chat-status').textContent = online ? '🟢 в мережі' : '⚫ не в мережі';
      document.getElementById('screen-chats').classList.remove('active');
      document.getElementById('screen-contacts').classList.remove('active');
      document.getElementById('screen-msg').classList.add('active');
      renderMessages();
    });
    el.appendChild(item);
  }
}

function renderMessages() {
  const el = document.getElementById('messages');
  el.innerHTML = '';
  if (!currentChat || !state.chats[currentChat] || state.chats[currentChat].length === 0) {
    el.innerHTML = '<div class="empty-state">Напишіть перше повідомлення</div>';
    return;
  }
  for (const m of state.chats[currentChat]) {
    const div = document.createElement('div');
    div.className = 'msg ' + (m.out ? 'out' : 'in');
    div.textContent = m.text;
    const t = document.createElement('div'); t.className = 'time';
    t.textContent = fmtTime(m.time);
    div.appendChild(t);
    el.appendChild(div);
  }
  el.scrollTop = el.scrollHeight;
}

function updateChatList() { renderChatList(); }
function updateUI() { renderChatList(); renderContacts(); }

function fmtTime(ts) {
  const d = new Date(ts);
  const now = new Date();
  if (d.toDateString() === now.toDateString()) return d.toLocaleTimeString('uk-UA', {hour:'2-digit',minute:'2-digit'});
  return d.toLocaleDateString('uk-UA', {day:'numeric',month:'short'});
}

function showModal(title, bodyHtml, buttons) {
  document.getElementById('modal-title').textContent = title;
  document.getElementById('modal-body').innerHTML = bodyHtml;
  const actions = document.getElementById('modal-actions');
  actions.innerHTML = '';
  for (const b of buttons) {
    const btn = document.createElement('button');
    btn.className = b.primary ? 'btn-p' : 'btn-s';
    btn.textContent = b.text;
    btn.addEventListener('click', () => { document.getElementById('modal').classList.remove('show'); b.cb(); });
    actions.appendChild(btn);
  }
  document.getElementById('modal').classList.add('show');
}

// Event listeners
document.getElementById('add-contact-btn').addEventListener('click', () => {
  showModal('Новий контакт', `
    <input id="m-name" placeholder="Ім'я..." maxlength="20">
    <input id="m-peer" placeholder="ID користувача..." style="font-size:13px">
  `, [
    { text:'Скасувати', cb:()=>{} },
    { text:'Додати', primary:true, cb:()=>{
      const name = document.getElementById('m-name').value.trim();
      const pid = document.getElementById('m-peer').value.trim();
      if (name && pid) addContact(name, pid);
    }}
  ]);
  setTimeout(() => document.getElementById('m-name').focus(), 100);
});

document.getElementById('edit-profile-btn').addEventListener('click', () => {
  let sel = state.avatar;
  showModal('Мій профіль', `
    <input id="m-my-name" placeholder="Ім'я..." value="${state.name}" maxlength="20">
    <div style="font-size:13px;color:#888;margin:4px 0">Виберіть аватар:</div>
    <div class="avatar-grid" id="av-grid">${AVATARS.map(a => `<span class="${a===sel?'sel':''}">${a}</span>`).join('')}</div>
  `, [
    { text:'Скасувати', cb:()=>{} },
    { text:'Зберегти', primary:true, cb:()=>{
      state.name = document.getElementById('m-my-name').value.trim() || 'Я';
      state.avatar = sel;
      document.getElementById('my-name').textContent = state.name;
      document.getElementById('my-avatar').textContent = state.avatar;
      save();
      // Broadcast name change to all connections
      for (const conn of Object.values(connections)) {
        if (conn.open) conn.send({ type:'hello', name:state.name, avatar:state.avatar });
      }
    }}
  ]);
  setTimeout(() => {
    document.getElementById('m-my-name').focus();
    document.querySelectorAll('#av-grid span').forEach(el => {
      el.addEventListener('click', () => {
        document.querySelectorAll('#av-grid span').forEach(s => s.classList.remove('sel'));
        el.classList.add('sel');
        sel = el.textContent;
      });
    });
  }, 100);
});

document.getElementById('my-id').addEventListener('click', () => {
  navigator.clipboard.writeText(myId).then(() => notify('ID скопійовано: ' + myId));
});

document.getElementById('back-btn').addEventListener('click', () => {
  currentChat = null;
  document.getElementById('screen-msg').classList.remove('active');
  document.getElementById('screen-chats').classList.add('active');
  updateChatList();
});

document.getElementById('send-btn').addEventListener('click', sendMessage);
document.getElementById('msg-input').addEventListener('keydown', (e) => {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); }
});
document.getElementById('msg-input').addEventListener('input', function() {
  this.style.height = 'auto'; this.style.height = Math.min(this.scrollHeight, 80) + 'px';
});

document.getElementById('call-btn').addEventListener('click', startCall);

document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    tab.classList.add('active');
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById('screen-' + tab.dataset.tab).classList.add('active');
    if (tab.dataset.tab === 'chats') updateChatList();
    if (tab.dataset.tab === 'contacts') renderContacts();
  });
});

// Init
load();
document.getElementById('my-name').textContent = state.name;
document.getElementById('my-avatar').textContent = state.avatar;
initPeer();
renderChatList();
renderContacts();

// Reconnect to known contacts on peer open
peer.on('open', () => {
  for (const c of state.contacts) {
    if (c.peerId !== myId) connectToPeer(c.peerId);
  }
});
</script>
</body>
</html>
