# Покрокова інструкція: трек розробника

Ви форкаєте upstream свого PM і реалізуєте його специфікацію **по-своєму**: свій форк, свій проєкт Supabase,
свій проєкт Vercel. Специфікацію (`product/`, `openspec/specs/`) у форку не змінюєте. Питання до неї ставите
issue в upstream, нову версію забираєте гілкою `sync/spec-vX.Y`.

- Рубрика й Definition of Done — у [README](../README.md#трек-розробника--100--5-балів).
- Зразок формату артефактів — [`examples/leave-portal/`](../examples/leave-portal/) (не відповідь).
- Як PM пише спеку (і як її читати) — [`docs/spec-guide.md`](spec-guide.md).

Позначення: `<я>` — ваш GitHub-логін, `<pm>` — логін PM, `<team-slug>` — назва upstream (слаг команди),
`<ref>` — Project ref проєкту Supabase, `<зріз>` — короткий слаг першого зрізу з `proposal.md` PM.
Команди — для Git Bash на Windows і терміналу macOS. Де PowerShell відрізняється, це сказано окремо. У Windows
PowerShell 5.1 немає `&&`: виконуйте команди по черзі.

**Бутстрап не чекає на спеку.** Кроки 1–9 від PM не залежать — починайте в день 0.

**Бюджет треку: 9,5–10,5 годин за тиждень** (сума кроків нижче, без бонусу). Бутстрап (кроки 0–9) — ~3 год, реалізація — 4–5 год.

| Крок | Коли | Бюджет | Трек |
|---|---|---|---|
| 0. Підготовка машини й акаунтів | до заняття | 30 хв | — |
| 1. Форк, Actions, клон, `set-default` | день 0 | 15 хв | A |
| 2. Ruleset на `main` | день 0 | 10 хв | A |
| 3. Залежності й локальні перевірки | день 0 | 15 хв | A |
| 4. Supabase: проєкт, Auth, link, міграції, користувачі | дні 0–1 | 40 хв | A |
| 5. Vercel: імпорт, змінні, захист preview, редеплой | дні 0–1 | 30 хв | A |
| 6. Supabase Auth: Site URL і Redirect URLs | дні 0–1 | 5 хв | A |
| 7. Секрети CI для job `db` | дні 0–1 | 10 хв | A |
| 8. CodeRabbit | дні 0–1 | 5 хв | A |
| 9. Перший PR: `docs/bootstrap.md`, зелений CI, preview | день 1 | 30 хв | A |
| 10. Синхронізація `spec-v0.1` | день 2, щойно є тег | 20 хв | B |
| 11. Читання спеки: `spec-clarify`, issues, журнал | дні 2–3 | 1 год | B |
| 12. Реалізація зрізу: `impl-<зріз>`, PR-и, матриця | дні 3–7 | 4–5 год | C |
| 13. Синхронізація `spec-v0.2` | дні 5–7 | 45 хв | D |
| 14. Здача | день 7 | 15 хв | — |

## 0. Підготовка (30 хв)

1. **Акаунти:** GitHub (особистий), [Supabase](https://supabase.com/dashboard) (власна організація),
   [Vercel](https://vercel.com) (Hobby, вхід через GitHub). Нічого клієнтського.
2. **Вільний слот Supabase.** На Free активних може бути лише **2** проєкти на всі організації, де ви Owner або
   Admin. Проєкти з WS5 поставте на паузу або видаліть.
3. **Node 24:** `node -v` → `v24.…`. Git (на Windows — з Git Bash), GitHub CLI: `gh --version` → `gh auth login`.
   На Windows без прав адміністратора: `winget install --id GitHub.cli --scope user --silent`, потім перезапустіть термінал.
4. **OpenSpec глобально** тієї ж версії, що в шаблоні. Його викликають `/opsx:*` і скіли агента:

   ```bash
   npm i -g @fission-ai/openspec@1.14.1
   openspec --version          # 1.14.1
   openspec config set telemetry.enabled false
   ```

5. Claude Code або Cursor. Docker **не потрібен**: працюємо з хмарним проєктом Supabase через міграції.

## 1. Форк, Actions, клон, `set-default` (15 хв) — трек A

```bash
gh repo fork <pm>/<team-slug> --clone
cd <team-slug>
gh repo set-default <я>/<team-slug>
gh repo set-default --view          # має показати <я>/<team-slug>
git remote -v                       # origin → ваш форк, upstream → репозиторій PM
```

- `gh repo fork --clone` сам додає remote `upstream`, але **ставить upstream репозиторієм за замовчуванням**. Без
  `set-default` команди `gh pr create` і `gh workflow run` полізуть у репозиторій PM. `gh secret set` завжди
  пишемо з `--repo <я>/<team-slug>` (крок 7): на секрети `set-default` не діє. Через браузер (Fork на
  сторінці upstream) — те саме, тоді клонуйте свій форк і додайте `git remote add upstream https://github.com/<pm>/<team-slug>.git`.
- **Actions у форку вимкнено за замовчуванням.** Вкладка Actions вашого форку → **I understand my workflows, go
  ahead and enable them**. Потім перший прогін: Actions → CI → **Run workflow** або `gh workflow run ci.yml`.
  Очікувано зелені `spec`, `app`, `conformance`. `db` пропущено, бо це не push у `main`.

**Доказ:** посилання на перший run і вивід `gh repo set-default --view` — у `docs/bootstrap.md` (крок 9).

## 2. Ruleset на `main` (10 хв) — трек A

Через браузер: Settings → Rules → Rulesets → New ruleset → New branch ruleset → назва `main: PR + CI`,
Enforcement status **Active**, Target branches → **Include default branch**, позначте **Restrict deletions**,
**Block force pushes**, **Require a pull request before merging** (Required approvals — `0`: власний PR
схвалити не можна), **Require status checks to pass** → `spec`, `app`, `conformance` → Create. Якщо перевірок
немає в пошуку, дочекайтеся першого прогону CI з кроку 1.

Або через `gh api` (JSON — [`materials/rulesets/fork-main.json`](../materials/rulesets/fork-main.json); ми перевіримо
його на ваших акаунтах на репетиції):

```bash
curl -sLo ruleset.json https://raw.githubusercontent.com/koldovsky/2026-quitcode-06-specs-in-practice-hw/main/materials/rulesets/fork-main.json
gh api repos/<я>/<team-slug>/rulesets --method POST --input ruleset.json
rm ruleset.json
```

PowerShell: `curl.exe` замість `curl`, `Remove-Item ruleset.json` замість `rm`.

Ruleset дозволяє всі три способи злиття. **Гілки `sync/spec-vX.Y` зливайте лише кнопкою Create a merge commit**,
інакше тег upstream не потрапить в історію `main`, і наступна синхронізація дасть конфлікти.

## 3. Залежності й локальні перевірки (15 хв) — трек A

```bash
npm ci
npm run lint
npm run typecheck
npm test
```

Попередження `npm warn deprecated …` / `allow-scripts` під час `npm ci` очікувані. `npm audit fix` і оновлення
пакетів до latest не запускайте: шаблон перевірено з TypeScript ^5 і ESLint ^9.

## 4. Supabase (40 хв) — трек A

1. **Проєкт:** Dashboard → New project → **ваша власна організація** (не через Vercel Marketplace: та інтеграція
   створює окрему організацію, керовану Vercel, і займає слот). Назва — `<team-slug>`, пароль бази збережіть у
   менеджері паролів, регіон — найближчий.
2. **Auth:** Authentication → Sign In / Providers → Email → вимкніть **Confirm email**. Вбудований SMTP надсилає
   2 листи на годину і лише членам команди проєкту, тож користувачів створює скрипт. Там само вимкніть **Allow
   new users to sign up**. Публічний ключ лежить у JS-бандлі, тож з увімкненою реєстрацією будь-хто зареєструється
   сам. `npm run users:demo` працює й так, бо це Admin API. Точне місце перемикача перевіримо на репетиції.
3. **Прив'язка й міграції** (без Docker):

   ```bash
   npx supabase login                       # відкриє браузер
   npx supabase link --project-ref <ref>     # спитає пароль бази
   npx supabase db push --dry-run            # покаже 20261012000000_profiles_and_roles.sql
   npm run db:push
   npm run db:types
   ```

   `<ref>` — з адреси Dashboard (`/project/<ref>`) або Project Settings → General. Якщо `db:types` впав,
   виконайте `git restore src/lib/supabase/database.types.ts`.
4. **`.env.local`** (у `.gitignore`):

   ```bash
   cp .env.example .env.local
   ```

   Заповніть за Project Settings → API Keys: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
   (`sb_publishable_…`), `SUPABASE_SECRET_KEY` (`sb_secret_…`), `DEMO_USER_PASSWORD` (свій, ≥ 8 символів).
   Значення не вставляйте в чат з агентом: `.claude/settings.json` шаблону забороняє агенту читати `.env*`.
5. **Користувачі й перший вхід:**

   ```bash
   npm run users:demo     # admin@example.test, member1@example.test, member2@example.test
   npm run dev            # http://localhost:3000 → вхід як admin@example.test
   ```

6. Для Supabase MCP з `.mcp.json` (лише читання) задайте змінну перед запуском агента:
   `export SUPABASE_PROJECT_REF=<ref>` (PowerShell: `$env:SUPABASE_PROJECT_REF = "<ref>"`).

**Правила бази (з `AGENTS.md` шаблону):** схему змінюємо лише міграціями, у Dashboard схему не редагуємо;
`supabase db reset --linked`, `migration repair`, `db push --include-all` — ніколи. CLI у сесії агента сам
підтверджує дії, тому `.claude/settings.json` питає вас перед кожним `db push`.

## 5. Vercel (30 хв) — трек A

1. Vercel → **Add New → Project** → Import Git Repository → ваш форк `<я>/<team-slug>` (дайте застосунку Vercel
   доступ лише до цього репозиторію). Framework — Next.js, Node 24.x береться з `engines` у `package.json`.
2. **Environment Variables** для **Production і Preview**: лише `NEXT_PUBLIC_SUPABASE_URL` і
   `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`. `SUPABASE_SECRET_KEY` у Vercel поки **не** додавайте: у шаблоні його
   читає лише скрипт `users:demo`, який ви запускаєте локально, а код застосунку — ні. Секретний ключ обходить RLS,
   тож він має бути лише там, де без нього не обійтись (найменші привілеї). Знадобиться вашому серверному коду —
   додайте тоді з типом **Secret** і зробіть редеплой. Ніколи в Client Components і ніколи з префіксом
   `NEXT_PUBLIC_`. **Не** підключайте Supabase через Marketplace-інтеграцію Vercel.
3. **Deploy.** Перевірте прод-URL `https://<проєкт>.vercel.app/login`.
4. Settings → **Deployment Protection** → **Vercel Authentication**: якщо увімкнено — вимкніть → Save. Інакше PM
   і викладач не відкриють ваші preview без входу у ваш акаунт Vercel (на Hobby немає учасників команди). Чи
   вмикає його Vercel для нових проєктів Hobby за замовчуванням, перевіряємо на репетиції. Перевірка — у кроці 9:
   preview відкривається у вікні інкогніто без входу.
5. **Редеплой після будь-якої зміни змінних:** Deployments → останній деплой → ⋯ → **Redeploy**. Змінні діють лише
   для нових деплоїв.

Обмеження Hobby: лише некомерційне особисте використання. Тому застосунок навчальний: вигадані дані, без
брендингу клієнта, клієнтам не передається. Cron — щонайбільше раз на добу.

## 6. Supabase Auth: Site URL і Redirect URLs (5 хв) — трек A

Supabase → Authentication → **URL Configuration**:

- **Site URL** — прод-URL Vercel (`https://<проєкт>.vercel.app`);
- **Redirect URLs** — `http://localhost:3000/**` і `https://*-<slug-акаунта-vercel>.vercel.app/**`. Slug — остання
  частина адреси будь-якого preview: `<проєкт>-<хеш>-<slug>.vercel.app`.

## 7. Секрети CI для job `db` (10 хв) — трек A

Job `db` у `.github/workflows/ci.yml` виконує `supabase db push` після кожного push у `main` вашого форку, але
лише якщо задано всі три секрети.

```bash
gh secret set SUPABASE_ACCESS_TOKEN --repo <я>/<team-slug>
gh secret set SUPABASE_DB_PASSWORD --repo <я>/<team-slug>
gh secret set SUPABASE_PROJECT_ID --repo <я>/<team-slug>     # значення — <ref>
```

`gh secret set` питає значення й ніде його не друкує. `SUPABASE_ACCESS_TOKEN` — окремий токен для CI:
Supabase → Account → Access Tokens → Generate. Якщо доступні токени з обмеженими правами, дайте лише потрібні
(перелік прав перевіримо на репетиції). Не використовуйте токен зі свого `supabase login`.

## 8. CodeRabbit (5 хв) — трек A

<https://github.com/apps/coderabbitai> → Install → ваш акаунт → Only select repositories → `<team-slug>`.
`.coderabbit.yaml` уже є в шаблоні. Репозиторії з < 10 зірок CodeRabbit не рецензує автоматично: після
створення PR напишіть **окремим коментарем** `@coderabbitai review`.

## 9. Перший PR (30 хв) — трек A

```bash
git switch -c bootstrap/evidence
```

Заповніть `docs/bootstrap.md` за пунктами з кроків 1–8: посилання на run, назви змінних і де вони задані,
регіон, прод-URL. **Жодних значень секретів.** Знімки екрана можна класти в `docs/img/`. Пункти «Тег спеки злито»
і «Після merge у `main` job `db`…» заповните пізніше. У пункті про ruleset допишіть `conformance`: у форку він теж
обов'язковий (крок 2), хоча чекліст шаблону називає лише `spec` і `app`. У пункті про змінні Vercel біля
`SUPABASE_SECRET_KEY` напишіть «не додано — коду застосунку не потрібен» (крок 5), хоча чекліст шаблону називає
три змінні.

**Перший PR — `docs/bootstrap.md`, а не тест.** У демо заняття `spec-v0.1` уже вийшов, тому там перший PR
одразу з тестом з AC ID. У вас у день 0–1 спеки у форку ще немає, а тест з AC ID до `sync/spec-v0.1` валить
`conformance` з `ORPHAN` (AC немає в спеці). Тому зелений CI і preview доводимо на `docs/bootstrap.md`, а тести
з AC ID — після кроку 10. Якщо `spec-v0.1` уже вийшов, спершу зробіть крок 10: тоді перший PR може містити й
перший тест з AC ID у назві.

```bash
git add docs
git commit -m "Record bootstrap evidence"
git push -u origin bootstrap/evidence
gh pr create --web       # base: main вашого форку
```

Очікувано: зелені `spec`, `app`, `conformance`, коментар Vercel із preview-URL, який відкривається без входу.
Окремим коментарем — `@coderabbitai review`. Злийте PR. Після merge перевірте Actions: job `db` зелений і
виконав крок `supabase db push` (точний текст виводу CLI перевіримо на репетиції; база вже актуальна, бо міграцію
ви застосували в кроці 4). Якщо в лозі «db push пропущено», якогось секрету з кроку 7 бракує.

> Форкнули вже після тегу `spec-v0.1`? Тоді `conformance` упаде з `NO-ROW`: спека вже у форку, а матриця
> порожня. Спершу зробіть крок 10, а `docs/bootstrap.md` закомітьте в ту саму гілку `sync/spec-v0.1`.

## 10. Синхронізація `spec-v0.1` (20 хв) — трек B

Щойно PM поставив тег (до кінця дня 2):

```bash
git switch main
git pull
git fetch upstream --tags
git switch -c sync/spec-v0.1
git merge spec-v0.1
```

1. У `.spec-source` розкоментуйте два рядки: `upstream: <pm>/<team-slug>`, `version: spec-v0.1`.
2. `npm run spec:source` → OK.
3. У `docs/conformance.md` додайте рядок на кожен AC зі спеки. Поки код не написано — статус
   `відкладено (не реалізовано, задача з tasks.md)` або інша конкретна причина. Це вміє `/conformance` у Claude Code
   чи Cursor.
4. `npm run ac:coverage` → OK.
5. PR у форк → зелений CI → злийте кнопкою **Create a merge commit** (не Squash і не Rebase).

## 11. Читання спеки (1 год) — трек B

У Claude Code або Cursor: «Уточни спеку spec-v0.1 скілом spec-clarify». Скіл:

1. проходить 11 категорій таксономії і показує таблицю Ясно / Частково / Немає;
2. ставить вам **по одному** питанню з варіантами й рекомендацією — ви вирішуєте, чи йде воно до PM;
3. видає готові тексти issues і додає рядки в `docs/spec-reading.md`.

**Власне око обов'язкове.** Перечитайте спеку самі до запуску скіла й запишіть свої питання: у журналі має
бути видно, що знайшли ви, а що скіл (розділ «Що пропустив би без скіла / що скіл знайшов зайве»).

**Створюйте issues через вебформу**: `https://github.com/<pm>/<team-slug>/issues/new/choose` → «Питання до
специфікації», вставте поля з тексту скіла. Не через `gh issue create --label spec-question`: мітку через API може
ставити лише користувач із правом push у репозиторій, інакше GitHub мовчки її відкидає. Форма ставить мітку сама.

Потім у `docs/spec-reading.md`:

- **таблицю скану** (11 категорій, статус Ясно / Частково / Немає, де саме) збережіть у розділ
  `## Скан за таксономією` над «Питання до PM». Скіл показує її лише в чаті, а в шаблоні файлу такого розділу
  немає: додайте його самі (рубрика B його перевіряє). Зразок —
  [`spec-reading.md` демо](../examples/leave-portal/2-clarify/spec-reading.md);
- колонка «Issue» — посилання `<pm>/<team-slug>#N`;
- «Рішення (що зробили)» — ваше **робоче припущення**, поки PM не відповів;
- коли PM відповість — «Відповідь PM» і що змінили;
- «Власні припущення» — те, що ви вирішили самі без питання.

Зразок — [`examples/leave-portal/2-clarify/spec-reading.md`](../examples/leave-portal/2-clarify/spec-reading.md) і
[`issue-Q2.md`](../examples/leave-portal/2-clarify/issue-Q2.md). Журнал комітьте PR-ом у форк.

**PM ще не випустив `spec-v0.1`?** Читайте відкриту зміну з його PR: `git fetch upstream`, потім
`git show upstream/spec/add-<зріз>:openspec/changes/add-<зріз>/proposal.md` (і так само `specs/…/spec.md`). У полі
«Версія специфікації» issue вкажіть id зміни (`add-<зріз>`). Так само ставте питання й коментарі в PR PM — це
найдешевший момент для відповіді.

**Доказ:** щонайменше 3 issues `spec-question` в upstream від вас, `docs/spec-reading.md` у `main` форку.

## 12. Реалізація зрізу (4–5 год) — трек C

### 12.1 План: зміна `impl-<зріз>` (45 хв)

```bash
git switch main
git pull
git switch -c impl/<зріз>-plan
npx openspec new change impl-<зріз>
```

1. У `openspec/changes/impl-<зріз>/.openspec.yaml` допишіть рядок `skip_specs: true` (саме `skip_specs`:
   помилку в ключі OpenSpec мовчки ігнорує). Зміна розробника не змінює спеку PM.
2. `proposal.md` — коротко: який зріз і яка версія спеки; `design.md` — таблиці, міграції (GRANT + RLS +
   політики), server actions, сторінки, і кожне рішення з посиланням на AC ID; `tasks.md` — **вертикальні** задачі
   (таблиця/RLS + доменна функція + server action/сторінка + тест), кожна закінчується рядком `Done when: AC-…`.
   Правила для агента — у `openspec/config.yaml` (`rules: design`, `rules: tasks`), шаблони —
   `npx openspec instructions design --change impl-<зріз>` і `npx openspec instructions tasks --change impl-<зріз>`.
3. Рішення, яких спека свідомо не дає (модель даних, бібліотеки), записуйте як ADR у `docs/decisions/`.
4. Перевірте: `npx openspec validate impl-<зріз> --strict` → PR у форк → злийте.

Зразок — [`examples/leave-portal/3-impl/tasks.md`](../examples/leave-portal/3-impl/tasks.md) і
[`design.md`](../examples/leave-portal/3-impl/design.md).

### 12.2 Задача за задачею (3–4 год)

Одна задача (або кілька маленьких) — одна гілка й один PR. Агенту давайте одну задачу з `tasks.md` за раз
(у Claude Code може допомогти `/opsx:apply impl-<зріз>`), а не всю спеку.

1. **Міграція** (якщо задача її потребує):

   ```bash
   npx supabase migration new <назва>
   # SQL: enable row level security; revoke all … from anon, authenticated; явні grant;
   #      політика на кожну операцію to authenticated з (select auth.uid())
   npx supabase db push --dry-run
   npm run db:push
   npm run db:types
   ```

   Застосовані міграції не редагуємо, лише новий файл. Довідкові дані першої задачі — в ідемпотентний
   `supabase/seed.sql` (`insert … on conflict do nothing`), у хмару його застосовує лише
   `npx supabase db push --include-seed` (звичайний `db:push` і job `db` у CI сид **не** виконують). **У хмарі сид
   виконується лише один раз:** відредагований `seed.sql` повторно не виконується, CLI лише оновлює його хеш
   (`Updating seed hash…`). Довідкові дані, додані пізніше (або якщо `--include-seed` уже запускали), — новою
   міграцією з `insert … on conflict do nothing`.
   Нові ролі — нова міграція з `profiles_role_check`, потім оновіть `supabase/demo-users.json` і
   `npm run users:demo`.
2. **Код і тести.** Назва кожного тесту поведінки починається з AC ID: `it("AC-…-02: …")`. Пропущені тести
   (`.skip`, `.todo`) AC не покривають.
3. **Матриця.** `/conformance` перебудовує `docs/conformance.md`. Статус — дослівно `реалізовано` або
   `відкладено (причина)`.
4. **Перевірки перед PR:**

   ```bash
   npm run lint
   npm run typecheck
   npm test
   npm run build          # у виводі має бути ƒ Proxy (Middleware)
   npm run ac:coverage
   ```

5. **PR у форк** → зелений CI → preview відкривається → коментар `@coderabbitai review` → виправлення → merge.
   Після merge Vercel деплоїть прод, а job `db` застосовує міграції. Якщо прод одразу після merge показує помилку
   бази, дочекайтеся завершення job `db`.
6. **Останній PR зрізу, до merge:** `npx openspec archive impl-<зріз> -y` у тій самій гілці (прямі коміти в
   `main` ruleset не пустить).

**PM ще не випустив `spec-v0.1`?** `design.md` і `tasks.md` можна почати з відкритої зміни в PR PM (як у
кроці 11), код — писати й зливати. Тести поведінки (з AC ID у назві) — лише після злиття `sync/spec-v0.1`:
до того `conformance` падає з `ORPHAN`, а ruleset без нього PR не пустить. Затримка PM балів не знімає
([README](../README.md#що-знімає-бали-розробник)).

**Відповіді PM ще немає?** Або реалізуйте своє робоче припущення з `docs/spec-reading.md` і назвіть його в PR,
або залиште AC зі статусом `відкладено (чекаємо відповідь PM на Q3, <pm>/<team-slug>#N)`.

**Доказ:** PR-и у форку зі зеленим CI, `docs/conformance.md` на `main` з рядком для кожного AC, прод-URL, на
якому зріз працює під синтетичними користувачами.

## 13. Синхронізація `spec-v0.2` (45 хв) — трек D

```bash
git switch main
git pull
git fetch upstream --tags
git switch -c sync/spec-v0.2
git merge spec-v0.2
```

1. `.spec-source` → `version: spec-v0.2`; `npm run spec:source` → OK.
2. `npm run ac:coverage` **упаде** з `NO-ROW` на нових AC — це і є дрейф, який ловить CI (зразок —
   [`drift-ci.txt`](../examples/leave-portal/4-spec-v0.2/drift-ci.txt)). Додайте рядки.
3. **Змінені сценарії (MODIFIED) перевірка не бачить** — звірте їх самі:

   ```bash
   git diff spec-v0.1 spec-v0.2 -- openspec/specs
   ```

   Для кожного зміненого AC: код уже відповідає (бо ви взяли ту саму відповідь як робоче припущення), треба
   змінити код і тест, або `відкладено (причина)`. Таблицю «Синхронізація на spec-v0.2» додайте в
   `docs/spec-reading.md`.
4. Реалізація дельти — у тій самій гілці або окремими PR після злиття sync-PR. Великій дельті дайте окрему зміну
   `impl-<зріз>-v0-2` з `skip_specs: true`.
5. PR → зелений CI → **Create a merge commit**.

Якщо попередній sync-PR злили squash-ем і `git merge` дав конфлікти в `product/` чи `openspec/specs/`, візьміть
версію тегу: `git checkout --theirs -- product openspec/specs`.

**PM не випустив `spec-v0.2` до дня 6?** Трек D оцінюємо за тим, що є: sync-PR на останній тег PM (або рядок у
файлі здачі «`spec-v0.2` не випущено»). Затримка PM не знімає з вас балів.

## 14. Здача (15 хв)

Інструкція — [`submissions/README.md`](../submissions/README.md). SHA для розробника — коміт `main` вашого форку,
який задеплоєно на прод:

```bash
git switch main
git pull
git rev-parse HEAD
```

## Бонус E (+5, 1–2 год) — одне з двох

- **Бутстрап без шаблону.** В окремому порожньому репозиторії попросіть агента зібрати той самий стек з нуля
  (Next.js 16.4 + Supabase + вхід email/пароль + CI). Порівняйте результат зі [списком пасток, які обходить
  шаблон](https://github.com/koldovsky/2026-quitcode-06-project-template#як-цей-шаблон-зібрано-з-нуля):
  де лежить `proxy.ts`, Cache Components, `setAll` із заголовками, `*_ANON_KEY` замість publishable, GRANT у
  міграції, `.env.example` у `.gitignore`, `next typegen` перед `tsc`. Таблиця «пастка → що зробив агент → як помітили».
- **`/opsx:verify` + власний розбір.** До archive запустіть `/opsx:verify impl-<зріз>`. Для кожної знахідки
  (CRITICAL / WARNING / SUGGESTION) вирішіть: справжня проблема, хибна тривога чи смак — і чому. Підсумуйте, що
  verify знайшов такого, чого не ловлять `npm test` і `ac:coverage`, і де він помилився.

Звіт — у розділі «Бонус E» файлу здачі: посилання і до пів сторінки висновків.
