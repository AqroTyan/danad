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
    { username: "цумуги", displayName: "Цумуги Широгане", password: "0387", color: "blue", avatar: "https://i.postimg.cc/MKDL3Brk/IMG-20251212-023652-731.jpg",markerAvatar: "", description: "ыыы", sheet: "" },
    { username: "нагито", displayName: "Нагито Комаэда", password: "1492", color: "white", avatar: "",markerAvatar: "", description: "", sheet: "" },
    { 
  username: "миу",
  displayName: "Миу Ирума",
  password: "2750",
  color: "pink",
  avatar: "https://i.postimg.cc/pXPP3DMm/IMG-20260207-215033-607.jpg",
  markerAvatar: "https://i.postimg.cc/PJQTPqYt/IMG-20260207-215010-115.png",
  description: `Демоничный колдун.

Миу родилась человеком в бедной семье ремесленников. С юности отличалась дерзким характером и желанием доказать своё превосходство над другими. Когда в ней проявились магические задатки, она поступила в столичную академию и быстро освоила азы колдовства. Но, даже закончив обучение, Миу чувствовала, что её сил недостаточно.

Гонимая жаждой могущества, она обратилась к древним ритуалам тифлингов и заключила договор с низшим демоном Ignus, рождённым из сущности Терры. Сделка изменила её навсегда: её тело стало носителем демонической крови, а в глазах загорелся огонь пламени. Так человек превратился в тифлинга, заклеймённого собственной жадностью до силы.

Магия Миу сочетает в себе дерзость и практичность. Она свободно пользуется простыми заговорами — «Волшебная рука», «Чудотворство», «Ядовитые брызги», — а также владеет более сильными чарами: «Очарование личности», чтобы подчинять волю, и «Защита от зла и добра», чтобы отражать чужое тёмное влияние.

Миу презирает тех, кто называет её проклятой или продажной: для неё это лишь доказательство её смелости. Она гордится тем, что решилась на то, чего другие боятся — отказаться от человечности ради настоящей силы.`,
  sheet: "https://i.postimg.cc/fL58ynkQ/IMG-20260208-001546-912.jpg"
},
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

В течение 17 лет находился в полной изоляции. Единственное, что известно о его деятельности в этот период — фрагментарные находки: философские заметки без автора, рисунки из символов, с
