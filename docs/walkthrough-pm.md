# Покрокова інструкція: трек PM

Ви створюєте **upstream** команди з шаблону й ведете в ньому продукт і специфікацію. Розробники форкають ваш
upstream і реалізують кожен своє. Ви не пишете код і не потребуєте Supabase чи Vercel. Ваш продукт — `product/`,
`openspec/` і теги `spec-vX.Y`.

- Рубрика й Definition of Done — у [README](../README.md#трек-pm--100--5-балів).
- Як писати вимоги й AC — [`docs/spec-guide.md`](spec-guide.md).
- Бриф вашої команди — [`materials/briefs/`](../materials/briefs/).
- Зразок формату — [`examples/leave-portal/`](../examples/leave-portal/).

Позначення: `<pm>` — ваш GitHub-логін, `<team-slug>` — слаг команди (`property-workflows`, `therapy-scheduling`,
`school-purchasing` або `leave-portal`). Команди нижче — для Git Bash на Windows і терміналу macOS. Де PowerShell
відрізняється, це сказано окремо. У Windows PowerShell 5.1 немає `&&`: виконуйте команди по черзі.

**Бюджет треку: 6–7 годин за тиждень** (без бонусу). Найбільше часу йде на перший зріз (крок 6) і цикл уточнень (крок 8).

| Крок | Коли | Бюджет | Трек |
|---|---|---|---|
| 0. Підготовка машини й акаунтів | до заняття | 20–30 хв | — |
| 1. Upstream із шаблону | день 0, на занятті | 10 хв | A |
| 2. Ruleset на `main` | день 0 | 10 хв | A |
| 3. Мітка, секрет denylist, CodeRabbit | день 0 | 10 хв | A |
| 4. Продуктовий контекст | дні 0–1 | 1–1,5 год | A |
| 5. Перший зріз: чернетка зміни | дні 1–2 | 1,5–2 год | B |
| 6. Перевірка: `validate`, `spec:check`, `spec-review` | день 2 | 45 хв | B |
| 7. Реліз `spec-v0.1` | до кінця дня 2 | 20 хв | B |
| 8. Цикл уточнень і `spec-v0.2` | дні 2–4 | 1,5–2 год | C |
| 9. Нотатка PM | дні 6–7 | 30 хв | D |
| 10. Здача | день 7 | 15 хв | — |

## 0. Підготовка (20–30 хв)

1. Акаунт GitHub. Якщо в браузері кілька акаунтів, працюйте з особистого, не з клієнтського.
2. Node 24: `node -v` → `v24.…`. Windows — через nvm-windows або fnm, macOS — через fnm, nvm або Homebrew.
3. Git і GitHub CLI: `gh --version`, потім `gh auth login` (браузерний вхід). На Windows без прав адміністратора:
   `winget install --id GitHub.cli --scope user --silent`, після цього перезапустіть термінал.
4. OpenSpec CLI глобально, тієї ж версії, що в шаблоні. Його викликають команди `/opsx:*` агента:

   ```bash
   npm i -g @fission-ai/openspec@1.14.1
   openspec --version          # 1.14.1
   openspec config set telemetry.enabled false
   ```

5. Claude Code або Cursor. Обидва бачать команди `/opsx:*` (у Cursor — `/opsx-*`) і скіли шаблону.

**Доказ:** нічого не комітимо. Якщо `openspec --version` показує іншу версію, `/opsx:*` поводитимуться інакше, ніж на занятті.

## 1. Upstream із шаблону (10 хв) — трек A

Через браузер: <https://github.com/koldovsky/2026-quitcode-06-project-template> → **Use this template** →
**Create a new repository** → Owner — ваш акаунт, Repository name — `<team-slug>`, **Public** → Create.

Або однією командою (з теки, де тримаєте проєкти):

```bash
gh repo create <team-slug> --public --template koldovsky/2026-quitcode-06-project-template --clone
cd <team-slug>
npm ci
```

Репозиторій має бути **публічним**: на GitHub Free ruleset у приватних репозиторіях не діє, а розробникам
потрібен доступ до тегів для синхронізації.

**Доказ:** URL upstream — для файлу здачі. Перший прогін CI (Actions → CI) — зелені `spec` і `app`. Якщо прогону
немає, Actions → CI → **Run workflow** або `gh workflow run ci.yml`.

## 2. Ruleset на `main` (10 хв) — трек A

Ruleset забороняє прямі коміти в `main`: усе йде через PR із зеленим CI. Тому archive і запис у CHANGELOG ви
робите в гілці зміни, ще до merge (крок 7).

**Через браузер:** Settings → Rules → Rulesets → New ruleset → New branch ruleset:

- Ruleset name — `main: PR + CI`, Enforcement status — **Active**;
- Target branches → Add target → **Include default branch**;
- позначте **Restrict deletions**, **Block force pushes**, **Require a pull request before merging** (Required
  approvals — `0`), **Require status checks to pass** → Add checks → `spec`, `app` → Create.

Якщо `spec` і `app` не з'являються в пошуку, дочекайтеся першого прогону CI (крок 1) і спробуйте ще раз.

**Або через `gh api`** (готовий JSON — [`materials/rulesets/upstream-main.json`](../materials/rulesets/upstream-main.json);
працює з GitHub REST API rulesets, але на ваших акаунтах ми ще перевіримо його на репетиції):

```bash
curl -sLo ruleset.json https://raw.githubusercontent.com/koldovsky/2026-quitcode-06-specs-in-practice-hw/main/materials/rulesets/upstream-main.json
gh api repos/<pm>/<team-slug>/rulesets --method POST --input ruleset.json
rm ruleset.json
```

PowerShell: `curl.exe` замість `curl` (у Windows PowerShell 5.1 `curl` — це інша команда), `Remove-Item ruleset.json` замість `rm`.

**Доказ:** знімок екрана сторінки ruleset або вивід `gh api repos/<pm>/<team-slug>/rulesets`.

## 3. Мітка, секрет denylist, CodeRabbit (10 хв) — трек A

```bash
gh label create spec-question --repo <pm>/<team-slug> --description "Питання до специфікації"
gh secret set SPEC_DENYLIST --repo <pm>/<team-slug>
```

- **Мітку створіть першого ж дня.** Форма issue «Питання до специфікації» ставить лише наявну мітку.
- `gh secret set` попросить значення: впишіть **справжні** назви клієнта через кому. Значення ніде не друкується, а
  CI підставляє його в `npm run spec:check`. Репозиторій публічний, тому справжніх назв немає ні в
  `product/denylist.txt`, ні в будь-якому іншому файлі, issue чи PR.
- CodeRabbit: <https://github.com/apps/coderabbitai> → Install → ваш акаунт → Only select repositories →
  `<team-slug>`. У публічних репозиторіях із < 10 зірок рев'ю не запускається само: після створення PR напишіть
  окремим **коментарем** `@coderabbitai review`.
- Надішліть у чат команди посилання на upstream: розробники форкають його вже сьогодні, бо бутстрап від спеки
  не залежить.

## 4. Продуктовий контекст (1–1,5 год) — трек A

```bash
git switch -c product/context
```

Заповніть файли за коментарями в них:

| Файл | Що має бути | На що звернути увагу |
|---|---|---|
| `product/vision.md` | проблема, для кого, результат (2–4 вимірні ознаки), межі продукту | одна сторінка; розділ «Обмеження навчального проєкту» лишіть |
| `product/glossary.md` | 8–15 термінів; колонка «Не вживаємо» | кожен іменник зі сценаріїв першого зрізу має бути тут |
| `product/roles.md` | ролі зі значеннями `profiles.role` і **матриця роль × сутність × дія** | заповнені клітинки або «—», без порожніх; ролі шаблону `member`/`admin` замініть своїми |
| `product/roadmap.md` | зріз на кожне заняття WS6–WS11 | WS6 — перший зріз; далі — з брифу («поза зрізом») |
| `product/denylist.txt` | внутрішні слова, яких не має бути в спеках (кодові назви, назви чужих систем) | справжніх назв клієнта тут **немає** — вони в секреті |
| `openspec/config.yaml` | рядок `Продукт: …` у `context` — назва й одне речення | правила `rules:` не чіпайте; якщо змінюєте — кожен пункт у лапках ([пастки](spec-guide.md#10-пастки-формату-openspec-1141)) |

Перевірте й відкрийте PR:

```bash
npm run spec:check
git add product openspec/config.yaml
git commit -m "Fill product context: vision, glossary, roles, roadmap"
git push -u origin product/context
gh pr create --web
```

У шаблоні PR позначте тип «Зміна специфікації». Після зеленого CI злийте PR.

**Доказ:** злитий PR (посилання — у файл здачі).

## 5. Перший зріз: чернетка зміни (1,5–2 год) — трек B

Зріз — з брифу («Рекомендований перший зріз»), але з вашими формулюваннями й рішеннями. Орієнтир: 3–5 вимог,
6–10 AC.

```bash
git switch main
git pull
git switch -c spec/add-<зріз>
```

**З агентом** (Claude Code): `/opsx:propose <опис зрізу одним-двома реченнями>` (у Cursor — `/opsx-propose`). Агент
поставить вам питання й напише `openspec/changes/add-<зріз>/proposal.md` і `specs/<capability>/spec.md`. Формат він
бере з `openspec/config.yaml`. Ви відповідаєте на питання, а потім **читаєте й правите кожен рядок**: агент
охоче вигадує правила, яких ви не казали, і пише «тощо».

**Вручну:** `npx openspec new change add-<зріз>`, далі файли за [скелетом](spec-guide.md#скелет-зміни).

Що перевірити перед кроком 6:

- `proposal.md`: Why, What Changes з AC ID, Capabilities, Users & roles (таблиця), Non-goals, Assumptions,
  Open questions;
- кожна вимога — EARS з `SHALL`; кожен сценарій — `#### Scenario: AC-<AREA>-<NN> …`, один критерій, числа й дати;
- жодних таблиць, API, компонентів; дані синтетичні.

## 6. Перевірка (45 хв) — трек B

```bash
npx openspec validate add-<зріз> --strict
npm run spec:validate
npm run spec:check
```

Потім у Claude Code або Cursor: «Запусти скіл spec-review для зміни add-<зріз>». Скіл запускає ті самі перевірки,
проходить [чекліст якості](spec-guide.md#8-чекліст-перед-релізом) і таксономію
неоднозначностей і видає таблицю знахідок (БЛОКЕР / ВАЖЛИВО / ДРІБНИЦЯ) та «питання, які розробники, найімовірніше,
поставлять». Правки він вносить лише після вашого «так».

**Доказ (обов'язковий для треку B):** у PR зміни (крок 7) розділ опису `## spec-review` — таблиця знахідок і що
ви виправили, а що свідомо лишили (і чому). Без цього розділу — мінус бали за B.

## 7. Реліз `spec-v0.1` (20 хв) — трек B, до кінця дня 2

```bash
git add openspec
git commit -m "Propose add-<зріз>"
git push -u origin spec/add-<зріз>
gh pr create --web
```

1. Опис PR: тип «Зміна специфікації», розділ `## spec-review`. Окремим коментарем — `@coderabbitai review`.
2. Дайте команді кілька годин на коментарі до PR. Запитання розробників на цьому етапі — подарунок: відповідь
   до релізу дешевша за `spec-v0.2`.
3. **У тій самій гілці, до merge:** archive і запис у журнал.

   ```bash
   npx openspec archive add-<зріз> -y
   ```

   Додайте в `product/CHANGELOG.md` запис `## spec-v0.1 — <дата>` за форматом із коментаря у файлі (зразок —
   [`examples/leave-portal/4-spec-v0.2/CHANGELOG.md`](../examples/leave-portal/4-spec-v0.2/CHANGELOG.md)).

   ```bash
   npm run spec:validate
   git add openspec product/CHANGELOG.md
   git commit -m "Archive add-<зріз> and log spec-v0.1"
   git push
   ```

4. CI зелений → **Merge** → тег на `main`:

   ```bash
   git switch main
   git pull
   git tag spec-v0.1
   git push origin spec-v0.1
   ```

5. Напишіть у чат команди: «`spec-v0.1` готовий, питання — issues `spec-question` в upstream».

**Доказ:** тег `spec-v0.1` на `https://github.com/<pm>/<team-slug>/tags`, PR зміни з розділом `spec-review`.
Після archive в upstream `npm run ac:coverage` падає з `NO-ROW`. Для upstream так і має бути: матрицю ведуть
розробники у форках.

## 8. Цикл уточнень і `spec-v0.2` (1,5–2 год за дні 2–4) — трек C

Розробники відкривають issues за формою «Питання до специфікації»: варіанти A–E, рекомендований, вплив.

1. **Відповідайте протягом доби** коментарем в issue: який варіант і чому, одним-двома реченнями. Розробник без
   відповіді робить робоче припущення, а п'ять різних припущень — п'ять різних реалізацій.
2. **Записуйте** кожне рішення в `product/decisions.md`: питання (з `#N`) → відповідь → що змінилось у специфікації →
   версія. Зразок — [`examples/leave-portal/2-clarify/decisions.md`](../examples/leave-portal/2-clarify/decisions.md).
   Відповіді на ваші власні Open questions — сюди ж.
3. **Якщо відповідь змінює поведінку**, створіть нову зміну в гілці `spec/clarify-<зріз>`:
   - змінений сценарій: `## MODIFIED Requirements` з **повним** блоком вимоги й усіма сценаріями; AC ID
     незмінні;
   - нове правило: новий сценарій із **новим** AC ID (наступний номер) або `## ADDED Requirements`;
   - перед PR перегляньте diff проти поточної спеки:

     ```bash
     npx openspec show clarify-<зріз> --diff
     npx openspec validate clarify-<зріз> --strict
     npm run spec:check
     ```

     Зразок diff — [`examples/leave-portal/4-spec-v0.2/spec-v0.2-diff.txt`](../examples/leave-portal/4-spec-v0.2/spec-v0.2-diff.txt).
4. `spec-review` для нової зміни → PR → archive у гілці → запис `## spec-v0.2` у `product/CHANGELOG.md` (MODIFIED,
   ADDED, на які питання відповідає) → merge → `git tag spec-v0.2` → `git push origin spec-v0.2` **до кінця дня 4**.
5. Закрийте issues з посиланням на зміну. Повідомте команду: «`spec-v0.2` — синхронізуйтеся гілкою `sync/spec-v0.2`».

Питання, що надійшли після `spec-v0.2`, — відповідь в issue й запис у `decisions.md` зі статусом «наступна версія».

**Доказ:** відповіді в issues, `product/decisions.md`, `product/CHANGELOG.md`, тег `spec-v0.2` зі щонайменше
одним MODIFIED або ADDED.

## 9. Нотатка PM (30 хв) — трек D

Це заготовка для фінальної презентації на demo day: де одна й та сама спека прочиталась по-різному. Джерела:
issues `spec-question`, `docs/spec-reading.md` у форках, перші PR розробників (назви тестів, рядки
`docs/conformance.md`).

Пишіть у розділ «Нотатка PM» свого файлу здачі ([шаблон](../submissions/_template.md)). Щонайменше три
розбіжності, кожна так:

- **AC / вимога:** `AC-…-02` — «…» (цитата з `spec-v0.1`);
- **як прочитали:** щонайменше два різні прочитання з посиланнями (issue `<pm>/<team-slug>#N`, PR чи рядок у форку
  `<dev>/<team-slug>`) — логіни, не імена;
- **чому:** яке слово чи пропуск у тексті спеки це дозволив (категорія з таксономії);
- **що зробив PM:** відповідь, MODIFIED / ADDED у `spec-v0.2` або «залишив і чому».

Якщо розбіжностей не знайшлося, напишіть, які AC прочитали однаково і що, на вашу думку, цьому допомогло.

## 10. Здача (15 хв)

Інструкція — [`submissions/README.md`](../submissions/README.md). Коротко: PR у цей репозиторій із файлом
`submissions/<team-slug>/<github-login>.md`, заголовок `WS6: Ім'я Прізвище`. SHA для PM — коміт, на який
вказує тег `spec-v0.2`:

```bash
git fetch origin --tags
git rev-list -n 1 spec-v0.2
```

## Бонус E (+5, ~1 год) — одне з двох

- **Друга OpenSpec-зміна** — наступний зріз з `roadmap.md` (`add-<зріз-2>`), відкритий PR в upstream зі
  зеленим CI і розділом `spec-review`. Не архівуйте й не тегуйте її до WS7, якщо команда ще не закінчила перший зріз.
- **Порівняння з Spec Kit:** проженіть одну свою вимогу через `/speckit-clarify` (GitHub Spec Kit; `pip install
  specify-cli` — Python ≥ 3.11, ставте окремо від upstream) і через `spec-review`. Порівняйте, які питання знайшов
  кожен, які пропустив і який формат зручніший для вашої команди.

Звіт — у розділі «Бонус E» файлу здачі: посилання і до пів сторінки висновків.
