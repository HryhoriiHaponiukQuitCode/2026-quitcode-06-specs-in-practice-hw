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
Якщо PowerShell відповідає «running scripts is disabled on this system», пишіть `npx.cmd`, `npm.cmd`,
`openspec.cmd` замість `npx`, `npm`, `openspec`.

**Бюджет треку: 7–8 годин за тиждень** (сума кроків нижче, без бонусу). Найбільше часу йде на перший зріз (кроки 5–6) і цикл уточнень (крок 8).

Дні календарні (з вихідними); «кінець дня N» — 23:59 за Києвом N-го дня після заняття (день 0 — день заняття).

| Крок | Коли | Бюджет | Трек |
|---|---|---|---|
| 0. Підготовка машини й акаунтів | до заняття | 20–30 хв | — |
| 1. Upstream із шаблону | день 0, одразу після заняття | 10 хв | A |
| 2. Ruleset на `main` | день 0 | 10 хв | A |
| 3. Мітка, denylist (локальний файл і секрет), заступник, CodeRabbit | день 0 | 10 хв | A |
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

Цей крок робимо **одразу після заняття** (день 0): створіть upstream і надішліть його URL у чат команди — розробники
форкають його того ж дня.

Команди нижче може виконати агент — давайте їх по одній і підтверджуйте кожен push.

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
  approvals — `0`; Allowed merge methods — лише **Merge**, точну назву поля звіряємо на репетиції),
  **Require status checks to pass** → Add checks → `spec`, `app` → Create.

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

## 3. Мітка, denylist (локальний файл і секрет), заступник, CodeRabbit (10 хв) — трек A

**Через браузер:** мітка — Issues → Labels → **New label** → назва `spec-question`, опис «Питання до
специфікації» → Create label; секрет — Settings → Secrets and variables → Actions → **New repository secret** →
Name `SPEC_DENYLIST`, Secret — назви через кому → Add secret.

Або в терміналі:

```bash
gh label create spec-question --repo <pm>/<team-slug> --description "Питання до специфікації"
gh secret set SPEC_DENYLIST --repo <pm>/<team-slug>
```

- **Мітку створіть першого ж дня.** Форма issue «Питання до специфікації» ставить лише наявну мітку.
- **Справжні назви клієнта — лише в локальному `product/denylist.local.txt` (не в git) і секреті
  `SPEC_DENYLIST`.** Репозиторій публічний, тому справжніх назв немає ні в `product/denylist.txt`, ні в будь-якому
  іншому файлі, issue чи PR.
  - **Локальний файл.** У клоні upstream створіть у редакторі `product/denylist.local.txt`: по одній назві на
    рядок, `#` — коментар. Шаблон уже має цей шлях у `.gitignore`. Перевірка — у виводі першої команди має
    бути рядок `!! product/denylist.local.txt` (`!!` — git ігнорує файл) і жодного рядка з `??`, а друга має показати
    «слів у denylist: N», де N — не менше кількості ваших назв:

    ```bash
    git status --short --ignored product
    npm run spec:check
    ```

    Бачите `denylist.local.txt.txt`? Провідник Windows приховав розширення, і до імені додалось друге `.txt`:
    увімкніть Провідник → Вигляд → Показати → Розширення імен файлів і перейменуйте файл на `denylist.local.txt`.

    Тепер `npm run spec:check` на вашій машині ловить назви ще до push, але значень не друкує (лише «слово #N із
    product/denylist.local.txt»).
  - **Секрет для CI.** `gh secret set` попросить значення: ті самі назви через кому. Значення ніде не друкується,
    а CI підставляє його в `npm run spec:check` (job `spec`). Але CI у PR із форків (пропозиції змін спеки від
    розробників) секретів не отримує: там `spec:check` бачить лише `product/denylist.txt`. Перед merge такого PR
    перевірте його локально: `gh pr checkout <номер>` → `npm run spec:check`. Якщо в такому PR замість
    перевірок `spec` і `app` GitHub показує кнопку **Approve and run workflows** (перший внесок розробника в ваш
    репозиторій), натисніть її — інакше обов'язкові перевірки чекатимуть безкінечно (точний текст кнопки й
    налаштування за замовчуванням звіряємо на репетиції).
  - **Список для розробників.** Надішліть ті самі назви кожному розробнику команди **особистим повідомленням**
    (не в загальний чат і не в issue): вони збережуть їх у `product/denylist.local.txt` свого клону, і
    `npm run spec:check` у форках ловитиме назви в документації, коді, сидах і тестах ще до push.
- **Заступник.** Settings → Collaborators → Add people: викладач і один розробник команди (за домовленістю в
  команді); на особистому репозиторії вони отримують доступ на запис. Якщо ви хворієте чи не встигаєте, заступник
  зливає PR спеки й ставить тег `spec-vX.Y` за тим самим порядком релізу; ruleset однаково вимагає PR і зелений CI.
  Бали PM — за зроблене вами. Якщо наприкінці дня 3 `spec-v0.1` немає, викладач переглядає ваш відкритий PR і
  випускає `spec-v0.1` сам ([README](../README.md#що-знімає-бали-pm)).
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
| `product/denylist.txt` | внутрішні слова, яких не має бути ні в спеках, ні в коді й документації (кодові назви, заглушки з чужих шаблонів на кшталт Contoso); назви сторонніх систем із брифу (QuickBooks, Buildium) — дозволені | справжніх назв клієнта тут **немає** — вони в локальному `product/denylist.local.txt` (не в git) і секреті `SPEC_DENYLIST` (крок 3) |
| `openspec/config.yaml` | рядок `Продукт: …` у `context` — назва й одне речення | правила `rules:` не чіпайте; якщо змінюєте — кожен пункт у лапках ([пастки](spec-guide.md#10-пастки-формату-openspec-1141)) |

Перевірте й відкрийте PR:

```bash
npm run spec:check
git status --short --ignored product      # denylist.local.txt — лише з !!, жодного ?? поруч
git add product openspec/config.yaml
git commit -m "Fill product context: vision, glossary, roles, roadmap"
git push -u origin product/context
gh pr create --web
```

**З агентом:** «Зроби крок 4: `npm run spec:check`, commit `product/` і `openspec/config.yaml` у гілку
`product/context`, push і PR» — підтверджуйте кожну команду.

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

`/opsx:propose` (у Cursor `/opsx-propose`) за своєю інструкцією генерує всі артефакти зміни, зокрема `design.md` і
`tasks.md` (як реалізувати). У зміні PM їх не лишаємо — це робота розробника у форку: додайте до запиту «пиши лише
proposal.md і specs/, без design.md і tasks.md», а якщо агент їх усе ж створив — видаліть обидва файли до коміту.

**Вручну:** `npx openspec new change add-<зріз>`, далі файли за [скелетом](spec-guide.md#скелет-зміни).

Що перевірити перед кроком 6:

- у теці зміни лише `.openspec.yaml`, `proposal.md` і `specs/` — без `design.md` і `tasks.md`;
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

Не працювали з гілками й тегами? Попросіть агента: «Виконай реліз spec-v0.1 за README, «Шлях PM», крок 5; перед
кожним git push, створенням PR і merge питай мене». Агент питає перед push — так задано в AGENTS.md («Спитай
спершу»). Merge — лише «Create a merge commit».

Порядок релізу (так само для `spec-v0.2`): гілка зміни → PR → **у гілці** archive і запис у `CHANGELOG.md` →
merge (**Create a merge commit**) → `git switch main` і `git pull` → `git tag spec-vX.Y` →
`git push origin spec-vX.Y`.

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

   **З агентом:** «Зроби крок 7.3: archive add-<зріз>, запис spec-v0.1 у CHANGELOG за форматом, commit і push у
   цю гілку» — підтверджуйте кожну команду.

4. CI зелений → злийте PR кнопкою **Create a merge commit** → тег на `main`:

   ```bash
   git switch main
   git pull
   git tag spec-v0.1
   git push origin spec-v0.1
   ```

   **Або через браузер (без терміналу):** Code → Releases → **Draft a new release** → Choose a tag → введіть
   `spec-v0.1` → **Create new tag on publish** → Target — `main` → Release title `spec-v0.1`, в описі — запис із
   `CHANGELOG.md` → **Publish release** (точні назви кнопок звіряємо на репетиції).

   **Перевірка тегу (обидва способи):** відкрийте `https://github.com/<pm>/<team-slug>/tree/spec-v0.1/openspec/specs`
   — там тека вашої можливості з `spec.md`, а в `openspec/changes/` немає теки `add-<зріз>` (лише `archive/`).
   Якщо не так, тег стоїть не на тому коміті. Тег `spec-vX.Y` після push не переносимо й не видаляємо: виправте
   `main` через PR і випустіть тег-виправлення `spec-vX.Y.1` (наприклад, `spec-v0.1.1`) за тим самим порядком релізу.

5. Напишіть у чат команди: «`spec-v0.1` готовий, питання — issues `spec-question` в upstream».

**Доказ:** тег `spec-v0.1` на `https://github.com/<pm>/<team-slug>/tags`, PR зміни з розділом `spec-review`.
Після archive в upstream `npm run ac:coverage` падає з `NO-ROW`. Для upstream так і має бути: матрицю ведуть
розробники у форках.

## 8. Цикл уточнень і `spec-v0.2` (1,5–2 год за дні 2–4) — трек C

Розробники відкривають issues за формою «Питання до специфікації»: варіанти A–E, рекомендований, вплив.

1. **Відповідайте протягом доби** коментарем в issue: який варіант і чому, одним-двома реченнями. Розробник без
   відповіді робить робоче припущення, а п'ять різних припущень — п'ять різних реалізацій. **Дублікати:** дубль
   PM закриває коментарем «Дубль #N — відповідь там» і Close as duplicate — це теж відповідь; у `decisions.md` —
   один рядок на питання з усіма номерами (#3, #7, #9).
2. **Записуйте** кожне рішення в `product/decisions.md`: питання (з `#N`) → відповідь → що змінилось у специфікації →
   версія. Зразок — [`examples/leave-portal/2-clarify/decisions.md`](../examples/leave-portal/2-clarify/decisions.md).
   Відповіді на ваші власні Open questions — сюди ж. Прямі коміти в `main` ruleset не пустить, тому
   `decisions.md` комітьте в гілку `spec/clarify-<зріз>` з пункту 3, разом зі зміною. Якщо жодна відповідь не
   змінює поведінку, усе одно потрібна гілка (наприклад, `product/decisions`) і PR.
3. **Якщо відповідь змінює поведінку**, створіть нову зміну в окремій гілці:

   ```bash
   git switch main
   git pull
   git switch -c spec/clarify-<зріз>
   npx openspec new change clarify-<зріз>     # або /opsx:propose у Claude Code (/opsx-propose у Cursor)
   ```

   `openspec new change` створює лише теку зміни з `.openspec.yaml`. Обидва файли нижче пишете ви (або агент):

   - `openspec/changes/clarify-<зріз>/proposal.md` — усі розділи, як у кроці 5: пропущений розділ —
     блокер у `spec-review`. Why — на які питання відповідає зміна (`#N`, Q1…); What Changes — що змінилось, з
     AC ID; Users & roles — що змінюється для ролей (або «ролі не змінюються» і таблиця змін); Non-goals і
     Assumptions — для цих відповідей; Open questions — що лишилось відкритим, або «—». Без `proposal.md`
     `validate` проходить, а `openspec show --diff` нижче падає з «has no proposal.md yet». Зразок —
     `openspec/changes/archive/2026-10-09-clarify-leave-rules/proposal.md` у
     [демо-репозиторії](https://github.com/koldovsky/2026-quitcode-06-leave-portal-demo);
   - `openspec/changes/clarify-<зріз>/specs/<capability>/spec.md` — дельта:
     - змінений сценарій: `## MODIFIED Requirements` з **повним** блоком вимоги й усіма сценаріями; AC ID
       незмінні;
     - нове правило: новий сценарій із **новим** AC ID (наступний номер) або `## ADDED Requirements`.

   Перед PR перегляньте diff проти поточної спеки:

   ```bash
   npx openspec show clarify-<зріз> --diff
   npx openspec validate clarify-<зріз> --strict
   npm run spec:check
   ```

   Зразок diff — [`examples/leave-portal/4-spec-v0.2/spec-v0.2-diff.txt`](../examples/leave-portal/4-spec-v0.2/spec-v0.2-diff.txt).
4. **Реліз `spec-v0.2`** — так само, як `spec-v0.1` у кроці 7, **до кінця дня 4**. `spec-review` для нової
   зміни, потім коміт і PR:

   ```bash
   git add openspec product
   git commit -m "Propose clarify-<зріз>"
   git push -u origin spec/clarify-<зріз>
   gh pr create --web
   ```

   Опис PR — тип «Зміна специфікації», розділ `## spec-review`. **У тій самій гілці, до merge:** archive і запис
   `## spec-v0.2` у `product/CHANGELOG.md` (MODIFIED, ADDED, на які питання відповідає):

   ```bash
   npx openspec archive clarify-<зріз> -y
   npm run spec:validate
   git add openspec product/CHANGELOG.md
   git commit -m "Archive clarify-<зріз> and log spec-v0.2"
   git push
   ```

   **З агентом:** «Зроби крок 8.4: archive clarify-<зріз>, запис spec-v0.2 у CHANGELOG за форматом, commit і push у
   цю гілку» — підтверджуйте кожну команду.

   CI зелений → злийте PR кнопкою **Create a merge commit** → тег на `main`, а не на гілці:

   ```bash
   git switch main
   git pull
   git tag spec-v0.2
   git push origin spec-v0.2
   ```

   Або через Releases, як у кроці 7; перевірка тегу — та сама (`…/tree/spec-v0.2/openspec/specs`).

5. Закрийте issues з посиланням на зміну. Повідомте команду: «`spec-v0.2` — синхронізуйтеся гілкою `sync/spec-v0.2`».

Питання, що надійшли після `spec-v0.2`, — відповідь в issue й запис у `decisions.md` зі статусом «наступна версія».
Ці записи з'являться вже після тегу, тож у треку C не оцінюються: оцінюємо все до `spec-v0.2` включно.

**Доказ:** відповіді в issues, `product/decisions.md`, `product/CHANGELOG.md`, тег `spec-v0.2` зі щонайменше
одним MODIFIED або ADDED.

## Оновлення шаблону (коли оголосить викладач, 10–15 хв)

Виправлення шаблону викладач оголошує в чаті курсу з SHA коміту. Не `git merge` шаблону: «Use this template»
починає нову історію, і merge дає конфлікт у кожному файлі, що відрізняється. Натомість:

```bash
git remote add template https://github.com/koldovsky/2026-quitcode-06-project-template.git   # один раз
git fetch template
git switch main
git pull
git switch -c template/<коротко>
git cherry-pick <sha>
npm ci
npm run lint
npm run typecheck
npm test
npm run spec:check
git push -u origin template/<коротко>
gh pr create --web
```

У шаблоні PR позначте тип «Оновлення шаблону» → зелений CI → **Create a merge commit**. Розробники отримають
виправлення з наступним тегом спеки (гілка `sync/spec-vX.Y`).

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

Якщо явних розбіжностей менше трьох, спершу доберіть місця, де прочитання різняться в деталях (назви тестів і
статуси в `docs/conformance.md`, робочі припущення в `docs/spec-reading.md` форків). Якщо й так менше трьох (команда
відкрила мало issues чи ще не має PR), бракуючий пункт — ваш AC, який, на вашу думку, найлегше прочитати
по-різному, з двома можливими прочитаннями й поміткою «прогноз» (без посилань). Пасивність команди балів не знімає
([README](../README.md#що-знімає-бали-pm)).

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
- **Порівняння з Spec Kit:** в окремій порожній теці (не в upstream) `pip install specify-cli==1.1.2` (Python ≥ 3.11)
  → `specify init --here --integration claude` (у Cursor — `--integration cursor-agent`) → `/speckit-specify` з
  текстом однієї вашої вимоги → `/speckit-clarify`; та сама вимога — через `spec-review`. Порівняйте, які питання
  знайшов кожен, які пропустив і який формат зручніший для вашої команди.

Звіт — у розділі «Бонус E» файлу здачі: посилання і до пів сторінки висновків.
