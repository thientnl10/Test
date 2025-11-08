<!doctype html>

<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mobile RPG UI - Inventory (Dropdown)</title>
  <style>
:root{--gap:10px;--nav-height:64px;--accent:#6b5ce7}
    *{box-sizing:border-box;font-family:Inter,system-ui,Arial}
    html,body{height:100%;margin:0;background:#0f172a;color:#e6eef8}
    .app{min-height:100vh;display:flex;flex-direction:column}
    .header{padding:12px 14px;background:linear-gradient(180deg,#081028 0%,#071224 100%);display:flex;align-items:center;justify-content:space-between}
    .header h1{font-size:16px;margin:0}
    .view{flex:1;overflow:auto;padding:12px}
    .row{display:flex;gap:var(--gap)}
    .left{flex:1;min-width:220px}
    .right{width:40%;min-width:200px}
    .char-wrap{position:relative;background:linear-gradient(180deg,#0b1220,#071025);padding:8px;border-radius:12px}
    canvas#charCanvas{width:100%;height:320px;border-radius:8px;background:#08121b;display:block}
    .equip-slot{position:absolute;width:48px;height:48px;border-radius:6px;background:rgba(255,255,255,0.03);display:flex;align-items:center;justify-content:center;border:1px dashed rgba(255,255,255,0.04);font-size:20px;cursor:pointer}
    .equip-slot {opacity: 0.6;border: 1px dashed rgba(255,255,255,0.15);transition: all 0.2s ease;}
    .equip-slot.equipped {opacity: 1;border: 0.5px solid gold;box-shadow: 0 0 6px gold;}
    .panel{background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);padding:10px;border-radius:10px;margin-bottom:10px}
    .panel h3{margin:0 0 8px 0;font-size:13px}
    .list-item{padding:8px;border-radius:8px;background:rgba(0,0,0,0.15);margin-bottom:6px;font-size:13px}
    .nav{position:fixed;left:0;right:0;bottom:0;height:var(--nav-height);display:flex;background:linear-gradient(180deg,#061021,#04101a);border-top:1px solid rgba(255,255,255,0.03);gap:6px;padding:8px;  z-index: 10}
    .nav button{flex:1;border-radius:10px;border:0;padding:10px;font-size:14px;cursor:pointer}/* inventory grid: responsive square cells */
    .training-panel, 
	.modal {position: fixed;top: 0;left: 0;right: 0;bottom: 0;display: none;background: rgba(0,0,0,0.6);z-index: 100;}
	.inv-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(120px,1fr));gap:8px;align-items:start}
	.inv-slot{background:linear-gradient(180deg,#071024,#061220);border-radius:8px;min-height:0;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:10px;cursor:pointer;aspect-ratio:1/1;overflow:hidden;text-align:center}
	.inv-slot .icon{font-size:28px;line-height:1}
	.inv-slot .name{font-size:13px;margin-top:8px;line-height:1.1;color:inherit;overflow:hidden;text-overflow:ellipsis;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical}
	.inv-slot .qty{font-size:12px;color:rgba(255,255,255,0.75);margin-top:6px}
 	.header-toggle  {display: inline-block;border: 1px solid rgba(255,255,255,0.2);border-radius: 6px;padding: 4px 10px;background: rgba(0,0,0,0.25);margin-bottom: 10px;}
 	.header-toggle  h3 {margin: 0;font-size: 14px;font-weight: normal;}

/* dropdown filter */
	.filter-container{position:relative;display:inline-block}
	.filter-btn{background:rgba(255,255,255,0.03);border:0;color:inherit;padding:8px 10px;border-radius:8px;cursor:pointer}
	.filter-options{display:none;position:absolute;top:110%;left:0;background:linear-gradient(180deg,#071025,#061224);border:1px solid rgba(255,255,255,0.03);border-radius:8px;padding:6px;z-index:30;min-width:160px}
	.filter-options button{display:block;width:100%;background:none;border:0;color:inherit;text-align:left;padding:8px;border-radius:6px;cursor:pointer}
	.filter-options button:hover{background:rgba(255,255,255,0.03)}
	
	.tag{font-size:14px;padding:4px 6px;border-radius:6px;background:rgba(255,255,255,0.03);display:inline-block}
	.modal{position:fixed;left:0;top:0;right:0;bottom:0;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,0.6);padding:12px}
 	.modal .card {background:#071025;padding:14px;border-radius:10px;max-width:420px;width:100%;max-height:80vh;overflow-y:auto;}

	.skills-list{display:flex;flex-direction:column;gap:8px;}
	.skill{padding:10px;border-radius:8px;background:linear-gradient(180deg,#061226,#04101a);display:flex;justify-content:space-between;align-items:center}
 	.skill-slots {display: grid;grid-template-columns: repeat(2, 60px);grid-template-rows: repeat(5, 60px);gap: 8px;}
	.skill-slot {width: 60px;height: 60px;border-radius: 6px;border: 1px dashed rgba(255,255,255,0.2);display: flex;align-items: center;justify-content: center;font-size: 12px;text-align: center;cursor: pointer;}
	.skill-slot.active {border: 1px solid gold;box-shadow: 0 0 6px gold;}
	.skills-container {display:flex;gap:12px;max-height:400px; /* để kéo */overflow-y:auto; /* scroll được */}
	.equip-slots {display:grid;grid-template-columns: repeat(2, 60px);grid-template-rows: repeat(5, 60px);gap:8px;}
	.slot {width:60px; height:60px;border:1px solid rgba(255,255,255,0.2);border-radius:8px;display:flex; align-items:center; justify-content:center;font-size:12px;opacity:0.3;}
	.slot.active {opacity:1;}
	.learned-skills {flex:1;display:flex;flex-direction:column;gap:8px;}
	.learned-skills .skill-item {padding:8px;border-radius:6px;background:rgba(0,0,0,0.2);cursor:pointer;}
	#view-skills .panel {max-height: 400px;overflow-y: auto;}
 	#skillsList {max-height: 400px;overflow-y: auto;}
	#invGrid {max-height: calc(100vh - 180px);overflow-y: auto;scrollbar-width: thin;scrollbar-color: #6b5ce7 #08121b;}
	#invGrid::-webkit-scrollbar {width: 6px;}
	#invGrid::-webkit-scrollbar-thumb {background: #6b5ce7;border-radius: 6px;}

	#worldMapCanvas {position: fixed;top: 0;left: 0;width: 100vw;height: 100vh;display: block;background: black;z-index: 1;}
	#mapControls {position: fixed;top: 12px;left: 12px;z-index: 9999;display: flex;gap: 8px;flex-wrap: wrap;}
	#mapControls button {background: rgba(255, 255, 255, 0.08);border: 1px solid rgba(255, 255, 255, 0.2);color: #fff;padding: 6px 12px;border-radius: 8px;cursor: pointer;font-size: 13px;backdrop-filter: blur(4px);transition: all 0.2s ease;}
	#mapControls button:hover {background: rgba(107, 92, 231, 0.4); /* tím accent giống UI */border-color: #6b5ce7;box-shadow: 0 0 6px #6b5ce7;}
	
#dungeonExitBtn button{
  min-width:130px;
  padding:8px 12px;
  border-radius:10px;
  border:0;
  cursor:pointer;
  background: rgba(255,255,255,0.06);
}
#dungeonExitBtn button:hover{
  background: rgba(255,255,255,0.12);
}

	.map-wrap{background:linear-gradient(180deg,#061122,#04111a);padding:10px;border-radius:10px}
	.world-map{width:100%;height:320px;border-radius:8px;background-image:radial-gradient(circle at 30% 20%, rgba(255,255,255,0.02), transparent 10%), linear-gradient(180deg,#091524,#04111a)}
	.stat-btn{background:transparent;border:0;color:inherit;font:inherit;cursor:pointer;padding:4px 6px;border-radius:6px}
	.stat-row{display:flex;flex-wrap:wrap;gap:8px;align-items:center}
	.panel:last-of-type{margin-bottom: calc(var(--nav-height) + 0px)}
	@media (max-width:720px){.row{flex-direction:column}.right{width:auto}canvas#charCanvas{height:260px}.inv-grid{grid-template-columns:repeat(auto-fill,minmax(100px,1fr))}}
  @keyframes fadeOut {
  from { opacity: 1; transform: scale(1) translate(-50%, -50%); }
  to { opacity: 0; transform: scale(2) translate(-50%, -50%); }
}

/* progress bars */ .bar-wrap { margin:6px 0; text-align:left; } .bar-label { font-size:12px; margin-bottom:2px; display:block; } .bar { width:100%; height:16px; border-radius:6px; background:rgba(255,255,255,0.08); overflow:hidden; position:relative; } .bar-fill { height:100%; width:50%; background:#6b5ce7; transition:width .3s ease; } .bar-text { position:absolute; top:0;left:0;right:0;bottom:0; display:flex;align-items:center;justify-content:center; font-size:11px; color:#fff; } .bar.hp .bar-fill {background:#e74c3c;} .bar.mp .bar-fill {background:#3498db;} .bar.stamina .bar-fill {background:#27ae60;}

  </style>
</head>
<body>
  <div class="app">
    <div class="header"><h1>RPG Mobile UI (Demo)</h1><div id="headerInfo">Level: <span id="playerLevel">1</span> • XP: <span id="playerXP">0</span>/<span id="xpToNext">100</span>
         <button id="btnReset" style="margin-left:6px;padding:4px 8px;font-size:12px;border-radius:6px;cursor:pointer;">Reset</button>
     <button id="btnAdd10Lv" style="margin-left:6px;padding:4px 8px;font-size:12px;border-radius:6px;cursor:pointer;">+10 Lv</button>
      <button id="btnMinus10" style="margin-left:6px;padding:4px 8px;font-size:12px;border-radius:6px;cursor:pointer;">-10%</button>
    </div></div>
    <main class="view" id="mainView">
      <section id="view-start" class="view-page">
        <div class="row">
          <div class="left">
            <div class="char-wrap">
              <canvas id="charCanvas" width="400" height="320"></canvas>
              <!-- Trang bị -->
              <div class="equip-slot" style="left:145px;top:0px" data-slot="head">👑</div>
              <div class="equip-slot" style="left:145px;top:110px" data-slot="body">🛡️</div>
              <div class="equip-slot" style="left:90px;top:70px" data-slot="hand">🖐️</div>
              <div class="equip-slot" style="left:145px;top:190px" data-slot="leg">🦵</div>
              <div class="equip-slot" style="left:200px;top:70px" data-slot="main-weapon">⚔️</div>
              <div class="equip-slot" style="left:250px;top:70px" data-slot="sub-weapon">🗡️</div>
              <div class="equip-slot" style="left:145px;top:50px" data-slot="necklace">📿</div>
              <div class="equip-slot" style="left:90px;top:120px" data-slot="bracelet1">📿</div>
              <div class="equip-slot" style="left:200px;top:120px" data-slot="bracelet2">📿</div>
              <div class="equip-slot" style="left:20px;top:0px" data-slot="ring1">💍</div>
              <div class="equip-slot" style="left:20px;top:50px" data-slot="ring2">💍</div>
              <div class="equip-slot" style="left:20px;top:100px" data-slot="ring3">💍</div>
              <div class="equip-slot" style="left:20px;top:150px" data-slot="ring4">💍</div>
              <div class="equip-slot" style="left:20px;top:200px" data-slot="ring5">💍</div>
              <!-- Khu hiển thị HP/MP/Thể lực -->
              <div id="char-stats" style="margin-top:10px;padding:8px;background:rgba(0,0,0,0.25);border-radius:8px;font-size:13px">
                <div class="bar-wrap">
                  <span class="bar-label">HP</span>
                  <div class="bar hp"><div class="bar-fill" id="hpBar"></div><div class="bar-text" id="hpText"></div></div>
                </div>
                <div class="bar-wrap">
                  <span class="bar-label">MP</span>
                  <div class="bar mp"><div class="bar-fill" id="mpBar"></div><div class="bar-text" id="mpText"></div></div>
                </div>
                <div class="bar-wrap">
                  <span class="bar-label">Sức Bền</span>
                  <div class="bar stamina"><div class="bar-fill" id="staminaBar"></div><div class="bar-text" id="staminaText"></div></div>
                </div>
              </div>
            </div>
          </div>
          <div class="right">
            <div class="panel" id="panel-status"><h3>Trạng thái</h3><div class="list-item" id="statusText">Bình thường</div></div>
            <div class="panel" id="panel-buffs"><h3>Buff / Debuff</h3><div class="list-item">Không có</div></div>
            <div class="panel" id="panel-title"><h3>Danh hiệu</h3><div class="list-item">Tân thủ</div></div>
            <div class="panel" id="panel-class"><h3>Đẳng cấp</h3><div class="list-item">Võ giả</div></div>
            <div class="panel" id="panel-stats"><h3>Chỉ số</h3><div class="list-item" id="statsList">Đang tải...</div></div>
          </div>
        </div>
      </section>
  <section id="view-inv" class="view-page" style="display:none">
    <div class="panel" style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
      <div>Số lượng: <strong id="invCount">6</strong>/<strong id="invMax">50</strong></div>
      <div class="filter-container">
        <button class="filter-btn" id="filterToggle">Phân loại: <span id="invFilter">Tất cả</span></button>
        <div class="filter-options" id="filterOptions">
          <button data-filter="all">Tất cả</button>
          <button data-filter="technique">Công pháp</button>
          <button data-filter="equip">Trang bị</button>
          <button data-filter="potion">Đan dược</button>
          <button data-filter="other">Khác</button>
        </div>
      </div>
    </div><div class="inv-grid" id="invGrid"></div>
  </section>
<section id="view-skills" class="view-page" style="display:none">
  <div class="panel" style="margin-bottom:12px">
    <div class="header-toggle">
      <h3 id="skillsHeader">Công pháp đã học</h3>
    </div>
    <div class="skills-list" id="skillsList"></div>
    <div class="skills-container" id="skillsContainer" style="display:none">
      <div class="skill-slots" id="skillSlots">
 	 <div class="skill-slot"></div>
	  <div class="skill-slot"></div>
	  <div class="skill-slot"></div>
	  <div class="skill-slot"></div>
	  <div class="skill-slot"></div>
  	<div class="skill-slot"></div>
	  <div class="skill-slot"></div>
 	 <div class="skill-slot"></div>
 	 <div class="skill-slot"></div>
	  <div class="skill-slot"></div>
</div>
      <div class="learned-skills" id="learnedSkillsList"></div>
    </div>
   </div>
  </section>
<section id="view-map" class="view-page" style="display:none">
  <div class="map-wrap" style="text-align:center">
    <canvas id="worldMapCanvas"></canvas>

    <!-- Dungeon canvas -->
    <canvas id="dungeonCanvas" style="display:none;position:fixed;top:0;left:0;width:100vw;height:100vh;z-index:100;background:black"></canvas>

    <!-- Container điều khiển map -->
    <div id="mapControls"></div>
  <div id="dungeonExitBtn" style="margin-top:10px"></div>  
</section>
</main>
<nav class="nav">
  <button id="btnHome">🏠 Khởi đầu</button>
  <button id="btnInv">🎒 Túi đồ</button>
  <button id="btnSkills">📖 Công pháp</button>
  <button id="btnMap">🗺️ Bản đồ</button>
</nav>
<div class="modal" id="modal">
  <div class="card" id="modalCard"></div>
</div>
  </div>
  <script>
    // --- Dữ liệu mặc định ---
    const playerDefault = {
  level:1, xp:0, xpToNext:100,
  stats:{ HP:100, MaxHP:100, Mana:50, MaxMana:50, SucBen:20, MaxSucBen:20,
    TheChat:10, TriLuc:9, NhanhNhen:7, MayMan:10, Khang:10, CanCot:10, NgoTinh:10 },
  inventory:[ {id:'1001', qty:3}, {id:'1004', qty:1}, {id:'1003', qty:1}, {id:'1002', qty:1}, {id:'1005', qty:1}, {id:'2002', qty:1}, {id:'2001', qty:1}, {id:'1006', qty:1} ],
  invMax: 50,
  techniques:[], // công pháp đã học
  skills:[],     // kỹ năng đã mở
  equipment:{
    head:null, body:null, hand:null, leg:null,
    necklace:null, bracelet1:null, bracelet2:null,
    ring1:null, ring2:null, ring3:null, ring4:null, ring5:null,
    "main-weapon":null,"sub-weapon":null
  }
};


    const gameData = {
      "items": [
        {"id":"1001","type":"potion","name":"Đan Dược Hồi Phục","rarity":"Phổ","effect":{"HP":50},"description":"Khôi phục 50 HP ngay lập tức."},
        {"id":"1002","type":"weapon","name":"Kiếm Huyết Ảnh","slot":"main-weapon","rarity":"Huyền","stats":{"attack":15,"critRate":0.1},"durability":200,"effects":[{"type":"lifesteal","value":0.05},{"type":"statusBoost","status":"bloodlust","value":0.1}],"description":"Thanh kiếm đỏ như máu, tăng sức mạnh khi HP thấp."},
        {"id":"1003","type":"armor","name":"Giáp Hàn Băng","slot":"body","rarity":"Địa","stats":{"defense":20,"resistance":{"ice":0.2}},"durability":300,"effects":[{"type":"statusResistance","status":"freeze","value":0.5}],"description":"Giáp băng giá, giảm sát thương băng và kháng đóng băng."},
        {"id":"1004","type":"potion","name":"Đan Dược Hồi Phục Trung Cấp","rarity":"Phổ","effect":{"HP":200},"description":"Khôi phục 200 HP ngay lập tức."},
        {"id":"1005","type":"ring","name":"Nhẫn Vận Mệnh","slot":"ring","rarity":"Thiên","stats":{"MayMan":10},"durability":100,"effects":[{"type":"critBoost","value":0.15},{"type":"dropRateBoost","value":0.2}],"description":"Nhẫn cổ truyền, làm gia tăng may mắn và cơ hội nhặt đồ hiếm."},
        {"id":"1006","type":"ring","name":"Nhẫn Không Gian","slot":"ring","effects":[{"type":"invBonus","value":10}]},
       ],
  // === Công pháp (Techniques) ===
  "techniques": [
    {
      id: "2001",
      type: "techniques",
      name: "Bát Chuyển Huyền Công",      // Tên công pháp
      summary: "Cường hóa thân thể, mở huyệt đạo", // Tổng cương
      rarity: "Địa",                      // Đẳng cấp: Hoàng/Huyền/Địa/Thiên/Thần/Nguyên Sơ
      rules: [                            // Quy tắc lĩnh ngộ
        { level: 3, effect: "Ngộ Quy Tắc Lực – +10% sát thương vật lý" },
        { level: 6, effect: "Ngộ Quy Tắc Kháng – giảm 10% sát thương nhận vào" }
      ],
      hiddenLine: [                       // Dòng ẩn (kích hoạt khi đủ điều kiện)
        { condition: "Căn Cốt ≥ 25 và đã học Cuồng Quyền", effect: "Thể Huyết Chi Lực – HP +20%" }
      ],
      description: "Công pháp rèn luyện huyết nhục, gia tăng thể chất.", // Mô tả
      levelDetails:[
  {level:1, bonus:{MaxHP:50,MaxSucBen:5}, expRequired: 100, description:"Tăng cường thể chất ban đầu"},
  {level:2, bonus:{MaxHP:100,MaxSucBen:10}, expRequired: 150, description:"Gia tăng sinh lực đáng kể"},
  {level:3, bonus:{MaxHP:150,MaxSucBen:15}, expRequired: 200, description:"Thân thể đạt độ bền vững cao"}
],
      skillsUnlocked: ["3001"],           // Kỹ năng nhận được khi học công pháp
      skillLevelUpRules: [              // 🌟 Thêm phần này
      { skillId: "3001", techLv: 1, skillLv: 1 },
      { skillId: "3001", techLv: 3, skillLv: 2 },
      { skillId: "3001", techLv: 6, skillLv: 3 }
    ]
    },
    {
      id: "2002",
      type: "techniques",
      name: "Cửu Chuyển Linh Khí",
      summary: "Dẫn dắt linh khí thiên địa", 
      rarity: "Thiên",
      rules: [
        { level: 5, effect: "Ngộ Quy Tắc Linh Khí – Mana phục hồi gấp đôi" }
      ],
      hiddenLine: [
        { condition: "Ngộ tính ≥ 30", effect: "Linh Hồn Chi Hỏa – kỹ năng hệ hỏa +20%" }
      ],
      description: "Tăng cường nội lực, đại幅提升 Mana.",
      levelDetails: [
        { level: 1, bonus: { MaxMana: 50, TriLuc: 5 }, expRequired: 150 },
        { level: 2, bonus: { MaxMana: 120, TriLuc: 10 }, expRequired: 300 }
      ],
      skillsUnlocked: ["3002"],
      skillLevelUpRules: [
      { skillId: "3002", techLv: 1, skillLv: 1 },
      { skillId: "3002", techLv: 2, skillLv: 2 },
      { skillId: "3002", techLv: 5, skillLv: 3 }
    ]
    }
  ],

  // === Kỹ năng (Skills) ===
  "skills": [
    {
      id: "3001",
      name: "Cuồng Quyền",              // Tên kỹ năng
      level: 1,                          // Lv kỹ năng
      description: "Ra đòn liên hoàn cực mạnh.", // Giới thiệu
      cooldown: 5,                       // Cooldown
      manaCost: 10,                      // Mana Cost
      damage: { physical: 20 }           // Sát thương
    },
    {
      id: "3002",
      name: "Linh Khí Bạo Tạc",
      level: 1,
      description: "Giải phóng linh khí trong cơ thể gây sát thương diện rộng.",
      cooldown: 8,
      manaCost: 20,
      damage: { magic: 40 }
    }
  ]
};

  

    // helper DOM
    
function $(id){return document.getElementById(id);}

    // --- Lưu / tải ---
  function findSkillName(id){ const it = gameData.items.find(x=>String(x.id)===String(id)); return it ? it.name : ('Kỹ năng ' + id); }

    function saveGame(){
  try {
    // đảm bảo luôn là mảng 10 phần tử
    if (!Array.isArray(equippedSkills) || equippedSkills.length !== 10) {
      equippedSkills = Array(10).fill(null).map((v, i) => equippedSkills[i] || null);
    }

    const saveData = {
      player: player,
      equippedSkills: equippedSkills
    };

    localStorage.setItem('rpgSave', JSON.stringify(saveData));
  } catch(e){
    console.error('save failed', e);
  }
}
function loadGame(){
  const data = localStorage.getItem('rpgSave');
  if(data){
    try {
      const parsed = JSON.parse(data) || null;

      // lấy player
      player = parsed.player || null;
      if(!player || typeof player !== 'object') throw new Error('Invalid save data');

      if(!player.stats) player.stats = JSON.parse(JSON.stringify(playerDefault.stats));
      if(!player.inventory) player.inventory = JSON.parse(JSON.stringify(playerDefault.inventory));
      if(!player.techniques) player.techniques = [];
      if(!player.skills) player.skills = [];

      // chuẩn hóa công pháp
      player.techniques = (player.techniques||[]).map(t=>{
        const base = gameData.techniques.find(x=>x.id===t.id) || {};
        return { 
          id: t.id, 
          name: t.name || base.name || ('Công pháp ' + t.id), 
          lvl: t.lvl || 1, 
          exp: t.exp || 0 
        };
      });

      // chuẩn hóa kỹ năng
      player.skills = (player.skills||[]).map(s=>{
        const base = gameData.skills.find(x=>x.id===s.id) || {};
        return { 
          id: s.id, 
          name: s.name || base.name || ('Kỹ năng ' + s.id), 
          lvl: s.lvl || base.level || 1, 
          exp: s.exp || 0 
        };
      });

      // lấy equippedSkills (nếu có), mặc định 10 slot
      equippedSkills = parsed.equippedSkills || Array(10).fill(null);

    } catch(e){
      console.error('Load error', e);
      player = JSON.parse(JSON.stringify(playerDefault));
      equippedSkills = Array(10).fill(null);
      saveGame();
    }
  } else {
    player = JSON.parse(JSON.stringify(playerDefault));
    equippedSkills = Array(10).fill(null);
    saveGame();
  }
}

  function add10Levels(){
  if(!player) return;
  for(let i=0;i<10;i++){
    player.level += 1;
    const delta = getLevelUpDeltaForLevel(player.level);
    // Cộng chỉ số theo quy tắc
    Object.keys(player.stats).forEach(k=>{
      player.stats[k] = (player.stats[k] || 0) + delta;
    });
    // tăng XP yêu cầu cho cấp tiếp theo
    player.xpToNext = Math.floor(player.xpToNext * 1.25);
  }
  saveGame();
  updateHeader();
  renderStats();
    updateBars();
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `<h3>Tăng cấp nhanh</h3>
    <p>Nhân vật đã tăng thêm 10 cấp! Hiện tại: Lv ${player.level}</p>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button onclick="closeModal()">Đóng</button>
    </div>`;
}

    // --- Công thức (placeholder) ---
function calcFinalStat(base, key) {
  let val = Math.floor(Number(base) || 0);

  // Cộng từ trang bị
  if (player && player.equipment) {
    Object.values(player.equipment).forEach(eid => {
      const item = findItemById(eid);
      if (item && item.stats && key in item.stats) {
        val += Number(item.stats[key]) || 0;
      }
    });
  }

  // Cộng từ công pháp (techniques)
  (player.techniques || []).forEach(t => {
    const tech = gameData.techniques.find(x => x.id === t.id);
    if (tech && tech.levelDetails) {
      const current = tech.levelDetails.find(ld => ld.level === t.lvl);
      if (current && current.bonus && key in current.bonus) {
        val += current.bonus[key];
      }
    }
  });

  return val;
}

function getLevelUpDeltaForLevel(level) {
  // ❖ Giữ lại ý tưởng cũ: mỗi 10 cấp thưởng thêm
  // ❖ Nhưng thêm ảnh hưởng của CanCot và NgoTinh

  const base = (level % 10 === 0) ? 5 : 1;

  const canCot = calcFinalStat(player.stats.CanCot, "CanCot");
  const ngoTinh = calcFinalStat(player.stats.NgoTinh, "NgoTinh");

  // CanCốt cao → cần ít exp hơn, nên thưởng ít hơn
  // Ngộ tính cao → thưởng thêm exp để đồng bộ hóa tốc độ tiến bộ
  const factor = 1 + (ngoTinh * 0.01) - (canCot * 0.005);

  return Math.max(1, Math.round(base * factor));
}


    // --- Inventory filter state ---
    let currentInvFilter = 'all'; // all, skill, equip, potion, other
// Biến dùng cho animation nhân vật
let animFrame = 0;
let blinkToggle = false;

  
function mapItemToCategory(item){
  if(!item || !item.type) return 'other';
  const t = item.type;
  if(t === 'techniques') return 'technique'; // Công pháp
  if(t === 'potion') return 'potion';
  if(t === 'weapon' || t === 'armor' || t === 'ring') return 'equip';
  return 'other';
}


    function showView(name){
  const views = ['start','inv','skills','map'];
  views.forEach(v=>{
    const el = document.getElementById('view-'+v);
    if(el) el.style.display = (v === name ? 'block' : 'none');
  });

  // nếu rời map -> xóa luôn controls
  if (name !== 'map') {
    removeDungeonControls();
  } else {
    // vào map: nếu đang trong dungeon thì đảm bảo controls tồn tại
    if (currentMapLevel === 'dungeon') {
      // tránh tạo đôi
      if (!document.getElementById('dungeonControls')) setupDungeonControls();
    } else {
      // đang ở region/world/cosmos trong view map -> không hiển thị controls
      removeDungeonControls();
    }
  }

  updateHeader();
  if(name === 'inv') renderInventory();
  if(name === 'skills') renderSkills();
  renderStats();
  updateBars();
  saveGame();
}
  
  function getClassByLevel(level){
  if(level <= 9) return "Phàm nhân - không năng lực siêu phàm";
  if(level <= 39){
    if(level < 20) return "Khai Linh sơ kỳ";
    if(level < 30) return "Khai Linh trung kỳ";
    return "Khai Linh hậu kỳ";
  }
  if(level <= 69){
    if(level < 50) return "Lột Xác sơ kỳ";
    if(level < 60) return "Lột Xác trung kỳ";
    return "Lột Xác hậu kỳ";
  }
  if(level <= 98){
    if(level < 80) return "Hóa Thần sơ kỳ";
    if(level < 90) return "Hóa Thần trung kỳ";
    return "Hóa Thần hậu kỳ";
  }
  if(level === 99) return "Bán Thần";
  if(level >= 100){
    if(player.specialClass === "nguyenso") return "Nguyên Sơ - Thế Giới chi chủ";
    return "Đăng Thần"; // mặc định giữ nguyên
  }
  return "Phàm nhân";
}


    function updateHeader(){
  if(!player) return;
  $('playerLevel').innerText = player.level;
  $('playerXP').innerText = player.xp;
  $('xpToNext').innerText = player.xpToNext;
  $('statusText').innerText = "Bình thường";

  const hpNow = Number(player.stats.HP) || 0;
  const hpMax = calcFinalStat(player.stats.MaxHP, "MaxHP");
  const hpPct = Math.max(0, Math.min(100, Math.round((hpNow/hpMax)*100)));
  const hpBar = $('hpBar'); if(hpBar) hpBar.style.width = hpPct + "%";
  const hpText = $('hpText'); if(hpText) hpText.innerText = `${hpNow}/${hpMax}`;

  const mpNow = Number(player.stats.Mana) || 0;
  const mpMax = calcFinalStat(player.stats.MaxMana, "MaxMana");
  const mpPct = Math.max(0, Math.min(100, Math.round((mpNow/mpMax)*100)));
  const mpBar = $('mpBar'); if(mpBar) mpBar.style.width = mpPct + "%";
  const mpText = $('mpText'); if(mpText) mpText.innerText = `${mpNow}/${mpMax}`;

  const stNow = Number(player.stats.SucBen) || 0;
  const stMax = calcFinalStat(player.stats.MaxSucBen, "MaxSucBen");
  const stPct = Math.max(0, Math.min(100, Math.round((stNow/stMax)*100)));
  const stBar = $('staminaBar'); if(stBar) stBar.style.width = stPct + "%";
  const stText = $('staminaText'); if(stText) stText.innerText = `${stNow}/${stMax}`;
  
  const classPanel = $('panel-class');
  if(classPanel){
    const el = classPanel.querySelector('.list-item');
    if(el) el.innerText = getClassByLevel(player.level);
  }
}

function updateBars() {
  if(!player) return;

  const hpNow = Number(player.stats.HP) || 0;
  const hpMax = calcFinalStat(player.stats.MaxHP, "MaxHP");
  $('hpBar').style.width = Math.max(0, Math.min(100, Math.round((hpNow/hpMax)*100))) + "%";
  $('hpText').innerText = `${hpNow}/${hpMax}`;

  const mpNow = Number(player.stats.Mana) || 0;
  const mpMax = calcFinalStat(player.stats.MaxMana, "MaxMana");
  $('mpBar').style.width = Math.max(0, Math.min(100, Math.round((mpNow/mpMax)*100))) + "%";
  $('mpText').innerText = `${mpNow}/${mpMax}`;

  const stNow = Number(player.stats.SucBen) || 0;
  const stMax = calcFinalStat(player.stats.MaxSucBen, "MaxSucBen");
  $('staminaBar').style.width = Math.max(0, Math.min(100, Math.round((stNow/stMax)*100))) + "%";
  $('staminaText').innerText = `${stNow}/${stMax}`;
}

let showingTechniques = true;

$('skillsHeader').onclick = function(){
  showingTechniques = !showingTechniques;
  if(showingTechniques){
    $('skillsHeader').innerText = "Công pháp đã học";
    $('skillsList').style.display = "block";
    $('skillsContainer').style.display = "none";
    renderTechniques();
  } else {
    $('skillsHeader').innerText = "Kỹ năng đã học";
    $('skillsList').style.display = "none";
    $('skillsContainer').style.display = "flex";
    renderLearnedSkills();
  }
};

function renderLearnedSkills(){
  const list = $('learnedSkillsList');
  list.innerHTML = '';

  if(player.skills.length === 0){
    list.innerHTML = '<div class="skill-item">Chưa học kỹ năng nào</div>';
    return;
  }

  player.skills.forEach(s=>{
    const el = document.createElement('div'); 
    el.className = 'skill-item';
    el.innerHTML = `<strong>${s.name}</strong> <span style="font-size:12px">Lv ${s.lvl}</span>`;
    el.onclick = ()=>viewSkill(s.id, "learned"); // ✅ chỉ chỗ này cho phép trang bị
    list.appendChild(el);
  });

  renderSkillSlots();
}




let equippedSkills = Array(10).fill(null);

function equipSkill(skillId){
  // kiểm tra kỹ năng đã trang bị chưa
  const alreadyIndex = equippedSkills.findIndex(s => s === skillId);
  if(alreadyIndex !== -1){
    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>Thông báo</h3>
      <p>Kỹ năng này đã được trang bị ở ô ${alreadyIndex+1}.</p>
      <div style="margin-top:8px;display:flex;gap:8px">
        <button onclick="closeModal()">Đóng</button>
      </div>`;
    return;
  }

  // tìm slot trống
  const index = equippedSkills.findIndex(s => s === null);
  if(index === -1){
    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>Thông báo</h3>
      <p>Tất cả ô kỹ năng đã đầy!</p>
      <div style="margin-top:8px;display:flex;gap:8px">
        <button onclick="closeModal()">Đóng</button>
      </div>`;
    return;
  }

  equippedSkills[index] = skillId;
  saveGame();
  closeModal();
  renderSkillSlots();
}

function unequipSkill(index){
  equippedSkills[index] = null;
 saveGame();
  renderSkillSlots();
}

let draggedIndex = null;

function renderSkillSlots(){
  const slots = document.querySelectorAll('#skillSlots .skill-slot');
  slots.forEach((slot, i)=>{
    const skillId = equippedSkills[i];

    // reset ô
    slot.innerHTML = '';
    slot.classList.remove('active');

    if(skillId){
      const skill = player.skills.find(s=>s.id === skillId);
      const data = gameData.skills.find(s=>s.id === skillId);
      if(skill && data){
        slot.classList.add('active');
        slot.innerHTML = `
          <div style="text-align:center">
            <div style="font-weight:bold">${data.name}</div>
            <div style="font-size:11px;opacity:0.8">Lv ${skill.lvl}</div>
          </div>`;
      }
    }

    // click → gỡ kỹ năng khỏi slot
    slot.onclick = ()=>unequipSkill(i);

    // drag & drop
    slot.setAttribute("draggable", "true");

    slot.ondragstart = ()=>{
      draggedIndex = i;
    };

    slot.ondragover = (e)=>{
      e.preventDefault(); // cho phép drop
    };

    slot.ondrop = (e)=>{
      e.preventDefault();
      if(draggedIndex !== null && draggedIndex !== i){
        // hoán đổi skill giữa 2 slot
        [equippedSkills[draggedIndex], equippedSkills[i]] = [equippedSkills[i], equippedSkills[draggedIndex]];
        saveGame();
        renderSkillSlots();
      }
      draggedIndex = null;
    };
  });
}


    // === RENDER TÚI ĐỒ (có lọc) ===
    function renderInventory(){
  const grid = $('invGrid'); if(!grid) return;
  grid.innerHTML = '';
  if(!player || !player.inventory || player.inventory.length===0){ 
    grid.innerHTML = '<div class="small">Túi đồ rỗng</div>'; 
    $('invCount').innerText = 0; 
    return; 
  }

  // TÍNH TỔNG SỐ LƯỢNG TOÀN BỘ
  let totalAll = 0;
  player.inventory.forEach(entry=>{
    totalAll += (entry.qty || 1);
  });
  $('invCount').innerText = totalAll;
$('invMax').innerText = getInventoryLimit();


  // Lọc theo phân loại để hiển thị
  const filtered = player.inventory.filter(entry=>{
    const data = findItemById(entry.id);
    if(!data) return false;
    const cat = mapItemToCategory(data);
    if(currentInvFilter === 'all') return true;
    return cat === currentInvFilter;
  });

  filtered.forEach(entry=>{
    const data = findItemById(entry.id);
    if(!data) return;
    const el = document.createElement('div'); el.className='inv-slot';
    el.innerHTML = `<div class="icon">${getItemIcon(data)}</div><div class="name">${data.name}</div><div class="qty">x${entry.qty||1}</div>`;
    el.onclick = ()=> openItem(entry.id);
    grid.appendChild(el);
  });

  // update visible filter label
  const labelMap = { 
  all: 'Tất cả', 
  technique: 'Công pháp', 
  skill: 'Kỹ năng', 
  equip: 'Trang bị', 
  potion: 'Đan dược', 
  other: 'Khác' 
};
  const filt = $('invFilter'); if(filt) filt.innerText = labelMap[currentInvFilter] || currentInvFilter;

  if(filtered.length === 0){ 
    grid.innerHTML = '<div class="list-item">Không tìm thấy vật phẩm theo phân loại này.</div>'; 
  }
}

function getInventoryLimit(){
  let limit = player.invMax || 50;

  // ví dụ: nhẫn Vận Mệnh tăng 10 ô
  if(player.equipment){
    Object.values(player.equipment).forEach(eid=>{
      const it = findItemById(eid);
      if(it && it.effects){
        it.effects.forEach(e=>{
          if(e.type === "invBonus"){  // 👈 thêm type mới
            limit += e.value || 0;
          }
        });
      }
    });
  }
  return limit;
}
function addItem(id, qty=1){
  const totalAll = player.inventory.reduce((a,b)=>a+(b.qty||1),0);
  const limit = getInventoryLimit();
  if(totalAll + qty > limit){
    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>Thông báo</h3>
      <p>Túi đồ đã đầy (${totalAll}/${limit}). Không thể thêm ${qty} vật phẩm.</p>
      <div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>`;
    return false;
  }

  let entry = player.inventory.find(x=>x.id===id);
  if(entry) entry.qty += qty;
  else player.inventory.push({id:id, qty:qty});

  saveGame();
  renderInventory();
  return true;
}


    function getItemIcon(it){
  switch(it.type){
    case 'techniques': return '📜'; // công pháp
    case 'skill': return '✨';      // kỹ năng
    case 'weapon': return '⚔️';
    case 'potion': return '⚗️';
    case 'armor': return '🛡️';
    case 'ring': return '💍';
    default: return 'ℹ️';
  }
}

    function findItemById(id){
  let it = null;
  if(gameData.items) it = gameData.items.find(x => String(x.id) === String(id));
  if(!it && gameData.techniques) it = gameData.techniques.find(x => String(x.id) === String(id));
  if(!it && gameData.skills) it = gameData.skills.find(x => String(x.id) === String(id));
  return it || null;
}


    // --- Hiển thị chi tiết item ---
function getItemDetailsHTML(data, inv){
  if(!data) return '<div class="small">Dữ liệu vật phẩm không tìm thấy.</div>';

  let html = `<div style="font-size:18px;margin-bottom:6px">${getItemIcon(data)} 
    <strong>${data.name}</strong> <span class="small">${data.rarity||''}</span></div>`;

  // Công pháp
  if(data.type === 'techniques' || data.type === 'technique'){
    html += `<div><strong>Tổng cương:</strong> ${data.summary||''}</div>`;
    html += `<div><strong>Đẳng cấp:</strong> ${data.rarity||''}</div>`;
    if(data.description) html += `<div><strong>Mô tả:</strong> ${data.description}</div>`;

    // Chi tiết tất cả cấp độ
    if(Array.isArray(data.levelDetails) && data.levelDetails.length){
      html += `<div style="margin-top:8px"><strong>Chi tiết:</strong>`;
      data.levelDetails.forEach(ld=>{
        let bonusText = Object.entries(ld.bonus)
          .map(([k,v])=>`+${v} ${translateStatName(k)}`)
          .join(", ");
        html += `<div class="small">Lv ${ld.level}: ${ld.description||''}</div>`;
html += `<div class="small" style="margin-left:12px">${bonusText}</div>`;
      });
      html += `</div>`;
    }

    // Quy tắc lĩnh ngộ
    if(Array.isArray(data.rules) && data.rules.length){
      html += `<div style="margin-top:8px"><strong>Quy tắc lĩnh ngộ:</strong>`;
      data.rules.forEach(r=>{
        html += `<div class="small">Lv ${r.level}: ${r.effect}</div>`;
      });
      html += `</div>`;
    }

    // Dòng ẩn
    if(Array.isArray(data.hiddenLine) && data.hiddenLine.length){
      html += `<div style="margin-top:8px"><strong>Dòng ẩn:</strong>`;
      data.hiddenLine.forEach(r=>{
        html += `<div class="small">• ${r.condition} → ${r.effect}</div>`;
      });
      html += `</div>`;
    }

    // Kỹ năng học được
    if(Array.isArray(data.skillsUnlocked) && data.skillsUnlocked.length && gameData && Array.isArray(gameData.skills)){
      html += `<div style="margin-top:8px"><strong>Kỹ năng học được:</strong>`;
      data.skillsUnlocked.forEach(sid=>{
        const skill = gameData.skills.find(x=>x.id===sid);
        if(skill){
          html += `<div class="small">✨ ${skill.name} (Lv ${skill.level}) - ${skill.description}</div>`;
        }
      });
      html += `</div>`;
    }

    if(inv) html += `<div class="small" style="margin-top:8px">Số lượng: ${inv.qty||1}</div>`;
  }
  // Vật phẩm khác (trang bị, đan dược…)
  else{
    if(data.stats){ 
      for(const k in data.stats){ 
        html += `<div class="small">${k}: ${JSON.stringify(data.stats[k])}</div>`; 
      } 
    }
    if(data.effect){ 
      for(const k in data.effect){ 
        html += `<div class="small">+${data.effect[k]} ${k}</div>`; 
      } 
    }
    if(Array.isArray(data.effects)){ 
      data.effects.forEach(e=> html += `<div class="small">* ${e.type}: ${e.value}</div>`); 
    }
    if(data.description) 
      html += `<div style="margin-top:8px;font-style:italic;color:rgba(255,255,255,0.8)">${data.description}</div>`;
    if(inv) html += `<div class="small" style="margin-top:8px">Số lượng: ${inv.qty||1}</div>`;
  }
  return html;
}

    // === MODAL ITEM ===
    function openItem(id){
  const inv = player.inventory.find(x=>String(x.id)===String(id));
  const data = findItemById(id);
  if(!inv || !data){ console.warn('openItem: not found', id); return; }
  const details = getItemDetailsHTML(data, inv);

  let actionBtns = `<button onclick="closeModal()">Đóng</button>`;
  if(data.type === 'potion'){
    actionBtns = `<button id="useBtn">Sử dụng</button>` + actionBtns;
  }
  // Nếu là trang bị -> thêm nút "Trang bị"
  if(data.type === 'weapon' || data.type === 'armor' || data.type === 'ring' || data.slot === 'necklace'){
    actionBtns = `<button onclick="autoEquipItem('${id}')">Trang bị</button>` + actionBtns;
  }
      if(data.type === 'techniques'){
  actionBtns = `<button onclick="learnTechnique('${id}')">Học</button>` + actionBtns;
}


  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `<div id="itemDetail">${details}</div><div style="display:flex;gap:8px;margin-top:8px">${actionBtns}</div>`;

  const useBtn = $('useBtn');
  if(data.type === 'potion' && useBtn){
    useBtn.onclick = ()=> useItem(id);
  }
}

    function closeModal() {
  // 🚫 Ngăn đóng khi đang tu luyện
  if (trainingInProgress) {
    return;
  }

  const m = $('modal');
  if (m) {
    // 🩵 Thêm hiệu ứng mờ dần khi đóng
    m.style.transition = "opacity 0.2s ease";
    m.style.opacity = "0";
    setTimeout(() => {
      m.style.display = "none";
      m.style.opacity = "";
      m.style.zIndex = ""; // 🔹 reset z-index để tránh ảnh hưởng giao diện khác
    }, 200);
  }

  // Gỡ sự kiện dùng vật phẩm (nếu có)
  const useBtn = $('useBtn');
  if (useBtn) useBtn.onclick = null;
}


    // === SỬ DỤNG VẬT PHẨM ===
    function useItem(itemId){
  const idx = player.inventory.findIndex(x=>String(x.id)===String(itemId));
  if(idx === -1){ alert('Item không tồn tại'); return; }

  const inv  = player.inventory[idx];
  const data = findItemById(itemId);
  if(!data){ alert('Dữ liệu vật phẩm không hợp lệ'); return; }

  if(data.type === 'potion' && data.effect){
    if(data.effect.HP)   player.stats.HP   = Math.min(player.stats.MaxHP   || 99999, (player.stats.HP   || 0) + Number(data.effect.HP));
    if(data.effect.Mana) player.stats.Mana = Math.min(player.stats.MaxMana || 99999, (player.stats.Mana || 0) + Number(data.effect.Mana));

    // hiện bảng kết quả (không dùng alert)
    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>Kết quả</h3>
      <p>Sử dụng ${data.name}: ${data.effect.hp ? `+${data.effect.hp} HP` : ''} ${data.effect.mana ? `+${data.effect.mana} Mana` : ''}</p>
      <div style="display:flex;gap:8px;margin-top:8px">
        <button onclick="afterUseItem('${itemId}')">Đóng</button>
      </div>`;
  }

  // giảm số lượng & lưu
  inv.qty = (inv.qty||1) - 1;
  if(inv.qty <= 0) player.inventory.splice(idx, 1);

  saveGame();
  renderInventory();
  updateHeader();
    updateBars();
  // KHÔNG closeModal ở đây!
}

function afterUseItem(itemId){
  const inv = player.inventory.find(x=>String(x.id)===String(itemId));
  if(inv && inv.qty > 0){
    // còn vật phẩm → quay lại chi tiết
    openItem(itemId);
  } else {
    // hết vật phẩm → về túi đồ
    closeModal();
    renderInventory();
  }
  updateHeader();
  updateBars();
}

  
  function learnTechnique(id) {
  const tech = gameData.techniques.find(t => t.id === id);
  if (!tech) return;

  if (player.techniques.some(t => t.id === id)) {
    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>Thông báo</h3>
      <p>Bạn đã học công pháp này rồi: ${tech.name}</p>
      <div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>`;
    return;
  }

  // thêm công pháp
  player.techniques.push({ id: tech.id, name: tech.name, lvl: 1, exp: 0 });

  // mở kỹ năng từ công pháp
  if (Array.isArray(tech.skillsUnlocked)) {
    tech.skillsUnlocked.forEach(sid=>{
      const sk = gameData.skills.find(x=>x.id===sid);
      if (sk && !player.skills.some(s=>s.id===sid)) {
        player.skills.push({ id: sk.id, name: sk.name, lvl: sk.level || 1, exp: 0 });
      }
    });
  }

  // xóa khỏi túi đồ
  const idx = player.inventory.findIndex(x=>String(x.id)===String(id));
  if(idx !== -1) player.inventory.splice(idx,1);

  saveGame();
  renderInventory();
renderTechniques();
renderSkills();
renderStats();    // ← để cập nhật danh sách chỉ số
updateHeader();   // ← để cập nhật HP/MP bars nếu cần
updateBars();

  // hiện thông báo
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>Đã học công pháp</h3>
    <p>${tech.name} đã được thêm vào danh sách công pháp của bạn.</p>
    <div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>`;
}  
 
function renderTechniques(){
  const list = $('skillsList'); 
  if(!list) return; 
  list.innerHTML = '';
  if(player.techniques.length===0){
    list.innerHTML = '<div class="list-item">Chưa học công pháp nào</div>';
    return;
  }
  player.techniques.forEach(t=>{
    const el = document.createElement('div'); el.className='skill';
    el.innerHTML = `<div><strong>${t.name}</strong> <div style="font-size:12px">Lv ${t.lvl} • Exp ${t.exp}</div></div>
      <div style="display:flex;gap:6px">
        <button onclick="viewTechnique('${t.id}')">Xem</button>
        <button onclick="trainTechnique('${t.id}')">🧘 Tu luyện</button>
      </div>`;
    list.appendChild(el);
  });
}

function renderSkills(){
  const container = $('panel-skills'); // bạn cần thêm 1 panel riêng trong HTML
  if(!container) return;
  container.innerHTML = '';
  if(player.skills.length===0){
    container.innerHTML = '<div class="list-item">Chưa có kỹ năng nào</div>';
    return;
  }
  player.skills.forEach(s=>{
    const el = document.createElement('div'); el.className='list-item';
    el.innerHTML = `✨ ${s.name} (Lv ${s.lvl})`;
    container.appendChild(el);
  });
}


function viewTechnique(id){
  const learned = player.techniques.find(x=>x.id===id);
  const data = gameData.techniques.find(t=>t.id===id);
  if(!learned || !data) return;

  $('modal').style.display = 'flex';

  let html = `<h3>${getItemIcon(data)} ${data.name} (Lv ${learned.lvl})</h3>`;
  if(data.summary) html += `<div><strong>Tổng cương:</strong> ${data.summary}</div>`;
  html += `<div><strong>Đẳng cấp:</strong> ${data.rarity||''}</div>`;
  html += `<div><strong>Lv hiện tại:</strong> ${learned.lvl}</div>`;
  if(data.description) html += `<div><strong>Mô tả:</strong> ${data.description}</div>`;

  // Chi tiết: chỉ các Lv đã đạt
  if(Array.isArray(data.levelDetails)){
    html += `<div style="margin-top:8px"><strong>Chi tiết:</strong>`;
    data.levelDetails.forEach(ld=>{
      if(ld.level <= learned.lvl){
        let bonusText = "";
        if(ld.bonus) bonusText = Object.entries(ld.bonus)
          .map(([k,v])=>`+${v} ${translateStatName(k)}`)
          .join(", ");
        html += `<div class="small">Lv ${ld.level}: ${ld.description||''}</div>`;
        html += `<div class="small" style="margin-left:12px">${bonusText}</div>`;
      }
    });
    html += `</div>`;
  }

  // Kỹ năng học được => clickable -> showSkillDetail
  if(Array.isArray(data.skillsUnlocked) && data.skillsUnlocked.length && Array.isArray(gameData.skills)){
    html += `<div style="margin-top:8px"><strong>Kỹ năng học được:</strong>`;
    data.skillsUnlocked.forEach(sid=>{
      const sk = gameData.skills.find(x=>x.id===sid);
      if(sk){
        html += `<div class="small">✨ <span style="color:#ffd700;cursor:pointer" onclick="showSkillDetail('${sid}')">${sk.name} (Lv ${sk.level||1})</span> - ${sk.description}</div>`;
      }
    });
    html += `</div>`;
  }

  // nút chuyển sang giao diện Kỹ năng đã học (modal chứa danh sách skills + 10 slot)
  html += `<div style="display:flex;gap:8px;margin-top:10px">
        <button onclick="closeModal()">Đóng</button>
  </div>`;
  $('modalCard').innerHTML = html;
}

  // ==== Training globals (thay thế let trainingInterval = null;)
let trainingInterval = null;          // interval tăng exp
let trainingAnimationRaf = null;      // requestAnimationFrame id cho animation canvas
let trainingInProgress = false;       // cờ báo đang tu luyện (chặn click ngoài)
let currentTrainingTechId = null;     // id công pháp đang tu luyện (nếu có)

function trainTechnique(id){
  // ensure player.skillSlots exists
  player.skillSlots = player.skillSlots || Array(10).fill(null);

  const t = player.techniques.find(x=>x.id===id);
  const data = gameData.techniques.find(tt=>tt.id===id);
  if(!t || !data) return;

  // mark current training
  currentTrainingTechId = id;
  trainingInProgress = true;

  // Lưu trạng thái bắt đầu training để dùng cho offline resume
  const trainingState = {
    techId: id,
    start: Date.now(),
    initialExp: t.exp || 0,
    techLvl: t.lvl || 1
  };
  try{ localStorage.setItem('rpg_training', JSON.stringify(trainingState)); }catch(e){}

  // Show modal (nút Dừng to & đặt giữa)
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>Đang tu luyện: ${t.name} (Lv ${t.lvl})</h3>
    <canvas id="trainCanvas" width="300" height="200" style="background:#111;border-radius:8px;margin:8px auto;display:block"></canvas>
    <div class="bar exp" style="margin-top:8px">
      <div class="bar-fill" id="expBar"></div>
      <div class="bar-text" id="expText"></div>
    </div>
    <div style="display:flex;justify-content:center;margin-top:12px">
      <button id="stopTrainingBtn" style="font-size:16px;padding:10px 18px;border-radius:8px;cursor:pointer">Dừng</button>
    </div>
  `;

  // stop handler
  document.getElementById('stopTrainingBtn').onclick = ()=> stopTraining();

  // Canvas animation (bắt đầu ngay)
  const canvas = document.getElementById("trainCanvas");
  if(!canvas) return;
  const ctx = canvas.getContext("2d");
  let frame = 0;
  function drawTrainingChar(f){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.strokeStyle = "#0f0";
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.arc(150,60,20,0,Math.PI*2); ctx.stroke(); // đầu
    ctx.beginPath(); ctx.moveTo(150,80); ctx.lineTo(150,140); ctx.stroke(); // thân
    ctx.beginPath();
    ctx.moveTo(150,100); ctx.lineTo(120 + Math.sin(f*0.2)*20,120);
    ctx.moveTo(150,100); ctx.lineTo(180 - Math.sin(f*0.2)*20,120);
    ctx.stroke(); // tay
    ctx.beginPath();
    ctx.moveTo(150,140); ctx.lineTo(130 + Math.cos(f*0.2)*20,180);
    ctx.moveTo(150,140); ctx.lineTo(170 - Math.cos(f*0.2)*20,180);
    ctx.stroke(); // chân
  }
  function animate(){
    if(!trainingInProgress) return;
    frame++;
    drawTrainingChar(frame);
    trainingAnimationRaf = requestAnimationFrame(animate);
  }
  // start animation immediately
  if(trainingAnimationRaf) cancelAnimationFrame(trainingAnimationRaf);
  animate();

  // helper: update UI from current t
  function updateExpUI(){
    const currentLevelData = (data.levelDetails||[]).find(ld => ld.level === t.lvl) || {};
    const expMax = currentLevelData.expRequired || 100;
    const pct = Math.min(100, Math.round(((t.exp||0)/expMax)*100));
    const bar = $('expBar'); if(bar) bar.style.width = pct + "%";
    const txt = $('expText'); if(txt) txt.innerText = `${t.exp||0}/${expMax}`;
  }

  // if any previous interval running, clear
  if(trainingInterval){ clearInterval(trainingInterval); trainingInterval = null; }

  // Main tick: 1s per exp
  trainingInterval = setInterval(()=>{
    // re-fetch data (in case leveled up earlier)
    const techData = gameData.techniques.find(tt=>tt.id===id);
    if(!techData){ stopTraining(); return; }

    const currentLevelData = (techData.levelDetails||[]).find(ld => ld.level === t.lvl) || {};
    const expMax = currentLevelData.expRequired || 100;

    // +1 exp per second
    t.exp = (t.exp||0) + 1;
    player.xp = (player.xp||0) + 1;

    // check level-up for technique, with cap: technique lvl cannot exceed player.level + 5
    while((t.exp || 0) >= expMax){
      if(t.lvl + 1 > (player.level || 1) + 5){
        // hit cap: don't increase lvl, clamp exp to expMax-1 and stop adding further exp for leveling
        t.exp = Math.min(t.exp, expMax - 1);
        break;
      }
      t.exp -= expMax;
      t.lvl = (t.lvl || 1) + 1;
      // after leveling up technique, recompute expMax for new level
      const nextLevelData = (techData.levelDetails||[]).find(ld => ld.level === t.lvl) || {};
      if(!nextLevelData) break;
      // optionally: show short notice (keeps modal open)
      $('modalCard').querySelector('h3').innerText = `Đang tu luyện: ${t.name} (Lv ${t.lvl})`;
    }

    // Giới hạn cấp nhân vật: max nhân vật không vượt quá highest technique level
    const highestTechLv = Math.max(1, ...(player.techniques||[]).map(x=>x.lvl||1));
    let leveledPlayer = false;
    while((player.xp || 0) >= (player.xpToNext || 100) && (player.level || 1) < highestTechLv){
      player.xp -= player.xpToNext;
      player.level = (player.level || 1) + 1;
      player.xpToNext = Math.floor((player.xpToNext || 100) * 1.25);
      const delta = getLevelUpDeltaForLevel(player.level);
      Object.keys(player.stats).forEach(k=>{
        player.stats[k] = (player.stats[k] || 0) + delta;
      });
      leveledPlayer = true;
    }
    if(player.level >= highestTechLv){
      player.xp = Math.min(player.xp, (player.xpToNext || 100) - 1);
    }

    // save state each tick (also update training record to count elapsed correctly)
    const nowState = {
      techId: id,
      start: Date.now(),
      initialExp: t.exp || 0,
      techLvl: t.lvl || 1
    };
    try{ localStorage.setItem('rpg_training', JSON.stringify(nowState)); }catch(e){}

    updateExpUI();
    renderTechniques();
    renderSkills();
    updateHeader();
    renderStats();
    saveGame();
    updateBars();

    // if player leveled up -> show short notice (modal remains)
    if(leveledPlayer){
      // small non-blocking notice: change header text only
      $('modalCard').querySelector('h3').innerText = `Đang tu luyện: ${t.name} (Lv ${t.lvl}) — Nhân vật Lv ${player.level}`;
    }

  }, 1000);

  // init ui immediately
  updateExpUI();
}

function stopTraining(){
  // clear interval
  if(trainingInterval){
    clearInterval(trainingInterval);
    trainingInterval = null;
  }
  // cancel animation
  if(trainingAnimationRaf){
    cancelAnimationFrame(trainingAnimationRaf);
    trainingAnimationRaf = null;
  }
  // clear flags & storage
  trainingInProgress = false;
  currentTrainingTechId = null;
  try{ localStorage.removeItem('rpg_training'); }catch(e){}
  // close modal
  const m = $('modal'); if(m) m.style.display = 'none';
  // save final state
  saveGame();
}
 
 // Hiện chi tiết kỹ năng (từ gameData.skills)
function showSkillDetail(skillId){
  const sk = gameData.skills.find(s=>s.id===skillId);
  const learned = player.skills.find(s=>s.id===skillId);
  if(!sk) return;
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>✨ ${sk.name} ${learned ? `(Lv ${learned.lvl})` : ''}</h3>
    <div><strong>Mô tả:</strong> ${sk.description || "-"}</div>
    ${sk.cooldown ? `<div>⏱️ Hồi chiêu: ${sk.cooldown}s</div>` : ''}
    ${sk.manaCost ? `<div>🔹 Tiêu hao Mana: ${sk.manaCost}</div>` : ''}
    ${sk.damage ? `<div><strong>Sát thương:</strong>${Object.entries(sk.damage).map(([k,v])=>`<div class="small">- ${k}: ${v}</div>`).join('')}</div>` : ''}
    <div style="display:flex;gap:8px;margin-top:8px">
      <button onclick="closeModal()">Đóng</button>
    </div>`;
}

// Resume training if there's a saved training state (offline progress)
function resumeTrainingIfAny(){
  try{
    const raw = localStorage.getItem('rpg_training');
    if(!raw) return;
    const st = JSON.parse(raw);
    if(!st || !st.techId) return;
    // find technique in player
    const t = player.techniques.find(x=>x.id === st.techId);
    const data = gameData.techniques.find(tt=>tt.id === st.techId);
    if(!t || !data) { localStorage.removeItem('rpg_training'); return; }

    // calculate elapsed seconds since stored start
    const elapsedMs = Date.now() - (st.start || Date.now());
    const elapsedSec = Math.max(0, Math.floor(elapsedMs / 1000));

    // apply offline exp: initialExp + elapsedSec
    t.exp = (st.initialExp || 0) + elapsedSec;

    // handle level-ups (with cap tech lvl <= player.level + 5)
    let loopGuard = 0;
    while(loopGuard < 100){
      loopGuard++;
      const levelData = (data.levelDetails||[]).find(ld => ld.level === t.lvl) || {};
      const expMax = levelData.expRequired || 100;
      if((t.exp || 0) >= expMax){
        if(t.lvl + 1 > (player.level || 1) + 5){
          t.exp = Math.min(t.exp, expMax - 1);
          break;
        }
        t.exp -= expMax;
        t.lvl = (t.lvl || 1) + 1;
      } else break;
    }

    saveGame();

    // reopen modal and continue training from now (so animation + interval continue)
    // set new training state start at now with initialExp = current t.exp
    currentTrainingTechId = st.techId;
    const newState = { techId: st.techId, start: Date.now(), initialExp: t.exp||0, techLvl: t.lvl||1 };
    try{ localStorage.setItem('rpg_training', JSON.stringify(newState)); }catch(e){}
    // open the modal and call trainTechnique to start intervals/animation
    trainTechnique(st.techId);
  }catch(e){
    console.error('resumeTrainingIfAny failed', e);
  }
}
  
  function viewSkill(id, source){
  const learned = player.skills.find(x=>x.id===id);
  const data = gameData.skills.find(s=>s.id===id);
  if(!learned || !data) return;

  $('modal').style.display = 'flex';

  let html = `<h3>✨ ${data.name} (Lv ${learned.lvl})</h3>`;
  html += `<div><strong>Mô tả:</strong> ${data.description}</div>`;
  if(data.cooldown) html += `<div>⏱️ Hồi chiêu: ${data.cooldown}s</div>`;
  if(data.manaCost) html += `<div>🔹 Tiêu hao Mana: ${data.manaCost}</div>`;

  // Hiện sát thương
  if(data.damage){
    html += `<div><strong>Sát thương:</strong>`;
    Object.entries(data.damage).forEach(([type,val])=>{
      html += `<div class="small">- ${type}: ${val}</div>`;
    });
    html += `</div>`;
  }

  // Chỉ hiển thị nút Trang bị khi mở từ "Kỹ năng đã học"
  if(source === "learned"){
    html += `
      <div style="margin-top:8px;display:flex;gap:8px">
        <button onclick="equipSkill('${id}')">Trang bị</button>
        <button onclick="closeModal()">Đóng</button>
      </div>`;
  } else {
    html += `<div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>`;
  }

  $('modalCard').innerHTML = html;
}


  function trainSkill(skillId){
  const skill = player.skills.find(s=>s.id===skillId);
  const skillData = gameData.skills.find(s=>s.id===skillId);
  if(!skill || !skillData) return;

  // tìm công pháp đã mở kỹ năng này
  const tech = player.techniques.find(t=>{
    const base = gameData.techniques.find(tt=>tt.id===t.id);
    return base && base.skillsUnlocked && base.skillsUnlocked.includes(skillId);
  });
  if(!tech){
    showModal("Thông báo", "Kỹ năng này chưa liên kết công pháp nào.");
    return;
  }

  // lấy quy tắc nâng cấp từ công pháp
  const techData = gameData.techniques.find(tt=>tt.id===tech.id);
  const rules = techData.skillLevelUpRules?.filter(r=>r.skillId===skillId) || [];

  // xác định cấp kỹ năng cao nhất mà công pháp hiện tại cho phép
  let allowedLv = 1;
  rules.forEach(r=>{
    if(tech.lvl >= r.techLv && r.skillLv > allowedLv){
      allowedLv = r.skillLv;
    }
  });

  if(allowedLv > skill.lvl){
    skill.lvl = allowedLv;
    saveGame();
    renderSkills();

    showModal("Nâng cấp kỹ năng",
      `${skillData.name} đã tăng lên Lv ${skill.lvl} (nhờ công pháp ${techData.name} Lv ${tech.lvl}).`
    );
  } else {
    showModal("Tu luyện kỹ năng",
      `Công pháp ${techData.name} (Lv ${tech.lvl}) chưa đạt mốc để nâng cấp kỹ năng này.`
    );
  }
}


    const statDescriptions = {
      MaxHP:"Sinh mệnh tổng thể. Có tốc độ hồi phục riêng.",
      MaxMana:"Nguồn năng lượng. Dùng cho kỹ năng, ma pháp.",
      MaxSucBen:"Ảnh hưởng tấn công vật lý, chịu đòn.",
      TheChat:"Duy trì hoạt động liên tục, dùng khi vận động kéo dài.",
      TriLuc:"Tốc độ học tập, thi triển kỹ năng trí tuệ.",
      NhanhNhen:"Phản xạ, tốc độ đánh, né tránh.",
      MayMan:"Tỷ lệ phát hiện vật phẩm, bạo kích, duyên số.",
      Khang:"Kháng trạng thái như cháy, đóng băng, độc, chảy máu,...",
      CanCot:"Cần để học công pháp.",
      NgoTinh:"Ngộ tính ảnh hưởng các tương tác học tập/kinh nghiệm."
    };

    function translateStatName(k){
      const map = { MaxHP: 'HP', MaxMana: 'Mana', MaxSucBen: 'Sức Bền', TheChat: 'Thể Chất', TriLuc: 'Trí Lực', NhanhNhen: 'Nhanh Nhẹn', MayMan: 'May Mắn', Khang: 'Kháng', CanCot: 'Căn Cốt', NgoTinh: 'Ngộ Tính' };
      return map[k] || k;
    }

    function getStatSources(statKey){
  let sources = [];
  // chỉ số gốc
  const base = playerDefault.stats[statKey] || 0;
  sources.push(`Gốc: ${base}`);
  // trang bị
  let equipVal = 0;
  Object.values(player.equipment).forEach(eid=>{
    const it = findItemById(eid);
    if(it && it.stats && statKey in it.stats) equipVal += it.stats[statKey];
  });
  if(equipVal) sources.push(`Trang bị: +${equipVal}`);
  // công pháp
  // công pháp (nguồn từ các công pháp đã học)
  (player.techniques||[]).forEach(t => {
    const tech = gameData.techniques.find(td => td.id === t.id);
    if (!tech || !Array.isArray(tech.levelDetails)) return;
    const current = tech.levelDetails.find(ld => ld.level === t.lvl);
    if (current && current.bonus && statKey in current.bonus) {
      sources.push(`Công pháp (${tech.name} Lv${t.lvl}): +${current.bonus[statKey]}`);
    }
  });
     
  // danh hiệu (ví dụ)
  if(player.titleBonuses && statKey in player.titleBonuses){
    sources.push(`Danh hiệu: +${player.titleBonuses[statKey]}`);
  }
  // buff/debuff
  if(player.tempBonuses && statKey in player.tempBonuses){
    sources.push(`Buff/Debuff: ${player.tempBonuses[statKey]}`);
  }
  return sources.join("<br>");
}

function renderStats(){
  const container = $('statsList'); if(!container) return;
  container.innerHTML = '';
  if(!player) { container.innerHTML = '<div class="list-item">Dữ liệu nhân vật trống</div>'; return; }

  const stats = Object.assign({}, playerDefault.stats || {}, player.stats || {});
  const wrapper = document.createElement('div'); wrapper.className='stat-row';
  const hidden = new Set(['HP','Mana','SucBen']);
  const displayOrder = ['MaxHP','MaxMana','MaxSucBen','TheChat','TriLuc','NhanhNhen','MayMan','Khang','CanCot','NgoTinh'];

  displayOrder.forEach(k=>{
    if(hidden.has(k)) return;
    if(!(k in stats)) return;
    const val = calcFinalStat(stats[k], k);
    const btn = document.createElement('button'); btn.className = 'stat-btn';
    btn.innerText = `${translateStatName(k)}: ${val}`;
    btn.onclick = ()=>{
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>${translateStatName(k)}</h3>
    <p>${statDescriptions[k]||'Không có mô tả.'}</p>
    <div style="margin-top:8px;font-size:12px;opacity:0.8">
      <strong>Nguồn chỉ số:</strong><br>${getStatSources(k)}
    </div>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button onclick="closeModal()">Đóng</button>
    </div>`;

  // 🧩 Đảm bảo modal nổi trên giao diện 👤
  const modal = $('modal');
  if (modal) modal.style.zIndex = "13050"; // cao hơn charPopup (12000)
};

    wrapper.appendChild(btn);
  });

  if(wrapper.children.length === 0){
    container.innerHTML = '<div class="list-item">Không có chỉ số để hiển thị</div>';
  } else container.appendChild(wrapper);
}

  
  function renderEquipmentSlots(){
  document.querySelectorAll('.equip-slot').forEach(el=>{
    const slot = el.dataset.slot; 
    const eid = player.equipment[slot];
    if(eid){
      const item = findItemById(eid);
      el.innerText = item ? getItemIcon(item) : "✔️";
      el.classList.add("equipped");
    } else {
      if(slot.includes("ring")) el.innerText="💍";
      else if(slot==="head") el.innerText="👑";
      else if(slot==="body") el.innerText="🛡️";
      else if(slot==="hand") el.innerText="🖐️";
      else if(slot==="leg") el.innerText="🦵";
      else if(slot==="main-weapon") el.innerText="⚔️";
      else if(slot==="sub-weapon") el.innerText="🗡️";
      else el.innerText="📿";
      el.classList.remove("equipped");
    }
  });
}
// === Auto trang bị vào slot hợp lệ ===
function autoEquipItem(itemId){
  const data = findItemById(itemId);
  if(!data) return;

  let targetSlot = null;
  if(data.type === 'weapon') targetSlot = player.equipment["main-weapon"] ? "sub-weapon" : "main-weapon";
  else if(data.type === 'armor') targetSlot = "body";
  else if(data.type === 'ring'){
    const ringSlots = ["ring1","ring2","ring3","ring4","ring5"];
    targetSlot = ringSlots.find(s=>!player.equipment[s]) || "ring1";
  }
  else if(data.slot === "necklace") targetSlot = "necklace";

  if(targetSlot) equipItem(itemId, targetSlot);
}
  
  function equipItem(itemId,slot){
  const idx=player.inventory.findIndex(x=>String(x.id)===String(itemId));
  if(idx===-1) return;
  if(player.equipment[slot]) player.inventory.push({id:player.equipment[slot],qty:1});
  player.equipment[slot]=itemId; player.inventory.splice(idx,1);
  saveGame(); renderInventory(); renderStats(); renderEquipmentSlots(); updateHeader(); closeModal(); updateBars();
}
function unequipItem(slot){
  if(!player.equipment[slot]) return;
  player.inventory.push({id:player.equipment[slot],qty:1}); player.equipment[slot]=null;
  saveGame(); renderInventory(); renderStats(); renderEquipmentSlots(); updateHeader(); closeModal(); updateBars();
}
  
    function openEquipSlot(slot){
  $('modal').style.display='flex'; let html=`<h3>Slot: ${slot}</h3>`;
  if(player.equipment[slot]){
    const item=findItemById(player.equipment[slot]);
    html+=`<p>Đang trang bị: ${item?item.name:"Item"}</p><button onclick="unequipItem('${slot}')">Tháo ra</button>`;
  } else {
    const equips=player.inventory.filter(it=>{ const d=findItemById(it.id); if(!d) return false;
      if(d.type==="weapon"&&(slot==="main-weapon"||slot==="sub-weapon")) return true;
      if(d.type==="armor"&&slot==="body") return true;
      if(d.type==="ring"&&slot.startsWith("ring")) return true;
      if(d.slot==="necklace"&&slot==="necklace") return true;
      return false; });
    if(equips.length===0) html+="<p>Không có trang bị phù hợp</p>";
    else equips.forEach(inv=>{ const d=findItemById(inv.id);
      html+=`<div class="list-item"><span>${d.name}</span><button onclick="equipItem('${inv.id}','${slot}')">Trang bị</button></div>`; });
  }
  $('modalCard').innerHTML=html+`<div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>`;
}

    document.addEventListener('click', function globalClickHandler(e){
  if(e.target && e.target.id === 'modal'){
    // nếu đang tu luyện thì không cho đóng modal bằng click ngoài
    if(trainingInProgress){
      // optional: bạn có thể hiện tooltip nhỏ thông báo "Đang tu luyện, hãy bấm Dừng để thoát"
      return;
    }
    closeModal();
  }
});  
// === Định nghĩa 16 tư thế stickman ===
const poses = [
  // Pose 0: đứng thẳng
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:160}, elbowR:{x:230,y:160},
    handL:{x:170,y:200}, handR:{x:230,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 1: khoanh tay (trái chéo)
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:190,y:150}, elbowR:{x:210,y:150},
    handL:{x:210,y:160}, handR:{x:190,y:160},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 2: khoanh tay (phải chéo)
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:190,y:150}, elbowR:{x:210,y:150},
    handL:{x:210,y:165}, handR:{x:190,y:165},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 3: tay chống hông phải
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:150}, elbowR:{x:220,y:160},
    handL:{x:170,y:180}, handR:{x:215,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 4: tay ôm bụng
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:190,y:150}, elbowR:{x:210,y:150},
    handL:{x:195,y:190}, handR:{x:205,y:190},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 5: tay trong túi
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:185,y:160}, elbowR:{x:215,y:160},
    handL:{x:185,y:200}, handR:{x:215,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 6: gãi đầu trái
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:160,y:100}, elbowR:{x:230,y:160},
    handL:{x:180,y:70}, handR:{x:230,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 7: hai tay sau đầu
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:160,y:100}, elbowR:{x:240,y:100},
    handL:{x:170,y:70}, handR:{x:230,y:70},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 8: hai tay chống hông
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:180,y:150}, elbowR:{x:220,y:150},
    handL:{x:185,y:200}, handR:{x:215,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 9: hai tay trong túi
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:185,y:160}, elbowR:{x:215,y:160},
    handL:{x:185,y:200}, handR:{x:215,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 10: đứng vắt chéo chân trái
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:160}, elbowR:{x:230,y:160},
    handL:{x:170,y:200}, handR:{x:230,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:195,y:230}, kneeR:{x:215,y:240},
    footL:{x:205,y:280}, footR:{x:215,y:280}
  },
  // Pose 11: đứng vắt chéo chân phải
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:160}, elbowR:{x:230,y:160},
    handL:{x:170,y:200}, handR:{x:230,y:200},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:205,y:230},
    footL:{x:185,y:280}, footR:{x:195,y:280}
  },
  // Pose 12: dựa nghiêng trái
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:160,y:150}, elbowR:{x:230,y:150},
    handL:{x:150,y:180}, handR:{x:230,y:200},
    hipL:{x:180,y:200}, hipR:{x:210,y:200},
    kneeL:{x:175,y:240}, kneeR:{x:210,y:240},
    footL:{x:170,y:280}, footR:{x:210,y:280}
  },
  // Pose 13: dựa nghiêng phải
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:150}, elbowR:{x:240,y:150},
    handL:{x:170,y:200}, handR:{x:250,y:180},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:225,y:240},
    footL:{x:185,y:280}, footR:{x:225,y:280}
  },
  // Pose 14: ngả lưng thư giãn
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:160,y:100}, elbowR:{x:240,y:100},
    handL:{x:170,y:70}, handR:{x:230,y:70},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  },
  // Pose 15: chỉ tay sang phải
  {
    head:{x:200,y:80}, neck:{x:200,y:120},
    shoulderL:{x:190,y:120}, shoulderR:{x:210,y:120},
    elbowL:{x:170,y:160}, elbowR:{x:250,y:120},
    handL:{x:170,y:200}, handR:{x:280,y:120},
    hipL:{x:190,y:210}, hipR:{x:210,y:210},
    kneeL:{x:185,y:240}, kneeR:{x:215,y:240},
    footL:{x:185,y:280}, footR:{x:215,y:280}
  }
];

// scale & offset để phóng to nhân vật
const scale = 1.3;
const offsetX = 200;
const offsetY = 10;

// === Quản lý chọn pose ===
let currentPose = JSON.parse(JSON.stringify(poses[0]));
let targetPose = poses[0];
let tPose = 0;
 let poseHold = 0;

function pickNextPose(){
  let i = pickPose();   // dùng logic random có trọng số
  targetPose = poses[i];

  // ---- cập nhật trọng số ----
  if(i > 0){ // bỏ qua pose 0 (tư thế gốc)
    poseWeights[i] = Math.max(1, poseWeights[i]-0.5);
    for(let j=1;j<poseWeights.length;j++){
      if(j!==i) poseWeights[j]+=0.2;
    }
  }

  return i;
}
let poseWeights = new Array(poses.length).fill(1);
poseWeights[0] = 0; // tư thế gốc không random

function pickPose(){
  let total = poseWeights.reduce((a,b)=>a+b,0);
  let r = Math.random() * total;
  let sum = 0;
  for(let i=0;i<poseWeights.length;i++){
    sum += poseWeights[i];
    if(r <= sum) return i;
  }
}

function applyScale(joint){
  return {
    x: offsetX + (joint.x-200) * scale,
    y: offsetY + (joint.y-60) * scale
  };
}

// easing
function easeInOut(t){
  return t<0.5 ? 2*t*t : -1+(4-2*t)*t;
}

// --- Blink (nhấp nháy khi HP ≤ 10%) — khởi tạo 1 lần ---
(function initCharBlink(){
  if (typeof window.blinkToggle === "undefined") {
    window.blinkToggle = true;
  }
  if (typeof window.blinkTimerId === "undefined") {
    window.blinkTimerId = setInterval(function(){
      window.blinkToggle = !window.blinkToggle;
    }, 300);
  }
})();

function drawCharacterBase(){
  const c = document.getElementById("charCanvas");
  if(!c) return;
  const ctx = c.getContext("2d");
  ctx.clearRect(0,0,c.width,c.height);

  if (typeof player === "undefined" || !player.stats) return;

  const hpNow = Number(player.stats.HP) || 0;
  const hpMax = Number(player.stats.MaxHP) || 1;
  const hpPct = (hpNow / hpMax) * 100;
 

  // Màu
let strokeColor = "#fff";
ctx.globalAlpha = 1.0;
if (hpNow <= 0) {
  strokeColor = "gray";
  ctx.globalAlpha = 0.4;
} else {
  if (hpPct <= 10) {
    strokeColor = window.blinkToggle ? "red" : "rgba(255,0,0,0.6)";
  } else if (hpPct <= 20) {
    strokeColor = "red";
  } else if (hpPct <= 50) {
    strokeColor = "yellow";
  } else if (hpPct <= 80) {
    strokeColor = "lime";
  } else {
    strokeColor = "white";
  }
}


  // Thở gấp
  let breathSpeed = 0.002, breathAmp = 2;
  if(hpPct <= 80){ breathSpeed = 0.004; breathAmp = 4; }
  if(hpPct <= 50){ breathSpeed = 0.006; breathAmp = 6; }
  if(hpPct <= 20){ breathSpeed = 0.008; breathAmp = 8; }
  if(hpPct <= 10){ breathSpeed = 0.012; breathAmp = 10; }
  const breathing = hpNow>0 ? Math.sin(Date.now()*breathSpeed)*breathAmp : 0;


  let joints = {};
  let tt = easeInOut(tPose);
  for (let k in currentPose){
    joints[k] = {
      x: currentPose[k].x + (targetPose[k].x - currentPose[k].x) * tt,
      y: currentPose[k].y + (targetPose[k].y - currentPose[k].y) * tt
    };
    joints[k] = applyScale(joints[k]);
  }

  // áp breathing vào cổ & ngực
  if (joints.head) joints.head.y += breathing * 0.3;
if (joints.neck) joints.neck.y += breathing * 0.6;
if (joints.hipL && joints.hipR) {
  joints.hipL.y += breathing * 0.4;
  joints.hipR.y += breathing * 0.4;
}


  // --- Vẽ ---

  // Nếu joints không đầy đủ -> bail out
  if(!joints || !joints.head || !joints.neck || !joints.shoulderL) return;

  // compute mid-hip (để vẽ thân chính)
  const midHip = {
    x: ( (joints.hipL?.x || joints.hip?.x || 200) + (joints.hipR?.x || joints.hip?.x || 200) ) / 2,
    y: ( (joints.hipL?.y || joints.hip?.y || 200) + (joints.hipR?.y || joints.hip?.y || 200) ) / 2
  };

  // stroke/fill style (strokeColor phải tồn tại ở scope trên)
  ctx.strokeStyle = (typeof strokeColor !== 'undefined') ? strokeColor : "#fff";
  ctx.fillStyle = ctx.strokeStyle;
  ctx.lineCap = "round";
  ctx.lineJoin = "round";

  // Vẽ thân (cổ → midHip) dày
  ctx.lineWidth = 35;
  ctx.beginPath();
  ctx.moveTo(joints.neck.x, joints.neck.y);
  ctx.lineTo(midHip.x, midHip.y);
  ctx.stroke();

  // Vẽ đầu
  ctx.beginPath();
  ctx.arc(joints.head.x, joints.head.y, 30, 0, Math.PI*2);
  ctx.fill();

  // Mắt
  ctx.fillStyle = "#000";
  ctx.beginPath();
  ctx.arc(joints.head.x - 10, joints.head.y - 8, 5, 0, Math.PI*2);
  ctx.arc(joints.head.x + 10, joints.head.y - 8, 5, 0, Math.PI*2);
  ctx.fill();
  ctx.fillStyle = ctx.strokeStyle;

  // Vẽ tay: vai -> khuỷu -> tay
  ctx.lineWidth = 12;
  ctx.beginPath();
  ctx.moveTo(joints.shoulderL.x, joints.shoulderL.y);
  ctx.lineTo(joints.elbowL.x, joints.elbowL.y);
  ctx.lineTo(joints.handL.x, joints.handL.y);
  ctx.moveTo(joints.shoulderR.x, joints.shoulderR.y);
  ctx.lineTo(joints.elbowR.x, joints.elbowR.y);
  ctx.lineTo(joints.handR.x, joints.handR.y);
  ctx.stroke();

  // Vẽ chân: hipL -> kneeL -> footL  & hipR -> kneeR -> footR
  ctx.lineWidth = 14;
  ctx.beginPath();
  ctx.moveTo(joints.hipL.x, joints.hipL.y);
  ctx.lineTo(joints.kneeL.x, joints.kneeL.y);
  ctx.lineTo(joints.footL.x, joints.footL.y);
  ctx.moveTo(joints.hipR.x, joints.hipR.y);
  ctx.lineTo(joints.kneeR.x, joints.kneeR.y);
  ctx.lineTo(joints.footR.x, joints.footR.y);
  ctx.stroke();

  // Vẽ các "khớp" nhỏ để nhìn rõ
  ctx.fillStyle = ctx.strokeStyle;
if (player.stats.HP > 0 && player.stats.HP / player.statsMaxHP > 0.1) {
  // HP > 10% → luôn vẽ khớp
  for (let k in joints) {
    if (joints[k]) {
      ctx.beginPath();
      ctx.arc(joints[k].x, joints[k].y, 6, 0, Math.PI*2);
      ctx.fill();
    }
  }
} else if (player.stats.HP > 0 && player.stats.HP / player.stats.MaxHP <= 0.1) {
  // HP ≤ 10% → chỉ vẽ khớp khi blinkToggle = true
  if (window.blinkToggle) {
    for (let k in joints) {
      if (joints[k]) {
        ctx.beginPath();
        ctx.arc(joints[k].x, joints[k].y, 6, 0, Math.PI*2);
        ctx.fill();
      }
    }
  }
}
// HP = 0 → không vẽ khớp

}


function animateChar() {
  let hpNow = player.statsHP; // hoặc player.stats.HP nếu bạn dùng chữ hoa

  if (hpNow > 0) {
    // tăng nội suy giữa currentPose và targetPose
    tPose += 0.01;
    if (tPose >= 1) {
      if (poseHold < 2400) { // giữ pose lâu hơn
        poseHold++;
        tPose = 1;
      } else {
        // kết thúc pose hiện tại, chuẩn bị pose tiếp theo
        currentPose = JSON.parse(JSON.stringify(targetPose));
        let nextIndex = (targetPose === poses[0]) ? pickNextPose() : 0;
        targetPose = poses[nextIndex];
        tPose = 0;
        poseHold = 0;
      }
    }
  } else {
    // HP = 0 → nhân vật về tư thế gốc
    targetPose = poses[0];
    currentPose = poses[0];
  }

  // tạo joints nội suy giữa currentPose và targetPose
  let joints = {};
  let tt = easeInOut(tPose);
  for (let k in currentPose) {
    joints[k] = {
      x: currentPose[k].x + (targetPose[k].x - currentPose[k].x) * tt,
      y: currentPose[k].y + (targetPose[k].y - currentPose[k].y) * tt
    };
    joints[k] = applyScale(joints[k]);
  }

  // hiệu ứng lắc tay nhẹ nếu đang ở tư thế gốc
  if (targetPose === poses[0]) {
    let sway = Math.sin(Date.now() / 400) * 2;
    if (joints.handL) joints.handL.x += sway;
    if (joints.handR) joints.handR.x -= sway;
  }

  return joints; // để drawCharacterBase vẽ
}

function gameLoop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // cập nhật nhân vật
  let joints = animateChar();

  // vẽ nhân vật
  drawCharacterBase(ctx, joints);

  // cập nhật nhấp nháy
  updateBlinking();

  requestAnimationFrame(gameLoop);
}
// --- FULLSCREEN MAP MODULE (thay thế phần map cũ) ---
// 🌍 Các loại khu vực địa hình
const REGION_TYPES = {
  desert:     { name: "Sa mạc",      color: "#e1c16e", nodeColor: "#c2a04e" },
  ice:        { name: "Hàn băng",    color: "#aeeaff", nodeColor: "#88ccee" },
  cave:       { name: "Hang động",   color: "#555555", nodeColor: "#777777" },
  mountain:   { name: "Đồi núi",     color: "#556b2f", nodeColor: "#6b8e23" },
  wasteland:  { name: "Đất hoang",   color: "#8b4513", nodeColor: "#a0522d" },
  sea:        { name: "Biển",        color: "#006699", nodeColor: "#3399cc" },
  deepsea:    { name: "Lòng biển",   color: "#001f3f", nodeColor: "#003366" },
  island:     { name: "Đảo",         color: "#228b22", nodeColor: "#2e8b57" },
  volcano:    { name: "Núi lửa",     color: "#cc3300", nodeColor: "#ff4500" },
  jungle:     { name: "Rừng rậm",    color: "#0b6623", nodeColor: "#228b22" },
  plain:      { name: "Đồng bằng",   color: "#b4e197", nodeColor: "#9cd67d" },
  swamp:      { name: "Đầm lầy",     color: "#3b5323", nodeColor: "#556b2f" },
  tundra:     { name: "Tuyết phủ",   color: "#dfefff", nodeColor: "#b0d8e5" },
  grassland:  { name: "Thảo nguyên", color: "#66aa33", nodeColor: "#77cc44" },
  ruins:      { name: "Tàn tích",    color: "#7a6c5d", nodeColor: "#a3917a" },
};
   let currentMapLevel = "cosmos"; // "cosmos" | "world" | "region"
let selectedWorld = null;
let selectedRegion = null;

let worlds = [];   // trong cosmos
let regions = [];  // trong world
let maps = [];     // trong region


window.addEventListener('load', () => {
  const canvas = document.getElementById("worldMapCanvas");
  if(!canvas) return;
  const ctx = canvas.getContext("2d");

  // cho phép thay đổi khi resize
  let W = 0, H = 0, CX = 0, CY = 0;
  let t = 0;

  // khai báo tất cả biến trước (tránh lỗi TDZ)
  let stars = [], asteroids = [], spiralDust = [], planets = [];

  // tinh vân tĩnh (cố định cấu trúc)
  const nebulae = [
    { xOffset:-200, yOffset:-100, r: 400, baseColors:["128,0,128","75,0,130"] },
    { xOffset:150, yOffset:50, r: 300, baseColors:["0,128,255","135,206,250"] },
    { xOffset:0, yOffset:-200, r: 350, baseColors:["255,105,180","255,182,193"] }
  ];
  const rotatingNebula = { rRatio: 0.6, colors: ["rgba(100,150,255,0.06)","rgba(200,100,255,0.04)","rgba(0,0,0,0)"], angle:0, speed:0.0005 };

 canvas.addEventListener("click", e=>{
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  if(currentMapLevel === "cosmos") {
    // chọn world
    const dx = x - CX;
    const dy = y - CY;
    const dist = Math.sqrt(dx*dx + dy*dy);
    const base = Math.min(W, H);

    const innerThreshold = base * 0.27; // Đại thiên
    const midThreshold   = base * 0.38; // Trung thiên
    const outerThreshold = base * 0.55; // Tiểu thiên

    if(dist < innerThreshold) {
      console.log("Bạn đã vào thế giới: Đại thiên");
      enterWorld({ type: "dai" });
    }
    else if(dist < midThreshold) {
      console.log("Bạn đã vào thế giới: Trung thiên");
      enterWorld({ type: "trung" });
    }
    else if(dist < outerThreshold) {
      console.log("Bạn đã vào thế giới: Tiểu thiên");
      enterWorld({ type: "tieu" });
    } else {
      console.log("Click ngoài các vòng thế giới");
    }
  } 
else if(currentMapLevel === "world") {
  for (let r of regions) {
    if(r.z > 0 && pointInRegion(ctx, x, y, r)) {
      enterRegion(r);
      break;
    }
  }
}
  else if(currentMapLevel === "region") {
    // chọn map trong region
    for (let m of maps) {
      const dx = x - m.x;
      const dy = y - m.y;
      if(Math.sqrt(dx*dx + dy*dy) <= m.r) {
        console.log("Bạn đã chọn map chi tiết")   
     enterDungeon(selectedRegion); // mở dungeon
  break;
        }
    }
  }
});

// --- Thêm biến cache toàn cục (ngay sau khi khai báo currentMapLevel, selectedWorld...) ---
let worldCache = {};   // cache regions theo world.type
let regionCache = {};  // cache maps theo region.id


// --- Sửa hàm vào World ---
function enterWorld(world){
  currentMapLevel = "world";
  selectedWorld = world;

  // ensure quota: dùng world.id nếu có, fallback world.type
  const wk = world.id || world.type || world.name || world.type;
  ensureWorldQuota(wk, world.type || "tieu");

  if(worldCache[world.type]){
    regions = worldCache[world.type];
  } else {
    regions = [];
    generateRegions();
    worldCache[world.type] = regions;
  }
  updateMapButtons();
}

// --- Sửa hàm vào Region ---
function enterRegion(region) {
  currentMapLevel = "region";
  selectedRegion = region;

  if (regionCache[region.id]) {
    continents = regionCache[region.id].continents;
    maps       = regionCache[region.id].maps;
  } else {
    generateContinents(region);   // có thể tuỳ biến theo loại
    generateMaps(region);         // sinh map riêng theo region.type
    regionCache[region.id] = {
      continents: continents,
      maps: maps
    };
  }

  // 🔹 Khởi tạo quota phòng cho region này (theo loại thế giới)
  // Nếu bạn có selectedWorld, ta dựa theo world.type; nếu không, lấy region.type.
  const worldType = (selectedWorld && selectedWorld.type) ? selectedWorld.type : (region.type || "tieu");
  const worldKey  = (selectedWorld && selectedWorld.id) ? selectedWorld.id : (region.id || region.name || worldType);

  // đảm bảo quota cho worldKey này đã tồn tại
  ensureWorldQuota(worldKey, worldType);

  console.log(`🌍 Đã vào region: ${region.name || region.id} (${worldType}) — Quota phòng còn lại: ${window.worldRoomQuota[worldKey]}`);

  updateMapButtons();
}

// --- Sửa hàm quay lại World ---
function backToWorld(){
  currentMapLevel = "world";
  selectedRegion = null;
  maps = [];
  removeDungeonControls();   // << thêm
  updateMapButtons();
  resetAllQuotas();
}

function backToCosmos(){
  currentMapLevel = "cosmos";
  selectedWorld = null;
  regions = [];
  maps = [];
  worldCache = {};
  regionCache = {};
  initPlanets();
  removeDungeonControls();   // << thêm
  updateMapButtons();
}


window.backToCosmos = backToCosmos;
window.backToWorld = backToWorld;
window.exitDungeon = exitDungeon;
function updateMapButtons(){
  const ctrl = document.getElementById("mapControls");
  if(!ctrl) return;
  ctrl.innerHTML = "";

  // Map Thế giới → có nút quay lại Vũ trụ
  if (currentMapLevel === "world") {
    ctrl.innerHTML = `
      <button onclick="backToCosmos()">⬅️ Quay lại Vũ trụ</button>
    `;
  }

  // Map Vùng (Region) → chỉ có nút quay lại Thế giới
  else if (currentMapLevel === "region") {
    ctrl.innerHTML = `
      <button onclick="backToWorld()">⬅️ Quay lại Thế giới</button>
    `;
  }

  // Map Dungeon → chỉ có nút thoát dungeon
  else if (currentMapLevel === "dungeon") {
    ctrl.innerHTML = `
      <button onclick="exitDungeon()">🚪 Thoát Dungeon</button>
    `;
  }

  // Các cấp khác (cosmos, vũ trụ, vv) thì không hiện gì
}


// tiện ích màu
  function randomColor() {
    const colors = ["#ff4444","#44ff44","#4444ff","#ffaa00","#00ffaa","#ff00aa"];
    return colors[Math.floor(Math.random()*colors.length)];
  }

  // khởi tạo kích thước + trả về lại các đối tượng
  function resizeCanvas() {
    // đặt canvas full viewport (không set CSS ở đây, chỉ kích thước bitmap)
    canvas.style.position = 'fixed';
    canvas.style.left = '0';
    canvas.style.top = '0';
    canvas.style.width = '100vw';
    canvas.style.height = '100vh';
    canvas.style.zIndex = '1';

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    W = canvas.width;
    H = canvas.height;
    CX = W/2;
    CY = H/2;

    initStars();
    initAsteroids();
    initSpiralDust();
    initPlanets(); // tạo planets dựa trên kích thước mới
  }

  // tạo sao nền
  function initStars(count=200) {
    stars = [];
    for (let i = 0; i < count; i++) {
      stars.push({
        x: Math.random()*W,
        y: Math.random()*H,
        r: Math.random()*1.6,
        tw: Math.random()*1000
      });
    }
  }

  // thiên thạch vòng
  function initAsteroids(count=250) {
    asteroids = [];
    const minWH = Math.min(W,H);
    for (let i = 0; i < count; i++) {
      asteroids.push({
        r: (minWH*0.08) + Math.random()*(minWH*0.4), // dựa theo kích thước màn hình
        size: Math.random()*2,
        speed: 0.001 + Math.random()*0.004,
        angle: Math.random()*Math.PI*2
      });
    }
  }

  // spiral dust
  function initSpiralDust(count=2000) {
    spiralDust = [];
    const minWH = Math.min(W,H);
    for (let i=0;i<count;i++){
      const angle = Math.random()*Math.PI*2;
      const radius = (minWH*0.12) + Math.random()*(minWH*0.45);
      spiralDust.push({
        angle,
        r: radius,
        speed: 0.00005 + Math.random()*0.0004,
        size: 10 + Math.random()*18
      });
    }
  }

  // planets (dùng kiểm tra khoảng cách để không chồng khi spawn)
  function initPlanets() {
    planets = [];
    const minWH = Math.min(W,H);

    function placePlanet(rMin, rMax, size, count, minDist, type) {
      let placed = 0, tries = 0;
      while(placed < count && tries < count*80) {
        tries++;
        const r = rMin + Math.random()*(rMax-rMin);
        const angle = Math.random()*Math.PI*2;
        const x = CX + Math.cos(angle)*r;
        const y = CY + Math.sin(angle)*r*0.6;
        // kiểm tra khoảng cách
        let ok = planets.every(p=>{
          const dx = x - p._x;
          const dy = y - p._y;
          return Math.sqrt(dx*dx + dy*dy) > (size + p.size + minDist);
        });
        if(ok) {
          planets.push({
            r, angle, size,
            speed: (0.0002 + Math.random()*0.0008),
            color: randomColor(),
            _x: x, _y: y,
            zPhase: Math.random()*Math.PI*2,
            zSpeed: 0.001 + Math.random()*0.003,
            type: type
          });
          placed++;
        } else {
          // nếu không ok, thử đẩy r ra/đổi góc
          // (dời r chút để tránh deadlock)
        }
      }
    }

    // các tầng (tỉ lệ theo minWH)
    const base = Math.min(W,H);
    // Đại thiên: gần tâm
    placePlanet(base*0.19, base*0.20, Math.max(12, Math.round(base*0.02)), 10, 17, "dai");
  // Trung thiên: giữa
  placePlanet(base*0.20, base*0.37, Math.max(8, Math.round(base*0.015)), 8, 30, "trung");
  // Tiểu thiên: rìa
  placePlanet(base*0.43, base*0.53, Math.max(6, Math.round(base*0.01)), 6, 50, "tieu");
  }
let usedLabels = []; // reset mỗi lần vẽ frame

function drawWorldRings() {
  const base = Math.min(W,H);
  const rings = [
    {r: base*0.26, label: "Đại Thiên Thế Giới", angle: -Math.PI/2}, // trên
    {r: base*0.39, label: "Trung Thiên Thế Giới", angle: Math.PI},   // trái
    {r: base*0.55, label: "Tiểu Thiên Thế Giới", angle: Math.PI/2},  // dưới
  ];

  usedLabels = [];

  ctx.save();
  ctx.strokeStyle = "rgba(200,255,200,0.25)";
  ctx.lineWidth = 2;

  rings.forEach(r=>{
    // ellipse
    ctx.beginPath();
    ctx.ellipse(CX, CY, r.r, r.r*0.65, 0, 0, Math.PI*2);
    ctx.stroke();

    // nhãn
    drawRingLabel(r.label, r.r, r.angle);
  });

  ctx.restore();
}
function drawRingLabel(text, radius, baseAngle) {
  ctx.font = "18px sans-serif";
  const metrics = ctx.measureText(text);
  const textW = metrics.width;
  const textH = 22;

  let angle = baseAngle; 
  let x = CX + Math.cos(angle) * radius * 1.3;
  let y = CY + Math.sin(angle) * radius * 1.0;

  const box = {x: x, y: y - textH/2, w: textW+12, h: textH};

  // Nếu box vượt ngoài màn hình hoặc chồng nhãn → dịch góc một chút
  for (let tries=0; tries<8; tries++) {
    if (box.x>=0 && box.x+box.w<=W && box.y>=0 && box.y+box.h<=H &&
        !usedLabels.some(b => !(box.x+box.w < b.x || b.x+b.w < box.x || box.y+box.h < b.y || b.y+b.h < box.y))) {
      break;
    }
    angle += Math.PI/16; // xoay nhẹ nếu trùng
    x = CX + Math.cos(angle) * radius * 1.3;
    y = CY + Math.sin(angle) * radius * 1.0;
    box.x = x; box.y = y - textH/2;
  }

  usedLabels.push(box);

  // gạch nối từ ellipse ra gạch chân chữ
  const lineStartX = x + 6;
  const lineY = y + textH/2 + 4;
  const lineEndX = lineStartX + textW + 10;

  ctx.strokeStyle = "white";
  ctx.beginPath();
  ctx.moveTo(CX + Math.cos(angle) * radius * 0.95, CY + Math.sin(angle) * radius * 0.65);
  ctx.lineTo(lineStartX, lineY); 
  ctx.stroke();

  // nền chữ
  ctx.fillStyle = "rgba(0,0,0,0.6)";
  ctx.fillRect(x+6, y-textH/2, textW+12, textH);

  // chữ
  ctx.fillStyle = "white";
  ctx.textAlign = "left";
  ctx.fillText(text, x+12, y+6);

  // gạch dưới chữ
  ctx.beginPath();
  ctx.moveTo(lineStartX, lineY);
  ctx.lineTo(lineEndX, lineY);
  ctx.stroke();
}

  // Vẽ background (stars + nebulae + rotating nebula)
  function drawBackground() {
    ctx.fillStyle = "black";
    ctx.fillRect(0,0,W,H);

    // sao nền lấp lánh
    stars.forEach(s=>{
      const tw = (Math.sin((t+s.tw)*0.02)+1)/2;
      ctx.fillStyle = `rgba(255,255,255,${0.25 + 0.7*tw})`;
      ctx.beginPath();
      ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
      ctx.fill();
    });

    // tinh vân tĩnh (vị trí dựa trên CX,CY)
    nebulae.forEach((n, idx)=>{
      const x = CX + n.xOffset;
      const y = CY + n.yOffset;
      const pulse = (Math.sin(t*0.01 + idx*2)+1)/2;
      const c1 = n.baseColors[0];
      const c2 = n.baseColors[1];
      const colA = `rgba(${c1},${0.08 + 0.22*pulse})`;
      const colB = `rgba(${c2},0)`;
      const g = ctx.createRadialGradient(x,y,0,x,y,n.r);
      g.addColorStop(0,colA);
      g.addColorStop(1,colB);
      ctx.fillStyle = g;
      ctx.beginPath();
      ctx.arc(x,y,n.r,0,Math.PI*2);
      ctx.fill();
    });

    // rotating nebula: scale theo min(W,H)
    ctx.save();
    ctx.translate(CX, CY);
    ctx.rotate(rotatingNebula.angle);
    const rotR = Math.min(W,H) * rotatingNebula.rRatio;
    const g = ctx.createRadialGradient(0,0,0,0,0,rotR);
    g.addColorStop(0, rotatingNebula.colors[0]);
    g.addColorStop(0.5, rotatingNebula.colors[1]);
    g.addColorStop(1, rotatingNebula.colors[2]);
    ctx.fillStyle = g;
    ctx.beginPath();
    ctx.arc(0,0,rotR,0,Math.PI*2);
    ctx.fill();
    ctx.restore();

    rotatingNebula.angle += rotatingNebula.speed;
  }

  function drawSpiralNebulaBackground() {
    ctx.save();
    ctx.translate(CX, CY);
    ctx.scale(1,0.6);
    spiralDust.forEach(d=>{
      // update angle
      d.angle += d.speed;
      const x = Math.cos(d.angle)*d.r;
      const y = Math.sin(d.angle)*d.r;
      ctx.fillStyle = `rgba(200,150,100,0.06)`;
      ctx.beginPath();
      ctx.arc(x,y,d.size,0,Math.PI*2);
      ctx.fill();
    });
    ctx.restore();
  }

  // Vẽ các hành tinh: update vị trí, dao động r, z, alpha; sort theo y trước khi vẽ
  function drawPlanets() {
    // cập nhật vị trí (tính _x,_y trước)
    planets.forEach(p=>{
      p.angle += p.speed;
      const rOffset = 6 * Math.sin(t*0.001 + p.zPhase); // trôi ra/vào nhẹ
      const rDynamic = p.r + rOffset;
      const baseX = CX + Math.cos(p.angle)*rDynamic;
      const baseY = CY + Math.sin(p.angle)*rDynamic*0.6;

      // z dao động
      const z = Math.sin(t*p.zSpeed + p.zPhase); // -1..1
      const zHeight = z * (Math.min(W,H)*0.06); // biên độ phụ thuộc màn hình
      const wobble = Math.sin(t*0.002 + p.zPhase*3) * (Math.min(W,H)*0.02);

      p._x = baseX + wobble*0.4;
      p._y = baseY + zHeight + wobble*0.6;
      p._depth = (z+1)/2; // 0..1
      p._size = p.size * (0.8 + 0.4*p._depth);
      p._zval = z;
    });

    // sort theo y để vẽ đúng lớp trước/sau
    planets.sort((a,b) => (a._y - b._y));

    // vẽ
    planets.forEach(p=>{
      const size = p._size;
      // ánh sáng trung tâm (gần tâm sáng hơn)
      const dx = p._x - CX, dy = p._y - CY;
      const dist = Math.sqrt(dx*dx + dy*dy);
      const lightFactor = Math.max(0, 1 - dist / (Math.min(W,H)*0.5));
      let alpha = (0.45 + 0.55*lightFactor) * (0.45 + 0.55*p._depth);

      // chìm sâu -> mờ thêm
      if(p._zval < -0.6) alpha *= Math.max(0, 1 + p._zval + 0.6);

      // gradient hành tinh
      const grad = ctx.createRadialGradient(
        p._x - size*0.5, p._y - size*0.5, 1,
        p._x, p._y, size
      );
      grad.addColorStop(0, `rgba(255,255,255,${0.45*alpha})`);
      grad.addColorStop(0.45, p.color);
      grad.addColorStop(1, `rgba(0,0,0,0.8)`);

      ctx.beginPath();
      ctx.fillStyle = grad;
      ctx.arc(p._x, p._y, size, 0, Math.PI*2);
      ctx.fill();

      // khi chìm (z < 0) phủ fog cục bộ (phần dấu hiệu chìm)
      if(p._zval < 0) {
        ctx.beginPath();
        ctx.fillStyle = `rgba(200,180,150,${0.08 * Math.abs(p._zval)})`;
        ctx.arc(p._x, p._y, size*1.15, 0, Math.PI*2);
        ctx.fill();
      }
    });
  }

  // foreground fog (che phía trên hành tinh)
  function drawNebulaForeground() {
    ctx.save();
    ctx.translate(CX, CY);
    ctx.scale(1,0.6);
    // Chọn một phần spiralDust để vẽ foreground, và dùng alpha động
    for (let i=0;i<spiralDust.length;i+=3){
      const d = spiralDust[i];
      // vị trí cập nhật đã trong drawSpiralNebulaBackground, dùng d.angle
      const x = Math.cos(d.angle)*d.r;
      const y = Math.sin(d.angle)*d.r;
      // alpha dao động nhịp nhàng để foreground động
      const a = 0.06 + 0.06 * Math.sin(t*0.003 + i);
      ctx.fillStyle = `rgba(200,150,100,${a})`;
      ctx.beginPath();
      ctx.arc(x,y,d.size*1.2,0,Math.PI*2);
      ctx.fill();
    }
    ctx.restore();
  }
 // --- VẼ thiên thạch vòng (asteroids) ---
function drawAsteroids() {
  if(!asteroids || asteroids.length === 0) return;
  asteroids.forEach(a=>{
    a.angle += a.speed;
    // vị trí theo tâm, giữ tỉ lệ theo chiều cao để có hiệu ứng ellipse
    const x = CX + Math.cos(a.angle) * a.r;
    const y = CY + Math.sin(a.angle) * a.r * 0.5;
    // tính xem đang "ở phía sau" hay "phía trước" (dùng sin để phân lớp)
    const behind = Math.sin(a.angle) > 0;

    ctx.beginPath();
    // đảm bảo radius tối thiểu để nhìn rõ trên màn nhỏ
    const drawSize = Math.max(0.6, a.size * (behind ? 0.6 : 1));
    ctx.arc(x, y, drawSize, 0, Math.PI * 2);

    // tone mờ phía sau, sáng phía trước
    ctx.fillStyle = behind ? "rgba(180,180,180,0.28)" : "rgba(220,220,220,0.9)";
    ctx.fill();
  });
}

// --- VẼ lõi trung tâm (core) với gradient thích ứng màn hình ---
function drawCore(t) {
  const coreR = Math.max(7, Math.min(W,H)*0.07); 
  const pulse = 1 + 0.08*Math.sin(t*0.05); 

  // quả cầu
  ctx.save();
  const gSphere = ctx.createRadialGradient(CX, CY, 0, CX, CY, coreR);
  gSphere.addColorStop(0, "rgba(80,40,20,1)");  // lõi tối
  gSphere.addColorStop(1, "rgba(20,10,5,1)");
  ctx.fillStyle = gSphere;
  ctx.beginPath();
  ctx.arc(CX, CY, coreR, 0, Math.PI*2);
  ctx.fill();

  // quầng sáng
  const glowR = coreR*3*pulse;
  const gGlow = ctx.createRadialGradient(CX, CY, coreR*0.8, CX, CY, glowR);
  gGlow.addColorStop(0, "rgba(255,220,120,0.9)");
  gGlow.addColorStop(0.3, "rgba(255,150,50,0.6)");
  gGlow.addColorStop(0.6, "rgba(255,100,20,0.25)");
  gGlow.addColorStop(1, "rgba(0,0,0,0)");
  ctx.fillStyle = gGlow;
  ctx.beginPath();
  ctx.arc(CX, CY, glowR, 0, Math.PI*2);
  ctx.fill();

  ctx.restore();
}

 // ====================== WORLD MAP ======================
// --- thêm 1 biến toàn cục ở đầu module map (cùng scope với W,H,CX,CY,regions)
let mapLonOffset = 0;

// --- Hàm tạo đa giác (1 lần) ---
function generatePolygon(r, irregularity=0.5, points=12) {
  const shape = [];
  for (let i=0; i<points; i++) {
    const angle = (i/points)*Math.PI*2;
    const radius = r * (0.7 + Math.random()*irregularity);
    shape.push({ angle, radius });
  }
  return shape;
}

// --- Thay thế generateRegions() ---


function generateRegions() {
  regions = [];
  const base = Math.min(W, H);
  const R = Math.min(W, H) / 3; 
  let targetCount;
  if (selectedWorld.type === "dai") targetCount = 8 + Math.floor(Math.random()*5);
  if (selectedWorld.type === "trung") targetCount = 7 + Math.floor(Math.random()*3);
  if (selectedWorld.type === "tieu") targetCount = 5 + Math.floor(Math.random()*3);

  const minPx = Math.round(base * 0.05);
  const maxPx = Math.round(base * 0.10);

  mapLonOffset = Math.random() * Math.PI * 2;

  function sampleLatLon() {
    const u = Math.random();
    const lat = Math.asin(2*u - 1);
    const lon = Math.random() * Math.PI * 2;
    return { lat, lon };
  }

  function angularDistance(lat1, lon1, lat2, lon2) {
    const dlon = lon1 - lon2;
    const cosAng = Math.sin(lat1)*Math.sin(lat2) + Math.cos(lat1)*Math.cos(lat2)*Math.cos(dlon);
    return Math.acos(Math.max(-1, Math.min(1, cosAng)));
  }

  let attempts = 0;
  while (regions.length < targetCount && attempts < targetCount * 800) {
    attempts++;
    const { lat, lon } = sampleLatLon();
    const pxRadius = minPx + Math.random() * (maxPx - minPx);
    const angRadius = pxRadius / R;

    const rlon = lon + mapLonOffset;
    const X = R * Math.cos(lat) * Math.cos(rlon);
    const Y = R * Math.sin(lat);
    const Z = R * Math.cos(lat) * Math.sin(rlon);
    if (Z <= 0) continue;

    const distFromCenter = Math.sqrt(X*X + Y*Y);
    if (distFromCenter + pxRadius > (R - 2)) continue;

    let ok = true;
    for (const e of regions) {
      const ang = angularDistance(lat, lon, e.lat, e.lon);
      if (ang < (angRadius + e.angRadius + 0.25)) { ok = false; break; }
      const dx = (CX + X) - e.x;
      const dy = (CY + Y) - e.y;
      const dist2D = Math.sqrt(dx*dx + dy*dy);
      if (dist2D < (pxRadius + e.r) * 1.1) { ok = false; break; }
    }
    if (!ok) continue;

    // 👉 chọn loại region ngẫu nhiên từ REGION_TYPES
    const typeKeys = Object.keys(REGION_TYPES);
    const type = typeKeys[Math.floor(Math.random()*typeKeys.length)];
    const cfg = REGION_TYPES[type];

    regions.push({
      id: selectedWorld.type + "_" + type + "_" + regions.length + "_" + Date.now(), // 👈 id duy nhất
      lat, lon, r: pxRadius, angRadius,
      x: CX + X, y: CY + Y, z: Z,
      type, name: cfg.name,
      shape: generatePolygon(pxRadius, 0.5, 14)
    });
  }

  if (regions.length < targetCount) {
    console.warn(`⚠️ Chỉ sinh được ${regions.length}/${targetCount} lục địa (hạn chế chồng lấn).`);
  }
}


// --- Hàm vẽ lục địa đã có shape sẵn ---
function drawContinent(ctx, cx, cy, shape, type) {
  ctx.beginPath();
  shape.forEach((p,i)=>{
    const x = cx + Math.cos(p.angle)*p.radius;
    const y = cy + Math.sin(p.angle)*p.radius;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  });
  ctx.closePath();

  const cfg = REGION_TYPES[type] || {color:"gray"};
  ctx.fillStyle = cfg.color;     // màu nền theo loại khu vực
  ctx.strokeStyle = "white";     // viền trắng
  ctx.lineWidth = 2;
  ctx.fill();
  ctx.stroke();
}



// --- Render regions (bỏ qua mặt sau) ---
function renderRegions(ctx) {
  regions.forEach(r=>{
    if(r.z <= 0) return; 
    drawContinent(ctx, r.x, r.y, r.shape, r.type); // 👈 thêm r.type
  });
}

// --- Hàm kiểm tra click chính xác trong shape ---
function pointInRegion(ctx, x, y, region) {
  const path = new Path2D();
  region.shape.forEach((p,i)=>{
    const px = region.x + Math.cos(p.angle)*p.radius;
    const py = region.y + Math.sin(p.angle)*p.radius;
    if(i===0) path.moveTo(px, py); else path.lineTo(px, py);
  });
  path.closePath();
  return ctx.isPointInPath(path, x, y);
}

// biến toàn cục lưu sao
let worldStars = null;

function drawWorld() {
  // ===== Nền trời =====
  ctx.fillStyle = "black";
  ctx.fillRect(0,0,W,H);

  // ===== Sinh sao 1 lần duy nhất =====
  if(!worldStars){
    worldStars = [];
    const starCount = 150;
    for (let i=0; i<starCount; i++){
      worldStars.push({
        x: Math.random()*W,
        y: Math.random()*H,
        r: Math.random()*1.5 + 0.5,
        alpha: Math.random()*0.5 + 0.5,
        speed: (Math.random()*0.02 + 0.01)
      });
    }
  }

  // ===== Vẽ sao (nhấp nháy nhẹ) =====
  worldStars.forEach(s=>{
    s.alpha += (Math.random()>0.5 ? 1 : -1) * s.speed;
    if(s.alpha < 0.3) s.alpha = 0.3;
    if(s.alpha > 1)   s.alpha = 1;

    ctx.beginPath();
    ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
    ctx.fillStyle = `rgba(255,255,255,${s.alpha})`;
    ctx.fill();
  });

  // ===== Quả cầu =====
  const R = Math.min(W,H)/3;
  const g = ctx.createRadialGradient(CX, CY, R*0.1, CX, CY, R);
  g.addColorStop(0, "rgba(100,200,255,1)");
  g.addColorStop(1, "rgba(0,0,60,1)");
  ctx.fillStyle = g;
  ctx.beginPath();
  ctx.arc(CX,CY,R,0,Math.PI*2);
  ctx.fill();

  // ===== Region =====
  const visibleRegions = regions.filter(r=>r.z > 0).sort((a,b)=> a.z - b.z);
  visibleRegions.forEach(r => drawContinent(ctx, r.x, r.y, r.shape, r.type));

  // ===== Tên lục địa =====
  ctx.font = "bold 14px sans-serif";
  ctx.textAlign = "center";
  ctx.textBaseline = "bottom"; 
  visibleRegions.forEach(r => {
    let minY = Infinity, avgX = 0;
    r.shape.forEach(p => {
      const px = r.x + Math.cos(p.angle) * p.radius;
      const py = r.y + Math.sin(p.angle) * p.radius;
      avgX += px;
      if (py < minY) minY = py;
    });
    avgX /= r.shape.length;

    const labelX = avgX;
    const labelY = minY - 5;

    ctx.lineWidth = 4;
    ctx.strokeStyle = "black";
    ctx.strokeText(r.name, labelX, labelY);

    ctx.fillStyle = "white";
    ctx.fillText(r.name, labelX, labelY);
  });
}
// ====================== REGION MAP ======================

let continents = []; // mảng các polygon

function generateContinents() {
  continents = [];
  let n = 2 + Math.floor(Math.random()*3); // 2–4 châu lục
  let cols = n;              // chia ngang
  let cellW = W / cols;      // chiều rộng 1 ô

  for (let i=0; i<n; i++) {
    // tâm lục địa nằm trong ô thứ i
    let cx = (i+0.5)*cellW + (Math.random()*cellW*0.4 - cellW*0.2);
    let cy = H/2 + (Math.random()*200 - 100);

    let pts = [];
    let sides = 6; // 6 đỉnh cho tự nhiên hơn
    let radius = 100 + Math.random()*60;
    for (let a=0; a<Math.PI*2; a+= (Math.PI*2)/sides) {
      let r = radius * (0.7 + Math.random()*0.6);
      pts.push({
        x: cx + Math.cos(a)*r,
        y: cy + Math.sin(a)*r
      });
    }
    continents.push(pts);
  }
}

// kiểm tra 1 điểm có nằm trong polygon không (thuật toán ray-casting)
function pointInPolygon(x, y, poly) {
  let inside = false;
  for (let i=0, j=poly.length-1; i<poly.length; j=i++) {
    const xi = poly[i].x, yi = poly[i].y;
    const xj = poly[j].x, yj = poly[j].y;
    const intersect = ((yi > y) !== (yj > y)) &&
                      (x < (xj - xi) * (y - yi) / (yj - yi) + xi);
    if (intersect) inside = !inside;
  }
  return inside;
}

function generateMaps(region) {
  maps = [];

  // --- số node cơ bản dựa trên world.type ---
  let baseCount = 0;
  if(selectedWorld.type === "dai")   baseCount = 9 + Math.floor(Math.random()*4);
  if(selectedWorld.type === "trung") baseCount = 5 + Math.floor(Math.random()*3);
  if(selectedWorld.type === "tieu")  baseCount = 3 + Math.floor(Math.random()*3);

  // --- modifier theo region.type ---
  let modifier = 0;
  switch(region.type){
    case "cave":      modifier = +3; break;   // hang động nhiều ngõ nhỏ
    case "jungle":    modifier = +6; break;   // rừng rậm → nhiều node hơn
    case "volcano":   modifier = -2; break;   // núi lửa → ít node hơn
    case "mountain":  modifier = 0;  break;
    case "ice":       modifier = +2; break;
    case "desert":    modifier = -1; break;
    case "sea":       modifier = -2; break;
    case "island":    modifier = +1; break;
    case "wasteland": modifier = -3; break;
    default:          modifier = 0;  break;
  }

  const count = Math.max(2, baseCount + modifier); // không để < 2

  for (let i=0;i<count;i++) {
    let node, tries=0;
    do {
      node = {
        x: 50 + Math.random()*(W-100),
        y: 50 + Math.random()*(H-100),
        r: 20 + Math.random()*10,
        name: "Map " + (i+1),
        color: REGION_TYPES[region.type]?.nodeColor || "#999"
      };
      tries++;
    } while (
      (tries<100) && (
        !continents.some(poly => pointInPolygon(node.x, node.y, poly)) ||
        maps.some(m=>{
          let dx=m.x-node.x, dy=m.y-node.y;
          return Math.sqrt(dx*dx + dy*dy) < m.r+m.r+10;
        })
      )
    );
    maps.push(node);
  }
}

function drawRegion() {
  // nền biển chung
  ctx.fillStyle = "#002b55";
  ctx.fillRect(0,0,W,H);

  // chọn kiểu region hiện tại
  const cfg = REGION_TYPES[selectedRegion.type] || REGION_TYPES.jungle;

  // vẽ châu lục
  ctx.fillStyle = cfg.color;
  continents.forEach(poly=>{
    ctx.beginPath();
    ctx.moveTo(poly[0].x, poly[0].y);
    for(let k=1;k<poly.length;k++) ctx.lineTo(poly[k].x, poly[k].y);
    ctx.closePath();
    ctx.fill();
  });

  // vẽ các map node
  maps.forEach(m=>{
    ctx.beginPath();
    ctx.fillStyle = cfg.nodeColor;
    ctx.arc(m.x, m.y, m.r, 0, Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.fillStyle = "white";
    ctx.font = "14px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText(m.name, m.x, m.y - m.r - 6);
  });
}

  // vẽ map nodes
  maps.forEach(m=>{
    ctx.beginPath();
    ctx.fillStyle = "rgba(255,215,0,0.9)";
    ctx.arc(m.x, m.y, m.r, 0, Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.fillStyle = "white";
    ctx.font = "14px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText(m.name, m.x, m.y - m.r - 6);
  });
 
  // ==================== DUNGEON MODULE (V9.5 – đầy đủ: loop, resize, controls hiển thị, boss) ====================
let dungeon = null;
let playerPos = { x: 1, y: 1 };
let tileSize = 32;
let lightRadius = 8;
let dungeonCtx = null;
let dungeonBlink = 0;
let inBossRoom = false;
let dungeonLoopId = null;

// ======== HỆ THỐNG TẦNG DUNGEON ========
window.currentFloor = 1; // bắt đầu tầng 1

function floorDifficultyMultiplier(floor) {
  // hệ số tăng theo tầng: 1.0, 1.2, 1.5, 1.8, ...
  return 1 + (floor - 1) * 0.2;
}
// ⚙️ Sinh đặc điểm dungeon dựa trên loại vùng
// === Cấu hình đặc điểm Dungeon theo từng loại khu vực ===
const DUNGEON_TRAITS = Object.fromEntries(Object.entries(REGION_TYPES).map(([key, region]) => {
  let base = {
    floorColor: region.color,
    wallColor: region.nodeColor,
    chestChance: 0.1,
    enemyChance: 0.15,
    enemyType: "default",
    lavaChance: 0,
    slippery: false,
    poisonous: false,
    trapChance: 0,
    pitChance: 0,
    fog: null,
    wallSymbol: null,
  };

  switch (key) {
    case "desert":
      Object.assign(base, {
        enemyType: "sandWorm",
        chestChance: 0.08,
        fog: "light",
        trapChance: 0.03,
        wallSymbol: "⛰️",
        wallColor: "#c2a04e",
        floorColor: "#e1c16e",
      });
      break;

    case "ice":
    case "tundra":
      Object.assign(base, {
        enemyType: "iceSpirit",
        chestChance: 0.12,
        fog: "frost",
        slippery: true,
        trapChance: 0.02,
        wallSymbol: "❄️",
        wallColor: "#b0d8e5",
        floorColor: "#dfefff",
      });
      break;

    case "cave":
      Object.assign(base, {
        enemyType: "bat",
        darkness: true,
        torchChance: 0.25,
        pitChance: 0.03,
        fog: "dense",
        wallSymbol: "🪨",
        wallColor: "#3b3b3b",
        floorColor: "#1e1e1e",
      });
      break;

    case "mountain":
      Object.assign(base, {
        enemyType: "rockGolem",
        heightEffect: true,
        pitChance: 0.04,
        wallSymbol: "⛰️",
        wallColor: "#505050",
        floorColor: "#707070",
      });
      break;

    case "volcano":
      Object.assign(base, {
        enemyType: "fireDemon",
        lavaChance: 0.15,
        fog: "heat",
        trapChance: 0.02,
        wallSymbol: "🌋",
        wallColor: "#7a1f00",
        floorColor: "#3a0d00",
      });
      break;

    case "jungle":
      Object.assign(base, {
        enemyType: "giantSpider",
        vineEffect: true,
        chestChance: 0.1,
        fog: "dense",
        poisonous: true,
        trapChance: 0.02,
        wallSymbol: "🌳",
        wallColor: "#144015",
        floorColor: "#1e5620",
      });
      break;

    case "swamp":
      Object.assign(base, {
        enemyType: "slime",
        slowEffect: true,
        chestChance: 0.05,
        poisonous: true,
        fog: "dense",
        wallSymbol: "🌫️",
        wallColor: "#2f3b2d",
        floorColor: "#3b4f35",
      });
      break;

    case "sea":
      Object.assign(base, {
        enemyType: "seaSerpent",
        underwater: true,
        fog: "water",
        chestChance: 0.07,
        pitChance: 0.03,
        wallSymbol: "🌊",
        wallColor: "#006699",
        floorColor: "#3399cc",
      });
      break;

    case "deepsea":
      Object.assign(base, {
        enemyType: "deepSerpent",
        underwater: true,
        fog: "water",
        chestChance: 0.06,
        pitChance: 0.04,
        poisonous: true,
        wallSymbol: "🪸",
        wallColor: "#002b4a",
        floorColor: "#003b66",
      });
      break;

    case "island":
      Object.assign(base, {
        enemyType: "boar",
        chestChance: 0.1,
        fog: "light",
        trapChance: 0.02,
        wallSymbol: "🌴",
        wallColor: "#2a6b37",
        floorColor: "#3b8044",
      });
      break;

    case "wasteland":
      Object.assign(base, {
        enemyType: "skeleton",
        darkness: true,
        chestChance: 0.07,
        poisonous: true,
        wallSymbol: "💀",
        wallColor: "#4b3621",
        floorColor: "#7a5230",
      });
      break;

    case "ruins":
      Object.assign(base, {
        enemyType: "ancientGuardian",
        darkness: true,
        torchChance: 0.2,
        trapChance: 0.05,
        wallSymbol: "🧱",
        wallColor: "#6b5c4a",
        floorColor: "#84715a",
      });
      break;

    case "deepforest":
      Object.assign(base, {
        enemyType: "shadowWolf",
        poisonous: true,
        fog: "dense",
        chestChance: 0.08,
        wallSymbol: "🌲",
        wallColor: "#0b4f1c",
        floorColor: "#1a6f2d",
      });
      break;

    case "grassland":
      Object.assign(base, {
        enemyType: "boar",
        chestChance: 0.1,
        fog: "light",
        wallSymbol: "🌿",
        wallColor: "#3b7a2a",
        floorColor: "#4e8c3a",
      });
      break;

    case "plain":
      Object.assign(base, {
        enemyType: "boar",
        chestChance: 0.12,
        fog: "light",
        wallSymbol: "🌾",
        wallColor: "#7aa84f",
        floorColor: "#9dd465",
      });
      break;

    default:
      Object.assign(base, {
        wallSymbol: "⬜",
        wallColor: region.nodeColor || "#777",
        floorColor: region.color || "#555",
      });
      break;
  }

  return [key, base];
}));

// === WORLD ROOM QUOTA (giữ tổng số phòng theo loại thế giới) ===
window.worldRoomQuota = window.worldRoomQuota || {}; // { worldKey: remainingCount }

// trả về số ban đầu theo worldType
function initialQuotaForWorld(worldType) {
  if (worldType === "dai") return 10;
  if (worldType === "trung") return 7;
  return 4; // tieu
}

// khởi tạo quota cho 1 thế giới (gọi khi vào world lần đầu)
function ensureWorldQuota(worldKey, worldType) {
  if (!window.worldRoomQuota[worldKey]) {
    window.worldRoomQuota[worldKey] = initialQuotaForWorld(worldType);
  }
}

// tiêu 1 room nếu còn quota, trả về true nếu thành công
function consumeRoomQuota(worldKey) {
  if (!window.worldRoomQuota[worldKey]) return false;
  if (window.worldRoomQuota[worldKey] <= 0) return false;
  window.worldRoomQuota[worldKey]--;
  return true;
}

// reset quota (nếu cần) — gọi khi quay lại Cosmos hoặc khi muốn reset toàn bộ
function resetAllQuotas(preserveContext=true) {
  const backup = preserveContext ? window.currentWorldType : null;
  window.worldRoomQuota = {};
  if (preserveContext && backup) window.currentWorldType = backup;
}

// --- Gọi khi click region ---
// --- Vào dungeon ---
function enterDungeon(region) {
  currentMapLevel = "dungeon";
  inBossRoom = false;

  // 🧭 Xác định loại thế giới hiện tại
  let worldType = "tieu";
  if (selectedWorld && selectedWorld.type) {
    worldType = selectedWorld.type;
  }
  window.currentWorldType = worldType; // 🔹 lưu global để movePlayer dùng
	window.lastWorldContext = {
  type: worldType,
  worldId: selectedWorld?.id || worldType,
  worldName: selectedWorld?.name || worldType,
  regionId: region.id || null
};
// 🪐 LOG PHÂN LOẠI DUNGEON
let worldLabel = "Tiểu Thiên Thế Giới";
if (worldType === "trung") worldLabel = "Trung Thiên Thế Giới";
else if (worldType === "dai") worldLabel = "Đại Thiên Thế Giới";

console.log(`🌌 Vào Dungeon thuộc ${worldLabel}`);
console.log(`🗺️ Khu vực: ${region.name || region.id || "(Không rõ)"} | Chủ đề: ${region.type || "cave"}`);
console.log(`🏷️ worldType = ${worldType}, worldId = ${selectedWorld?.id || "?"}`);

  // 🔹 Lưu chủ đề gốc của dungeon để giữ nguyên qua các tầng
  window.dungeonBaseTheme = region.type || "cave";

  // ♻️ Reset flag để dungeon mới không kế thừa boss cũ
  window.bossRoomCreated = false;

  // 🧩 Tạo dungeon theo loại thế giới
  dungeon = generateDungeon(region.type || "cave", 25, 18, worldType);
  playerPos = { ...dungeon.playerStart };

  setupDungeonCanvas();
  setupDungeonControls();
  startDungeonLoop();
  setupDungeonHUD();

  // --- QUOTA: kiểm tra và tiêu cho phòng đầu tiên ---
  const worldKey = selectedWorld ? (selectedWorld.id || selectedWorld.type) : worldType;
  ensureWorldQuota(worldKey, worldType);

  const consumed = consumeRoomQuota(worldKey);
  dungeon._canSpawnChildren = consumed;

  console.log("Đã vào dungeon:", region.type, "thuộc thế giới:", worldType);
  updateMapButtons && updateMapButtons();

  // Mini-map setup
  initRoomMiniMap();
  markRoomVisited(dungeon);
  drawRoomMiniMap();
  setupMiniMapToggle();

  // 🚫 Khóa các nút điều hướng khi đang trong dungeon
  const navBtns = ["btnHome", "btnInv", "btnSkills", "btnMap"];
  navBtns.forEach(id => {
    const el = document.getElementById(id);
    if (el) {
      el.disabled = true;
      el.style.opacity = "0.4";
      el.style.pointerEvents = "none";
      el.style.filter = "grayscale(0.6)";
      el.style.transition = "all 0.3s ease";
    }
  });
}

// --- Thoát dungeon ---
function exitDungeon() {
  console.log("🚪 Thoát dungeon...");
window.currentFloor = 1; // reset tầng
resetAllQuotas(); // reset quota tất cả thế giới
console.log("🔁 Reset toàn bộ room quota khi thoát dungeon");

  // 1️⃣ Chuyển trạng thái map
  currentMapLevel = "region";

  // 2️⃣ Dừng vòng lặp dungeon
  stopDungeonLoop();

  // 3️⃣ Xóa listener & D-pad
  document.removeEventListener("keydown", handleDungeonKey);
  removeDungeonControls();
  removeDungeonHUD();

  // 4️⃣ Xóa canvas dungeon khỏi giao diện (nếu có)
  const c = document.getElementById("worldMapCanvas");
  if (c) {
    const ctx = c.getContext("2d");
    ctx && ctx.clearRect(0, 0, c.width, c.height);
  }

  // 5️⃣ Làm sạch dữ liệu tạm dungeon (nếu cần)
  window.dungeon = null;
  window.dungeonGraph = {};
if (window.lastWorldContext) {
  window.currentWorldType = window.lastWorldContext.type;
  if (!selectedWorld) {
    selectedWorld = {
      id: window.lastWorldContext.worldId,
      type: window.lastWorldContext.type,
      name: window.lastWorldContext.worldName
    };
  }
}

  // 6️⃣ Cập nhật lại nút giao diện
  if (typeof updateMapButtons === "function") updateMapButtons();

  // 7️⃣ Gọi lại hàm hiển thị bản đồ region (nếu có)
  if (typeof showRegionMap === "function") {
    showRegionMap();
  } else {
    console.warn("⚠️ Không tìm thấy hàm showRegionMap() để hiển thị lại bản đồ vùng.");
  }

  // 8️⃣ Dọn canvas mini-map
  const mm = document.getElementById("roomMiniMap");
  if (mm) mm.remove();
  const toggleBtn = document.getElementById("toggleMiniMapBtn");
  if (toggleBtn) toggleBtn.remove();

  // 9️⃣ 🔹 Reset dữ liệu mini-map (rất quan trọng!)
  roomMapVisited = {};
  roomMapPositions = {};
  roomMapCanvas = null;
  window.roomMapCenter = null;

  // ✅ Mở khóa lại nút điều hướng sau khi thoát dungeon
  const navBtns = ["btnHome", "btnInv", "btnSkills", "btnMap"];
  navBtns.forEach(id => {
    const el = document.getElementById(id);
    if (el) {
      el.disabled = false;
      el.style.opacity = "1";
      el.style.pointerEvents = "auto";
      el.style.filter = "none";
    }
  });

  console.log("🧹 Đã xoá dữ liệu mini-map phòng sau khi thoát dungeon");
  try {
    resizeCanvas?.();  // đảm bảo kích thước canvas đúng
    startLoop?.();     // khởi động lại vòng lặp chính
    console.log("🌍 Quay lại bản đồ region thành công!");
  } catch (err) {
    console.error("❌ Lỗi khi khởi động lại region loop:", err);
  }
}

// === Sinh dungeon (phòng + hành lang + cửa) ===
function generateDungeon(theme = "cave", w = 25, h = 18, worldType = "tieu") {
  // 🏷️ Lưu chủ đề gốc toàn cục (để các tầng sau giữ nguyên)
  window.dungeonBaseTheme = window.dungeonBaseTheme || theme;

  const map = Array.from({ length: h }, () => Array(w).fill(1)); // 1 = tường
  const trait = DUNGEON_TRAITS[theme] || {}; // đặc điểm theo vùng

  // --- thiết lập số lượng phòng theo loại thế giới ---
  let roomCount;
  if (worldType === "dai") roomCount = 10;
  else if (worldType === "trung") roomCount = 7;
  else roomCount = 4; // tiểu

  // --- tạo phòng ---
  for (let i = 0; i < roomCount; i++) {
    const rw = 3 + Math.floor(Math.random() * 4);
    const rh = 3 + Math.floor(Math.random() * 3);
    const rx = 1 + Math.floor(Math.random() * (w - rw - 2));
    const ry = 1 + Math.floor(Math.random() * (h - rh - 2));
    for (let y = ry; y < ry + rh; y++) {
      for (let x = rx; x < rx + rw; x++) {
        map[y][x] = 0;
      }
    }
  }

  // --- hành lang ngẫu nhiên ---
  for (let i = 0; i < roomCount - 1; i++) {
    const x1 = 2 + Math.floor(Math.random() * (w - 4));
    const y1 = 2 + Math.floor(Math.random() * (h - 4));
    const x2 = 2 + Math.floor(Math.random() * (w - 4));
    const y2 = 2 + Math.floor(Math.random() * (h - 4));
    for (let x = Math.min(x1, x2); x <= Math.max(x1, x2); x++) map[y1][x] = 0;
    for (let y = Math.min(y1, y2); y <= Math.max(y1, y2); y++) map[y][x2] = 0;
  }

  // --- tường ngoài ---
  for (let x = 0; x < w; x++) { map[0][x] = 1; map[h - 1][x] = 1; }
  for (let y = 0; y < h; y++) { map[y][0] = 1; map[y][w - 1] = 1; }

  // --- spawn rương & quái ---
  const chestChance = trait.chestChance ?? 0.1;
  const enemyChance = trait.enemyChance ?? 0.15;

  const floorCells = [];
  for (let y = 1; y < h - 1; y++)
    for (let x = 1; x < w - 1; x++)
      if (map[y][x] === 0) floorCells.push([x, y]);

  for (const [x, y] of floorCells) {
    const rnd = Math.random();
    if (rnd < chestChance) map[y][x] = "C";
    else if (rnd < chestChance + enemyChance) map[y][x] = "E";
    else if (Math.random() < 0.02) map[y][x] = "?";
  }

  // --- hiệu ứng môi trường (tùy theme) ---
  if (trait.lavaChance) for (let i = 0; i < w * h * trait.lavaChance; i++) {
    const x = Math.floor(Math.random() * w);
    const y = Math.floor(Math.random() * h);
    if (map[y]?.[x] === 0) map[y][x] = "🔥";
  }
  if (trait.slippery) for (let i = 0; i < w * h * 0.02; i++) {
    const x = Math.floor(Math.random() * w);
    const y = Math.floor(Math.random() * h);
    if (map[y]?.[x] === 0) map[y][x] = "❄️";
  }
  if (trait.poisonous) for (let i = 0; i < w * h * 0.02; i++) {
    const x = Math.floor(Math.random() * w);
    const y = Math.floor(Math.random() * h);
    if (map[y]?.[x] === 0) map[y][x] = "☠️";
  }
  if (trait.trapChance) for (let i = 0; i < w * h * trait.trapChance; i++) {
    const x = Math.floor(Math.random() * w);
    const y = Math.floor(Math.random() * h);
    if (map[y]?.[x] === 0) map[y][x] = "🧨";
  }
  if (trait.pitChance) for (let i = 0; i < w * h * trait.pitChance; i++) {
    const x = Math.floor(Math.random() * w);
    const y = Math.floor(Math.random() * h);
    if (map[y]?.[x] === 0) map[y][x] = "🕳️";
  }
  if (["swamp", "ruins", "deepsea"].includes(theme)) {
    const fogDensity = theme === "swamp" ? 0.05 : 0.02;
    for (let i = 0; i < w * h * fogDensity; i++) {
      const x = Math.floor(Math.random() * w);
      const y = Math.floor(Math.random() * h);
      if (map[y]?.[x] === 0) map[y][x] = "🌫️";
    }
  }

  // 🌳🌋🪨 Thêm tường đặc trưng và biến thể địa hình
  const wallSymbol = trait.wallSymbol || null;
  for (let y = 0; y < h; y++) {
    for (let x = 0; x < w; x++) {
      // xen kẽ biểu tượng tường đặc trưng
      if (map[y][x] === 1 && wallSymbol && Math.random() < 0.2) {
        map[y][x] = wallSymbol;
      }

      // dòng dung nham trong vùng núi lửa
      if (theme === "volcano" && map[y][x] === 0 && Math.random() < 0.05)
        map[y][x] = "🔥";

      // đầm lầy / rừng rậm: bãi độc
      if (["swamp", "jungle"].includes(theme) && map[y][x] === 0 && Math.random() < 0.03)
        map[y][x] = "☠️";
    }
  }

  // --- cửa & boss ---
  const doors = [];
  const wallPositions = [];
  for (let x = 1; x < w - 1; x++) {
    if (map[1][x] === 1 && map[2][x] === 0) wallPositions.push({ x, y: 1, openTo: { x, y: 2 } });
    if (map[h - 2][x] === 0 && map[h - 1][x] === 1) wallPositions.push({ x, y: h - 1, openTo: { x, y: h - 2 } });
  }
  for (let y = 1; y < h - 1; y++) {
    if (map[y][1] === 1 && map[y][2] === 0) wallPositions.push({ x: 1, y, openTo: { x: 2, y } });
    if (map[y][w - 2] === 0 && map[y][w - 1] === 1) wallPositions.push({ x: w - 1, y, openTo: { x: w - 2, y } });
  }

  const makeDoor = (obj, special = false) => ({
    id: 'door_' + Math.random().toString(36).slice(2, 9),
    x: obj.x,
    y: obj.y,
    openTo: obj.openTo,
    special,
  });

  if (wallPositions.length) {
    const doorCount = Math.min(3, 1 + Math.floor(Math.random() * 3));
    for (let i = 0; i < doorCount; i++) {
      const pos = wallPositions[Math.floor(Math.random() * wallPositions.length)];
      map[pos.y][pos.x] = "D";
      map[pos.openTo.y][pos.openTo.x] = 0;
      doors.push(makeDoor(pos));
    }
  }

  if (wallPositions.length > 1 && !window.bossRoomCreated) {
    window.bossRoomCreated = true;
    const candidate = wallPositions[Math.floor(Math.random() * wallPositions.length)];
    map[candidate.y][candidate.x] = "G";
    map[candidate.openTo.y][candidate.openTo.x] = 0;
    doors.push(makeDoor(candidate, true));
  }

  // --- hoàn thiện dungeon ---
  const dungeonObj = { map, w, h, theme, doors, trait };

  connectIsolatedAreas(dungeonObj);
  dungeonObj.playerStart = findSafeSpawn(dungeonObj);

  // 🌈 Hiệu ứng nền riêng theo theme
  dungeonObj.visual = {
    background: (() => {
      switch (theme) {
        case "desert": return "#d4b483";
        case "ice": return "#b5d8f0";
        case "jungle": return "#2e7d32";
        case "volcano": return "#2b0d0d";
        case "cave": return "#1a1a1a";
        case "swamp": return "#3e4035";
        case "ruins": return "#6b6659";
        case "deepsea": return "#0b1d3a";
        case "mountain": return "#5b5b5b";
        case "wasteland": return "#5a4b3f";
        default: return "#222";
      }
    })(),
    lightColor: (() => {
      switch (theme) {
        case "desert": return "rgba(255,240,180,0.4)";
        case "ice": return "rgba(180,220,255,0.3)";
        case "jungle": return "rgba(80,255,120,0.2)";
        case "volcano": return "rgba(255,80,0,0.3)";
        case "swamp": return "rgba(100,160,80,0.2)";
        default: return "rgba(255,255,255,0.2)";
      }
    })()
  };

  return dungeonObj;
}

function findClosestWallToPlayer(dungeon) {
  const { map, w, h } = dungeon;
  const px = Math.floor(dungeon.playerStart.x);
  const py = Math.floor(dungeon.playerStart.y);
  const dirs = [
    { x: 0, y: -1 },
    { x: 0, y: 1 },
    { x: -1, y: 0 },
    { x: 1, y: 0 }
  ];
  for (const d of dirs) {
    const tx = px + d.x;
    const ty = py + d.y;
    if (map[ty] && map[ty][tx] === 1) {
      return { x: tx, y: ty, openTo: { x: px, y: py } };
    }
  }
  // fallback: chọn tường ngẫu nhiên
  for (let y = 1; y < h - 1; y++) {
    for (let x = 1; x < w - 1; x++) {
      if (map[y][x] === 1 && map[y + 1]?.[x] === 0) return { x, y, openTo: { x, y: y + 1 } };
    }
  }
  return null;
}
// === Phòng Boss riêng biệt ===
function generateBossRoom() {
  const w = 25, h = 18;
  const map = Array.from({ length: h }, () => Array(w).fill(1));

  for (let y = 2; y < h - 2; y++)
    for (let x = 2; x < w - 2; x++)
      map[y][x] = 0;

  const centerX = Math.floor(w / 2);
  const centerY = Math.floor(h / 2);

  map[centerY][centerX] = "👹";

  return {
    map, w, h,
    doors: [],
    theme: "boss",
    playerStart: { x: centerX, y: centerY + 3 },
    isBossRoom: true,
    visual: {
      background: "#200000",
      lightColor: "rgba(255, 80, 0, 0.35)",
      gradient: ["#220000", "#5a0000", "#9c1a00"],
      particle: "🔥"
    }
  };
}


// --- nối các vùng tách rời bằng đường hầm ---
function connectIsolatedAreas(dungeonObj) {
  const { w, h, map } = dungeonObj;
  const visited = Array.from({ length: h }, () => Array(w).fill(false));
  const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];

  function floodFill(sx, sy) {
    const stack = [[sx, sy]];
    const cells = [];
    while (stack.length) {
      const [cx, cy] = stack.pop();
      if (cx < 0 || cy < 0 || cx >= w || cy >= h) continue;
      if (visited[cy][cx]) continue;
      if (map[cy][cx] !== 0) continue;
      visited[cy][cx] = true;
      cells.push([cx, cy]);
      for (const [dx, dy] of dirs) stack.push([cx + dx, cy + dy]);
    }
    return cells;
  }

  const regions = [];
  for (let y = 0; y < h; y++) for (let x = 0; x < w; x++) if (!visited[y][x] && map[y][x] === 0) regions.push(floodFill(x, y));
  if (regions.length <= 1) return;

  for (let i = 1; i < regions.length; i++) {
    const rA = regions[0][Math.floor(Math.random() * regions[0].length)];
    const rB = regions[i][Math.floor(Math.random() * regions[i].length)];
    let [cx, cy] = rA, [x2, y2] = rB;
    while (cx !== x2 || cy !== y2) {
      map[cy][cx] = 0;
      const dx = Math.sign(x2 - cx);
      const dy = Math.sign(y2 - cy);
      if (Math.random() < 0.6) { if (Math.abs(x2 - cx) > 0) cx += dx; else cy += dy; }
      else { if (Math.abs(y2 - cy) > 0) cy += dy; else cx += dx; }
    }
  }
}

// === Tìm spawn an toàn (ô 0) ===
function findSafeSpawn(dungeonObj) {
  const { map, w, h } = dungeonObj;
  for (let tries = 0; tries < 1000; tries++) {
    const x = Math.floor(Math.random() * (w - 2)) + 1;
    const y = Math.floor(Math.random() * (h - 2)) + 1;
    if (map[y][x] === 0) return { x, y };
  }
  return { x: 1, y: 1 };
}

// === Setup canvas & resize handler ===
function setupDungeonCanvas() {
  const c = document.getElementById("worldMapCanvas");
  if (!c) {
    console.warn("Canvas #worldMapCanvas không tồn tại");
    return;
  }
  // canvas full viewport
  c.style.position = "fixed";
  c.style.left = "0";
  c.style.top = "0";
  c.style.width = "100vw";
  c.style.height = "100vh";
  c.width = window.innerWidth;
  c.height = window.innerHeight;
  c.style.zIndex = "1";

  dungeonCtx = c.getContext("2d");
  dungeonCtx.imageSmoothingEnabled = false;

  window.addEventListener("resize", onDungeonResize);
  document.addEventListener("keydown", handleDungeonKey);
}

// ==================== PATCH HUD + POPUP NHÂN VẬT ====================

// ==================== HUD + Popup nâng cao (ghi đè các hàm cũ) ====================

// REMOVE HUD (gọi khi thoát dungeon / backToWorld / backToCosmos)
function removeDungeonHUD() {
  // close popup nếu mở
  if (charPopup) {
    // trả các node đã move về vị trí cũ
    try { toggleCharacterPopup(); } catch(e){}
  }
  const el = document.getElementById("dungeonHUD");
  if (el) el.remove();

  // cũng đảm bảo mini-map toggle bị xóa (tuỳ ý)
  const tbtn = document.getElementById("toggleMiniMapBtn");
  if (tbtn) tbtn.remove();
}

window.updateTinyBars = safeUpdateBars;
// SETUP HUD: avatar + 3 thanh nhỏ nằm ngang bên phải avatar
function setupDungeonHUD() {
  removeDungeonHUD(); // sạch trước khi tạo

  const hud = document.createElement("div");
  hud.id = "dungeonHUD";
  Object.assign(hud.style, {
    position: "fixed",
    left: "12px",
    zIndex: "10001",
    pointerEvents: "auto",
    display: "flex",
    alignItems: "center",
    gap: "10px"
  });

  // đặt HUD dưới #mapControls nếu có (tránh che 'Thoát Dungeon')
  const mapCtrl = document.getElementById("mapControls");
  if (mapCtrl) {
    const rect = mapCtrl.getBoundingClientRect();
    hud.style.top = (rect.bottom && rect.bottom > 0 ? rect.bottom + 8 : 10) + "px";
  } else {
    hud.style.top = "10px";
  }

  // Avatar button
  const charBtn = document.createElement("button");
  charBtn.id = "dungeonCharBtn";
  charBtn.innerText = "👤";
  Object.assign(charBtn.style, {
    width: "44px", height: "44px", fontSize: "22px",
    borderRadius: "10px", border: "1px solid rgba(255,255,255,0.18)",
    background: "rgba(0,0,0,0.36)", color: "#fff", cursor: "pointer",
    display: "flex", alignItems: "center", justifyContent: "center"
  });
  charBtn.onclick = toggleCharacterPopup;
  hud.appendChild(charBtn);

  // small bars container (to the right of avatar)
  const bars = document.createElement("div");
  bars.id = "dungeonSmallBars";
  Object.assign(bars.style, {
    display: "flex",
    flexDirection: "column",
    gap: "6px",
    width: "160px",
    padding: "8px",
    borderRadius: "8px",
    background: "rgba(0,0,0,0.36)",
    border: "1px solid rgba(255,255,255,0.04)"
  });

  // each bar: just a colored fill (no numbers)
  function mkBar(id, color) {
    const wrap = document.createElement("div");
    wrap.style.width = "100%";
    wrap.style.height = "8px";
    wrap.style.background = "#2a2a2a";
    wrap.style.borderRadius = "6px";
    wrap.style.overflow = "hidden";
    const fill = document.createElement("div");
    fill.id = id;
    fill.style.height = "100%";
    fill.style.width = "0%";
    fill.style.background = color;
    wrap.appendChild(fill);
    return wrap;
  }

  bars.appendChild(mkBar("hpBarTiny", "#ff6161"));
  bars.appendChild(mkBar("mpBarTiny", "#5fb0ff"));
  bars.appendChild(mkBar("stBarTiny", "#79ffb5"));

  hud.appendChild(bars);
  document.body.appendChild(hud);

  // Tạo nút mini-map nhưng ẩn (setupMiniMapToggle() có thể được gọi sau)
  setupMiniMapToggle && setupMiniMapToggle();

  // cập nhật giá trị nhỏ ngay
  safeUpdateBars();
}

// SAFE update: cập nhật cả thanh nhỏ từ các nguồn chính
function safeUpdateBars(){
  // gọi updateHeader/updateBars nếu có (để cập nhật các id chính trong #view-start)
  try {
    if (typeof updateHeader === "function") updateHeader();
    if (typeof updateBars === "function") updateBars();
  } catch(e){ /* ignore */ }

  // if primary (view-start) bars exist, copy values -> tiny bars
  const copyFromPrimary = function() {
    const pHpBar = document.getElementById("hpBar");
    const pMpBar = document.getElementById("mpBar");
    const pStBar = document.getElementById("staminaBar");

    const tHp = document.getElementById("hpBarTiny");
    const tMp = document.getElementById("mpBarTiny");
    const tSt = document.getElementById("stBarTiny");

    if (pHpBar && tHp) tHp.style.width = pHpBar.style.width || "0%";
    if (pMpBar && tMp) tMp.style.width = pMpBar.style.width || "0%";
    if (pStBar && tSt) tSt.style.width = pStBar.style.width || "0%";

    // also copy text if you want tiny text (we skip per request)
  };

  copyFromPrimary();

  // fallback: nếu chưa có primary hoặc player stats thay đổi, compute from player
  if ((!document.getElementById("hpBar") || !document.getElementById("mpBar")) && typeof player === "object" && player.stats) {
    const hpNow = Number(player.stats.HP) || 0;
    const hpMax = calcFinalStat(player.stats.MaxHP, "MaxHP") || 1;
    const mpNow = Number(player.stats.Mana) || 0;
    const mpMax = calcFinalStat(player.stats.MaxMana, "MaxMana") || 1;
    const stNow = Number(player.stats.SucBen) || 0;
    const stMax = calcFinalStat(player.stats.MaxSucBen, "MaxSucBen") || 1;

    const per = (a,b)=> Math.max(0,Math.min(100,Math.round((a/b)*100))) + "%";

    const tHp = document.getElementById("hpBarTiny");
    const tMp = document.getElementById("mpBarTiny");
    const tSt = document.getElementById("stBarTiny");
    if (tHp) tHp.style.width = per(hpNow,hpMax);
    if (tMp) tMp.style.width = per(mpNow,mpMax);
    if (tSt) tSt.style.width = per(stNow,stMax);
  }
}

// ==================== Toggle Character POPUP (move panels for realtime) ====================

let charPopup = null;
let movedNodes = [];

// === Mở / đóng giao diện nhân vật ===
function toggleCharacterPopup() {
  // --- Nếu đang mở thì đóng ---
  if (charPopup) {
    // Trả lại các node về vị trí gốc
    for (let info of movedNodes) {
      try {
        if (info.originalNextSibling && info.originalNextSibling.parentNode === info.originalParent) {
          info.originalParent.insertBefore(info.node, info.originalNextSibling);
        } else {
          info.originalParent.appendChild(info.node);
        }
      } catch (e) {
        try { info.originalParent.appendChild(info.node); } catch(e){}
      }
    }
    movedNodes = [];

    if (charPopup) charPopup.remove();
    charPopup = null;

    const overlay = document.getElementById("charOverlay");
    if (overlay) overlay.remove();

    // 🧹 Dọn toàn bộ popup/phụ trợ khi đóng giao diện 👤
    const extraPopups = [
      "equipTooltip",
      "equipManagePopup",
      "itemDetailPopup"
    ];
    for (const id of extraPopups) {
      const el = document.getElementById(id);
      if (el) el.remove();
    }

    const dc = document.getElementById("dungeonControls");
    if (dc) { dc.style.visibility = ""; dc.style.pointerEvents = ""; }

    const statDetail = document.getElementById("statDetail");
    if (statDetail) statDetail.style.zIndex = "15000";
    return;
  }

  // --- Khi mở ---
  const dc = document.getElementById("dungeonControls");
  if (dc) { dc.style.visibility = "hidden"; dc.style.pointerEvents = "none"; }

  // Overlay nền mờ
  const overlay = document.createElement("div");
  overlay.id = "charOverlay";
  Object.assign(overlay.style, {
    position: "fixed", inset: "0",
    background: "rgba(0,0,0,0.5)",
    backdropFilter: "blur(4px)",
    zIndex: "11999"
  });
  overlay.onclick = (e) => {
    if (e.target === overlay) toggleCharacterPopup();
  };
  document.body.appendChild(overlay);

  // Popup chính
  charPopup = document.createElement("div");
  charPopup.id = "charPopup";
  Object.assign(charPopup.style, {
    position: "fixed",
    top: "50%", left: "50%",
    transform: "translate(-50%,-50%)",
    width: "min(92%, 440px)",
    maxHeight: "82vh",
    background: "rgba(25,25,25,0.9)",
    color: "#fff",
    border: "1px solid rgba(255,255,255,0.12)",
    borderRadius: "12px",
    padding: "12px",
    overflowY: "auto",
    zIndex: "12000",
    boxShadow: "0 0 18px rgba(0,0,0,0.45)"
  });

  // Nút đóng
  const close = document.createElement("button");
  close.innerText = "✖";
  Object.assign(close.style, {
    position: "absolute", top: "8px", right: "8px",
    background: "rgba(255,255,255,0.08)",
    border: "none", color: "#fff",
    borderRadius: "6px", cursor: "pointer"
  });
  close.onclick = () => toggleCharacterPopup();
  charPopup.appendChild(close);

  // === TRANG BỊ ===
  const equipHeader = document.createElement("h3");
  equipHeader.innerText = "Trang bị";
  equipHeader.style.marginTop = "4px";
  equipHeader.style.borderBottom = "1px solid rgba(255,255,255,0.1)";
  equipHeader.style.paddingBottom = "4px";
  charPopup.appendChild(equipHeader);

  // Grid ô trang bị
  const equipGrid = document.createElement("div");
  Object.assign(equipGrid.style, {
    display: "grid",
    gridTemplateColumns: "repeat(5, 1fr)",
    gap: "6px",
    marginBottom: "10px",
    marginTop: "6px"
  });

  const slots = [
    ["head","👑"],["body","🛡️"],["hand","🖐️"],
    ["main-weapon","⚔️"],["sub-weapon","🗡️"],
    ["necklace","📿"],["bracelet1","📿"],["bracelet2","📿"],
    ["ring1","💍"],["ring2","💍"],["ring3","💍"],["ring4","💍"],["ring5","💍"]
  ];

  // === Tạo từng ô trang bị ===
  for (const [slot, icon] of slots) {
    const slotDiv = document.createElement("div");
    slotDiv.className = "equip-slot"; // ⚡ Dùng chung class với giao diện khởi đầu
    slotDiv.dataset.slot = slot;
    Object.assign(slotDiv.style, {
      width: "50px", height: "50px",
      borderRadius: "8px",
      display: "flex", alignItems: "center", justifyContent: "center",
      fontSize: "22px",
      background: "rgba(255,255,255,0.05)",
      cursor: "pointer",
      position: "relative",
      transition: "box-shadow 0.2s, border 0.2s"
    });
    slotDiv.innerText = icon;
    slotDiv.title = slot;

    // --- Click và double click ---
    let lastClickTime = 0;
    slotDiv.addEventListener("click", (e) => {
      e.stopPropagation();
      const now = performance.now();

      // Dọn tooltip trước khi hiển thị mới
      const oldTip = document.getElementById("equipTooltip");
      if (oldTip) oldTip.remove();

      if (now - lastClickTime < 250) {
        // Double click → mở popup chi tiết
        showEquipManagePopup(slot);
      } else {
        // Single click → tooltip thông tin cơ bản
        showEquipTooltip(slot, slotDiv);
      }
      lastClickTime = now;
    });

    equipGrid.appendChild(slotDiv);
  }

  charPopup.appendChild(equipGrid);

  // --- Chèn các panel thông tin ---
  const idsToMove = [
    "char-stats", "panel-status", "panel-buffs", 
    "panel-title", "panel-class", "panel-stats"
  ];
  for (const id of idsToMove) {
    const node = document.getElementById(id);
    if (!node) continue;
    movedNodes.push({
      node: node,
      originalParent: node.parentNode,
      originalNextSibling: node.nextSibling
    });
    charPopup.appendChild(node);
  }

  if (movedNodes.length === 0) {
    const fallback = document.createElement("div");
    fallback.innerHTML = `<h3>Nhân vật</h3><div>Không tìm thấy các panel.</div>`;
    charPopup.appendChild(fallback);
  }

  document.body.appendChild(charPopup);

  // ✅ Sau khi charPopup gắn vào DOM mới render
  requestAnimationFrame(() => {
    renderEquipmentSlots(); // cập nhật trạng thái trang bị
  });

  // Đảm bảo z-index cho chi tiết chỉ số
  const statDetail = document.getElementById("statDetail");
  if (statDetail) statDetail.style.zIndex = "13000";

  safeUpdateBars();
}


window.showItemDetailInPopup = function(itemId) {
  const data = findItemById(itemId);
  const inv = player.inventory.find(x => String(x.id) === String(itemId));

  const old = document.getElementById("itemDetailPopup");
  if (old) old.remove();

  const popup = document.createElement("div");
  popup.id = "itemDetailPopup";
  Object.assign(popup.style, {
    position: "fixed",
    top: "50%", left: "50%",
    transform: "translate(-50%, -50%)",
    background: "rgba(25,25,25,0.97)",
    color: "#fff",
    border: "1px solid rgba(255,255,255,0.2)",
    borderRadius: "12px",
    boxShadow: "0 0 12px rgba(0,0,0,0.5)",
    padding: "12px",
    width: "min(90%, 420px)",
    maxHeight: "80vh",
    overflowY: "auto",
    zIndex: "200000",
    animation: "fadeIn 0.15s ease-out"
  });

  popup.innerHTML = `
    <div style="display:flex;justify-content:space-between;align-items:center;
      border-bottom:1px solid rgba(255,255,255,0.1);padding-bottom:6px;margin-bottom:8px;">
      <strong>Chi tiết vật phẩm</strong>
      <button onclick="document.getElementById('itemDetailPopup').remove()" 
        style="background:none;border:none;color:#fff;font-size:18px;cursor:pointer;">✖</button>
    </div>
    <div>${getItemDetailsHTML(data, inv)}</div>
  `;

  document.body.appendChild(popup);
};


// === Tooltip trang bị ===
function showEquipTooltip(slot, target) {
  const old = document.getElementById("equipTooltip");
  if (old) old.remove();

  const tip = document.createElement("div");
  tip.id = "equipTooltip";
  Object.assign(tip.style, {
    position: "fixed",
    background: "rgba(0,0,0,0.9)",
    color: "#fff",
    border: "1px solid rgba(255,255,255,0.2)",
    borderRadius: "8px",
    padding: "8px 10px",
    fontSize: "13px",
    zIndex: "99999",
    maxWidth: "260px",
    lineHeight: "1.4em",
    pointerEvents: "none",
  });

  const itemId = player?.equipment?.[slot];
  if (!itemId) {
    tip.innerHTML = `<b>${slot}</b><br><i>Chưa trang bị</i>`;
  } else {
    const item = findItemById(itemId);
    if (!item) {
      tip.innerHTML = `<b>${slot}</b><br><i>Không tìm thấy dữ liệu vật phẩm</i>`;
    } else {
      tip.innerHTML = `
        <b>${item.name}</b><br>
        <span style="color:gold;">Phẩm chất: ${item.rarity || "Thường"}</span><br>
        ${item.description || item.desc || "<i>Không có mô tả</i>"}
      `;
    }
  }

  document.body.appendChild(tip);

  // --- Tính vị trí an toàn ---
  const rect = target.getBoundingClientRect();
  const tipW = tip.offsetWidth;
  const tipH = tip.offsetHeight;
  let left = rect.right + 10;
  let top = rect.top;

  if (left + tipW > window.innerWidth) left = rect.left - tipW - 10;
  if (top + tipH > window.innerHeight) top = window.innerHeight - tipH - 8;
  if (top < 8) top = 8;

  tip.style.left = `${left}px`;
  tip.style.top = `${top}px`;

  const hideTip = (e) => {
    if (!tip.contains(e.target)) {
      tip.remove();
      document.removeEventListener("click", hideTip);
    }
  };
  document.addEventListener("click", hideTip);
}

function showEquipManagePopup(slot) {
  const old = document.getElementById("equipManagePopup");
  if (old) old.remove();

  // --- Khung chính ---
  const popup = document.createElement("div");
  popup.id = "equipManagePopup";
  Object.assign(popup.style, {
    position: "fixed",
    top: "50%", left: "50%",
    transform: "translate(-50%,-50%)",
    background: "rgba(25,25,25,0.97)",
    color: "#fff",
    border: "1px solid rgba(255,255,255,0.2)",
    borderRadius: "12px",
    boxShadow: "0 0 12px rgba(0,0,0,0.5)",
    width: "min(90%,420px)",
    maxHeight: "82vh",
    zIndex: "100000",
    display: "flex",
    flexDirection: "column",
    overflow: "hidden"
  });

  // --- Tiêu đề & nút đóng ---
  popup.innerHTML = `
    <div style="padding:8px 12px;border-bottom:1px solid rgba(255,255,255,0.1);
      display:flex;justify-content:space-between;align-items:center;">
      <strong>Trang bị: ${slot}</strong>
      <button style="background:none;border:none;color:#fff;font-size:18px;cursor:pointer;"
        onclick="document.getElementById('equipManagePopup').remove()">✖</button>
    </div>
  `;

  // --- Phần trên: Trang bị hiện tại ---
  const currentContainer = document.createElement("div");
  Object.assign(currentContainer.style, {
    padding: "10px",
    flex: "0 0 auto",
    borderBottom: "1px solid rgba(255,255,255,0.08)",
  });

  const currentId = player?.equipment?.[slot];
  const currentItem = currentId ? findItemById(currentId) : null;

  if (currentItem) {
    currentContainer.innerHTML = `
      <div style="font-size:14px;margin-bottom:4px;">
        ${getItemIcon(currentItem)} 
        <b style="cursor:pointer;" onclick="showItemDetailInPopup('${currentItem.id}')">
          ${currentItem.name}
        </b> 
        <span style="color:gold;">(${currentItem.rarity || "Thường"})</span>
      </div>
      <div style="font-size:12px;color:rgba(255,255,255,0.85);margin-bottom:6px;">
        ${currentItem.description || currentItem.desc || ""}
      </div>
      <button style="padding:4px 8px;border-radius:6px;border:none;background:#c0392b;color:#fff;cursor:pointer;"
        onclick="unequipItem('${slot}');document.getElementById('equipManagePopup').remove();updateEquipSlotBorders();">
        Tháo ra
      </button>
    `;
  } else {
    currentContainer.innerHTML = `
      <div style="font-size:13px;color:rgba(255,255,255,0.6)">Hiện chưa trang bị</div>
    `;
  }

  popup.appendChild(currentContainer);

  // --- Phần dưới: Các trang bị khác ---
  const otherContainer = document.createElement("div");
  Object.assign(otherContainer.style, {
    flex: "1 1 auto",
    overflowY: "auto",
    padding: "8px 10px",
  });

  const equips = player.inventory.filter(it => {
    const d = findItemById(it.id);
    if (!d) return false;
    if (d.slot === "necklace" && slot === "necklace") return true;
    if (d.type === "ring" && slot.startsWith("ring")) return true;
    if (d.type === "armor" && slot === "body") return true;
    if (d.type === "weapon" && (slot === "main-weapon" || slot === "sub-weapon")) return true;
    return false;
  });

  if (equips.length === 0) {
    otherContainer.innerHTML = `<p style="font-size:13px;color:rgba(255,255,255,0.6);text-align:center;">
      Không có trang bị phù hợp
    </p>`;
  } else {
    equips.forEach(inv => {
      const d = findItemById(inv.id);
      const itemHTML = `
        <div style="margin-bottom:8px;padding:6px;border-radius:8px;
          background:rgba(255,255,255,0.05);">
          <div style="font-size:13px;display:flex;justify-content:space-between;align-items:center;">
            <div style="cursor:pointer;" onclick="showItemDetailInPopup('${inv.id}')">
              ${getItemIcon(d)} <b>${d.name}</b>
              <span style="color:gold;">(${d.rarity || "Thường"})</span>
            </div>
            <button style="padding:3px 6px;border:none;border-radius:6px;
              background:#2980b9;color:#fff;cursor:pointer;font-size:12px;"
              onclick="equipItem('${inv.id}','${slot}');document.getElementById('equipManagePopup').remove();updateEquipSlotBorders();">
              Trang bị
            </button>
          </div>
          <div style="font-size:12px;color:rgba(255,255,255,0.75);margin-top:3px;">
            ${d.description || d.desc || ""}
          </div>
        </div>
      `;
      otherContainer.insertAdjacentHTML("beforeend", itemHTML);
    });
  }

  popup.appendChild(otherContainer);
  document.body.appendChild(popup);
}

function showItemDetailInPopup(itemId) {
  const data = findItemById(itemId);
  const inv = player.inventory.find(x => String(x.id) === String(itemId));

  // Xóa popup cũ (nếu có)
  const old = document.getElementById("itemDetailPopup");
  if (old) old.remove();

  // Tạo popup mới
  const popup = document.createElement("div");
  popup.id = "itemDetailPopup";
  Object.assign(popup.style, {
    position: "fixed",
    top: "50%", left: "50%",
    transform: "translate(-50%, -50%)",
    background: "rgba(25,25,25,0.97)",
    color: "#fff",
    border: "1px solid rgba(255,255,255,0.2)",
    borderRadius: "12px",
    boxShadow: "0 0 12px rgba(0,0,0,0.5)",
    padding: "12px",
    width: "min(90%, 420px)",
    maxHeight: "80vh",
    overflowY: "auto",
    zIndex: "200000"
  });

  popup.innerHTML = `
    <div style="display:flex;justify-content:space-between;align-items:center;
      border-bottom:1px solid rgba(255,255,255,0.1);padding-bottom:6px;margin-bottom:8px;">
      <strong>Chi tiết vật phẩm</strong>
      <button onclick="document.getElementById('itemDetailPopup').remove()" 
        style="background:none;border:none;color:#fff;font-size:18px;cursor:pointer;">✖</button>
    </div>
    <div>${getItemDetailsHTML(data, inv)}</div>
  `;

  document.body.appendChild(popup);
}


// --- Gọi mỗi khi vào dungeon ---
const oldEnterDungeon = enterDungeon;
enterDungeon = function(region){
  oldEnterDungeon(region);
  setupDungeonHUD();
};

// --- Cập nhật mỗi frame ---
const oldDrawDungeon = drawDungeon;
drawDungeon = function(ctx){
  oldDrawDungeon(ctx);
  safeUpdateBars();
};

function onDungeonResize() {
  const c = document.getElementById("worldMapCanvas");
  if (!c) return;
  c.width = window.innerWidth;
  c.height = window.innerHeight;
}

// === Controls (D-Pad) với style chắc chắn hiển thị ---
function setupDungeonControls() {
  removeDungeonControls();
  const controls = document.createElement("div");
  controls.id = "dungeonControls";

  // style cố định + z-index cao để nổi trên canvas
  Object.assign(controls.style, {
    position: "fixed",
    bottom: "26px",
    left: "50%",
    transform: "translateX(-50%)",
    display: "grid",
    gridTemplateColumns: "repeat(3, 64px)",
    gridTemplateRows: "repeat(3, 64px)",
    gap: "8px",
    placeItems: "center",
    zIndex: "99999",
    pointerEvents: "auto",
    touchAction: "none",
    userSelect: "none",
    WebkitUserSelect: "none",
    background: "transparent",
  });

  controls.innerHTML = `
    <div></div>
    <button class="dpad" data-dir="up">⬆️</button>
    <div></div>
    <button class="dpad" data-dir="left">⬅️</button>
    <button class="dpad" data-dir="down">⬇️</button>
    <button class="dpad" data-dir="right">➡️</button>
    <div></div>
  `;

  document.body.appendChild(controls);

  controls.querySelectorAll(".dpad").forEach(btn => {
    Object.assign(btn.style, {
      width: "64px", height: "64px", fontSize: "26px",
      borderRadius: "12px", border: "1px solid rgba(255,255,255,0.16)",
      background: "rgba(0,0,0,0.28)", color: "#fff", backdropFilter: "blur(6px)"
    });
    btn.onpointerdown = (ev) => {
      ev.preventDefault();
      const d = btn.dataset.dir;
      if (d === "up") movePlayer(0, -1);
      if (d === "down") movePlayer(0, 1);
      if (d === "left") movePlayer(-1, 0);
      if (d === "right") movePlayer(1, 0);
    };
  });
}

function removeDungeonControls() {
  const el = document.getElementById("dungeonControls");
  if (el) el.remove();
}

// thêm dòng này ngay sau:
window.removeDungeonControls = removeDungeonControls;

// === Phím điều khiển ===
function handleDungeonKey(e) {
  if (currentMapLevel !== "dungeon") return;
  const k = e.key.toLowerCase();
  if (k === "arrowup" || k === "w") movePlayer(0, -1);
  if (k === "arrowdown" || k === "s") movePlayer(0, 1);
  if (k === "arrowleft" || k === "a") movePlayer(-1, 0);
  if (k === "arrowright" || k === "d") movePlayer(1, 0);
}

let lastTrapTriggered = null; // để tránh spam bẫy liên tục
let nearestDoor = null; // dùng cho vực

// === HP & Chết ===
function getMaxHP() {
  return calcFinalStat(player.stats.MaxHP || 100, "MaxHP");
}
window.modifyHP = modifyHP;

function modifyHP(amount, reason = "") {
  const maxHP = getMaxHP();
  player.stats.HP = Math.min(maxHP, Math.max(0, (Number(player.stats.HP) || maxHP) + Number(amount)));

  if (reason) console.log(`${reason}! HP hiện tại: ${player.stats.HP}/${maxHP}`);

  // ⚰️ Nếu chết → Thoát dungeon
  if (player.stats.HP <= 0) {
    console.log("💀 HP xuống 0 — thoát khỏi Dungeon!");
    player.stats.HP = Math.max(1, Math.floor(maxHP * 0.2));
    safeUpdateBars && safeUpdateBars();
    try { exitDungeon(); } catch (e) { console.warn("⚠️ exitDungeon() chưa định nghĩa:", e); }
  }

  safeUpdateBars && safeUpdateBars();
}


function canMove() {
  const curStamina = Number(player.stats.SucBen || 0);
  return curStamina > 0;
}

// === Xử lý hiệu ứng môi trường ===
function handleEnvironmentEffect(x, y, tile) {
  const map = dungeon.map;
  const now = Date.now();

  player._lastEffectPos = player._lastEffectPos || {};

  switch (tile) {
    // =================🔥 LAVA=================
    case "🔥": {
      modifyHP(-6, "🔥 Bỏng khi chạm");

      const last = player._lastEffectPos.burn || {};
      const sameTile = last.x === x && last.y === y;

      // Nếu vừa bước lên ô mới 🔥
      if (!sameTile) {
        applyEffect("burn", {
          power: 1.2,
          duration: 4000,
          tickInterval: 1000,
          maxStacks: 8
        });
        const total = activeEffects.filter(e => e.type === "burn").length;
        console.log(`🔥 +stack (bước lên) → tổng ${total}`);
        player._lastEffectPos.burn = { x, y, enteredAt: now, lastTickAt: now };
      } 
      // Nếu đang đứng yên trên cùng ô 🔥
      else if (now - last.lastTickAt > 1200) {
        applyEffect("burn", {
          power: 0.6,
          duration: 4000,
          tickInterval: 1000,
          maxStacks: 8
        });
        const total = activeEffects.filter(e => e.type === "burn").length;
        console.log(`🔥 +stack (đứng yên) → tổng ${total}`);
        player._lastEffectPos.burn.lastTickAt = now;
      }
      break;
    }

    // =================☠️ POISON=================
    case "☠️": {
      modifyHP(-2, "☠️ Trúng độc nhẹ");

      const last = player._lastEffectPos.poison || {};
      const sameTile = last.x === x && last.y === y;

      // Nếu mới bước vào vùng độc
      if (!sameTile) {
        applyEffect("poison", {
          power: 1.0,
          duration: 6000,
          tickInterval: 1500,
          maxStacks: 10
        });
        const total = activeEffects.filter(e => e.type === "poison").length;
        console.log(`☠️ +stack (bước lên) → tổng ${total}`);
        player._lastEffectPos.poison = { x, y, enteredAt: now, lastTickAt: now };
      }
      // Nếu vẫn đứng yên trên vùng độc
      else if (now - last.lastTickAt > 1800) {
        applyEffect("poison", {
          power: 0.5,
          duration: 6000,
          tickInterval: 1500,
          maxStacks: 10
        });
        const total = activeEffects.filter(e => e.type === "poison").length;
        console.log(`☠️ +stack (đứng yên) → tổng ${total}`);
        player._lastEffectPos.poison.lastTickAt = now;
      }
      break;
    }

    // =================❄️ ICE=================
    case "❄️": {
      applyEffect("freeze", { power: 0.8, duration: 4000, tickInterval: 1000, maxStacks: 4 });
      console.log("❄️ Hiệu ứng đóng băng kích hoạt!");

      // Logic trượt
      let dx = x - (playerPos.prevX ?? x);
      let dy = y - (playerPos.prevY ?? y);
      if (!dx && !dy) {
        const dirs = [
          { x: 1, y: 0 }, { x: -1, y: 0 },
          { x: 0, y: 1 }, { x: 0, y: -1 }
        ];
        const rnd = dirs[Math.floor(Math.random() * dirs.length)];
        dx = rnd.x; dy = rnd.y;
      }
      let slideCount = 0;
      while (slideCount < 3) {
        const nextX = x + dx, nextY = y + dy;
        if (map[nextY]?.[nextX] === 0) {
          playerPos = { x: nextX, y: nextY };
          x = nextX; y = nextY; slideCount++;
        } else break;
      }
      break;
    }

    // =================🧨 TRAP=================
    case "🧨":
      if (lastTrapTriggered && Date.now() - lastTrapTriggered < 1000) return;
      lastTrapTriggered = Date.now();
      modifyHP(-10, "🧨 Bị bẫy kích hoạt");
      map[y][x] = 0;
      break;

    // =================🕳️ HOLE=================
    case "🕳️":
      if (Math.random() < 0.15) {
        console.log("🕳️ Rơi sâu... Sang tầng tiếp theo!");
        nextDungeonFloor();
      } else {
        const emptyTiles = [];
        for (let yy = 1; yy < map.length - 1; yy++) {
          for (let xx = 1; xx < map[0].length - 1; xx++) {
            if (map[yy][xx] === 0) emptyTiles.push({ x: xx, y: yy });
          }
        }
        if (emptyTiles.length)
          playerPos = emptyTiles[Math.floor(Math.random() * emptyTiles.length)];
      }
      break;

    // =================🌀 STAIRS=================
    case "🌀":
      nextDungeonFloor();
      return;
  }

  playerPos.prevX = x;
  playerPos.prevY = y;
}

// === HỆ THỐNG SƯƠNG MÙ NÂNG CAO ===
let baseLightRadius = 6; // bán kính ánh sáng gốc
let fogEffectStrength = 0; // cường độ hiệu ứng hiện tại (0 → không, 1 → tối đa)

function checkFogProximity(dungeon) {
  if (!dungeon?.map) return;

  const px = playerPos.x;
  const py = playerPos.y;
  const fogRange = 6; // khoảng cách tối đa bị ảnh hưởng
  let nearestDist = Infinity;

  // tìm tile 🌫️ gần nhất
  for (let y = Math.max(0, py - fogRange); y <= Math.min(dungeon.h - 1, py + fogRange); y++) {
    for (let x = Math.max(0, px - fogRange); x <= Math.min(dungeon.w - 1, px + fogRange); x++) {
      if (dungeon.map[y][x] === "🌫️") {
        const dist = Math.hypot(px - x, py - y);
        if (dist < nearestDist) nearestDist = dist;
      }
    }
  }

  // nếu có tile 🌫️ trong phạm vi
  if (nearestDist <= fogRange) {
    // càng gần thì hiệu ứng càng mạnh
    const intensity = 1 - Math.min(1, nearestDist / fogRange);
    fogEffectStrength = Math.min(1, fogEffectStrength + (intensity - fogEffectStrength) * 0.2);
  } else {
    // không còn sương gần -> giảm dần hiệu ứng
    fogEffectStrength += (0 - fogEffectStrength) * 0.1;
  }

  // áp dụng hiệu ứng lên bán kính ánh sáng
  const fogMultiplier = 0.5 + (1 - fogEffectStrength) * 0.5;
  lightRadius = baseLightRadius * fogMultiplier;

  // (tuỳ chọn) log debug nhẹ
  // console.log(`🌫️ Near fog: dist=${nearestDist.toFixed(2)} → strength=${fogEffectStrength.toFixed(2)} → light=${lightRadius.toFixed(2)}`);
}


// === Di chuyển & tương tác ===
function movePlayer(dx, dy) {
  if (!dungeon) return;

  if (!canMove()) {
    console.log("⚠️ Sức Bền = 0, không thể di chuyển!");
    return;
  }

  const nx = playerPos.x + dx;
  const ny = playerPos.y + dy;
  if (nx < 0 || ny < 0 || nx >= dungeon.w || ny >= dungeon.h) return;

  const t = dungeon.map[ny][nx];
  if (t === 1) return; // Tường

  // ✅ Tiêu hao stamina mỗi bước
  const stepCost = 1;
  player.stats.SucBen = Math.max(0, (Number(player.stats.SucBen) || 0) - stepCost);
  safeUpdateBars && safeUpdateBars();
  refreshPlayerUI();

  // ✅ Ghi trạng thái di chuyển để hiển thị emoji động
  player.moving = true;
  player.moveDir = { dx, dy };
  clearTimeout(player._moveTimeout);
  player._moveTimeout = setTimeout(() => { player.moving = false; }, 250);

  // ✅ Lưu vị trí cũ
  playerPos.prevX = playerPos.x;
  playerPos.prevY = playerPos.y;
  playerPos = { x: nx, y: ny };

  // ✅ Hiệu ứng môi trường (nếu có)
  handleEnvironmentEffect(nx, ny, t);

  // ====================== RƯƠNG THƯỜNG ======================
if (t === "C") {
  dungeon.map[ny][nx] = 0;

  const floor = window.currentFloor || 1;
  const level = player.stats.Level || 1;
  const items = gameData.items || [];
  const techniques = gameData.techniques || [];

  if (items.length === 0 && techniques.length === 0) {
    console.log("🎁 Rương trống (không có dữ liệu vật phẩm hoặc công pháp)!");
    return;
  }

  // 🧭 Lấy loại thế giới hiện tại
  let worldType = window.currentWorldType || "tieu";
  if (worldType === "tieu") worldType = "tiểu";
  if (worldType === "trung") worldType = "trung";
  if (worldType === "dai")  worldType = "đại";

  // 🎯 Mốc cấp khuyến nghị
  let minLv = 10, maxLv = 39;
  if (worldType === "trung") { minLv = 40; maxLv = 69; }
  if (worldType === "đại") { minLv = 70; maxLv = 99; }

  // --- Độ hiếm cơ bản ---
  let rarityWeights = { 
    "Hoàng": 50, "Huyền": 30, "Địa": 10, "Thiên": 6, "Thần": 3, "Nguyên Sơ": 1 
  };
  if (floor >= 5)  rarityWeights = { "Hoàng": 35, "Huyền": 30, "Địa": 15, "Thiên": 10, "Thần": 7, "Nguyên Sơ": 3 };
  if (floor >= 10) rarityWeights = { "Hoàng": 20, "Huyền": 25, "Địa": 20, "Thiên": 15, "Thần": 12, "Nguyên Sơ": 8 };

  // --- Điều chỉnh theo cấp nhân vật ---
  if (level > maxLv) {
    rarityWeights["Hoàng"] += 30;
    rarityWeights["Huyền"] += 15;
  } else if (level < minLv) {
    rarityWeights["Địa"] += 10;
    rarityWeights["Thiên"] += 10;
    rarityWeights["Thần"] += 5;
    rarityWeights["Nguyên Sơ"] += 5;
  }

  // --- Random độ hiếm ---
  const rarityList = Object.entries(rarityWeights);
  const totalWeight = rarityList.reduce((a, b) => a + b[1], 0);
  let r = Math.random() * totalWeight;
  let chosenRarity = "Hoàng";
  for (const [rar, w] of rarityList) {
    if (r < w) { chosenRarity = rar; break; }
    r -= w;
  }

  // --- Chọn vật phẩm hoặc công pháp ---
  let dropType = "item";
  let selected = null;

  if (Math.random() < 0.1 && techniques.length > 0) {
    // 10% rơi công pháp
    const techPool = techniques.filter(t => (t.rarity || "Hoàng") === chosenRarity);
    selected = techPool.length > 0 
      ? techPool[Math.floor(Math.random() * techPool.length)] 
      : techniques[Math.floor(Math.random() * techniques.length)];
    dropType = "tech";
  }

  if (!selected) {
    const pool = items.filter(it => (it.rarity || "Hoàng") === chosenRarity);
    selected = pool.length > 0 
      ? pool[Math.floor(Math.random() * pool.length)] 
      : items[Math.floor(Math.random() * items.length)];
  }

  if (selected) {
    if (dropType === "item") {
      addItem(selected.id, 1);
      console.log(`🎁 Mở rương và nhận được: ${selected.name} (${selected.rarity || "Hoàng"})`);
    } else {
      player.techniques = player.techniques || [];
      if (!player.techniques.find(t => t.id === selected.id)) {
        player.techniques.push({ id: selected.id, lvl: 1, exp: 0 });
        console.log(`📜 Học được công pháp mới: ${selected.name} (${selected.rarity || "Hoàng"})`);
      } else {
        console.log(`📘 Bạn đã biết công pháp này.`);
      }
    }

    $('modal').style.display = 'flex';
    $('modalCard').innerHTML = `
      <h3>🎁 Nhặt được rương!</h3>
      <p>Bạn nhận được: <b>${selected.name}</b> (${selected.rarity || "Hoàng"})</p>
      <p><i>${selected.description || ''}</i></p>
      <div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>
    `;
  }

  refreshPlayerUI();
  drawDungeon(dungeonCtx);
}
  // ====================== EXP ======================
  else if (t === "E") {
    dungeon.map[ny][nx] = 0;
    const enemy = generateEnemy("normal");
  showCombatUI(enemy);
    refreshPlayerUI();
  }

  // ====================== BOSS ======================
  else if (t === "👹") {
    dungeon.map[ny][nx] = 0;
const enemy = generateEnemy("boss");
  showCombatUI(enemy);
    refreshPlayerUI();
    const cx = nx, cy = ny;
    if (dungeon.map[cy] && dungeon.map[cy][cx + 2] === 0) dungeon.map[cy][cx + 2] = "🎁";
    if (dungeon.map[cy] && dungeon.map[cy][cx - 2] === 0) dungeon.map[cy][cx - 2] = "🌀";
    drawDungeon(dungeonCtx);
  }

  // ====================== RƯƠNG BOSS ======================
else if (t === "🎁") {
  dungeon.map[ny][nx] = 0;
  console.log("🏆 Mở rương Boss!");

  const level = player.stats.Level || 1;
  const items = gameData.items || [];
  const techniques = gameData.techniques || [];

  if (items.length === 0 && techniques.length === 0) {
    console.log("🎁 Rương boss trống (không có dữ liệu vật phẩm hoặc công pháp)!");
    return;
  }

  // 🧭 Lấy loại thế giới
  let worldType = window.currentWorldType || "tieu";
  if (worldType === "tieu") worldType = "tiểu";
  if (worldType === "trung") worldType = "trung";
  if (worldType === "dai")  worldType = "đại";

  // 🎯 Mốc cấp khuyến nghị
  let minLv = 10, maxLv = 39;
  if (worldType === "trung") { minLv = 40; maxLv = 69; }
  if (worldType === "đại")   { minLv = 70; maxLv = 99; }

  // --- Độ hiếm cao hơn ---
  let rarityWeights = { 
    "Hoàng": 15, "Huyền": 25, "Địa": 25, "Thiên": 20, "Thần": 10, "Nguyên Sơ": 5
  };
  if (worldType === "đại") rarityWeights = { 
    "Hoàng": 10, "Huyền": 20, "Địa": 25, "Thiên": 25, "Thần": 12, "Nguyên Sơ": 8
  };

  if (level > maxLv) {
    rarityWeights["Hoàng"] += 10;
    rarityWeights["Huyền"] += 5;
  } else if (level < minLv) {
    rarityWeights["Thiên"] += 10;
    rarityWeights["Thần"] += 5;
    rarityWeights["Nguyên Sơ"] += 5;
  }

  // --- Random độ hiếm ---
  const rarityList = Object.entries(rarityWeights);
  const totalWeight = rarityList.reduce((a, b) => a + b[1], 0);
  let r = Math.random() * totalWeight;
  let chosenRarity = "Hoàng";
  for (const [rar, w] of rarityList) {
    if (r < w) { chosenRarity = rar; break; }
    r -= w;
  }

  // --- Chọn vật phẩm hoặc công pháp ---
  let dropType = "item";
  let selected = null;

  if (Math.random() < 0.25 && techniques.length > 0) {
    // 25% cơ hội rơi công pháp
    const techPool = techniques.filter(t => (t.rarity || "Hoàng") === chosenRarity);
    selected = techPool.length > 0 
      ? techPool[Math.floor(Math.random() * techPool.length)] 
      : techniques[Math.floor(Math.random() * techniques.length)];
    dropType = "tech";
  }

  if (!selected) {
    const pool = items.filter(it => (it.rarity || "Hoàng") === chosenRarity);
    selected = pool.length > 0 
      ? pool[Math.floor(Math.random() * pool.length)] 
      : items[Math.floor(Math.random() * items.length)];
  }

  // --- Nhận thưởng ---
  if (dropType === "item") {
    addItem(selected.id, 1);
    console.log(`🏆 Nhận được vật phẩm hiếm từ Boss: ${selected.name} (${selected.rarity || "Hoàng"})`);
  } else {
    player.techniques = player.techniques || [];
    if (!player.techniques.find(t => t.id === selected.id)) {
      player.techniques.push({ id: selected.id, lvl: 1, exp: 0 });
      console.log(`📜 Học được công pháp từ Boss: ${selected.name} (${selected.rarity || "Hoàng"})`);
    } else {
      console.log(`📘 Bạn đã biết công pháp này.`);
    }
  }

  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>🏆 Rương Boss Mở Ra!</h3>
    <p>Bạn nhận được: <b>${selected.name}</b> (${selected.rarity || "Hoàng"})</p>
    <p><i>${selected.description || ''}</i></p>
    <div style="margin-top:8px"><button onclick="closeModal()">Đóng</button></div>
  `;

  const flash = document.createElement("div");
  flash.innerText = "✨";
  Object.assign(flash.style, {
    position: "fixed", left: "50%", top: "40%",
    fontSize: "80px", transform: "translate(-50%, -50%)",
    animation: "fadeOut 1.5s ease-out forwards"
  });
  document.body.appendChild(flash);
  setTimeout(()=>flash.remove(), 1500);

  refreshPlayerUI();
  drawDungeon(dungeonCtx);
}

  // ====================== SỰ KIỆN "?" ======================
  else if (t === "?") {
    dungeon.map[ny][nx] = 0;
    console.log("❓ Khám phá bí ẩn!");

    const type = dungeon.type || "default";
    const rand = Math.random();

    if (type === "volcano") { 
      if (rand < 0.4) {
        console.log("🔥 Trúng bẫy lửa! Mất 10% HP.");
        const lost = Math.floor(player.stats.MaxHP * 0.1);
        player.stats.HP = Math.max(0, player.stats.HP - lost);
      } else if (rand < 0.7) {
        console.log("👹 Xuất hiện quái vật lửa!");
        dungeon.map[ny][nx] = "👹";
      } else {
        console.log("🎁 Nhặt được rương giữa dung nham!");
        dungeon.map[ny][nx] = "C";
      }
    }
    else if (type === "ice") {
      if (rand < 0.4) {
        console.log("❄️ Trượt ngã trên băng! Mất 5% thể lực.");
        const lost = Math.floor(player.stats.MaxSucBen * 0.05);
        player.stats.SucBen = Math.max(0, player.stats.SucBen - lost);
      } else if (rand < 0.8) {
        console.log("🧊 Nhặt được tinh thể băng!");
        dungeon.map[ny][nx] = "C";
      } else {
        console.log("👻 Quái băng xuất hiện!");
        dungeon.map[ny][nx] = "👹";
      }
    }
    else if (type === "cave") {
      if (rand < 0.5) {
        console.log("🕷️ Đụng phải quái hang!");
        dungeon.map[ny][nx] = "👹";
      } else {
        console.log("💎 Phát hiện rương ẩn trong bóng tối!");
        dungeon.map[ny][nx] = "C";
      }
    }
    else if (type === "jungle") {
      if (rand < 0.4) {
        console.log("🌿 Dính bẫy dây leo! Giảm thể lực 10%");
        const lost = Math.floor(player.stats.MaxSucBen * 0.1);
        player.stats.SucBen = Math.max(0, player.stats.SucBen - lost);
      } else if (rand < 0.8) {
        console.log("🐍 Rắn độc phục kích!");
        dungeon.map[ny][nx] = "👹";
      } else {
        console.log("🍃 Tìm thấy hòm cổ trong rừng!");
        dungeon.map[ny][nx] = "C";
      }
    }
    else {
      if (rand < 0.3) {
        console.log("🎁 Phát hiện rương bí ẩn!");
        dungeon.map[ny][nx] = "C";
      } else if (rand < 0.6) {
        console.log("👹 Một kẻ địch bất ngờ xuất hiện!");
        dungeon.map[ny][nx] = "👹";
      } else {
        console.log("✨ Không có gì đặc biệt... nhưng không khí hơi kỳ lạ.");
      }
    }

    refreshPlayerUI();
    drawDungeon(dungeonCtx);
  }

  // ====================== CỬA & DỊCH CHUYỂN ======================
  else if (t === "🌀") {
    console.log(`🌀 Dịch chuyển đến tầng ${window.currentFloor + 1}!`);
    nextDungeonFloor();
    return;
  } else if (t === "D" || t === "G") {
    handleDoorAt(nx, ny, t);
    return;
  }

  // ✅ Cập nhật mini-map & render
  markRoomVisited(dungeon);
  drawRoomMiniMap();
  drawDungeon(dungeonCtx);
}


function handleDoorAt(nx, ny, t) {
  const isGold = (t === "G");

  // đảm bảo dungeon có id
  dungeon.id = dungeon.id || (crypto?.randomUUID ? crypto.randomUUID() : ('d_' + Math.random().toString(36).slice(2, 9)));
  window.dungeonGraph = window.dungeonGraph || {};

  // tìm đối tượng cửa tại ô đang bước vào
  const thisDoor = (dungeon.doors || []).find(d => d.x === nx && d.y === ny);
  if (!thisDoor) {
    console.warn("Không tìm thấy door object tại", nx, ny);
    return;
  }

  // --- xác định hướng tương đối của cửa để vẽ mini-map ---
  let doorDir = "down";
  const px = playerPos.prevX ?? playerPos.x;
  const py = playerPos.prevY ?? playerPos.y;
  if (thisDoor.x < px) doorDir = "left";
  else if (thisDoor.x > px) doorDir = "right";
  else if (thisDoor.y < py) doorDir = "up";
  else if (thisDoor.y > py) doorDir = "down";
  // ------------------------------------------------------

  // --- nếu cửa đã liên kết tới dungeon khác ---
  if (thisDoor.toDungeonId && window.dungeonGraph[thisDoor.toDungeonId]) {
    const targetDungeon = window.dungeonGraph[thisDoor.toDungeonId];
    const prevDungeonId = dungeon.id;
    dungeon = targetDungeon;
    inBossRoom = !!dungeon.isBossRoom;
    if (inBossRoom) window.bossRoomCreated = false;

    // tìm cửa đích
    let linkedDoor = null;
    if (thisDoor.toDoorId) linkedDoor = (dungeon.doors || []).find(d => d.id === thisDoor.toDoorId);
    if (!linkedDoor) linkedDoor = (dungeon.doors || []).find(d => d.toDungeonId === prevDungeonId);
    if (!linkedDoor) linkedDoor = (dungeon.doors || [])[0];

    if (linkedDoor && linkedDoor.openTo)
      playerPos = { x: linkedDoor.openTo.x, y: linkedDoor.openTo.y };
    else if (linkedDoor) {
      const tryCoords = [
        { x: linkedDoor.x, y: linkedDoor.y - 1 },
        { x: linkedDoor.x, y: linkedDoor.y + 1 },
        { x: linkedDoor.x - 1, y: linkedDoor.y },
        { x: linkedDoor.x + 1, y: linkedDoor.y }
      ];
      let placed = false;
      for (const c of tryCoords) {
        if (dungeon.map[c.y] && dungeon.map[c.y][c.x] === 0) {
          playerPos = { x: c.x, y: c.y };
          placed = true;
          break;
        }
      }
      if (!placed) playerPos = { ...dungeon.playerStart };
    } else playerPos = { ...dungeon.playerStart };

    markRoomVisited(dungeon);
    drawRoomMiniMap();
    drawDungeon(dungeonCtx);
    return;
  }

  // --- nếu cửa chưa liên kết -> tạo dungeon mới hoặc boss ---
  const fromDungeonId = dungeon.id;
  const worldType = window.currentWorldType || (selectedWorld?.type ?? "tieu");
  const worldKey = selectedWorld ? (selectedWorld.id || selectedWorld.type) : worldType;
  ensureWorldQuota(worldKey, worldType);

  if (isGold) {
    // 🟡 TẠO BOSS ROOM
    const bossRoom = generateBossRoom();
    bossRoom.id = (crypto?.randomUUID ? crypto.randomUUID() : ('d_' + Math.random().toString(36).slice(2, 9)));
    bossRoom.prevId = fromDungeonId;
    bossRoom.isBossRoom = true;
    bossRoom.doors = bossRoom.doors || [];

    const backX = Math.floor(bossRoom.w / 2);
    const backY = bossRoom.h - 2;

    const bossDoorId = (crypto?.randomUUID ? crypto.randomUUID() : ('door_' + Math.random().toString(36).slice(2, 9)));
    bossRoom.map[backY][backX] = "G";
    bossRoom.map[backY - 1][backX] = 0;

    const bossBackDoor = {
      id: bossDoorId,
      x: backX, y: backY,
      openTo: { x: backX, y: backY - 1 },
      special: true,
      toDungeonId: fromDungeonId,
      toDoorId: thisDoor.id
    };
    bossRoom.doors.push(bossBackDoor);

    thisDoor.toDungeonId = bossRoom.id;
    thisDoor.toDoorId = bossDoorId;

    window.dungeonGraph[bossRoom.id] = bossRoom;
    window.dungeonGraph[fromDungeonId] = window.dungeonGraph[fromDungeonId] || dungeon;

    dungeon = bossRoom;
    inBossRoom = true;
    playerPos = { x: backX, y: backY - 1 };
    bossRoom._canSpawnChildren = true;

    linkMiniMapRooms(window.dungeonGraph[fromDungeonId], dungeon, thisDoor,
      (dungeon.doors || []).find(d => d.toDungeonId === fromDungeonId || d.toDoorId === thisDoor.id)
    );
    markRoomVisited(dungeon);
    drawRoomMiniMap();
    drawDungeon(dungeonCtx);
    return;
  }

  // 🧩 TẠO DUNGEON THƯỜNG
  const canSpawnFromThis = !!dungeon._canSpawnChildren;
  if (!canSpawnFromThis) {
    console.log("⚠️ Phòng hiện tại không được phép sinh thêm (quota đã hết hoặc boss).");
    drawDungeon(dungeonCtx);
    return;
  }

  ensureWorldQuota(worldKey, worldType);
  const canSpawnFurther = (window.worldRoomQuota[worldKey] > 0);
  if (canSpawnFurther) consumeRoomQuota(worldKey);

  const newDungeon = generateDungeon(dungeon.theme, dungeon.w, dungeon.h);
  newDungeon.id = (crypto?.randomUUID ? crypto.randomUUID() : ('d_' + Math.random().toString(36).slice(2, 9)));
  newDungeon._canSpawnChildren = canSpawnFurther;

  // 1️⃣ Xác định hướng đối diện
  let oppositeDir;
  switch (doorDir) {
    case "up": oppositeDir = "down"; break;
    case "down": oppositeDir = "up"; break;
    case "left": oppositeDir = "right"; break;
    case "right": oppositeDir = "left"; break;
    default: oppositeDir = "up"; break;
  }

  // 2️⃣ Tạo cửa đối hướng
  let doorBX, doorBY, openX, openY;
  const margin = 1;

  if (oppositeDir === "up") {
    doorBX = Math.floor(newDungeon.w / 2);
    doorBY = margin;
    openX = doorBX; openY = doorBY + 1;
  } else if (oppositeDir === "down") {
    doorBX = Math.floor(newDungeon.w / 2);
    doorBY = newDungeon.h - margin - 1;
    openX = doorBX; openY = doorBY - 1;
  } else if (oppositeDir === "left") {
    doorBX = margin;
    doorBY = Math.floor(newDungeon.h / 2);
    openX = doorBX + 1; openY = doorBY;
  } else if (oppositeDir === "right") {
    doorBX = newDungeon.w - margin - 1;
    doorBY = Math.floor(newDungeon.h / 2);
    openX = doorBX - 1; openY = doorBY;
  }

  newDungeon.map[doorBY][doorBX] = "D";
  newDungeon.map[openY][openX] = 0;

  // Tạo đường thông giữa cửa và trung tâm
  const centerX = Math.floor(newDungeon.w / 2);
  const centerY = Math.floor(newDungeon.h / 2);
  let cx = openX, cy = openY;
  while (cx !== centerX || cy !== centerY) {
    if (cx < centerX) cx++;
    else if (cx > centerX) cx--;
    if (cy < centerY) cy++;
    else if (cy > centerY) cy--;
    if (newDungeon.map[cy] && newDungeon.map[cy][cx] === 1)
      newDungeon.map[cy][cx] = 0;
  }

  // 4️⃣ Tạo object cửa đối diện
  const doorBId = (crypto?.randomUUID ? crypto.randomUUID() : ('door_' + Math.random().toString(36).slice(2, 9)));
  const doorB = {
    id: doorBId,
    x: doorBX,
    y: doorBY,
    openTo: { x: openX, y: openY },
    special: false,
    toDungeonId: dungeon.id,
    toDoorId: thisDoor.id
  };

  thisDoor.toDungeonId = newDungeon.id;
  thisDoor.toDoorId = doorBId;

  window.dungeonGraph[newDungeon.id] = newDungeon;
  window.dungeonGraph[dungeon.id] = dungeon;
  newDungeon.doors = newDungeon.doors || [];
  newDungeon.doors.push(doorB);

  dungeon = newDungeon;
  inBossRoom = false;
  playerPos = { x: openX, y: openY };

  linkMiniMapRooms(window.dungeonGraph[fromDungeonId], dungeon, thisDoor, doorB);
  markRoomVisited(dungeon);
  drawRoomMiniMap();
  drawDungeon(dungeonCtx);
}


// ==================== MINI-MAP PHÒNG (CẢI TIẾN) ====================

let roomMapVisited = {};   // { dungeonId: true }
let roomMapPositions = {}; // { dungeonId: {x, y} }
let roomMapCanvas = null;

// --- Mini-map toggle: mặc định tắt ---
function setupMiniMapToggle() {
  const old=document.getElementById("toggleMiniMapBtn");
  if(old) old.remove();

  initRoomMiniMap(); // tạo canvas
  const btn=document.createElement("button");
  btn.id="toggleMiniMapBtn";
  btn.textContent="🧭";
  Object.assign(btn.style,{
    position:"fixed",top:"10px",right:"10px",
    width:"44px",height:"44px",fontSize:"22px",
    borderRadius:"10px",border:"1px solid rgba(255,255,255,0.2)",
    background:"rgba(0,0,0,0.35)",color:"#fff",
    backdropFilter:"blur(6px)",cursor:"pointer",zIndex:"10000"
  });

  const map=document.getElementById("roomMiniMap");
  if(map){map.style.display="none";btn.style.opacity="0.7";}
  btn.onclick=()=>{
    const map=document.getElementById("roomMiniMap");
    if(!map) return;
    const hidden=map.style.display==="none";
    map.style.display=hidden?"block":"none";
    btn.style.opacity=hidden?"1":"0.7";
  };
  document.body.appendChild(btn);
}

// Khởi tạo/tái sử dụng mini-map
function initRoomMiniMap() {
  if (!document.getElementById("roomMiniMap")) {
    const c = document.createElement("canvas");
    c.id = "roomMiniMap";
    Object.assign(c.style, {
      position: "fixed",
      top: "10px",
      right: "10px",
      width: "180px",
      height: "180px",
      background: "rgba(0,0,0,0.5)",
      borderRadius: "8px",
      border: "1px solid rgba(255,255,255,0.2)",
      zIndex: "9999",
      imageRendering: "pixelated"
    });
    document.body.appendChild(c);
  }
  roomMapCanvas = document.getElementById("roomMiniMap");

  if (!window.dungeonGraph) window.dungeonGraph = {};

  // đảm bảo dungeon có id
  if (dungeon && !dungeon.id) dungeon.id = 'd_' + Math.random().toString(36).slice(2, 9);

  // Khi lần đầu vào mini-map, nếu chưa có vị trí cho phòng hiện tại -> đặt làm trung tâm (0,0)
  if (dungeon && !roomMapPositions[dungeon.id]) {
    roomMapPositions[dungeon.id] = { x: 0, y: 0 };
    roomMapVisited[dungeon.id] = true;
  }
}

// Đánh dấu phòng đã ghé qua (dùng khi spawn hoặc khi link)
function markRoomVisited(d) {
  if (!d || !d.id) return;
  roomMapVisited[d.id] = true;
  if (!roomMapPositions[d.id]) {
    // nếu chưa có vị trí (lỗi fallback) -> đặt gần origin
    roomMapPositions[d.id] = { x: 0, y: 0 };
  }
  drawRoomMiniMap();
}

// helper: xác định hướng cửa trên phòng 'fromDungeon' dựa vào tọa độ cửa (doorA)
function determineExitDirection(fromDungeon, doorA) {
  if (!fromDungeon || !doorA) return { dx: 0, dy: 1 };

  const { w, h } = fromDungeon;
  const margins = {
    left: doorA.x,
    right: w - 1 - doorA.x,
    top: doorA.y,
    bottom: h - 1 - doorA.y
  };

  // Hướng về phía cạnh gần nhất
  const minDist = Math.min(margins.left, margins.right, margins.top, margins.bottom);
  if (minDist === margins.left) return { dx: -1, dy: 0 };
  if (minDist === margins.right) return { dx: 1, dy: 0 };
  if (minDist === margins.top) return { dx: 0, dy: -1 };
  if (minDist === margins.bottom) return { dx: 0, dy: 1 };

  // fallback
  return { dx: 0, dy: 1 };
}

// helper: tính offset phụ để phân tách nhiều cửa trên cùng một cạnh
function computeOrthogonalOffset(fromDungeon, doorA, dir) {
  // dir: {dx,dy} với một trong hai là 0
  const w = fromDungeon.w, h = fromDungeon.h;
  // khi đi ngang (dx != 0) -> offset theo trục y dựa doorA.y - center
  if (dir.dx !== 0) {
    const rel = doorA.y - h / 2;
    const scale = Math.max(1, Math.floor(h / 6)); // chia khu vực thành vài phần
    return Math.round(rel / scale); // có thể âm/0/dương
  }
  // khi đi dọc (dy != 0) -> offset theo trục x
  const rel = doorA.x - w / 2;
  const scale = Math.max(1, Math.floor(w / 6));
  return Math.round(rel / scale);
}

// helper: kiểm tra vị trí đã có phòng khác chưa
function isPosOccupied(pos) {
  for (const id in roomMapPositions) {
    const p = roomMapPositions[id];
    if (p.x === pos.x && p.y === pos.y) return id; // trả về id phòng đang chiếm
  }
  return null;
}

// helper: tìm vị trí trống gần desiredPos (spiral search)
function findNearestFreePos(desiredPos, maxRadius = 6) {
  // spiral offsets
  const dirs = [[1,0],[0,1],[-1,0],[0,-1]];
  if (!isPosOccupied(desiredPos)) return desiredPos;
  for (let r = 1; r <= maxRadius; r++) {
    // iterate ring r
    let x = desiredPos.x - r;
    let y = desiredPos.y - r;
    // go around perimeter
    for (let side = 0; side < 4; side++) {
      const steps = r * 2;
      for (let s = 0; s < steps; s++) {
        if (!isPosOccupied({ x, y })) return { x, y };
        x += dirs[side][0];
        y += dirs[side][1];
      }
    }
  }
  // fallback: move further right until free
  let fx = desiredPos.x, fy = desiredPos.y;
  for (let i = 1; i < 50; i++) {
    if (!isPosOccupied({ x: fx + i, y: fy })) return { x: fx + i, y: fy };
    if (!isPosOccupied({ x: fx - i, y: fy })) return { x: fx - i, y: fy };
    if (!isPosOccupied({ x: fx, y: fy + i })) return { x: fx, y: fy + i };
    if (!isPosOccupied({ x: fx, y: fy - i })) return { x: fx, y: fy - i };
  }
  return desiredPos; // xấu nhưng trả về để không crash
}

/*
  Link two rooms on mini-map in a deterministic, non-overlapping way.

  fromDungeon: dungeon object we are coming FROM
  toDungeon: dungeon object we are going TO
  doorA: door object in fromDungeon (the door we stepped onto)
  doorB: door object in toDungeon that links back (optional)
*/
function linkMiniMapRooms(fromDungeon, toDungeon, doorA, doorB) {
  if (!fromDungeon || !toDungeon || !doorA) return;

  // ensure from has position
  if (!roomMapPositions[fromDungeon.id]) {
    roomMapPositions[fromDungeon.id] = { x: 0, y: 0 };
    roomMapVisited[fromDungeon.id] = true;
  }

  // if toDungeon already positioned, nothing to do
  if (roomMapPositions[toDungeon.id]) return;

  // 1) compute primary direction using fromDungeon & doorA
  const dir = determineExitDirection(fromDungeon, doorA); // {dx,dy}

  // 2) compute orthogonal offset based on doorA location so two north-doors differ by offset
  const orth = computeOrthogonalOffset(fromDungeon, doorA, dir); // integer, can be negative

  // desired base position (one cell away in dir)
  let desired = { x: roomMapPositions[fromDungeon.id].x + dir.dx, y: roomMapPositions[fromDungeon.id].y + dir.dy };

  // apply orthogonal offset
  if (dir.dx !== 0) {
    desired.y += orth; // moving left/right, offset along y
  } else if (dir.dy !== 0) {
    desired.x += orth; // moving up/down, offset along x
  }

  // 3) if doorB exists we can further nudge towards doorB relative orientation
  if (doorB) {
    // compute doorB offset relative to toDungeon center, normalized similar to orth to better align
    const w2 = toDungeon.w, h2 = toDungeon.h;
    const relXB = doorB.x - w2 / 2;
    const relYB = doorB.y - h2 / 2;
    // if directions disagree, try to align sign
    if (Math.abs(relXB) > Math.abs(relYB)) {
      // horizontal bias
      desired.y += Math.round(relYB / Math.max(1, Math.floor(h2 / 6)));
      desired.x += Math.sign(relXB); // nudge horizontally small
    } else {
      desired.x += Math.round(relXB / Math.max(1, Math.floor(w2 / 6)));
      desired.y += Math.sign(relYB);
    }
  }

  // 4) avoid collision: if desired occupied, find nearest free pos
  const occupiedId = isPosOccupied(desired);
  if (occupiedId) {
    const freePos = findNearestFreePos(desired, 8);
    desired = freePos;
  }

  // assign
  roomMapPositions[toDungeon.id] = { x: desired.x, y: desired.y };
  roomMapVisited[toDungeon.id] = true;

  // draw
  drawRoomMiniMap();
}

// Vẽ mini-map (có nối)
function drawRoomMiniMap() {
  const c = roomMapCanvas;
  if (!c || !dungeon) return;
  const ctx = c.getContext("2d");
  const cw = c.width = 180;
  const ch = c.height = 180;

  ctx.clearRect(0, 0, cw, ch);

  const rooms = Object.entries(roomMapPositions);
  if (!rooms.length) return;

  // === 1️⃣ Tính phạm vi phòng ===
  let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
  for (const [, pos] of rooms) {
    minX = Math.min(minX, pos.x);
    maxX = Math.max(maxX, pos.x);
    minY = Math.min(minY, pos.y);
    maxY = Math.max(maxY, pos.y);
  }

  const roomCountX = maxX - minX + 1;
  const roomCountY = maxY - minY + 1;

  // === 2️⃣ Tự động scale để vừa khung (padding nhỏ) ===
  const padding = 20;
  const cellX = (cw - padding * 2) / Math.max(roomCountX, 1);
  const cellY = (ch - padding * 2) / Math.max(roomCountY, 1);
  const cell = Math.min(cellX, cellY, 26); // không to quá

  // === 3️⃣ Tính offset để canh giữa tất cả ===
  const totalW = roomCountX * cell;
  const totalH = roomCountY * cell;
  const offsetX = (cw - totalW) / 2 - minX * cell;
  const offsetY = (ch - totalH) / 2 - minY * cell;

  // === 4️⃣ Vẽ các đường nối (rút ngắn hợp lý) ===
  ctx.lineWidth = 2;
  ctx.strokeStyle = "rgba(200,200,200,0.25)";
  for (const id in roomMapPositions) {
    const pos = roomMapPositions[id];
    const dObj = window.dungeonGraph && window.dungeonGraph[id];
    if (!dObj || !dObj.doors) continue;

    for (const door of dObj.doors) {
      const targetId = door.toDungeonId;
      if (!roomMapPositions[targetId]) continue;

      const p1 = roomMapPositions[id];
      const p2 = roomMapPositions[targetId];
      const x1 = offsetX + p1.x * cell;
      const y1 = offsetY + p1.y * cell;
      const x2 = offsetX + p2.x * cell;
      const y2 = offsetY + p2.y * cell;

      // điều chỉnh để nối từ mép phòng này sang mép phòng kia
const dx = x2 - x1;
const dy = y2 - y1;
const dist = Math.hypot(dx, dy);
const nx = dx / dist, ny = dy / dist;

// bán kính “phòng” nhỏ ~4px
const roomHalf = 4;

ctx.beginPath();
ctx.moveTo(x1 + nx * roomHalf, y1 + ny * roomHalf);
ctx.lineTo(x2 - nx * roomHalf, y2 - ny * roomHalf);
ctx.stroke();
    }
  }

  // === 5️⃣ Vẽ phòng (ô vuông) ===
  for (const id in roomMapPositions) {
    const pos = roomMapPositions[id];
    const x = offsetX + pos.x * cell;
    const y = offsetY + pos.y * cell;
    const visited = !!roomMapVisited[id];
    ctx.fillStyle = visited ? "rgba(255,255,255,0.9)" : "rgba(80,80,80,0.35)";
    ctx.fillRect(x - 4, y - 4, 8, 8);
  }

  // === 6️⃣ Phòng hiện tại (vàng) ===
  const curPos = roomMapPositions[dungeon.id];
  if (curPos) {
    const x = offsetX + curPos.x * cell;
    const y = offsetY + curPos.y * cell;
    ctx.fillStyle = "gold";
    ctx.fillRect(x - 5, y - 5, 10, 10);
    ctx.strokeStyle = "rgba(255,255,255,0.5)";
    ctx.strokeRect(x - 7, y - 7, 14, 14);
  }

  // === 7️⃣ Viền khung mini-map (tùy chọn, giúp test) ===
  // === Hiển thị số tầng ===
  ctx.fillStyle = "rgba(255,255,255,0.9)";
  ctx.font = "14px sans-serif";
  ctx.textAlign = "left";
  ctx.fillText(`Tầng ${window.currentFloor}`, 10, 18);
  ctx.strokeStyle = "rgba(255,255,255,0.15)";
  ctx.strokeRect(0, 0, cw, ch);
}

// === Biểu tượng nhân vật động có hướng + hiệu ứng nhún ===
let playerWalkToggle = false;     // luân phiên 🚶 / 🏃
let lastWalkAnimTime = 0;         // thời điểm đổi frame gần nhất
let lastMoveDir = { dx: 0, dy: 0 }; // hướng di chuyển gần nhất
let walkBobPhase = 0;             // pha dao động "nhún"
let lastBobTime = 0;

// ========== NHÂN VẬT CÓ BIỂU CẢM SỐNG (v5 - 🥴 + hiệu ứng ngủ 💤) ==========

let idleStartTime = 0;       // thời điểm bắt đầu đứng yên
let lastIdleMood = "🙂";      // biểu cảm hiện tại
let sleepStage = 0;          // 0=thức,1=🥱,2=😴,3=😪
let justWokeUp = false;      // vừa tỉnh sau 😪
let wakeFaceUntil = 0;       // hết thời gian 🥴
let lastRandomBlink = 0;     // thay biểu cảm ngẫu nhiên
let zFloatPhase = 0;         // hiệu ứng "💤" nổi

function drawPlayerSymbol(ctx, px, py) {
  const now = performance.now();
  const walkInterval = 250;
  const bobSpeed = 7;
  const bobHeight = 1;

  // --- ghi nhớ hướng ---
  if (player.moveDir) lastMoveDir = { ...player.moveDir };

  // --- đổi frame khi di chuyển ---
  if (player.moving && now - lastWalkAnimTime > walkInterval) {
    playerWalkToggle = !playerWalkToggle;
    lastWalkAnimTime = now;
  }

  let sym = "🙂";
  let flip = false;

  // =========================================================
  // 1️⃣ Khi đang di chuyển
  // =========================================================
  if (player.moving) {
    // Nếu vừa thoát khỏi giấc ngủ sâu → đánh dấu vừa tỉnh
    if (sleepStage === 3) {
      justWokeUp = true;
    }

    // Icon di chuyển bình thường
    if (lastMoveDir.dx !== 0) {
      sym = playerWalkToggle ? "🚶" : "🏃";
      flip = lastMoveDir.dx > 0;
    } else if (lastMoveDir.dy !== 0) {
      sym = "🧍"; // đi lên/xuống
    }

    // reset trạng thái idle
    idleStartTime = now;
    lastIdleMood = "🙂";
    sleepStage = 0;
  }

  // =========================================================
  // 2️⃣ Khi đứng yên
  // =========================================================
  else {
    if (idleStartTime === 0) idleStartTime = now;
    const idleTime = (now - idleStartTime) / 1000;

    // Nếu vừa tỉnh và dừng lại → 🥴 trong 2s
    if (justWokeUp && now > wakeFaceUntil) {
      sym = "🥴";
      wakeFaceUntil = now + 2000;
      justWokeUp = false;
      sleepStage = 0;
    }
    else if (now < wakeFaceUntil) {
      sym = "🥴";
    }
    else {
      // Biểu cảm ngủ dần dần
      if (idleTime > 25) {
        sym = "😪";
        sleepStage = 3;
      } else if (idleTime > 18) {
        sym = "😴";
        sleepStage = 2;
      } else if (idleTime > 12) {
        sym = "🥱";
        sleepStage = 1;
      } else {
        // Bình thường / biểu cảm phụ
        sleepStage = 0;
        if (now - lastRandomBlink > 4000 + Math.random() * 4000) {
          const r = Math.random();
          if (r < 0.25) lastIdleMood = "😎";
          else if (r < 0.5) lastIdleMood = "👀";
          else lastIdleMood = "🙂";
          lastRandomBlink = now;
        }
        sym = lastIdleMood;
      }
    }

    if (lastMoveDir.dx > 0) flip = true;
  }

  // =========================================================
  // 3️⃣ Hiệu ứng nhún
  // =========================================================
  if (player.moving) {
    const delta = (now - lastBobTime) / 1000;
    walkBobPhase += delta * bobSpeed;
    lastBobTime = now;
  } else {
    walkBobPhase = 0;
    lastBobTime = now;
  }

  const bobOffset = player.moving ? Math.sin(walkBobPhase * Math.PI * 2) * bobHeight : 0;

  // =========================================================
  // 4️⃣ Vẽ nhân vật
  // =========================================================
  ctx.save();
  ctx.translate(px, py + bobOffset);
  if (flip) ctx.scale(-1, 1);

  ctx.font = "22px monospace";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText(sym, 0, 0);

  // =========================================================
  // 5️⃣ Hiệu ứng "💤" khi ngủ
  // =========================================================
  if (sleepStage >= 2) { // 😴 hoặc 😪
    zFloatPhase += 0.03;
    const floatY = Math.sin(zFloatPhase) * 5 - 25; // dao động lên xuống
    ctx.font = "16px monospace";
    ctx.fillText("💤", 0, floatY);
  }

  ctx.restore();
}

// === ICON HIỆU ỨNG XOAY QUANH NHÂN VẬT (🔥☠️❄️) ===
function drawEffectIcons(ctx, px, py) {
  if (!window.activeEffects || !activeEffects.length) return;
  const now = Date.now();
  const total = activeEffects.length;

  const orbitR = 26; // bán kính quay quanh nhân vật
  const iconSize = 16; // kích thước nhỏ gọn

  activeEffects.forEach((e, i) => {
    const remaining = Math.max(0, e.expires - now);
    const progress = e.initialDuration ? remaining / e.initialDuration : 0;
    const angle = ((i / total) * Math.PI * 2) + (now / 900); // quay đều
    const pulse = 0.8 + 0.2 * Math.sin(now / 300 + i); // nhấp nháy nhẹ

    const x = px + Math.cos(angle) * orbitR;
    const y = py + Math.sin(angle) * orbitR;

    // Chọn icon và màu ánh sáng
    let icon = "✨";
    let glow = "rgba(255,255,255,0.5)";
    if (e.type === "burn") { icon = "🔥"; glow = "rgba(255,80,0,0.8)"; }
    else if (e.type === "poison") { icon = "☠️"; glow = "rgba(100,255,100,0.8)"; }
    else if (e.type === "freeze") { icon = "❄️"; glow = "rgba(120,200,255,0.9)"; }

    // Nếu đang tan → làm nổ sáng nhẹ rồi mờ dần
    const alpha = e.fadingOut ? e.alpha ** 1.5 : pulse * 0.9;
    const scale = e.fadingOut ? 1 + (1 - e.alpha) * 1.5 : 1;

    ctx.save();
    ctx.translate(x, y);
    ctx.scale(scale, scale);
    ctx.globalAlpha = alpha;
    ctx.font = `${iconSize}px monospace`;
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.shadowBlur = 6;
    ctx.shadowColor = glow;
    ctx.fillText(icon, 0, 0);
    ctx.restore();
  });

  // Hiển thị số stack (🔥x3, ☠️x2, ...)
  const grouped = {};
  activeEffects.forEach(e => grouped[e.type] = (grouped[e.type] || 0) + 1);
  const offsetY = -38;
  let offsetX = -12;
  Object.entries(grouped).forEach(([type, count]) => {
    const sym = type === "burn" ? "🔥" : type === "poison" ? "☠️" : "❄️";
    ctx.save();
    ctx.font = "15px monospace";
    ctx.globalAlpha = 0.9;
    ctx.fillText(`${sym}x${count}`, px + offsetX, py + offsetY);
    offsetX += 45;
    ctx.restore();
  });
}

// === Vẽ dungeon (gọi mỗi frame) 
function drawDungeon(ctx) {
  if (!ctx || !dungeon) return;
  dungeonBlink += 0.06;
  const map = dungeon.map, w = dungeon.w, h = dungeon.h;
  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  const trait = DUNGEON_TRAITS[dungeon.theme] || {};
  const baseColor = trait.floorColor || "#444";
  const theme = dungeon.theme || "cave";

  // 🌄 Gradient nền tùy theo theme
  const grad = ctx.createLinearGradient(0, 0, 0, ctx.canvas.height);
  switch (theme) {
    case "desert": grad.addColorStop(0, "#f7e8b3"); grad.addColorStop(1, "#d8b871"); break;
    case "ice": grad.addColorStop(0, "#b8e0ff"); grad.addColorStop(1, "#8cb4e8"); break;
    case "jungle": grad.addColorStop(0, "#204d24"); grad.addColorStop(1, "#357a38"); break;
    case "volcano": grad.addColorStop(0, "#400000"); grad.addColorStop(1, "#7a1f00"); break;
    case "cave": grad.addColorStop(0, "#1b1b1b"); grad.addColorStop(1, "#2f2f2f"); break;
    case "swamp": grad.addColorStop(0, "#2f3229"); grad.addColorStop(1, "#444b3a"); break;
    case "mountain": grad.addColorStop(0, "#4a4a4a"); grad.addColorStop(1, "#6e6e6e"); break;
    case "boss": grad.addColorStop(0, "#4a0000"); grad.addColorStop(0.5, "#7a1a00"); grad.addColorStop(1, "#a83a00"); break;
    default: grad.addColorStop(0, "#222"); grad.addColorStop(1, "#333"); break;
  }
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  // === camera ===
  const viewW = Math.floor(ctx.canvas.width / tileSize);
  const viewH = Math.floor(ctx.canvas.height / tileSize);
  const halfW = Math.floor(viewW / 2), halfH = Math.floor(viewH / 2);
  let camX = Math.max(0, Math.min(w - viewW, playerPos.x - halfW));
  let camY = Math.max(0, Math.min(h - viewH, playerPos.y - halfH));

  // === nền & tường ===
  for (let y = 0; y < viewH; y++) {
    for (let x = 0; x < viewW; x++) {
      const mapX = camX + x, mapY = camY + y;
      const t = map[mapY]?.[mapX];
      if (t == null) continue;

      // --- Nếu là tường ---
      if (t === 1) {
        const wallSym = (trait.wallSymbol || "⬛").trim();
        ctx.font = `${tileSize}px monospace`;
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(wallSym, x * tileSize + tileSize / 2, y * tileSize + tileSize / 2);
        continue;
      }
      
      // --- Biểu tượng đặc biệt ---
      if (["C","E","?","D","G","🔥","❄️","☠️","🧨","🕳️","👹","🎁","🌀","🌫️"].includes(t)) {
        ctx.font = "20px monospace";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        const sym = {
          "C":"💰","E":"👾","?":"❓","D":"🚪",
          "G":"🚪✨","🔥":"🔥","❄️":"❄️","☠️":"☠️",
          "🧨":"🧨","🕳️":"🕳️","👹":"👹","🎁":"🎁","🌀":"🌀","🌫️":"🌫️"
        }[t];
        ctx.fillText(sym, x * tileSize + tileSize / 2, y * tileSize + tileSize / 2);
      }
    }
  }

  // === Vẽ player ===
  const px = (playerPos.x - camX) * tileSize + tileSize / 2;
  const py = (playerPos.y - camY) * tileSize + tileSize / 2;
  drawPlayerSymbol(ctx, px, py);
  drawEffectIcons(ctx, px, py);

  // === Vẽ particle (phát hiện phòng boss)
  const isBossRoom = !!dungeon.isBossRoom;
  updateDungeonParticles(ctx, theme, trait.fog, isBossRoom);

  // === Hiệu ứng nhịp tim quanh player khi ở phòng boss
  if (isBossRoom) {
    drawBossLightPulse(ctx, px, py, lightRadius * tileSize * 1.3);
  } else {
    const g = ctx.createRadialGradient(px, py, 0, px, py, lightRadius * tileSize);
    g.addColorStop(0, "rgba(255,255,255,0)");
    g.addColorStop(1, "rgba(0,0,0,0.45)");
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);
    window._bossLightFade = 0;
  }

  try { safeUpdateBars(); } catch (e) {}

  // === Particle động ===
  updateDungeonParticles(ctx, theme);

  // === Fog (sương mù) ===
  if (trait.fog === "frost") ctx.fillStyle = "rgba(200,240,255,0.2)";
  else if (trait.fog === "heat") ctx.fillStyle = "rgba(255,100,0,0.15)";
  else if (trait.fog === "water") ctx.fillStyle = "rgba(0,80,200,0.2)";
  else if (trait.fog === "dense") ctx.fillStyle = "rgba(30,30,30,0.35)";
  else if (trait.fog === "light") ctx.fillStyle = "rgba(255,255,200,0.1)";
  if (trait.fog) ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  // === Ánh sáng quanh player ===
  const g = ctx.createRadialGradient(px, py, 0, px, py, lightRadius * tileSize);
  g.addColorStop(0, "rgba(255,255,255,0)");
  g.addColorStop(1, "rgba(0,0,0,0.45)");
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);

  // cập nhật HUD
  try { safeUpdateBars(); } catch(e){}
}


// === Reset hạt khi đổi tầng hoặc thoát dungeon ===
let dungeonParticles = [];
let dungeonParticleTick = 0;

function resetDungeonParticles() {
  dungeonParticles = [];
  dungeonParticleTick = 0;
}

// 🌋 Hiệu ứng particle động cho dungeon (kéo dài, có boss mode)
function updateDungeonParticles(ctx, theme, fogType, isBossRoom = false) {
  const MAX_PARTICLES = isBossRoom ? 100 : 70;
  dungeonParticleTick++;

  // Tạo hạt mới nếu thiếu
  while (dungeonParticles.length < MAX_PARTICLES) {
    dungeonParticles.push({
      x: Math.random() * ctx.canvas.width,
      y: Math.random() * ctx.canvas.height,
      s: Math.random() * 2.2 + 0.5,
      v: Math.random() * 0.4 + 0.1,
      alpha: Math.random() * 0.7 + 0.2,
      drift: Math.random() * 0.5 - 0.25,
      life: Math.random() * 1000 + 1000, // sống rất lâu (20–40s)
      age: Math.random() * 500,
      colorShift: Math.random(),
    });
  }

  // Màu và kiểu hạt theo khu vực
  let color = "rgba(255,255,255,0.3)";
  let mode = "float";

  switch (theme) {
    case "ice":        color = "rgba(210,240,255,0.8)"; mode = "snow"; break;
    case "tundra":     color = "rgba(230,250,255,0.9)"; mode = "snow"; break;
    case "volcano":    color = "rgba(255,120,40,0.5)";  mode = "ash"; break;
    case "desert":     color = "rgba(255,220,150,0.4)"; mode = "sand"; break;
    case "jungle":     color = "rgba(140,255,140,0.5)"; mode = "firefly"; break;
    case "swamp":      color = "rgba(180,255,160,0.3)"; mode = "mist"; break;
    case "deepsea":    color = "rgba(120,200,255,0.4)"; mode = "bubble"; break;
    case "sea":        color = "rgba(80,180,255,0.3)";  mode = "bubble"; break;
    case "cave":       color = "rgba(220,220,255,0.25)";mode = "dust"; break;
    case "mountain":   color = "rgba(255,255,255,0.3)"; mode = "wind"; break;
    case "wasteland":  color = "rgba(255,210,160,0.3)"; mode = "dust"; break;
    case "ruins":      color = "rgba(255,255,200,0.2)"; mode = "mote"; break;
    case "grassland":  color = "rgba(200,255,200,0.4)"; mode = "pollen"; break;
    case "plain":      color = "rgba(255,255,180,0.4)"; mode = "pollen"; break;
  }

  // Boss room có hiệu ứng đặc biệt
  if (isBossRoom) {
    color = "rgba(255,80,40,0.55)";
    mode = "ember";
  }

  ctx.globalAlpha = 1;
  ctx.fillStyle = color;

  for (let p of dungeonParticles) {
    p.age++;

    // Làm hạt lung linh nhẹ
    const flicker = 0.7 + 0.3 * Math.sin(dungeonParticleTick * 0.05 + p.colorShift * Math.PI * 2);
    const fade = Math.max(0, 1 - p.age / p.life);
    ctx.globalAlpha = p.alpha * fade * flicker;

    ctx.beginPath();
    ctx.arc(p.x, p.y, p.s, 0, Math.PI * 2);
    ctx.fill();

    // Di chuyển tùy mode
    switch (mode) {
      case "snow":      p.y += p.v; p.x += Math.sin(p.y * 0.02) * 0.3; break;
      case "ash":       p.y -= p.v; p.x += p.drift * 0.3; break;
      case "sand":      p.x += p.drift * 1.5; p.y += Math.sin(p.x * 0.03) * 0.1; break;
      case "firefly":   p.x += Math.sin(p.y * 0.05) * 0.5; p.y += Math.cos(p.x * 0.05) * 0.2; break;
      case "bubble":    p.y -= p.v * 0.8; p.x += Math.sin(p.y * 0.02) * 0.2; break;
      case "dust":      p.y -= p.v * 0.2; p.x += p.drift * 0.2; break;
      case "mist":      p.y += p.v * 0.1; p.x += Math.sin(p.y * 0.03) * 0.3; break;
      case "wind":      p.x += p.v * 0.8; p.y += Math.sin(p.x * 0.02) * 0.3; break;
      case "pollen":    p.y += Math.sin(p.x * 0.02) * 0.2; p.x += p.drift * 0.5; break;
      case "ember":     p.y -= p.v * 0.4; p.x += Math.sin(p.y * 0.04) * 0.6; break;
      default:          p.y += Math.sin(p.x * 0.02) * 0.1; break;
    }

    // Reset hạt khi vượt giới hạn hoặc chết
    if (
      p.y > ctx.canvas.height + 10 || p.y < -10 ||
      p.x < -10 || p.x > ctx.canvas.width + 10 ||
      p.age > p.life
    ) {
      Object.assign(p, {
        x: Math.random() * ctx.canvas.width,
        y: Math.random() * ctx.canvas.height,
        s: Math.random() * 2.2 + 0.5,
        v: Math.random() * 0.4 + 0.1,
        alpha: Math.random() * 0.7 + 0.2,
        drift: Math.random() * 0.5 - 0.25,
        life: Math.random() * 1000 + 1000,
        age: 0,
        colorShift: Math.random(),
      });
    }
  }

  ctx.globalAlpha = 1;
}

function drawBossLightPulse(ctx, px, py, radius) {
  if (!ctx) return;

  const now = performance.now();
  // 🫀 Nhịp tim dao động từ 0.4 → 1.0
  const pulse = 0.4 + 0.6 * Math.abs(Math.sin(now / 250));

  // Hiệu ứng fade-in khi mới vào boss room
  window._bossLightFade = Math.min(1, (window._bossLightFade || 0) + 0.03);

  // Gradient ánh sáng đỏ cam mạnh hơn và mở rộng hơn một chút
  const g = ctx.createRadialGradient(px, py, 0, px, py, radius);
  g.addColorStop(0, `rgba(255, 80, 40, ${0.35 * window._bossLightFade})`);
  g.addColorStop(0.4, `rgba(255, 40, 20, ${0.25 * pulse * window._bossLightFade})`);
  g.addColorStop(0.7, `rgba(120, 0, 0, ${0.15 * window._bossLightFade})`);
  g.addColorStop(1, "rgba(0, 0, 0, 0.65)");

  ctx.save();
  ctx.globalCompositeOperation = "lighter"; // làm sáng lớp boss
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);
  ctx.restore();
}


// === Dungeon Loop ===

function refreshPlayerUI(full = false) {
  try {
    safeUpdateBars?.();              // HP, Mana, Stamina
    if (typeof updateExpBar === "function") updateExpBar();
    if (typeof renderStats === "function") renderStats();
    if (typeof renderInventory === "function") renderInventory();
    if (typeof renderActiveEffects === "function") renderActiveEffects();
    if (full) console.log("🔄 Làm mới toàn bộ giao diện nhân vật!");
  } catch (err) {
    console.warn("⚠️ refreshPlayerUI lỗi:", err);
  }
}

let lastUIRefresh = 0; // thời điểm cập nhật UI gần nhất

function dungeonLoop() {
updateEffects();
  checkFogProximity?.(dungeon);
  drawDungeon?.(dungeonCtx);

  // 🕒 Cập nhật UI mỗi 0.5 giây
  const now = Date.now();
  if (now - lastUIRefresh > 500) { // khoảng cách giữa 2 lần refresh
    refreshPlayerUI?.();           // cập nhật thanh máu, stamina, exp, inventory...
    lastUIRefresh = now;
  }

  dungeonLoopId = requestAnimationFrame(dungeonLoop);
}


function startDungeonLoop() {
  resetDungeonParticles();
  if (dungeonLoopId) cancelAnimationFrame(dungeonLoopId);
  dungeonLoopId = requestAnimationFrame(dungeonLoop);
}

function stopDungeonLoop() {
  if (dungeonLoopId) cancelAnimationFrame(dungeonLoopId);
  dungeonLoopId = null;
  resetDungeonParticles();
}

if (!window.currentFloor) window.currentFloor = 1;

// Khóa chống gọi trùng tầng
let nextFloorLock = false;

// --- Qua tầng kế tiếp trong dungeon ---
function nextDungeonFloor() {
  console.log("⬇️ Chuẩn bị sang tầng kế tiếp...");

  // --- Tăng tầng ---
  if (!window.currentFloor) window.currentFloor = 1;
  window.currentFloor++;

  // --- Xác định loại thế giới ---
  const worldType = window.currentWorldType || "tieu";
  const baseTheme = window.dungeonBaseTheme || "cave";

  // --- Ghi log tầng mới ---
  let worldLabel = "Tiểu Thiên Thế Giới";
  if (worldType === "trung") worldLabel = "Trung Thiên Thế Giới";
  else if (worldType === "dai") worldLabel = "Đại Thiên Thế Giới";

  const colorMap = {
    dai: "color:#ff4444;font-weight:bold;",
    trung: "color:#ffbb33;font-weight:bold;",
    tieu: "color:#44ff44;font-weight:bold;"
  };
  console.log(`%c⬇️ Sang tầng ${window.currentFloor} của ${worldLabel}`, colorMap[worldType] || "");

  // --- Reset quota cho tầng mới (mỗi tầng mới là một dungeon độc lập) ---
  const worldKey = selectedWorld ? (selectedWorld.id || selectedWorld.type) : worldType;
  window.worldRoomQuota[worldKey] = initialQuotaForWorld(worldType);
  console.log(`🗺️ Reset quota: ${window.worldRoomQuota[worldKey]} phòng cho tầng ${window.currentFloor}`);

  // --- Tạo dungeon mới cùng theme ---
  const newDungeon = generateDungeon(baseTheme, 25, 18, worldType);
  newDungeon._canSpawnChildren = true; // cho phép sinh thêm phòng
  dungeon = newDungeon;
  playerPos = { ...dungeon.playerStart };

  // --- Reset boss state ---
  inBossRoom = false;
  window.bossRoomCreated = false;

  // --- Reset mini-map & tạo lại ngay ---
  const mm = document.getElementById("roomMiniMap");
  if (mm) mm.remove();
  const toggleBtn = document.getElementById("toggleMiniMapBtn");
  if (toggleBtn) toggleBtn.remove();

  roomMapVisited = {};
  roomMapPositions = {};
  roomMapCanvas = null;
  window.roomMapCenter = null;

  initRoomMiniMap();
  markRoomVisited(dungeon);
  drawRoomMiniMap();
  setupMiniMapToggle();

  // --- Cập nhật canvas & vòng lặp ---
  drawDungeon(dungeonCtx);
  startDungeonLoop();

  console.log(`🌌 Đã sang tầng ${window.currentFloor} (theme: ${baseTheme}, worldType: ${worldType})`);
}


function findClosestWallToPlayer(dungeon) {
  const { map, w, h } = dungeon;
  // ưu tiên tìm wall có ô sàn kề nó (trong vùng trung tâm)
  for (let y = 1; y < h - 1; y++) {
    for (let x = 1; x < w - 1; x++) {
      if (map[y][x] === 1) {
        if (map[y-1] && map[y-1][x] === 0) return { x, y, openTo: { x, y: y-1 } };
        if (map[y+1] && map[y+1][x] === 0) return { x, y, openTo: { x, y: y+1 } };
        if (map[y] && map[y][x-1] === 0) return { x, y, openTo: { x: x-1, y } };
        if (map[y] && map[y][x+1] === 0) return { x, y, openTo: { x: x+1, y } };
      }
    }
  }
  return null;
}

// --- VÒNG LẶP CHÍNH ---
let rafId = null;
  let tick = 0;
function startLoop(){
  if(rafId) cancelAnimationFrame(rafId);

  (function loop(){
    ctx.clearRect(0,0,W,H);

    if(currentMapLevel==="cosmos") {
      drawBackground();
      drawAsteroids();
      drawSpiralNebulaBackground();
      drawWorldRings();
      drawPlanets();
      drawNebulaForeground();
            drawCore(t);

    }
    else if(currentMapLevel==="world") {
      drawWorld();
      renderRegions(ctx);
      
    }
    else if(currentMapLevel==="region") {
      drawRegion();
    }
    else if(currentMapLevel==="dungeon") {
  if (dungeon) drawDungeon(ctx);
}
    
    t++;
    rafId = requestAnimationFrame(loop);
  })();
}


  // khởi động lần đầu
  let resizeTimeout = null;
  window.addEventListener('resize', ()=> {
    clearTimeout(resizeTimeout);
    resizeTimeout = setTimeout(()=> {
      resizeCanvas();
    }, 120);
  });

  // ban đầu gọi resizeCanvas → init tất cả arrays, rồi start loop
  resizeCanvas();
  startLoop();
});
// --- END MAP MODULE ---


// ==================== HỆ THỐNG HIỆU ỨNG (🔥 ☠️ ❄️) ====================
window.updateEffects = updateEffects;
window.applyEffect = applyEffect;

// Danh sách hiệu ứng đang hoạt động (stackable)
var activeEffects = window.activeEffects = window.activeEffects || [];

function applyEffect(effectId, options = {}) {
  const defaultDurations = { burn: 5000, poison: 7000, freeze: 4000 };
  const defaultIntervals = { burn: 1000, poison: 1500, freeze: 1000 };

  // Chỉ số kháng (Khang)
  const resist = calcFinalStat(player.stats.Khang || 0, "Khang") || 0;
  const durationFactor = 1 - Math.min(0.6, resist / 200); // Giảm thời gian tối đa 60%

  const sameType = activeEffects.filter(e => e.type === effectId);
  const maxStacks = options.maxStacks || 5;

  // Nếu đã đạt stack tối đa -> tăng cường stack cũ
  if (sameType.length >= maxStacks) {
    const oldest = sameType[0];
    oldest.power += (options.power || 1) * 0.5;
    oldest.expires += (options.duration || defaultDurations[effectId]) * 0.3 * durationFactor;
    console.log(`💥 ${effectId} đạt giới hạn stack, tăng cường sức mạnh.`);
    return;
  }

  // Tạo stack mới
  const now = Date.now();
  const newEffect = {
    id: `${effectId}_${now}_${Math.floor(Math.random() * 9999)}`,
    type: effectId,
    created: now,
    expires: now + ((options.duration || defaultDurations[effectId]) * durationFactor),
    tickInterval: options.tickInterval || defaultIntervals[effectId],
    lastTick: now,
    power: options.power || 1,
    source: options.source || "environment",
    initialDuration: options.duration || defaultDurations[effectId],
    
  };

  activeEffects.push(newEffect);
  console.log(`🌀 Hiệu ứng [${effectId}] stack mới (${sameType.length + 1}/${maxStacks})`);
}

function updateEffects() {
  const now = Date.now();
  const resist = calcFinalStat(player.stats.Khang || 0, "Khang") || 0;
  const resistFactor = 1 - Math.min(0.8, resist / 100); // giảm damage tối đa 80%

  activeEffects = activeEffects.filter(e => {
    if (now >= e.expires) return false;

    if (now - e.lastTick >= e.tickInterval) {
      e.lastTick = now;

      let dmg = 0, label = "";
      switch (e.type) {
        case "burn": dmg = 2 * e.power; label = "🔥 Bỏng"; break;
        case "poison": dmg = 1.5 * e.power; label = "☠️ Độc"; break;
        case "freeze": dmg = 1 * e.power; label = "❄️ Lạnh cóng"; break;
      }

      dmg = Math.ceil(dmg * resistFactor);
      if (dmg > 0) modifyHP(-dmg, label);
    }
    return true;
  });
}


function safeNum(v, d = 0) {
  const n = Number(v);
  return isNaN(n) ? d : n;
}

function getMaxHP() {
  const base = safeNum(player.stats.MaxHP, 100);
  const theChat = calcFinalStat(player.stats.TheChat || 0, "TheChat");
  return base + theChat * 10; // Thể Chất mỗi điểm +10 HP
}
function getMaxMana() {
  const base = safeNum(player.stats.MaxMana, 50);
  const triLuc = calcFinalStat(player.stats.TriLuc || 0, "TriLuc");
  return base + triLuc * 5; // Trí Lực mỗi điểm +5 Mana
}
function getMaxStamina() {
  const base = safeNum(player.stats.MaxSucBen, 20);
  const theChat = calcFinalStat(player.stats.TheChat || 0, "TheChat");
  return base + Math.floor(theChat / 2); // Thể Chất ảnh hưởng nhẹ thể lực
}
// === Hồi phục chỉ số chung, tự động dừng khi đầy ===
function recoverStat(statKey, amount) {
  if (!player || !player.stats) return;
  const maxKey = "Max" + statKey;
  const current = Number(player.stats[statKey]) || 0;
  const max = Number(player.stats[maxKey]) || 0;
  if (current >= max) return; // đầy rồi thì bỏ qua

  player.stats[statKey] = Math.min(max, current + amount);
}

function regenerationTick() {
  if (!player || !player.stats) return;

  // Nếu nhân vật đã chết thì không hồi
  if (player.stats.HP <= 0) return;

  const maxHP = getMaxHP();
  const maxMana = getMaxMana();
  const maxSt = getMaxStamina();

  const theChat = calcFinalStat(player.stats.TheChat || 0, "TheChat");
  const triLuc = calcFinalStat(player.stats.TriLuc || 0, "TriLuc");

  // --- Hệ số hồi cơ bản ---
  let hpMult = 1.0, manaMult = 1.0, stMult = 1.0;

  // ⚔️ Giảm hồi khi đang chiến đấu
  if (player.state === "combat") {
    hpMult *= 0.5;
    manaMult *= 0.7;
    stMult *= 0.6;
  }

  // 🧘 Tăng hồi khi ngồi thiền
  if (player.state === "meditating") {
    hpMult *= 2.0;
    manaMult *= 2.5;
    stMult *= 1.8;
  }

  // 💀 Giảm hồi khi bị trọng thương
  if (player.debuffs?.includes("bleeding")) hpMult *= 0.5;
  if (player.debuffs?.includes("poison")) hpMult *= 0.7;
  if (player.debuffs?.includes("manaBurn")) manaMult *= 0.5;
  if (player.debuffs?.includes("fatigue")) stMult *= 0.5;

  // 🩸 Buff tăng hồi phục
  if (player.buffs?.includes("regenAura")) hpMult *= 1.5;
  if (player.buffs?.includes("manaFlow")) manaMult *= 1.8;
  if (player.buffs?.includes("ironWill")) stMult *= 1.4;

  // --- Tính lượng hồi thực tế ---
  const regenHP = Math.max(1, Math.floor((maxHP * 0.005 + theChat * 0.2) * hpMult));
  const regenMana = Math.max(1, Math.floor((maxMana * 0.004 + triLuc * 0.15) * manaMult));
  const regenSt = Math.max(1, Math.floor((maxSt * 0.01 + theChat * 0.1) * stMult));

  // --- Hồi từng thanh, dừng nếu đầy ---
  if (player.stats.HP < maxHP)
    player.stats.HP = Math.min(maxHP, safeNum(player.stats.HP, maxHP) + regenHP);

  if (player.stats.Mana < maxMana)
    player.stats.Mana = Math.min(maxMana, safeNum(player.stats.Mana, maxMana) + regenMana);

  if (player.stats.SucBen < maxSt)
    player.stats.SucBen = Math.min(maxSt, safeNum(player.stats.SucBen, maxSt) + regenSt);

  // --- Cập nhật giao diện ---
  if (typeof safeUpdateBars === "function") safeUpdateBars();
  if (typeof refreshPlayerUI === "function") refreshPlayerUI();
}

// Gọi tự động mỗi 5 giây
setInterval(regenerationTick, 5000);



function generateEnemy(type="normal"){
  const playerLv = player.stats.Level || 1;
  const worldType = window.currentWorldType || "tieu";
  const worldMul = worldType==="dai" ? 2.5 : (worldType==="trung" ? 1.5 : 1);
  const level = Math.max(1, playerLv + (type==="boss" ? 3 : (Math.random()*2-1)));
  
  const hp = Math.floor((50 + level*12) * worldMul * (type==="boss"?2:1));
  const atk = Math.floor((8 + level*2) * worldMul * (type==="boss"?1.5:1));
  const def = Math.floor((5 + level*1.2) * worldMul * (type==="boss"?1.3:1));
  
  // xác suất có công pháp
  const hasTech = Math.random() < (type==="boss"?0.6:0.2);
  let tech = null;
  if (hasTech && gameData.techniques) {
    tech = gameData.techniques[Math.floor(Math.random() * gameData.techniques.length)];
  }

  return { 
    name: type==="boss" ? "Boss Cổ Thần" : "Quái Vật", 
    type, level,
    stats:{HP:hp,Atk:atk,Def:def}, 
    tech,
    element: type==="boss"?"🔥":"☠️",
    alive:true 
  };
}

function showCombatUI(enemy){
  // Xóa UI cũ nếu có
  const old = document.getElementById("combatUI");
  if(old) old.remove();

  const wrap = document.createElement("div");
  wrap.id = "combatUI";
  Object.assign(wrap.style,{
    position:"fixed",left:"0",top:"0",width:"100%",height:"100%",
    background:"rgba(0,0,0,0.85)",zIndex:"15000",
    display:"flex",flexDirection:"column",justifyContent:"space-between",
    alignItems:"center",color:"#fff"
  });

  // --- Trên: tên và thanh máu ---
  const top = document.createElement("div");
  Object.assign(top.style,{width:"100%",padding:"12px",textAlign:"center"});
  top.innerHTML = `
    <div style="font-size:20px">⚔️ ${enemy.name} (Lv ${enemy.level})</div>
    <div style="background:#333;height:10px;border-radius:8px;margin-top:4px;overflow:hidden;">
      <div id="enemyHpBar" style="background:#ff4747;width:100%;height:100%;transition:width 0.2s;"></div>
    </div>`;
  wrap.appendChild(top);

  // --- Giữa: nhân vật vs quái ---
  const mid = document.createElement("div");
  Object.assign(mid.style,{flex:"1",display:"flex",alignItems:"center",justifyContent:"space-around"});
  mid.innerHTML = `
    <div id="playerSide" style="font-size:60px;">👤</div>
    <div id="vsText" style="font-size:30px;">VS</div>
    <div id="enemySide" style="font-size:60px;">💀</div>`;
  wrap.appendChild(mid);

  // --- Dưới: nút tấn công & kỹ năng ---
  const bottom = document.createElement("div");
  Object.assign(bottom.style,{
    width:"100%",padding:"8px",display:"flex",
    justifyContent:"center",gap:"8px",flexWrap:"wrap",marginBottom:"12px"
  });

  // Nút đánh thường
  const atkBtn = document.createElement("button");
  atkBtn.innerText = "🗡️ Tấn công";
  Object.assign(atkBtn.style,{padding:"10px 18px",fontSize:"18px",borderRadius:"8px",cursor:"pointer"});
  atkBtn.onclick = ()=> playerAttack(enemy);
  bottom.appendChild(atkBtn);

  // Kỹ năng trang bị
  equippedSkills.forEach(id=>{
    if(!id) return;
    const sk = gameData.skills.find(s=>s.id===id);
    if(!sk) return;
    const btn = document.createElement("button");
    btn.innerText = sk.name;
    Object.assign(btn.style,{padding:"8px 12px",fontSize:"14px",borderRadius:"6px",cursor:"pointer"});
    btn.onclick = ()=> playerAttack(enemy,sk);
    bottom.appendChild(btn);
  });

  wrap.appendChild(bottom);
  document.body.appendChild(wrap);
}

function playerAttack(enemy, skill=null){
  if(!enemy.alive) return;
  const dmg = calcDamage(player.stats.TheChat || 10, enemy.stats.Def, !!skill);
  enemy.stats.HP -= dmg;
  document.getElementById("enemyHpBar").style.width = Math.max(0, enemy.stats.HP / 100 * 100) + "%";
  console.log(`💥 Bạn gây ${dmg} sát thương cho ${enemy.name}`);

  if(enemy.stats.HP <= 0){
    enemy.alive = false;
    console.log("🏆 Đã hạ gục kẻ địch!");
    endCombat(enemy);
    return;
  }

  setTimeout(()=>{
    const enemyDmg = calcDamage(enemy.stats.Atk, player.stats.Khang || 5);
    player.stats.HP = Math.max(0, player.stats.HP - enemyDmg);
    console.log(`😵 ${enemy.name} phản công gây ${enemyDmg} sát thương!`);
    safeUpdateBars();
    if(player.stats.HP <= 0){
      console.log("💀 Bạn đã bại trận...");
      document.getElementById("combatUI").remove();
    }
  }, 600);
}

function endCombat(enemy){
  document.getElementById("combatUI")?.remove();
  applyCombatResult(enemy);
}

function resetGame(){
      if(confirm("Bạn có chắc chắn muốn reset dữ liệu?")){
        try{ localStorage.removeItem('rpgSave'); }catch(e){}
        player = JSON.parse(JSON.stringify(playerDefault));
                saveGame();
        location.reload();
      }
    }


document.getElementById("btnMinus10").addEventListener("click", minusTenPercent);

function minusTenPercent() {
  // Giả sử bạn lưu max HP, Mana, Stamina trong player.stats
  const lostHP   = Math.floor(player.stats.MaxHP * 0.1);
  const lostMana = Math.floor(player.stats.MaxMana * 0.1);
  const lostSta  = Math.floor(player.stats.MaxSucBen * 0.1);

  // Trừ đi, không để âm
  player.stats.HP       = Math.max(0, player.stats.HP - lostHP);
  player.stats.Mana     = Math.max(0, player.stats.Mana - lostMana);
  player.stats.SucBen  = Math.max(0, player.stats.SucBen - lostSta);

  // Hiện thông báo
  $('modal').style.display = 'flex';
  $('modalCard').innerHTML = `
    <h3>Thông báo</h3>
    <p>-10% HP (${lostHP}), -10% Mana (${lostMana}), -10% Stamina (${lostSta})</p>
    <p>Còn lại: HP ${player.stats.HP}, Mana ${player.stats.Mana}, Stamina ${player.stats.SucBen}</p>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button onclick="closeModal()">Đóng</button>
    </div>`;

  renderStats();
  updateBars();
  saveGame();
}


    function init(){
      loadGame();
      if(!player) player = JSON.parse(JSON.stringify(playerDefault));
      if(!player.xpToNext) player.xpToNext = 100;
     // after loadGame()
player.skillSlots = player.skillSlots || Array(10).fill(null); // ensure slots exist
resumeTrainingIfAny(); // resume offline training if any
      renderInventory(); renderSkills(); renderTechniques(); renderStats(); updateHeader(); drawCharacterBase(); renderEquipmentSlots(); loadGame(); animateChar(); updateBars(); 

      
      // nav
      $('btnHome').onclick = ()=> { removeDungeonControls(); showView('start'); };
$('btnInv').onclick = ()=> { removeDungeonControls(); showView('inv'); };
$('btnSkills').onclick = ()=> { removeDungeonControls(); showView('skills'); };
$('btnMap').onclick = ()=> { 
  // nếu chuyển tới map, showView sẽ tái tạo controls nếu đang ở dungeon
  showView('map'); 
};
      $('btnReset').onclick = resetGame;

      document.querySelectorAll('.equip-slot').forEach(el=>{ el.addEventListener('click', ()=> openEquipSlot(el.dataset.slot)); });
            const modalCard = $('modalCard'); if(modalCard) modalCard.addEventListener('click', (ev)=> ev.stopPropagation());
      if(!$('invMax').innerText) $('invMax').innerText = 24;

      // dropdown filter listeners
      const filterToggle = $('filterToggle'); const filterOptions = $('filterOptions');
      filterToggle && filterToggle.addEventListener('click', (e)=>{ e.stopPropagation(); filterOptions.style.display = filterOptions.style.display === 'block' ? 'none' : 'block'; });
      filterOptions && filterOptions.querySelectorAll('button').forEach(btn=>{
        btn.addEventListener('click', ()=>{
          currentInvFilter = btn.dataset.filter || 'all';
          const labelMap = { 
  all: 'Tất cả', 
  technique: 'Công pháp', 
  skill: 'Kỹ năng', 
  equip: 'Trang bị', 
  potion: 'Đan dược', 
  other: 'Khác' 
}
          
          const filt = $('invFilter'); if(filt) filt.innerText = labelMap[currentInvFilter] || currentInvFilter;
          filterOptions.style.display = 'none';
          renderInventory();
        });
      });

      // close filter when clicking outside
      document.addEventListener('click', (e)=>{ if(!filterOptions.contains(e.target) && !filterToggle.contains(e.target)){ filterOptions.style.display = 'none'; } });

      saveGame();
    }

    window.onload = init;
  document.getElementById("btnAdd10Lv").onclick = add10Levels;
    window.openItem = openItem; window.useItem = useItem; window.closeModal = closeModal; window.viewSkill = viewSkill; window.trainSkill = trainSkill; window.openEquipSlot = openEquipSlot; window.learnTechnique = learnTechnique;
  

  </script></body>
</html>
