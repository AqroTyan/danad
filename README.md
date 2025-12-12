# 1. Инициализируем репозиторий Git (если ещё не инициализирован)
git init

# 2. Создаем README.md с названием проекта
echo "# danad" >> README.md

# 3. Добавляем все файлы в индекс Git
git add .

# 4. Делаем первый коммит
git commit -m "Добавил сайт и README"

# 5. Связываем локальный репозиторий с GitHub
git remote add origin https://github.com/AqroTyan/danad.git

# 6. Переименовываем ветку в main
git branch -M main

# 7. Отправляем все файлы на GitHub
git push -u origin main
