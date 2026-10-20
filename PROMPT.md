# Промт: Quiz Breakout — арканоид с образовательным квизом

> Это готовое техническое задание для нейросети. Скопируйте весь файл и отдайте его целиком.
> Документ описывает игру **в точности такой, какая она лежит в этом репозитории**. Если игра меняется, этот файл обновляется вместе с ней (см. `AGENTS.md`).
>
> Файл состоит из двух частей:
> 1. **ТЗ** — что и почему должна делать игра. Важные места и исправленные ошибки отмечены знаком ⚠️.
> 2. **Полные исходники** в конце (раздел «Исходный код») — эталонная рабочая версия.
>
> **Нейросети:** если нужна точная копия игры, создай три файла **один в один** по разделу «Исходный код» и скачай звуки по ссылкам. Если просят изменить игру, бери исходники за основу и соблюдай ТЗ.

---

## Роль и цель

Ты — опытный разработчик образовательных браузерных игр. Напиши игру «Quiz Breakout» — арканоид с образовательным квизом — на **HTML, CSS и чистом JavaScript (Canvas 2D)**, без библиотек, TypeScript, React и сборки. Игру показывают детям на уроке программирования: код простой, с подробными комментариями на русском, игра красивая и работает на компьютере и телефоне.

Что в игре:
- платформа с героем отбивает шар, шар разбивает кирпичи; кирпичи-враги прочные (число ударов на кирпиче);
- разбил золотой кирпич ❓ — пауза и вопрос, верный ответ — выбор 1 из 3 бонусов (14 бонусов: мульти-шар, лазеры, огненный шар…);
- каждый 3-й уровень — босс с полоской здоровья, который ходит и стреляет;
- платформа управляется клавишами, мышью или пальцем, угол отскока зависит от места удара;
- 3 мира: Фэнтези, Космос, Эко-Панк; предметы: математика, русский, английский; ответ кнопками или вводом.

## Лаборатория (раздел 0 в начале `script.js`)

| Поле | По умолчанию | Что делает |
|---|---|---|
| `titleTop` | `'Quiz'` | Название в меню |
| `heroEmoji` | `''` | Свой герой на платформе |
| `lives` | `3` | Жизни |
| `paddleWidth` | `110` | Ширина платформы |
| `ballSpeed` | `1` | Скорость шара |
| `quizBricks` | `3` | Кирпичей с вопросом на уровне |
| `brickRows` | `5` | Рядов кирпичей |
| `bossEvery` | `3` | Босс каждые N уровней |
| `startBalls` | `1` | Шаров на старте |
| `rainbowBall` | `false` | Радужный шар |

### Механика: арканоид
- Высота мира 720, ширина под экран (420–820). Платформа на `PADDLE_Y = H − 70`, ширина `LAB.paddleWidth`, герой-эмодзи стоит под ней. Управление: клавиши (620 пикс/с) или мышь/палец (платформа плавно едет к указателю).
- Шар: радиус 9, скорость Легко 340 / Нормально 410 / Сложно 480, +5% за уровень. ⚠️ Движение шара — 3 подшага за кадр, чтобы не «проскакивал» кирпичи. Отскок от платформы: угол до ~60° в зависимости от места удара. Отскок от кирпича — по оси с меньшим перекрытием.
- Уровень: `brickRows + (уровень−1)/2` рядов (до 9), ~12% кирпичей пропущено. Кирпичи: обычные (1 удар), враги (эмодзи врага мира, прочность = сложность + уровень/3, число на кирпиче, прозрачность от прочности), ❓ (`quizBricks` штук, золотые).
- ⚠️ Разбил ❓ → пауза → вопрос; верно — 3 бонуса из 14 (мульти-шар, широкая платформа, огненный шар насквозь, лазеры, медленный шар, +1 жизнь, липкая платформа, очки ×2, большой шар, щит снизу, взрывной шар, урон ×2, подсказка 50/50, магнит), неверно — мигание и правильный ответ.
- Босс каждые `bossEvery` уровней (меньше рядов, кирпичи ниже): HP 18 + 4·уровень, ходит влево-вправо, стреляет снарядами мира (попадание в платформу — тряска и −20 очков). Полоска здоровья босса в HUD.
- Все шары упали → −1 жизнь, новый шар на платформе (подсказка «Пробел или тап — запустить шар»). Нет кирпичей и босса → следующий уровень (+200·уровень).

## Общее для всех пяти игр (Quiz Shooter, Quiz Tower, Quiz Snake, Quiz Jump, Quiz Breakout)

### Миры (`THEMES`)
| id | Название | Акцент | Враги | Босс |
|---|---|---|---|---|
| fantasy | 🏰 Фэнтези — рыцари и магия | `#FFD700` (тёмно-синий фон) | 👹 💀 | 🐉 Дракон |
| space | 🚀 Космос — лазеры и НЛО | `#00FFFF` (чёрный фон) | ☄️ 🛸 | 👾 Мега-Мозг |
| eco | 🌿 Эко-Панк — защита природы | `#39FF14` (тёмно-зелёный фон) | 🗑️ 🦗 | 🧟 Токсичный Монстр |

Герои миров: Фэнтези 🤺 🧙 🧝‍♂️, Космос 🧑‍🚀 🤖 👽, Эко-Панк 🧑‍🌾 🐼 🦉. Акцент мира пишется в CSS-переменную `--accent`.

### Квиз (раздел «Вопросы» одинаковый во всех играх)
- Предметы (`SUBJECTS`): 🧮 Математика (пул из 20 случайных примеров: сложение, вычитание без отрицательных ответов, умножение; числа растут со сложностью), 📝 Русский (28 словарных слов с пропущенной гласной: `М_ЛОКО`), 🇬🇧 Английский (26 слов для перевода: `SPACE → Космос`).
- Задания перемешиваются и идут по очереди (`questionPool`), пул пересобирается, когда кончился. 4 варианта: верный + 3 неверных (близкие числа / другие гласные / другие переводы).
- Способ ответа (`INPUT_MODES`): **Кнопки** — 4 большие кнопки (клавиши `1`–`4`); **Ввод** — поле `<input>` + Enter, ⚠️ автофокус при открытии, `inputMode = numeric` для математики. Сравнение `normalize()`: без пробелов, без регистра, Ё = Е.
- ⚠️ Пока открыт вопрос — **жёсткая пауза** (мир не двигается). Неверно — окно мигает красным, подсвечивается/пишется правильный ответ (1.1 с). Верно — зелёная вспышка (0.5 с) и награда своей игры.
- Улучшения (перки) — рогалик: 3 случайных карточки из пула, каждое можно взять один раз за игру.

### Экраны и интерфейс
- HTML-слои поверх `<canvas>`: меню (название из `LAB`, подпись «мир · предмет · способ ответа», рекорд, «Начать», «Настройки», подсказки по управлению), выбор сложности (3 карточки), вопрос, улучшения, настройки (мир, предмет, способ ответа — карточки; звук + громкость; сброс рекорда), пауза, GAME OVER (очки, рекорд, «Рестарт», «В меню»), отсчёт 3-2-1 (кроме Quiz Tower), всплывающие надписи.
- ⚠️ Верхняя панель — сетка `grid-template-columns: 1fr auto 1fr` (у змейки `1fr 1fr`, вопрос — плашкой по центру): центральный блок **точно по центру**, все блоки и кнопка паузы одной высоты и на одной линии. Через `space-between` центр уезжал (до 46 px), высоты были разные. Экраны — `position: fixed` на всё окно; всплывающие надписи переносятся (`max-width: 92%`).
- Стекло-панели `rgba(12,16,40,0.84)` + `backdrop-filter: blur`, объёмные кнопки (янтарная и бирюзовая), шрифты Russo One + Rubik через `@import` в CSS, адаптив `@media`.
- `Esc`/`P` — пауза, автопауза при скрытии вкладки, после показа экрана `blur()` активной кнопки. Пока печатают ответ — клавиши игры не работают.
- Сохранение в `localStorage` (`<игра>-settings`, `<игра>-best`) в `try/catch`.

### Звук
⚠️ Только Web Audio API (`fetch → decodeAudioData`, `createBufferSource → createGain`), `SOUND_START = { coin: 0.045 }`. Файлы в `sound/`: `jump.wav`, `coin.mp3`, `tick.wav`, `start.wav`, `crash.wav`. В CodePen (`codepen`/`cdpn.io` в адресе) звуки берутся с GitHub Pages игры.

### Файлы и CodePen
- `index.html` / `style.css` / `script.js` — строго раздельно, чистый JavaScript + Canvas 2D, графика — эмодзи и фигуры, без библиотек и сборки. Подключение с `?v=N`.
- ⚠️ `codepen/index.pen.html` — часть `index.html` между метками «Для CodePen», без `<head>`, `<link>`, `<script>`: CodePen сам подключает CSS и JS, иначе скрипт загрузится дважды (`Identifier ... has already been declared`).
- Код для детей: объекты и функции (без классов), подробные русские комментарии, нумерованные разделы, раздел 0 — «Лаборатория» `LAB` из 10 настроек (значения по умолчанию = обычная игра).

## Чек-лист

- [ ] Три отдельных файла, чистый JS без библиотек, комментарии на русском, `LAB` в начале.
- [ ] Нет ошибок в консоли; движение через `dt` (секунды, не больше 0.05).
- [ ] Вопрос ставит игру на жёсткую паузу; верный ответ даёт награду, неверный показывает правильный ответ.
- [ ] Работают оба способа ответа (кнопки и ввод с автофокусом) и все три предмета и мира.
- [ ] Настройки и рекорд сохраняются; `?v=` у CSS и JS.
- [ ] В CodePen работают `codepen/index.pen.html` + `style.css` + `script.js`, звуки грузятся с GitHub Pages.

---

## Звуковые файлы

Положи в папку `sound/` рядом с `index.html` (или скачай из репозитория https://github.com/si0683/QuizBreakout):

| Файл | Скачать |
|---|---|
| `jump.wav` | https://si0683.github.io/QuizBreakout/sound/jump.wav |
| `coin.mp3` | https://si0683.github.io/QuizBreakout/sound/coin.mp3 |
| `tick.wav` | https://si0683.github.io/QuizBreakout/sound/tick.wav |
| `start.wav` | https://si0683.github.io/QuizBreakout/sound/start.wav |
| `crash.wav` | https://si0683.github.io/QuizBreakout/sound/crash.wav |

## Исходный код

<!-- SOURCES:START -->

<!-- Этот раздел создаётся скриптом tools/build.py — не редактируйте вручную -->

### `index.html`

````html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
  <title>Quiz Breakout</title>
  <!-- Подключаем файл с оформлением.
       ?v=2 — номер версии: меняем его при каждом обновлении игры,
       чтобы браузер не брал старый файл из кэша -->
  <link rel="stylesheet" href="style.css?v=2">
</head>
<body>

<!-- ↓↓↓ Для CodePen копируйте в окно HTML всё, что ниже этой строки ↓↓↓
     Только эту часть! CodePen сам подключает style.css и script.js.
     Если скопировать строки <link> или <script src>, игра загрузится дважды
     и появится ошибка «Identifier ... has already been declared». -->

<!-- «Сцена» — рамка игры. Внутри холст (canvas) и все экраны поверх него -->
<div class="stage" id="stage">

  <!-- Холст: здесь JavaScript рисует кирпичи, шар и платформу -->
  <canvas id="gameCanvas"></canvas>

  <!-- Панель во время игры -->
  <div class="hud hidden" id="hud">
    <div class="hud-top">
      <div class="hud-left">
        <div class="hud-box">
          <span class="hud-label">Жизни</span>
          <b class="hearts" id="lives">❤️❤️❤️</b>
        </div>
      </div>
      <div class="hud-box hud-center">
        <span class="hud-label">Очки</span>
        <b id="score">0</b>
      </div>
      <div class="hud-right">
        <div class="hud-box">
          <span class="hud-label">Уровень</span>
          <b id="level">1</b>
        </div>
        <button class="icon-btn" id="pauseButton" type="button" aria-label="Пауза">
          <svg viewBox="0 0 24 24" width="22" height="22"><rect x="6" y="5" width="4" height="14" rx="1.5" fill="currentColor"/><rect x="14" y="5" width="4" height="14" rx="1.5" fill="currentColor"/></svg>
        </button>
      </div>
    </div>
    <!-- Улучшения, которые уже взяли -->
    <div class="weapon" id="perkChips"></div>
    <!-- Подсказка: как запустить шар -->
    <div class="launch-hint hidden" id="launchHint">Пробел или тап — запустить шар</div>
    <!-- Полоска здоровья босса -->
    <div class="boss-bar hidden" id="bossBar">
      <span id="bossName">Босс</span>
      <div class="bar"><div class="bar-fill" id="bossFill"></div></div>
    </div>
  </div>

  <!-- Обратный отсчёт 3 - 2 - 1 -->
  <div class="countdown hidden" id="countdown"></div>

  <!-- Всплывающая надпись -->
  <div class="toast" id="toast"></div>

  <!-- ЭКРАН 1. Стартовое меню -->
  <section class="screen" id="menuScreen">
    <div class="card">
      <h1 class="title">
        <span class="title-top">Quiz</span>
        <span class="title-bottom">Breakout</span>
      </h1>
      <p class="subtitle" id="menuInfo">Разбивай кирпичи, отвечай на вопросы, получай бонусы!</p>
      <div class="best">🏆 Рекорд: <b id="menuBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="startButton" type="button">Начать</button>
        <button class="btn btn-secondary" id="settingsButton" type="button">Настройки</button>
      </div>
      <div class="howto">
        <div><kbd>←</kbd> <kbd>→</kbd>, <kbd>A</kbd> <kbd>D</kbd> или мышь — платформа</div>
        <div><kbd>Пробел</kbd> или тап — запустить шар</div>
        <div>❓ Разбей кирпич с вопросом — верный ответ даст бонус</div>
        <div><kbd>Esc</kbd> — пауза</div>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 3. Выбор сложности -->
  <section class="screen hidden" id="difficultyScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Выбери сложность</h2>
      <div class="choice-grid" id="difficultyChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 4. Вопрос (квиз) -->
  <section class="screen hidden" id="quizScreen">
    <div class="card quiz-card" id="quizCard">
      <div class="quiz-subject" id="quizSubject">🧮 Математика</div>
      <div class="quiz-question" id="quizQuestion">8 × 7 = ?</div>
      <div class="quiz-options" id="quizOptions"></div>
      <form class="quiz-form hidden" id="quizForm" autocomplete="off">
        <input class="quiz-input" id="quizInput" type="text" placeholder="Ответ" autocomplete="off" autocapitalize="characters" spellcheck="false">
        <button class="btn btn-small" type="submit">Ответить ⏎</button>
      </form>
      <div class="quiz-hint" id="quizHint">Верный ответ — бонус!</div>
    </div>
  </section>

  <!-- ЭКРАН 5. Улучшения (рогалик) -->
  <section class="screen hidden" id="perkScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Верно! Выбери бонус</h2>
      <div class="choice-grid" id="perkChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 6. Настройки -->
  <section class="screen screen-settings hidden" id="settingsScreen">
    <div class="settings-panel">
      <div class="settings-header">
        <h2>Настройки</h2>
        <button class="icon-btn" id="settingsClose" type="button" aria-label="Закрыть">✕</button>
      </div>

      <div class="setting">
        <h3>Мир</h3>
        <div class="options options-cards" id="themeOptions"></div>
      </div>

      <div class="setting">
        <h3>Предмет</h3>
        <div class="options options-cards" id="subjectOptions"></div>
      </div>

      <div class="setting">
        <h3>Как отвечать</h3>
        <div class="options options-cards" id="inputOptions"></div>
      </div>

      <div class="setting">
        <h3>Звук</h3>
        <div class="row">
          <label class="switch">
            <input type="checkbox" id="soundToggle">
            <span class="switch-slider"></span>
          </label>
          <input class="volume" type="range" id="volumeRange" min="0" max="100" step="5" aria-label="Громкость">
        </div>
      </div>

      <div class="settings-footer">
        <button class="btn btn-ghost btn-small" id="resetBestButton" type="button">Сбросить рекорд</button>
        <button class="btn btn-small" id="settingsDone" type="button">Готово</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 7. Пауза -->
  <section class="screen hidden" id="pauseScreen">
    <div class="card">
      <h2 class="screen-title">Пауза</h2>
      <div class="buttons">
        <button class="btn" id="resumeButton" type="button">Продолжить</button>
        <button class="btn btn-secondary" id="pauseMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 8. Игра окончена -->
  <section class="screen hidden" id="gameOverScreen">
    <div class="card">
      <h2 class="over-title">GAME OVER</h2>
      <div class="new-record hidden" id="newRecord">Новый рекорд!</div>
      <div class="stats">
        <div class="stat"><span>Очки</span><b id="finalScore">0</b></div>
        <div class="stat"><span>Уровни</span><b id="finalLevels">0</b></div>
        <div class="stat"><span>Ответы</span><b id="finalAnswers">0</b></div>
      </div>
      <div class="best">🏆 Рекорд: <b id="finalBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="restartButton" type="button">Рестарт</button>
        <button class="btn btn-secondary" id="overMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

</div>

<!-- ↑↑↑ Для CodePen копируйте до этой строки (строку <script> ниже — НЕ копировать) ↑↑↑ -->

<!-- Подключаем файл с логикой игры (номер версии — как у style.css) -->
<script src="script.js?v=2"></script>
</body>
</html>
````

### `style.css`

````css
/* =========================================================
   QUIZ BREAKOUT (АРКАНОИД) — оформление (цвета, шрифты, экраны, кнопки)
   ========================================================= */

/* Шрифты Google Fonts: Russo One — заголовки и цифры, Rubik — текст */
@import url('https://fonts.googleapis.com/css2?family=Russo+One&family=Rubik:wght@400;500;700&display=swap');

/* ---------- Цвета игры (переменные) ---------- */
/* --accent JavaScript меняет под выбранный мир */
:root {
  --bg: #070b1f;
  --panel: rgba(12, 16, 40, 0.84);
  --line: rgba(255, 255, 255, 0.14);
  --text: #eef1ff;
  --dim: #a7b0dd;
  --accent: #00ffff;
  --amber: #ffb84d;
  --amber-light: #ffd27a;
  --amber-dark: #d9821c;
  --cyan: #4fd6e8;
  --cyan-dark: #1f8a99;
  --good: #39d97a;
  --bad: #ff4d6d;
  --outline: #0d1030;
}

* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  height: 100%;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background:
    radial-gradient(900px 600px at 20% -10%, #1c2560 0%, transparent 60%),
    radial-gradient(800px 600px at 100% 100%, #2a1450 0%, transparent 55%),
    var(--bg);
  color: var(--text);
  font-family: 'Rubik', sans-serif;
  user-select: none;
  -webkit-user-select: none;
  -webkit-tap-highlight-color: transparent;
}

/* ---------- Сцена и холст ---------- */
/* Размер сцены задаёт JavaScript (функция resizeGame) */
.stage {
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 80px rgba(0, 0, 0, 0.6);
  touch-action: none; /* чтобы касание не прокручивало страницу */
}

#gameCanvas {
  display: block;
  width: 100%;
  height: 100%;
}

.hidden {
  display: none !important;
}

/* ---------- Панель во время игры (HUD) ---------- */
.hud {
  position: absolute;
  inset: 0;
  padding: 12px;
  pointer-events: none; /* касания проходят сквозь панель */
}

.hud-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 8px;
}

.hud-box {
  display: flex;
  flex-direction: column;
  padding: 6px 12px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  backdrop-filter: blur(8px);
}

.hud-label {
  color: var(--dim);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  white-space: nowrap;
}

.hud-box b {
  font-family: 'Russo One', sans-serif;
  font-size: 22px;
  font-weight: 400;
  line-height: 1.15;
}

.hud-hp {
  min-width: 140px;
}

.hud-hp b {
  font-size: 14px;
}

.hud-center {
  align-items: center;
}

.hud-center b {
  color: var(--accent);
  text-shadow: 0 0 12px var(--accent);
}

.hud-right {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.bar {
  height: 10px;
  margin: 5px 0 3px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.12);
  overflow: hidden;
}

.bar-fill {
  width: 100%;
  height: 100%;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--bad), #ff9f43, var(--good));
  background-size: 300% 100%;
  transition: width 0.25s;
}

.weapon {
  position: absolute;
  left: 12px;
  bottom: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  max-width: calc(100% - 24px);
}

.chip {
  padding: 4px 10px;
  border: 1px solid var(--line);
  border-radius: 999px;
  background: var(--panel);
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
}

.boss-bar {
  position: absolute;
  top: 78px;
  left: 50%;
  width: min(420px, 80%);
  transform: translateX(-50%);
  text-align: center;
  font-family: 'Russo One', sans-serif;
  font-size: 14px;
  color: var(--bad);
  text-shadow: 0 0 10px var(--bad);
}

.boss-bar .bar {
  height: 12px;
  border: 1px solid rgba(255, 77, 109, 0.6);
}

.boss-bar .bar-fill {
  background: linear-gradient(90deg, #ff2a5c, #ff7a45);
}

.icon-btn {
  pointer-events: auto;
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  backdrop-filter: blur(8px);
}

/* ---------- Экраны ---------- */
.screen {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 14px;
  background: radial-gradient(circle at 50% 45%, rgba(7, 11, 31, 0.3), rgba(7, 11, 31, 0.75));
  animation: fadeIn 0.2s ease;
}

.card {
  width: min(400px, 100%);
  max-height: 100%;
  overflow-y: auto;
  padding: 24px 22px;
  border: 1px solid var(--line);
  border-radius: 26px;
  background: var(--panel);
  box-shadow: 0 24px 60px -20px rgba(0, 0, 0, 0.7), 0 0 0 1px rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px);
  text-align: center;
  animation: rise 0.3s cubic-bezier(0.2, 0.9, 0.3, 1.2);
}

.card-wide {
  width: min(720px, 100%);
}

/* Заголовок игры с неоновым свечением в цвет мира */
.title {
  margin: 0 0 6px;
  font-family: 'Russo One', sans-serif;
  font-weight: 400;
  line-height: 0.95;
}

.title span {
  display: block;
  -webkit-text-stroke: 5px var(--outline);
  paint-order: stroke fill;
}

.title-top {
  font-size: 52px;
  color: #fff;
  text-shadow: 0 0 16px var(--accent), 0 6px 0 var(--outline);
}

.title-bottom {
  font-size: 58px;
  color: var(--accent);
  text-shadow: 0 0 24px var(--accent), 0 6px 0 var(--outline);
}

.subtitle {
  margin: 12px 0 14px;
  color: var(--dim);
  font-size: 15px;
}

.best {
  display: inline-block;
  margin-bottom: 18px;
  padding: 6px 14px;
  border-radius: 999px;
  background: rgba(255, 184, 77, 0.12);
  color: var(--amber);
  font-weight: 500;
}

.screen-title {
  margin: 0 0 16px;
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.howto {
  display: grid;
  gap: 8px;
  margin-top: 20px;
  color: var(--dim);
  font-size: 13px;
}

kbd {
  padding: 2px 7px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-bottom: 2px solid rgba(255, 255, 255, 0.22);
  color: var(--text);
  font-family: inherit;
  font-size: 12px;
}

/* ---------- Кнопки (объёмные, «нажимаются» вниз) ---------- */
.btn {
  padding: 14px 28px;
  border: 0;
  border-radius: 16px;
  background: linear-gradient(180deg, var(--amber-light), var(--amber));
  box-shadow: 0 6px 0 var(--amber-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #3b2300;
  font-family: 'Russo One', sans-serif;
  font-size: 20px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s, filter 0.15s;
}

.btn:hover {
  filter: brightness(1.07);
}

.btn:active {
  transform: translateY(4px);
  box-shadow: 0 2px 0 var(--amber-dark);
}

.btn-secondary {
  background: linear-gradient(180deg, #7ae9f6, var(--cyan));
  box-shadow: 0 6px 0 var(--cyan-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #05303a;
}

.btn-secondary:active {
  box-shadow: 0 2px 0 var(--cyan-dark);
}

.btn-ghost {
  background: rgba(255, 255, 255, 0.08);
  box-shadow: none;
  color: var(--dim);
}

.btn-ghost:active {
  box-shadow: none;
}

.btn-small {
  padding: 11px 18px;
  font-size: 15px;
}

/* ---------- Карточки выбора: герои, сложность, улучшения ---------- */
.choice-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 12px;
}

.choice {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 16px 12px;
  border: 2px solid var(--line);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--text);
  font-family: inherit;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, box-shadow 0.15s;
}

.choice:hover,
.choice:focus-visible {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: 0 0 22px -4px var(--accent);
  outline: none;
}

.choice-icon {
  font-size: 46px;
  line-height: 1.1;
}

.choice-title {
  font-family: 'Russo One', sans-serif;
  font-size: 18px;
}

.choice-desc {
  color: var(--dim);
  font-size: 13px;
  line-height: 1.35;
}

.choice-stats {
  display: grid;
  gap: 3px;
  width: 100%;
  margin-top: 4px;
  font-size: 12px;
  text-align: left;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.stat-row span {
  width: 64px;
  color: var(--dim);
}

.stat-dots {
  letter-spacing: 1px;
  color: var(--accent);
}

/* ---------- Окно вопроса ---------- */
.quiz-card {
  width: min(460px, 100%);
  border-color: var(--accent);
  box-shadow: 0 0 40px -10px var(--accent), 0 24px 60px -20px rgba(0, 0, 0, 0.7);
}

.quiz-subject {
  color: var(--dim);
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.quiz-question {
  margin: 14px 0 18px;
  font-family: 'Russo One', sans-serif;
  font-size: 40px;
  letter-spacing: 0.04em;
  word-break: break-word;
}

.quiz-question small {
  display: block;
  margin-top: 6px;
  color: var(--dim);
  font-family: 'Rubik', sans-serif;
  font-size: 15px;
  letter-spacing: 0;
}

.quiz-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.quiz-option {
  min-height: 64px;
  padding: 12px;
  border: 2px solid var(--line);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.07);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  cursor: pointer;
  transition: transform 0.12s, border-color 0.12s, background 0.12s;
}

.quiz-option:hover {
  transform: translateY(-3px);
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.12);
}

.quiz-option.right {
  border-color: var(--good);
  background: rgba(57, 217, 122, 0.3);
}

.quiz-option.wrong {
  border-color: var(--bad);
  background: rgba(255, 77, 109, 0.3);
}

.quiz-form {
  display: flex;
  gap: 10px;
}

.quiz-input {
  flex: 1;
  min-width: 0;
  padding: 14px 16px;
  border: 2px solid var(--accent);
  border-radius: 16px;
  outline: none;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  text-align: center;
  text-transform: uppercase;
}

.quiz-hint {
  margin-top: 14px;
  color: var(--dim);
  font-size: 13px;
}

/* Окно мигает красным при неверном ответе и зелёным при верном */
.quiz-card.flash-bad {
  animation: flashBad 0.6s ease;
}

.quiz-card.flash-good {
  animation: flashGood 0.5s ease;
}

/* ---------- Экран настроек ---------- */
.screen-settings {
  justify-content: flex-end;
  align-items: stretch;
  background: linear-gradient(90deg, transparent 30%, rgba(7, 11, 31, 0.6));
}

.settings-panel {
  width: min(420px, 100%);
  max-height: 100%;
  overflow-x: hidden;
  overflow-y: auto;
  padding: 18px 20px 20px;
  border: 1px solid var(--line);
  border-radius: 24px;
  background: var(--panel);
  backdrop-filter: blur(10px);
  animation: slideIn 0.3s ease;
}

.settings-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.settings-header h2 {
  margin: 0;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  font-weight: 400;
}

.setting {
  padding: 12px 0;
  border-bottom: 1px solid var(--line);
}

.setting h3 {
  margin: 0 0 10px;
  color: var(--dim);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.options-cards .option {
  flex: 1;
  min-width: 100px;
}

.option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 8px;
  border: 2px solid transparent;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--dim);
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, background 0.15s;
}

.option:hover {
  transform: translateY(-2px);
}

.option.selected {
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.option-icon {
  font-size: 26px;
}

.option-title {
  color: #fff;
  font-size: 14px;
  font-weight: 700;
}

.settings-footer {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding-top: 16px;
}

/* Переключатель (вкл/выкл) */
.switch {
  position: relative;
  flex-shrink: 0;
  width: 52px;
  height: 30px;
  cursor: pointer;
}

.switch input {
  position: absolute;
  opacity: 0;
}

.switch-slider {
  position: absolute;
  inset: 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.15);
  transition: background 0.2s;
}

.switch-slider::before {
  content: '';
  position: absolute;
  top: 4px;
  left: 4px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #fff;
  transition: transform 0.2s;
}

.switch input:checked + .switch-slider {
  background: var(--amber);
}

.switch input:checked + .switch-slider::before {
  transform: translateX(22px);
}

.volume {
  flex: 1;
  accent-color: var(--amber);
}

.volume:disabled {
  opacity: 0.35;
}

/* ---------- Экран «Игра окончена» ---------- */
.over-title {
  margin: 0 0 14px;
  font-family: 'Russo One', sans-serif;
  font-size: 46px;
  font-weight: 400;
  color: var(--bad);
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 24px var(--bad);
}

.stats {
  display: flex;
  gap: 10px;
  margin-bottom: 14px;
}

.stat {
  flex: 1;
  padding: 10px 6px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.07);
}

.stat span {
  display: block;
  color: var(--dim);
  font-size: 12px;
}

.stat b {
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.new-record {
  display: inline-block;
  margin-bottom: 14px;
  padding: 6px 16px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--accent), var(--amber));
  color: #10132e;
  font-weight: 700;
  animation: pulse 1s ease-in-out infinite;
}

/* ---------- Обратный отсчёт и всплывающая надпись ---------- */
.countdown {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  pointer-events: none;
  font-family: 'Russo One', sans-serif;
  font-size: 150px;
  color: #fff;
  -webkit-text-stroke: 6px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 30px var(--accent), 0 8px 0 var(--outline);
}

.countdown span {
  animation: countPop 0.7s ease-out both;
}

.countdown .go {
  font-size: 90px;
  color: var(--accent);
}

.toast {
  position: absolute;
  top: 26%;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  pointer-events: none;
  white-space: nowrap;
  font-family: 'Russo One', sans-serif;
  font-size: 30px;
  color: #fff;
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 18px var(--accent);
}

.toast.show {
  animation: toast 1.6s ease both;
}

/* ---------- Анимации ---------- */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes rise {
  from { opacity: 0; transform: translateY(24px) scale(0.96); }
  to { opacity: 1; transform: none; }
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(40px); }
  to { opacity: 1; transform: none; }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.07); }
}

@keyframes flashBad {
  0%, 100% { background: var(--panel); }
  20%, 60% { background: rgba(255, 77, 109, 0.75); }
  40%, 80% { background: rgba(255, 77, 109, 0.25); }
}

@keyframes flashGood {
  0%, 100% { background: var(--panel); }
  40% { background: rgba(57, 217, 122, 0.6); }
}

@keyframes countPop {
  0% { opacity: 0; transform: scale(2.2); }
  30% { opacity: 1; transform: scale(1); }
  80% { opacity: 1; transform: scale(0.95); }
  100% { opacity: 0; transform: scale(0.7); }
}

@keyframes toast {
  0% { opacity: 0; transform: translate(-50%, 20px) scale(0.8); }
  15% { opacity: 1; transform: translate(-50%, 0) scale(1.05); }
  25% { transform: translate(-50%, 0) scale(1); }
  80% { opacity: 1; }
  100% { opacity: 0; transform: translate(-50%, -20px); }
}

/* ---------- Телефоны ---------- */
@media (max-width: 640px) {
  .screen-settings {
    align-items: flex-end;
    padding: 10px;
  }

  .settings-panel {
    max-height: 80%;
  }

  .hud-hp {
    min-width: 110px;
  }

  .hud-box b {
    font-size: 18px;
  }

  .title-top { font-size: 42px; }
  .title-bottom { font-size: 46px; }
  .quiz-question { font-size: 32px; }
  .quiz-option { font-size: 20px; }
  .choice-grid { grid-template-columns: 1fr; }
  .choice { flex-direction: row; text-align: left; }
  .choice-icon { font-size: 38px; }
  .countdown { font-size: 120px; }
}

/* Если в системе включено «уменьшить движение» */
@media (prefers-reduced-motion: reduce) {
  .new-record {
    animation: none;
  }
}

/* =========================================================
   ДОПОЛНЕНИЯ ДЛЯ АРКАНОИДА
   ========================================================= */

.hud-left {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.hearts {
  font-size: 20px;
  letter-spacing: 2px;
}

.launch-hint {
  position: absolute;
  bottom: 120px;
  left: 50%;
  transform: translateX(-50%);
  padding: 8px 16px;
  border-radius: 999px;
  background: var(--panel);
  color: #fff;
  font-weight: 700;
  white-space: nowrap;
  animation: blink 1.1s ease-in-out infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}

@media (max-width: 640px) {
  .hud-box {
    padding: 3px 8px;
  }

  .hud-box b,
  .hearts {
    font-size: 15px;
  }

  .boss-bar {
    top: 58px;
  }
}

/* =========================================================
   РОВНАЯ ВЕРХНЯЯ ПАНЕЛЬ И ЭКРАНЫ
   ========================================================= */

/* Сетка: левая и правая колонки одинаковой ширины (1fr),
   поэтому средний блок стоит точно по центру */
.hud-top {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: start;
}

.hud-top > :first-child {
  justify-self: start;
}

.hud-top > :last-child {
  justify-self: end;
}

.hud-left,
.hud-right {
  display: flex;
  gap: 8px;
  align-items: stretch;
}

/* Все блоки одной высоты, содержимое по центру */
.hud-top .hud-box {
  justify-content: center;
  min-width: 80px;
  height: 54px;
  text-align: center;
}

.hud-top .hud-label {
  line-height: 16px;
}

.hud-top .icon-btn {
  width: 54px;
  height: 54px;
}

/* Экраны (меню, вопрос, улучшения, настройки…) — на всё окно,
   даже если игровое поле на телефоне узкой полосой */
.screen {
  position: fixed;
}

/* Длинные подсказки переносятся, а не обрезаются краем экрана */
.toast {
  width: max-content;
  max-width: 92%;
  white-space: normal;
  text-align: center;
  line-height: 1.15;
}

@media (max-width: 700px) {
  .hud-top .hud-box {
    min-width: 0;
    height: 40px;
    padding: 2px 8px;
  }

  .hud-top .hud-label {
    line-height: 12px;
  }

  .hud-top .icon-btn {
    width: 40px;
    height: 40px;
  }

  .hud-left,
  .hud-right {
    gap: 4px;
  }
}
````

### `script.js`

````javascript
/* =========================================================
   QUIZ BREAKOUT — арканоид + образовательный квиз + бонусы
   Чистый JavaScript и Canvas, графика из эмодзи
   ---------------------------------------------------------
   Как устроен файл (читайте сверху вниз):
     0. Лаборатория: 10 настроек для экспериментов
     1. Данные: миры, предметы, сложность
     2. Вопросы: математика, русский, английский
     3. Бонусы (перки)
     4. Настройки игрока и рекорд
     5. Звуки
     6. Холст и размер игры
     7. Состояние игры, платформа, шары, кирпичи
     8. Строим уровень
     9. Рисование: фон, кирпичи, шары, частицы
    10. Окно вопроса (квиз)
    11. Экран бонусов
    12. Игровые действия
    13. Обновление мира за один кадр
    14. Рисование одного кадра
    15. Экраны и интерфейс
    16. Экран настроек
    17. Управление
    18. Игровой цикл и запуск
   ========================================================= */


/* ---------------------------------------------------------
   0. ЛАБОРАТОРИЯ ИГРЫ 🧪
   Поменяй одно значение, и игра изменится!
   Сейчас здесь обычные значения: игра такая, как задумано.
   --------------------------------------------------------- */

const LAB = {
  // 1. Название игры в меню: верхнее и нижнее слово (до 8 букв)
  titleTop: 'Quiz',
  titleBottom: 'Breakout',

  // 2. Свой герой-эмодзи на платформе, например '🐸'. Пусто '' — герой мира
  heroEmoji: '',

  // 3. Жизни: 3 — обычно, 10 — много
  lives: 3,

  // 4. Ширина платформы: 110 — обычная, 200 — широкая
  paddleWidth: 110,

  // 5. Скорость шара: 1 — обычная, 0.7 — медленный, 1.4 — быстрый
  ballSpeed: 1,

  // 6. Кирпичей с вопросом на уровне: 3 — обычно, 8 — много вопросов
  quizBricks: 3,

  // 7. Рядов кирпичей на первом уровне: 5 — обычно, 8 — стена
  brickRows: 5,

  // 8. Босс каждые N уровней: 3 — обычно, 1 — всегда
  bossEvery: 3,

  // 9. Шаров на старте: 1 — обычно, 3 — сразу мульти-шар
  startBalls: 1,

  // 10. Радужный шар: false — цвет мира, true — радуга
  rainbowBall: false
};


/* ---------------------------------------------------------
   1. ДАННЫЕ: МИРЫ, ПРЕДМЕТЫ, СЛОЖНОСТЬ
   --------------------------------------------------------- */

const THEMES = [
  {
    id: 'fantasy', icon: '🏰', label: 'Фэнтези', desc: 'Рыцари и магия',
    accent: '#FFD700',
    sky: ['#0b1030', '#1b1f5a', '#3a2a6a'],
    bricks: ['#ffd700', '#ff9f43', '#c084fc', '#60a5fa', '#34d399'],
    hero: '🧙', enemies: ['👹', '💀'], boss: { emoji: '🐉', name: 'Дракон', shot: '🔥' }
  },
  {
    id: 'space', icon: '🚀', label: 'Космос', desc: 'Лазеры и НЛО',
    accent: '#00FFFF',
    sky: ['#000008', '#05051a', '#0d0a2e'],
    bricks: ['#00ffff', '#3b82f6', '#a855f7', '#ec4899', '#22d3ee'],
    hero: '🧑‍🚀', enemies: ['☄️', '🛸'], boss: { emoji: '👾', name: 'Мега-Мозг', shot: '⚡' }
  },
  {
    id: 'eco', icon: '🌿', label: 'Эко-Панк', desc: 'Защита природы',
    accent: '#39FF14',
    sky: ['#04140c', '#0b2a18', '#123d22'],
    bricks: ['#39ff14', '#a3e635', '#facc15', '#2dd4bf', '#86efac'],
    hero: '🐼', enemies: ['🗑️', '🦗'], boss: { emoji: '🧟', name: 'Токсичный Монстр', shot: '☢️' }
  }
];

const SUBJECTS = [
  { id: 'math', icon: '🧮', label: 'Математика', desc: 'Примеры' },
  { id: 'rus',  icon: '📝', label: 'Русский', desc: 'Пропущенная буква' },
  { id: 'eng',  icon: '🇬🇧', label: 'Английский', desc: 'Перевод слов' }
];

const INPUT_MODES = [
  { id: 'buttons',  icon: '👆', label: 'Кнопки', desc: '4 варианта' },
  { id: 'keyboard', icon: '⌨️', label: 'Ввод', desc: 'Напечатать ответ' }
];

// Сложность: скорость шара, прочность кирпичей-врагов, множитель очков
const DIFFICULTIES = [
  { id: 'easy',   icon: '🙂', label: 'Легко',     desc: 'Спокойный шар',            speed: 340, enemyHp: 2, score: 1 },
  { id: 'normal', icon: '😎', label: 'Нормально', desc: 'Быстрее, очки ×1.5',        speed: 410, enemyHp: 3, score: 1.5 },
  { id: 'hard',   icon: '😈', label: 'Сложно',    desc: 'Очень быстро, очки ×2',     speed: 480, enemyHp: 4, score: 2 }
];

function findById(list, id) {
  return list.find(item => item.id === id) || list[0];
}

function currentTheme() {
  return findById(THEMES, settings.theme);
}

function difficulty() {
  return findById(DIFFICULTIES, settings.difficulty);
}


/* ---------------------------------------------------------
   2. ВОПРОСЫ
   Для каждого предмета — пул заданий. Задания перемешиваются
   и идут по очереди, поэтому подряд не повторяются.
   --------------------------------------------------------- */

// Русский язык: слово, номер пропущенной буквы (с нуля)
const RUS_WORDS = [
  ['МОЛОКО', 1], ['СОБАКА', 1], ['КОРОВА', 1], ['ВОРОНА', 1], ['БЕРЁЗА', 1],
  ['ЗАЯЦ', 1], ['ПЕТУХ', 1], ['ЛИСИЦА', 1], ['МЕДВЕДЬ', 1], ['КАРАНДАШ', 1],
  ['ТЕТРАДЬ', 1], ['ПЕНАЛ', 1], ['ЯГОДА', 3], ['МАГАЗИН', 1], ['ГОРОД', 1],
  ['УЧЕНИК', 2], ['ДЕЖУРНЫЙ', 1], ['РЕБЯТА', 1], ['КАПУСТА', 1], ['ПОГОДА', 1],
  ['СОРОКА', 1], ['ВЕТЕР', 1], ['МОРКОВЬ', 1], ['ПАЛЬТО', 1], ['ОГУРЕЦ', 2],
  ['ХОРОШО', 1], ['ДЕВОЧКА', 1], ['ЯЗЫК', 1]
];
const VOWELS = ['А', 'О', 'Е', 'И', 'Я', 'У'];

// Английский: слово и перевод
const ENG_WORDS = [
  ['APPLE', 'Яблоко'], ['DOG', 'Собака'], ['CAT', 'Кошка'], ['SPACE', 'Космос'],
  ['STAR', 'Звезда'], ['SUN', 'Солнце'], ['MOON', 'Луна'], ['HOUSE', 'Дом'],
  ['BOOK', 'Книга'], ['WATER', 'Вода'], ['TREE', 'Дерево'], ['BIRD', 'Птица'],
  ['FISH', 'Рыба'], ['SCHOOL', 'Школа'], ['FRIEND', 'Друг'], ['MOTHER', 'Мама'],
  ['FATHER', 'Папа'], ['GREEN', 'Зелёный'], ['RED', 'Красный'], ['BIG', 'Большой'],
  ['MILK', 'Молоко'], ['BREAD', 'Хлеб'], ['SKY', 'Небо'], ['EARTH', 'Земля'],
  ['FLOWER', 'Цветок'], ['CAR', 'Машина']
];

let questionPool = [];

function shuffle(list) {
  const copy = list.slice();
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}

function randInt(min, max) {
  return Math.floor(min + Math.random() * (max - min + 1));
}

// Один пример по математике. Чем сложнее игра — тем больше числа
function makeMathQuestion(level) {
  const kind = randInt(0, 2);
  let a, b, text, answer;
  if (kind === 0) {
    a = randInt(2, 10 + level * 20);
    b = randInt(2, 10 + level * 20);
    text = a + ' + ' + b + ' = ?';
    answer = a + b;
  } else if (kind === 1) {
    a = randInt(5, 15 + level * 20);
    b = randInt(1, a);           // без отрицательных ответов
    text = a + ' − ' + b + ' = ?';
    answer = a - b;
  } else {
    a = randInt(2, 5 + level * 2);
    b = randInt(2, 9);
    text = a + ' × ' + b + ' = ?';
    answer = a * b;
  }
  // Неверные варианты — близкие числа
  const wrong = new Set();
  while (wrong.size < 3) {
    const w = answer + randInt(-10, 10);
    if (w !== answer && w >= 0) wrong.add(w);
  }
  return { text: text, hint: '', answer: String(answer), options: shuffle([answer, ...wrong].map(String)), numeric: true };
}

function makeRusQuestion([word, index]) {
  const letter = word[index] === 'Ё' ? 'Е' : word[index];
  const text = word.slice(0, index) + '_' + word.slice(index + 1);
  const wrong = shuffle(VOWELS.filter(v => v !== letter)).slice(0, 3);
  return { text: text, hint: 'Какая буква пропущена?', answer: letter, options: shuffle([letter, ...wrong]) };
}

function makeEngQuestion([word, translation]) {
  const wrong = shuffle(ENG_WORDS.filter(w => w[1] !== translation)).slice(0, 3).map(w => w[1]);
  return { text: word, hint: 'Переведи на русский', answer: translation, options: shuffle([translation, ...wrong]) };
}

// Пул из 20+ заданий для выбранного предмета
function buildQuestionPool() {
  const subject = settings.subject;
  if (subject === 'rus') {
    questionPool = shuffle(RUS_WORDS).map(makeRusQuestion);
  } else if (subject === 'eng') {
    questionPool = shuffle(ENG_WORDS).map(makeEngQuestion);
  } else {
    const level = DIFFICULTIES.indexOf(findById(DIFFICULTIES, settings.difficulty));
    questionPool = [];
    for (let i = 0; i < 20; i++) questionPool.push(makeMathQuestion(level));
  }
}

function nextQuestion() {
  if (questionPool.length === 0) buildQuestionPool();
  return questionPool.pop();
}

// Сравнение ответов: без пробелов, без разницы в регистре, Ё = Е
function normalize(text) {
  return String(text).trim().toUpperCase().replace(/Ё/g, 'Е');
}


/* ---------------------------------------------------------
   3. БОНУСЫ (ПЕРКИ)
   Верный ответ → 3 случайных бонуса. Каждый — один раз за игру
   --------------------------------------------------------- */

const PERKS = [
  { id: 'multi',  icon: '🔮', title: 'Мульти-шар',      desc: '+2 шара прямо сейчас',              apply: p => { splitBalls(); } },
  { id: 'wide',   icon: '↔️', title: 'Широкая платформа', desc: 'Платформа шире на 40%',           apply: p => { p.width *= 1.4; } },
  { id: 'fire',   icon: '🔥', title: 'Огненный шар',    desc: 'Пробивает кирпичи насквозь',        apply: p => { p.fire = true; } },
  { id: 'laser',  icon: '🔫', title: 'Лазеры',          desc: 'Платформа стреляет вверх',          apply: p => { p.laser = true; } },
  { id: 'slow',   icon: '🐢', title: 'Медленный шар',   desc: 'Шар на 20% медленнее',              apply: p => { p.speed *= 0.8; slowAllBalls(0.8); } },
  { id: 'life',   icon: '❤️', title: '+1 жизнь',        desc: 'Ещё одна попытка',                  apply: p => { game.lives++; } },
  { id: 'sticky', icon: '🍯', title: 'Липкая платформа', desc: 'Шар прилипает, запуск — пробел',   apply: p => { p.sticky = true; } },
  { id: 'score',  icon: '⭐', title: 'Двойные очки',    desc: 'Очки ×2',                           apply: p => { p.scoreMult *= 2; } },
  { id: 'big',    icon: '🔵', title: 'Большой шар',     desc: 'Шар больше и бьёт сильнее',         apply: p => { p.ballSize *= 1.5; p.damage += 1; } },
  { id: 'shield', icon: '🛡️', title: 'Щит снизу',       desc: 'Один раз спасёт упавший шар',       apply: p => { p.shield = true; } },
  { id: 'boom',   icon: '💣', title: 'Взрывной шар',    desc: 'Задевает соседние кирпичи',         apply: p => { p.explosive = true; } },
  { id: 'power',  icon: '💥', title: 'Сила ×2',         desc: 'Двойной урон кирпичам-врагам',      apply: p => { p.damage *= 2; } },
  { id: 'hint',   icon: '✂️', title: 'Подсказка 50/50', desc: 'В вопросе только 2 варианта',       apply: p => { p.hint = true; } },
  { id: 'magnet', icon: '🧲', title: 'Магнит',          desc: 'Шар тянется к платформе',           apply: p => { p.magnet = true; } }
];


/* ---------------------------------------------------------
   4. НАСТРОЙКИ ИГРОКА И РЕКОРД
   --------------------------------------------------------- */

const DEFAULT_SETTINGS = {
  theme: 'space',
  subject: 'math',
  inputMode: 'buttons',
  difficulty: 'easy',
  sound: true,
  volume: 0.7
};

let settings = loadSettings();
let bestScore = loadBestScore();

function loadSettings() {
  try {
    const saved = JSON.parse(localStorage.getItem('quizbreakout-settings'));
    return Object.assign({}, DEFAULT_SETTINGS, saved);
  } catch (error) {
    return Object.assign({}, DEFAULT_SETTINGS);
  }
}

function saveSettings() {
  try {
    localStorage.setItem('quizbreakout-settings', JSON.stringify(settings));
  } catch (error) {
    // Если браузер запретил сохранение — просто играем без него
  }
}

function loadBestScore() {
  try {
    return Number(localStorage.getItem('quizbreakout-best')) || 0;
  } catch (error) {
    return 0;
  }
}

function saveBestScore() {
  try {
    localStorage.setItem('quizbreakout-best', String(bestScore));
  } catch (error) {
    // ничего страшного
  }
}


/* ---------------------------------------------------------
   5. ЗВУКИ (Web Audio — звук играет мгновенно)
   --------------------------------------------------------- */

const isCodePen = location.hostname.includes('codepen') || location.hostname.includes('cdpn.io');
const SOUND_PATH = isCodePen ? 'https://si0683.github.io/QuizBreakout/sound/' : 'sound/';

const SOUND_FILES = {
  jump:  'jump.wav',   // шар ударился о кирпич
  coin:  'coin.mp3',   // верный ответ
  tick:  'tick.wav',   // отсчёт, удар о платформу
  start: 'start.wav',  // бонус, уровень, босс
  crash: 'crash.wav'   // шар упал, неверный ответ
};

const SOUND_START = { coin: 0.045 };

const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const sounds = {};

for (const name in SOUND_FILES) {
  fetch(SOUND_PATH + SOUND_FILES[name])
    .then(response => response.arrayBuffer())
    .then(data => audioContext.decodeAudioData(data))
    .then(buffer => { sounds[name] = buffer; })
    .catch(() => {});
}

function playSound(name) {
  if (!settings.sound || !sounds[name]) return;
  if (audioContext.state === 'suspended') audioContext.resume();
  const source = audioContext.createBufferSource();
  source.buffer = sounds[name];
  const volume = audioContext.createGain();
  volume.gain.value = settings.volume;
  source.connect(volume);
  volume.connect(audioContext.destination);
  source.start(0, SOUND_START[name] || 0);
}


/* ---------------------------------------------------------
   6. ХОЛСТ И РАЗМЕР ИГРЫ
   Высота мира 720, ширина под экран (от 420 до 820)
   --------------------------------------------------------- */

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const stage = document.getElementById('stage');

const H = 720;
let W = 640;
let scale = 1;

function resizeGame() {
  scale = window.innerHeight / H;
  W = window.innerWidth / scale;
  if (W < 420) {
    W = 420;
    scale = window.innerWidth / W;
  }
  if (W > 820) W = 820;
  stage.style.width = W * scale + 'px';
  stage.style.height = H * scale + 'px';
  const ratio = window.devicePixelRatio || 1;
  canvas.width = Math.round(W * scale * ratio);
  canvas.height = Math.round(H * scale * ratio);
  ctx.setTransform(scale * ratio, 0, 0, scale * ratio, 0, 0);
  paddle.x = clamp(paddle.x, paddle.width / 2, W - paddle.width / 2);
}


/* ---------------------------------------------------------
   7. СОСТОЯНИЕ ИГРЫ, ПЛАТФОРМА, ШАРЫ, КИРПИЧИ
   --------------------------------------------------------- */

// Состояния: 'menu', 'choose', 'countdown', 'playing',
// 'quiz', 'perk', 'paused', 'over'
const game = {
  state: 'menu',
  score: 0,
  lives: 0,         // ставится из LAB.lives при старте
  level: 1,
  answers: 0,
  time: 0,
  laserTimer: 0,
  shake: 0,
  usedPerks: []
};

const PADDLE_Y = H - 70;

const paddle = {
  x: 320,
  width: 110,
  // Бонусы (меняются перками)
  speed: 1, fire: false, laser: false, sticky: false, scoreMult: 1, ballSize: 1,
  damage: 1, shield: false, explosive: false, hint: false, magnet: false
};

function resetPaddle() {
  paddle.width = LAB.paddleWidth;
  paddle.speed = LAB.ballSpeed;
  paddle.fire = false;
  paddle.laser = false;
  paddle.sticky = false;
  paddle.scoreMult = 1;
  paddle.ballSize = 1;
  paddle.damage = 1;
  paddle.shield = false;
  paddle.explosive = false;
  paddle.hint = false;
  paddle.magnet = false;
}

let balls = [];      // { x, y, vx, vy, stuck }
let bricks = [];     // { x, y, w, h, hp, maxHp, kind: 'plain'|'enemy'|'quiz', color, emoji }
let boss = null;
let bossShots = [];
let lasers = [];
let particles = [];

const keys = { left: false, right: false };
let pointerX = null;

function heroEmoji() {
  return LAB.heroEmoji || currentTheme().hero;
}

function ballRadius() {
  return 9 * paddle.ballSize;
}

function ballSpeed() {
  return difficulty().speed * paddle.speed * (1 + (game.level - 1) * 0.05);
}

// Новый шар, лежащий на платформе
function newBall() {
  return { x: paddle.x, y: PADDLE_Y - 14 - ballRadius(), vx: 0, vy: 0, stuck: true };
}

// Запустить прилипшие шары вверх под небольшим углом
function launchBalls() {
  for (const b of balls) {
    if (!b.stuck) continue;
    b.stuck = false;
    const angle = (Math.random() - 0.5) * 0.8;
    b.vx = Math.sin(angle) * ballSpeed();
    b.vy = -Math.cos(angle) * ballSpeed();
  }
  document.getElementById('launchHint').classList.add('hidden');
}

// Мульти-шар: каждый летящий шар делится на три
function splitBalls() {
  const source = balls.filter(b => !b.stuck);
  if (source.length === 0) {
    launchBalls();
    source.push(...balls);
  }
  for (const b of source.slice(0, 3)) {
    for (const turn of [-0.4, 0.4]) {
      const angle = Math.atan2(b.vx, -b.vy) + turn;
      balls.push({ x: b.x, y: b.y, vx: Math.sin(angle) * ballSpeed(), vy: -Math.cos(angle) * ballSpeed(), stuck: false });
    }
  }
}

function slowAllBalls(factor) {
  for (const b of balls) {
    b.vx *= factor;
    b.vy *= factor;
  }
}


/* ---------------------------------------------------------
   8. СТРОИМ УРОВЕНЬ
   Кирпичи: обычные (1 удар), враги (прочные, с эмодзи)
   и кирпичи-вопросы ❓. На босс-уровне сверху ходит босс
   --------------------------------------------------------- */

function buildLevel() {
  const theme = currentTheme();
  bricks = [];
  boss = null;
  bossShots = [];
  lasers = [];
  const isBossLevel = game.level % LAB.bossEvery === 0;
  const rows = Math.min(9, LAB.brickRows + Math.floor((game.level - 1) / 2)) - (isBossLevel ? 2 : 0);
  const cols = Math.max(6, Math.floor((W - 40) / 66));
  const gap = 6;
  const brickW = (W - 40 - gap * (cols - 1)) / cols;
  const brickH = 26;
  const top = isBossLevel ? 200 : 110;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      // Узор: иногда пропускаем кирпич, чтобы уровни были разными
      if (Math.random() < 0.12) continue;
      const enemy = Math.random() < 0.18 + game.level * 0.02;
      const hp = enemy ? difficulty().enemyHp + Math.floor(game.level / 3) : 1;
      bricks.push({
        x: 20 + c * (brickW + gap), y: top + r * (brickH + gap), w: brickW, h: brickH,
        hp: hp, maxHp: hp,
        kind: enemy ? 'enemy' : 'plain',
        color: theme.bricks[r % theme.bricks.length],
        emoji: enemy ? theme.enemies[Math.floor(Math.random() * theme.enemies.length)] : '',
        hit: 0
      });
    }
  }

  // Кирпичи-вопросы: заменяем случайные обычные
  const plain = shuffle(bricks.filter(b => b.kind === 'plain'));
  for (const b of plain.slice(0, LAB.quizBricks)) {
    b.kind = 'quiz';
    b.hp = b.maxHp = 1;
    b.emoji = '❓';
  }

  if (isBossLevel) {
    const hp = 18 + game.level * 4;
    boss = { x: W / 2, y: 120, size: 90, hp: hp, maxHp: hp, dir: 1, shotTimer: 2.5, hit: 0 };
    document.getElementById('bossName').textContent = theme.boss.emoji + ' ' + theme.boss.name;
  }

  balls = [];
  for (let i = 0; i < LAB.startBalls; i++) balls.push(newBall());
  document.getElementById('launchHint').classList.remove('hidden');
}


/* ---------------------------------------------------------
   9. РИСОВАНИЕ: ФОН, КИРПИЧИ, ШАРЫ, ЧАСТИЦЫ
   --------------------------------------------------------- */

const TAU = Math.PI * 2;

function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value));
}

function seeded(n) {
  const x = Math.sin(n * 127.1 + 311.7) * 43758.5453;
  return x - Math.floor(x);
}

function drawCircle(x, y, r, color) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x, y, r, 0, TAU);
  ctx.fill();
}

function drawGlow(x, y, r, color) {
  const g = ctx.createRadialGradient(x, y, 0, x, y, r);
  g.addColorStop(0, color);
  g.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = g;
  ctx.fillRect(x - r, y - r, r * 2, r * 2);
}

function drawEmoji(emoji, x, y, size) {
  ctx.font = size + 'px "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji", sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(emoji, x, y);
}

function hexToRgba(color, alpha) {
  if (!color.startsWith('#')) return color.replace('hsl(', 'hsla(').replace(')', ', ' + alpha + ')');
  const n = parseInt(color.slice(1), 16);
  return 'rgba(' + ((n >> 16) & 255) + ',' + ((n >> 8) & 255) + ',' + (n & 255) + ',' + alpha + ')';
}

function drawBackground() {
  const theme = currentTheme();
  const g = ctx.createLinearGradient(0, 0, 0, H);
  g.addColorStop(0, theme.sky[0]);
  g.addColorStop(0.6, theme.sky[1]);
  g.addColorStop(1, theme.sky[2]);
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);
  for (let i = 0; i < 90; i++) {
    ctx.globalAlpha = 0.25 + 0.6 * Math.abs(Math.sin(game.time * 1.3 + i));
    const y = (seeded(i + 30) * H + game.time * (6 + seeded(i) * 18)) % H;
    drawCircle(seeded(i) * W, y, 0.6 + seeded(i + 5) * 1.6, theme.id === 'eco' ? '#b6ff9a' : '#ffffff');
  }
  ctx.globalAlpha = 1;
  drawGlow(W * 0.2, H * 0.35, 260, hexToRgba(theme.accent, 0.08));
  // Пол-«опасная зона» внизу
  const danger = ctx.createLinearGradient(0, H - 40, 0, H);
  danger.addColorStop(0, 'rgba(255,60,90,0)');
  danger.addColorStop(1, 'rgba(255,60,90,0.25)');
  ctx.fillStyle = danger;
  ctx.fillRect(0, H - 40, W, 40);
  if (paddle.shield) {
    ctx.fillStyle = 'rgba(127,220,255,0.7)';
    ctx.fillRect(0, H - 8, W, 4);
  }
}

function drawBricks() {
  for (const b of bricks) {
    const color = b.kind === 'quiz' ? '#ffd84a' : b.color;
    const pop = b.hit > 0 ? 2 : 0;
    ctx.fillStyle = hexToRgba(color, b.kind === 'enemy' ? 0.35 + 0.5 * b.hp / b.maxHp : 0.85);
    ctx.beginPath();
    ctx.roundRect(b.x - pop, b.y - pop, b.w + pop * 2, b.h + pop * 2, 7);
    ctx.fill();
    ctx.strokeStyle = 'rgba(255,255,255,0.35)';
    ctx.lineWidth = 1.5;
    ctx.stroke();
    // Блик сверху
    ctx.fillStyle = 'rgba(255,255,255,0.25)';
    ctx.fillRect(b.x + 4, b.y + 3, b.w - 8, 4);
    if (b.kind === 'quiz') {
      drawGlow(b.x + b.w / 2, b.y + b.h / 2, 30 + Math.sin(game.time * 5) * 4, 'rgba(255,215,0,0.5)');
      drawEmoji('❓', b.x + b.w / 2, b.y + b.h / 2 + 1, 20);
    } else if (b.kind === 'enemy') {
      drawEmoji(b.emoji, b.x + 16, b.y + b.h / 2 + 1, 20);
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 14px "Russo One", Rubik, sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(Math.ceil(b.hp), b.x + b.w - 14, b.y + b.h / 2 + 1);
    }
  }
}

function drawBoss() {
  if (!boss) return;
  drawGlow(boss.x, boss.y, boss.size * 1.1, 'rgba(255,60,90,0.35)');
  drawEmoji(currentTheme().boss.emoji, boss.x, boss.y, boss.size * (boss.hit > 0 ? 1.1 : 1));
  for (const s of bossShots) drawEmoji(currentTheme().boss.shot, s.x, s.y, 28);
}

function drawPaddle() {
  const theme = currentTheme();
  const x = paddle.x - paddle.width / 2;
  drawGlow(paddle.x, PADDLE_Y, paddle.width, hexToRgba(theme.accent, 0.25));
  const g = ctx.createLinearGradient(0, PADDLE_Y - 8, 0, PADDLE_Y + 8);
  g.addColorStop(0, '#ffffff');
  g.addColorStop(0.4, paddle.sticky ? '#ffb84d' : theme.accent);
  g.addColorStop(1, 'rgba(0,0,0,0.4)');
  ctx.fillStyle = g;
  ctx.beginPath();
  ctx.roundRect(x, PADDLE_Y - 8, paddle.width, 16, 8);
  ctx.fill();
  if (paddle.laser) {
    ctx.fillStyle = '#ff4d6d';
    ctx.fillRect(x + 4, PADDLE_Y - 16, 6, 10);
    ctx.fillRect(x + paddle.width - 10, PADDLE_Y - 16, 6, 10);
  }
  // Герой стоит на платформе
  drawEmoji(heroEmoji(), paddle.x, PADDLE_Y + 30, 34);
}

function drawBalls() {
  const theme = currentTheme();
  for (const b of balls) {
    const color = LAB.rainbowBall ? 'hsl(' + Math.round((game.time * 300) % 360) + ', 100%, 60%)'
      : paddle.fire ? '#ff7a2a' : theme.accent;
    drawGlow(b.x, b.y, ballRadius() * 3, hexToRgba(color, 0.5));
    drawCircle(b.x, b.y, ballRadius(), color);
    drawCircle(b.x - ballRadius() * 0.3, b.y - ballRadius() * 0.3, ballRadius() * 0.35, 'rgba(255,255,255,0.8)');
  }
  for (const l of lasers) {
    ctx.fillStyle = '#ff4d6d';
    ctx.fillRect(l.x - 2, l.y - 10, 4, 14);
  }
}

function spawnParticles(x, y, count, color, power) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * TAU;
    const speed = (0.3 + Math.random()) * power;
    particles.push({ x: x, y: y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, life: 0.4 + Math.random() * 0.5, size: 2 + Math.random() * 4, color: color });
  }
}

function updateParticles(dt) {
  for (const p of particles) {
    p.x += p.vx * dt;
    p.y += p.vy * dt;
    p.vy += 400 * dt;
    p.life -= dt;
  }
  particles = particles.filter(p => p.life > 0);
}

function drawParticles() {
  for (const p of particles) {
    ctx.globalAlpha = Math.max(0, p.life * 2);
    drawCircle(p.x, p.y, p.size, p.color);
  }
  ctx.globalAlpha = 1;
}


/* ---------------------------------------------------------
   10. ОКНО ВОПРОСА (КВИЗ)
   --------------------------------------------------------- */

let currentQuestion = null;
let quizLocked = false;

function openQuiz() {
  game.state = 'quiz';
  keys.left = keys.right = false;
  currentQuestion = nextQuestion();
  quizLocked = false;
  const subject = findById(SUBJECTS, settings.subject);

  document.getElementById('quizSubject').textContent = subject.icon + ' ' + subject.label;
  document.getElementById('quizQuestion').innerHTML = currentQuestion.text +
    (currentQuestion.hint ? '<small>' + currentQuestion.hint + '</small>' : '');
  document.getElementById('quizHint').textContent = 'Верный ответ — бонус!';
  document.getElementById('quizCard').classList.remove('flash-bad', 'flash-good');

  const options = document.getElementById('quizOptions');
  const form = document.getElementById('quizForm');
  const inputEl = document.getElementById('quizInput');

  if (settings.inputMode === 'keyboard') {
    options.classList.add('hidden');
    form.classList.remove('hidden');
    inputEl.value = '';
    inputEl.inputMode = currentQuestion.numeric ? 'numeric' : 'text';
    if (paddle.hint) document.getElementById('quizHint').textContent = 'Подсказка: начинается на «' + String(currentQuestion.answer)[0] + '»';
    showScreen('quiz');
    setTimeout(() => inputEl.focus(), 50);
  } else {
    form.classList.add('hidden');
    options.classList.remove('hidden');
    options.innerHTML = '';
    let list = currentQuestion.options;
    if (paddle.hint) {
      const wrong = list.filter(o => normalize(o) !== normalize(currentQuestion.answer));
      list = shuffle([currentQuestion.answer, wrong[0]]);
    }
    list.forEach((text, i) => {
      const button = document.createElement('button');
      button.type = 'button';
      button.className = 'quiz-option';
      button.textContent = text;
      button.title = 'Клавиша ' + (i + 1);
      button.addEventListener('click', () => answerQuiz(text, button));
      options.appendChild(button);
    });
    showScreen('quiz');
  }
}

function answerQuiz(text, button) {
  if (quizLocked || game.state !== 'quiz') return;
  quizLocked = true;
  const correct = normalize(text) === normalize(currentQuestion.answer);
  if (button) button.classList.add(correct ? 'right' : 'wrong');
  restartAnimation(document.getElementById('quizCard'), correct ? 'flash-good' : 'flash-bad');
  if (correct) {
    game.answers++;
    addScore(30);
    playSound('coin');
    setTimeout(openPerks, 500);
  } else {
    playSound('crash');
    for (const b of document.querySelectorAll('.quiz-option')) {
      if (normalize(b.textContent) === normalize(currentQuestion.answer)) b.classList.add('right');
    }
    document.getElementById('quizHint').textContent = 'Правильно: ' + currentQuestion.answer;
    setTimeout(resumeAfterQuiz, 1100);
  }
}

function resumeAfterQuiz() {
  game.state = 'playing';
  showScreen(null);
  updateHud();
}

document.getElementById('quizForm').addEventListener('submit', (event) => {
  event.preventDefault();
  const value = document.getElementById('quizInput').value;
  if (value.trim()) answerQuiz(value, null);
});


/* ---------------------------------------------------------
   11. ЭКРАН БОНУСОВ
   --------------------------------------------------------- */

let perkOffer = [];

function openPerks() {
  const free = PERKS.filter(perk => !game.usedPerks.includes(perk.id));
  if (free.length === 0) {
    addScore(100);
    showToast('Все бонусы собраны! +100');
    resumeAfterQuiz();
    return;
  }
  game.state = 'perk';
  perkOffer = shuffle(free).slice(0, 3);
  const box = document.getElementById('perkChoices');
  box.innerHTML = '';
  perkOffer.forEach((perk, i) => {
    box.appendChild(makeChoice(perk.icon, perk.title, perk.desc, () => pickPerk(perk), i + 1));
  });
  showScreen('perk');
}

function pickPerk(perk) {
  if (game.state !== 'perk') return;
  perk.apply(paddle);
  game.usedPerks.push(perk.id);
  playSound('start');
  showToast(perk.icon + ' ' + perk.title);
  resumeAfterQuiz();
}


/* ---------------------------------------------------------
   12. ИГРОВЫЕ ДЕЙСТВИЯ
   --------------------------------------------------------- */

function addScore(points) {
  game.score += Math.round(points * difficulty().score * paddle.scoreMult);
}

// Удар по кирпичу. Возвращает true, если кирпич разбит
function hitBrick(brick, damage) {
  brick.hp -= brick.kind === 'enemy' ? damage : 1;
  brick.hit = 0.1;
  if (brick.hp > 0) {
    playSound('tick');
    return false;
  }
  bricks = bricks.filter(b => b !== brick);
  spawnParticles(brick.x + brick.w / 2, brick.y + brick.h / 2, brick.kind === 'enemy' ? 18 : 10, brick.kind === 'quiz' ? '#ffd84a' : brick.color, 200);
  playSound('jump');
  addScore(brick.kind === 'enemy' ? 15 * brick.maxHp : 10);
  if (brick.kind === 'quiz') openQuiz();
  return true;
}

// Взрывной шар: задевает кирпичи рядом
function explodeAround(brick) {
  const cx = brick.x + brick.w / 2;
  const cy = brick.y + brick.h / 2;
  for (const other of bricks.slice()) {
    if (Math.hypot(other.x + other.w / 2 - cx, other.y + other.h / 2 - cy) < 80 && other.kind !== 'quiz') hitBrick(other, paddle.damage);
  }
  spawnParticles(cx, cy, 16, '#ff9f43', 260);
}

function hitBoss(damage) {
  boss.hp -= damage;
  boss.hit = 0.15;
  playSound('tick');
  spawnParticles(boss.x, boss.y, 10, '#ff4d6d', 200);
  if (boss.hp <= 0) {
    spawnParticles(boss.x, boss.y, 60, currentTheme().accent, 340);
    addScore(500);
    showToast(currentTheme().boss.name + ' побеждён! +500');
    playSound('start');
    boss = null;
    bossShots = [];
  }
  updateHud();
}

// Все шары упали — минус жизнь
function loseLife() {
  game.lives--;
  game.shake = 0.35;
  playSound('crash');
  if (game.lives <= 0) {
    game.lives = 0;
    gameOver();
    return;
  }
  balls = [newBall()];
  document.getElementById('launchHint').classList.remove('hidden');
  showToast('Шар упал!');
  updateHud();
}

function nextLevel() {
  game.level++;
  addScore(200 * (game.level - 1));
  buildLevel();
  playSound('start');
  showToast('Уровень ' + game.level + (game.level % LAB.bossEvery === 0 ? ': БОСС!' : ''));
  updateHud();
}

function gameOver() {
  game.state = 'over';
  const isRecord = game.score > bestScore;
  if (isRecord) {
    bestScore = game.score;
    saveBestScore();
  }
  document.getElementById('finalScore').textContent = game.score;
  document.getElementById('finalLevels').textContent = game.level - 1;
  document.getElementById('finalAnswers').textContent = game.answers;
  document.getElementById('finalBest').textContent = bestScore;
  document.getElementById('newRecord').classList.toggle('hidden', !isRecord || game.score === 0);
  setTimeout(() => showScreen('over'), 600);
}

function resetGame() {
  resetPaddle();
  game.score = 0;
  game.lives = LAB.lives;
  game.level = 1;
  game.answers = 0;
  game.laserTimer = 0;
  game.usedPerks = [];
  particles = [];
  paddle.x = W / 2;
  buildQuestionPool();
  buildLevel();
  updateHud();
}

function openDifficultySelect() {
  game.state = 'choose';
  const box = document.getElementById('difficultyChoices');
  box.innerHTML = '';
  DIFFICULTIES.forEach((d, i) => {
    box.appendChild(makeChoice(d.icon, d.label, d.desc, () => {
      settings.difficulty = d.id;
      saveSettings();
      startGame();
    }, i + 1));
  });
  showScreen('difficulty');
}

let countdownTimer = null;

function startGame() {
  if (audioContext.state === 'suspended') audioContext.resume();
  resetGame();
  showScreen(null);
  game.state = 'countdown';
  let number = 3;
  showCountdown(number);
  clearInterval(countdownTimer);
  countdownTimer = setInterval(() => {
    number--;
    if (number > 0) {
      showCountdown(number);
    } else {
      clearInterval(countdownTimer);
      showCountdown('Старт!');
      game.state = 'playing';
      playSound('start');
    }
  }, 750);
}

function pauseGame() {
  if (game.state !== 'playing') return;
  game.state = 'paused';
  showScreen('pause');
}

function resumeGame() {
  if (game.state !== 'paused') return;
  game.state = 'playing';
  showScreen(null);
}

function goToMenu() {
  clearInterval(countdownTimer);
  hideCountdown();
  game.state = 'menu';
  showScreen('menu');
}


/* ---------------------------------------------------------
   13. ОБНОВЛЕНИЕ МИРА ЗА ОДИН КАДР
   --------------------------------------------------------- */

function update(dt) {
  if (['quiz', 'perk', 'paused'].includes(game.state)) return;
  game.time += dt;
  updateParticles(dt);
  if (game.shake > 0) game.shake -= dt;
  if (!['playing', 'countdown'].includes(game.state)) return;

  updatePaddle(dt);
  if (game.state !== 'playing') return;

  // Шар двигаем маленькими шагами, чтобы он не «проскакивал» кирпичи
  const steps = 3;
  for (let i = 0; i < steps; i++) {
    updateBalls(dt / steps);
    if (game.state !== 'playing') return;
  }
  updateLasers(dt);
  if (game.state !== 'playing') return;
  updateBoss(dt);
  for (const b of bricks) if (b.hit > 0) b.hit -= dt;

  // Все кирпичи и босс побеждены — следующий уровень
  if (game.state === 'playing' && bricks.length === 0 && !boss) nextLevel();
}

function updatePaddle(dt) {
  let move = 0;
  if (keys.left) move -= 1;
  if (keys.right) move += 1;
  paddle.x += move * 620 * dt;
  if (pointerX !== null) paddle.x += (pointerX - paddle.x) * Math.min(1, dt * 18);
  paddle.x = clamp(paddle.x, paddle.width / 2, W - paddle.width / 2);
  // Прилипший шар едет вместе с платформой
  for (const b of balls) {
    if (b.stuck) {
      b.x = paddle.x + (b.offset || 0);
      b.y = PADDLE_Y - 8 - ballRadius();
    }
  }
}

function updateBalls(dt) {
  const r = ballRadius();
  for (const b of balls) {
    if (b.stuck) continue;
    // Магнит тянет шар к платформе, когда он падает
    if (paddle.magnet && b.vy > 0) b.vx += clamp(paddle.x - b.x, -1, 1) * 260 * dt;

    b.x += b.vx * dt;
    b.y += b.vy * dt;

    // Стены и потолок
    if (b.x < r) { b.x = r; b.vx = Math.abs(b.vx); }
    if (b.x > W - r) { b.x = W - r; b.vx = -Math.abs(b.vx); }
    if (b.y < r) { b.y = r; b.vy = Math.abs(b.vy); }

    // Платформа: угол отскока зависит от того, куда попал шар
    if (b.vy > 0 && b.y + r >= PADDLE_Y - 8 && b.y + r <= PADDLE_Y + 10 && Math.abs(b.x - paddle.x) <= paddle.width / 2 + r) {
      const hit = clamp((b.x - paddle.x) / (paddle.width / 2), -1, 1);
      const angle = hit * 1.05;               // до ~60° в сторону
      const speed = ballSpeed();
      b.vx = Math.sin(angle) * speed;
      b.vy = -Math.cos(angle) * speed;
      b.y = PADDLE_Y - 8 - r;
      if (paddle.sticky) {
        b.stuck = true;
        b.offset = b.x - paddle.x;
        document.getElementById('launchHint').classList.remove('hidden');
      }
      playSound('tick');
    }

    // Кирпичи
    for (const brick of bricks) {
      const nearX = clamp(b.x, brick.x, brick.x + brick.w);
      const nearY = clamp(b.y, brick.y, brick.y + brick.h);
      if (Math.hypot(b.x - nearX, b.y - nearY) > r) continue;
      // Огненный шар пробивает насквозь (кроме прочных врагов)
      const broken = hitBrick(brick, paddle.damage);
      if (paddle.explosive) explodeAround(brick);
      if (!(paddle.fire && broken)) {
        // Отскок: по той оси, где перекрытие меньше
        const overlapX = Math.min(b.x + r - brick.x, brick.x + brick.w - (b.x - r));
        const overlapY = Math.min(b.y + r - brick.y, brick.y + brick.h - (b.y - r));
        if (overlapX < overlapY) b.vx = b.x < brick.x + brick.w / 2 ? -Math.abs(b.vx) : Math.abs(b.vx);
        else b.vy = b.y < brick.y + brick.h / 2 ? -Math.abs(b.vy) : Math.abs(b.vy);
      }
      break; // за шаг — один кирпич
    }
    if (game.state !== 'playing') return;

    // Босс
    if (boss && Math.hypot(b.x - boss.x, b.y - boss.y) < boss.size * 0.45 + r) {
      b.vy = Math.abs(b.vy);
      b.vx += (b.x - boss.x) * 2;
      hitBoss(paddle.damage);
    }
  }

  // Упавшие шары
  for (const b of balls) {
    if (b.y - r > H) {
      if (paddle.shield) {
        paddle.shield = false;
        b.y = H - r - 10;
        b.vy = -Math.abs(b.vy);
        showToast('🛡️ Щит!');
      } else {
        b.lost = true;
      }
    }
  }
  balls = balls.filter(b => !b.lost);
  if (balls.length === 0) loseLife();
}

function updateLasers(dt) {
  if (paddle.laser) {
    game.laserTimer -= dt;
    if (game.laserTimer <= 0) {
      game.laserTimer = 0.45;
      lasers.push({ x: paddle.x - paddle.width / 2 + 7, y: PADDLE_Y - 16 });
      lasers.push({ x: paddle.x + paddle.width / 2 - 7, y: PADDLE_Y - 16 });
    }
  }
  for (const l of lasers) {
    l.y -= 700 * dt;
    const brick = bricks.find(b => l.x > b.x && l.x < b.x + b.w && l.y > b.y && l.y < b.y + b.h);
    if (brick) {
      l.dead = true;
      hitBrick(brick, 1);
      if (game.state !== 'playing') return;
    } else if (boss && Math.hypot(l.x - boss.x, l.y - boss.y) < boss.size * 0.45) {
      l.dead = true;
      hitBoss(1);
    }
  }
  lasers = lasers.filter(l => !l.dead && l.y > -20);
}

// Босс ходит из стороны в сторону и стреляет вниз
function updateBoss(dt) {
  if (!boss) return;
  if (boss.hit > 0) boss.hit -= dt;
  boss.x += boss.dir * (90 + game.level * 6) * dt;
  if (boss.x < 70 || boss.x > W - 70) boss.dir *= -1;
  boss.shotTimer -= dt;
  if (boss.shotTimer <= 0) {
    boss.shotTimer = Math.max(1, 2.4 - game.level * 0.08);
    bossShots.push({ x: boss.x, y: boss.y + 40 });
  }
  for (const s of bossShots) {
    s.y += 200 * dt;
    // Попал в платформу — тряска и минус 20 очков
    if (s.y > PADDLE_Y - 14 && s.y < PADDLE_Y + 14 && Math.abs(s.x - paddle.x) < paddle.width / 2) {
      s.dead = true;
      game.shake = 0.25;
      playSound('crash');
      spawnParticles(s.x, PADDLE_Y, 14, '#ff4d6d', 200);
      addScore(-20);
    }
  }
  bossShots = bossShots.filter(s => !s.dead && s.y < H + 20);
}


/* ---------------------------------------------------------
   14. РИСОВАНИЕ ОДНОГО КАДРА
   --------------------------------------------------------- */

function draw() {
  ctx.save();
  if (game.shake > 0) ctx.translate((Math.random() - 0.5) * 10, (Math.random() - 0.5) * 10);
  drawBackground();
  if (game.state !== 'menu') {
    drawBricks();
    drawBoss();
    drawPaddle();
    drawBalls();
  }
  drawParticles();
  ctx.restore();
  updateBossBar();
}


/* ---------------------------------------------------------
   15. ЭКРАНЫ И ИНТЕРФЕЙС
   --------------------------------------------------------- */

const screens = {
  menu: document.getElementById('menuScreen'),
  difficulty: document.getElementById('difficultyScreen'),
  quiz: document.getElementById('quizScreen'),
  perk: document.getElementById('perkScreen'),
  settings: document.getElementById('settingsScreen'),
  pause: document.getElementById('pauseScreen'),
  over: document.getElementById('gameOverScreen')
};

function showScreen(name) {
  for (const key in screens) {
    screens[key].classList.toggle('hidden', key !== name);
  }
  const inGame = name === null || ['pause', 'quiz', 'perk'].includes(name);
  document.getElementById('hud').classList.toggle('hidden', !inGame);
  document.getElementById('menuBest').textContent = bestScore;
  if (document.activeElement && document.activeElement.blur) document.activeElement.blur();
}

function isVisible(el) {
  return !el.classList.contains('hidden');
}

function makeChoice(icon, title, desc, onPick, key) {
  const button = document.createElement('button');
  button.type = 'button';
  button.className = 'choice';
  button.innerHTML = '<span class="choice-icon">' + icon + '</span><span><span class="choice-title">' + title + '</span>' +
    (desc ? '<br><span class="choice-desc">' + desc + '</span>' : '') + '</span>';
  button.title = 'Клавиша ' + key;
  button.addEventListener('click', onPick);
  return button;
}

function updateHud() {
  document.getElementById('lives').textContent = game.lives > 6 ? '❤️×' + game.lives : '❤️'.repeat(game.lives) || '💔';
  document.getElementById('score').textContent = game.score;
  document.getElementById('level').textContent = game.level;
  document.getElementById('perkChips').innerHTML = game.usedPerks
    .map(id => '<span class="chip">' + PERKS.find(p => p.id === id).icon + '</span>').join('');
}

function updateBossBar() {
  const bar = document.getElementById('bossBar');
  bar.classList.toggle('hidden', !boss || game.state === 'menu');
  if (boss) document.getElementById('bossFill').style.width = Math.max(0, boss.hp / boss.maxHp * 100) + '%';
}

function showCountdown(value) {
  const el = document.getElementById('countdown');
  el.classList.remove('hidden');
  const isText = typeof value === 'string';
  el.innerHTML = '<span class="' + (isText ? 'go' : '') + '">' + value + '</span>';
  if (!isText) playSound('tick');
  if (isText) setTimeout(hideCountdown, 600);
}

function hideCountdown() {
  document.getElementById('countdown').classList.add('hidden');
}

function showToast(text) {
  const el = document.getElementById('toast');
  el.textContent = text;
  restartAnimation(el, 'show');
}

function restartAnimation(el, className) {
  el.classList.remove(className);
  void el.offsetWidth;
  el.classList.add(className);
}

function applyTheme() {
  const theme = currentTheme();
  const subject = findById(SUBJECTS, settings.subject);
  document.documentElement.style.setProperty('--accent', theme.accent);
  document.getElementById('menuInfo').textContent =
    theme.icon + ' ' + theme.label + ' · ' + subject.icon + ' ' + subject.label + ' · ' + findById(INPUT_MODES, settings.inputMode).label;
}


/* ---------------------------------------------------------
   16. ЭКРАН НАСТРОЕК
   --------------------------------------------------------- */

function buildOptions(containerId, items, settingName) {
  const container = document.getElementById(containerId);
  container.innerHTML = '';
  for (const item of items) {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'option' + (settings[settingName] === item.id ? ' selected' : '');
    button.innerHTML = '<span class="option-icon">' + item.icon + '</span><span class="option-title">' + item.label + '</span><span>' + item.desc + '</span>';
    button.addEventListener('click', () => {
      settings[settingName] = item.id;
      saveSettings();
      applyTheme();
      buildSettings();
    });
    container.appendChild(button);
  }
}

function buildSettings() {
  buildOptions('themeOptions', THEMES, 'theme');
  buildOptions('subjectOptions', SUBJECTS, 'subject');
  buildOptions('inputOptions', INPUT_MODES, 'inputMode');
  document.getElementById('soundToggle').checked = settings.sound;
  document.getElementById('volumeRange').value = Math.round(settings.volume * 100);
  document.getElementById('volumeRange').disabled = !settings.sound;
}

document.getElementById('soundToggle').addEventListener('change', (event) => {
  settings.sound = event.target.checked;
  saveSettings();
  buildSettings();
  playSound('coin');
});

document.getElementById('volumeRange').addEventListener('change', (event) => {
  settings.volume = Number(event.target.value) / 100;
  saveSettings();
  playSound('coin');
});

document.getElementById('resetBestButton').addEventListener('click', () => {
  bestScore = 0;
  saveBestScore();
  showToast('Рекорд сброшен');
});


/* ---------------------------------------------------------
   17. УПРАВЛЕНИЕ
   --------------------------------------------------------- */

document.addEventListener('keydown', (event) => {
  if (document.activeElement && document.activeElement.id === 'quizInput') return;
  const code = event.code;
  if (['ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown', 'Space'].includes(code)) event.preventDefault();

  if (code === 'ArrowLeft' || code === 'KeyA') { keys.left = true; pointerX = null; }
  if (code === 'ArrowRight' || code === 'KeyD') { keys.right = true; pointerX = null; }

  const digit = Number(event.key);
  if (digit >= 1 && digit <= 4) {
    let list = null;
    if (game.state === 'quiz') list = document.querySelectorAll('.quiz-option');
    else if (isVisible(screens.difficulty)) list = document.querySelectorAll('#difficultyChoices .choice');
    else if (game.state === 'perk') list = document.querySelectorAll('#perkChoices .choice');
    if (list && list[digit - 1]) list[digit - 1].click();
  }

  if (code === 'Space' || code === 'Enter' || code === 'ArrowUp') {
    if (isVisible(screens.menu)) openDifficultySelect();
    else if (isVisible(screens.over)) startGame();
    else if (game.state === 'playing') launchBalls();
  }
  if (code === 'Escape' || code === 'KeyP') {
    if (game.state === 'playing') pauseGame();
    else if (game.state === 'paused') resumeGame();
    else if (isVisible(screens.settings)) showScreen('menu');
  }
});

document.addEventListener('keyup', (event) => {
  if (event.code === 'ArrowLeft' || event.code === 'KeyA') keys.left = false;
  if (event.code === 'ArrowRight' || event.code === 'KeyD') keys.right = false;
});

// Мышь и палец: платформа едет за указателем, тап — запуск шара
function toGameX(event) {
  const rect = canvas.getBoundingClientRect();
  return (event.clientX - rect.left) / rect.width * W;
}

canvas.addEventListener('pointermove', (event) => {
  if (event.pointerType === 'mouse' || event.buttons) pointerX = toGameX(event);
});
canvas.addEventListener('pointerdown', (event) => {
  pointerX = toGameX(event);
  if (game.state === 'playing') launchBalls();
});

document.getElementById('startButton').addEventListener('click', openDifficultySelect);
document.getElementById('settingsButton').addEventListener('click', () => {
  buildSettings();
  showScreen('settings');
});
document.getElementById('settingsClose').addEventListener('click', () => showScreen('menu'));
document.getElementById('settingsDone').addEventListener('click', () => showScreen('menu'));
document.getElementById('pauseButton').addEventListener('click', pauseGame);
document.getElementById('resumeButton').addEventListener('click', resumeGame);
document.getElementById('pauseMenuButton').addEventListener('click', goToMenu);
document.getElementById('restartButton').addEventListener('click', startGame);
document.getElementById('overMenuButton').addEventListener('click', goToMenu);

document.addEventListener('visibilitychange', () => {
  if (document.hidden) pauseGame();
});

window.addEventListener('resize', resizeGame);


/* ---------------------------------------------------------
   18. ИГРОВОЙ ЦИКЛ И ЗАПУСК
   --------------------------------------------------------- */

let lastTime = performance.now();

function loop(now) {
  let dt = (now - lastTime) / 1000;
  lastTime = now;
  if (dt > 0.05) dt = 0.05;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

document.querySelector('.title-top').textContent = LAB.titleTop;
document.querySelector('.title-bottom').textContent = LAB.titleBottom;
document.title = LAB.titleTop + ' ' + LAB.titleBottom;
resetPaddle();
resizeGame();
applyTheme();
showScreen('menu');
requestAnimationFrame(loop);
````

<!-- SOURCES:END -->
