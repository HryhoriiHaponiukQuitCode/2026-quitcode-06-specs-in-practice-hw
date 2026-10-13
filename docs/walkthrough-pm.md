# Покрокова інструкція: трек PM

Ви — власник **репозиторію команди**: створюєте його з шаблону у своєму особистому акаунті GitHub, запрошуєте
розробників, налаштовуєте Supabase, Vercel і правила злиття, а потім ведете в ньому продукт і специфікацію.
Розробники працюють у тому самому репозиторії: одна спільна реалізація, гілки, PR-и й рев'ю одне одного. Код ви
не пишете. Ваш продукт — `product/`, `openspec/`, теги `spec-vX.Y` і задачі для команди.

- Рубрика й Definition of Done — у [README](../README.md#трек-pm--100--5-балів).
- Як писати вимоги й AC — [`docs/spec-guide.md`](spec-guide.md).
- Бриф вашої команди — [`materials/briefs/`](../materials/briefs/).
- Зразок формату — [`examples/leave-portal/`](../examples/leave-portal/).
- Короткий чекліст налаштування є й у самому репозиторії команди: `docs/bootstrap.md` (з шаблону).

Позначення: `<pm>` — ваш GitHub-логін, `<team-slug>` — слаг команди (`property-workflows`, `therapy-scheduling`,
`school-purchasing` або `leave-portal`), `<techlead>` — логін техліда, `<ref>` — Project ref проєкту Supabase.
Команди нижче — для Git Bash на Windows і терміналу macOS. Де PowerShell відрізняється, це сказано окремо. У
Windows PowerShell 5.1 немає `&&`: виконуйте команди по черзі. Якщо PowerShell відповідає «running scripts is
disabled on this system», пишіть `npx.cmd`, `npm.cmd`, `openspec.cmd` замість `npx`, `npm`, `openspec`.

**Бюджет треку: 9–11 годин за тиждень** (сума кроків нижче, без бонусу). Найбільше часу йде на налаштування
команди в день 0 (кроки 1–8, ~2 год), перший зріз (кроки 10–12) і цикл уточнень (крок 14).

Дні календарні (з вихідними); «кінець дня N» — 23:59 за Києвом N-го дня після заняття (день 0 — день заняття).

| Крок | Коли | Бюджет | Трек |
|---|---|---|---|
| 0. Підготовка машини й акаунтів | до заняття | 20 хв | — |
| 1. Репозиторій команди з шаблону | день 0, одразу після заняття | 10 хв | A |
| 2. Collaborators, мітки, denylist, CodeRabbit | день 0 | 15 хв | A |
| 3. Supabase: організація, проєкт, Developer-інвайти, Auth | день 0 | 30 хв | A |
| 4. Environment `production` і секрет `SUPABASE_DB_URL` | день 0 | 15 хв | A |
| 5. Vercel: імпорт, дві змінні, Require Log In, адреси Auth | день 0 | 20 хв | A |
| 6. Перша база: `db-deploy` з `include_seed`, синтетичні користувачі | день 0 | 15 хв | A |
| 7. CODEOWNERS — перший PR | день 0 | 10 хв | A |
| 8. Ruleset на `main` і докази налаштування | день 0 | 15 хв | A |
| 9. Продуктовий контекст | дні 0–1 | 1–1,5 год | A |
| 9а. Зустріч команди: перший раунд питань до брифу | дні 0–1 | 15–20 хв | — |
| 10. Перший зріз: чернетка зміни | дні 1–2 | 1,5–2 год | B |
| 11. Перевірка: `validate`, `spec:check`, `spec-review` | день 2 | 45 хв | B |
| 12. Реліз `spec-v0.1` з рядками матриці | до кінця дня 2 | 30 хв | B |
| 13. Задачі разом із техлідом | дні 2–3 | 20–30 хв | — |
| 14. Цикл уточнень і `spec-v0.2` | дні 2–4 | 1,5–2 год | C |
| 15. Нотатка PM | дні 6–7 | 30 хв | D |
| 16. Здача | день 7 | 15 хв | — |

## Як працює `main` команди (прочитайте до кроку 1)

Після кроку 8 у `main` нічого не потрапляє без PR. Правила ruleset ([`team-main.json`](../materials/rulesets/team-main.json))
і `.github/CODEOWNERS` змінюють звичну механіку, тож запам'ятайте чотири речі:

- **Свій PR схвалити не можна** (так влаштований GitHub). Ваші PR зі спекою схвалює техлід, PR розробника — будь-хто з
  колег.
- **Approve дає не будь-хто, якщо PR чіпає чужу зону.** Зміни в `/product/`, `/openspec/` і `/.github/` схвалює
  власник коду — ви або техлід; міграції в `/supabase/migrations/` — техлід або ви. Тому ваш PR зі спекою схвалює
  саме техлід, а міграцію техліда — ви.
- **Approve має стояти після останнього push.** Кожен новий коміт у гілку (зокрема archive, рядок у CHANGELOG,
  кнопка Update branch чи злиття `main` у гілку) потребує нового approve від когось, хто не робив цей push. Тому
  спершу всі коміти, потім approve, потім merge.
- **Обов'язкові перевірки** — `spec`, `app`, `conformance`, `db-check` (GitHub Actions). Vercel — не обов'язкова
  перевірка. Усі коментарі рев'ю мають бути розв'язані (Resolve conversation).

## 0. Підготовка (20 хв)

1. **Акаунти (особисті, не клієнтські):** GitHub; [Supabase](https://supabase.com/dashboard) — потрібен **вільний
   слот**: на Free активних може бути лише 2 проєкти на всі організації, де ви Owner або Administrator (проєкти з
   WS5 поставте на паузу або видаліть); [Vercel](https://vercel.com) Hobby (вхід через GitHub).
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
6. **Техлід.** На занятті або в чаті команди до вечора дня 0 команда обирає техліда — одного з розробників. Його
   логін потрібен у кроці 7.

**Доказ:** нічого не комітимо. Якщо `openspec --version` показує іншу версію, `/opsx:*` поводитимуться інакше, ніж на занятті.

## 1. Репозиторій команди з шаблону (10 хв) — трек A

Цей крок робимо **одразу після заняття** (день 0): розробники чекають на інвайти.

Через браузер: <https://github.com/koldovsky/2026-quitcode-06-project-template> → **Use this template** →
**Create a new repository** → Owner — ваш особистий акаунт (не організація), Repository name — `<team-slug>`,
**Public** → Create.

Або однією командою (з теки, де тримаєте проєкти):

```bash
gh repo create <team-slug> --public --template koldovsky/2026-quitcode-06-project-template --clone
cd <team-slug>
npm ci
```

**Публічним — одразу й назавжди.** На GitHub Free ruleset, CODEOWNERS і секрети environment діють лише в публічних
репозиторіях. А Vercel Hobby не деплоїть коміти колег, запушені, поки репозиторій був приватним, і вони лишаються
заблокованими навіть після відкриття. Тому ніякого «спершу приватний, потім відкрию».

**Доказ:** URL репозиторію команди — для файлу здачі. Перший прогін CI (Actions → CI) має стартувати після
створення; якщо прогону немає, Actions → CI → **Run workflow** або `gh workflow run ci.yml`. У ньому всі п'ять jobs,
зокрема `db-check` (на `main` теж); `db-deploy` зелений, але міграцій не застосовує — повідомлення про порожній
`SUPABASE_DB_URL`, так і має бути до кроку 4. Саме цей прогін потрібен ruleset-у в кроці 8.

## 2. Collaborators, мітки, denylist, CodeRabbit (15 хв) — трек A

**Collaborators.** Settings → Collaborators → **Add people** → логін кожного розробника → Add. На особистому
репозиторії всі collaborators отримують право запису. Додайте й викладача (`koldovsky`): він потрібен лише як
запасний варіант, якщо реліз спеки застрягне (крок 12). Інвайт діє, лише коли його **прийняли**: напишіть у чат
команди «прийміть інвайт: <https://github.com/<pm>/<team-slug>/invitations>».

Або в терміналі (по одному на логін):

```bash
gh api --method PUT repos/<pm>/<team-slug>/collaborators/<login>
```

**Мітки.** Форми issues ставлять лише мітки, які вже є в репозиторії, тож створіть їх, якщо їх ще немає: Issues →
Labels → **New label** → `spec-question` («Питання до специфікації») і `task` («Задача зрізу»). Або
(`--force` оновить мітку, якщо вона вже існує, замість помилки):

```bash
gh label create spec-question --force --repo <pm>/<team-slug> --description "Питання до специфікації"
gh label create task --force --repo <pm>/<team-slug> --description "Задача зрізу"
```

**Denylist (локальний файл і секрет).** Справжні назви клієнта — **лише** в локальному `product/denylist.local.txt`
(не в git) і в секреті `SPEC_DENYLIST`. Репозиторій публічний, тому справжніх назв немає ні в
`product/denylist.txt`, ні в будь-якому іншому файлі, issue чи PR.

- **Локальний файл.** У своєму клоні створіть у редакторі `product/denylist.local.txt`: по одній назві на рядок,
  `#` — коментар. Шаблон уже має цей шлях у `.gitignore`. Перевірка — у виводі першої команди має бути рядок
  `!! product/denylist.local.txt` (`!!` — git ігнорує файл) і жодного рядка з `??`, а друга має показати
  «слів у denylist: N», де N — не менше кількості ваших назв:

  ```bash
  git status --short --ignored product
  npm run spec:check
  ```

  Бачите `denylist.local.txt.txt`? Провідник Windows приховав розширення, і до імені додалось друге `.txt`:
  увімкніть Провідник → Вигляд → Показати → Розширення імен файлів і перейменуйте файл на `denylist.local.txt`.
- **Секрет для CI.** Settings → Secrets and variables → Actions → **New repository secret** → Name
  `SPEC_DENYLIST`, Secret — ті самі назви через кому. Або `gh secret set SPEC_DENYLIST --repo <pm>/<team-slug>`
  (значення спитає й ніде не надрукує). Це секрет **репозиторію**, а не environment: job `spec` бачить його і в PR
  гілок розробників, бо всі гілки живуть в одному репозиторії.
- **Список для розробників.** Надішліть ті самі назви кожному розробнику **особистим повідомленням** (не в
  загальний чат і не в issue): вони збережуть їх у `product/denylist.local.txt` свого клону, і `npm run spec:check`
  ловитиме назви в документації, коді, сидах і тестах ще до push.

**CodeRabbit.** <https://github.com/apps/coderabbitai> → Install → ваш акаунт → Only select repositories →
`<team-slug>`. Встановлює лише власник репозиторію, тобто ви; розробникам нічого ставити не треба. У публічних
репозиторіях із < 10 зірок рев'ю не запускається само: після створення PR автор пише окремим **коментарем**
`@coderabbitai review`.

## 3. Supabase: організація, проєкт, Developer-інвайти, Auth (30 хв) — трек A

Одна база на команду: її створюєте ви, і вона займає **ваш** безкоштовний слот.

1. **Організація.** Dashboard → New organization → назва `<team-slug>`, план **Free**.
2. **Проєкт** у цій організації: назва `<team-slug>`, регіон — найближчий. **Пароль бази** — довгий (≥ 20
   символів), лише з літер і цифр: тоді його не треба кодувати в рядку підключення (крок 4). Збережіть у менеджері
   паролів. Розробникам пароль не потрібен і його ніхто, крім вас, не знає.
3. **Інвайти.** Organization → Team → **Invite** → email, з яким кожен розробник зареєстрований у Supabase, роль
   **Developer**. Не Administrator і не Owner: така роль займе безкоштовний слот у кожного з них. Developer бачить
   URL і ключі проєкту й керує тестовими користувачами, але не змінює налаштувань проєкту. Напишіть у чат: «прийміть
   інвайт Supabase з пошти».
4. **Auth** (лише ви: Developer налаштувань Auth не змінює). У налаштуваннях Authentication вимкніть **Allow new
   users to sign up** і, у провайдері Email, **Confirm email** (орієнтовно Authentication → Sign In / Providers;
   точний шлях у Dashboard звіряємо на репетиції). Публічний ключ лежить у JS-бандлі, тож з увімкненою реєстрацією
   будь-хто зареєструється сам. Вбудована пошта Supabase надсилає 2 листи на годину на весь проєкт і лише членам
   організації, тому користувачів створює скрипт (крок 6), а не лист. `supabase/config.toml` шаблону має
   `[auth] enable_signup = false`, але це конфігурація CLI: у хмару вона не переноситься, перемикач у Dashboard
   обов'язковий.

Проєкт Free **засинає** після ~7 днів без активності, а відновити його (Dashboard → Restore project; точну назву кнопки звіряємо на репетиції) може лише
власник, тобто ви. Перевіряйте перед кожним заняттям і перед здачею.

## 4. Environment `production` і секрет `SUPABASE_DB_URL` (15 хв) — трек A

Міграції в хмарну базу застосовує лише CI: job `db-deploy` після кожного merge у `main`. Йому потрібен рядок
підключення з паролем бази, і цей секрет має бачити тільки `main`, а не гілки розробників.

1. **Environment.** Settings → Environments → `production`. Перший прогін CI з кроку 1 міг уже створити його
   автоматично (job `db-deploy` на нього посилається) — тоді відкрийте його. `Production` з великої літери, яке
   створює Vercel для своїх деплоїв, — те саме середовище (регістр у назвах GitHub не розрізняє): налаштуйте
   його, нового не створюйте. Якщо середовища ще немає — **New environment** →
   `production` → Configure environment. Deployment branches and tags → **Selected branches and tags** → Add deployment branch or tag rule → `main`.
2. **Рядок підключення.** Supabase → кнопка **Connect** угорі проєкту → **Session pooler** (порт `5432`) → URI
   вигляду `postgresql://postgres.<ref>:[YOUR-PASSWORD]@<хост пулера>:5432/postgres`. Замініть `[YOUR-PASSWORD]`
   паролем бази. Саме Session pooler: пряме підключення на Free працює лише через IPv6, а раннери GitHub — IPv4;
   Transaction pooler (порт `6543`) для зміни схеми не годиться.
3. **Секрет environment.** У тому ж environment → Environment secrets → **Add environment secret** → Name
   `SUPABASE_DB_URL`, Value — рядок із пункту 2. Або в терміналі:

   ```bash
   gh secret set SUPABASE_DB_URL --env production --repo <pm>/<team-slug>
   ```

   Не кладіть його в Repository secrets: секрет репозиторію дістане будь-який workflow будь-якої гілки.

**Запасний шлях**, якщо `db-deploy` не підключається через пулер: офіційний варіант Supabase — `supabase link` +
`supabase db push` із трьома секретами в тому самому environment `production`: `SUPABASE_ACCESS_TOKEN` (токен з
обмеженими правами, який створюєте ви: Supabase → Account → Access Tokens, лише цей проєкт, право Project Settings →
API Keys → API Key Secrets — Read), `SUPABASE_DB_PASSWORD` і `SUPABASE_PROJECT_ID` (`<ref>`). Секрет
`SUPABASE_DB_URL` тоді видаліть: `db-deploy` сам перейде на цей шлях, у workflow нічого не змінюємо (README шаблону,
розділ «Міграції в прод»). Основним цей шлях не робимо: токени з обмеженими правами вийшли 06.10.2026, і з
`supabase link` у них є відкрита помилка.

## 5. Vercel: імпорт, дві змінні, Require Log In, адреси Auth (20 хв) — трек A

Імпортувати особистий репозиторій у Vercel може лише його власник, тому Vercel — теж ваш.

1. Vercel → **Add New → Project** → Import Git Repository → `<pm>/<team-slug>` (дайте застосунку Vercel доступ лише
   до цього репозиторію). Framework — Next.js, Node 24.x береться з `engines` у `package.json`.
2. **Environment Variables** для **Production і Preview** — рівно дві: `NEXT_PUBLIC_SUPABASE_URL` і
   `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (Supabase → Project Settings → API Keys, ключ `sb_publishable_…`).
   `SUPABASE_SECRET_KEY` у Vercel **ніколи** не додаємо: він обходить RLS, а застосунку першого зрізу не потрібен.
   **Не** підключайте Supabase через Marketplace-інтеграцію Vercel: вона створює окрему організацію й займає слот.
3. **Deploy** → прод-URL `https://<проєкт>.vercel.app/login` відкривається.
4. Settings → **Deployment Protection** → **Vercel Authentication**: вимкніть перемикач **Require Log In** → **Save**
   (на новому Hobby-проєкті він увімкнений; список «Standard Protection» поруч не чіпайте). Інакше колеги не
   відкриють preview своїх PR: на Hobby немає учасників команди, і без входу у ваш акаунт preview закрите. Чому
   це прийнятно лише для навчального проєкту і за яких трьох правил — у
   [walkthrough-dev](walkthrough-dev.md#ризики-відкритих-preview).
5. **Адреси Auth.** Supabase → Authentication → **URL Configuration**: Site URL — прод-URL Vercel; Redirect URLs —
   `http://localhost:3000/**` і `https://*-<slug-акаунта-vercel>.vercel.app/**` (slug — остання частина адреси
   будь-якого preview: `<проєкт>-<хеш>-<slug>.vercel.app`).
6. Після будь-якої зміни змінних — Deployments → останній деплой → ⋯ → **Redeploy**: змінні діють лише для нових
   деплоїв.

Обмеження Hobby: лише некомерційне особисте використання — застосунок навчальний, дані вигадані, без брендингу
клієнта. Cron — щонайбільше раз на добу.

## 6. Перша база: `db-deploy` з `include_seed`, синтетичні користувачі (15 хв) — трек A

1. **Міграції й сид шаблону.** Actions → CI → **Run workflow** → Branch `main`, позначте `include_seed` → Run. Або:

   ```bash
   gh workflow run ci.yml --repo <pm>/<team-slug> --ref main -f include_seed=true
   ```

   Job `db-deploy` має бути зеленим і в лозі — застосована міграція шаблону `20261012000000_profiles_and_roles.sql`
   (точний текст виводу CLI звіряємо на репетиції). Сид у хмарі виконується **лише один раз**: пізніші довідкові
   дані — новою міграцією. `include_seed` після цього не вмикайте. Червоний job — перевірте пароль у рядку
   підключення і чи проєкт не на паузі; не допомогло — запасний шлях із кроку 4.
2. **Синтетичні користувачі.** Їх створює `npm run users:demo`, якому потрібен `SUPABASE_SECRET_KEY`. Запускаєте ви
   або техлід (він Developer і бачить ключі):

   ```bash
   cp .env.example .env.local
   # заповніть: NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY,
   # SUPABASE_SECRET_KEY (sb_secret_…), DEMO_USER_PASSWORD (довгий і випадковий)
   npm run users:demo     # admin@example.test, member1@example.test, member2@example.test
   npm run dev            # http://localhost:3000 → вхід як admin@example.test
   ```

   `DEMO_USER_PASSWORD` надішліть команді особистими повідомленнями. `.env.local` — у `.gitignore`; значень не
   вставляйте в чат з агентом.

**Ніколи** не застосовуйте міграції з ноутбука (`supabase db push`) і не змінюйте схему в SQL Editor чи Table
Editor: база спільна, і зміна поза `main` ламає наступний `db-deploy` усій команді.

## 7. CODEOWNERS — перший PR (10 хв) — трек A

Техлід має **прийняти** інвайт GitHub до цього кроку: логін без доступу до репозиторію GitHub у CODEOWNERS просто
ігнорує, і ваші PR зі спекою не матимуть кому схвалити.

```bash
git switch main
git pull
git switch -c setup/codeowners
```

У `.github/CODEOWNERS` замініть плейсхолдери за коментарями у файлі, щоб вийшло:

```text
/product/              @<pm> @<techlead>
/openspec/             @<pm> @<techlead>
/.github/              @<pm> @<techlead>
/supabase/migrations/  @<techlead> @<pm>
```

```bash
git add .github/CODEOWNERS
git commit -m "Set code owners: PM and tech lead"
git push -u origin setup/codeowners
gh pr create --web
```

Тип PR — «Онбординг» (він дозволяє `.github/CODEOWNERS` у день 0). Цей PR — перший прогін CI на події `pull_request`: ті самі чотири перевірки, що на `main`, а `db-check` тут ще й
звіряє порядок міграцій із `main` (у прогонах на `main` цей крок пропущено). Дочекайтеся зелених `spec`, `app`,
`conformance`, `db-check`. Ruleset ще немає, але PR без approve колеги рубрика A не зараховує, тож попросіть
техліда схвалити (Review changes → Approve) і лише тоді злийте (Squash and merge). Відкрийте файл на GitHub: якщо
логін указано з помилкою, GitHub підсвічує рядок (точний вигляд звіряємо на репетиції).

## 8. Ruleset на `main` і докази налаштування (15 хв) — трек A

Порядок важливий: спершу CI пройшов на `main` (кроки 1 і 6) і в PR (крок 7), потім ruleset. Обов'язкові перевірки
в ньому прив'язані до джерела GitHub Actions, а GitHub приймає таку прив'язку, лише коли Actions уже нещодавно
звітували перевірки в цьому репозиторії.

**Налаштування злиття.** Settings → General → Pull Requests: **Allow merge commits** і **Allow squash merging** мають
бути увімкнені (так за замовчуванням), **Allow rebase merging** можна вимкнути. Ruleset дозволяє squash і merge
commit, а метод, вимкнений тут, не запрацює й з ruleset-ом.

**Через `gh api`** (рекомендовано; JSON — [`materials/rulesets/team-main.json`](../materials/rulesets/team-main.json)):

```bash
curl -sLo ruleset.json https://raw.githubusercontent.com/koldovsky/2026-quitcode-06-specs-in-practice-hw/main/materials/rulesets/team-main.json
gh api repos/<pm>/<team-slug>/rulesets --method POST --input ruleset.json
rm ruleset.json
```

PowerShell: `curl.exe` замість `curl` (у Windows PowerShell 5.1 `curl` — це інша команда), `Remove-Item ruleset.json` замість `rm`.

**Або через браузер:** Settings → Rules → Rulesets → New ruleset → New branch ruleset:

- Ruleset name — `main: PR + review + CI`, Enforcement status — **Active**, Bypass list — **порожній**;
- Target branches → Add target → **Include default branch**;
- **Restrict deletions**, **Block force pushes**;
- **Require a pull request before merging**: Required approvals — `1`; **Dismiss stale pull request approvals when
  new commits are pushed** — вимкнено; **Require review from Code Owners** — увімкнено; **Require approval of the most
  recent reviewable push** — увімкнено; **Require conversation resolution before merging** — увімкнено; Allowed merge
  methods — **Merge** і **Squash**, без Rebase (точні назви полів звіряємо на репетиції);
- **Require status checks to pass** → Add checks → `spec`, `app`, `conformance`, `db-check` (джерело — GitHub
  Actions); **Require branches to be up to date before merging** — вимкнено → Create.

Якщо перевірка не знаходиться в пошуку, відкрийте Actions: прогони з кроків 1, 6 і 7 мають містити jobs саме з
такими назвами (`db-check` на `main` теж запускається). Те саме, якщо `gh api` відповідає помилкою про
`integration_id`: спершу зелений прогін CI, потім повторіть команду.

Чому саме так: approve колеги — рев'ю, яке ми оцінюємо; «stale approvals» вимкнено, бо інакше кожен merge у `main`
знімав би approve з усіх відкритих PR команди; squash — для задач, merge commit — запасний шлях для Vercel (див.
[walkthrough-dev](walkthrough-dev.md#vercel-preview-і-прод)); loose-перевірки — щоб не оновлювати гілку після кожного
чужого merge.

**Перший PR під ruleset-ом — докази налаштування.** Він же перевіряє ruleset:

```bash
git switch main
git pull
git switch -c onboarding/<pm>
```

- У `docs/team.md` — ваш рядок: `| @<pm> | PM | специфікація й налаштування |`.
- У `docs/bootstrap.md` — позначте розділи 1–6 і додайте докази **без жодного значення**: посилання на прогони CI і
  `db-deploy`, назви змінних Vercel і де вони задані, прод-URL, знімки екрана в `docs/img/` (Supabase Team з
  ролями Developer — email учасників замажте; вимкнені «Allow new users to sign up» і «Confirm email»; сторінка
  ruleset-у). Знімки `spec:check` не читає: перевірте самі, що на них немає назв клієнта, імен людей, email не з
  `example.test` і значень ключів.

```bash
git add docs
git commit -m "Onboarding: PM row in the team table, setup evidence in bootstrap.md"
git push -u origin onboarding/<pm>
gh pr create --web
```

Тип PR — «Онбординг». **Перевірка ruleset-у:** кнопка merge заблокована, доки немає approve (посилання на цей PR
потім піде в перший пункт розділу 7 «Перевірка в перший день» у `docs/bootstrap.md`). Якщо GitHub таки дає вам, власнику, злити
власний PR без approve, не користуйтеся цим і напишіть у чат курсу: рубрика A перевіряє, що кожен PR у `main` має
approve колеги. Approve колеги (`docs/` власника коду не має, тож будь-хто) → Squash and merge.

Розділ 7 (цей PR, preview PR колеги, прод-деплой після squash-merge колеги, PR із міграцією й `db-deploy`) позначте,
коли це станеться за тиждень, — невеликим PR «Онбординг» до здачі.

**Доказ:** злиті PR кроків 7 і 8 (посилання — у полі «PR налаштування» файлу здачі), сторінка ruleset-у або вивід
`gh api repos/<pm>/<team-slug>/rulesets`.

Наприкінці дня 0 — повідомлення в чат команди: «Репо готове: <URL>. Прийміть інвайти GitHub і Supabase, далі —
`docs/onboarding.md`; перший PR кожного — рядок про себе в `docs/team.md`».

## 9. Продуктовий контекст (1–1,5 год) — трек A

```bash
git switch main
git pull
git switch -c product/context
```

Заповніть файли за коментарями в них:

| Файл | Що має бути | На що звернути увагу |
|---|---|---|
| `product/vision.md` | проблема, для кого, результат (2–4 вимірні ознаки), межі продукту | одна сторінка; розділ «Обмеження навчального проєкту» лишіть |
| `product/glossary.md` | 8–15 термінів; колонка «Не вживаємо» | кожен іменник зі сценаріїв першого зрізу має бути тут |
| `product/roles.md` | ролі зі значеннями `profiles.role` і **матриця роль × сутність × дія** | заповнені клітинки або «—», без порожніх; ролі шаблону `member`/`admin` замініть своїми |
| `product/roadmap.md` | зріз на кожне заняття WS6–WS11 | WS6 — перший зріз; далі — з брифу («поза зрізом») |
| `product/denylist.txt` | внутрішні слова, яких не має бути ні в спеках, ні в коді й документації (кодові назви, заглушки з чужих шаблонів на кшталт Contoso); назви сторонніх систем із брифу (QuickBooks, Buildium) — дозволені | справжніх назв клієнта тут **немає** — вони в локальному `product/denylist.local.txt` (не в git) і секреті `SPEC_DENYLIST` (крок 2) |
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

**З агентом:** «Зроби крок 9: `npm run spec:check`, commit `product/` і `openspec/config.yaml` у гілку
`product/context`, push і PR» — підтверджуйте кожну команду.

У шаблоні PR позначте тип «Зміна специфікації». Зелений CI → approve техліда → злийте (Squash and merge).

**Доказ:** злитий PR з approve техліда (посилання — у файл здачі).

## 9а. Зустріч команди: перший раунд питань до брифу (15–20 хв)

Цю розмову команда проводить сама, у день 0 або 1, ще до чернетки зміни (крок 10).
Організовуєте ви: призначте дзвінок на 15–20 хвилин і покличте всіх розробників. Матеріал — бриф команди в
[`materials/briefs/`](../materials/briefs/).

1. **2 хв — ви:** вголос мета зрізу і три правила з розділу «Рекомендований перший зріз (WS6)» брифу.
2. **4 хв — розробники:** кожен називає одну діру — що неясно, категорію з одинадцяти (слайд 21 деки) і 2–3 варіанти поведінки.
3. **3 хв — ви:** відповідаєте або записуєте питання як відкрите.
4. **1 хв — підсумок:** одне питання, яке вас найбільше здивувало.

Результат — список «питання → відповідь» або «питання → відкрите» у вашій нотатці (без назв клієнта); у кроці 10 він
іде в Open questions `proposal.md`. Хто не зміг прийти — пише свою діру в чат команди до кінця дня 1. Не вдається
зібратися — та сама розмова асинхронно в чаті команди: ви публікуєте мету й правила, кожен розробник до кінця дня 1 пише
одну діру, ви відповідаєте там само. Окремих балів за зустріч немає.

## 10. Перший зріз: чернетка зміни (1,5–2 год) — трек B

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

`/opsx:propose` за своєю інструкцією генерує всі артефакти зміни, зокрема `design.md` і `tasks.md` (як
реалізувати). У зміні PM їх не лишаємо — це окрема зміна `impl-<зріз>`, яку техлід веде з командою (крок 13):
додайте до запиту «пиши лише proposal.md і specs/, без design.md і tasks.md», а якщо агент їх усе ж створив —
видаліть обидва файли до коміту.

**Вручну:** `npx openspec new change add-<зріз>`, далі файли за [скелетом](spec-guide.md#скелет-зміни).

Що перевірити перед кроком 11:

- у теці зміни лише `.openspec.yaml`, `proposal.md` і `specs/` — без `design.md` і `tasks.md`;
- `proposal.md`: Why, What Changes з AC ID, Capabilities, Users & roles (таблиця), Non-goals, Assumptions,
  Open questions;
- кожна вимога — EARS з `SHALL`; кожен сценарій — `#### Scenario: AC-<AREA>-<NN> …`, один критерій, числа й дати;
- жодних таблиць, API, компонентів; дані синтетичні.

## 11. Перевірка (45 хв) — трек B

```bash
npx openspec validate add-<зріз> --strict
npm run spec:validate
npm run spec:check
```

Потім у Claude Code або Cursor: «Запусти скіл spec-review для зміни add-<зріз>». Скіл запускає ті самі перевірки,
проходить [чекліст якості](spec-guide.md#8-чекліст-перед-релізом) і таксономію
неоднозначностей і видає таблицю знахідок (БЛОКЕР / ВАЖЛИВО / ДРІБНИЦЯ) та «питання, які розробники, найімовірніше,
поставлять». Правки він вносить лише після вашого «так».

**Доказ (обов'язковий для треку B):** у PR зміни (крок 12) розділ опису `## spec-review` — таблиця знахідок і що
ви виправили, а що свідомо лишили (і чому). Без цього розділу — мінус бали за B.

## 12. Реліз `spec-v0.1` з рядками матриці (30 хв) — трек B, до кінця дня 2

Не працювали з гілками й тегами? Попросіть агента: «Виконай реліз spec-v0.1 за walkthrough-pm, крок 12; перед
кожним git push, створенням PR і merge питай мене». Агент питає перед push — так задано в AGENTS.md шаблону.

Порядок релізу (так само для `spec-v0.2`): гілка зміни → PR → коментарі команди → **у гілці** archive, запис у
`CHANGELOG.md` і рядки матриці для нових AC → **approve техліда після останнього push** → merge → `git switch main` і
`git pull` → `git tag spec-vX.Y` → `git push origin spec-vX.Y`.

```bash
git add openspec
git commit -m "Propose add-<зріз>"
git push -u origin spec/add-<зріз>
gh pr create --web
```

1. Опис PR: тип «Зміна специфікації», розділ `## spec-review`. Окремим коментарем — `@coderabbitai review`.
   Попросіть техліда бути рецензентом (Reviewers → `<techlead>`; CODEOWNERS запросить його й так).
2. Дайте команді кілька годин на коментарі до PR. Запитання розробників на цьому етапі — подарунок: відповідь
   до релізу дешевша за `spec-v0.2`. Розв'яжіть кожну розмову (Resolve conversation) — без цього merge заблоковано.
3. **У тій самій гілці, до merge:** archive, запис у журнал і рядки матриці.

   ```bash
   npx openspec archive add-<зріз> -y
   ```

   - У `product/CHANGELOG.md` — запис `## spec-v0.1 — <дата>` за форматом із коментаря у файлі (зразок —
     [`examples/leave-portal/4-spec-v0.2/CHANGELOG.md`](../examples/leave-portal/4-spec-v0.2/CHANGELOG.md)).
   - У `docs/conformance.md` — **рядок на кожен новий AC** зі статусом дослівно
     `відкладено (новий AC — чекає задачі)`, колонки «Код» і «Тест» — `—`. Без цих рядків `conformance` червоний
     (`NO-ROW`: AC є в спеці, а рядка в матриці немає), і PR не злити. Рядки потім змінюють розробники у своїх
     PR задач.

   ```bash
   npm run spec:validate
   npm run ac:coverage          # OK
   git add openspec product/CHANGELOG.md docs/conformance.md
   git commit -m "Archive add-<зріз>, log spec-v0.1, add conformance rows"
   git push
   ```

   **З агентом:** «Зроби крок 12.3: archive add-<зріз>, запис spec-v0.1 у CHANGELOG за форматом, рядки в
   docs/conformance.md для кожного нового AC зі статусом «відкладено (новий AC — чекає задачі)», commit і push у цю
   гілку» — підтверджуйте кожну команду.
4. CI зелений → **техлід схвалює PR зараз**, після вашого останнього push (approve на раніших комітах не діє) →
   злийте PR (**Squash and merge** або **Create a merge commit** — обидва підходять) → тег на `main`:

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
   `main` через PR і випустіть тег-виправлення `spec-vX.Y.Z` (наприклад, `spec-v0.1.1`) за тим самим порядком релізу.
5. Напишіть у чат команди: «`spec-v0.1` готовий, питання — issues `spec-question` у репозиторії команди».

**Якщо реліз застряг.** Ви хворієте чи не встигаєте — допоможуть техлід і викладач (обидва collaborators). Зробити
реліз за вас техлід сам не може: свій push він не схвалить, а інший власник коду — лише ви. Тому: викладач пушить
archive, CHANGELOG і рядки матриці у вашу гілку, техлід схвалює, викладач зливає й ставить тег. Якщо наприкінці
дня 3 `spec-v0.1` немає, викладач робить це з вашим відкритим PR сам; бали за терміни B ви втрачаєте
([README](../README.md#що-знімає-бали-pm)).

**Доказ:** тег `spec-v0.1` на `https://github.com/<pm>/<team-slug>/tags`, PR зміни з розділом `spec-review` і
approve техліда, `npm run ac:coverage` зелений на `main`.

## 13. Задачі разом із техлідом (20–30 хв)

Спека каже **що**, а як ділити роботу між 4–6 людьми, вирішує команда. Техлід із розробниками відкриває PR зі
зміною `impl-<зріз>` (`design.md` і `tasks.md`, кожна задача з `Done when: AC-…`). Ви — власник коду `/openspec/`,
тому цей PR схвалюєте ви: перевірте, що кожен AC зрізу покрито задачею і що в `design.md` немає нових правил
поведінки (вони — лише через спеку).

Після merge — задачі-issues за формою **«Задача»** (мітка `task`): Issues → New issue → «Задача» → AC ID, що
зробити, Done when, область (UI / дія / міграція / тест). Заводить їх техлід або ви, з посиланням на рядок
`tasks.md`; кожен розробник призначає собі свої (Assignees). Задач має вистачити на всіх: щонайменше дві на
розробника. На команду з 5–6 розробників це 10–12 задач — більше, ніж AC у зрізі, тому задача — це один AC **або
одна область одного AC** (міграція й RLS, дія, UI). PR, що завершує AC, ставить у його рядку матриці `реалізовано`;
попередні частини лишають `відкладено (причина)` з посиланням на задачу, що лишилась, але вже несуть тест із AC ID
для своєї частини (наприклад, тест бази для міграції). Тут ви не оцінюєтесь окремо, але без задач команда не почне.

## 14. Цикл уточнень і `spec-v0.2` (1,5–2 год за дні 2–4) — трек C

Розробники відкривають issues за формою «Питання до специфікації» у **цьому ж** репозиторії: варіанти A–E,
рекомендований, вплив.

1. **Відповідайте протягом доби** коментарем в issue: який варіант і чому, одним-двома реченнями. Розробник без
   відповіді робить робоче припущення, і два розробники зі своїми припущеннями пишуть несумісний код у спільний
   `main`. **Дублікати:** дубль PM закриває коментарем «Дубль #N — відповідь там» і Close as duplicate — це теж
   відповідь; у `decisions.md` — один рядок на питання з усіма номерами (#3, #7, #9).
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

   - `openspec/changes/clarify-<зріз>/proposal.md` — усі розділи, як у кроці 10: пропущений розділ —
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
4. **Реліз `spec-v0.2`** — так само, як `spec-v0.1` у кроці 12, **до кінця дня 4**. `spec-review` для нової
   зміни, потім коміт і PR:

   ```bash
   git add openspec product
   git commit -m "Propose clarify-<зріз>"
   git push -u origin spec/clarify-<зріз>
   gh pr create --web
   ```

   Опис PR — тип «Зміна специфікації», розділ `## spec-review`. **У тій самій гілці, до approve:** archive, запис
   `## spec-v0.2` у `product/CHANGELOG.md` (MODIFIED, ADDED, на які питання відповідає) і **рядки
   `docs/conformance.md` для кожного нового AC** зі статусом `відкладено (новий AC — чекає задачі)`:

   ```bash
   npx openspec archive clarify-<зріз> -y
   npm run spec:validate
   npm run ac:coverage          # без рядків для нових AC — NO-ROW
   git add openspec product/CHANGELOG.md docs/conformance.md
   git commit -m "Archive clarify-<зріз>, log spec-v0.2, add conformance rows"
   git push
   ```

   Рядки змінених (MODIFIED) AC не чіпайте: `ac:coverage` змінений текст сценарію не бачить. Натомість
   перелічіть змінені AC в описі PR — техлід заведе на них задачі (пункт 5 нижче).

   **З агентом:** «Зроби крок 14.4: archive clarify-<зріз>, запис spec-v0.2 у CHANGELOG за форматом, рядки нових AC
   у docs/conformance.md, commit і push у цю гілку» — підтверджуйте кожну команду.

   CI зелений → approve техліда після останнього push → merge → тег на `main`, а не на гілці:

   ```bash
   git switch main
   git pull
   git tag spec-v0.2
   git push origin spec-v0.2
   ```

   Або через Releases, як у кроці 12; перевірка тегу — та сама (`…/tree/spec-v0.2/openspec/specs`).
5. Закрийте issues з посиланням на зміну. Повідомте команду: «`spec-v0.2`: нові AC — …, змінені — …». Разом із
   техлідом заведіть задачі-issues на нові AC і на змінені, де код уже не відповідає спеці
   (`git diff spec-v0.1 spec-v0.2 -- openspec/specs`).

Питання, що надійшли після `spec-v0.2`, — відповідь в issue й запис у `decisions.md` зі статусом «наступна версія».
Ці записи з'являться вже після тегу, тож у треку C не оцінюються: оцінюємо все до `spec-v0.2` включно.

**Доказ:** відповіді в issues, `product/decisions.md`, `product/CHANGELOG.md`, тег `spec-v0.2` зі щонайменше
одним MODIFIED або ADDED, рядки нових AC у `docs/conformance.md`.

## Оновлення шаблону (коли оголосить викладач, 10–15 хв)

Виправлення шаблону викладач оголошує в чаті курсу з SHA коміту. Не `git merge` шаблону: «Use this template»
починає нову історію, і merge дає конфлікт у кожному файлі, що відрізняється. Натомість ви або техлід:

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

У шаблоні PR позначте тип «Оновлення шаблону» → зелений CI → approve колеги (якщо коміт чіпає `.github/` — вас або
техліда) → merge. Розробники отримають виправлення звичайним `git pull` у `main`.

## 15. Нотатка PM (30 хв) — трек D

Це заготовка для фінальної презентації на demo day: де одна й та сама спека прочиталась по-різному всередині однієї
команди. Джерела: issues `spec-question` і коментарі в них, рев'ю PR-ів (де рецензент і автор розуміли AC
по-різному), журнали `docs/spec-reading/<login>.md`, назви тестів і рядки `docs/conformance.md` у PR задач.

Пишіть у розділ «Нотатка PM» свого файлу здачі ([шаблон](../submissions/_template.md)). Щонайменше три
розбіжності, кожна так:

- **AC / вимога:** `AC-…-02` — «…» (цитата з `spec-v0.1`);
- **як прочитали:** щонайменше два різні прочитання з посиланнями (issue `<pm>/<team-slug>#N`, коментар рев'ю,
  PR чи рядок коду) — логіни, не імена;
- **чому:** яке слово чи пропуск у тексті спеки це дозволив (категорія з таксономії);
- **що зробив PM:** відповідь, MODIFIED / ADDED у `spec-v0.2` або «залишив і чому».

Якщо явних розбіжностей менше трьох, спершу доберіть місця, де прочитання різняться в деталях (робочі припущення в
`docs/spec-reading/<login>.md`, зауваження в рев'ю). Якщо й так менше трьох (команда відкрила мало issues чи ще не
має PR), бракуючий пункт — ваш AC, який, на вашу думку, найлегше прочитати по-різному, з двома можливими
прочитаннями й поміткою «прогноз» (без посилань). Пасивність команди балів не знімає
([README](../README.md#що-знімає-бали-pm)).

## 16. Здача (15 хв)

Інструкція — [`submissions/README.md`](../submissions/README.md). Коротко: PR у цей репозиторій із файлом
`submissions/<team-slug>/<github-login>.md`, заголовок `WS6: Ім'я Прізвище`. SHA — коміт `main` репозиторію команди
на момент здачі (той самий для всієї команди, якщо здаєте одночасно):

```bash
git fetch origin
git rev-parse origin/main
```

## Бонус E (+5, ~1 год) — одне з двох

- **Друга OpenSpec-зміна** — наступний зріз з `roadmap.md` (`add-<зріз-2>`), відкритий PR у репозиторії команди зі
  зеленим CI і розділом `spec-review`. Не архівуйте й не тегуйте її до WS7, якщо команда ще не закінчила перший зріз.
- **Порівняння з Spec Kit:** в окремій порожній теці (не в репозиторії команди) `pip install specify-cli==1.1.2`
  (Python ≥ 3.11) → `specify init --here --integration claude` (у Cursor — `--integration cursor-agent`) →
  `/speckit-specify` з текстом однієї вашої вимоги → `/speckit-clarify`; та сама вимога — через `spec-review`.
  Порівняйте, які питання знайшов кожен, які пропустив і який формат зручніший для вашої команди.

Звіт — у розділі «Бонус E» файлу здачі: посилання і до пів сторінки висновків.
