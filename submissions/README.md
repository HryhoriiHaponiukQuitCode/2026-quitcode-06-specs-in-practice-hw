# Здача WS6

Здача — **один PR у цей репозиторій з одним новим файлом**:

```text
submissions/<team-slug>/<github-login>.md
```

- `<team-slug>` — `property-workflows`, `therapy-scheduling`, `school-purchasing` або `leave-portal`;
- `<github-login>` — ваш логін GitHub, точно як у профілі (той самий, з якого відкрито PR);
- вміст — за [шаблоном](_template.md), усі поля заповнені, зайва для вашої ролі частина — «—».

Здає кожен окремо: PM, техлід і кожен розробник — свій файл. Інших файлів PR не змінює: ні README, ні `docs/`, ні
`materials/`, ні `examples/`, ні чужих файлів у `submissions/`.

## SHA — що саме вказати

Конвеєр перевіряє репозиторій команди **на цьому коміті**, тому коміт має бути запушений, а SHA — повний
(40 символів). Для всіх ролей це коміт `main` репозиторію команди на момент здачі:

```bash
git fetch origin
git rev-parse origin/main
```

Якщо команда здає в різний час, SHA у файлах можуть відрізнятися — це нормально: кожен вказує `main`, на якому
його робота вже злита.

## Посилання — у якому вигляді

| Що | Вигляд |
|---|---|
| Репозиторій команди | `https://github.com/<pm>/<team-slug>` |
| PR | `https://github.com/<pm>/<team-slug>/pull/N` |
| Issue | `https://github.com/<pm>/<team-slug>/issues/N` |
| Коментар в issue | `https://github.com/<pm>/<team-slug>/issues/N#issuecomment-M` |
| Рев'ю PR | `https://github.com/<pm>/<team-slug>/pull/N#pullrequestreview-M` (у вкладці Conversation — посилання з часу рев'ю) |
| Тег | `https://github.com/<pm>/<team-slug>/releases/tag/spec-v0.1` або `…/tree/spec-v0.1` |

## Покроково

```bash
gh repo fork koldovsky/2026-quitcode-06-specs-in-practice-hw --clone
cd 2026-quitcode-06-specs-in-practice-hw
git switch -c ws06/<github-login>
cp submissions/_template.md submissions/<team-slug>/<github-login>.md
# заповніть файл
git add submissions/<team-slug>/<github-login>.md
git commit -m "WS6 submission: <team-slug>/<github-login>"
git push -u origin ws06/<github-login>
gh pr create --repo koldovsky/2026-quitcode-06-specs-in-practice-hw --web
```

Цей репозиторій курсу — єдине місце, де ви робите fork: у нього в учасників немає права запису, тож PR здачі йде
з вашої копії. До роботи команди це не стосується.

PowerShell: `Copy-Item` замість `cp` (або `cp` — у PowerShell це той самий alias).

1. Відкриється сторінка нового PR з шаблоном опису. **Заголовок — `WS6: Ім'я Прізвище`.** У шаблоні заповніть
   **«Ім'я та Прізвище»** (обов'язково: GitHub-логін не завжди дозволяє вас ідентифікувати, а сертифікат
   виписується на реальне ім'я) і роль. Не додавайте `--fill`: з ним опис береться з комітів, і шаблон не
   підставляється.
2. Цей репозиторій має менше 10 зірок, тому CodeRabbit не почне сам. Напишіть у PR **окремим коментарем**
   `@coderabbitai review`.
3. CodeRabbit перевіряє лише файл здачі: шлях, заповнені поля, SHA, посилання, що нічого іншого не змінено.
   Виправте зауваження й запуште в ту саму гілку.

Ім'я та прізвище — лише в описі PR, не у файлі: файли цього репозиторію публічні.
