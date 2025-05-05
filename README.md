# BudgetTrackerApp

A modern Android app for tracking budgets, expenses, and financial goals. Built with Kotlin, MVVM architecture, Room database, and Material Design.

## Features
- User registration and login
- Add, edit, and delete categories
- Add, edit, and delete expenses (with optional photo)
- Set and track monthly budget goals
- View transaction history and filter/sort
- Dashboard with budget and expense overview
- Data stored locally using Room
- MVVM architecture with repositories and ViewModels
- Material Design UI with intuitive navigation

## Project Structure
- `data/` — Room entities, DAOs, and database
- `repository/` — Data repositories for each entity
- `ui/` — Activities, fragments, adapters, and custom views
- `viewmodels/` and `ui/viewmodel/` — ViewModel classes
- `utils/` — Utility classes (SessionManager, FileUtils, etc.)
- `res/` — Layouts, drawables, navigation, menus, and values

## Getting Started
1. Open the project in Android Studio (Giraffe or newer recommended)
2. Build the project to download dependencies
3. Run on an emulator or device (API 24+)

## Usage Guide
- Register a new user or log in
- Add categories for your expenses
- Add expenses, optionally attaching a photo
- Set monthly budget goals
- View and filter transactions
- Check the dashboard for a summary of your spending and budgets

## Architecture
- MVVM (Model-View-ViewModel)
- Room for local data persistence
- LiveData/StateFlow for UI updates
- Hilt for dependency injection (if enabled)

## Customization
- Update categories, budgets, and goals as needed
- Change currency and preferences in user profile (if implemented)

## Resources
- See `budget-tracker-guide.md` for a full implementation walkthrough
- App icons and images are in `res/drawable/`
- Layouts are in `res/layout/`

## License
This project is for educational purposes.
