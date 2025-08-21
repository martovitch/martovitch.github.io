<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Игра: Дизайн и Заработок</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Общие стили для body */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #000000; /* Черный фон */
            display: flex;
            flex-direction: column; /* Элементы будут располагаться по вертикали */
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            box-sizing: border-box;
            position: relative; /* Для позиционирования уведомления и эффекта огня */
            transition: background-color 0.1s ease-out; /* Плавный переход для фона */
            overflow: hidden; /* Скрыть любой скролл, особенно во время переходов меню */
        }

        /* --- Меню (поднимается снизу) --- */
        .side-menu {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 90vh; /* Консистентная высота для расчетов */
            /* Изначально меню полностью скрыто снизу */
            transform: translateY(100%);
            transition: transform 0.5s ease-out; /* Плавный переход для трансформации */
            z-index: 1100; /* Выше игрового поля, но ниже оверлея меню */
            display: flex;
            flex-direction: column;
            border-top-left-radius: 20px; /* Закругление верхних углов */
            border-top-right-radius: 20px; /* Закругление верхних углов */
            background-color: #ffffff; /* Фон меню */
            box-shadow: 0 -4px 15px rgba(0, 0, 0, 0.25); /* Тень сверху */
            padding-bottom: 2rem; /* Отступ снизу для контента */
            box-sizing: border-box; /* Учитываем padding */
        }

        /* Класс, который добавляется JS, когда меню открыто */
        html.menu-open .side-menu {
            transform: translateY(0%); /* Выдвигается, возвращаясь на место */
        }

        /* --- Содержимое меню --- */
        .menu-content {
            flex-grow: 1; /* Позволяет содержимому занимать доступное пространство */
            padding: 0 1.5rem; /* Горизонтальные отступы */
            overflow-y: auto; /* Добавляем скролл, если контента много */
            max-height: 100%; /* Убедимся, что не выходит за пределы родителя */
            box-sizing: border-box; /* Учитываем padding */
            /* Стили для полосы прокрутки */
            scrollbar-width: thin; /* Firefox */
            scrollbar-color: #888 #f1f1f1; /* Firefox */
        }

        /* Webkit browsers (Chrome, Safari) */
        .menu-content::-webkit-scrollbar {
            width: 8px; /* Ширина полосы прокрутки */
        }

        .menu-content::-webkit-scrollbar-track {
            background: #f1f1f1; /* Цвет дорожки прокрутки */
            border-radius: 10px;
        }

        .menu-content::-webkit-scrollbar-thumb {
            background: #888; /* Цвет ползунка прокрутки */
            border-radius: 10px;
        }

        .menu-content::-webkit-scrollbar-thumb:hover {
            background: #555; /* Цвет ползунка прокрутки при наведении */
        }

        .achievement-display {
            margin-top: 1rem;
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
            padding: 0;
        }

        .achievement-display h3 {
            font-size: 1rem;
            font-weight: bold;
            color: #2d3748;
            margin-bottom: 0.75rem;
            text-align: center;
        }

        /* Стиль для отдельного фрейма достижения */
        .achievement-item {
            background-color: #e2e8f0;
            border-radius: 0.75rem;
            padding: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            text-align: center;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
        }

        .achievement-item .achievement-icon {
            font-size: 2.5rem;
            margin-bottom: 0.5rem;
        }

        .achievement-item .achievement-text {
            font-size: 0.95rem;
            color: #4a5568;
            line-height: 1.4;
            margin: 0;
        }

        .achievement-display .no-achievements {
            text-align: center;
            color: #718096;
            font-style: italic;
            padding: 1rem;
        }

        /* --- Наложение меню (для затемнения фона) --- */
        .menu-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background-color: rgba(0, 0, 0, 0.5);
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.5s ease-in-out, visibility 0.5s ease-in-out;
            z-index: 1050; /* Между игровым полем и меню */
        }

        html.menu-open .menu-overlay {
            opacity: 1;
            visibility: visible;
        }

        /* --- Оболочка игрового контента (с эффектом параллакса) --- */
        .game-content-wrapper {
            transition: transform 0.5s ease-out, opacity 0.5s ease-out, filter 0.5s ease-out, border-radius 0.5s ease-out; /* Добавлен border-radius для эффекта карточки */
            width: calc(100% - 32px); /* Учитываем padding body */
            height: calc(100vh - 32px); /* Учитываем padding body */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: absolute; /* Используем absolute для параллакса */
            top: 16px; /* Отступ сверху */
            left: 16px; /* Отступ слева */
            box-sizing: border-box;
            background-color: #000000; /* Цвет фона игрового поля */
            border-radius: 0; /* Изначально без закруглений */
            will-change: transform, opacity, filter, border-radius; /* Оптимизация для анимации */
            z-index: 100; /* Ниже меню и оверлея */
        }
        /* Класс, который добавляется JS, когда меню открыто */
        html.menu-open .game-content-wrapper {
            transform: translateY(-5vh) scale(0.9); /* Сдвиг вверх и уменьшение */
            opacity: 0.7; /* Затемнение */
            filter: blur(2px); /* Небольшое размытие */
            border-radius: 20px; /* Закругление углов для эффекта карточки */
            box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2); /* Тень */
        }

        /* --- Ручка меню (всегда видна внизу экрана) --- */
        .menu-handle {
            position: fixed;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 80px; /* Ширина ручки */
            height: 40px; /* Высота ручки */
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            z-index: 1200; /* Выше всего */
            background-color: #ffffff; /* Фон ручки */
            border-top-left-radius: 20px;
            border-top-right-radius: 20px;
            box-shadow: 0 -4px 10px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease-in-out, background-color 0.3s ease-in-out;
        }

        .menu-handle-arrow {
            /* Эти стили теперь не используются, так как span пустой */
            font-size: 1.5rem; 
            color: #4a5568; 
            transition: none; 
        }
        
        /* --- Подсказка для свайпа --- */
        .swipe-up-hint {
            position: fixed;
            bottom: 0; /* Базовая позиция для анимации */
            left: 50%;
            transform: translateX(-50%) translateY(100%); /* Изначально скрыто (внизу экрана), как personality-motivation */
            transition: transform 0.5s ease-out, opacity 0.5s ease-out; /* Плавные переходы для появления/скрытия */
            z-index: 95;
            color: #4a5568;
            font-size: 1rem;
            font-weight: 500;
            pointer-events: none; /* Не блокирует клики по кнопке */
            opacity: 0; /* Изначально полностью прозрачно */
            padding: 8px 16px; /* Внутренние отступы */
            border-radius: 20px; /* Закругленные углы */
            background-color: rgba(255, 255, 255, 0.7); /* Полупрозрачный фон */
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
            text-align: center; /* Центрируем текст */
            box-sizing: border-box;
            width: 100%; /* Занимает всю ширину родителя для отступов */
            max-width: 480px; /* Ограничиваем максимальную ширину для десктопов, как у высказываний */
            padding-bottom: 80px; /* Отступ текста от нижнего края элемента */
        }

        /* Класс для отображения и анимации появления подсказки */
        .swipe-up-hint.show {
            transform: translateX(-50%) translateY(0%); /* Выезжает в видимую область */
            opacity: 1; /* Становится полностью видимым */
        }

        /* Класс для полного скрытия, когда элемент не должен занимать место */
        .hidden {
            display: none !important;
        }

        /* Когда меню открыто, подсказка должна скрываться с анимацией */
        html.menu-open .swipe-up-hint {
            transform: translateX(-50%) translateY(100%) !important; /* Принудительно скрываем вниз */
            opacity: 0 !important; /* Принудительно делаем прозрачным */
        }

        /* --- Бургер-иконка (больше не используется) --- */
        .menu-toggle {
            display: none;
        }

        /* --- Стили для круглой кнопки --- */
        .click-button {
            background-color: #ffffff; /* Белый цвет */
            color: #000000; /* Черный цвет текста */
            font-weight: bold;
            display: flex; /* Используем flexbox для центрирования содержимого */
            justify-content: center;
            align-items: center;
            width: 200px; /* Фиксированная ширина */
            height: 200px; /* Фиксированная высота */
            border-radius: 9999px; /* Изначально круглая */
            box-shadow: none; /* Убираем тень */
            font-size: 2.25rem; /* Большой размер шрифта */
            line-height: 2.5rem;
            cursor: pointer;
            /* Добавлено transform для анимации размера и настроен transition */
            transform: scale(var(--button-scale, 1)); 
            transition: transform 0.3s ease-out, background-color 0.1s ease-in-out, border-radius 0.3s ease-out; /* Добавили transition для border-radius */
            margin: 0 auto; /* Центрируем кнопку */
            flex-shrink: 0; /* Предотвращаем сжатие кнопки */
            z-index: 10; /* Убедимся, что кнопка поверх эффекта огня */
            position: relative; /* Для z-index */
        }
        .click-button:hover {
            background-color: #f0f0f0; /* Чуть темнее белый при наведении */
        }
        .click-button:active {
            transform: scale(0.95); /* Небольшое уменьшение при нажатии */
        }

        /* Стили для уведомления (Android-style toast) */
        .notification-toast {
            position: fixed; /* Фиксированное положение на экране */
            top: 20px; /* Отступ от верхнего края */
            left: 50%; /* Центрируем по горизонтали */
            transform: translateX(-50%); /* Точное центрирование */
            width: 100%; /* Занимает всю ширину, учитывая padding */
            max-width: 480px; /* Ограничиваем максимальную ширину как у высказываний */
            box-sizing: border-box; /* Включаем padding в ширину */
            padding: 0 20px; /* Отступы от левого и правого края экрана */
            opacity: 0; /* Изначально скрыто */
            visibility: hidden; /* Скрыто для доступности */
            transition: opacity 0.3s ease-in-out, visibility 0.3s ease-in-out;
            z-index: 1000; /* Поверх других элементов */
        }

        .notification-toast.show {
            opacity: 1;
            visibility: visible;
        }

        /* Стили для содержимого уведомления */
        .notification-content {
            background-color: rgba(0, 0, 0, 0.7); /* Полупрозрачный черный фон */
            color: white;
            padding: 12px 20px; /* Внутренние отступы для текста, совпадает с высказываниями */
            border-radius: 20px; /* Закругление 20px */
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            font-size: 1rem; /* Размер шрифта уведомления */
            text-align: center; /* Центрируем текст внутри */
            white-space: normal; /* Разрешаем перенос текста */
            width: 100%; /* Занимает всю ширину родителя */
            box-sizing: border-box; /* Включаем padding в ширину */
        }

        /* Стили для эффекта белого градиента при интенсивном клике */
        .fire-effect {
            position: fixed; /* Фиксированное положение на экране */
            top: 0;
            left: 0;
            width: 100vw; /* Занимает всю ширину окна просмотра */
            height: 100vh; /* Занимает всю высоту окна просмотра */
            background-color: rgba(255, 255, 255, 0); /* Белый, изначально полностью прозрачный */
            opacity: 0; /* Изначально скрыто */
            transition: background-color 0.1s ease-out, opacity 0.1s ease-out; /* Плавные переходы для opacity */
            pointer-events: none; /* Не блокирует клики по кнопке */
            z-index: 5; /* Позиционируем между фоном и кнопкой/счетчиком */
        }

        /* Стили для появления личностей и пузыря речи */
        .personality-motivation {
            position: fixed;
            bottom: 0; /* Изначально находится внизу экрана, за его пределами */
            left: 50%;
            transform: translateX(-50%) translateY(100%); /* Полностью перемещаем за нижнюю границу */
            transition: transform 0.5s ease-out; /* Плавное выезжание */
            z-index: 900; /* Поверх эффекта огня, но под тостом-уведомлением */
            display: flex;
            flex-direction: column;
            align-items: center;
            padding-bottom: 80px; /* Отступ от самого низа экрана */
            /* Добавлены горизонтальные отступы для всего контейнера */
            padding-left: 20px;
            padding-right: 20px;
            box-sizing: border-box; /* Учитываем padding в размере */
            width: 100%; /* Чтобы контейнер занимал всю ширину и отступы работали */
            max-width: 480px; /* Ограничиваем максимальную ширину для десктопов */
        }

        .personality-motivation.show {
            transform: translateX(-50%) translateY(0%); /* Выезжает в видимую область */
        }

        .personality-icon {
            font-size: 1.5rem; /* Эмодзи в 2 раза меньше (было 3rem) */
            margin-bottom: 10px;
        }

        .speech-bubble {
            background-color: rgba(0, 0, 0, 0.75); /* Темный полупрозрачный фон */
            color: white;
            padding: 12px 20px;
            border-radius: 20px; /* Закругление 20px */
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Небольшая тень */
            font-size: 1rem; /* Размер шрифта высказываний теперь совпадает с уведомлениями */
            /* Убрано max-width, чтобы он подстраивался под ширину родителя */
            width: 100%; /* Занимает 100% ширины родительского элемента (с учетом его padding) */
            text-align: center;
            position: relative;
        }

        /* --- Стили для визуального отображения бонуса --- */
        .bonus-visual {
            position: absolute;
            padding: 10px 15px;
            background-color: #ffd700; /* Золотой цвет для бонуса */
            color: #4a5568;
            font-weight: bold;
            font-size: 1.2rem;
            white-space: nowrap;
            opacity: 1;
            animation: bonusFade 5s forwards; /* Анимация, длительность 5 секунд */
            z-index: 1005; /* Выше всего, кроме меню */
            pointer-events: none; /* Пропускает клики */
            display: flex; /* Для центрирования текста */
            justify-content: center;
            align-items: center;
            clip-path: none; 
            border-radius: 9999px; /* Делаем его овальным, как уведомление */
            box-shadow: 0 0 5px rgba(255, 215, 0, 0.8); /* Эффект свечения */
            min-width: 100px; /* Минимальная ширина */
            min-height: 50px; /* Минимальная высота */
        }

        /* Анимация для бонуса: масштабирование, движение вверх и исчезновение */
        @keyframes bonusFade {
            0% {
                opacity: 1;
                transform: translate(-50%, -50%) scale(1) rotate(var(--random-rotation, 0deg));
            }
            100% {
                opacity: 0;
                transform: translate(-50%, -100%) scale(1.2) rotate(var(--random-rotation, 0deg)); /* Движение вверх и небольшое увеличение */
            }
        }
    </style>
</head>
<body>
    <!-- Canvas для фигур -->
    <canvas id="shapesCanvas" style="position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; z-index: 1090; pointer-events: none;"></canvas>

    <!-- Оболочка игрового контента, которая будет сдвигаться -->
    <div id="gameContentWrapper" class="game-content-wrapper">
        <!-- Счет кликов теперь внутри кнопки -->
        <button id="clickButton" class="click-button">
            <span id="score" class="font-extrabold">0 ₽</span>
        </button>

        <!-- Элемент для эффекта белого градиента при интенсивном клике -->
        <div id="fireEffect" class="fire-effect"></div>

        <!-- Элемент для уведомлений -->
        <div id="notificationToast" class="notification-toast">
            <!-- Текст уведомления будет вставляться сюда JS -->
        </div>

        <!-- Элемент для личности и ее речи -->
        <div id="personalityMotivation" class="personality-motivation">
            <div id="personalityIcon" class="personality-icon"></div>
            <div id="personalitySpeechBubble" class="speech-bubble"></div>
        </div>

        <!-- Подсказка для свайпа -->
        <div id="swipeUpHint" class="swipe-up-hint hidden">Свайпни вверх, чтобы посмотреть все достижения</div>
    </div>

    <!-- Боковое меню (теперь поднимается снизу) -->
    <div id="sideMenu" class="side-menu">
        <div class="menu-content">
            <div id="achievementDisplay" class="achievement-display">
                <!-- Достижения будут отображаться здесь при открытии меню -->
            </div>
        </div>
    </div>

    <!-- Наложение меню (для затемнения фона) -->
    <div id="menuOverlay" class="menu-overlay"></div>

    <!-- Ручка меню (всегда видна внизу экрана) -->
    <div id="menuHandle" class="menu-handle">
        <span id="menuHandleArrow" class="menu-handle-arrow"></span> <!-- Стрелка убрана -->
    </div>

    <script>
        // Получаем элементы DOM
        const html = document.documentElement;
        const scoreDisplay = document.getElementById('score');
        const clickButton = document.getElementById('clickButton');
        const notificationToast = document.getElementById('notificationToast');
        const fireEffect = document.getElementById('fireEffect');
        const personalityMotivation = document.getElementById('personalityMotivation');
        const personalityIcon = document.getElementById('personalityIcon');
        const personalitySpeechBubble = document.getElementById('personalitySpeechBubble');

        // Элементы меню
        const sideMenu = document.getElementById('sideMenu');
        const menuHandle = document.getElementById('menuHandle');
        const menuHandleArrow = document.getElementById('menuHandleArrow'); 
        const menuOverlay = document.getElementById('menuOverlay');
        const gameContentWrapper = document.getElementById('gameContentWrapper');
        const achievementDisplay = document.getElementById('achievementDisplay');
        const swipeUpHint = document.getElementById('swipeUpHint');

        // Canvas для фигур
        const shapesCanvas = document.getElementById('shapesCanvas');
        const shapesCtx = shapesCanvas.getContext('2d');

        // Инициализируем счет
        let score = 0;

        // Переменные для отслеживания интенсивности кликов и эффекта
        let clickIntensity = 0;
        const INTENSITY_INCREASE_PER_CLICK = 0.1;
        const INTENSITY_DECAY_RATE = 0.005;
        const MAX_INTENSITY = 1;

        // Пороги для наград и соответствующие сообщения
        const rewards = [
            { threshold: 10, message: 'Первый заказ на 10 ₽! Отличное начало портфолио.' },
            { threshold: 50, message: '50 ₽! Ваш дизайн продается! Клиенты ценят вашу работу.' },
            { threshold: 100, message: '100 ₽! Вы закрыли первый крупный проект. Так держать!' },
            { threshold: 200, message: '200 ₽! Заработок растет! Время подумать о новых инструментах.' },
            { threshold: 500, message: '500 ₽! Вы - полноценный фрилансер. Дизайн - это ваша стихия!' },
            { threshold: 1000, message: '1 000 ₽! Отличный доход за креативность. Подумайте о расширении.' },
            { threshold: 2000, message: '2 000 ₽! Заказчики в восторге! Ваши навыки — ваша прибыль.' },
            { threshold: 5000, message: '5 000 ₽! Поздравляем, вы стали востребованным дизайнером!' },
            { threshold: 10000, message: '10 000 ₽! Ваш бренд узнают. Заказы сыпятся один за другим.' },
            { threshold: 25000, message: '25 000 ₽! Вы управляете своей дизайн-империей!' },
            { threshold: 50000, message: '50 000 ₽! Вы настоящий гуру дизайна и бизнеса!' }
        ];

        // Мотивационные фразы и иконки известных личностей (теперь дизайнеры/предприниматели)
        const personalities = [
            { icon: '🎨', phrases: [ // Иконка палитры для креативности
                'Каждый ваш клик — это мазок на холсте вашего успеха.',
                'Дизайн – это не то, что ты видишь, а то, что ты заставляешь других видеть.',
                'Начните с чистого листа, а закончите шедевром. Просто кликните!',
                'Креативность — это не талант, а способ оперирования. Кликайте с идеями!',
                'Лучший дизайн еще не создан. Возможно, он рождается прямо сейчас, с каждым кликом.'
            ]},
            { icon: '💼', phrases: [ // Иконка портфеля для бизнеса/заработка
                'Время — деньги. Каждый ваш клик увеличивает ваш капитал.',
                'Не бойтесь рисковать в дизайне, бойтесь не создавать ничего нового.',
                'Успех в бизнесе и дизайне — это 1% вдохновения и 99% кликов.',
                'Ваша прибыль — это отражение вашего труда и креативности.',
                'Деньги приходят к тем, кто создает ценность. Продолжайте создавать!'
            ]},
            { icon: '💡', phrases: [ // Иконка лампочки для идей
                'Гениальные идеи начинаются с одного клика.',
                'Кликайте, чтобы осветить новые пути в дизайне и заработке.',
                'Инновации — это то, что отличает лидера от последователя.',
                'Каждый клик — это новая искра вдохновения.',
                'Не просто кликайте, кликайте с целью — ищите идеи!'
            ]},
            { icon: '📈', phrases: [ // Иконка графика для роста
                'Ваш доход растет с каждым кликом. Вижу потенциал!',
                'Кликайте на пути к финансовой свободе.',
                'Успех — это сумма маленьких усилий, повторяемых изо дня в день.',
                'Стремитесь к росту, как в дизайне, так и в заработке.',
                'Ваш клик — это инвестиция в ваше будущее.'
            ]},
            { icon: '🏆', phrases: [ // Иконка кубка для достижений
                'Каждый клик приближает вас к победе в мире дизайна.',
                'Зарабатывайте свои трофеи кликами!',
                'Достижения — это результат упорного труда и веры в себя.',
                'Вы достойны всех наград за ваш труд.',
                'Побеждайте в каждом проекте, в каждом клике.'
            ]},
            { icon: '💻', phrases: [ // Иконка ноутбука для цифрового дизайна
                'Ваш цифровой холст ждет ваших кликов!',
                'Кликайте, чтобы создать идеальный UI/UX.',
                'Век цифрового дизайна — это ваш золотой век.',
                'Ваши навыки на экране приносят реальные деньги.',
                'Осваивайте новые программы, кликайте уверенно.'
            ]}
        ];

        const achievedRewards = new Set();
        let notificationTimeoutId = null;

        let lastPersonalityShowTime = 0;
        const PERSONALITY_MIN_DELAY = 10000;
        const PERSONALITY_APPEAR_CHANCE = 0.075;

        const BONUS_CHANCE = 0.03;
        const BASE_BONUS_CLICKS = 10;
        const MAX_ADDITIONAL_BONUS_CLICKS = 50;
        
        const EXP_BASE_FOR_SCORE_BONUS = 1.002;
        const EXP_POWER_DIVISOR_FOR_SCORE_BONUS = 50;

        let startY = 0; 
        let isSwiping = false; 
        let currentMenuState = 'closed';

        // Добавляем переменную для хранения текущей формы кнопки
        let currentButtonShape = 'circle';
        // Возможные формы кнопки
        const buttonShapes = ['circle', 'square', 'triangle'];

        const notificationContent = document.createElement('div');
        notificationContent.classList.add('notification-content');
        notificationToast.appendChild(notificationContent);

        // --- Функции меню ---
        function updateAchievementsDisplay() {
            achievementDisplay.innerHTML = ''; 
            
            const earnedAchievements = rewards.filter(reward => achievedRewards.has(reward.threshold));

            const achievementsHeader = document.createElement('h3');
            achievementsHeader.textContent = 'Ваши дизайнерские доходы:';
            achievementDisplay.appendChild(achievementsHeader);

            if (earnedAchievements.length === 0) {
                const noAchievementsText = document.createElement('p');
                noAchievementsText.classList.add('no-achievements');
                noAchievementsText.textContent = 'Пока нет значимых доходов. Создавайте и зарабатывайте!';
                achievementDisplay.appendChild(noAchievementsText);
            } else {
                earnedAchievements.forEach(reward => {
                    const achievementItem = document.createElement('div');
                    achievementItem.classList.add('achievement-item');

                    const achievementIcon = document.createElement('div');
                    achievementIcon.classList.add('achievement-icon');
                    achievementIcon.textContent = '💰';

                    const achievementText = document.createElement('p');
                    achievementText.classList.add('achievement-text');
                    achievementText.textContent = reward.message;

                    achievementItem.appendChild(achievementIcon);
                    achievementItem.appendChild(achievementText);
                    achievementDisplay.appendChild(achievementItem);
                });
            }
        }

        // --- Функции управления параллаксом и состоянием меню ---
        function updateMenuParallax(openRatio) {
            openRatio = Math.min(1, Math.max(0, openRatio));

            const menuTransformY = 100 - openRatio * 100;
            sideMenu.style.transform = `translateY(${menuTransformY}%)`;

            gameContentWrapper.style.transform = `translateY(${-5 * openRatio}vh) scale(${1 - 0.1 * openRatio})`;
            gameContentWrapper.style.opacity = `${1 - 0.3 * openRatio}`;
            gameContentWrapper.style.filter = `blur(${2 * openRatio}px)`;
            gameContentWrapper.style.borderRadius = `${20 * openRatio}px`;
            gameContentWrapper.style.boxShadow = `0 ${5 * openRatio}px ${20 * openRatio}px rgba(0, 0, 0, ${0.2 * openRatio})`;
        }

        function updateSwipeUpHintState() {
            const shouldDisplay = achievedRewards.size >= 2 && 
                                  currentMenuState === 'closed' && 
                                  !personalityMotivation.classList.contains('show');

            if (shouldDisplay) {
                if (swipeUpHint.classList.contains('hidden')) {
                    swipeUpHint.classList.remove('hidden'); 
                    setTimeout(() => {
                        swipeUpHint.classList.add('show'); 
                    }, 50); 
                } else if (!swipeUpHint.classList.contains('show')) {
                    swipeUpHint.classList.add('show');
                }
            } else {
                if (swipeUpHint.classList.contains('show')) {
                    swipeUpHint.classList.remove('show');
                    setTimeout(() => {
                        if (!swipeUpHint.classList.contains('show')) {
                             swipeUpHint.classList.add('hidden');
                        }
                    }, 500); 
                } else if (!swipeUpHint.classList.contains('hidden')) {
                    swipeUpHint.classList.add('hidden');
                }
            }
        }

        const handleTouchStart = (e) => {
            if (currentMenuState === 'opening' || currentMenuState === 'closing') return;
            startY = e.touches[0].clientY;
            isSwiping = true;
            sideMenu.style.transition = 'none';
            gameContentWrapper.style.transition = 'none';
        };

        const handleTouchMove = (e) => {
            if (!isSwiping) return;
            e.preventDefault(); 
        };

        const handleTouchEnd = (e) => {
            if (!isSwiping) return;
            isSwiping = false;

            const totalVerticalMovement = startY - e.changedTouches[0].clientY; 
            const swipeThreshold = 5; 
            const activeAreaThreshold = window.innerHeight * 0.70; 

            if ((currentMenuState === 'closed' || currentMenuState === 'closing') && totalVerticalMovement > swipeThreshold) {
                if (startY > activeAreaThreshold) {
                    openMenu(); 
                }
            } else if ((currentMenuState === 'open' || currentMenuState === 'opening') && totalVerticalMovement < -swipeThreshold) {
                closeMenu(); 
            }
        };

        gameContentWrapper.addEventListener('touchstart', handleTouchStart);
        gameContentWrapper.addEventListener('touchmove', handleTouchMove);
        gameContentWrapper.addEventListener('touchend', handleTouchEnd);

        menuHandle.addEventListener('touchstart', handleTouchStart);
        menuHandle.addEventListener('touchmove', handleTouchMove);
        menuHandle.addEventListener('touchend', handleTouchEnd);

        sideMenu.addEventListener('touchstart', handleTouchStart);
        sideMenu.addEventListener('touchmove', handleTouchMove);
        sideMenu.addEventListener('touchend', handleTouchEnd);

        menuHandle.addEventListener('click', () => {
            if (currentMenuState === 'open') {
                closeMenu();
            } else {
                openMenu();
            }
        });

        menuOverlay.addEventListener('click', closeMenu);
        
        // --- Основные функции игры ---
        function updateScoreDisplay() {
            scoreDisplay.textContent = `${score} ₽`;
        }

        function showNotification(message) {
            if (notificationTimeoutId) {
                clearTimeout(notificationTimeoutId);
            }

            notificationContent.textContent = message;
            notificationToast.classList.add('show');

            notificationTimeoutId = setTimeout(() => {
                notificationToast.classList.remove('show');
            }, 6000);
        }

        function showPersonality() {
            updateSwipeUpHintState(); 

            if (Date.now() - lastPersonalityShowTime < PERSONALITY_MIN_DELAY) {
                return;
            }

            const randomPersonality = personalities[Math.floor(Math.random() * personalities.length)];
            const randomPhrase = randomPersonality.phrases[Math.floor(Math.random() * randomPersonality.phrases.length)];

            personalityIcon.textContent = randomPersonality.icon;
            personalitySpeechBubble.textContent = randomPhrase;
            personalityMotivation.classList.add('show');
            lastPersonalityShowTime = Date.now();

            setTimeout(() => {
                personalityMotivation.classList.remove('show');
                updateSwipeUpHintState(); 
            }, 5000); 
        }

        function animateIntensity() {
            clickIntensity = Math.max(0, clickIntensity - INTENSITY_DECAY_RATE);

            const buttonScale = 1 + clickIntensity * 0.70;
            clickButton.style.setProperty('--button-scale', buttonScale);

            // Белый прозрачный фон при интенсивном нажатии
            fireEffect.style.backgroundColor = `rgba(255, 255, 255, ${clickIntensity * 0.5})`; // Макс 50% прозрачности
            fireEffect.style.opacity = clickIntensity; // Дополнительный контроль видимости

            requestAnimationFrame(animateIntensity);
        }

        // --- Логика появления фигур ---
        let activeShapes = [];

        // Функция для изменения размера холста фигур
        function resizeShapesCanvas() {
            shapesCanvas.width = window.innerWidth;
            shapesCanvas.height = window.innerHeight;
        }

        // Класс для отдельных фигур
        class Shape {
            constructor(x, y, type, color, size) {
                this.x = x;
                this.y = y;
                this.type = type; // 'circle', 'square', 'triangle'
                this.color = color;
                this.size = size;
                this.alpha = 1;
                this.fadingRate = Math.random() * 0.01 + 0.005; 
                this.rotation = Math.random() * Math.PI * 2;
                this.rotationSpeed = (Math.random() - 0.5) * 0.05; 
                this.vx = (Math.random() - 0.5) * 4; 
                this.vy = (Math.random() - 0.5) * 4; 
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.alpha -= this.fadingRate;
                this.rotation += this.rotationSpeed;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.alpha);
                ctx.fillStyle = this.color;
                ctx.translate(this.x, this.y);
                ctx.rotate(this.rotation);

                if (this.type === 'circle') {
                    ctx.beginPath();
                    ctx.arc(0, 0, this.size / 2, 0, Math.PI * 2);
                    ctx.fill();
                } else if (this.type === 'square') {
                    ctx.fillRect(-this.size / 2, -this.size / 2, this.size, this.size);
                } else if (this.type === 'triangle') { 
                    ctx.beginPath();
                    ctx.moveTo(0, -this.size / 2);
                    ctx.lineTo(this.size / 2, this.size / 2);
                    ctx.lineTo(-this.size / 2, this.size / 2);
                    ctx.closePath();
                    ctx.fill();
                }
                ctx.restore();
            }
        }

        // Главный цикл анимации фигур
        function animateShapes() {
            shapesCtx.clearRect(0, 0, shapesCanvas.width, shapesCanvas.height); 

            activeShapes.forEach(shape => shape.update());
            activeShapes.forEach(shape => shape.draw(shapesCtx));

            activeShapes = activeShapes.filter(shape => shape.alpha > 0);

            requestAnimationFrame(animateShapes);
        }

        // Функция для запуска фигур (теперь черно-белых)
        function triggerShapes() {
            const shapeTypes = ['circle', 'square', 'triangle'];
            const colors = ['#000000', '#FFFFFF']; // Черный и белый цвета
            
            for (let i = 0; i < 15; i++) { 
                const randomX = Math.random() * shapesCanvas.width;
                const randomY = Math.random() * shapesCanvas.height;
                const randomType = shapeTypes[Math.floor(Math.random() * shapeTypes.length)];
                const randomColor = colors[Math.floor(Math.random() * colors.length)];
                const randomSize = Math.random() * 20 + 10; 

                activeShapes.push(new Shape(randomX, randomY, randomType, randomColor, randomSize));
            }
        }

        // Функция для изменения формы кнопки
        function changeButtonShape() {
            // Выбираем новую форму, исключая текущую
            let newShape;
            do {
                newShape = buttonShapes[Math.floor(Math.random() * buttonShapes.length)];
            } while (newShape === currentButtonShape);

            currentButtonShape = newShape;

            // Применяем соответствующий border-radius
            if (currentButtonShape === 'circle') {
                clickButton.style.borderRadius = '9999px';
            } else if (currentButtonShape === 'square') {
                clickButton.style.borderRadius = '0px'; // Для квадрата
            } else if (currentButtonShape === 'triangle') {
                // Для треугольника используем clip-path, а border-radius убираем
                clickButton.style.borderRadius = '0px';
                clickButton.style.clipPath = 'polygon(50% 0%, 0% 100%, 100% 100%)';
            } else {
                clickButton.style.borderRadius = '9999px'; // По умолчанию круг
                clickButton.style.clipPath = 'none';
            }

            // Убедимся, что clipPath сбрасывается для не-треугольников
            if (currentButtonShape !== 'triangle') {
                clickButton.style.clipPath = 'none';
            }
        }

        // Обработчик клика по основной кнопке игры
        clickButton.addEventListener('click', () => {
            score++; 
            updateScoreDisplay();

            clickIntensity = Math.min(MAX_INTENSITY, clickIntensity + INTENSITY_INCREASE_PER_CLICK);

            if (Math.random() < BONUS_CHANCE) {
                const scoreBonus = Math.round(Math.pow(EXP_BASE_FOR_SCORE_BONUS, Math.max(1, score) / EXP_POWER_DIVISOR_FOR_SCORE_BONUS));
                const bonusClicks = BASE_BONUS_CLICKS + Math.round(clickIntensity * MAX_ADDITIONAL_BONUS_CLICKS) + scoreBonus;

                score += bonusClicks;
                updateScoreDisplay();

                const bonusVisual = document.createElement('div');
                bonusVisual.classList.add('bonus-visual');
                bonusVisual.textContent = `БОНУС +${bonusClicks} ₽`; 

                const randomRotation = Math.random() * 180 - 90;
                bonusVisual.style.setProperty('--random-rotation', `${randomRotation}deg`);

                const buttonRect = clickButton.getBoundingClientRect();
                const gameWrapperRect = gameContentWrapper.getBoundingClientRect();

                const bonusVisualHeight = 50;
                const posX = buttonRect.left + (buttonRect.width / 2) - gameWrapperRect.left;
                const posY = buttonRect.top - gameWrapperRect.top - bonusVisualHeight / 2 - 20;

                bonusVisual.style.left = `${posX}px`;
                bonusVisual.style.top = `${posY}px`;
                
                gameContentWrapper.appendChild(bonusVisual);

                setTimeout(() => {
                    bonusVisual.remove();
                }, 5000);
            }

            rewards.forEach(reward => {
                if (score >= reward.threshold && !achievedRewards.has(reward.threshold)) {
                    showNotification(reward.message);
                    achievedRewards.add(reward.threshold);
                    triggerShapes(); // Запускаем фигуры при получении достижения
                    changeButtonShape(); // Меняем форму кнопки при получении достижения
                    updateSwipeUpHintState(); 
                }
            });

            if (Math.random() < PERSONALITY_APPEAR_CHANCE) {
                showPersonality(); 
            }
        });

        // Открытие меню
        function openMenu() {
            if (currentMenuState === 'open' || currentMenuState === 'opening') return;
            currentMenuState = 'opening';
            html.classList.add('menu-open');
            
            updateSwipeUpHintState(); 
            
            sideMenu.style.transition = 'transform 0.5s ease-out';
            gameContentWrapper.style.transition = 'transform 0.5s ease-out, opacity 0.5s ease-out, filter 0.5s ease-out, border-radius 0.5s ease-out, box-shadow 0.5s ease-out';
            
            updateMenuParallax(1); 
            updateAchievementsDisplay();

            setTimeout(() => {
                currentMenuState = 'open';
                sideMenu.style.transition = 'none';
                gameContentWrapper.style.transition = 'none';
            }, 500);
        }

        // Закрытие меню
        function closeMenu() {
            if (currentMenuState === 'closed' || currentMenuState === 'closing') return;
            currentMenuState = 'closing';
            html.classList.remove('menu-open');
            
            sideMenu.style.transition = 'transform 0.5s ease-out';
            gameContentWrapper.style.transition = 'transform 0.5s ease-out, opacity 0.5s ease-out, filter 0.5s ease-out, border-radius 0.5s ease-out, box-shadow 0.5s ease-out';

            updateMenuParallax(0); 
            
            setTimeout(() => {
                currentMenuState = 'closed';
                sideMenu.style.transition = 'none';
                gameContentWrapper.style.transition = 'none';
                updateSwipeUpHintState(); 
            }, 500);
        }

        // Инициализация при загрузке страницы
        document.addEventListener('DOMContentLoaded', function() {
            resizeShapesCanvas(); // Устанавливаем начальный размер холста фигур
            updateScoreDisplay();
            animateIntensity();
            animateShapes(); // Запускаем анимацию фигур
            updateMenuParallax(0); 
            updateSwipeUpHintState(); 
            // Устанавливаем начальную форму кнопки
            clickButton.style.borderRadius = '9999px'; 
            clickButton.style.clipPath = 'none';
        });

        // Слушаем изменение размера окна для корректного отображения фигур
        window.addEventListener('resize', resizeShapesCanvas);
    </script>
</body>
</html>
