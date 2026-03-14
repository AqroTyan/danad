<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Монопад</title>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #1a1a1a; color: white; overflow: hidden; }
    header { background: #222; padding: 15px; text-align: center; font-size: 22px; font-weight: bold; border-bottom: 2px solid #ff2b2b; }
    nav { display: none; background: #333; display: flex; z-index: 100; position: relative; }
    nav button { flex: 1; padding: 15px; background: #333; border: none; color: white; font-size: 14px; cursor: pointer; }
    nav button.active { background: #444; border-bottom: 3px solid #ff2b2b; }
    
    .page { display: none; padding: 20px; height: calc(100vh - 120px); overflow-y: auto; box-sizing: border-box; }
    .page.active { display: block; }

    /* --- КАРТА --- */
    #map { padding: 0; overflow: hidden; position: relative; height: calc(100vh - 110px); background: #000; }
    #mapControls { 
        position: absolute; top: 10px; left: 10px; z-index: 50; 
        display:flex; gap:5px; flex-direction: column; background: rgba(0,0,0,0.8); padding: 10px; border-radius: 8px; border: 1px solid #ff2b2b;
    }
    .floorBtn, .myFloorBtn { padding: 8px; border-radius: 4px; border: 1px solid #666; background: #333; color: white; cursor: pointer; font-size: 12px; }
    .floorBtn.active { background: #ff2b2b; border-color: #fff; }

    #mapWrapper { width: 100%; height: 100%; overflow: hidden; position: relative; touch-action: none; cursor: grab; }
    #mapArea { 
        width: 1600px; height: 1600px; position: absolute; top: 0; left: 0; 
        transform-origin: 0 0; will-change: transform; background-repeat: no-repeat; background-size: contain;
    }

    /* МАРКЕР И ТУЛТИП */
    .playerMarker {
      width: 34px; height: 34px; position: absolute; transform: translate(-50%, -50%);
      background-size: cover; border-radius: 50%; border: 2px solid white; box-shadow: 0 0 10px black; z-index: 5;
    }
    .markerTooltip {
      position: absolute; bottom: 45px; left: 50%; transform: translateX(-50%);
      width: 100px; height: 100px; border-radius: 10px; border: 2px solid #ff2b2b;
      background-size: cover; background-position: center; background-color: #000;
      display: none; pointer-events: none; z-index: 10; box-shadow: 0 0 15px rgba(255,0,0,0.7);
    }
    .playerMarker:hover .markerTooltip { display: block; }

    /* ПРАВИЛА */
    .rulesPad { max-width: 600px; margin: 0 auto; background: #0b0b0b; border: 2px solid #8b0000; padding: 20px; border-radius: 10px; box-shadow: 0 0 15px red; }
    .rulesTitle { color: #ff2b2b; text-align: center; font-size: 24px; text-shadow: 0 0 10px red; }
    .rulesText p { color: #ff3b3b; font-size: 15px; border-bottom: 1px dashed #400; padding: 5px 0; }

    /* УЛИКИ */
    .bulletItem { background: #222; padding: 15px; margin-bottom: 10px; border-radius: 8px; border-left: 5px solid #ff2b2b; cursor: pointer; }
    #bulletModal { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.9); align-items:center; justify-content:center; z-index:2000; }
    .modalContent { background:#222; padding:25px; width:85%; max-width:400px; border-radius:10px; border: 2px solid #ff2b2b; }

    /* OBJECTION */
    #objectionOverlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.95); z-index: 9999; align-items: center; justify-content: center; text-align: center; }
    .obj-card { background: #111; border: 5px solid #ff2b2b; padding: 30px; border-radius: 20px; width: 85%; max-width: 500px; animation: pop 0.3s; }
    .obj-badge { font-size: 40px; color: #fff; background: #ff2b2b; padding: 10px; transform: rotate(-3deg); font-weight: 900; display: inline-block; margin-bottom: 20px; }
    @keyframes pop { from { transform: scale(0.5); } to { transform: scale(1); } }
  </style>
</head>
<body>

<header id="mainHeader">Монопад</header>
<nav id="mainNav">
  <button onclick="openPage('home')" id="tab_home">Главная</button>
  <button onclick="openPage('map')" id="tab_map">Карта</button>
  <button onclick="openPage('info')" id="tab_info">Профиль</button>
  <button onclick="openPage('rules')" id="tab_rules">Правила</button>
  <button onclick="openPage('bullets')" id="tab_bullets">Улики</button>
</nav>

<div class="page active" id="home">
  <div id="logo" style="text-align:center; font-size:40px; margin-top:40px; text-shadow:0 0 10px red;">MONOPAD</div>
  <div id="loginBox" style="margin:40px auto; width:280px; padding:20px; background:#2b2b2b; border-radius:8px;">
    <input id="login_input" placeholder="Логин" style="width:100%; padding:10px; margin-top:10px; border-radius:5px; border:none;">
    <input id="pass_input" placeholder="Пароль" type="password" style="width:100%; padding:10px; margin-top:10px; border-radius:5px; border:none;">
    <button onclick="login()" style="width:100%; margin-top:15px; padding:12px; background:#ff2b2b; color:white; border:none; border-radius:5px; font-weight:bold; cursor:pointer;">ВОЙТИ</button>
    <div id="login_status" style="margin-top:10px; text-align:center; color:#f55;"></div>
  </div>
</div>

<div class="page" id="map">
  <div id="mapControls">
    <span style="font-size:10px">ВИД ЭТАЖА:</span>
    <button class="floorBtn" id="v_1" onclick="changeViewFloor(1)">1F</button>
    <button class="floorBtn" id="v_2" onclick="changeViewFloor(2)">2F</button>
    <button class="floorBtn" id="v_b" onclick="changeViewFloor('b')">BF</button>
    <hr style="width:100%; border:0; border-top:1px solid #444;">
    <span style="font-size:10px">Я НА ЭТАЖЕ:</span>
    <button class="myFloorBtn" onclick="setMyFloor(1)">1</button>
    <button class="myFloorBtn" onclick="setMyFloor(2)">2</button>
    <button class="myFloorBtn" onclick="setMyFloor('b')">B</button>
  </div>
  <div id="mapWrapper"><div id="mapArea"></div></div>
</div>

<div class="page" id="info">
  <div style="display:flex; flex-direction:column; align-items:center; gap:20px;">
    <div id="avatarBox" style="width:180px; height:180px; border:4px solid #ff2b2b; background-size:cover; border-radius:15px;"></div>
    <h2 id="charNameDisplay" style="margin:0; color:#ff2b2b;"></h2>
    <div id="descriptionBox" style="background:#222; padding:15px; border-radius:10px; font-size:14px;"></div>
    <div id="characterSheet" style="width:100%;"></div>
  </div>
</div>

<div class="page" id="rules">
  <div class="rulesPad">
    <div class="rulesTitle">ВНУТРЕННИЙ РЕГЛАМЕНТ</div>
    <div class="rulesText" style="margin-top:15px;">
      <p>1. убийство разрешено лишь с мотивом победить.</p>
      <p>2. После обнаружения тела начинается расследование .</p>
      <p>3. Суд обязателен для всех участников.</p>
      <p>4. Если виновный не будет разоблачен, выживет только он.</p>
      <p>5. Нарушение правил наказывается немедленно. </p>
      <p>6 попытки уничтожить монопод запрещены</p>
      <p>7 монокума не может быть задействован в убийстве</p>
      <p>8 никто не может трогать монокуму без его разрешения</p>
    </div>
  </div>
</div>

<div class="page" id="bullets">
  <button onclick="openAddBullet()" style="width:100%; padding:15px; background:#444; color:white; border:none; border-radius:8px; font-weight:bold;">+ НОВАЯ УЛИКА</button>
  <div id="bulletsList" style="margin-top:20px;"></div>
</div>

<div id="bulletModal">
  <div class="modalContent">
    <input id="bulletName" placeholder="Название" style="width:100%; background:#333; color:white; border:none; padding:10px; border-radius:5px;">
    <textarea id="bulletDesc" placeholder="Описание" style="width:100%; background:#333; color:white; border:none; padding:10px; border-radius:5px; margin-top:10px; height:80px;"></textarea>
    <input id="bulletEmoji" placeholder="Эмодзи" style="width:100%; background:#333; color:white; border:none; padding:10px; border-radius:5px; margin-top:10px;">
    
    <div style="display:flex; gap:10px; justify-content:space-between; margin-top:20px;">
      <div style="display:flex; gap:5px;">
        <button onclick="closeBulletModal()">Отмена</button>
        <button id="deleteBulletBtn" style="background:#f66; color:white; display:none;" onclick="deleteCurrentBullet()">УДАЛИТЬ</button>
      </div>
      <div style="display:flex; gap:5px;">
        <button id="presentBtn" style="background:#ff2b2b; color:white; display:none;" onclick="presentToMonokuma()">ПРЕДЪЯВИТЬ</button>
        <button onclick="saveBullet()" style="background:#4CAF50; color:white;">ОК</button>
      </div>
    </div>
  </div>
</div>

<div id="objectionOverlay">
  <div class="obj-card">
    <div class="obj-badge">OBJECTION!</div>
    <div id="objSender" style="color:#ff2b2b; font-weight:bold;"></div>
    <h3 id="objTitle" style="color:white;"></h3>
    <p id="objText" style="background:#000; padding:15px; border-radius:10px; font-size:14px; text-align:left;"></p>
    <button onclick="closeObjection()" style="margin-top:15px; padding:10px 30px; cursor:pointer;">ПОНЯТНО</button>
  </div>
</div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
const firebaseConfig = {
  apiKey: "AIzaSyBiHWlMDeUv5FmPh4Aqv7aKCGHFbco5YIM",
  authDomain: "dandd-592cb.firebaseapp.com",
  databaseURL: "https://dandd-592cb-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "dandd-592cb",
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

const accounts = [
    { username: "цумуги", displayName: "Цумуги Широгане", password: "0387", color: "blue", avatar: "https://i.postimg.cc/MKDL3Brk/IMG-20251212-023652-731.jpg",markerIcon: "https://i.postimg.cc/NjJg3gs7/IMG-20260214-145036-602.png", description: "", sheet: "" },
    { username: "нагито", displayName: "Нагито Комаэда", password: "1492", color: "white", avatar: "https://i.postimg.cc/1369x3DX/IMG-20260212-224508-534.png",markerIcon: "https://i.postimg.cc/fbmrxKry/IMG-20260214-145036-728.png", description: "Святой мученик.                                      Сын полуэльфа-учёного и благородной жрицы, Нагито с детства рос под сводами Великого Храма Стеллы. Ему предсказывали блестящее будущее: ясный ум, врождённое обаяние и редкая способность вызывать доверие у людей делали его любимцем наставников.Но счастье было недолгим. Едва достигнув зрелости, Нагито пережил серию страшных личных трагедий — сначала умерла мать, затем отец погиб в странных обстоятельствах, а вскоре загадочная болезнь унесла почти всех его друзей. Каждый раз он оставался единственным, кто чудом выживал. Эти совпадения укрепили в нём болезненную веру: Стелла дарует ему «священную удачу» — но лишь ценой чужих жизней.Его необычная харизма и готовность принять любой удар судьбы сделали его самым молодым Верховным Жрецом в истории церкви. Под его руководством Великий Храм превратился в место паломничества: одни приходят за исцелением, другие — чтобы испытать «путь очищения через страдание», который он проповедует. Для народа он — живое чудо. Для некоторых жрецов — опасный фанатик, готовый бросить любого в объятия судьбы, если это приблизит их к свету Стеллы.", sheet: "https://i.postimg.cc/4yq62CYD/IMG-20260214-145949-097.jpg" },
    { username: "миу", displayName: "Миу Ирума", password: "2750", avatar: "https://i.postimg.cc/pXPP3DMm/IMG-20260207-215033-607.jpg",markerIcon: "https://i.postimg.cc/PJQTPqYt/IMG-20260207-215010-115.png", description: `Демоничный колдун.                                                                                                                                                      
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
    { username: "каэде", displayName: "Каэде Акаматсу", password: "4069", color: "yellow", avatar: "https://i.postimg.cc/N0FvsGHT/IMG-20260213-002748-276.png",markerIcon: "https://i.postimg.cc/13S8Z1sp/IMG-20260108-122721-302.png", description: "Звёздный избранник.                                 Когда родилась в маленькой эльфийской деревне на окраине государства, где густые леса соприкасаются с землями людей. С ранних лет она чувствовала себя «на границе миров»: слушала древние песни о Сириусе, боге всего живого, и одновременно слышала истории о людях и их вере в Стеллу. Её родные почитали природу и жили в гармонии, но сама Каэде всегда мечтала о большем, чем жизнь в тихой глуши.Когда в ней пробудился магический талант, её отправили в эльфийскую академию. Там она поразила наставников необычным подходом к магии: заклинания в её руках звучали как музыка, будто отражение внутренней гармонии. Она верила, что сама сила Сириуса проявляется через мелодию, соединяющую всё живое.Но Каэде не могла довольствоваться только изучением чар. Она всё больше задумывалась о том, что два государства, столь разные в вере, не должны оставаться разделёнными. Для неё магия и музыка были не оружием, а ключом к объединению. Многие считали её идеалисткой, но в народе начали шептаться о пророчестве: «звезда, рождённая на границе, зазвучит и для людей, и для эльфов».     Теперь Каэде путешествует по миру скитаясь, неся с собой свет знаний и гармонии. Для эльфов она — дитя Сириуса, для людей — загадочный посредник, а для самой себя — всего лишь волшебница, которая пытается сыграть мелодию, способную изменить судьбу обоих народов.", sheet: "https://i.postimg.cc/4NQDpHvw/IMG-20260213-003330-914.jpg" },
    { username: "рантаро", displayName: "Рантаро Амами", password: "5821",  avatar: "https://i.postimg.cc/XqxgLf7X/IMG-20251212-024217-121.png",markerIcon: "https://i.postimg.cc/T38jr6Mn/IMG-20260214-145036-158.png", description: `Падший паладин.

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
    { username: "чихиро", displayName: "Чихиро Фуджисаки", password: "6825", color: "brown", avatar: "https://i.postimg.cc/KYSvH4HK/IMG-20260126-174934-610.png",markerIcon: "https://i.postimg.cc/BvrDNqxR/IMG-20260126-174934-578.png", description: "Тихий хранитель.                                       Чихиро родился в зелёных лесах эльфийского королевства, где каждое дерево старше любого из мудрецов, а магия течёт сквозь землю так же естественно, как кровь сквозь вены. Эльфы жили в согласии с природой, под покровительством Сириуса — богини всего живого, что учит видеть жизнь во всём, даже в мху на камне.Он вырос в эпоху перемен: Парламент Мудрецов готовился к коронации новой королевы Сонии — монархини, призванной не править, а напоминать эльфам о единстве. В те годы Чихиро служил при местном святилище как прислужник-друид, ухаживая за рощами, животными и источниками, где вода считалась «дыханием Сириуса».Он был робким, тихим и застенчивым юношей. Часто избегал разговоров, предпочитая слушать треск ветра и шелест листьев. Его мир — это не города и речи Мудрецов, а запах земли после дождя, трепетное движение травы под солнцем.Сослужители смеялись: «Чихиро ближе к оленю, чем к эльфу».Но никто не мог отрицать, что природа отвечала ему взаимностью — растения оживали под его руками, звери не боялись подходить близко.Когда начались волнения на северных границах — там, где людские караваны нарушали покой священных земель, — Чихиро не взял оружие. Он ушёл туда один, чтобы умолить природу не ответить яростью на ярость.С тех пор он странствует между рощами, помогая тому, что живо, и избегая того, что жадно.Он не ищет славы и не ждёт награды. Он просто живёт по завету своей богини:«Слушай, не властвуй. Береги, не приказывай. Живи — и давай жить.»", sheet: "https://i.postimg.cc/q7x0Dwnf/IMG-20260127-181059-011.jpg" },
    { username: "кай", displayName: "Кай Монтего", password: "6904", color: "purple", avatar: "https://i.postimg.cc/GhK4rS2J/IMG-20260108-122721-262.png",markerIcon: "https://i.postimg.cc/Kjc0WHj9/IMG-20260108-122721-404.png", description: `Амбициозный мошенник.

Кай Монтего родился в семье известных актёров в эльфийском королевстве, где власть избирают мудрецы, леса бесконечны, а вера в Сириус является нормой, а не обязанностью. Его дом с детства был наполнен репетициями, голосами, инструментами и шумом сцены — он вырос в мире, где каждый говорит громче, чем думает, где важно не что ты чувствуешь, а как это выглядит со стороны.

Он рано начал выступать — сначала в детских постановках, позже в больших театрах столицы, где его запоминали за энергичность и смелость. Кай обожал внимание публики, зависел от него, как растение от солнца. Когда зрители аплодировали — он чувствовал себя великим. Когда молчали — казалось, что внутри него пусто. Именно тогда он впервые попробовал другое искусство — искусство обмана.

Во время гастролей он заметил, что люди охотнее раскрывают кошельки, если верят не артисту, а другу. Он стал играть роли прямо на улицах — потерянного отпрыска, доверчивого новичка, случайного прохожего, которому «нужна помощь». Так он открыл в себе плута: ловкого в словах, остроумного, умеющего подстраиваться и менять личности как костюмы.

Слава росла, но вместе с ней рос и внутренний страх быть разоблачённым. Кай всё чаще критиковал себя, считал, что без масок он пуст, а талант — всего лишь удачная иллюзия. Родители любили его, но он всегда думал, что должен стать лучше, ярче, громче, чтобы их гордость была заслуженной, а не автоматической.

Он участвовал в спектаклях столицы, путешествовал между городами, выступал в парламенте мудрецов на культурных фестивалях. Иногда за кулисами пропадали драгоценности, иногда зрители уходили с облегчёнными кошельками — но никто не мог связать это с улыбчивым юным артистом. Он был хорош, слишком хорош, чтобы сомненье задержалось надолго.

Однажды, после успешного выступления на летнем празднике, Кая пригласили на частную встречу. Незнакомец наблюдал за ним долго, изучал поведение, эмоции, слабости. Тогда Кай впервые услышал предложение участвовать в игре — опасной, закрытой, смертельной. Но в этом была слава, внимание, шанс войти в историю. Не как актёр — как легенда.

Он согласился почти без размышлений, ощущая азарт, предвкушая громкие овации, которые услышат даже за пределами королевства.

Он не знал, что организатор видел в нём удобную марионетку — слишком яркую, чтобы не заметили, и слишком зависимую от признания, чтобы отказаться. Теперь судьба Кая — больше не только театр. Это сцена, где цена спектакля — жизнь.`, sheet: "https://i.postimg.cc/pLyrSWTk/sheet7.jpg" },
    { username: "кокичи", displayName: "Кокичи Ома", password: "7318", color: "violet", avatar: "https://i.postimg.cc/yxSZg0gm/IMG-20251212-030500-800.png",markerIcon: "https://i.postimg.cc/4NWpQ6Jf/IMG-20260214-000501-523.png", description: `Голос Обмана+

Настоящее имя неизвестно.
На сцене его знали под сотней масок, но имя Кокичи Ома он выбрал сам — в честь мертвого героя, которого никто, кроме него, не помнит.

Он вырос в балагане на краю империи: фальшивые чудеса, поддельные пророчества, ядовитый грим и настоящая нищета. Родителей заменили актёры, друзей — роли. Искренность высмеивалась. Ложь — поощрялась. Там он научился: правда не ведёт за собой толпу, но хорошая история — всегда.

С малых лет он играл бога, дракона, шутника, мертвеца, вождя.
Но однажды он сорвал с себя все образы — и остался пуст. Бросив труппу, он ушёл в мир, чтобы играть теперь уже свою собственную игру.

Он лжёт, чтобы смешить. Он провоцирует, чтобы люди увидели себя. Он добр, но никогда прямо. Он обманщик — но не предатель.
Для него ложь — кисть, а реальность — холст. И если придётся перевернуть мир, чтобы сделать его красивее — он не задумываясь это сделает.`, sheet: "https://i.postimg.cc/63pV37fD/IMG-20251212-024143-183.jpg" },
    { username: "изуру", displayName: "Изуру Камукура", password: "8046", color: "gray", avatar: "https://i.postimg.cc/RVZ6zK07/IMG-20260213-000428-127.png",markerIcon: "https://i.postimg.cc/FFDng1DB/IMG-20260214-145035-916.png", description: `Уединение. Осознание.

Изуру Камукура родился среди эльфов, но его происхождение тщательно скрыто. Ни одного подтверждённого факта о его семье, детстве или наставниках не существует.

В возрасте, не указанном ни в одном эльфийском реестре, он ушёл в горы — один. Причины ухода не названы. Добровольное изгнание, без следов и записей.

В течение 17 лет находился в полной изоляции. Единственное, что известно о его деятельности в этот период — фрагментарные находки: философские заметки без автора, рисунки из символов, совпадающих с древними молитвами, выжженные на камне круги.

Первое появление в обществе — случайное. Он спас умирающего ребёнка, не назвав имени, и исчез до рассвета. Потом снова — в разрушенном храме, где тела лежали нетронутыми, а стены были очищены.

Он не упоминает богов, но его заклинания работают. Он говорит редко, чаще — наблюдает. Не вступает в конфликты, если в этом нет необходимости. Его вера — не в существо, а в порядок или хаос как природное явление.

Именем Изуру Камукура пользуется сам. Считается, что оно не настоящее. Личность до изоляции не установлена. Возраст не определён.

Его цель неясна. Его путь не объясняется. Он просто идёт вперёд.`, sheet: "https://i.postimg.cc/4x4xjzfV/IMG-20251212-025454-723.jpg" },
    { username: "монака", displayName: "Монака Това", password: "9163", color: "lightgreen", avatar: "https://i.postimg.cc/J4vp2Q57/IMG-20251212-025905-402.png",markerIcon: "https://i.postimg.cc/Pqvk0YFs/IMG-20260214-145036-434.png", description: `Святое притворство:

Когда-то в глухой долине, за чёрными болотами и зловещим холмом стоял приют под названием Дом Милосердия Элании. Местные шептались, что дети туда попадают не по воле богов, и что сама земля вокруг приюта плачет ночью. Именно там, в дождливую ночь, была найдена крошечная девочка — наполовину эльфийка, с чёртами лица столь хрупкими, что она казалась игрушечной.
Её назвали Монакой.
С юных лет она проявляла необычную проницательность. Она могла рассказать, кто из воспитателей лжёт, а кто — ворует еду. Но, вместо того чтобы быть благодарной, Монака начала играть с их слабостями. Вскоре в приюте вспыхнули конфликты, начались доносы, исчезновения… и пожары. Никто не мог доказать, что это она, но все знали — всё началось с Монаки.
В 10 лет она уже командовала младшими детьми как предводительница маленькой секты, говоря, что взрослые не настоящие, а миром должны править дети, свободные от морали.
Однажды ночью она устроила массовый побег из приюта, уведя за собой целую группу посвящённых.

Монака участвовала в ограблениях, поджогах и подстрекательстве к убийствам, прикрываясь маской невинного дитя. Она использовала священные символы, чтобы убедить крестьян, что их обманывают местные жрецы, а затем развращала паству, создавая новую веру — культ детской власти.
Когда орден Пелора попытался остановить её, она сдалась без боя, улыбаясь, и прошептала:
«Пока вы стареете и боитесь, я останусь маленькой. И дети меня запомнят».

Один из жрецов, по странной жалости, не казнил её, а предложил исправление — направить её через путь служения. Монака согласилась… но только для того, чтобы стать жрецом, изучить силу веры и обратить её в оружие.`, sheet: "https://i.postimg.cc/6qZGXqjJ/Picsart-25-07-05-11-30-55-133-1.jpg" },
    { username: "сония", displayName: "Сония Невермaйнд", password: "2479", color: "lightyellow", avatar: "https://i.postimg.cc/QM2Lbkys/IMG-20260122-002814-353.png",markerIcon: "https://i.postimg.cc/fy3wLTF5/IMG-20260122-002811-777.png", description: "Запретное знание   Сония Невермайнд с рождения была предназначена стать символом эльфийского государства. Парламент Мудрецов существовал почти с момента сотворения королевства, и корона всегда была знаком преемственности, а не инструментом власти. Сонию готовили именно к этой роли: быть лицом государства, хранительницей традиций, точкой стабильности в мире, где даже леса меняются медленно, но неизбежно.Она была образованной, дисциплинированной и искренне доброй. Сония верила в законы, уважала решения парламента и никогда не ставила под сомнение саму систему. Однако её личный интерес лежал глубже официальных учений. С юности она увлекалась оккультизмом — не как запретным искусством, а как формой познания мира. Древние ритуалы, забытые символы, крайние философские течения, баланс между жизнью и смертью — всё это интересовало её как способ понять, где проходит граница дозволенного и необходимого.Её тянуло к экстремальным идеям не из жажды разрушения, а из желания проверить пределы: может ли порядок существовать без крайностей, и что происходит, когда ради сохранения жизни приходится идти на шаг за грань привычной морали. При этом Сония никогда не нарушала законов — она изучала, а не практиковала, наблюдала, а не действовала. Для парламента она оставалась идеальной кандидатурой: умной, сдержанной, надёжной.Именно это сочетание — абсолютная лояльность и интерес к запретному — привело её к открытию, которое не должно было касаться будущей королевы. Задолго до коронации Сония узнала о существовании механизма, скрытого за фасадом стабильности: системы, в которой крайние меры применялись тайно, во имя сохранения баланса мира. Убийственная игра была частью этой системы — не развлечением, а инструментом.Сония узнала об этом до того, как оказалась в игре.И поняла, что если она станет королевой, это знание сделает её либо лгуньей, либо соучастницей.Она приняла решение ещё тогда — молча, без свидетелей. Коронация должна была состояться, но не состоялась. Вместо трона она оказалась среди участников игры, единственная, кто понимал, что происходит и почему. Она не сопротивлялась и не пыталась сбежать, потому что знала: её присутствие — часть уже запущенного процесса.Сония остаётся доброй и сострадательной. Она помогает другим, поддерживает слабых и не поощряет насилие. Но внутри она несёт тяжёлое знание: иногда мир держится не на законах и не на символах, а на том, что кто-то соглашается заплатить цену молча.Корона не была утрачена.Она была отложена — вместе с правдой, которую нельзя было произнести вслух. ", sheet: "https://i.postimg.cc/s2f6WVCj/Picsart-26-01-22-14-22-32-655.jpg" },
    { username: "кируми", displayName: "Кируми Тоджо", password: "3508", color: "darkviolet", avatar: "[https://png.klev.club/10324-barli.html](https://static.wikia.nocookie.net/brawlstars/images/5/5a/%D0%91%D0%B0%D1%80%D0%BB%D0%B8_Skin-Default.png/revision/latest?cb=20250524203143&path-prefix=ru)",markerAvatar: "https://i.pinimg.com/736x/b1/18/8c/b1188ce053737a274590f131ac3b3d21.jpg", description: "", sheet: "" },
    { username: "леон", displayName: "Леон Кувата", password: "4682", color: "orange", avatar: "https://i.postimg.cc/k5ZCxDYG/IMG-20260116-232420-319.png",markerIcon: "https://i.postimg.cc/SK4Nnn5N/IMG-20260116-232420-592.png", description: "Благородный маг.      Леон Кувата родился в семье древнего человеческого рода, который веками служил магии и государству. С детства его обучали этикету, искусству и магическим практикам, ожидая, что он станет достойным наследником семейного имени. Несмотря на строгие рамки, Леон всегда сохранял жизнерадостный, острый ум и чувство юмора, что делало его популярным среди сверстников и наставников. Поступив в столичный университет магии, он блеснул талантом и умением сочетать теорию с практикой, быстро заслужив уважение профессоров. Благородство, воспитание и врождённая харизма позволили ему не только овладеть магией, но и обрести авторитет среди коллег и младших студентов. Став выпускником, Леон покинул университет, намереваясь использовать свои навыки и знания во благо людей и укрепление магического искусства, но при этом оставаясь верным духу приключений и исследовательской страсти, заложенной ещё в детстве.", sheet: "https://i.postimg.cc/kGmkcSQR/IMG-20260116-233738-319.jpg" },
    { username: "макото", displayName: "Макото Наэги", password: "5790", color: "lightbrown", avatar: "https://i.postimg.cc/CKgbZrJw/IMG-20260210-120752-235.png",markerIcon: "https://i.postimg.cc/xCgdVPpG/IMG-20260214-000502-462.png", description: "Страж троп.                                          Макото Наэги родился в одном из старых лесных кантонов эльфийского государства — местах, где парламент мудрецов ощущается лишь через законы, а не через лица. Его семья веками служила проводниками и хранителями троп: они не управляли лесом, а жили с ним, передавая знания не в книгах, а в привычках и молчании. Для них важнее всего были стабильность и сохранение порядка вещей.С ранних лет Макото помогал в приютах для верующих и странников, что объясняет его путь прислужника. Он носил воду, чинил кровлю, сопровождал стариков и детей по лесным дорогам. Именно там он научился внимательности к мелочам и людям — умению замечать усталость, страх и ложь раньше, чем слова. Он никогда не стремился быть лидером, но часто становился тем, кому доверяли.Став следопытом, Макото начал служить связующим звеном между общинами: передавал вести, сопровождал караваны, разыскивал пропавших в чащобах. Он плохо переносит насилие, но понимает его необходимость, предпочитая избегать прямых столкновений. Его стиль — осторожность, подготовка и путь назад. Он не охотник за славой, а гарант того, что кто-то вернётся домой.Макото глубоко привязан к семье и традициям, но именно это делает его уязвимым: он боится перемен и часто сомневается в собственных решениях. Его самокритичность граничит с безнадёжностью, особенно когда мир вокруг начинает рушиться быстрее, чем он способен его удержать.Он верит, что даже самый обычный человек может стать опорой для других — не потому, что должен, а потому что иначе некому.…а иногда судьба проверяет эту веру слишком жестоко.", sheet: "https://i.postimg.cc/zvsZsbNh/IMG-20260214-000734-095.jpg" },
  { username: "монокума", displayName: "Монокума", password: "8743", color: "red", avatar: "https://i.postimg.cc/5tq95ymv/Monokuma-Illustration.webp",markerIcon:"https://i.postimg.cc/2jXk7PnF/Ball-Monokuma-transparent-jpg.webp", description: "Директор школы." }
];

let currentUser = null, currentAcc = null, currentEditingId = null;
let viewFloor = 1, myFloor = 1, scale = 0.6, posX = 0, posY = 0;

const mapImages = {
  1: "https://i.postimg.cc/CMfrDjLK/Znimok-ekrana-2025-12-12-222248.png",
  2: "https://i.postimg.cc/3wBv6rC3/izobrazenie-2025-12-13-010749253.png",
  b: "https://i.postimg.cc/pTSQr1nK/Znimok-ekrana-2025-12-12-230210.png"
};

function login() {
  const u = document.getElementById('login_input').value.trim().toLowerCase();
  const p = document.getElementById('pass_input').value.trim();
  const acc = accounts.find(a => a.username === u && a.password === p);
  if(!acc) { document.getElementById('login_status').innerText = "Ошибка!"; return; }
  
  currentUser = acc.username; currentAcc = acc;
  document.getElementById('mainNav').style.display = 'flex';
  document.getElementById('home').classList.remove('active');
  
  document.getElementById('charNameDisplay').innerText = acc.displayName;
  document.getElementById('avatarBox').style.backgroundImage = `url(${acc.avatar})`;
  document.getElementById('descriptionBox').innerText = acc.description;
  document.getElementById('characterSheet').innerHTML = acc.sheet ? `<img src="${acc.sheet}" style="width:100%;">` : '';

  if(currentUser === "монокума") startMonokumaListener();
  db.ref('positions/' + currentUser).once('value').then(snap => {
    myFloor = snap.val() ? snap.val().floor : 1;
    openPage('map');
  });
}

function openPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
  document.getElementById('tab_'+id).classList.add('active');
  if(id === 'map') renderMap();
  if(id === 'bullets') loadBullets();
}

/* --- КАРТА: ЗУМ И ПЕРЕМЕЩЕНИЕ --- */
const area = document.getElementById('mapArea'), wrapper = document.getElementById('mapWrapper');
function updateTransform() { area.style.transform = `translate(${posX}px, ${posY}px) scale(${scale})`; }

let isDown = false, startX, startY, lastDist = 0;

wrapper.onmousedown = (e) => { isDown = true; startX = e.clientX - posX; startY = e.clientY - posY; };
window.onmousemove = (e) => { if(isDown) { posX = e.clientX - startX; posY = e.clientY - startY; updateTransform(); } };
window.onmouseup = () => isDown = false;
wrapper.onwheel = (e) => { scale = Math.min(Math.max(0.2, scale + (e.deltaY > 0 ? -0.05 : 0.05)), 2); updateTransform(); };

wrapper.addEventListener('touchstart', (e) => {
  if(e.touches.length === 1) { isDown = true; startX = e.touches[0].clientX - posX; startY = e.touches[0].clientY - posY; }
  else if(e.touches.length === 2) lastDist = Math.hypot(e.touches[0].clientX - e.touches[1].clientX, e.touches[0].clientY - e.touches[1].clientY);
});
wrapper.addEventListener('touchmove', (e) => {
  if(e.touches.length === 1 && isDown) { posX = e.touches[0].clientX - startX; posY = e.touches[0].clientY - startY; }
  else if(e.touches.length === 2) {
    let d = Math.hypot(e.touches[0].clientX - e.touches[1].clientX, e.touches[0].clientY - e.touches[1].clientY);
    scale = Math.min(Math.max(0.2, scale + (d - lastDist) * 0.005), 2);
    lastDist = d;
  }
  updateTransform();
});

area.onclick = (e) => {
    const rect = area.getBoundingClientRect();
    db.ref('positions/' + currentUser).update({ x: (e.clientX - rect.left) / scale, y: (e.clientY - rect.top) / scale, floor: myFloor });
};

function changeViewFloor(f) { 
  viewFloor = f; 
  document.querySelectorAll('.floorBtn').forEach(b => b.classList.remove('active'));
  document.getElementById('v_'+f).classList.add('active');
  renderMap(); 
}
function setMyFloor(f) { myFloor = f; db.ref('positions/'+currentUser+'/floor').set(f); renderMap(); }

function renderMap() {
  area.style.backgroundImage = `url(${mapImages[viewFloor]})`;
  updateTransform();
  loadMarkers();
}

function loadMarkers() {
  db.ref('positions').on('value', snap => {
    area.querySelectorAll('.playerMarker').forEach(m => m.remove());
    const data = snap.val();
    for(let u in data) {
      if(data[u].floor == viewFloor) {
        const acc = accounts.find(a => a.username === u);
        const m = document.createElement('div');
        m.className = 'playerMarker';
        m.style.left = data[u].x + 'px'; m.style.top = data[u].y + 'px';
        m.style.backgroundImage = `url(${acc ? acc.markerIcon : ''})`;

        const tool = document.createElement('div');
        tool.className = 'markerTooltip';
        tool.style.backgroundImage = `url(${acc ? acc.avatar : ''})`;
        
        m.appendChild(tool);
        area.appendChild(m);
      }
    }
  });
}

/* --- УЛИКИ --- */
function loadBullets() {
  db.ref('bullets/' + currentUser).on('value', snap => {
    const list = document.getElementById('bulletsList'); list.innerHTML = '';
    const data = snap.val();
    for(let id in data) {
      const b = data[id];
      const d = document.createElement('div');
      d.className = 'bulletItem'; d.onclick = () => openEditBullet(id, b);
      d.innerHTML = `<span>${b.emoji || '🔍'}</span> <b>${b.name}</b>`;
      list.appendChild(d);
    }
  });
}

function openAddBullet() {
  currentEditingId = null;
  document.getElementById('bulletName').value = ''; document.getElementById('bulletDesc').value = ''; 
  document.getElementById('presentBtn').style.display = 'none';
  document.getElementById('deleteBulletBtn').style.display = 'none';
  document.getElementById('bulletModal').style.display = 'flex';
}

function openEditBullet(id, b) {
  currentEditingId = id;
  document.getElementById('bulletName').value = b.name;
  document.getElementById('bulletDesc').value = b.desc;
  document.getElementById('bulletEmoji').value = b.emoji || '🔍';
  document.getElementById('presentBtn').style.display = 'block';
  document.getElementById('deleteBulletBtn').style.display = 'block';
  document.getElementById('bulletModal').style.display = 'flex';
}

function saveBullet() {
  const b = { name: document.getElementById('bulletName').value, desc: document.getElementById('bulletDesc').value, emoji: document.getElementById('bulletEmoji').value };
  if(currentEditingId) db.ref('bullets/'+currentUser+'/'+currentEditingId).set(b);
  else db.ref('bullets/'+currentUser).push(b);
  closeBulletModal();
}

function deleteCurrentBullet() {
  if (confirm("Удалить улику?") && currentEditingId) {
    db.ref('bullets/'+currentUser+'/'+currentEditingId).remove().then(() => closeBulletModal());
  }
}

function closeBulletModal() { document.getElementById('bulletModal').style.display = 'none'; }

/* --- ПРЕДЪЯВИТЬ / OBJECTION --- */
function presentToMonokuma() {
  db.ref('monokuma_alert').set({ sender: currentAcc.displayName, title: document.getElementById('bulletName').value, text: document.getElementById('bulletDesc').value });
  closeBulletModal();
}

function startMonokumaListener() {
  db.ref('monokuma_alert').on('value', snap => {
    const d = snap.val();
    if(d) {
      document.getElementById('objSender').innerText = d.sender;
      document.getElementById('objTitle').innerText = d.title;
      document.getElementById('objText').innerText = d.text;
      document.getElementById('objectionOverlay').style.display = 'flex';
    }
  });
}
function closeObjection() {
  document.getElementById('objectionOverlay').style.display = 'none';
  if(currentUser === "монокума") db.ref('monokuma_alert').remove();
}
</script>
</body>
</html>
