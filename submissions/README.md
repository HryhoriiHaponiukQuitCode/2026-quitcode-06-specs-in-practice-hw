# Здача WS6

Здача — **один PR у цей репозиторій з одним новим файлом**:

```text
submissions/<team-slug>/<github-login>.md
```

- `<team-slug>` — `property-workflows`, `therapy-scheduling`, `school-purchasing` або `leave-portal`;
- `<github-login>` — ваш логін GitHub, точно як у профілі (той самий, з якого відкрито PR);
- вміст — за [шаблоном](_template.md), усі поля заповнені, зайва для вашої ролі частина — «—».

Здає кожен окремо: PM і кожен розробник — свій файл. Роль у файлі — `pm` або `dev`. Інших файлів PR не змінює: ні
README, ні `docs/`, ні `materials/`, ні `examples/`, ні чужих файлів у `submissions/`.

## Яке репо й SHA вказати

Кожен здає **своє** репо:

| Роль | «Ваше репо» | «Репо вимог PM» |
|---|---|---|
| `pm` | репо вимог `https://github.com/<pm>/<team-slug>-spec` | те саме |
| `dev` | своє репо `https://github.com/<dev>/<team-slug>` | репо вимог PM вашої команди |

Конвеєр перевіряє «Ваше репо» **на цьому коміті**, тому коміт має бути запушений, а SHA — повний (40 символів): коміт
`main` вашого репо на момент здачі.

```bash
git fetch origin
git rev-parse origin/main
```

Розробник ще копіює дослівно рядок «Вимоги:» із шапки `docs/conformance.md` на цьому SHA:
`Вимоги: https://github.com/<pm>/<team-slug>-spec · коміт <40 символів> · <YYYY-MM-DD>`. Його записує `/spec-sync`;
рядок `Вимоги: — (заповнює /spec-sync)` означає, що вимоги у ваше репо ще не потрапили.

## Посилання — у якому вигляді

| Що | Де | Вигляд |
|---|---|---|
| Ваше репо | — | `https://github.com/<ваш-логін>/<repo>` |
| PR | ваше репо | `https://github.com/<ваш-логін>/<repo>/pull/N` |
| Файл або тека на SHA | ваше репо | `https://github.com/<ваш-логін>/<repo>/blob/<SHA>/docs/spec-reading.md`, `…/tree/<SHA>/openspec/changes/…` |
| Питання (issue) | репо вимог PM | `https://github.com/<pm>/<team-slug>-spec/issues/N` |
| Коментар в issue | репо вимог PM | `https://github.com/<pm>/<team-slug>-spec/issues/N#issuecomment-M` |
| Рядок коду в нотатці PM | репо розробника | `https://github.com/<dev>/<team-slug>/blob/<SHA>/src/…#L10` (посилання на коміт, а не на `main`, — щоб рядок не з'їхав) |

PM посилається на PR та issues свого репо вимог, а в «Нотатці PM» — ще й на репо розробників. Розробник
посилається на PR і файли свого репо, а питання — на issues і коментарі в репо вимог PM.

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

Fork цього репозиторію курсу потрібен **лише для здачі**: у нього в учасників немає права запису, тож PR здачі йде з
вашої копії. До вашої роботи fork не має стосунку: PM працює в репо вимог, розробник — у своєму репо, обидва
створені з шаблонів через «Use this template», а не fork-ом.

PowerShell: `Copy-Item` замість `cp` (або `cp` — у PowerShell це той самий alias).

1. Відкриється сторінка нового PR з шаблоном опису. **Заголовок — `WS6: Ім'я Прізвище`.** У шаблоні заповніть
   **«Ім'я та Прізвище»** (обов'язково: GitHub-логін не завжди дозволяє вас ідентифікувати, а сертифікат
   виписується на реальне ім'я) і роль. Не додавайте `--fill`: з ним опис береться з комітів, і шаблон не
   підставляється.
2. Цей репозиторій має менше 10 зірок, тому CodeRabbit не почне сам. Напишіть у PR **окремим коментарем**
   `@coderabbitai review`.
3. CodeRabbit перевіряє лише файл здачі: шлях, заповнені поля, SHA, рядок «Вимоги:», посилання, що нічого іншого не
   змінено. Виправте зауваження й запуште в ту саму гілку.

Ім'я та прізвище — лише в описі PR, не у файлі: файли цього репозиторію публічні.
