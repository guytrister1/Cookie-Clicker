<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>משחק האימפריית עוגיות</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.22.1/firebase-app-compat.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.22.1/firebase-auth-compat.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.22.1/firebase-database-compat.min.js"></script>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body { font-family:Arial, sans-serif; background:linear-gradient(45deg,#FF6B6B,#4ECDC4,#45B7D1,#96CEB4,#FFEAA7); background-size:400% 400%; animation:gradientShift 10s ease infinite; min-height:100vh; display:flex; direction:rtl; overflow-x:hidden; }
@keyframes gradientShift { 0%{background-position:0% 50%;} 50%{background-position:100% 50%;} 100%{background-position:0% 50%;} }
.login-screen { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); display:flex; justify-content:center; align-items:center; z-index:10000; }
.login-form { background:rgba(255,255,255,0.95); padding:40px; border-radius:20px; text-align:center; box-shadow:0 20px 60px rgba(0,0,0,0.3); min-width:350px; }
.login-form h2 { color:#2C3E50; margin-bottom:25px; font-size:2rem; }
.login-form input { width:100%; padding:12px; margin:10px 0; border:2px solid #ddd; border-radius:10px; font-size:1rem; text-align:center; }
.login-form button { background:linear-gradient(45deg,#4ECDC4,#45B7D1); color:white; padding:12px 25px; border:none; border-radius:10px; font-size:1.1rem; cursor:pointer; margin:10px 5px; transition:all 0.3s ease; }
.login-form button:hover { transform:translateY(-2px); box-shadow:0 5px 15px rgba(0,0,0,0.2); }
.login-form button:disabled { opacity:0.6; cursor:not-allowed; transform:none; }
.remember-me { display:flex; align-items:center; justify-content:center; margin:15px 0; }
.remember-me input { width:auto; margin-left:8px; }
.top-buttons { position:fixed; top:20px; left:20px; z-index:1000; display:flex; gap:10px; flex-wrap:wrap; }
.club-button, .donate-button { background:linear-gradient(45deg,#8E44AD,#E74C3C); color:white; padding:10px 18px; border:none; border-radius:15px; font-size:0.95rem; font-weight:bold; cursor:pointer; box-shadow:0 5px 15px rgba(0,0,0,0.2); }
.donate-button { background:linear-gradient(45deg,#27AE60,#2ECC71); }
.club-button:hover, .donate-button:hover { transform:translateY(-2px); }
.settings-button { position:fixed; top:20px; right:20px; background:linear-gradient(45deg,#34495E,#2C3E50); color:white; padding:10px 20px; border:none; border-radius:15px; font-size:1rem; font-weight:bold; cursor:pointer; z-index:1000; }
.admin-button { position:fixed; top:80px; right:20px; background:linear-gradient(45deg,#9B59B6,#8E44AD); color:white; padding:10px 20px; border:none; border-radius:15px; font-size:1rem; font-weight:bold; cursor:pointer; z-index:1000; border:2px solid rgba(255,255,255,0.3); }
.modal { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); display:none; justify-content:center; align-items:center; z-index:5000; }
.modal-content { background:rgba(255,255,255,0.95); padding:30px; border-radius:20px; text-align:center; box-shadow:0 20px 60px rgba(0,0,0,0.3); min-width:350px; max-width:500px; max-height:80vh; overflow-y:auto; }
.modal-content h3 { color:#2C3E50; margin-bottom:20px; font-size:1.8rem; }
.modal-content input { width:100%; padding:12px; margin:10px 0; border:2px solid #ddd; border-radius:10px; font-size:1rem; text-align:center; }
.modal-content button { background:linear-gradient(45deg,#4ECDC4,#45B7D1); color:white; padding:12px 25px; border:none; border-radius:10px; font-size:1.05rem; cursor:pointer; margin:8px 5px; }
.modal-content button:disabled { opacity:0.6; cursor:not-allowed; }
.close-button { background:linear-gradient(45deg,#E74C3C,#C0392B) !important; }
.settings-section, .admin-section { background:rgba(0,0,0,0.05); padding:18px; margin:12px 0; border-radius:15px; }
.settings-section h4, .admin-section h4 { color:#2C3E50; margin-bottom:12px; font-size:1.2rem; }
.gender-selector { display:flex; justify-content:center; gap:15px; margin:12px 0; }
.gender-option { display:flex; align-items:center; gap:6px; padding:8px 14px; background:rgba(0,0,0,0.05); border-radius:10px; cursor:pointer; border:2px solid transparent; }
.gender-option.selected { background:rgba(52,152,219,0.3); border-color:#3498DB; }
.current-info { background:rgba(0,0,0,0.08); padding:8px; border-radius:8px; margin-bottom:8px; color:#2C3E50; font-weight:bold; }
.admin-warning { background:rgba(231,76,60,0.1); border:2px solid rgba(231,76,60,0.3); padding:12px; border-radius:10px; margin:10px 0; color:#E74C3C; font-weight:bold; }
.player-list { max-height:280px; overflow-y:auto; margin:15px 0; }
.player-item { background:rgba(0,0,0,0.05); padding:10px 14px; margin:6px 0; border-radius:10px; cursor:pointer; text-align:right; }
.player-item:hover { background:rgba(0,0,0,0.1); }
.player-name { font-weight:bold; color:#2C3E50; }
.member-list { background:rgba(0,0,0,0.05); padding:12px; border-radius:10px; margin-top:12px; text-align:right; }
.member-item { display:flex; justify-content:space-between; padding:6px 0; border-bottom:1px solid rgba(0,0,0,0.1); }
.member-role { font-size:0.85rem; color:#8E44AD; background:rgba(142,68,173,0.15); padding:2px 8px; border-radius:8px; }
.chat-container { position:fixed; bottom:20px; right:20px; width:330px; height:380px; background:rgba(255,255,255,0.95); border-radius:15px; box-shadow:0 10px 30px rgba(0,0,0,0.3); display:flex; flex-direction:column; z-index:1000; transition:all 0.3s ease; }
.chat-container.minimized { height:48px; overflow:hidden; }
.chat-header { background:linear-gradient(45deg,#4ECDC4,#45B7D1); color:white; padding:12px 15px; border-radius:13px 13px 0 0; display:flex; justify-content:space-between; cursor:pointer; font-weight:bold; }
.chat-messages { flex:1; padding:10px; overflow-y:auto; background:rgba(255,255,255,0.85); }
.chat-message { margin-bottom:6px; padding:6px 10px; border-radius:10px; font-size:0.85rem; }
.chat-message.own { background:linear-gradient(45deg,#4ECDC4,#45B7D1); color:white; margin-left:18px; }
.chat-message.other { background:rgba(0,0,0,0.06); margin-right:18px; }
.chat-message.sys { background:linear-gradient(45deg,#F39C12,#E67E22); color:white; text-align:center; }
.message-sender { font-weight:bold; font-size:0.75rem; opacity:0.8; }
.chat-input-container { display:flex; padding:8px; gap:6px; }
.chat-input { flex:1; padding:8px 12px; border:2px solid #ddd; border-radius:20px; font-size:0.85rem; }
.chat-send-btn { background:linear-gradient(45deg,#4ECDC4,#45B7D1); color:white; border:none; padding:8px 14px; border-radius:20px; cursor:pointer; }
.game-container { display:flex; width:100%; min-height:100vh; }
.main-game { flex:1; padding:20px; display:flex; flex-direction:column; align-items:center; justify-content:center; position:relative; }
.stats { background:rgba(255,255,255,0.25); border-radius:25px; padding:25px; margin-bottom:25px; text-align:center; border:2px solid rgba(255,255,255,0.3); min-width:300px; }
.cookie-count { font-size:3.2rem; font-weight:bold; color:#FF8C00; margin-bottom:8px; animation:pulse 2s infinite; }
@keyframes pulse { 0%,100%{transform:scale(1);} 50%{transform:scale(1.05);} }
.cookies-per-second, .cookies-per-click { font-size:1.1rem; color:#2C3E50; font-weight:bold; background:rgba(255,255,255,0.7); padding:6px 14px; border-radius:15px; margin-bottom:8px; }
.prestige-info { font-size:1rem; color:#8E44AD; font-weight:bold; margin-bottom:8px; }
.prestige-button { background:linear-gradient(45deg,#8E44AD,#E74C3C); color:white; padding:10px 20px; border:none; border-radius:20px; font-size:1rem; font-weight:bold; cursor:pointer; }
.cookie-container { position:relative; margin:25px 0; }
.cookie { width:230px; height:230px; background:radial-gradient(circle at 30% 30%, #DEB887, #D2691E, #8B4513); border-radius:50%; cursor:pointer; border:8px solid #654321; box-shadow:0 15px 30px rgba(0,0,0,0.3); animation:cookieFloat 4s ease-in-out infinite; }
@keyframes cookieFloat { 0%,100%{transform:translateY(0px);} 50%{transform:translateY(-10px);} }
.cookie:hover { transform:scale(1.08); box-shadow:0 0 20px rgba(255,215,0,0.5); }
.cookie:active { transform:scale(0.92); }
.click-effect { position:absolute; color:#FFD700; font-size:2.2rem; font-weight:bold; pointer-events:none; animation:floatUp 1.2s ease-out forwards; z-index:1000; }
@keyframes floatUp { 0%{opacity:1;transform:translateY(0) scale(1);} 100%{opacity:0;transform:translateY(-120px) scale(1.5);} }
.particle { position:absolute; width:4px; height:4px; border-radius:50%; pointer-events:none; animation:particleFloat 2s ease-out forwards; }
@keyframes particleFloat { 0%{opacity:1;transform:translateY(0) scale(1);} 100%{opacity:0;transform:translateY(-100px) scale(0);} }
.shop { width:380px; background:rgba(255,255,255,0.15); border-left:2px solid rgba(255,255,255,0.3); padding:20px; overflow-y:auto; max-height:100vh; }
.shop h2 { color:#2C3E50; font-size:1.9rem; margin-bottom:18px; text-align:center; background:rgba(255,255,255,0.3); padding:12px; border-radius:15px; }
.shop-section { margin-bottom:20px; }
.shop-section h3 { color:#34495E; font-size:1.25rem; margin-bottom:12px; text-align:center; background:rgba(255,255,255,0.2); padding:8px; border-radius:10px; }
.shop-item { background:rgba(255,255,255,0.2); border-radius:15px; padding:12px; margin-bottom:10px; border:2px solid rgba(255,255,255,0.3); cursor:pointer; transition:all 0.2s ease; }
.shop-item:hover { background:rgba(255,255,255,0.3); transform:translateY(-2px); }
.shop-item.disabled { opacity:0.4; cursor:not-allowed; }
.item-header { display:flex; justify-content:space-between; margin-bottom:6px; }
.item-name { font-size:1.05rem; font-weight:bold; color:#2C3E50; }
.item-count { background:linear-gradient(45deg,#FF6B6B,#4ECDC4); color:white; padding:2px 10px; border-radius:15px; font-size:0.85rem; font-weight:bold; }
.item-description { font-size:0.85rem; color:#34495E; margin-bottom:6px; }
.item-price { font-size:1rem; font-weight:bold; color:#E67E22; text-align:center; background:rgba(255,255,255,0.4); padding:6px; border-radius:10px; }
.item-production { font-size:0.8rem; color:#27AE60; font-weight:bold; text-align:center; margin-top:4px; }
.error-message { color:#E74C3C; font-weight:bold; margin:8px 0; }
.success-message { color:#27AE60; font-weight:bold; margin:8px 0; }
.info-message { color:#2980B9; font-weight:bold; margin:8px 0; }
.firebase-status { position:fixed; bottom:20px; left:20px; background:rgba(255,255,255,0.9); padding:8px 14px; border-radius:10px; font-size:0.85rem; z-index:1000; }
.firebase-connected { color:#27AE60; border:2px solid #27AE60; }
.firebase-disconnected { color:#E74C3C; border:2px solid #E74C3C; }
.achievement-popup { position:fixed; top:20px; right:20px; background:linear-gradient(135deg,#4CAF50,#45a049); color:white; padding:18px 22px; border-radius:15px; box-shadow:0 10px 30px rgba(0,0,0,0.3); transform:translateX(450px); transition:transform 0.6s ease; z-index:3000; min-width:220px; }
.achievement-popup.show { transform:translateX(0); }
.golden-cookie { position:fixed; width:55px; height:55px; background:radial-gradient(circle,#FFD700,#FFA500); border-radius:50%; cursor:pointer; z-index:2500; animation:goldenFloat 3s ease-in-out infinite; box-shadow:0 0 20px rgba(255,215,0,0.8); border:3px solid #FF8C00; }
@keyframes goldenFloat { 0%,100%{transform:scale(1);} 50%{transform:scale(1.1);} }
.floating-cookies { position:fixed; top:0; left:0; width:100%; height:100%; pointer-events:none; z-index:-1; }
.floating-cookie { position:absolute; width:30px; height:30px; background:radial-gradient(circle,#DEB887,#D2691E); border-radius:50%; opacity:0.12; animation:floatBg 25s infinite linear; }
@keyframes floatBg { 0%{transform:translateY(100vh);} 100%{transform:translateY(-100px);} }
@media (max-width:768px){ .game-container{flex-direction:column;} .shop{width:100%; max-height:50vh;} .cookie{width:170px;height:170px;} .cookie-count{font-size:2.2rem;} .chat-container{width:90%;right:5%;} }
</style>
</head>
<body>
<div class="login-screen" id="loginScreen">
    <div class="login-form">
        <h2>🍪 משחק האימפריית עוגיות 🍪</h2>
        <input type="text" id="usernameInput" placeholder="שם משתמש" maxlength="20">
        <input type="password" id="passwordInput" placeholder="סיסמה">
        <div class="remember-me">
            <input type="checkbox" id="rememberMe" checked>
            <label for="rememberMe">זכור אותי</label>
        </div>
        <button id="loginBtn" onclick="login()">התחבר</button>
        <button id="registerBtn" onclick="register()">הרשם</button>
        <div id="loginMessage"></div>
    </div>
</div>

<div class="top-buttons">
    <div id="clubButtonsContainer">
        <button class="club-button" onclick="openJoinClubModal()">🏆 הצטרף למועדון</button>
        <button class="club-button" onclick="openCreateClubModal()">⭐ צור מועדון</button>
    </div>
    <button class="donate-button" onclick="openDonateModal()">🎁 תרום עוגיות</button>
</div>

<button class="settings-button" onclick="openSettingsModal()">⚙️ הגדרות</button>
<button class="admin-button" onclick="openAdminModal()">👑 מסך יוצר</button>

<div class="modal" id="joinClubModal">
    <div class="modal-content">
        <h3>🏆 הצטרפות למועדון</h3>
        <input type="text" id="clubCodeInput" placeholder="קוד מועדון (4 ספרות)" maxlength="4">
        <button id="joinClubBtn" onclick="joinClub()">אישור</button>
        <button class="close-button" onclick="closeModal('joinClubModal')">ביטול</button>
        <div id="joinClubMessage"></div>
    </div>
</div>

<div class="modal" id="createClubModal">
    <div class="modal-content">
        <h3>⭐ יצירת מועדון</h3>
        <input type="text" id="clubNameInput" placeholder="שם המועדון" maxlength="30">
        <button id="createClubBtn" onclick="createClub()">צור</button>
        <button class="close-button" onclick="closeModal('createClubModal')">ביטול</button>
        <div id="createClubMessage"></div>
    </div>
</div>

<div class="modal" id="clubInfoModal">
    <div class="modal-content">
        <h3 id="clubInfoName">🏆 המועדון שלי</h3>
        <div id="clubInfoCode"></div>
        <div id="clubInfoStats"></div>
        <div class="member-list"><h4>חברים:</h4><div id="memberList"></div></div>
        <button onclick="leaveClub()">עזוב מועדון</button>
        <button class="close-button" onclick="closeModal('clubInfoModal')">סגור</button>
    </div>
</div>

<div class="modal" id="donateModal">
    <div class="modal-content">
        <h3>🎁 בחר שחקן לתרומה</h3>
        <div class="player-list" id="playerList"></div>
        <button class="close-button" onclick="closeModal('donateModal')">ביטול</button>
        <div id="donateMessage"></div>
    </div>
</div>

<div class="modal" id="donationAmountModal">
    <div class="modal-content">
        <h3>🎁 תרום עוגיות</h3>
        <p>לשחקן: <strong id="donationTargetPlayer"></strong></p>
        <p>יש לך: <strong id="currentCookiesDisplay"></strong> עוגיות</p>
        <input type="number" id="donationAmountInput" placeholder="כמות עוגיות" min="1">
        <button id="confirmDonationBtn" onclick="confirmDonation()">אשר תרומה</button>
        <button class="close-button" onclick="closeModal('donationAmountModal')">ביטול</button>
        <div id="donationMessage"></div>
    </div>
</div>

<div class="modal" id="settingsModal">
    <div class="modal-content">
        <h3>⚙️ הגדרות חשבון</h3>
        <div class="settings-section">
            <h4>👤 שם משתמש</h4>
            <div class="current-info">נוכחי: <span id="currentUsername"></span></div>
            <input type="text" id="newUsernameInput" placeholder="שם משתמש חדש" maxlength="20">
            <input type="password" id="usernameChangePasswordInput" placeholder="הסיסמה שלך (לאימות)">
            <button id="changeUsernameBtn" onclick="changeUsername()">שנה שם משתמש</button>
        </div>
        <div class="settings-section">
            <h4>🔒 סיסמה</h4>
            <input type="password" id="currentPasswordInput" placeholder="סיסמה נוכחית">
            <input type="password" id="newPasswordInput" placeholder="סיסמה חדשה">
            <input type="password" id="confirmPasswordInput" placeholder="אשר סיסמה חדשה">
            <button id="changePasswordBtn" onclick="changePassword()">שנה סיסמה</button>
        </div>
        <div class="settings-section">
            <h4>👫 מגדר</h4>
            <div class="current-info">נוכחי: <span id="currentGender">לא נבחר</span></div>
            <div class="gender-selector">
                <label class="gender-option" id="maleOption"><input type="radio" name="gender" value="male"> 👦 בן</label>
                <label class="gender-option" id="femaleOption"><input type="radio" name="gender" value="female"> 👧 בת</label>
            </div>
            <button id="changeGenderBtn" onclick="changeGender()">עדכן מגדר</button>
        </div>
        <button class="close-button" onclick="doLogout()">🚪 התנתק</button>
        <button class="close-button" onclick="closeModal('settingsModal')">סגור</button>
        <div id="settingsMessage"></div>
    </div>
</div>

<div class="modal" id="adminModal">
    <div class="modal-content">
        <h3>👑 מסך יוצר המשחק</h3>
        <div class="admin-warning">⚠️ אזור מוגבל ליוצרים בלבד ⚠️</div>
        <div class="admin-section">
            <h4>🔐 אימות</h4>
            <input type="password" id="adminPasswordInput" placeholder="סיסמת יוצר">
            <button id="verifyAdminBtn" onclick="verifyAdmin()">אמת גישה</button>
        </div>
        <div class="admin-section" id="adminActionsSection" style="display:none;">
            <h4>💰 העברת עוגיות</h4>
            <input type="text" id="targetPlayerInput" placeholder="שם השחקן">
            <input type="number" id="cookieAmountInput" placeholder="כמות עוגיות" min="1">
            <button id="adminTransferBtn" onclick="adminTransferCookies()">העבר עוגיות</button>
        </div>
        <button class="close-button" onclick="closeModal('adminModal')">סגור</button>
        <div id="adminMessage"></div>
    </div>
</div>

<div class="floating-cookies" id="floatingCookies"></div>
<div class="firebase-status" id="firebaseStatus">🔄 מתחבר...</div>

<div class="chat-container" id="chatContainer">
    <div class="chat-header" onclick="toggleChat()">
        <span>💬 צ'ט גלובלי</span><span id="chatToggleIcon">−</span>
    </div>
    <div class="chat-messages" id="chatMessages"></div>
    <div class="chat-input-container">
        <input type="text" class="chat-input" id="chatInput" placeholder="הקלד הודעה..." maxlength="200">
        <button class="chat-send-btn" onclick="sendMessage()">שלח</button>
    </div>
</div>

<div class="game-container">
    <div class="main-game">
        <div class="stats">
            <div class="cookie-count" id="cookieCount">0</div>
            <div class="cookies-per-second" id="cpsDisplay">0 עוגיות לשנייה</div>
            <div class="cookies-per-click" id="cpcDisplay">+1 ללחיצה</div>
            <div class="prestige-info" id="prestigeInfo">רמת יוקרה: 0 | כפולה: x1</div>
            <button class="prestige-button" onclick="prestige()">🌟 איפוס יוקרה 🌟</button>
        </div>
        <div class="cookie-container"><div class="cookie" id="cookie"></div></div>
    </div>
    <div class="shop">
        <h2>🛒 חנות 🛒</h2>
        <div class="shop-section"><h3>🏭 מבנים</h3><div id="buildingItems"></div></div>
        <div class="shop-section"><h3>👆 שדרוגי לחיצה</h3><div id="clickUpgradeItems"></div></div>
        <div class="shop-section"><h3>⚡ שדרוגים</h3><div id="upgradeItems"></div></div>
        <div class="shop-section"><h3>🎯 הישגים</h3><div id="achievementItems"></div></div>
    </div>
</div>

<div class="achievement-popup" id="achievementPopup"><div id="achievementText"></div></div>

<script>
var firebaseConfig = {
    apiKey: "AIzaSyALFR2iaqpGS8kSWUr51VnUqPiVFoasHoQ",
    authDomain: "logi2-467e6.firebaseapp.com",
    databaseURL: "https://logi2-467e6-default-rtdb.europe-west1.firebasedatabase.app",
    projectId: "logi2-467e6",
    storageBucket: "logi2-467e6.firebasestorage.app",
    messagingSenderId: "263662852070",
    appId: "1:263662852070:web:ed2c3cdb5eca7b2ef62827"
};

var database, auth;
var firebaseInit = false;
try {
    firebase.initializeApp(firebaseConfig);
    database = firebase.database();
    auth = firebase.auth();
    firebaseInit = true;
    setFirebaseStatus(true);
} catch (err) {
    console.log("Firebase init error:", err);
    setFirebaseStatus(false);
}

function setFirebaseStatus(ok) {
    var el = document.getElementById("firebaseStatus");
    if (ok) { el.className = "firebase-status firebase-connected"; el.textContent = "✅ מחובר ל-Firebase"; }
    else { el.className = "firebase-status firebase-disconnected"; el.textContent = "❌ מצב אופליין"; }
}

function withTimeout(promise, ms) {
    return Promise.race([
        promise,
        new Promise(function(_, reject) { setTimeout(function() { reject(new Error("timeout")); }, ms); })
    ]);
}

function emailFor(username) {
    return username.toLowerCase().replace(/[^a-z0-9\u0590-\u05FF]/g, "") + "@cookiegame.local";
}

var cookies = 0, cookiesPerSecond = 0, clickPower = 1;
var prestigeLevel = 0, prestigeMultiplier = 1, totalCookiesBaked = 0;
var goldenCookieClicks = 0, lastGoldenCookie = 0, totalClicks = 0;

var currentUid = null, currentUsername = null, currentClub = null;
var usersCache = {};
var clubsCache = {};
var chatMessages = [];
var selectedDonationTarget = null;
var unlockedAchievements = [];

var buildings = {
    cursor: { name:"סמן אוטומטי", emoji:"👆", description:"לוחץ אוטומטית", baseCost:15, cps:0.1, count:0 },
    grandma: { name:"סבתא", emoji:"👵", description:"אופה עוגיות", baseCost:100, cps:1, count:0 },
    farm: { name:"חווה", emoji:"🚜", description:"מגדלת עוגיות", baseCost:1100, cps:8, count:0 },
    mine: { name:"מכרה", emoji:"⛏", description:"חופר עוגיות", baseCost:12000, cps:47, count:0 },
    factory: { name:"מפעל", emoji:"🏭", description:"מייצר עוגיות", baseCost:130000, cps:260, count:0 },
    bank: { name:"בנק", emoji:"🏦", description:"ריבית עוגיות", baseCost:1400000, cps:1400, count:0 },
    temple: { name:"מקדש", emoji:"🏛", description:"מקדש קדוש", baseCost:20000000, cps:7800, count:0 },
    wizardTower: { name:"מגדל קוסמים", emoji:"🧙", description:"מזמן עוגיות", baseCost:330000000, cps:44000, count:0 },
    shipment: { name:"חללית", emoji:"🚀", description:"מביאה מהחלל", baseCost:5100000000, cps:260000, count:0 },
    alchemyLab: { name:"מעבדה אלכימית", emoji:"⚗", description:"הופכת חומרים", baseCost:75000000000, cps:1600000, count:0 },
    portal: { name:"פורטל", emoji:"🌀", description:"ממדים אחרים", baseCost:1000000000000, cps:10000000, count:0 },
    timeMachine: { name:"מכונת זמן", emoji:"⏰", description:"מהעתיד", baseCost:14000000000000, cps:65000000, count:0 }
};

var clickUpgrades = {
    clickPower1: { name:"אצבע חזקה", emoji:"👆", description:"+1 ללחיצה", baseCost:50, clickBonus:1, level:0 },
    clickPower2: { name:"יד ברזל", emoji:"✊", description:"+5 ללחיצה", baseCost:500, clickBonus:5, level:0 },
    clickPower3: { name:"זרוע טיטניום", emoji:"🦾", description:"+25 ללחיצה", baseCost:5000, clickBonus:25, level:0 },
    clickPower4: { name:"כוח על-אנושי", emoji:"⚡", description:"+100 ללחיצה", baseCost:50000, clickBonus:100, level:0 },
    clickPower5: { name:"אצבע אלוהית", emoji:"🌟", description:"+500 ללחיצה", baseCost:500000, clickBonus:500, level:0 }
};

var upgrades = {
    reinforcedIndex: { name:"אצבע מחוזקת", emoji:"💪", description:"כפולה x2 לסמנים", cost:100, condition:function(){return buildings.cursor.count>=1;}, effect:function(){buildings.cursor.cps*=2;}, purchased:false },
    carpalTunnel: { name:"מניעת פציעה", emoji:"🖱", description:"כפולה x2 לסמנים", cost:500, condition:function(){return buildings.cursor.count>=5;}, effect:function(){buildings.cursor.cps*=2;}, purchased:false },
    ambidextrous: { name:"דו-צדדי", emoji:"👐", description:"כפולה x2 לסמנים", cost:10000, condition:function(){return buildings.cursor.count>=25;}, effect:function(){buildings.cursor.cps*=2;}, purchased:false },
    grandmaRecipes: { name:"מתכונים מסבתא", emoji:"📧", description:"כפולה x2 לסבתות", cost:1000, condition:function(){return buildings.grandma.count>=1;}, effect:function(){buildings.grandma.cps*=2;}, purchased:false },
    rollingPins: { name:"מערוכי מתכת", emoji:"🍴", description:"כפולה x2 לסבתות", cost:5000, condition:function(){return buildings.grandma.count>=5;}, effect:function(){buildings.grandma.cps*=2;}, purchased:false },
    lubrication: { name:"שיפור מכונות", emoji:"🛠", description:"כפולה x2 למפעלים", cost:650000, condition:function(){return buildings.factory.count>=1;}, effect:function(){buildings.factory.cps*=2;}, purchased:false },
    goldenTouch: { name:"מגע הזהב", emoji:"✨", description:"כפולה x3 לכל הייצור", cost:10000000, condition:function(){return goldenCookieClicks>=5;}, effect:function(){for(var k in buildings){buildings[k].cps*=3;}}, purchased:false }
};

var achievements = [
    { id:"first_click", name:"הלחיצה הראשונה", emoji:"👆", desc:"לחצת על העוגייה!", condition:function(){return totalCookiesBaked>=1;} },
    { id:"hundred", name:"מאה עוגיות", emoji:"💯", desc:"100 עוגיות!", condition:function(){return totalCookiesBaked>=100;} },
    { id:"first_building", name:"הבנייה הראשונה", emoji:"🏗", desc:"קנית בניין!", condition:function(){for(var k in buildings){if(buildings[k].count>0)return true;}return false;} },
    { id:"thousand", name:"אלף עוגיות", emoji:"🎯", desc:"1,000 עוגיות!", condition:function(){return totalCookiesBaked>=1000;} },
    { id:"ten_thousand", name:"עשרת אלפים", emoji:"🏆", desc:"10,000 עוגיות!", condition:function(){return totalCookiesBaked>=10000;} },
    { id:"hundred_thousand", name:"מאה אלף", emoji:"🎖", desc:"100,000 עוגיות!", condition:function(){return totalCookiesBaked>=100000;} },
    { id:"million", name:"מיליון עוגיות", emoji:"💎", desc:"מיליון עוגיות!", condition:function(){return totalCookiesBaked>=1000000;} },
    { id:"first_golden", name:"עוגיית הזהב", emoji:"🌟", desc:"לחצת על עוגיית זהב!", condition:function(){return goldenCookieClicks>=1;} },
    { id:"golden_master", name:"מאסטר הזהב", emoji:"👑", desc:"10 עוגיות זהב!", condition:function(){return goldenCookieClicks>=10;} },
    { id:"hundred_buildings", name:"מאה בניינים", emoji:"🏙", desc:"100 בניינים!", condition:function(){var s=0;for(var k in buildings){s+=buildings[k].count;}return s>=100;} },
    { id:"first_prestige", name:"יוקרה ראשונה", emoji:"⭐", desc:"איפוס יוקרה!", condition:function(){return prestigeLevel>=1;} }
];

function formatNum(num) {
    if (num < 1000) return Math.floor(num).toString();
    if (num < 1000000) return (num/1000).toFixed(1) + "K";
    if (num < 1000000000) return (num/1000000).toFixed(1) + "M";
    if (num < 1000000000000) return (num/1000000000).toFixed(1) + "B";
    return (num/1000000000000).toFixed(1) + "T";
}

function calcClickPower() {
    clickPower = 1;
    for (var k in clickUpgrades) { clickPower += clickUpgrades[k].level * clickUpgrades[k].clickBonus; }
}
function calcCPS() {
    cookiesPerSecond = 0;
    for (var k in buildings) { cookiesPerSecond += buildings[k].count * buildings[k].cps * prestigeMultiplier; }
}
function updateDisplay() {
    document.getElementById("cookieCount").textContent = formatNum(cookies);
    document.getElementById("cpsDisplay").textContent = formatNum(cookiesPerSecond) + " עוגיות לשנייה";
    document.getElementById("cpcDisplay").textContent = "+" + formatNum(Math.floor(clickPower*prestigeMultiplier)) + " ללחיצה";
    document.getElementById("prestigeInfo").textContent = "רמת יוקרה: " + prestigeLevel + " | כפולה: x" + prestigeMultiplier.toFixed(1);
}

function getCost(b) { return Math.floor(b.baseCost * Math.pow(1.15, b.count)); }
function getClickCost(u) { return Math.floor(u.baseCost * Math.pow(2, u.level)); }

function makeShop() {
    var bc = document.getElementById("buildingItems");
    bc.innerHTML = "";
    for (var key in buildings) {
        (function(key) {
            var b = buildings[key], cost = getCost(b), afford = cookies >= cost;
            var itm = document.createElement("div");
            itm.className = afford ? "shop-item" : "shop-item disabled";
            itm.innerHTML = "<div class='item-header'><span class='item-name'>"+b.emoji+" "+b.name+"</span><span class='item-count'>"+b.count+"</span></div><div class='item-description'>"+b.description+"</div><div class='item-production'>מייצר: "+formatNum(b.cps*prestigeMultiplier)+"/שנייה</div><div class='item-price'>💰 "+formatNum(cost)+"</div>";
            if (afford) itm.addEventListener("click", function(){ buyBuilding(key); });
            bc.appendChild(itm);
        })(key);
    }
    var cc = document.getElementById("clickUpgradeItems");
    cc.innerHTML = "";
    for (var key2 in clickUpgrades) {
        (function(key2) {
            var u = clickUpgrades[key2], cost = getClickCost(u), afford = cookies >= cost;
            var itm = document.createElement("div");
            itm.className = afford ? "shop-item" : "shop-item disabled";
            itm.innerHTML = "<div class='item-header'><span class='item-name'>"+u.emoji+" "+u.name+"</span><span class='item-count'>רמה "+u.level+"</span></div><div class='item-description'>"+u.description+"</div><div class='item-price'>💰 "+formatNum(cost)+"</div>";
            if (afford) itm.addEventListener("click", function(){ buyClickUpgrade(key2); });
            cc.appendChild(itm);
        })(key2);
    }
    var uc = document.getElementById("upgradeItems");
    uc.innerHTML = "";
    for (var key3 in upgrades) {
        (function(key3) {
            var u = upgrades[key3];
            if (u.purchased || !u.condition()) return;
            var afford = cookies >= u.cost;
            var itm = document.createElement("div");
            itm.className = afford ? "shop-item" : "shop-item disabled";
            itm.innerHTML = "<div class='item-header'><span class='item-name'>"+u.emoji+" "+u.name+"</span><span class='item-count'>🆕</span></div><div class='item-description'>"+u.description+"</div><div class='item-price'>💰 "+formatNum(u.cost)+"</div>";
            if (afford) itm.addEventListener("click", function(){ buyUpgrade(key3); });
            uc.appendChild(itm);
        })(key3);
    }
    var ac = document.getElementById("achievementItems");
    ac.innerHTML = "";
    achievements.forEach(function(a) {
        var unlocked = unlockedAchievements.indexOf(a.id) !== -1;
        var itm = document.createElement("div");
        itm.className = unlocked ? "shop-item" : "shop-item disabled";
        itm.style.cursor = "default";
        itm.innerHTML = "<div class='item-header'><span class='item-name'>"+a.emoji+" "+a.name+"</span><span class='item-count'>"+(unlocked?"✅":"❌")+"</span></div><div class='item-description'>"+a.desc+"</div>";
        ac.appendChild(itm);
    });
}

function buyBuilding(key) {
    var b = buildings[key], cost = getCost(b);
    if (cookies >= cost) {
        cookies -= cost; b.count++; totalCookiesBaked += cost;
        calcClickPower(); calcCPS(); updateDisplay(); makeShop(); checkAchievements(); saveUserData();
    }
}
function buyClickUpgrade(key) {
    var u = clickUpgrades[key], cost = getClickCost(u);
    if (cookies >= cost) {
        cookies -= cost; u.level++; totalCookiesBaked += cost;
        calcClickPower(); updateDisplay(); makeShop(); checkAchievements(); saveUserData();
    }
}
function buyUpgrade(key) {
    var u = upgrades[key];
    if (cookies >= u.cost && !u.purchased) {
        cookies -= u.cost; totalCookiesBaked += u.cost; u.purchased = true; u.effect();
        calcCPS(); updateDisplay(); makeShop(); checkAchievements();
        showAchievement({ name:"שדרוג נקנה!", desc:u.name, emoji:"⚡" });
        saveUserData();
    }
}

function showClickEffect(x, y) {
    var eff = document.createElement("div");
    eff.className = "click-effect";
    eff.textContent = "+" + formatNum(clickPower*prestigeMultiplier);
    eff.style.left = (x-25)+"px"; eff.style.top = (y-25)+"px";
    document.body.appendChild(eff);
    setTimeout(function(){ if(document.body.contains(eff)) document.body.removeChild(eff); }, 1200);
}
function makeParticles(x, y, color) {
    for (var i=0;i<8;i++) {
        var p = document.createElement("div");
        p.className = "particle"; p.style.background = color;
        p.style.left = x+"px"; p.style.top = y+"px";
        p.style.transform = "translate("+((Math.random()-0.5)*100)+"px,"+((Math.random()-0.5)*100)+"px)";
        document.body.appendChild(p);
        setTimeout(function(){ if(document.body.contains(p)) document.body.removeChild(p); }, 2000);
    }
}
function checkAchievements() {
    achievements.forEach(function(a) {
        if (unlockedAchievements.indexOf(a.id) === -1 && a.condition()) {
            unlockedAchievements.push(a.id);
            showAchievement(a);
        }
    });
}
function showAchievement(a) {
    var popup = document.getElementById("achievementPopup");
    document.getElementById("achievementText").innerHTML = "<strong>🏆</strong><br>"+a.emoji+" "+a.name+"<br>"+a.desc;
    popup.classList.add("show");
    setTimeout(function(){ popup.classList.remove("show"); }, 4000);
}
function makeFloatingCookies() {
    var c = document.getElementById("floatingCookies");
    for (var i=0;i<15;i++) {
        var el = document.createElement("div");
        el.className = "floating-cookie";
        el.style.left = Math.random()*100+"%";
        el.style.animationDelay = Math.random()*25+"s";
        el.style.animationDuration = (20+Math.random()*10)+"s";
        c.appendChild(el);
    }
}
function spawnGoldenCookie() {
    if (Date.now() - lastGoldenCookie < 30000) return;
    var gc = document.createElement("div");
    gc.className = "golden-cookie";
    gc.style.left = Math.random()*(window.innerWidth-60)+"px";
    gc.style.top = Math.random()*(window.innerHeight-60)+"px";
    gc.onclick = function(e) {
        goldenCookieClicks++;
        var bonus = Math.floor(cookies*0.15) + Math.floor(cookiesPerSecond*77);
        cookies += bonus; totalCookiesBaked += bonus;
        showClickEffect(e.clientX, e.clientY);
        makeParticles(e.clientX, e.clientY, "#FFD700");
        showAchievement({ name:"עוגיית זהב!", desc:"+"+formatNum(bonus), emoji:"🌟" });
        document.body.removeChild(gc);
        lastGoldenCookie = Date.now();
        checkAchievements(); updateDisplay();
    };
    document.body.appendChild(gc);
    setTimeout(function(){ if(document.body.contains(gc)) document.body.removeChild(gc); }, 13000);
}
function prestige() {
    if (totalCookiesBaked < 1000000) {
        showAchievement({ name:"לא מוכן", desc:"צריך מיליון עוגיות לפחות", emoji:"❌" });
        return;
    }
    if (confirm("איפוס יוקרה יאפס הכל אבל ייתן בונוס קבוע. להמשיך?")) {
        prestigeLevel++; prestigeMultiplier = 1 + prestigeLevel*0.1;
        cookies = 0; totalCookiesBaked = 0;
        for (var k in buildings) buildings[k].count = 0;
        for (var k2 in upgrades) upgrades[k2].purchased = false;
        calcCPS(); updateDisplay(); makeShop(); checkAchievements();
        showAchievement({ name:"איפוס יוקרה!", desc:"רמה "+prestigeLevel, emoji:"⭐" });
        saveUserData();
    }
}

function login() {
    var username = document.getElementById("usernameInput").value.trim();
    var pass = document.getElementById("passwordInput").value;
    var remember = document.getElementById("rememberMe").checked;
    var msg = document.getElementById("loginMessage");
    var loginBtn = document.getElementById("loginBtn");
    var registerBtn = document.getElementById("registerBtn");

    if (!username || !pass) { msg.innerHTML = "<div class='error-message'>הזן שם וסיסמה</div>"; return; }
    if (!firebaseInit) { msg.innerHTML = "<div class='error-message'>Firebase לא מחובר</div>"; return; }

    loginBtn.disabled = true; registerBtn.disabled = true;
    msg.innerHTML = "<div class='info-message'>מתחבר...</div>";

    var persistence = remember ? firebase.auth.Auth.Persistence.LOCAL : firebase.auth.Auth.Persistence.SESSION;

    auth.setPersistence(persistence).then(function() {
        return withTimeout(auth.signInWithEmailAndPassword(emailFor(username), pass), 10000);
    }).then(function(cred) {
        return withTimeout(database.ref("users/" + cred.user.uid).once("value"), 10000);
    }).then(function(snap) {
        if (!snap.exists()) throw new Error("no-data");
        finishLogin(auth.currentUser.uid, snap.val());
    }).catch(function(e) {
        console.log("Login error:", e);
        var code = e.code || e.message;
        if (code === "auth/user-not-found" || code === "auth/wrong-password" || code === "auth/invalid-credential" || code === "auth/invalid-login-credentials") {
            msg.innerHTML = "<div class='error-message'>שם משתמש או סיסמה שגויים</div>";
        } else if (code === "timeout") {
            msg.innerHTML = "<div class='error-message'>אין תגובה מהשרת - נסה שוב</div>";
        } else {
            msg.innerHTML = "<div class='error-message'>שגיאה: " + code + "</div>";
        }
    }).finally(function() {
        loginBtn.disabled = false; registerBtn.disabled = false;
    });
}

function register() {
    var username = document.getElementById("usernameInput").value.trim();
    var pass = document.getElementById("passwordInput").value;
    var remember = document.getElementById("rememberMe").checked;
    var msg = document.getElementById("loginMessage");
    var loginBtn = document.getElementById("loginBtn");
    var registerBtn = document.getElementById("registerBtn");

    if (!username || !pass) { msg.innerHTML = "<div class='error-message'>הזן שם וסיסמה</div>"; return; }
    if (username.length < 3) { msg.innerHTML = "<div class='error-message'>שם קצר מדי</div>"; return; }
    if (pass.length < 6) { msg.innerHTML = "<div class='error-message'>סיסמה חייבת 6 תווים לפחות</div>"; return; }
    if (!firebaseInit) { msg.innerHTML = "<div class='error-message'>Firebase לא מחובר</div>"; return; }

    loginBtn.disabled = true; registerBtn.disabled = true;
    msg.innerHTML = "<div class='info-message'>נרשם...</div>";

    var persistence = remember ? firebase.auth.Auth.Persistence.LOCAL : firebase.auth.Auth.Persistence.SESSION;
    var createdUid = null;

    auth.setPersistence(persistence).then(function() {
        return withTimeout(auth.createUserWithEmailAndPassword(emailFor(username), pass), 10000);
    }).then(function(cred) {
        createdUid = cred.user.uid;
        var newUser = {
            username: username, cookies:0, totalClicks:0, totalCookiesBaked:0,
            buildings: JSON.parse(JSON.stringify(buildings)),
            clickUpgrades: JSON.parse(JSON.stringify(clickUpgrades)),
            upgrades: JSON.parse(JSON.stringify(upgrades)),
            prestigeLevel:0, prestigeMultiplier:1, club:null, gender:null, created:Date.now()
        };
        return withTimeout(database.ref("users/" + createdUid).set(newUser), 10000).then(function() { return newUser; });
    }).then(function(newUser) {
        msg.innerHTML = "<div class='success-message'>נרשמת בהצלחה!</div>";
        finishLogin(createdUid, newUser);
    }).catch(function(e) {
        console.log("Register error:", e);
        var code = e.code || e.message;
        if (code === "auth/email-already-in-use") {
            msg.innerHTML = "<div class='error-message'>שם המשתמש כבר תפוס</div>";
        } else if (code === "auth/weak-password") {
            msg.innerHTML = "<div class='error-message'>סיסמה חלשה מדי</div>";
        } else if (code === "timeout") {
            msg.innerHTML = "<div class='error-message'>אין תגובה מהשרת - נסה שוב</div>";
        } else {
            msg.innerHTML = "<div class='error-message'>שגיאה: " + code + "</div>";
        }
    }).finally(function() {
        loginBtn.disabled = false; registerBtn.disabled = false;
    });
}

function finishLogin(uid, data) {
    currentUid = uid;
    currentUsername = data.username;
    usersCache[uid] = data;
    cookies = data.cookies || 0;
    totalClicks = data.totalClicks || 0;
    totalCookiesBaked = data.totalCookiesBaked || 0;
    prestigeLevel = data.prestigeLevel || 0;
    prestigeMultiplier = data.prestigeMultiplier || 1;
    currentClub = data.club || null;

    for (var k in buildings) buildings[k].count = (data.buildings && data.buildings[k]) ? (data.buildings[k].count||0) : 0;
    for (var k2 in clickUpgrades) clickUpgrades[k2].level = (data.clickUpgrades && data.clickUpgrades[k2]) ? (data.clickUpgrades[k2].level||0) : 0;
    for (var k3 in upgrades) upgrades[k3].purchased = (data.upgrades && data.upgrades[k3]) ? (data.upgrades[k3].purchased||false) : false;

    calcClickPower(); calcCPS(); updateDisplay(); makeShop(); updateClubButtons();
    document.getElementById("loginScreen").style.display = "none";
    startGameLoops();
}

function doLogout() {
    if (!confirm("להתנתק?")) return;
    saveUserData().finally(function() {
        auth.signOut().then(function() {
            currentUid = null; currentUsername = null; currentClub = null; selectedDonationTarget = null;
            cookies=0; cookiesPerSecond=0; clickPower=1; prestigeLevel=0; prestigeMultiplier=1;
            totalCookiesBaked=0; goldenCookieClicks=0; totalClicks=0; unlockedAchievements=[];
            for (var k in buildings) buildings[k].count = 0;
            for (var k2 in clickUpgrades) clickUpgrades[k2].level = 0;
            for (var k3 in upgrades) upgrades[k3].purchased = false;
            document.querySelectorAll(".modal").forEach(function(m){ m.style.display="none"; });
            document.getElementById("loginScreen").style.display = "flex";
            document.getElementById("usernameInput").value = "";
            document.getElementById("passwordInput").value = "";
            document.getElementById("loginMessage").innerHTML = "";
            updateClubButtons();
        });
    });
}

function saveUserData() {
    if (!currentUid) return Promise.resolve();
    var u = usersCache[currentUid] || {};
    u.username = currentUsername;
    u.cookies = cookies; u.totalClicks = totalClicks; u.totalCookiesBaked = totalCookiesBaked;
    u.buildings = JSON.parse(JSON.stringify(buildings));
    u.clickUpgrades = JSON.parse(JSON.stringify(clickUpgrades));
    u.upgrades = JSON.parse(JSON.stringify(upgrades));
    u.prestigeLevel = prestigeLevel; u.prestigeMultiplier = prestigeMultiplier;
    u.club = currentClub; u.lastSave = Date.now();
    usersCache[currentUid] = u;
    if (!firebaseInit) return Promise.resolve();
    return withTimeout(database.ref("users/" + currentUid).update(u), 10000).catch(function(e){ console.log("Save error:", e); });
}

function reauth(currentPassword) {
    var cred = firebase.auth.EmailAuthProvider.credential(emailFor(currentUsername), currentPassword);
    return withTimeout(auth.currentUser.reauthenticateWithCredential(cred), 10000);
}

function openSettingsModal() {
    if (!currentUid) { alert("התחבר קודם"); return; }
    document.getElementById("currentUsername").textContent = currentUsername;
    var data = usersCache[currentUid] || {};
    document.getElementById("currentGender").textContent = data.gender === "male" ? "בן" : data.gender === "female" ? "בת" : "לא נבחר";
    document.getElementById("maleOption").classList.toggle("selected", data.gender === "male");
    document.getElementById("femaleOption").classList.toggle("selected", data.gender === "female");
    document.getElementById("newUsernameInput").value = "";
    document.getElementById("usernameChangePasswordInput").value = "";
    document.getElementById("currentPasswordInput").value = "";
    document.getElementById("newPasswordInput").value = "";
    document.getElementById("confirmPasswordInput").value = "";
    document.getElementById("settingsMessage").innerHTML = "";
    document.getElementById("settingsModal").style.display = "flex";
}

document.querySelectorAll(".gender-option").forEach(function(opt) {
    opt.addEventListener("click", function() {
        document.querySelectorAll(".gender-option").forEach(function(o){ o.classList.remove("selected"); });
        this.classList.add("selected");
        this.querySelector("input").checked = true;
    });
});

function changeUsername() {
    var newName = document.getElementById("newUsernameInput").value.trim();
    var pass = document.getElementById("usernameChangePasswordInput").value;
    var msg = document.getElementById("settingsMessage");
    if (!newName || newName.length < 3) { msg.innerHTML = "<div class='error-message'>שם לא תקין</div>"; return; }
    if (!pass) { msg.innerHTML = "<div class='error-message'>הזן את הסיסמה שלך לאימות</div>"; return; }
    if (newName === currentUsername) { msg.innerHTML = "<div class='error-message'>זה כבר השם שלך</div>"; return; }

    msg.innerHTML = "<div class='info-message'>מעדכן...</div>";
    var newEmail = emailFor(newName);

    reauth(pass).then(function() {
        return withTimeout(auth.currentUser.updateEmail(newEmail), 10000);
    }).then(function() {
        var oldName = currentUsername;
        currentUsername = newName;
        return withTimeout(database.ref("users/" + currentUid + "/username").set(newName), 10000).then(function() {
            if (currentClub && clubsCache[currentClub]) {
                var club = clubsCache[currentClub];
                club.members = club.members.map(function(m) { if (m.uid === currentUid) m.username = newName; return m; });
                return withTimeout(database.ref("clubs/" + currentClub).set(club), 10000);
            }
        });
    }).then(function() {
        document.getElementById("currentUsername").textContent = currentUsername;
        msg.innerHTML = "<div class='success-message'>שונה בהצלחה!</div>";
    }).catch(function(e) {
        console.log(e);
        var code = e.code || e.message;
        if (code === "auth/wrong-password") msg.innerHTML = "<div class='error-message'>סיסמה שגויה</div>";
        else if (code === "auth/email-already-in-use") msg.innerHTML = "<div class='error-message'>שם תפוס</div>";
        else msg.innerHTML = "<div class='error-message'>שגיאה: " + code + "</div>";
    });
}

function changePassword() {
    var cur = document.getElementById("currentPasswordInput").value;
    var np = document.getElementById("newPasswordInput").value;
    var cp = document.getElementById("confirmPasswordInput").value;
    var msg = document.getElementById("settingsMessage");
    if (!cur || !np || !cp) { msg.innerHTML = "<div class='error-message'>מלא את כל השדות</div>"; return; }
    if (np.length < 6) { msg.innerHTML = "<div class='error-message'>סיסמה חייבת 6 תווים לפחות</div>"; return; }
    if (np !== cp) { msg.innerHTML = "<div class='error-message'>הסיסמאות לא תואמות</div>"; return; }

    msg.innerHTML = "<div class='info-message'>מעדכן...</div>";
    reauth(cur).then(function() {
        return withTimeout(auth.currentUser.updatePassword(np), 10000);
    }).then(function() {
        msg.innerHTML = "<div class='success-message'>הסיסמה שונתה!</div>";
        document.getElementById("currentPasswordInput").value = "";
        document.getElementById("newPasswordInput").value = "";
        document.getElementById("confirmPasswordInput").value = "";
    }).catch(function(e) {
        console.log(e);
        var code = e.code || e.message;
        if (code === "auth/wrong-password") msg.innerHTML = "<div class='error-message'>סיסמה נוכחית שגויה</div>";
        else msg.innerHTML = "<div class='error-message'>שגיאה: " + code + "</div>";
    });
}

function changeGender() {
    var sel = document.querySelector("input[name='gender']:checked");
    var msg = document.getElementById("settingsMessage");
    if (!sel) { msg.innerHTML = "<div class='error-message'>בחר מגדר</div>"; return; }
    usersCache[currentUid].gender = sel.value;
    withTimeout(database.ref("users/" + currentUid + "/gender").set(sel.value), 10000).then(function() {
        document.getElementById("currentGender").textContent = sel.value === "male" ? "בן" : "בת";
        msg.innerHTML = "<div class='success-message'>עודכן!</div>";
    }).catch(function(e) {
        msg.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    });
}

function openAdminModal() {
    document.getElementById("adminModal").style.display = "flex";
    document.getElementById("adminPasswordInput").value = "";
    document.getElementById("adminActionsSection").style.display = "none";
    document.getElementById("adminMessage").innerHTML = "";
}
function verifyAdmin() {
    var pass = document.getElementById("adminPasswordInput").value;
    var msg = document.getElementById("adminMessage");
    if (pass === "גיארון7865") {
        document.getElementById("adminActionsSection").style.display = "block";
        msg.innerHTML = "<div class='success-message'>גישה אושרה!</div>";
    } else {
        msg.innerHTML = "<div class='error-message'>סיסמה שגויה</div>";
    }
}

function findUserByUsername(username) {
    return withTimeout(database.ref("users").once("value"), 10000).then(function(snap) {
        var result = null;
        if (snap.exists()) {
            snap.forEach(function(child) {
                var val = child.val();
                if (val.username === username) { result = { uid: child.key, data: val }; }
            });
        }
        return result;
    });
}

function adminTransferCookies() {
    var target = document.getElementById("targetPlayerInput").value.trim();
    var amount = parseInt(document.getElementById("cookieAmountInput").value);
    var msg = document.getElementById("adminMessage");
    if (!target) { msg.innerHTML = "<div class='error-message'>הזן שם שחקן</div>"; return; }
    if (!amount || amount <= 0) { msg.innerHTML = "<div class='error-message'>כמות לא תקינה</div>"; return; }

    msg.innerHTML = "<div class='info-message'>מחפש שחקן...</div>";

    findUserByUsername(target).then(function(found) {
        if (!found) { msg.innerHTML = "<div class='error-message'>שחקן '" + target + "' לא נמצא</div>"; return; }
        var data = found.data;
        data.cookies = (data.cookies||0) + amount;
        data.totalCookiesBaked = (data.totalCookiesBaked||0) + amount;
        return withTimeout(database.ref("users/" + found.uid).set(data), 10000).then(function() {
            return withTimeout(database.ref("chat/" + Date.now() + "_admin").set({
                senderName:"מערכת", message:"👑 יוצר המשחק העביר ל" + target + " " + formatNum(amount) + " עוגיות!",
                timestamp:Date.now(), isSystemMessage:true
            }), 10000);
        }).then(function() {
            return withTimeout(database.ref("notifications/" + found.uid + "/" + Date.now()).set({
                senderName:"יוצר המשחק", amount:amount,
                message:"יוצר המשחק העביר לך " + formatNum(amount) + " עוגיות! 🎁👑",
                timestamp:Date.now(), type:"admin_transfer"
            }), 10000);
        }).then(function() {
            msg.innerHTML = "<div class='success-message'>הועברו " + formatNum(amount) + " עוגיות ל" + target + "!</div>";
            document.getElementById("targetPlayerInput").value = "";
            document.getElementById("cookieAmountInput").value = "";
        });
    }).catch(function(e) {
        console.log(e);
        msg.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    });
}

function openJoinClubModal() { document.getElementById("joinClubModal").style.display = "flex"; }
function openCreateClubModal() { document.getElementById("createClubModal").style.display = "flex"; }
function closeModal(id) { document.getElementById(id).style.display = "none"; }

function generateClubCode() {
    function tryCode(triesLeft) {
        var code = Math.floor(1000 + Math.random()*9000).toString();
        if (triesLeft <= 0) return Promise.resolve(code);
        return withTimeout(database.ref("clubs/" + code).once("value"), 7000).then(function(snap) {
            if (!snap.exists()) return code;
            return tryCode(triesLeft - 1);
        }).catch(function() { return code; });
    }
    return tryCode(50);
}

function joinClub() {
    var code = document.getElementById("clubCodeInput").value.trim();
    var msg = document.getElementById("joinClubMessage");
    if (!/^\d{4}$/.test(code)) { msg.innerHTML = "<div class='error-message'>קוד לא תקין</div>"; return; }
    if (currentClub) { msg.innerHTML = "<div class='error-message'>אתה כבר במועדון</div>"; return; }

    msg.innerHTML = "<div class='info-message'>מחפש מועדון...</div>";
    withTimeout(database.ref("clubs/" + code).once("value"), 10000).then(function(snap) {
        if (!snap.exists()) { msg.innerHTML = "<div class='error-message'>מועדון לא נמצא</div>"; return; }
        var club = snap.val();
        club.members.push({ uid: currentUid, username: currentUsername, role:"חבר רגיל", joinDate:Date.now() });
        clubsCache[code] = club;
        return withTimeout(database.ref("clubs/" + code).set(club), 10000).then(function() {
            return withTimeout(database.ref("users/" + currentUid + "/club").set(code), 10000);
        }).then(function() {
            currentClub = code;
            usersCache[currentUid].club = code;
            msg.innerHTML = "<div class='success-message'>הצטרפת בהצלחה!</div>";
            updateClubButtons();
            setTimeout(function() { closeModal("joinClubModal"); showClubInfo(); }, 800);
        });
    }).catch(function(e) {
        console.log(e);
        msg.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    });
}

function createClub() {
    var name = document.getElementById("clubNameInput").value.trim();
    var msg = document.getElementById("createClubMessage");
    if (!name || name.length < 3) { msg.innerHTML = "<div class='error-message'>שם קצר מדי</div>"; return; }
    if (currentClub) { msg.innerHTML = "<div class='error-message'>אתה כבר במועדון</div>"; return; }

    msg.innerHTML = "<div class='info-message'>יוצר...</div>";
    generateClubCode().then(function(code) {
        var club = { name:name, code:code, founderUid:currentUid, founderName:currentUsername, created:Date.now(),
            members:[{ uid:currentUid, username:currentUsername, role:"מייסד מועדון", joinDate:Date.now() }] };
        clubsCache[code] = club;
        return withTimeout(database.ref("clubs/" + code).set(club), 10000).then(function() {
            return withTimeout(database.ref("users/" + currentUid + "/club").set(code), 10000);
        }).then(function() {
            currentClub = code;
            usersCache[currentUid].club = code;
            msg.innerHTML = "<div class='success-message'>נוצר! קוד: " + code + "</div>";
            updateClubButtons();
            setTimeout(function() { closeModal("createClubModal"); showClubInfo(); }, 1200);
        });
    }).catch(function(e) {
        console.log(e);
        msg.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    });
}

function leaveClub() {
    if (!currentClub) return;
    var club = clubsCache[currentClub];
    var isFounder = club.founderUid === currentUid;
    var p;
    if (isFounder) {
        if (!confirm("אתה המייסד. עזיבה תמחק את המועדון. להמשיך?")) return;
        p = withTimeout(database.ref("clubs/" + currentClub).remove(), 10000).then(function() { delete clubsCache[currentClub]; });
    } else {
        club.members = club.members.filter(function(m) { return m.uid !== currentUid; });
        p = withTimeout(database.ref("clubs/" + currentClub + "/members").set(club.members), 10000);
    }
    p.then(function() {
        return withTimeout(database.ref("users/" + currentUid + "/club").remove(), 10000);
    }).then(function() {
        currentClub = null;
        usersCache[currentUid].club = null;
        updateClubButtons();
        closeModal("clubInfoModal");
    }).catch(function(e) { console.log(e); });
}

function showClubInfo() {
    if (!currentClub) return;
    withTimeout(database.ref("clubs/" + currentClub).once("value"), 10000).then(function(snap) {
        if (!snap.exists()) return;
        var club = snap.val();
        clubsCache[currentClub] = club;
        document.getElementById("clubInfoName").textContent = "🏆 " + club.name;
        document.getElementById("clubInfoCode").textContent = "קוד: " + club.code;
        document.getElementById("clubInfoStats").textContent = "חברים: " + club.members.length;
        var ml = document.getElementById("memberList");
        ml.innerHTML = "";
        club.members.forEach(function(m) {
            var d = document.createElement("div");
            d.className = "member-item";
            d.innerHTML = "<span>" + m.username + "</span><span class='member-role'>" + m.role + "</span>";
            ml.appendChild(d);
        });
        document.getElementById("clubInfoModal").style.display = "flex";
    }).catch(function(e) { console.log(e); });
}

function updateClubButtons() {
    var c = document.getElementById("clubButtonsContainer");
    if (currentClub && clubsCache[currentClub]) {
        c.innerHTML = "<button class='club-button' onclick='showClubInfo()'>🏆 " + clubsCache[currentClub].name + "</button>";
    } else {
        c.innerHTML = "<button class='club-button' onclick='openJoinClubModal()'>🏆 הצטרף למועדון</button><button class='club-button' onclick='openCreateClubModal()'>⭐ צור מועדון</button>";
    }
}

function toggleChat() {
    var c = document.getElementById("chatContainer");
    var icon = document.getElementById("chatToggleIcon");
    c.classList.toggle("minimized");
    icon.textContent = c.classList.contains("minimized") ? "+" : "−";
}
function sendMessage() {
    var input = document.getElementById("chatInput");
    var text = input.value.trim();
    if (!text || !currentUid) return;
    var m = { senderUid: currentUid, senderName: currentUsername, message: text, timestamp: Date.now() };
    input.value = "";
    withTimeout(database.ref("chat/" + Date.now()).set(m), 10000).catch(function(e) { console.log(e); });
}
function renderChat() {
    var box = document.getElementById("chatMessages");
    box.innerHTML = "";
    chatMessages.slice(-20).forEach(function(m) {
        var d = document.createElement("div");
        if (m.isSystemMessage) {
            d.className = "chat-message sys";
            d.textContent = m.message;
        } else {
            d.className = "chat-message " + (m.senderUid === currentUid ? "own" : "other");
            var senderHtml = m.senderUid !== currentUid ? "<div class='message-sender'>" + m.senderName + "</div>" : "";
            d.innerHTML = senderHtml + "<div>" + m.message + "</div>";
        }
        box.appendChild(d);
    });
    box.scrollTop = box.scrollHeight;
}
function setupChatListener() {
    if (!firebaseInit) return;
    try {
        database.ref("chat").limitToLast(50).on("value", function(snap) {
            if (snap.exists()) { chatMessages = Object.values(snap.val()); renderChat(); }
        });
    } catch (e) { console.log(e); }
}
document.getElementById("chatInput").addEventListener("keypress", function(e) { if (e.key === "Enter") sendMessage(); });

function openDonateModal() {
    if (!currentUid) { alert("התחבר קודם"); return; }
    loadPlayerList();
    document.getElementById("donateModal").style.display = "flex";
}
function loadPlayerList() {
    var list = document.getElementById("playerList");
    list.innerHTML = "<div class='info-message'>טוען...</div>";
    withTimeout(database.ref("users").once("value"), 10000).then(function(snap) {
        list.innerHTML = "";
        if (!snap.exists()) return;
        snap.forEach(function(child) {
            if (child.key === currentUid) return;
            var uname = child.val().username;
            var uid = child.key;
            var d = document.createElement("div");
            d.className = "player-item";
            d.innerHTML = "<div class='player-name'>👤 " + uname + "</div>";
            d.addEventListener("click", function() { selectPlayer(uid, uname); });
            list.appendChild(d);
        });
    }).catch(function(e) {
        list.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    });
}
function selectPlayer(uid, uname) {
    selectedDonationTarget = { uid: uid, username: uname };
    document.getElementById("donationTargetPlayer").textContent = uname;
    document.getElementById("currentCookiesDisplay").textContent = formatNum(cookies);
    document.getElementById("donationAmountInput").value = "";
    closeModal("donateModal");
    document.getElementById("donationAmountModal").style.display = "flex";
}
function confirmDonation() {
    var amount = parseInt(document.getElementById("donationAmountInput").value);
    var msg = document.getElementById("donationMessage");
    var btn = document.getElementById("confirmDonationBtn");
    if (!amount || amount <= 0) { msg.innerHTML = "<div class='error-message'>כמות לא תקינה</div>"; return; }
    if (amount > cookies) { msg.innerHTML = "<div class='error-message'>אין לך מספיק עוגיות</div>"; return; }

    btn.disabled = true;
    msg.innerHTML = "<div class='info-message'>מעביר...</div>";

    withTimeout(database.ref("users/" + selectedDonationTarget.uid).once("value"), 10000).then(function(snap) {
        if (!snap.exists()) throw new Error("no-target");
        var data = snap.val();
        cookies -= amount; totalCookiesBaked += amount;
        data.cookies = (data.cookies||0) + amount;
        data.totalCookiesBaked = (data.totalCookiesBaked||0) + amount;
        return withTimeout(database.ref("users/" + selectedDonationTarget.uid).set(data), 10000);
    }).then(function() {
        return saveUserData();
    }).then(function() {
        return withTimeout(database.ref("chat/" + Date.now() + "_don").set({
            senderName:"מערכת", message:"🎁 " + currentUsername + " תרם ל" + selectedDonationTarget.username + " " + formatNum(amount) + " עוגיות!",
            timestamp:Date.now(), isSystemMessage:true
        }), 10000);
    }).then(function() {
        updateDisplay();
        msg.innerHTML = "<div class='success-message'>נתרם בהצלחה!</div>";
        setTimeout(function() { closeModal("donationAmountModal"); }, 1200);
    }).catch(function(e) {
        console.log(e);
        msg.innerHTML = "<div class='error-message'>שגיאה: " + (e.message||e.code) + "</div>";
    }).finally(function() { btn.disabled = false; });
}

var cookieEl = document.getElementById("cookie");
cookieEl.addEventListener("click", function(e) {
    var val = clickPower * prestigeMultiplier;
    cookies += val; totalCookiesBaked += val; totalClicks++;
    updateDisplay();
    showClickEffect(e.clientX, e.clientY);
    makeParticles(e.clientX, e.clientY, "#FFD700");
    checkAchievements();
});

var loopsStarted = false;
function startGameLoops() {
    if (loopsStarted) return;
    loopsStarted = true;
    makeFloatingCookies();
    setupChatListener();
    renderChat();
    updateClubButtons();

    setInterval(function() {
        var gain = cookiesPerSecond / 10;
        cookies += gain; totalCookiesBaked += gain;
        updateDisplay(); checkAchievements();
    }, 100);
    setInterval(function() { makeShop(); }, 1000);
    setInterval(function() { if (Math.random() < 0.3 && cookiesPerSecond > 0) spawnGoldenCookie(); }, 10000);
    setInterval(function() { saveUserData(); }, 30000);
}

updateDisplay();
makeShop();

if (firebaseInit) {
    auth.onAuthStateChanged(function(user) {
        if (user && !currentUid) {
            withTimeout(database.ref("users/" + user.uid).once("value"), 10000).then(function(snap) {
                if (snap.exists()) finishLogin(user.uid, snap.val());
            }).catch(function(e) { console.log("Auto-login failed:", e); });
        }
    });
}
</script>
</body>
</html>S
