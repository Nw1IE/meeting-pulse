# meeting-pulse

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
