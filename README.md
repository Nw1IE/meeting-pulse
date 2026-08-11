# meeting-pulse
meeting-pulse/
├── 📁 extension/                  # 🌐 Браузерное расширение (Frontend - React + Vite)
│   ├── 📁 src/
│   │   ├── 📁 popup/              # Интерфейс всплывающего окна расширения
│   │   ├── 📁 background/         # Service Worker (авторизация, фоновые запросы)
│   │   └── 📁 content/            # Content Scripts (перехват DOM Meet/Телемоста, захват аудио)
│   ├── 📄 manifest.json           # Манифест расширения (Manifest V3)
│   └── 📄 package.json            # Зависимости фронтенда
│
├── 📁 backend/                    # ⚙️ Серверная часть (.NET 8 ASP.NET Core)
│   ├── 📁 Controllers/            # API-эндпоинты (прием аудио, выдача итогов)
│   ├── 📁 Services/               # Бизнес-логика, фоновые задачи и очереди
│   ├── 📁 Data/                   # EF Core (Context, миграции PostgreSQL)
│   └── 📄 Program.cs              # Точка входа в приложение
│
├── 📁 ai-service/                 # 🧠 Кастомный ИИ-модуль (Python / FastAPI)
│   ├── 📁 stt_module/             # Модуль распознавания речи (Speech-to-Text)
│   ├── 📁 nlp_module/             # Модуль саммаризации и выделения Action Items
│   └── 📄 main.py                 # API-мост между бэкендом и нейросетками
│
└── 📄 README.md                   # Документация проекта
