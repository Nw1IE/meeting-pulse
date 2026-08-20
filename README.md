# meeting-pulse

## Какая будет для MVP
```
meeting-pulse/
│
├── client/                                    # 💻 CLIENT SIDE (Chrome Extension: React + Vite + TypeScript)
│   ├── public/                                # Статические ресурсы
│   │   ├── icons/                             # Иконки расширения (icon16.png, icon48.png, icon128.png)
│   │   └── manifest.json                      # Манифест Chrome Extension (V3)
│   │
│   ├── src/
│   │   ├── api/                               # HTTP Клиенты для взаимодействия с Server API
│   │   │   ├── apiClient.ts                   # Базовый Axios/Fetch инстанс с JWT и 401 обработкой
│   │   │   ├── authApi.ts                     # Запросы авторизации (login, register, refresh)
│   │   │   └── meetingsApi.ts                 # REST-запросы (список созвонов, скачивание PDF)
│   │   │
│   │   ├── background/                        # Background Service Worker (фоновый процесс Chrome)
│   │   │   ├── index.ts                       # Точка входа service worker
│   │   │   ├── signalrClient.ts               # Управление WebSocket-соединением с Server Hub
│   │   │   └── authService.ts                 # Менеджмент JWT-токенов в chrome.storage.local
│   │   │
│   │   ├── content/                           # Content Scripts (Google Meet / Телемост)
│   │   │   ├── index.ts                       # Инициализация скрипта при заходе на созвон
│   │   │   ├── audioCapture.ts                # Захват аудио (tabCapture / Web Audio API -> чанки)
│   │   │   └── domInteractions.ts             # Парсинг DOM (имена спикеров, микрофоны)
│   │   │
│   │   ├── popup/                             # UI Всплывающего окна расширения (React SPA)
│   │   │   ├── index.html                     # HTML-контейнер для Popup
│   │   │   ├── main.tsx                       # Точка монтирования React
│   │   │   ├── Popup.tsx                      # Главный компонент (статус, кнопка Start/Stop)
│   │   │   ├── Popup.css                      # Стили (Glassmorphism / Tailwind)
│   │   │   └── components/                    # UI-компоненты (StatusBadge, LiveTranscript, etc.)
│   │   │
│   │   └── shared/                            # Общие типы и константы для фронтенда
│   │       ├── types/                         # TypeScript интерфейсы (IUser, IMeeting, ISignalR)
│   │       └── constants/                     # Константы (URL, SignalR события, ключи storage)
│   │
│   ├── package.json                           # Зависимости проекта
│   └── vite.config.ts                         # Конфигурация сборщика под Chrome Extension
│
│
├── server/                                    # ⚙️ SERVER SIDE (Modular Monolith + AI Engine: .NET 9)
│   ├── Controllers/                           # REST API Эндпоинты (AuthController, MeetingsController)
│   ├── Hubs/                                  # Real-Time транспорт (MeetingHub.cs для аудио чанков)
│   │
│   ├── AI/                                    # 🧠 МОДУЛЬ ИИ (Инкапсулирован внутри для легкого выноса в будущем)
│   │   ├── Inference/                         # Низкоуровневые движки (WhisperOnnxEngine, SlmNlpEngine)
│   │   ├── Pipelines/                         # Конвейеры (SttPipeline, SummaryPipeline)
│   │   └── Models/                            # Файлы весов ONNX-моделей (в .gitignore)
│   │
│   ├── Services/                              # 💼 Бизнес-логика (MeetingService, PdfReportGenerator, EmailService)
│   ├── Data/                                  # 🗄️ База данных (ApplicationDbContext, Entities, Migrations)
│   ├── Templates/                             # 📄 Razor-шаблоны для генерации PDF-отчетов (MeetingReportTemplate.razor)
│   ├── Models/                                # DTO и модели запросов/ответов
│   ├── appsettings.json                       # Конфигурация БД, JWT, путей к ONNX
│   └── Program.cs                             # Точка входа: DI-контейнер, регистрация AI и SignalR
│
└── README.md                                  # Описание проекта
```

## Какая должна быть архитектура к моменту выхода 
```
meeting-pulse/
│
├── extension/                                   # 🌐 FRONTEND (Chrome Extension: React + Vite + TS)
│   ├── public/                                  # Статические ресурсы
│   │   ├── icon-16.png                          # Иконка расширения (тулбар)
│   │   ├── icon-48.png                          # Иконка расширения (управление расширениями)
│   │   └── icon-128.png                         # Иконка расширения (Chrome Web Store)
│   │
│   ├── src/
│   │   ├── api/                                 # 🌐 Слой HTTP-запросов к REST API
│   │   │   ├── apiClient.ts                     # Базовый fetch-клиент с подстановкой Bearer JWT из chrome.storage
│   │   │   ├── authApi.ts                       # Запросы: POST /api/auth/login, POST /api/auth/refresh
│   │   │   ├── meetingsApi.ts                   # Запросы: GET/POST /api/meetings, GET /api/meetings/{id}/pdf
│   │   │   └── analyticsApi.ts                  # Запросы: GET /api/analytics/summary
│   │   │
│   │   ├── popup/                               # 🖼️ Всплывающий UI-интерфейс
│   │   │   ├── Popup.tsx                        # Главный компонент (кнопка Старт/Стоп, статус SignalR, список встреч)
│   │   │   ├── Popup.css                        # Glassmorphism-стили, анимация пульсации записи
│   │   │   ├── main.tsx                         # Точка монтирования React (ReactDOM.createRoot)
│   │   │   └── index.html                       # HTML-каркас для окно попапа
│   │   │
│   │   ├── background/                          # ⚙️ Service Worker (Фоновые процессы)
│   │   │   ├── index.ts                         # Точка входа фонового скрипта, маршрутизация сообщений
│   │   │   ├── signalrClient.ts                 # Real-time клиент SignalR (стриминг аудио, прием транскрипта)
│   │   │   └── authService.ts                   # Сохранение и чтение JWT-токенов из chrome.storage.local
│   │   │
│   │   ├── content/                             # 👁️ Content Scripts (Инжектируются на страницы Meet/Телемост)
│   │   │   ├── index.ts                         # Диспетчер инжекции (проверка URL и запуск наблюдателей)
│   │   │   ├── audioCapture.ts                  # Захват аудио-потока вкладки через Web Audio API, нарезка на PCM-чанки
│   │   │   └── domInteractions.ts               # MutationObserver: считывание активного спикера и списка участников
│   │   │
│   │   └── shared/                              # 📦 Общие типы и константы
│   │       ├── types/
│   │       │   ├── meeting.ts                   # DTO интерфейсы: Meeting, TranscriptSegment, ActionItem
│   │       │   └── api.ts                       # Интерфейсы ответов API: AuthResponse, ApiError
│   │       └── constants/
│   │           ├── endpoints.ts                 # Константы URL бэкенда и SignalR-хаба
│   │           └── events.ts                    # Константы Chrome runtime сообщений (START_RECORDING, etc.)
│   │
│   ├── manifest.json                            # Конфигурация Chrome Manifest V3
│   ├── package.json                             # Зависимости фронтенда (React, Vite, @microsoft/signalr)
│   ├── tsconfig.json                            # Настройки TypeScript
│   └── vite.config.ts                           # Сборка под расширение (отдельные бандлы popup, background, content)
│
│
├── backend/                                     # ⚙️ CORE API SERVER (.NET 9 / .NET 10)
│   ├── Controllers/                             # 🛣️ REST эндпоинты
│   │   ├── AuthController.cs                    # Авторизация: /api/auth/login, /api/auth/refresh
│   │   ├── MeetingsController.cs                # Встречи: CRUD операций, получение PDF-отчета
│   │   └── AnalyticsController.cs               # Дашборд: статистика по спикерам и встречам
│   │
│   ├── Hubs/                                    # 🔌 Real-Time WebSocket (SignalR)
│   │   ├── MeetingHub.cs                        # Прием бинарного аудио от расширения, отправка распознанного текста
│   │   └── Clients/
│   │       └── IMeetingClient.cs                # Строго типизированный интерфейс сигналов для фронтенда
│   │
│   ├── Data/                                    # 🗄️ База данных (EF Core + PostgreSQL + pgvector)
│   │   ├── ApplicationDbContext.cs              # Контекст БД, настройка векторов и связей
│   │   ├── Entities/
│   │   │   ├── User.cs                          # Пользователи (Email, PasswordHash, RefreshToken)
│   │   │   ├── Meeting.cs                       # Метаданные встречи (Title, Dates, PdfUrl, Vector)
│   │   │   ├── Participant.cs                   # Участники встречи (Email, Name)
│   │   │   ├── TranscriptSegment.cs             # Расшифрованные реплики (Speaker, Text, Timestamp)
│   │   │   └── ActionItem.cs                    # Сгенерированные поручения (Task, Assignee, Status)
│   │   └── Migrations/                          # Миграции EF Core
│   │
│   ├── Infrastructure/                          # 🏗️ Инфраструктурные сервисы
│   │   ├── RabbitMqProducer.cs                  # Отправка задач и аудио в очередь RabbitMQ
│   │   └── EmailClient.cs                       # SMTP-клиент MailKit для рассылки отчетов
│   │
│   ├── Services/                                # 💼 Бизнес-логика
│   │   ├── MeetingService.cs                    # Логика управления состоянием созвонов
│   │   ├── VectorSearchService.cs               # Поиск по прошлым встречам через pgvector
│   │   ├── PdfReportGenerator.cs                # Рендеринг PDF-отчетов из Razor в файл
│   │   ├── S3StorageService.cs                  # Загрузка и скачивание PDF/аудио из MinIO / S3
│   │   ├── NotificationService.cs               # Рассылка итоговых отчетов участникам
│   │   └── AudioStreamBuffer.cs                 # Накопительный буфер аудио перед отправкой в очередь
│   │
│   ├── Templates/                               # 📄 Шаблоны отчетов
│   │   └── MeetingReportTemplate.razor          # HTML/Razor верстка итогового PDF-документа
│   │
│   ├── Models/                                  # DTO запросов и ответов бэкенда
│   ├── appsettings.json                         # Конфигурация DB, JWT, RabbitMQ, S3
│   └── Program.cs                               # Точка входа, DI-контейнер, Middleware
│
│
├── ai-service/                                  # 🧠 AI ENGINE WORKER (.NET + ONNX Runtime)
│   ├── Inference/                               # ⚡ C# движки выполнения нейросетей
│   │   ├── WhisperOnnxEngine.cs                 # Speech-to-Text: Инференс ONNX модели Whisper
│   │   ├── SlmNlpEngine.cs                      # NLP: Инференс дообученной SLM (выделение тезисов и задач)
│   │   ├── TokenizerService.cs                  # Токенизация текста через Microsoft.ML.Tokenizers
│   │   └── AudioPreprocessor.cs                 # Ресемплинг аудио в 16kHz MONO, шумоподавление
│   │
│   ├── Pipelines/                               # 🔄 Конвейеры обработки
│   │   ├── SttPipeline.cs                       # Пайплайн: Audio Chunk -> Preprocess -> Whisper -> Text
│   │   └── SummaryPipeline.cs                   # Пайплайн: Full Transcript -> SLM -> Structured JSON
│   │
│   ├── Services/                                # 👂 Фоновые воркеры
│   │   ├── AudioQueueConsumer.cs                # BackgroundService: чтение аудио из RabbitMQ
│   │   └── EmbeddingGenerator.cs                # Генератор векторных представлений для pgvector
│   │
│   ├── Models/                                  # 🤖 Веса ONNX (Добавлены в .gitignore)
│   │   ├── .gitignore                           # Исключает .onnx файлы из Git
│   │   └── download_models.sh                   # Скрипт автоматического скачивания моделей при билде
│   │
│   ├── Program.cs                               # Точка входа Worker Service
│   └── ai-service.csproj                        # Подключение Microsoft.ML.OnnxRuntime
│
└── README.md                                    # Документация проекта и инструкция по запуску
```


