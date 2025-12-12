<!DOCTYPE html>
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
    input { width: 100%; padding: 10px; margin-top: 10px; border: none; border-radius: 5px; }
    #loginBtn { margin-top: 15px; width: 100%; padding: 12px; background: #4CAF50; border: none; color: white; border-radius: 5px; cursor: pointer; font-size: 16px; }

    /* Карта */
    #mapWrapper { width: 100%; height: 500px; overflow: hidden; position: relative; }
    #mapArea { width: 1200px; height: 1200px; background: #0f0f0f; border: 2px solid #555; position: absolute; touch-action: pan-x pan-y; }

    /* Маркеры */
    .playerMarker {
      width: 48px;
      height: 48px;
      position: absolute;
      transform: translate(-50%, -50%);
      background-size: cover;
      background-position: center;
      border-radius: 50%;
      border: 2px solid white;
    }

    /* Персонаж */
    #sheetWrapper { width: 100%; overflow: auto; touch-action: pinch-zoom pan-x pan-y; }
    #characterSheet img { width: 100%; max-width: none; }

    @media (max-width: 600px) {
      header { font-size: 18px; padding: 10px; }
      nav button { font-size: 14px; padding: 10px; }
      #logo { font-size: 28px; margin-top: 20px; }
      #loginBox { width: 90%; padding: 15px; }
      input { padding: 8px; font-size: 14px; }
      #loginBtn { padding: 10px; font-size: 14px; }
      #mapWrapper { height: 350px; overflow: scroll; }
      #mapArea { width: 1600px; height: 1600px; }
      #infoContent { flex-direction: column; align-items: center; }
      #leftPanel, #sheetWrapper { margin: 10px 0; width: 90%; }
    }
  </style>
</head>
<body>
<header>Онлайн-игра (демо-версия)</header>
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
  // Конфиг Firebase
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
  const database = firebase.database();

  let currentUser = null;

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

Один из жрецов, по странной жалости, не казнил её, а предложил исправление — направить её через путь служения. Монака согласилась… но только для того, чтобы стать жрецом, изучить силу веры и обратить её в оружие.`, sheet: "https://i.postimg.cc/4x4xjzfV/IMG-20251212-025454-723.jpg" },
    { username: "сония", displayName: "Сония Невермaйнд", password: "1111", color: "lightyellow", avatar: "", description: "", sheet: "" },
    { username: "кируми", displayName: "Кируми Тоджо", password: "1111", color: "darkviolet", avatar: "", description: "", sheet: "" },
    { username: "леон", displayName: "Леон Кувата", password: "1111", color: "orange", avatar: "", description: "", sheet: "" },
    { username: "макото", displayName: "Макото Наэги", password: "1111", color: "lightbrown", avatar: "", description: "", sheet: "" }
  ];

  function openPage(page) {
    document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
    document.getElementById(page).classList.add('active');
    document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
    document.getElementById('tab_' + page).classList.add('active');
  }

  function login() {
    const log = login_input.value.trim();
    const pass = pass_input.value.trim();
    const acc = accounts.find(a => a.username === log && a.password === pass);
    if (!acc) { login_status.style.color = '#f55'; login_status.innerText = "Неверные данные"; return; }

    currentUser = acc.username;
    document.getElementById('logo').innerText = acc.displayName;
    document.getElementById('infoBox').innerText = `Вы вошли как: ${acc.displayName}`;
    document.getElementById('avatarBox').innerHTML = `<img src='${acc.avatar}' alt='Аватар'>`;
    document.getElementById('descriptionBox').innerText = acc.description;
    document.getElementById('characterSheet').innerHTML = `<img src='${acc.sheet}' alt='Лист персонажа'>`;

    login_status.style.color = '#5f5';
    login_status.innerText = "Успешный вход";
    openPage('map');
    spawnPlayerMarkers();
  }

  function spawnPlayerMarkers() {
    const map = document.getElementById('mapArea');
    map.innerHTML = '';
    accounts.forEach(acc => {
      const marker = document.createElement('div');
      marker.className = 'playerMarker';
      marker.style.backgroundImage = `url('${acc.avatar}')`;
      // Читаем координаты из Firebase
      firebase.database().ref('positions/' + acc.username).once('value', snap => {
        const pos = snap.val() || { x: 150, y: 150 };
        marker.style.left = pos.x + 'px';
        marker.style.top = pos.y + 'px';
      });
      map.appendChild(marker);
    });
  }

  const mapArea = document.getElementById('mapArea');
  mapArea.addEventListener('click', e => {
    if (!currentUser) return;
    const rect = mapArea.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    // Сохраняем координаты в Firebase
    firebase.database().ref('positions/' + currentUser).set({ x, y });
    spawnPlayerMarkers();
  });

</script>
</body>
</html>
