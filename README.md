<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Игровой сайт</title>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #1a1a1a; color: white; }
    header { background: #222; padding: 15px; text-align: center; font-size: 22px; font-weight: bold; }
    nav { display: flex; background: #333; }
    nav button { flex: 1; padding: 15px; background: #333; border: none; color: white; font-size: 18px; cursor: pointer; }
    nav button.active { background: #444; }
    .page { display: none; padding: 20px; }
    .page.active { display: block; }
    #logo { font-size: 40px; text-align: center; margin-top: 40px; }
    #loginBox { margin: 40px auto; width: 280px; padding: 20px; background: #2b2b2b; border-radius: 8px; }
    input { width: 100%; padding: 10px; margin-top: 10px; border: none; border-radius: 5px; color: black; }
    #loginBtn { margin-top: 15px; width: 100%; padding: 12px; background: #4CAF50; border: none; color: white; border-radius: 5px; cursor: pointer; font-size: 16px; }
    #mapControls { display:flex; gap:8px; align-items:center; margin-bottom:10px; flex-wrap:wrap; }
    .floorBtn { padding:8px 12px; border-radius:6px; border:1px solid #666; background:#444; color:white; cursor:pointer; font-weight:bold; }
    .floorBtn.active { background:#5a9; color:black; border-color:#3c6; }
    .myFloorBtn { padding:6px 10px; border-radius:6px; border:1px dashed #888; background:#222; color:white; cursor:pointer; }

    #mapWrapper { width: 100%; height: 500px; overflow: hidden; position: relative; border-radius:6px; background:#111; }
    /* mapArea is the large inner surface that we translate for panning */
    #mapArea { width: 1600px; height: 1600px; background: #0f0f0f; border: 2px solid #555; position: absolute; left: 0; top: 0; touch-action: none; will-change: transform; }

    .playerMarker {
      width: 48px;
      height: 48px;
      position: absolute;
      transform: translate(-50%, -50%);
      background-size: cover;
      background-position: center;
      border-radius: 50%;
      border: 2px solid white;
      box-shadow: 0 2px 6px rgba(0,0,0,0.6);
      transition: left 200ms linear, top 200ms linear;
    }
    .markerAvatar {
  position: absolute;
  bottom: 55px;
  left: 50%;
  transform: translateX(-50%);
  width: 64px;
  height: 64px;
  background-size: cover;
  background-position: center;
  border-radius: 8px;
  border: 2px solid white;
  display: none;
  background-color: #000;
  z-index: 10;
}
.playerMarker:hover .markerAvatar {
  display: block;
}


    #sheetWrapper { width: 100%; overflow: auto; touch-action: pinch-zoom pan-x pan-y; }
    #characterSheet img { width: 100%; max-width: none; }

    /* Small info row */
    #mapInfo { margin-top:8px; color:#ccc; font-size:14px; display:flex; gap:10px; align-items:center; flex-wrap:wrap; }

    @media (max-width: 600px) {
      header { font-size: 18px; padding: 10px; }
      nav button { font-size: 14px; padding: 10px; }
      #logo { font-size: 28px; margin-top: 20px; }
      #loginBox { width: 90%; padding: 15px; }
      input { padding: 8px; font-size: 14px; }
      #loginBtn { padding: 10px; font-size: 14px; }

      #mapWrapper { height: 420px; }
      #mapArea { width: 2000px; height: 2000px; } /* a bit larger for mobile */
    }
    /* ===== DANGANRONPA RULES ===== */

#rules {
  background: radial-gradient(circle at top, #2a0000, #000);
  min-height: 100vh;
}

.rulesPad {
  max-width: 700px;
  margin: 40px auto;
  background: #0b0b0b;
  border: 3px solid #8b0000;
  border-radius: 14px;
  padding: 20px;
  box-shadow:
    0 0 20px rgba(180,0,0,0.6),
    inset 0 0 30px rgba(120,0,0,0.4);
}

.rulesTitle {
  text-align: center;
  font-size: 32px;
  color: #ff2b2b;
  text-shadow: 2px 2px 0 #000, 0 0 10px red;
  margin-bottom: 10px;
}

.rulesSubtitle {
  text-align: center;
  font-size: 14px;
  color: #ff9a9a;
  font-style: italic;
  margin-bottom: 20px;
}

.rulesText {
  color: #ff3b3b;
  font-size: 17px;
  line-height: 1.6;
  text-shadow: 1px 1px 0 #000, 0 0 6px #900;
}

.rulesText p {
  margin: 10px 0;
}

.rulesWarning {
  margin-top: 15px;
  text-align: center;
  font-size: 13px;
  color: #ff7777;
}
.bulletItem {
  display:flex;
  align-items:center;
  gap:10px;
  background:#2b2b2b;
  padding:8px 10px;
  border-radius:6px;
  margin-bottom:8px;
  cursor:pointer;
}
.bulletEmoji {
  font-size:22px;
}
.bulletDelete {
  margin-left:auto;
  cursor:pointer;
  color:#f66;
  font-weight:bold;
}

  </style>
</head>
<body>
<header>Монопад</header>
<nav>
  <button onclick="openPage('home')" class="active" id="tab_home">Главная</button>
  <button onclick="openPage('map')" id="tab_map">Карта</button>
  <button onclick="openPage('info')" id="tab_info">Персонаж</button>
  <button onclick="openPage('rules')" id="tab_rules">Правила</button>
  <button onclick="openPage('bullets')" id="tab_bullets">Улики</button>
</nav>

<div class="page active" id="home">
  <div id="logo">ЛОГО</div>
  <div id="loginBox">
    <input id="login_input" placeholder="Логин" />
    <input id="pass_input" placeholder="Пароль" type="password" />
    <button id="loginBtn" onclick="login()">Войти</button>
    <div id="login_status" style="margin-top:10px;color:#f55;"></div>
  </div>
</div>

<div class="page" id="map">
  <h2>Карта</h2>

  <!-- Управления этажами (вид) и управления своим этажом -->
  <div id="mapControls">
    <!-- большие иконки / кнопки этажей (view) -->
    <button class="floorBtn" id="view_floor_1">1 этаж</button>
    <button class="floorBtn" id="view_floor_2">2 этаж</button>
    <button class="floorBtn" id="view_floor_b">Подвал</button>

    <!-- разделитель и управление своим этажом -->
    <div style="width:12px"></div>
    <span style="color:#ccc">Мой этаж:</span>
    <button class="myFloorBtn" id="set_floor_1">1</button>
    <button class="myFloorBtn" id="set_floor_2">2</button>
    <button class="myFloorBtn" id="set_floor_b">B</button>

    <div style="flex:1"></div>
    <div id="mapInfo">Просмотр: <strong id="viewFloorLabel">1 этаж</strong> · Мой этаж: <strong id="myFloorLabel">не выбран</strong></div>
  </div>

  <div id="mapWrapper">
    <div id="mapArea"></div>
  </div>
</div>

<div class="page" id="info">
  <h2>Информация о персонаже</h2>
  <div id="infoBox">Вы не вошли в аккаунт.</div>
  <div id="infoContent">
    <div id="leftPanel">
      <div id="avatarBox"></div>
      <div id="descriptionBox"></div>
    </div>
    <div id="sheetWrapper">
      <div id="characterSheet"></div>
    </div>
  </div>
</div>
<div class="page" id="rules">
  <div class="rulesPad">

    <div class="rulesTitle">RULES OF THE GAME</div>

    <div class="rulesSubtitle">
      Monokuma is watching. Break the rules — face despair.
    </div>

    <div class="rulesText">
      <p>1. более 2-х убийств совершенных одним человеком- запрещено</p>
      <p>2. После обнаружения тела начинается расследование.</p>
      <p>3. Суд обязателен для всех участников.</p>
      <p>4. Если виновный не будет разоблачен, выживет только он.</p>
      <p>5. Нарушение правил наказывается немедленно. </p>
      <p>6 попытки уничтожить монопад запрещены</p>
      <p>7 монокума не может быть задействован в убийстве</p>
    </div>

    <div class="rulesWarning">
      ⚠ новые правила могут быть добавлены в любой момент.
    </div>

  </div>
</div>
<div class="page" id="bullets">
  <h2>Пули правды</h2>

  <button onclick="openAddBullet()" style="font-size:20px;padding:6px 12px;cursor:pointer;">
    ➕ Добавить пулю
  </button>

  <div id="bulletsList" style="margin-top:15px;"></div>
</div>

<!-- Окно добавления / просмотра пули -->
<div id="bulletModal" style="
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.7);
  align-items:center;
  justify-content:center;
  z-index:1000;
">
  <div style="
    background:#222;
    padding:20px;
    width:90%;
    max-width:400px;
    border-radius:8px;
  ">
    <h3 id="bulletModalTitle">Новая пуля</h3>

    <input id="bulletName" placeholder="Название" style="margin-bottom:8px;">
    <textarea id="bulletDesc" placeholder="Описание / почему это улика" style="width:100%;height:80px;margin-bottom:8px;"></textarea>
    <input id="bulletEmoji" placeholder="Эмодзи (например 🔫 🧠 📄)" style="margin-bottom:10px;">

    <div style="display:flex;gap:10px;justify-content:flex-end;">
      <button onclick="closeBulletModal()">Закрыть</button>
      <button onclick="saveBullet()">Сохранить</button>
    </div>
  </div>
</div>

<!-- Firebase CDN -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
/* =======================
   CONFIG — вставь свои URL карт для этажей
   Пример:
   mapImages[1] = 'https://site.com/floor1.png';
   mapImages[2] = 'https://site.com/floor2.png';
   mapImages['b'] = 'https://site.com/basement.png';
   Оставь пустые строки если хочешь вставить позже.
   ======================= */
const mapImages = {
  1: "https://i.postimg.cc/CMfrDjLK/Znimok-ekrana-2025-12-12-222248.png",      // <- вставь ссылку на изображение первого этажа
  2: "https://i.postimg.cc/3wBv6rC3/izobrazenie-2025-12-13-010749253.png",      // <- вставь ссылку на изображение второго этажа
  b: "https://i.postimg.cc/pTSQr1nK/Znimok-ekrana-2025-12-12-230210.png"       // <- вставь ссылку на изображение подвала (ключ 'b' = basement)
};

/* ============ Firebase init (твои данные уже стояли) ============ */
const firebaseConfig = {
  apiKey: "AIzaSyBiHWlMDeUv5FmPh4Aqv7aKCGHFbco5YIM",
  authDomain: "dandd-592cb.firebaseapp.com",
  databaseURL: "https://dandd-592cb-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "dandd-592cb",
  storageBucket: "dandd-592cb.appspot.com",
  messagingSenderId: "354514314558",
  appId: "1:354514314558:web:e81c95d0ae2a4c5b9bb86f",
  measurementId: "G-2KGQMKJP4X"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

/* ============ Переменные состояния ============ */
let currentUser = null;      // username
let viewFloor = 1;           // какой этаж показываем (1 / 2 / 'b')
let myFloor = null;          // этаж текущего игрока (1 / 2 / 'b') — синхронизируется в базе
const mapWrapper = document.getElementById('mapWrapper');
const mapArea = document.getElementById('mapArea');

/* Параметры панинга (перетаскивания) */
let offsetX = 0, offsetY = 0;
let isDragging = false;
let dragStartX = 0, dragStartY = 0;
let startOffsetX = 0, startOffsetY = 0;

/* Словарь маркеров DOM по username */
const markers = {};

/* Аккаунты — оставил твои данные, можно редактировать */
const accounts = [
    { username: "цумуги", displayName: "Цумуги Широгане", password: "0387", color: "blue", avatar: "https://i.postimg.cc/MKDL3Brk/IMG-20251212-023652-731.jpg",markerAvatar: "", description: "ыыыы", sheet: "" },
    { username: "нагито", displayName: "Нагито Комаэда", password: "1492", color: "white", avatar: "",markerAvatar: "", description: "", sheet: "" },
    { username: "миу", displayName: "Миу Ирума", password: "2750", color: "pink", avatar: "https://i.postimg.cc/pXPP3DMm/IMG-20260207-215033-607.jpg",markerAvatar: "https://i.postimg.cc/PJQTPqYt/IMG-20260207-215010-115.png", description: "Демоничный колдун.                                                                                                                                                       миу родилась человеком в бедной семье ремесленников. С юности отличалась дерзким характером и желанием доказать своё превосходство над другими. Когда в ней проявились магические задатки, она поступила в столичную академию и быстро освоила азы колдовства. Но, даже закончив обучение, Миу чувствовала, что её сил недостаточно.Гонимая жаждой могущества, она обратилась к древним ритуалам тифлингов и заключила договор с низшим демоном Ignus, рождённым из сущности Терры. Сделка изменила её навсегда: её тело стало носителем демонической крови, а в глазах загорелся огонь пламени. Так человек превратился в тифлинга, заклеймённого собственной жадностью до силы.Магия Миу сочетает в себе дерзость и практичность. Она свободно пользуется простыми заговорами — «Волшебная рука», «Чудотворство», «Ядовитые брызги», — а также владеет более сильными чарами: «Очарование личности», чтобы подчинять волю, и «Защита от зла и добра», чтобы отражать чужое тёмное влияние.Миу презирает тех, кто называет её проклятой или продажной: для неё это лишь доказательство её смелости. Она гордится тем, что решилась на то, чего другие боятся — отказаться от человечности ради настоящей силы.", sheet: "https://i.postimg.cc/fL58ynkQ/IMG-20260208-001546-912.jpg" },
    { username: "каэде", displayName: "Каэде Акаматсу", password: "4069", color: "yellow", avatar: "",markerAvatar: "https://i.postimg.cc/13S8Z1sp/IMG-20260108-122721-302.png", description: "", sheet: "" },
    { username: "рантаро", displayName: "Рантаро Амами", password: "5821",  avatar: "https://i.postimg.cc/XqxgLf7X/IMG-20251212-024217-121.png",markerAvatar: "", description: `Падший паладин.

В белоснежном королевстве, где магия — не дар, а закон, имя Стеллы Бетельгейм произносят, склоняя голову.
Она — богиня магии, сияние знания, что подчинило себе разум и волю человечества. Её культ — это не вера, а строй: каждый ребёнок с детства учится различать «чистую» магию от «еретической», каждый взрослый живёт под взором храмов, что возвышаются над улицами, будто сторожевые башни.

Рантаро не родился — его вызвали из пустоты.
Чужеземец, появившийся на снегу монастырского двора, без воспоминаний, без имени. Маги нарекли его сосудом Стеллы, созданным для того, чтобы хранить её магию в человеческом теле. Так он стал паладином света, хранителем порядка и проводником её воли.

Ему вручили длинное копьё — символ её пронизывающей мощи. Это было не просто оружие, а жезл света, способный проводить заклинания через сталь. Рантаро чувствовал, как магия богини течёт по древку, будто само дерево и металл поют её имя.

Годы он исполнял свой долг: карал тех, кто использовал магию без благословения церкви, жёг книги, закрывал уста. И всё же — каждое заклинание, каждый удар копья оставляли в его сердце трещины.

Однажды, среди руин деревни, сожжённой за «осквернение стихии», он нашёл ребёнка, что чертил знаки на снегу.
Знаки были неловкие, но живые. Магия ребёнка не разрушала — она созидала.
Тогда сосуд впервые понял: в магии нет света или тьмы — есть только воля того, кто её творит.

С тех пор Рантаро стал отступником.
Он ушёл из храмов, снял свой шлем и покинул белые города. Свет Стеллы больше не греет его — но он и не отринул её. Он верит, что богиня магии не зла и не добра, просто она равнодушна, как снег, падающий и на храм, и на руины.

Теперь Рантаро странствует по северным дорогам, скрываясь под белым плащом. Его копьё — это не инструмент веры, а напоминание о ней.
И когда ветер поёт над ледяными равнинами, кажется, что он слышит шёпот самой Стеллы —
или, может быть, самого себя, наконец ставшего живым.`, sheet: "https://i.postimg.cc/J44xn2Pw/IMG-20251212-024232-021.jpg" },
    { username: "чихиро", displayName: "Чихиро Фуджисаки", password: "6825", color: "brown", avatar: "https://i.postimg.cc/KYSvH4HK/IMG-20260126-174934-610.png",markerAvatar: "https://i.postimg.cc/BvrDNqxR/IMG-20260126-174934-578.png", description: "Тихий хранитель.                                       Чихиро родился в зелёных лесах эльфийского королевства, где каждое дерево старше любого из мудрецов, а магия течёт сквозь землю так же естественно, как кровь сквозь вены. Эльфы жили в согласии с природой, под покровительством Сириуса — богини всего живого, что учит видеть жизнь во всём, даже в мху на камне.Он вырос в эпоху перемен: Парламент Мудрецов готовился к коронации новой королевы Сонии — монархини, призванной не править, а напоминать эльфам о единстве. В те годы Чихиро служил при местном святилище как прислужник-друид, ухаживая за рощами, животными и источниками, где вода считалась «дыханием Сириуса».Он был робким, тихим и застенчивым юношей. Часто избегал разговоров, предпочитая слушать треск ветра и шелест листьев. Его мир — это не города и речи Мудрецов, а запах земли после дождя, трепетное движение травы под солнцем.Сослужители смеялись: «Чихиро ближе к оленю, чем к эльфу».Но никто не мог отрицать, что природа отвечала ему взаимностью — растения оживали под его руками, звери не боялись подходить близко.Когда начались волнения на северных границах — там, где людские караваны нарушали покой священных земель, — Чихиро не взял оружие. Он ушёл туда один, чтобы умолить природу не ответить яростью на ярость.С тех пор он странствует между рощами, помогая тому, что живо, и избегая того, что жадно.Он не ищет славы и не ждёт награды. Он просто живёт по завету своей богини:«Слушай, не властвуй. Береги, не приказывай. Живи — и давай жить.»", sheet: "https://i.postimg.cc/q7x0Dwnf/IMG-20260127-181059-011.jpg" },
    { username: "кай", displayName: "Кай Монтего", password: "6904", color: "purple", avatar: "https://i.postimg.cc/GhK4rS2J/IMG-20260108-122721-262.png",markerAvatar: "https://i.postimg.cc/Kjc0WHj9/IMG-20260108-122721-404.png", description: `Амбициозный мошенник.

Кай Монтего родился в семье известных актёров в эльфийском королевстве, где власть избирают мудрецы, леса бесконечны, а вера в Сириус является нормой, а не обязанностью. Его дом с детства был наполнен репетициями, голосами, инструментами и шумом сцены — он вырос в мире, где каждый говорит громче, чем думает, где важно не что ты чувствуешь, а как это выглядит со стороны.

Он рано начал выступать — сначала в детских постановках, позже в больших театрах столицы, где его запоминали за энергичность и смелость. Кай обожал внимание публики, зависел от него, как растение от солнца. Когда зрители аплодировали — он чувствовал себя великим. Когда молчали — казалось, что внутри него пусто. Именно тогда он впервые попробовал другое искусство — искусство обмана.

Во время гастролей он заметил, что люди охотнее раскрывают кошельки, если верят не артисту, а другу. Он стал играть роли прямо на улицах — потерянного отпрыска, доверчивого новичка, случайного прохожего, которому «нужна помощь». Так он открыл в себе плута: ловкого в словах, остроумного, умеющего подстраиваться и менять личности как костюмы.

Слава росла, но вместе с ней рос и внутренний страх быть разоблачённым. Кай всё чаще критиковал себя, считал, что без масок он пуст, а талант — всего лишь удачная иллюзия. Родители любили его, но он всегда думал, что должен стать лучше, ярче, громче, чтобы их гордость была заслуженной, а не автоматической.

Он участвовал в спектаклях столицы, путешествовал между городами, выступал в парламенте мудрецов на культурных фестивалях. Иногда за кулисами пропадали драгоценности, иногда зрители уходили с облегчёнными кошельками — но никто не мог связать это с улыбчивым юным артистом. Он был хорош, слишком хорош, чтобы сомненье задержалось надолго.

Однажды, после успешного выступления на летнем празднике, Кая пригласили на частную встречу. Незнакомец наблюдал за ним долго, изучал поведение, эмоции, слабости. Тогда Кай впервые услышал предложение участвовать в игре — опасной, закрытой, смертельной. Но в этом была слава, внимание, шанс войти в историю. Не как актёр — как легенда.

Он согласился почти без размышлений, ощущая азарт, предвкушая громкие овации, которые услышат даже за пределами королевства.

Он не знал, что организатор видел в нём удобную марионетку — слишком яркую, чтобы не заметили, и слишком зависимую от признания, чтобы отказаться. Теперь судьба Кая — больше не только театр. Это сцена, где цена спектакля — жизнь.`, sheet: "https://i.postimg.cc/pLyrSWTk/sheet7.jpg" },
    { username: "кокичи", displayName: "Кокичи Ома", password: "7318", color: "violet", avatar: "https://i.postimg.cc/yxSZg0gm/IMG-20251212-030500-800.png",markerAvatar: "", description: `Голос Обмана+

Настоящее имя неизвестно.
На сцене его знали под сотней масок, но имя Кокичи Ома он выбрал сам — в честь мертвого героя, которого никто, кроме него, не помнит.

Он вырос в балагане на краю империи: фальшивые чудеса, поддельные пророчества, ядовитый грим и настоящая нищета. Родителей заменили актёры, друзей — роли. Искренность высмеивалась. Ложь — поощрялась. Там он научился: правда не ведёт за собой толпу, но хорошая история — всегда.

С малых лет он играл бога, дракона, шутника, мертвеца, вождя.
Но однажды он сорвал с себя все образы — и остался пуст. Бросив труппу, он ушёл в мир, чтобы играть теперь уже свою собственную игру.

Он лжёт, чтобы смешить. Он провоцирует, чтобы люди увидели себя. Он добр, но никогда прямо. Он обманщик — но не предатель.
Для него ложь — кисть, а реальность — холст. И если придётся перевернуть мир, чтобы сделать его красивее — он не задумываясь это сделает.`, sheet: "https://i.postimg.cc/63pV37fD/IMG-20251212-024143-183.jpg" },
    { username: "изуру", displayName: "Изуру Камукура", password: "8046", color: "gray", avatar: "https://i.postimg.cc/mkQrWjF2/IMG-20251212-025433-144.png",markerAvatar: "", description: `Уединение. Осознание.

Изуру Камукура родился среди эльфов, но его происхождение тщательно скрыто. Ни одного подтверждённого факта о его семье, детстве или наставниках не существует.

В возрасте, не указанном ни в одном эльфийском реестре, он ушёл в горы — один. Причины ухода не названы. Добровольное изгнание, без следов и записей.

В течение 17 лет находился в полной изоляции. Единственное, что известно о его деятельности в этот период — фрагментарные находки: философские заметки без автора, рисунки из символов, совпадающих с древними молитвами, выжженные на камне круги.

Первое появление в обществе — случайное. Он спас умирающего ребёнка, не назвав имени, и исчез до рассвета. Потом снова — в разрушенном храме, где тела лежали нетронутыми, а стены были очищены.

Он не упоминает богов, но его заклинания работают. Он говорит редко, чаще — наблюдает. Не вступает в конфликты, если в этом нет необходимости. Его вера — не в существо, а в порядок или хаос как природное явление.

Именем Изуру Камукура пользуется сам. Считается, что оно не настоящее. Личность до изоляции не установлена. Возраст не определён.

Его цель неясна. Его путь не объясняется. Он просто идёт вперёд.`, sheet: "https://i.postimg.cc/4x4xjzfV/IMG-20251212-025454-723.jpg" },
    { username: "монака", displayName: "Монака Това", password: "9163", color: "lightgreen", avatar: "https://i.postimg.cc/J4vp2Q57/IMG-20251212-025905-402.png",markerAvatar: "", description: `Святое притворство:

Когда-то в глухой долине, за чёрными болотами и зловещим холмом стоял приют под названием Дом Милосердия Элании. Местные шептались, что дети туда попадают не по воле богов, и что сама земля вокруг приюта плачет ночью. Именно там, в дождливую ночь, была найдена крошечная девочка — наполовину эльфийка, с чёртами лица столь хрупкими, что она казалась игрушечной.
Её назвали Монакой.
С юных лет она проявляла необычную проницательность.     #loginBtn { margin-top: 15px; width: 100%; padding: 12px; background: #4CAF50; border: none; color: white; border-radius: 5px; cursor: pointer; font-size: 16px; }
    #mapControls { display:flex; gap:8px; align-items:center; margin-bottom:10px; flex-wrap:wrap; }
    .floorBtn { padding:8px 12px; border-radius:6px; border:1px solid #666; background:#444; color:white; cursor:pointer; font-weight:bold; }
    .floorBtn.active { background:#5a9; color:black; border-color:#3c6; }
    .myFloorBtn { padding:6px 10px; border-radius:6px; border:1px dashed #888; background:#222; color:white; cursor:pointer; }

    #mapWrapper { width: 100%; height: 500px; overflow: hidden; position: relative; border-radius:6px; background:#111; }
    /* mapArea is the large inner surface that we translate for panning */
    #mapArea { width: 1600px; height: 1600px; background: #0f0f0f; border: 2px solid #555; position: absolute; left: 0; top: 0; touch-action: none; will-change: transform; }

    .playerMarker {
      width: 48px;
      height: 48px;
      position: absolute;
      transform: translate(-50%, -50%);
      background-size: cover;
      background-position: center;
      border-radius: 50%;
      border: 2px solid white;
      box-shadow: 0 2px 6px rgba(0,0,0,0.6);
      transition: left 200ms linear, top 200ms linear;
    }
    .markerAvatar {
  position: absolute;
  bottom: 55px;
  left: 50%;
  transform: translateX(-50%);
  width: 64px;
  height: 64px;
  background-size: cover;
  background-position: center;
  border-radius: 8px;
  border: 2px solid white;
  display: none;
  background-color: #000;
  z-index: 10;
}
.playerMarker:hover .markerAvatar {
  display: block;
}


    #sheetWrapper { width: 100%; overflow: auto; touch-action: pinch-zoom pan-x pan-y; }
    #characterSheet img { width: 100%; max-width: none; }

    /* Small info row */
    #mapInfo { margin-top:8px; color:#ccc; font-size:14px; display:flex; gap:10px; align-items:center; flex-wrap:wrap; }

    @media (max-width: 600px) {
      header { font-size: 18px; padding: 10px; }
      nav button { font-size: 14px; padding: 10px; }
      #logo { font-size: 28px; margin-top: 20px; }
      #loginBox { width: 90%; padding: 15px; }
      input { padding: 8px; font-size: 14px; }
      #loginBtn { padding: 10px; font-size: 14px; }

      #mapWrapper { height: 420px; }
      #mapArea { width: 2000px; height: 2000px; } /* a bit larger for mobile */
    }
    /* ===== DANGANRONPA RULES ===== */

#rules {
  background: radial-gradient(circle at top, #2a0000, #000);
  min-height: 100vh;
}

.rulesPad {
  max-width: 700px;
  margin: 40px auto;
  background: #0b0b0b;
  border: 3px solid #8b0000;
  border-radius: 14px;
  padding: 20px;
  box-shadow:
    0 0 20px rgba(180,0,0,0.6),
    inset 0 0 30px rgba(120,0,0,0.4);
}

.rulesTitle {
  text-align: center;
  font-size: 32px;
  color: #ff2b2b;
  text-shadow: 2px 2px 0 #000, 0 0 10px red;
  margin-bottom: 10px;
}

.rulesSubtitle {
  text-align: center;
  font-size: 14px;
  color: #ff9a9a;
  font-style: italic;
  margin-bottom: 20px;
}

.rulesText {
  color: #ff3b3b;
  font-size: 17px;
  line-height: 1.6;
  text-shadow: 1px 1px 0 #000, 0 0 6px #900;
}

.rulesText p {
  margin: 10px 0;
}

.rulesWarning {
  margin-top: 15px;
  text-align: center;
  font-size: 13px;
  color: #ff7777;
}
  </style>
</head>
<body>
<header>Монопад (демо-версия)</header>
<nav>
  <button onclick="openPage('home')" class="active" id="tab_home">Главная</button>
  <button onclick="openPage('map')" id="tab_map">Карта</button>
  <button onclick="openPage('info')" id="tab_info">Персонаж</button>
  <button onclick="openPage('rules')" id="tab_rules">Правила</button>
</nav>

<div class="page active" id="home">
  <div id="logo">ЛОГО</div>
  <div id="loginBox">
    <input id="login_input" placeholder="Логин" />
    <input id="pass_input" placeholder="Пароль" type="password" />
    <button id="loginBtn" onclick="login()">Войти</button>
    <div id="login_status" style="margin-top:10px;color:#f55;"></div>
  </div>
</div>

<div class="page" id="map">
  <h2>Карта</h2>

  <!-- Управления этажами (вид) и управления своим этажом -->
  <div id="mapControls">
    <!-- большие иконки / кнопки этажей (view) -->
    <button class="floorBtn" id="view_floor_1">1 этаж</button>
    <button class="floorBtn" id="view_floor_2">2 этаж</button>
    <button class="floorBtn" id="view_floor_b">Подвал</button>

    <!-- разделитель и управление своим этажом -->
    <div style="width:12px"></div>
    <span style="color:#ccc">Мой этаж:</span>
    <button class="myFloorBtn" id="set_floor_1">1</button>
    <button class="myFloorBtn" id="set_floor_2">2</button>
    <button class="myFloorBtn" id="set_floor_b">B</button>

    <div style="flex:1"></div>
    <div id="mapInfo">Просмотр: <strong id="viewFloorLabel">1 этаж</strong> · Мой этаж: <strong id="myFloorLabel">не выбран</strong></div>
  </div>

  <div id="mapWrapper">
    <div id="mapArea"></div>
  </div>
</div>

<div class="page" id="info">
  <h2>Информация о персонаже</h2>
  <div id="infoBox">Вы не вошли в аккаунт.</div>
  <div id="infoContent">
    <div id="leftPanel">
      <div id="avatarBox"></div>
      <div id="descriptionBox"></div>
    </div>
    <div id="sheetWrapper">
      <div id="characterSheet"></div>
    </div>
  </div>
</div>
<div class="page" id="rules">
  <div class="rulesPad">

    <div class="rulesTitle">RULES OF THE GAME</div>

    <div class="rulesSubtitle">
      Monokuma is watching. Break the rules — face despair.
    </div>

    <div class="rulesText">
      <p>1. убийство разрешено лишь с мотивом победить.</p>
      <p>2. После обнаружения тела начинается расследование .</p>
      <p>3. Суд обязателен для всех участников.</p>
      <p>4. Если виновный не будет разоблачен, выживет только он.</p>
      <p>5. Нарушение правил наказывается немедленно. </p>
      <p>6 попытки уничтожить монопод запрещены</p>
      <p>7 монокума может быть задействован в убийстве</p>
    </div>

    <div class="rulesWarning">
      ⚠ новые правила могут быть добавлены в любой момент.
    </div>

  </div>
</div>

<!-- Firebase CDN -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>

const mapImages = {
  1: "https://i.postimg.cc/CMfrDjLK/Znimok-ekrana-2025-12-12-222248.png",     
  2: "https://i.postimg.cc/3wBv6rC3/izobrazenie-2025-12-13-010749253.png",      
  b: "https://i.postimg.cc/pTSQr1nK/Znimok-ekrana-2025-12-12-230210.png"      
};

/* ============ Firebase init  ============ */
const firebaseConfig = {
  apiKey: "AIzaSyBiHWlMDeUv5FmPh4Aqv7aKCGHFbco5YIM",
  authDomain: "dandd-592cb.firebaseapp.com",
  databaseURL: "https://dandd-592cb-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "dandd-592cb",
  storageBucket: "dandd-592cb.appspot.com",
  messagingSenderId: "354514314558",
  appId: "1:354514314558:web:e81c95d0ae2a4c5b9bb86f",
  measurementId: "G-2KGQMKJP4X"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

/* ============ Переменные состояния ============ */
let currentUser = null;      // username
let viewFloor = 1;           // какой этаж показываем (1 / 2 / 'b')
let myFloor = null;          // этаж текущего игрока (1 / 2 / 'b') — синхронизируется в базе
const mapWrapper = document.getElementById('mapWrapper');
const mapArea = document.getElementById('mapArea');

/* Параметры панинга (перетаскивания) */
let offsetX = 0, offsetY = 0;
let isDragging = false;
let dragStartX = 0, dragStartY = 0;
let startOffsetX = 0, startOffsetY = 0;

/* Словарь маркеров DOM по username */
const markers = {};

 #loginBtn { margin-top: 15px; width: 100%; padding: 12px; background: #4CAF50; border: none; color: white; border-radius: 5px; cursor: pointer; font-size: 16px; }
    #mapControls { display:flex; gap:8px; align-items:center; margin-bottom:10px; flex-wrap:wrap; }
    .floorBtn { padding:8px 12px; border-radius:6px; border:1px solid #666; background:#444; color:white; cursor:pointer; font-weight:bold; }
    .floorBtn.active { background:#5a9; color:black; border-color:#3c6; }
    .myFloorBtn { padding:6px 10px; border-radius:6px; border:1px dashed #888; background:#222; color:white; cursor:pointer; }

    #mapWrapper { width: 100%; height: 500px; overflow: hidden; position: relative; border-radius:6px; background:#111; }
    /* mapArea is the large inner surface that we translate for panning */
    #mapArea { width: 1600px; height: 1600px; background: #0f0f0f; border: 2px solid #555; position: absolute; left: 0; top: 0; touch-action: none; will-change: transform; }

    .playerMarker {
      width: 48px;
      height: 48px;
      position: absolute;
      transform: translate(-50%, -50%);
      background-size: cover;
      background-position: center;
      border-radius: 50%;
      border: 2px solid white;
      box-shadow: 0 2px 6px rgba(0,0,0,0.6);
      transition: left 200ms linear, top 200ms linear;
    }
    .markerAvatar {
  position: absolute;
  bottom: 55px;
  left: 50%;
  transform: translateX(-50%);
  width: 64px;
  height: 64px;
  background-size: cover;
  background-position: center;
  border-radius: 8px;
  border: 2px solid white;
  display: none;
  background-color: #000;
  z-index: 10;
}
.playerMarker:hover .markerAvatar {
  display: block;
}


    #sheetWrapper { width: 100%; overflow: auto; touch-action: pinch-zoom pan-x pan-y; }
    #characterSheet img { width: 100%; max-width: none; }

    /* Small info row */
    #mapInfo { margin-top:8px; color:#ccc; font-size:14px; display:flex; gap:10px; align-items:center; flex-wrap:wrap; }

    @media (max-width: 600px) {
      header { font-size: 18px; padding: 10px; }
      nav button { font-size: 14px; padding: 10px; }
      #logo { font-size: 28px; margin-top: 20px; }
      #loginBox { width: 90%; padding: 15px; }
      input { padding: 8px; font-size: 14px; }
      #loginBtn { padding: 10px; font-size: 14px; }

      #mapWrapper { height: 420px; }
      #mapArea { width: 2000px; height: 2000px; } /* a bit larger for mobile */
    }
  </style>
</head>
<body>
<header>Монопад (демо-версия)</header>
<nav>
  <button onclick="openPage('home')" class="active" id="tab_home">Главная</button>
  <button onclick="openPage('map')" id="tab_map">Карта</button>
  <button onclick="openPage('info')" id="tab_info">Персонаж</button>
</nav>

<div class="page active" id="home">
  <div id="logo">ЛОГО</div>
  <div id="loginBox">
    <input id="login_input" placeholder="Логин" />
    <input id="pass_input" placeholder="Пароль" type="password" />
    <button id="loginBtn" onclick="login()">Войти</button>
    <div id="login_status" style="margin-top:10px;color:#f55;"></div>
  </div>
</div>

<div class="page" id="map">
  <h2>Карта</h2>

  <!-- Управления этажами (вид) и управления своим этажом -->
  <div id="mapControls">
    <!-- большие иконки / кнопки этажей (view) -->
    <button class="floorBtn" id="view_floor_1">1 этаж</button>
    <button class="floorBtn" id="view_floor_2">2 этаж</button>
    <button class="floorBtn" id="view_floor_b">Подвал</button>

    <!-- разделитель и управление своим этажом -->
    <div style="width:12px"></div>
    <span style="color:#ccc">Мой этаж:</span>
    <button class="myFloorBtn" id="set_floor_1">1</button>
    <button class="myFloorBtn" id="set_floor_2">2</button>
    <button class="myFloorBtn" id="set_floor_b">B</button>

    <div style="flex:1"></div>
    <div id="mapInfo">Просмотр: <strong id="viewFloorLabel">1 этаж</strong> · Мой этаж: <strong id="myFloorLabel">не выбран</strong></div>
  </div>

  <div id="mapWrapper">
    <div id="mapArea"></div>
  </div>
</div>

<div class="page" id="info">
  <h2>Информация о персонаже</h2>
  <div id="infoBox">Вы не вошли в аккаунт.</div>
  <div id="infoContent">
    <div id="leftPanel">
      <div id="avatarBox"></div>
      <div id="descriptionBox"></div>
    </div>
    <div id="sheetWrapper">
      <div id="characterSheet"></div>
    </div>
  </div>
</div>

<!-- Firebase CDN -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
/* =======================
   CONFIG — вставь свои URL карт для этажей
   Пример:
   mapImages[1] = 'https://site.com/floor1.png';
   mapImages[2] = 'https://site.com/floor2.png';
   mapImages['b'] = 'https://site.com/basement.png';
   Оставь пустые строки если хочешь вставить позже.
   ======================= */
const mapImages = {
  1: "https://i.postimg.cc/CMfrDjLK/Znimok-ekrana-2025-12-12-222248.png",      // <- вставь ссылку на изображение первого этажа
  2: "https://i.postimg.cc/3wBv6rC3/izobrazenie-2025-12-13-010749253.png",      // <- вставь ссылку на изображение второго этажа
  b: "https://i.postimg.cc/pTSQr1nK/Znimok-ekrana-2025-12-12-230210.png"       // <- вставь ссылку на изображение подвала (ключ 'b' = basement)
};

/* ============ Firebase init (твои данные уже стояли) ============ */
const firebaseConfig = {
  apiKey: "AIzaSyBiHWlMDeUv5FmPh4Aqv7aKCGHFbco5YIM",
  authDomain: "dandd-592cb.firebaseapp.com",
  databaseURL: "https://dandd-592cb-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "dandd-592cb",
  storageBucket: "dandd-592cb.appspot.com",
  messagingSenderId: "354514314558",
  appId: "1:354514314558:web:e81c95d0ae2a4c5b9bb86f",
  measurementId: "G-2KGQMKJP4X"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();
  /* ================= ПУЛИ ПРАВДЫ ================= */

let editingBulletId = null;

function openAddBullet() {
  if (!currentUser) {
    alert("Войдите в аккаунт");
    return;
  }
  editingBulletId = null;
  document.getElementById('bulletModalTitle').innerText = "Новая пуля";
  document.getElementById('bulletName').value = "";
  document.getElementById('bulletDesc').value = "";
  document.getElementById('bulletEmoji').value = "";
  document.getElementById('bulletModal').style.display = "flex";
}

function closeBulletModal() {
  document.getElementById('bulletModal').style.display = "none";
}

function saveBullet() {
  const name = bulletName.value.trim();
  const desc = bulletDesc.value.trim();
  const emoji = bulletEmoji.value.trim() || "🔍";

  if (!name) {
    alert("Введите название");
    return;
  }

  const ref = db.ref("bullets/" + currentUser).push();
  ref.set({
    name,
    desc,
    emoji
  });

  closeBulletModal();
}

function loadBullets() {
  if (!currentUser) return;
  const list = document.getElementById('bulletsList');
  list.innerHTML = "";

  db.ref("bullets/" + currentUser).on("value", snap => {
    list.innerHTML = "";
    const data = snap.val();
    if (!data) return;

    Object.entries(data).forEach(([id, bullet]) => {
      const div = document.createElement("div");
      div.className = "bulletItem";

      div.innerHTML = `
        <div class="bulletEmoji">${bullet.emoji}</div>
        <div>${bullet.name}</div>
        <div class="bulletDelete">➖</div>
      `;

      div.onclick = () => {
        bulletName.value = bullet.name;
        bulletDesc.value = bullet.desc;
        bulletEmoji.value = bullet.emoji;
        document.getElementById('bulletModalTitle').innerText = "Пуля правды";
        document.getElementById('bulletModal').style.display = "flex";
      };

      div.querySelector(".bulletDelete").onclick = (e) => {
        e.stopPropagation();
        db.ref("bullets/" + currentUser + "/" + id).remove();
      };

      list.appendChild(div);
    });
  });
}

/* подгружаем пули при входе */
const originalLogin = login;
login = function () {
  originalLogin();
  setTimeout(loadBullets, 300);
};


/* ============ Переменные состояния ============ */
let currentUser = null;      // username
let viewFloor = 1;           // какой этаж показываем (1 / 2 / 'b')
let myFloor = null;          // этаж текущего игрока (1 / 2 / 'b') — синхронизируется в базе
const mapWrapper = document.getElementById('mapWrapper');
const mapArea = document.getElementById('mapArea');

/* Параметры панинга (перетаскивания) */
let offsetX = 0, offsetY = 0;
let isDragging = false;
let dragStartX = 0, dragStartY = 0;
let startOffsetX = 0, startOffsetY = 0;

/* Словарь маркеров DOM по username */
const markers = {};

/* Аккаунты — оставил твои данные, можно редактировать */
const accounts = [
    { username: "цумуги", displayName: "Цумуги Широгане", password: "0387", color: "blue", avatar: "https://i.postimg.cc/MKDL3Brk/IMG-20251212-023652-731.jpg",markerAvatar: "https://i.postimg.cc/NjJg3gs7/IMG-20260214-145036-602.png", description: "", sheet: "" },
    { username: "нагито", displayName: "Нагито Комаэда", password: "1492", color: "white", avatar: "https://i.postimg.cc/1369x3DX/IMG-20260212-224508-534.png",markerAvatar: "https://i.postimg.cc/fbmrxKry/IMG-20260214-145036-728.png", description: "Святой мученик.                                      Сын полуэльфа-учёного и благородной жрицы, Нагито с детства рос под сводами Великого Храма Стеллы. Ему предсказывали блестящее будущее: ясный ум, врождённое обаяние и редкая способность вызывать доверие у людей делали его любимцем наставников.Но счастье было недолгим. Едва достигнув зрелости, Нагито пережил серию страшных личных трагедий — сначала умерла мать, затем отец погиб в странных обстоятельствах, а вскоре загадочная болезнь унесла почти всех его друзей. Каждый раз он оставался единственным, кто чудом выживал. Эти совпадения укрепили в нём болезненную веру: Стелла дарует ему «священную удачу» — но лишь ценой чужих жизней.Его необычная харизма и готовность принять любой удар судьбы сделали его самым молодым Верховным Жрецом в истории церкви. Под его руководством Великий Храм превратился в место паломничества: одни приходят за исцелением, другие — чтобы испытать «путь очищения через страдание», который он проповедует. Для народа он — живое чудо. Для некоторых жрецов — опасный фанатик, готовый бросить любого в объятия судьбы, если это приблизит их к свету Стеллы.", sheet: "https://i.postimg.cc/4yq62CYD/IMG-20260214-145949-097.jpg" },
    { username: "миу", displayName: "Миу Ирума", password: "2750", avatar: "https://i.postimg.cc/pXPP3DMm/IMG-20260207-215033-607.jpg",markerAvatar: "https://i.postimg.cc/PJQTPqYt/IMG-20260207-215010-115.png", description: `Демоничный колдун.                                                                                                                                                      
     миу родилась человеком в бедной семье ремесленников.
      С юности отличалась дерзким характером и желанием доказать своё превосходство над другими.
       Когда в ней проявились магические задатки, она поступила в столичную академию и быстро освоила азы колдовства.
        Но, даже закончив обучение, Миу чувствовала, что её сил недостаточно.Гонимая жаждой могущества, 
        она обратилась к древним ритуалам тифлингов и заключила договор с низшим демоном Ignus, рождённым из сущности Терры.
         Сделка изменила её навсегда: её тело стало носителем демонической крови, а в глазах загорелся огонь пламени. 
         Так человек превратился в тифлинга, заклеймённого собственной жадностью до силы.Магия Миу сочетает в себе 
         дерзость и практичность. Она свободно пользуется простыми заговорами — «Волшебная рука», «Чудотворство», 
         «Ядовитые брызги», — а также владеет более сильными чарами: «Очарование личности», чтобы подчинять волю, и 
         «Защита от зла и добра», чтобы отражать чужое тёмное влияние.Миу презирает тех, кто называет её проклятой или продажной
          для неё это лишь доказательство её смелости. Она гордится тем, что решилась на то, чего другие боятся — отказаться от 
          человечности ради настоящей силы.`, sheet: "https://i.postimg.cc/fL58ynkQ/IMG-20260208-001546-912.jpg" },
    { username: "каэде", displayName: "Каэде Акаматсу", password: "4069", color: "yellow", avatar: "https://i.postimg.cc/N0FvsGHT/IMG-20260213-002748-276.png",markerAvatar: "https://i.postimg.cc/13S8Z1sp/IMG-20260108-122721-302.png", description: "Звёздный избранник.                                 Когда родилась в маленькой эльфийской деревне на окраине государства, где густые леса соприкасаются с землями людей. С ранних лет она чувствовала себя «на границе миров»: слушала древние песни о Сириусе, боге всего живого, и одновременно слышала истории о людях и их вере в Стеллу. Её родные почитали природу и жили в гармонии, но сама Каэде всегда мечтала о большем, чем жизнь в тихой глуши.Когда в ней пробудился магический талант, её отправили в эльфийскую академию. Там она поразила наставников необычным подходом к магии: заклинания в её руках звучали как музыка, будто отражение внутренней гармонии. Она верила, что сама сила Сириуса проявляется через мелодию, соединяющую всё живое.Но Каэде не могла довольствоваться только изучением чар. Она всё больше задумывалась о том, что два государства, столь разные в вере, не должны оставаться разделёнными. Для неё магия и музыка были не оружием, а ключом к объединению. Многие считали её идеалисткой, но в народе начали шептаться о пророчестве: «звезда, рождённая на границе, зазвучит и для людей, и для эльфов».     Теперь Каэде путешествует по миру скитаясь, неся с собой свет знаний и гармонии. Для эльфов она — дитя Сириуса, для людей — загадочный посредник, а для самой себя — всего лишь волшебница, которая пытается сыграть мелодию, способную изменить судьбу обоих народов.", sheet: "https://i.postimg.cc/4NQDpHvw/IMG-20260213-003330-914.jpg" },
    { username: "рантаро", displayName: "Рантаро Амами", password: "5821",  avatar: "https://i.postimg.cc/XqxgLf7X/IMG-20251212-024217-121.png",markerAvatar: "https://i.postimg.cc/T38jr6Mn/IMG-20260214-145036-158.png", description: `Падший паладин.

В белоснежном королевстве, где магия — не дар, а закон, имя Стеллы Бетельгейм произносят, склоняя голову.
Она — богиня магии, сияние знания, что подчинило себе разум и волю человечества. Её культ — это не вера, а строй: каждый ребёнок с детства учится различать «чистую» магию от «еретической», каждый взрослый живёт под взором храмов, что возвышаются над улицами, будто сторожевые башни.

Рантаро не родился — его вызвали из пустоты.
Чужеземец, появившийся на снегу монастырского двора, без воспоминаний, без имени. Маги нарекли его сосудом Стеллы, созданным для того, чтобы хранить её магию в человеческом теле. Так он стал паладином света, хранителем порядка и проводником её воли.

Ему вручили длинное копьё — символ её пронизывающей мощи. Это было не просто оружие, а жезл света, способный проводить заклинания через сталь. Рантаро чувствовал, как магия богини течёт по древку, будто само дерево и металл поют её имя.

Годы он исполнял свой долг: карал тех, кто использовал магию без благословения церкви, жёг книги, закрывал уста. И всё же — каждое заклинание, каждый удар копья оставляли в его сердце трещины.

Однажды, среди руин деревни, сожжённой за «осквернение стихии», он нашёл ребёнка, что чертил знаки на снегу.
Знаки были неловкие, но живые. Магия ребёнка не разрушала — она созидала.
Тогда сосуд впервые понял: в магии нет света или тьмы — есть только воля того, кто её творит.

С тех пор Рантаро стал отступником.
Он ушёл из храмов, снял свой шлем и покинул белые города. Свет Стеллы больше не греет его — но он и не отринул её. Он верит, что богиня магии не зла и не добра, просто она равнодушна, как снег, падающий и на храм, и на руины.

Теперь Рантаро странствует по северным дорогам, скрываясь под белым плащом. Его копьё — это не инструмент веры, а напоминание о ней.
И когда ветер поёт над ледяными равнинами, кажется, что он слышит шёпот самой Стеллы —
или, может быть, самого себя, наконец ставшего живым.`, sheet: "https://i.postimg.cc/J44xn2Pw/IMG-20251212-024232-021.jpg" },
    { username: "чихиро", displayName: "Чихиро Фуджисаки", password: "6825", color: "brown", avatar: "https://i.postimg.cc/KYSvH4HK/IMG-20260126-174934-610.png",markerAvatar: "https://i.postimg.cc/BvrDNqxR/IMG-20260126-174934-578.png", description: "Тихий хранитель.                                       Чихиро родился в зелёных лесах эльфийского королевства, где каждое дерево старше любого из мудрецов, а магия течёт сквозь землю так же естественно, как кровь сквозь вены. Эльфы жили в согласии с природой, под покровительством Сириуса — богини всего живого, что учит видеть жизнь во всём, даже в мху на камне.Он вырос в эпоху перемен: Парламент Мудрецов готовился к коронации новой королевы Сонии — монархини, призванной не править, а напоминать эльфам о единстве. В те годы Чихиро служил при местном святилище как прислужник-друид, ухаживая за рощами, животными и источниками, где вода считалась «дыханием Сириуса».Он был робким, тихим и застенчивым юношей. Часто избегал разговоров, предпочитая слушать треск ветра и шелест листьев. Его мир — это не города и речи Мудрецов, а запах земли после дождя, трепетное движение травы под солнцем.Сослужители смеялись: «Чихиро ближе к оленю, чем к эльфу».Но никто не мог отрицать, что природа отвечала ему взаимностью — растения оживали под его руками, звери не боялись подходить близко.Когда начались волнения на северных границах — там, где людские караваны нарушали покой священных земель, — Чихиро не взял оружие. Он ушёл туда один, чтобы умолить природу не ответить яростью на ярость.С тех пор он странствует между рощами, помогая тому, что живо, и избегая того, что жадно.Он не ищет славы и не ждёт награды. Он просто живёт по завету своей богини:«Слушай, не властвуй. Береги, не приказывай. Живи — и давай жить.»", sheet: "https://i.postimg.cc/q7x0Dwnf/IMG-20260127-181059-011.jpg" },
    { username: "кай", displayName: "Кай Монтего", password: "6904", color: "purple", avatar: "https://i.postimg.cc/GhK4rS2J/IMG-20260108-122721-262.png",markerAvatar: "https://i.postimg.cc/Kjc0WHj9/IMG-20260108-122721-404.png", description: `Амбициозный мошенник.

Кай Монтего родился в семье известных актёров в эльфийском королевстве, где власть избирают мудрецы, леса бесконечны, а вера в Сириус является нормой, а не обязанностью. Его дом с детства был наполнен репетициями, голосами, инструментами и шумом сцены — он вырос в мире, где каждый говорит громче, чем думает, где важно не что ты чувствуешь, а как это выглядит со стороны.

Он рано начал выступать — сначала в детских постановках, позже в больших театрах столицы, где его запоминали за энергичность и смелость. Кай обожал внимание публики, зависел от него, как растение от солнца. Когда зрители аплодировали — он чувствовал себя великим. Когда молчали — казалось, что внутри него пусто. Именно тогда он впервые попробовал другое искусство — искусство обмана.

Во время гастролей он заметил, что люди охотнее раскрывают кошельки, если верят не артисту, а другу. Он стал играть роли прямо на улицах — потерянного отпрыска, доверчивого новичка, случайного прохожего, которому «нужна помощь». Так он открыл в себе плута: ловкого в словах, остроумного, умеющего подстраиваться и менять личности как костюмы.

Слава росла, но вместе с ней рос и внутренний страх быть разоблачённым. Кай всё чаще критиковал себя, считал, что без масок он пуст, а талант — всего лишь удачная иллюзия. Родители любили его, но он всегда думал, что должен стать лучше, ярче, громче, чтобы их гордость была заслуженной, а не автоматической.

Он участвовал в спектаклях столицы, путешествовал между городами, выступал в парламенте мудрецов на культурных фестивалях. Иногда за кулисами пропадали драгоценности, иногда зрители уходили с облегчёнными кошельками — но никто не мог связать это с улыбчивым юным артистом. Он был хорош, слишком хорош, чтобы сомненье задержалось надолго.

Однажды, после успешного выступления на летнем празднике, Кая пригласили на частную встречу. Незнакомец наблюдал за ним долго, изучал поведение, эмоции, слабости. Тогда Кай впервые услышал предложение участвовать в игре — опасной, закрытой, смертельной. Но в этом была слава, внимание, шанс войти в историю. Не как актёр — как легенда.

Он согласился почти без размышлений, ощущая азарт, предвкушая громкие овации, которые услышат даже за пределами королевства.

Он не знал, что организатор видел в нём удобную марионетку — слишком яркую, чтобы не заметили, и слишком зависимую от признания, чтобы отказаться. Теперь судьба Кая — больше не только театр. Это сцена, где цена спектакля — жизнь.`, sheet: "https://i.postimg.cc/pLyrSWTk/sheet7.jpg" },
    { username: "кокичи", displayName: "Кокичи Ома", password: "7318", color: "violet", avatar: "https://i.postimg.cc/yxSZg0gm/IMG-20251212-030500-800.png",markerAvatar: "https://i.postimg.cc/4NWpQ6Jf/IMG-20260214-000501-523.png", description: `Голос Обмана+

Настоящее имя неизвестно.
На сцене его знали под сотней масок, но имя Кокичи Ома он выбрал сам — в честь мертвого героя, которого никто, кроме него, не помнит.

Он вырос в балагане на краю империи: фальшивые чудеса, поддельные пророчества, ядовитый грим и настоящая нищета. Родителей заменили актёры, друзей — роли. Искренность высмеивалась. Ложь — поощрялась. Там он научился: правда не ведёт за собой толпу, но хорошая история — всегда.

С малых лет он играл бога, дракона, шутника, мертвеца, вождя.
Но однажды он сорвал с себя все образы — и остался пуст. Бросив труппу, он ушёл в мир, чтобы играть теперь уже свою собственную игру.

Он лжёт, чтобы смешить. Он провоцирует, чтобы люди увидели себя. Он добр, но никогда прямо. Он обманщик — но не предатель.
Для него ложь — кисть, а реальность — холст. И если придётся перевернуть мир, чтобы сделать его красивее — он не задумываясь это сделает.`, sheet: "https://i.postimg.cc/63pV37fD/IMG-20251212-024143-183.jpg" },
    { username: "изуру", displayName: "Изуру Камукура", password: "8046", color: "gray", avatar: "https://i.postimg.cc/RVZ6zK07/IMG-20260213-000428-127.png",markerAvatar: "https://i.postimg.cc/FFDng1DB/IMG-20260214-145035-916.png", description: `Уединение. Осознание.

Изуру Камукура родился среди эльфов, но его происхождение тщательно скрыто. Ни одного подтверждённого факта о его семье, детстве или наставниках не существует.

В возрасте, не указанном ни в одном эльфийском реестре, он ушёл в горы — один. Причины ухода не названы. Добровольное изгнание, без следов и записей.

В течение 17 лет находился в полной изоляции. Единственное, что известно о его деятельности в этот период — фрагментарные находки: философские заметки без автора, рисунки из символов, совпадающих с древними молитвами, выжженные на камне круги.

Первое появление в обществе — случайное. Он спас умирающего ребёнка, не назвав имени, и исчез до рассвета. Потом снова — в разрушенном храме, где тела лежали нетронутыми, а стены были очищены.

Он не упоминает богов, но его заклинания работают. Он говорит редко, чаще — наблюдает. Не вступает в конфликты, если в этом нет необходимости. Его вера — не в существо, а в порядок или хаос как природное явление.

Именем Изуру Камукура пользуется сам. Считается, что оно не настоящее. Личность до изоляции не установлена. Возраст не определён.

Его цель неясна. Его путь не объясняется. Он просто идёт вперёд.`, sheet: "https://i.postimg.cc/4x4xjzfV/IMG-20251212-025454-723.jpg" },
    { username: "монака", displayName: "Монака Това", password: "9163", color: "lightgreen", avatar: "https://i.postimg.cc/J4vp2Q57/IMG-20251212-025905-402.png",markerAvatar: "https://i.postimg.cc/Pqvk0YFs/IMG-20260214-145036-434.png", description: `Святое притворство:

Когда-то в глухой долине, за чёрными болотами и зловещим холмом стоял приют под названием Дом Милосердия Элании. Местные шептались, что дети туда попадают не по воле богов, и что сама земля вокруг приюта плачет ночью. Именно там, в дождливую ночь, была найдена крошечная девочка — наполовину эльфийка, с чёртами лица столь хрупкими, что она казалась игрушечной.
Её назвали Монакой.
С юных лет она проявляла необычную проницательность. Она могла рассказать, кто из воспитателей лжёт, а кто — ворует еду. Но, вместо того чтобы быть благодарной, Монака начала играть с их слабостями. Вскоре в приюте вспыхнули конфликты, начались доносы, исчезновения… и пожары. Никто не мог доказать, что это она, но все знали — всё началось с Монаки.
В 10 лет она уже командовала младшими детьми как предводительница маленькой секты, говоря, что взрослые не настоящие, а миром должны править дети, свободные от морали.
Однажды ночью она устроила массовый побег из приюта, уведя за собой целую группу посвящённых.

Монака участвовала в ограблениях, поджогах и подстрекательстве к убийствам, прикрываясь маской невинного дитя. Она использовала священные символы, чтобы убедить крестьян, что их обманывают местные жрецы, а затем развращала паству, создавая новую веру — культ детской власти.
Когда орден Пелора попытался остановить её, она сдалась без боя, улыбаясь, и прошептала:
«Пока вы стареете и боитесь, я останусь маленькой. И дети меня запомнят».

Один из жрецов, по странной жалости, не казнил её, а предложил исправление — направить её через путь служения. Монака согласилась… но только для того, чтобы стать жрецом, изучить силу веры и обратить её в оружие.`, sheet: "https://i.postimg.cc/6qZGXqjJ/Picsart-25-07-05-11-30-55-133-1.jpg" },
    { username: "сония", displayName: "Сония Невермaйнд", password: "2479", color: "lightyellow", avatar: "https://i.postimg.cc/QM2Lbkys/IMG-20260122-002814-353.png",markerAvatar: "https://i.postimg.cc/fy3wLTF5/IMG-20260122-002811-777.png", description: "Запретное знание   Сония Невермайнд с рождения была предназначена стать символом эльфийского государства. Парламент Мудрецов существовал почти с момента сотворения королевства, и корона всегда была знаком преемственности, а не инструментом власти. Сонию готовили именно к этой роли: быть лицом государства, хранительницей традиций, точкой стабильности в мире, где даже леса меняются медленно, но неизбежно.Она была образованной, дисциплинированной и искренне доброй. Сония верила в законы, уважала решения парламента и никогда не ставила под сомнение саму систему. Однако её личный интерес лежал глубже официальных учений. С юности она увлекалась оккультизмом — не как запретным искусством, а как формой познания мира. Древние ритуалы, забытые символы, крайние философские течения, баланс между жизнью и смертью — всё это интересовало её как способ понять, где проходит граница дозволенного и необходимого.Её тянуло к экстремальным идеям не из жажды разрушения, а из желания проверить пределы: может ли порядок существовать без крайностей, и что происходит, когда ради сохранения жизни приходится идти на шаг за грань привычной морали. При этом Сония никогда не нарушала законов — она изучала, а не практиковала, наблюдала, а не действовала. Для парламента она оставалась идеальной кандидатурой: умной, сдержанной, надёжной.Именно это сочетание — абсолютная лояльность и интерес к запретному — привело её к открытию, которое не должно было касаться будущей королевы. Задолго до коронации Сония узнала о существовании механизма, скрытого за фасадом стабильности: системы, в которой крайние меры применялись тайно, во имя сохранения баланса мира. Убийственная игра была частью этой системы — не развлечением, а инструментом.Сония узнала об этом до того, как оказалась в игре.И поняла, что если она станет королевой, это знание сделает её либо лгуньей, либо соучастницей.Она приняла решение ещё тогда — молча, без свидетелей. Коронация должна была состояться, но не состоялась. Вместо трона она оказалась среди участников игры, единственная, кто понимал, что происходит и почему. Она не сопротивлялась и не пыталась сбежать, потому что знала: её присутствие — часть уже запущенного процесса.Сония остаётся доброй и сострадательной. Она помогает другим, поддерживает слабых и не поощряет насилие. Но внутри она несёт тяжёлое знание: иногда мир держится не на законах и не на символах, а на том, что кто-то соглашается заплатить цену молча.Корона не была утрачена.Она была отложена — вместе с правдой, которую нельзя было произнести вслух. ", sheet: "https://i.postimg.cc/s2f6WVCj/Picsart-26-01-22-14-22-32-655.jpg" },
    { username: "кируми", displayName: "Кируми Тоджо", password: "3508", color: "darkviolet", avatar: "[https://png.klev.club/10324-barli.html](https://static.wikia.nocookie.net/brawlstars/images/5/5a/%D0%91%D0%B0%D1%80%D0%BB%D0%B8_Skin-Default.png/revision/latest?cb=20250524203143&path-prefix=ru)",markerAvatar: "https://i.pinimg.com/736x/b1/18/8c/b1188ce053737a274590f131ac3b3d21.jpg", description: "", sheet: "" },
    { username: "леон", displayName: "Леон Кувата", password: "4682", color: "orange", avatar: "https://i.postimg.cc/k5ZCxDYG/IMG-20260116-232420-319.png",markerAvatar: "https://i.postimg.cc/SK4Nnn5N/IMG-20260116-232420-592.png", description: "Благородный маг.      Леон Кувата родился в семье древнего человеческого рода, который веками служил магии и государству. С детства его обучали этикету, искусству и магическим практикам, ожидая, что он станет достойным наследником семейного имени. Несмотря на строгие рамки, Леон всегда сохранял жизнерадостный, острый ум и чувство юмора, что делало его популярным среди сверстников и наставников. Поступив в столичный университет магии, он блеснул талантом и умением сочетать теорию с практикой, быстро заслужив уважение профессоров. Благородство, воспитание и врождённая харизма позволили ему не только овладеть магией, но и обрести авторитет среди коллег и младших студентов. Став выпускником, Леон покинул университет, намереваясь использовать свои навыки и знания во благо людей и укрепление магического искусства, но при этом оставаясь верным духу приключений и исследовательской страсти, заложенной ещё в детстве.", sheet: "https://i.postimg.cc/kGmkcSQR/IMG-20260116-233738-319.jpg" },
    { username: "макото", displayName: "Макото Наэги", password: "5790", color: "lightbrown", avatar: "https://i.postimg.cc/CKgbZrJw/IMG-20260210-120752-235.png",markerAvatar: "https://i.postimg.cc/xCgdVPpG/IMG-20260214-000502-462.png", description: "Страж троп.                                          Макото Наэги родился в одном из старых лесных кантонов эльфийского государства — местах, где парламент мудрецов ощущается лишь через законы, а не через лица. Его семья веками служила проводниками и хранителями троп: они не управляли лесом, а жили с ним, передавая знания не в книгах, а в привычках и молчании. Для них важнее всего были стабильность и сохранение порядка вещей.С ранних лет Макото помогал в приютах для верующих и странников, что объясняет его путь прислужника. Он носил воду, чинил кровлю, сопровождал стариков и детей по лесным дорогам. Именно там он научился внимательности к мелочам и людям — умению замечать усталость, страх и ложь раньше, чем слова. Он никогда не стремился быть лидером, но часто становился тем, кому доверяли.Став следопытом, Макото начал служить связующим звеном между общинами: передавал вести, сопровождал караваны, разыскивал пропавших в чащобах. Он плохо переносит насилие, но понимает его необходимость, предпочитая избегать прямых столкновений. Его стиль — осторожность, подготовка и путь назад. Он не охотник за славой, а гарант того, что кто-то вернётся домой.Макото глубоко привязан к семье и традициям, но именно это делает его уязвимым: он боится перемен и часто сомневается в собственных решениях. Его самокритичность граничит с безнадёжностью, особенно когда мир вокруг начинает рушиться быстрее, чем он способен его удержать.Он верит, что даже самый обычный человек может стать опорой для других — не потому, что должен, а потому что иначе некому.…а иногда судьба проверяет эту веру слишком жестоко.", sheet: "https://i.postimg.cc/zvsZsbNh/IMG-20260214-000734-095.jpg" },
    { username: "монокума", displayName: "МОН0КУМА", password: "8742", color: "white", avatar: "https://i.postimg.cc/5tq95ymv/Monokuma-Illustration.webp",markerAvatar: "https://i.postimg.cc/2jXk7PnF/Ball-Monokuma-transparent-jpg.webp", description: "", sheet: "" },
  ];

/* ================= UI helpers ================= */
function setViewFloorLabel() {
  const lbl = document.getElementById('viewFloorLabel');
  lbl.innerText = viewFloor === 'b' ? 'Подвал' : (viewFloor === 1 ? '1 этаж' : '2 этаж');
  document.querySelectorAll('.floorBtn').forEach(b => b.classList.remove('active'));
  if (viewFloor === 1) document.getElementById('view_floor_1').classList.add('active');
  else if (viewFloor === 2) document.getElementById('view_floor_2').classList.add('active');
  else document.getElementById('view_floor_b').classList.add('active');
}
function setMyFloorLabel() {
  const lbl = document.getElementById('myFloorLabel');
  lbl.innerText = myFloor === null ? 'не выбран' : (myFloor === 'b' ? 'Подвал' : (myFloor === 1 ? '1' : '2'));
}

/* ================= Floor buttons ================= */
document.getElementById('view_floor_1').addEventListener('click', () => { viewFloor = 1; applyMapBackground(); setViewFloorLabel(); refreshMarkersVisibility(); });
document.getElementById('view_floor_2').addEventListener('click', () => { viewFloor = 2; applyMapBackground(); setViewFloorLabel(); refreshMarkersVisibility(); });
document.getElementById('view_floor_b').addEventListener('click', () => { viewFloor = 'b'; applyMapBackground(); setViewFloorLabel(); refreshMarkersVisibility(); });

document.getElementById('set_floor_1').addEventListener('click', () => { if (!currentUser) return alert('Войдите в аккаунт'); setMyFloorForUser(1); });
document.getElementById('set_floor_2').addEventListener('click', () => { if (!currentUser) return alert('Войдите в аккаунт'); setMyFloorForUser(2); });
document.getElementById('set_floor_b').addEventListener('click', () => { if (!currentUser) return alert('Войдите в аккаунт'); setMyFloorForUser('b'); });

/* ================= Map background (image overlay) ================= */
function applyMapBackground() {
  const img = mapImages[viewFloor] || '';
  if (img) {
    mapArea.style.backgroundImage = `url('${img}')`;
    mapArea.style.backgroundSize = 'contain';
    mapArea.style.backgroundRepeat = 'no-repeat';
    mapArea.style.backgroundPosition = 'center';
  } else {
    mapArea.style.backgroundImage = '';
    mapArea.style.backgroundColor = '#0f0f0f';
  }
}

/* ================= Pan / Drag (desktop and touch) ================= */
function updateMapTransform() {
  mapArea.style.transform = `translate(${offsetX}px, ${offsetY}px)`;
}

/* Mouse */
mapWrapper.addEventListener('mousedown', (e) => {
  // only start dragging if clicked on empty space or background (not on marker)
  if (e.target.classList.contains('playerMarker')) return;
  isDragging = true;
  dragStartX = e.clientX;
  dragStartY = e.clientY;
  startOffsetX = offsetX;
  startOffsetY = offsetY;
  mapWrapper.style.cursor = 'grabbing';
});
window.addEventListener('mousemove', (e) => {
  if (!isDragging) return;
  const dx = e.clientX - dragStartX;
  const dy = e.clientY - dragStartY;
  offsetX = startOffsetX + dx;
  offsetY = startOffsetY + dy;
  updateMapTransform();
});
window.addEventListener('mouseup', () => {
  if (isDragging) {
    isDragging = false;
    mapWrapper.style.cursor = 'default';
  }
});

/* Touch */
mapWrapper.addEventListener('touchstart', (ev) => {
  if (ev.touches.length === 1) {
    const t = ev.touches[0];
    isDragging = true;
    dragStartX = t.clientX;
    dragStartY = t.clientY;
    startOffsetX = offsetX;
    startOffsetY = offsetY;
  }
});
mapWrapper.addEventListener('touchmove', (ev) => {
  if (!isDragging || ev.touches.length !== 1) return;
  const t = ev.touches[0];
  const dx = t.clientX - dragStartX;
  const dy = t.clientY - dragStartY;
  offsetX = startOffsetX + dx;
  offsetY = startOffsetY + dy;
  updateMapTransform();
});
mapWrapper.addEventListener('touchend', () => { isDragging = false; });

/* ================= Marker handling (realtime) ================= */

/*
 Structure in DB:
 positions: {
   username1: { x: 123, y: 234, floor: 1 },
   username2: { x: 400, y: 500, floor: 'b' },
   ...
 }
*/

function ensureMarkerElement(username, acc) {
  if (markers[username]) return markers[username];
  const el = document.createElement('div');
  el.className = 'playerMarker';
  el.dataset.user = username;
  el.title = acc ? acc.displayName : username;
  el.style.backgroundImage = acc && acc.markerAvatar ? `url('${acc.markerAvatar}')` : `radial-gradient(circle at 30% 30%, #fff 0, ${acc && acc.color ? acc.color : '#888'} 60%)`;
  if (acc && acc.avatar) {
  const avatarPopup = document.createElement('div');
  avatarPopup.className = 'markerAvatar';
  avatarPopup.style.backgroundImage = `url('${acc.avatar}')`;


  el.appendChild(avatarPopup);
}

  // click on marker -> show profile in info page
  
  mapArea.appendChild(el);
  markers[username] = el;
  return el;
}

/* Listen to all positions changes in realtime */
const positionsRef = db.ref('positions');
positionsRef.on('value', (snap) => {
  const data = snap.val() || {};
  // update markers or create them
  Object.keys(data).forEach(username => {
    const pos = data[username];
    const acc = accounts.find(a => a.username === username);
    const el = ensureMarkerElement(username, acc);
    // place marker only if floor matches viewFloor
    if (pos && pos.x != null && pos.y != null) {
      el.style.left = pos.x + 'px';
      el.style.top = pos.y + 'px';
      el.dataset.floor = pos.floor ?? 1;
    } else {
      el.style.left = '150px';
      el.style.top = '150px';
      el.dataset.floor = pos && pos.floor ? pos.floor : 1;
    }
    // visibility handled separately
  });
  // also ensure markers exist for accounts that may not have DB entry yet
  accounts.forEach(a => {
    if (!markers[a.username]) ensureMarkerElement(a.username, a);
  });
  refreshMarkersVisibility();
});

/* Refresh visibility according to viewFloor (show only markers whose stored floor == viewFloor) */
function refreshMarkersVisibility() {
  Object.keys(markers).forEach(username => {
    const el = markers[username];
    const userFloor = el.dataset.floor || 1;
    // convert to comparable values: 'b' for basement
    const matches = (String(userFloor) === String(viewFloor));
    el.style.display = matches ? 'block' : 'none';
  });
}

/* Save position for current user (x,y) and maintain floor (if not set yet, use myFloor or 1) */
function savePositionForCurrentUser(x, y, floor = null) {
  if (!currentUser) return;
  // determine floor: use provided floor, else existing myFloor, else 1
  const realFloor = (floor !== null && floor !== undefined) ? floor : (myFloor !== null ? myFloor : 1);
  db.ref('positions/' + currentUser).set({ x: Math.round(x), y: Math.round(y), floor: realFloor });
}

/* Set myFloor for current user (update DB positions/currentUser.floor preserving x,y if exist) */
function setMyFloorForUser(newFloor) {
  if (!currentUser) return;
  const pRef = db.ref('positions/' + currentUser);
  pRef.once('value').then(snap => {
    const cur = snap.val() || {};
    const x = cur.x != null ? cur.x : 150;
    const y = cur.y != null ? cur.y : 150;
    pRef.set({ x, y, floor: newFloor });
    myFloor = newFloor;
    setMyFloorLabel();
    // If the player moved floors, optionally hide their marker on other floors (handled by realtime listener)
    refreshMarkersVisibility();
  });
}

/* ================= Click on map -> place marker for currentUser with coords adjusted for pan ================= */
mapWrapper.addEventListener('click', function(e) {
  // if clicked on marker, ignore (marker handles click)
  if (e.target.classList.contains('playerMarker')) return;
  if (!currentUser) { alert('Войдите в аккаунт чтобы перемещаться'); return; }
  const rect = mapWrapper.getBoundingClientRect();
  // map coordinate = mouse position inside wrapper minus current offset (pan)
  const x = e.clientX - rect.left - offsetX;
  const y = e.clientY - rect.top  - offsetY;
  savePositionForCurrentUser(x, y); // uses myFloor if set
});

/* ================= Login / UI ================= */
function openPage(page) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(page).classList.add('active');
  document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
  document.getElementById('tab_' + page).classList.add('active');
}

function login() {
  const log = document.getElementById('login_input').value.trim();
  const pass = document.getElementById('pass_input').value.trim();
  const acc = accounts.find(a => a.username === log && a.password === pass);
  if (!acc) {
    document.getElementById('login_status').style.color = '#f55';
    document.getElementById('login_status').innerText = "Неверные данные";
    return;
  }
  currentUser = acc.username;
  // read user's stored floor to update myFloor var
  db.ref('positions/' + currentUser).once('value').then(snap => {
    const pos = snap.val();
    myFloor = pos && pos.floor !== undefined ? pos.floor : null;
    setMyFloorLabel();
  });
  document.getElementById('logo').innerText = acc.displayName;
  document.getElementById('infoBox').innerText = `Вы вошли как: ${acc.displayName}`;
  document.getElementById('avatarBox').innerHTML = acc.avatar ? `<img src='${acc.avatar}' alt='Аватар' style="max-width:100%;">` : '';
  document.getElementById('descriptionBox').innerText = acc.description || '';
  document.getElementById('characterSheet').innerHTML = acc.sheet ? `<img src='${acc.sheet}' alt='Лист персонажа'>` : '';
  document.getElementById('login_status').style.color = '#5f5';
  document.getElementById('login_status').innerText = "Успешный вход";
  openPage('map');
  // make sure view label and background updated
  setViewFloorLabel(); setMyFloorLabel(); applyMapBackground();
  // load markers (listener already set)
}

/* ================= Init UI ================= */
setViewFloorLabel();
setMyFloorLabel();
applyMapBackground();

/* Ensure markers for all existing accounts are created even before DB has data */
accounts.forEach(a => ensureMarkerElement(a.username, a));

/* On first load, refresh markers visibility */
refreshMarkersVisibility();

</script>
</body>
</html>
