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
    { username: "цумуги", displayName: "Цумуги Широгане", password: "1111", color: "blue", avatar: "https://i.postimg.cc/MKDL3Brk/IMG-20251212-023652-731.jpg", description: "", sheet: "" },
    { username: "нагито", displayName: "Нагито Комаэда", password: "1111", color: "white", avatar: "", description: "", sheet: "" },
    { username: "миу", displayName: "Миу Ирума", password: "1111", color: "pink", avatar: "", description: "", sheet: "" },
    { username: "каэде", displayName: "Каэде Акаматсу", password: "1111", color: "yellow", avatar: "", description: "", sheet: "" },
    { username: "рантаро", displayName: "Рантаро Амами", password: "1111",  avatar: "https://i.postimg.cc/XqxgLf7X/IMG-20251212-024217-121.png", description: `Падший паладин.

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
    { username: "чихиро", displayName: "Чихиро Фуджисаки", password: "1111", color: "brown", avatar: "", description: "", sheet: "" },
    { username: "кай", displayName: "Кай Монтего", password: "1111", color: "purple", avatar: "https://i.pinimg.com/474x/81/31/7f/81317f3efab2be3909ea95e53439b75c.jpg", description: `Амбициозный мошенник.

Кай Монтего родился в семье известных актёров в эльфийском королевстве, где власть избирают мудрецы, леса бесконечны, а вера в Сириус является нормой, а не обязанностью. Его дом с детства был наполнен репетициями, голосами, инструментами и шумом сцены — он вырос в мире, где каждый говорит громче, чем думает, где важно не что ты чувствуешь, а как это выглядит со стороны.

Он рано начал выступать — сначала в детских постановках, позже в больших театрах столицы, где его запоминали за энергичность и смелость. Кай обожал внимание публики, зависел от него, как растение от солнца. Когда зрители аплодировали — он чувствовал себя великим. Когда молчали — казалось, что внутри него пусто. Именно тогда он впервые попробовал другое искусство — искусство обмана.

Во время гастролей он заметил, что люди охотнее раскрывают кошельки, если верят не артисту, а другу. Он стал играть роли прямо на улицах — потерянного отпрыска, доверчивого новичка, случайного прохожего, которому «нужна помощь». Так он открыл в себе плута: ловкого в словах, остроумного, умеющего подстраиваться и менять личности как костюмы.

Слава росла, но вместе с ней рос и внутренний страх быть разоблачённым. Кай всё чаще критиковал себя, считал, что без масок он пуст, а талант — всего лишь удачная иллюзия. Родители любили его, но он всегда думал, что должен стать лучше, ярче, громче, чтобы их гордость была заслуженной, а не автоматической.

Он участвовал в спектаклях столицы, путешествовал между городами, выступал в парламенте мудрецов на культурных фестивалях. Иногда за кулисами пропадали драгоценности, иногда зрители уходили с облегчёнными кошельками — но никто не мог связать это с улыбчивым юным артистом. Он был хорош, слишком хорош, чтобы сомненье задержалось надолго.

Однажды, после успешного выступления на летнем празднике, Кая пригласили на частную встречу. Незнакомец наблюдал за ним долго, изучал поведение, эмоции, слабости. Тогда Кай впервые услышал предложение участвовать в игре — опасной, закрытой, смертельной. Но в этом была слава, внимание, шанс войти в историю. Не как актёр — как легенда.

Он согласился почти без размышлений, ощущая азарт, предвкушая громкие овации, которые услышат даже за пределами королевства.

Он не знал, что организатор видел в нём удобную марионетку — слишком яркую, чтобы не заметили, и слишком зависимую от признания, чтобы отказаться. Теперь судьба Кая — больше не только театр. Это сцена, где цена спектакля — жизнь.`, sheet: "https://i.postimg.cc/pLyrSWTk/sheet7.jpg" },
    { username: "кокичи", displayName: "Кокичи Ома", password: "1111", color: "violet", avatar: "https://i.postimg.cc/yxSZg0gm/IMG-20251212-030500-800.png", description: `Голос Обмана+

Настоящее имя неизвестно.
На сцене его знали под сотней масок, но имя Кокичи Ома он выбрал сам — в честь мертвого героя, которого никто, кроме него, не помнит.

Он вырос в балагане на краю империи: фальшивые чудеса, поддельные пророчества, ядовитый грим и настоящая нищета. Родителей заменили актёры, друзей — роли. Искренность высмеивалась. Ложь — поощрялась. Там он научился: правда не ведёт за собой толпу, но хорошая история — всегда.

С малых лет он играл бога, дракона, шутника, мертвеца, вождя.
Но однажды он сорвал с себя все образы — и остался пуст. Бросив труппу, он ушёл в мир, чтобы играть теперь уже свою собственную игру.

Он лжёт, чтобы смешить. Он провоцирует, чтобы люди увидели себя. Он добр, но никогда прямо. Он обманщик — но не предатель.
Для него ложь — кисть, а реальность — холст. И если придётся перевернуть мир, чтобы сделать его красивее — он не задумываясь это сделает.`, sheet: "https://i.postimg.cc/63pV37fD/IMG-20251212-024143-183.jpg" },
    { username: "изуру", displayName: "Изуру Камукура", password: "1111", color: "gray", avatar: "https://i.postimg.cc/mkQrWjF2/IMG-20251212-025433-144.png", description: `Уединение. Осознание.

Изуру Камукура родился среди эльфов, но его происхождение тщательно скрыто. Ни одного подтверждённого факта о его семье, детстве или наставниках не существует.

В возрасте, не указанном ни в одном эльфийском реестре, он ушёл в горы — один. Причины ухода не названы. Добровольное изгнание, без следов и записей.

В течение 17 лет находился в полной изоляции. Единственное, что известно о его деятельности в этот период — фрагментарные находки: философские заметки без автора, рисунки из символов, совпадающих с древними молитвами, выжженные на камне круги.

Первое появление в обществе — случайное. Он спас умирающего ребёнка, не назвав имени, и исчез до рассвета. Потом снова — в разрушенном храме, где тела лежали нетронутыми, а стены были очищены.

Он не упоминает богов, но его заклинания работают. Он говорит редко, чаще — наблюдает. Не вступает в конфликты, если в этом нет необходимости. Его вера — не в существо, а в порядок или хаос как природное явление.

Именем Изуру Камукура пользуется сам. Считается, что оно не настоящее. Личность до изоляции не установлена. Возраст не определён.

Его цель неясна. Его путь не объясняется. Он просто идёт вперёд.`, sheet: "https://i.postimg.cc/4x4xjzfV/IMG-20251212-025454-723.jpg" },
    { username: "монака", displayName: "Монака Това", password: "1111", color: "lightgreen", avatar: "https://i.postimg.cc/J4vp2Q57/IMG-20251212-025905-402.png", description: `Святое притворство:

Когда-то в глухой долине, за чёрными болотами и зловещим холмом стоял приют под названием Дом Милосердия Элании. Местные шептались, что дети туда попадают не по воле богов, и что сама земля вокруг приюта плачет ночью. Именно там, в дождливую ночь, была найдена крошечная девочка — наполовину эльфийка, с чёртами лица столь хрупкими, что она казалась игрушечной.
Её назвали Монакой.
С юных лет она проявляла необычную проницательность. Она могла рассказать, кто из воспитателей лжёт, а кто — ворует еду. Но, вместо того чтобы быть благодарной, Монака начала играть с их слабостями. Вскоре в приюте вспыхнули конфликты, начались доносы, исчезновения… и пожары. Никто не мог доказать, что это она, но все знали — всё началось с Монаки.
В 10 лет она уже командовала младшими детьми как предводительница маленькой секты, говоря, что взрослые не настоящие, а миром должны править дети, свободные от морали.
Однажды ночью она устроила массовый побег из приюта, уведя за собой целую группу посвящённых.

Монака участвовала в ограблениях, поджогах и подстрекательстве к убийствам, прикрываясь маской невинного дитя. Она использовала священные символы, чтобы убедить крестьян, что их обманывают местные жрецы, а затем развращала паству, создавая новую веру — культ детской власти.
Когда орден Пелора попытался остановить её, она сдалась без боя, улыбаясь, и прошептала:
«Пока вы стареете и боитесь, я останусь маленькой. И дети меня запомнят».

Один из жрецов, по странной жалости, не казнил её, а предложил исправление — направить её через путь служения. Монака согласилась… но только для того, чтобы стать жрецом, изучить силу веры и обратить её в оружие.`, sheet: "https://i.postimg.cc/6qZGXqjJ/Picsart-25-07-05-11-30-55-133-1.jpg" },
    { username: "сония", displayName: "Сония Невермaйнд", password: "1111", color: "lightyellow", avatar: "", description: "", sheet: "" },
    { username: "кируми", displayName: "Кируми Тоджо", password: "1111", color: "darkviolet", avatar: "", description: "", sheet: "" },
    { username: "леон", displayName: "Леон Кувата", password: "1111", color: "orange", avatar: "", description: "", sheet: "" },
    { username: "макото", displayName: "Макото Наэги", password: "1111", color: "lightbrown", avatar: "", description: "", sheet: "" }
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
  el.style.backgroundImage = acc && acc.avatar ? `url('${acc.avatar}')` : `radial-gradient(circle at 30% 30%, #fff 0, ${acc && acc.color ? acc.color : '#888'} 60%)`;
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
