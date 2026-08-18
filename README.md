# meeting-pulse

## Какая будет для MVP
```
meeting-pulse/
│
├── client/                             # 💻 CLIENT SIDE (Chrome Extension: React + Vite + TypeScript)
│   ├── public/                         # Статические ресурсы
│   │   ├── icons/                      # Иконки расширения (icon16.png, icon48.png, icon128.png)
│   │   └── manifest.json               # Манифест Chrome Extension (V3: permissions, background, content_scripts)
│   │
│   ├── src/
│   │   ├── api/                        # HTTP Клиенты для взаимодействия с Server API
│   │   │   ├── apiClient.ts            # Базовый Axios/Fetch инстанс с подстановкой JWT и обработкой 401
│   │   │   ├── authApi.ts              # Запросы авторизации (login, register, refresh-token)
│   │   │   └── meetingsApi.ts          # REST-запросы (получение списка созвонов, скачивание PDF-отчетов)
│   │   │
│   │   ├── background/                 # Background Service Worker (фоновый процесс Chrome)
│   │   │   ├── index.ts                # Точка входа service worker, обработка жизненного цикла расширения
│   │   │   ├── signalrClient.ts        # Управление WebSocket-соединением с Server Hub (авто-реконнект)
│   │   │   └── authService.ts          # Хранение и менеджмент JWT-токенов в chrome.storage.local
│   │   │
│   │   ├── content/                    # Content Scripts (внедряются в страницы Google Meet / Zoom / Yandex Telemost)
│   │   │   ├── index.ts                # Инициализация скрипта при заходе на страницу созвона
│   │   │   ├── audioCapture.ts         # Захват аудиопотока вкладки (tabCapture / Web Audio API -> PCM/WAV chunks)
│   │   │   └── domInteractions.ts      # Парсинг DOM для получения имён спикеров и статуса активности микрофона
│   │   │
│   │   ├── popup/                      # UI Всплывающего окна расширения (React SPA)
│   │   │   ├── index.html              # HTML-контейнер для Popup
│   │   │   ├── main.tsx                # Точка монтирования React-приложения
│   │   │   ├── Popup.tsx               # Главный компонент (статус записи, кнопка Start/Stop, выбор профиля)
│   │   │   ├── Popup.css               # Стили интерфейса (Glassmorphism / Tailwind)
│   │   │   └── components/             # Переиспользуемые UI-компоненты
│   │   │       ├── StatusBadge.tsx     # Компактный бейдж статуса (Запись / Пауза / Неактивно / Подключение) с пульсирующим индикатором
│   │   │       ├── LiveTranscript.tsx  # Область «живого» стриминга транскрипта с автоскроллом к последней реплике спикера
│   │   │       ├── ControlButtons.tsx  # Главная группа кнопок управления (Запустить запись, Поставить на паузу, Завершить созвон)
│   │   │       ├── MeetingSelector.tsx # Выпадающий список или текстовое поле для привязки текущей трансляции к названию/ID встречи
│   │   │       ├── SpeakerCard.tsx     # Карточка текущего активного спикера (аватар/инициалы, имя, индикатор уровня громкости)
│   │   │       ├── ActionItemsList.tsx # Виджет быстрого просмотра автоматически выделенных AI задач (Action Items) во время созвона
│   │   │       ├── Header.tsx          # Шапка Popup: логотип MeetingPulse, профиль текущего пользователя и кнопка настроек
│   │   │       └── AudioWaveform.tsx   # Визуализатор аудиопотока (анимация эквалайзера для подтверждения захвата звука вкладки)
│   │   │
│   │   └── shared/                     # Общие утилиты и типы для всего фронтенда
│   │       ├── types/                  # TypeScript интерфейсы
│   │       │   ├── index.ts            # Реэкспорт всех интерфейсов для удобного импорта (export * from './IUser' и т.д.)
│   │       │   ├── IUser.ts            # Модели пользователя (пользователь, профиль, токены авторизации, DTO входа/регистрации)
│   │       │   ├── IMeeting.ts         # Модели встречи (метаданные, длительность, статус, список участников, итоговое саммари)
│   │       │   ├── ITranscriptChunk.ts # Структура аудиочанка и текстового сегмента (текст, спикер, timestamp start/end, confidence)
│   │       │   ├── ISpeaker.ts         # Модель спикера встречи (имя, аватар, статус активности микрофона, суммарное время речи)
│   │       │   ├── ISignalR.ts         # Типы для WebSocket-сообщений (команды StartRecording, SendChunk, ReceiveTranscript, Error)
│   │       │   └── IApiResponse.ts     # Унифицированный оберточный тип для ответов REST API (data, error, isSuccess)
│   │       │
│   │       └── constants/              # Константы приложения
│   │           ├── index.ts            # Главный реэкспорт всех констант проекта
│   │           ├── api.constants.ts    # Базовые URL-адреса (API_URL, HUB_URL), эндпоинты REST (LOGIN, MEETINGS, PDF)
│   │           ├── events.constants.ts # Имена методов и событий SignalR (JOIN_MEETING, SEND_AUDIO_CHUNK, RECEIVE_SEGMENT)
│   │           ├── storage.constants.ts# Ключи для chrome.storage.local (AUTH_TOKEN, REFRESH_TOKEN, USER_SETTINGS)
│   │           └── config.constants.ts # Настройки захвата аудио (SAMPLE_RATE = 16000, CHUNK_SIZE_MS = 3000, CHANNELS = 1)
│   │
│   ├── package.json                    # Зависимости (React, SignalR JS Client, Vite, TypeScript)
│   └── vite.config.ts                  # Конфигурация сборщика под спецификацию Chrome Extension
│
│
├── server/                             # ⚙️ SERVER SIDE (Monolith API + AI Engine: .NET 9/10)
│   ├── Controllers/                    # REST API Эндпоинты
│   │   ├── AuthController.cs           # Регистрация, аутентификация, выдача JWT
│   │   └── MeetingsController.cs       # CRUD созвонов, получение финальных отчетов, отдаче PDF
│   │
│   ├── Hubs/                           # Real-Time транспорт (SignalR)
│   │   └── MeetingHub.cs               # Приём бинарных чанков аудио от client/content/audioCapture.ts
│   │
│   ├── AI/                             # 🧠 ИИ-МОДУЛЬ (Встроенный инференс ONNX-моделей)
│   │   ├── Inference/                  # Низкоуровневые движки выполнения моделей
│   │   │   ├── WhisperOnnxEngine.cs    # STT: Распознавание речи через Microsoft.ML.OnnxRuntime
│   │   │   ├── SlmNlpEngine.cs         # NLP: Выделение ключевых тезисов, задач (Action Items) и спикеров
│   │   │   ├── TokenizerService.cs     # Токенизация текста перед подачей в SLM модель
│   │   │   └── AudioPreprocessor.cs    # Конвертация аудиопотока от клиента в 16kHz MONO float32
│   │   ├── Pipelines/                  # Высший уровень обработки данных
│   │   │   ├── SttPipeline.cs          # Потоковый конвейер: Audio Chunk -> Text Segment
│   │   │   └── SummaryPipeline.cs      # Пакетная обработка: Полный транскрипт -> Итоговый структурированный JSON
│   │   └── Models/                     # Файлы весов ONNX-моделей (whisper.onnx, slm-summary.onnx)
│   │
│   ├── Services/                       # 💼 Бизнес-логика приложения
│   │   ├── MeetingService.cs           # Управление сессиями созвонов, агрегация сегментов расшифровки
│   │   ├── PdfReportGenerator.cs       # Генерация PDF-документов с итоговым саммари
│   │   └── NotificationService.cs      # Отправка сформированных отчетов участникам на Email
│   │
│   ├── Data/                           # 🗄️ База данных и ORM
│   │   ├── ApplicationDbContext.cs     # Контекст Entity Framework Core
│   │   ├── Entities/                   # Модели БД: User.cs, Meeting.cs, Participant.cs, TranscriptSegment.cs
│   │   └── Migrations/                 # Миграции структуры БД
│   │
│   ├── Templates/                      # Шаблоны оформления
│   │   └── MeetingReportTemplate.razor # Razor-шаблон для генерации красивых PDF-файлов
│   │
│   ├── Models/                         # DTOs (Data Transfer Objects) и модели запросов/ответов
│   ├── appsettings.json                # Конфигурация БД, JWT-ключи, пути к ONNX-моделям
│   └── Program.cs                      # Точка входа: сборка DI-контейнера, регистрация AI-сервисов и SignalR
│
└── README.md
```

