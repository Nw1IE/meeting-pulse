# meeting-pulse
```
meeting-pulse/
│
├── extension/                         # 🌐 Frontend (React + Vite + TypeScript)
│   ├── src/
│   │   ├── popup/                     # Всплывающее UI-окно расширения
│   │   │   ├── Popup.tsx              # Главный React-компонент (статус встречи, управление)
│   │   │   ├── Popup.css              # Glassmorphism стили
│   │   │   ├── main.tsx               # Точка монтирования React
│   │   │   └── index.html
│   │   │
│   │   ├── background/                # Service Worker (работает в фоне)
│   │   │   ├── index.ts               # Точка входа
│   │   │   ├── signalrClient.ts       # SignalR соединение с бэкендом
│   │   │   └── authService.ts         # JWT авторизация (chrome.storage)
│   │   │
│   │   ├── content/                   # Content Scripts (инжектируются в Meet/Телемост)
│   │   │   ├── index.ts               # Точка входа DOM-наблюдателей
│   │   │   ├── audioCapture.ts        # Захват звука с вкладки/микрофона (Web Audio API)
│   │   │   └── domInteractions.ts     # Извлечение списка участников и текущего спикера
│   │   │
│   │   └── shared/                    # Общие типы и константы
│   │       ├── types/                 # DTO, события, интерфейсы
│   │       └── constants/             # Эндпоинты, ключи
│   │
│   ├── manifest.json                  # Chrome Manifest V3
│   ├── package.json
│   └── vite.config.ts
│
│
├── backend/                           # ⚙️ Core API Server (.NET 9 / .NET 10)
│   ├── Controllers/                   # REST API эндпоинты
│   │   ├── AuthController.cs          # Авторизация, токены
│   │   ├── MeetingsController.cs      # Список встреч, детализация, ручная выгрузка PDF
│   │   └── AnalyticsController.cs     # Аналитика по созвонам
│   │
│   ├── Hubs/                          # Real-Time WebSocket коммуникация
│   │   ├── MeetingHub.cs              # Прием аудио-потока, стриминг транскрипта
│   │   └── Clients/IMeetingClient.cs  # Интерфейсы сигналов для фронтенда
│   │
│   ├── Services/                      # Бизнес-логика
│   │   ├── MeetingService.cs          # Управление состоянием созвона
│   │   ├── VectorSearchService.cs     # Поиск по эмбеддингам через pgvector
│   │   ├── PdfReportGenerator.cs      # Генерация красивых PDF-отчетов (QuestPDF / HTML-to-PDF)
│   │   ├── NotificationService.cs     # Рассылка PDF всем участникам встречи (SMTP / Email / Telegram)
│   │   └── AudioStreamBuffer.cs       # Буферизация аудио-чанков перед передачей в AI
│   │
│   ├── Data/                          # База данных (EF Core + PostgreSQL)
│   │   ├── ApplicationDbContext.cs
│   │   ├── Entities/
│   │   │   ├── User.cs                # Пользователи системы
│   │   │   ├── Meeting.cs             # Метаданные встречи, ссылка на PDF, вектор
│   │   │   ├── Participant.cs         # Участники встречи (Email, имя) для рассылки
│   │   │   ├── TranscriptSegment.cs   # Расшифрованные реплики спикеров
│   │   │   └── ActionItem.cs          # Сгенерированные задачи и поручения
│   │   └── Migrations/
│   │
│   ├── Templates/                     # Шаблоны документов
│   │   └── MeetingReportTemplate.razor# HTML/Razor шаблон для генерации PDF
│   │
│   ├── Models/                        # DTOs запросов и ответов
│   ├── appsettings.json
│   └── Program.cs
│
│
├── ai-service/                        # 🧠 Custom AI Engine (.NET 9/10 + ONNX Runtime)
│   ├── Inference/                     # C# движки запускa нейросетей
│   │   ├── WhisperOnnxEngine.cs       # STT: Преобразование аудио в текст
│   │   ├── SlmNlpEngine.cs            # NLP: Кастомная SLM (выделение тезисов, спикеров, Action Items)
│   │   ├── TokenizerService.cs        # Токенизатор текста (Microsoft.ML.Tokenizers)
│   │   └── AudioPreprocessor.cs       # Ресемплинг аудио до 16kHz
│   │
│   ├── Pipelines/                     # Пайплайны обработки
│   │   ├── SttPipeline.cs             # Потоковая транскрибация
│   │   └── SummaryPipeline.cs         # Формирование итогового структурированного JSON для бэкенда
│   │
│   ├── Services/                      # Фоновые воркеры
│   │   ├── AudioQueueConsumer.cs      # Обработчик очереди задач из бэкенда
│   │   └── EmbeddingGenerator.cs      # Генератор векторных представлений текста
│   │
│   ├── Models/                        # Веса дообученных моделей (ONNX)
│   │   ├── whisper-small-ru.onnx
│   │   └── custom-slm-summary.onnx
│   │
│   ├── Training/                      # Локальные Python-скрипты для дообучения моделей
│   │   ├── fine_tune_slm.py           # LoRA файнтюнинг вашей модели
│   │   └── export_to_onnx.py          # Экспорт весов в ONNX FP16
│   │
│   ├── Program.cs                     # Worker Service (.NET)
│   └── ai-service.csproj
│
└── README.md
```
