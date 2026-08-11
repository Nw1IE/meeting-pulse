# meeting-pulse
meeting-pulse/
├── extension/                 # Браузерное расширение (Frontend)
│   ├── src/
│   │   ├── popup/             # React + Vite интерфейс (окно расширения)
│   │   ├── background/        # Service Worker (авторизация, запросы)
│   │   └── content/           # Content Scripts (перехват DOM Meet/Телемоста, аудио)
│   ├── manifest.json          # Манифест расширения (Manifest V3)
│   └── package.json
│
├── backend/                   # Серверная часть (ASP.NET Core / C#)
│   ├── Controllers/           # API-эндпоинты (прием аудио, выдача итогов)
│   ├── Services/              # Бизнес-логика (очереди задач, рассылка)
│   ├── Data/                  # EF Core (Context, миграции PostgreSQL)
│   └── Program.cs
│
├── ai-service/                # Кастомный ИИ-модуль (Python / FastAPI)
│   ├── stt_module/            # Модуль распознавания речи (своя/дообученная модель)
│   ├── nlp_module/            # Модуль саммаризации и выделения Action Items
│   └── main.py                # API-мост между бэкендом и нейросетками
│
└── README.md                  # Описание проекта
