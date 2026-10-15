# Покрокова інструкція: трек розробника

> **Оновлення 15.10.** Кожен розробник тепер робить **власну реалізацію у власному репо**, а вимоги PM підтягує в
> нього командою агента `/spec-sync`. Спільного репо команди, техліда, CODEOWNERS і тегів `spec-vX.Y` більше немає.
> Уже почали в репо команди — [розділ нижче](#якщо-ви-вже-почали-в-репо-команди).

Ви будуєте **власну реалізацію** продукту команди у **своєму публічному репо** `<я>/<team-slug>`, створеному з
шаблону проєкту. Supabase, Vercel, CI і правила злиття там теж ваші. Вимоги веде PM у своєму репо
`<pm>/<team-slug>-spec`, і поточні вимоги — це те, що злито в його `main`. У ваше репо вони потрапляють лише
командою агента `/spec-sync`, а питання до вимог ви ставите issues у репо PM. Команда лишається: той самий бриф, чат
команди, зустріч на старті. Наприкінці PM порівнює, як різні розробники з агентами прочитали й реалізували ті самі
вимоги.

- Рубрика й Definition of Done — у [README](../README.md#трек-розробника--100--5-балів).
- Зразок формату артефактів — [`examples/leave-portal/`](../examples/leave-portal/) (не відповідь).
- Як PM пише вимоги (і як їх читати) — [`docs/spec-guide.md`](spec-guide.md).
- У вашому репо (з шаблону): `docs/bootstrap.md` (чекліст розробника, день 0), `AGENTS.md` (правила для агента),
  README шаблону (розділи «Міграції в прод» і «Збої й відновлення»).

Позначення: `<я>` — ваш GitHub-логін, `<pm>` — логін PM, `<team-slug>` — слаг команди (`property-workflows`,
`therapy-scheduling`, `school-purchasing` або `leave-portal`), `<зріз>` — короткий слаг першого зрізу з
`proposal.md` PM, `<ref>` — Project ref вашого проєкту Supabase, `<N>` — номер issue чи PR.
Команди — для Git Bash на Windows і терміналу macOS. Де PowerShell відрізняється, це сказано окремо. У Windows
PowerShell 5.1 немає `&&`: виконуйте команди по черзі. Якщо PowerShell відповідає «running scripts is disabled on
this system», пишіть `npx.cmd`, `npm.cmd`, `openspec.cmd` замість `npx`, `npm`, `openspec`.

**Бюджет треку: 9,5–11,5 години за тиждень** (сума кроків нижче, без бонусу й необов'язкового кроку 13). Найбільше часу
йде на налаштування власного репо, бази й деплою (кроки 1–6, ~2–2,5 год) і задачі (4–5 год).

Дні календарні (з вихідними); «кінець дня N» — 23:59 за Києвом N-го дня після заняття. День 0 — день заняття,
13.10. PM зливає перші вимоги в `main` до кінця дня 2 (15.10), уточнення — до кінця дня 4 (17.10). Здача для
всіх — до кінця дня 7 (20.10).

| Крок | Коли | Бюджет | Трек |
|---|---|---|---|
| 0. Підготовка машини й акаунтів | до старту | 20 хв | — |
| 1. Власне репо з шаблону | дні 0–2 | 10 хв | A |
| 2. Supabase: організація, проєкт, Auth | дні 0–2 | 30 хв | A |
| 3. Environment `production` і секрет `SUPABASE_DB_URL` | дні 0–2 | 15 хв | A |
| 4. Vercel: імпорт, дві змінні, Require Log In, адреси Auth | дні 0–2 | 30 хв | A |
| 5. Перша база, `users:demo`, локальний запуск | дні 0–2 | 20–30 хв | A |
| 6. Ruleset `solo-main` і CodeRabbit | дні 0–2 | 25 хв | A |
| 7. Зустріч команди: перший раунд питань до брифу | дні 0–1 | 15–20 хв | — |
| 8. Перші вимоги: `/spec-sync` | коли PM написав «вимоги оновлено» (до кінця дня 2) | 15–30 хв | A |
| 9. Читання вимог і питання | дні 2–4 (що раніше, то краще) | 1 год | B |
| 10. Власний план `impl-<зріз>` | дні 2–3 | 30–45 хв | C |
| 11. Задачі: гілка → PR → CodeRabbit → merge | дні 3–7 | 4–5 год | C |
| 12. Оновлення вимог після уточнень | після уточнень PM (до кінця дня 4), найпізніше день 6 | 45 хв – 1 год | D |
| 13. Реалізації колег (необов'язково) | дні 5–7 | — | без балів |
| 14. Здача | день 7 | 15 хв | — |

## Як це влаштовано

```text
 PM:  <pm>/<team-slug>-spec   (шаблон вимог, публічне)
      main: product/ + openspec/specs/ — це і є поточні вимоги; product/CHANGELOG.md — записи за датами
      issues «Питання до специфікації» (мітка spec-question) — тут питаєте ви, тут відповідає PM
            ▲ питання: вебформа issue          │ /spec-sync у Claude Code чи Cursor
            │                                  ▼ копія вимог з main PM → гілка sync/<дата> → PR «Оновлення вимог»
 Ви:  <я>/<team-slug>   (шаблон проєкту, публічне; свої Supabase, Vercel, CI, ruleset solo-main)
      копія вимог · impl-<зріз> · код · тести з AC ID · docs/conformance.md (у шапці — звідки й який коміт вимог)
```

Три правила:

1. **Вимоги — у `main` репо PM.** PM зливає зміну в `main`, додає запис із датою в `product/CHANGELOG.md` і пише в
   чат команди «вимоги оновлено». Тегів і номерів версій немає: версія вимог — це коміт `main` PM.
2. **У ваше репо вимоги потрапляють лише через `/spec-sync`.** `product/` і `openspec/specs/` руками не правите. Це
   правило є в `AGENTS.md` вашого репо, а CodeRabbit попереджає, якщо їх чіпає PR іншого типу.
3. **Питання й пропозиції змінити вимоги — issues у репо PM** через його вебформу. Відповіді там само, і їх бачить
   уся команда.

### Ваш `main` і типи PR

Після кроку 6 у ваш `main` нічого не потрапляє без PR, як на клієнтському проєкті агенції:

- **У `main` — лише через PR.** Прямий push відхиляється, force push і видалення `main` заборонено. Обійти правило не
  може ніхто, і ви теж.
- **Approve не потрібен.** Ви працюєте самі, а свій PR GitHub схвалити не дає. Рецензує CodeRabbit, і на кожен його
  коментар ви відповідаєте виправленням або аргументом.
- **Перевірки** `spec`, `app`, `conformance`, `db-check` мають бути зеленими. Злиття — **Squash and merge** (merge
  commit теж дозволено).
- **Три типи PR** (шаблон опису PR у вашому репо):

| Тип | Що змінює | Звідки |
|---|---|---|
| «Оновлення вимог» | лише `product/`, `openspec/specs/`, `docs/conformance.md` (виняток — окремий коміт після вашого «так»: ID прибраних AC у назвах тестів чи слово з нового denylist PM, кроки 8 і 12) | тільки з `/spec-sync` (кроки 8 і 12) |
| «Реалізація» | `src/`, `supabase/migrations/`, тести, `docs/conformance.md`, `docs/decisions/`, `docs/spec-reading.md`, своя зміна `openspec/changes/impl-<зріз>/`, а також `docs/bootstrap.md` зі знімками в `docs/img/` і `supabase/demo-users.json` | ваша робота (кроки 5–6, 9–12) |
| «Оновлення шаблону» | `git cherry-pick` виправлення шаблону | [розділ нижче](#оновлення-шаблону) |

## Якщо ви вже почали в репо команди

До 15.10 команди працювали в одному спільному репо PM. Якщо ви вже клонували його, писали код чи відкривали issues:

1. **Створіть власне репо** з шаблону й налаштуйте його — кроки 0–6 нижче. Доступ до старого репо команди й інвайт
   в організацію Supabase PM більше не потрібні: ваш застосунок працює з вашою базою.
2. **Перенесіть свій код.** Запушити гілку старого репо в нове не вийде: кожне «Use this template» починає нову
   історію, і GitHub не відкриє PR між непов'язаними історіями. Тому переносимо файли. У клоні старого репо
   подивіться, що змінено у вашій гілці:

   ```bash
   git -C ../<старий-клон> diff --name-only origin/main...<ваша-гілка>
   ```

   У новому репо створіть гілку від свіжого `main` (`git switch -c task/<коротко>`) і скопіюйте туди лише свій код:
   `src/`, `tests/`, `supabase/migrations/`, `docs/decisions/`. `product/`, `openspec/` і `docs/team.md` не
   переносьте: вимоги прийдуть через `/spec-sync`, а план `impl-<зріз>` ви напишете свій (крок 10). Міграції
   лишаються з тими самими мітками: у вашу базу їх застосує ваш `db-deploy` після merge.
   PR із тестами з AC ID зливайте лише після кроку 8. До першої синхронізації вимог у вашому репо немає жодного AC,
   і `conformance` падає з `ORPHAN`.
3. **Журнал** `docs/spec-reading/<я>.md` перенесіть у `docs/spec-reading.md` свого репо (PR «Реалізація», крок 9).
4. **Issues.** Питання тепер живуть у репо вимог PM. Якщо ваших issues там немає (PM їх не переніс), створіть їх
   заново через форму репо PM (крок 9) з посиланням на старе issue.
5. **Вимоги** приходять лише через `/spec-sync` (крок 8), коли PM зіллє їх у `main` свого репо вимог. `product/` і
   `openspec/` зі старого репо не копіюйте.

## 0. Підготовка (20 хв)

1. **Акаунти (особисті, не клієнтські):**
   - GitHub;
   - [Supabase](https://supabase.com/dashboard) з **вільним слотом**: на Free активних може бути лише 2 проєкти на всі
     організації, де ви Owner або Administrator. Проєкти з WS5 поставте на паузу або видаліть;
   - [Vercel](https://vercel.com) Hobby (вхід через GitHub);
   - CodeRabbit окремого акаунта не потребує: це GitHub App, яку ви ставите на своє репо в кроці 6.
2. **Node 24:** `node -v` → `v24.…`. Windows — через nvm-windows або fnm, macOS — через fnm, nvm або Homebrew.
3. **Git і GitHub CLI** (на Windows — Git з Git Bash): `gh --version`, потім `gh auth login` (браузерний вхід). На
   Windows без прав адміністратора: `winget install --id GitHub.cli --scope user --silent`, після цього перезапустіть
   термінал.
4. **OpenSpec CLI** глобально, тієї ж версії, що в шаблоні. Його викликають команди `/opsx:*` і скіли агента:

   ```bash
   npm i -g @fission-ai/openspec@1.14.1
   openspec --version          # 1.14.1
   openspec config set telemetry.enabled false
   ```

5. **Claude Code або Cursor.** Обидва бачать команди шаблону: `/spec-sync`, `/opsx:*` (у Cursor — `/opsx-*`),
   `/conformance`, а також скіли `spec-clarify` і `openspec-*`. Docker **не потрібен**: локально працює лише
   `npm run dev` проти вашої хмарної бази, а міграції на одноразовій базі перевіряє CI (`db-check` піднімає її в
   Docker на раннері GitHub).

**Доказ:** нічого не комітимо. Якщо `openspec --version` показує іншу версію, `/opsx:*` поводитимуться інакше, ніж на
занятті.

## 1. Власне репо з шаблону (10 хв) — трек A

Через браузер: <https://github.com/koldovsky/2026-quitcode-06-project-template> → **Use this template** →
**Create a new repository** → Owner — ваш особистий акаунт (не організація), Repository name — `<team-slug>`,
**Public** → Create. Потім клон: `gh repo clone <я>/<team-slug>`, `cd <team-slug>`, `npm ci`.

Або однією командою (з теки, де тримаєте проєкти):

```bash
gh repo create <team-slug> --public --template koldovsky/2026-quitcode-06-project-template --clone
cd <team-slug>
npm ci
```

Попередження `npm warn deprecated …` / `allow-scripts` під час `npm ci` очікувані. `npm audit fix` і оновлення
пакетів до latest не запускайте: шаблон перевірено з TypeScript ^5 і ESLint ^9.

**Публічним — одразу й назавжди.** На GitHub Free ruleset і секрети environment діють лише в публічних
репозиторіях. До того ж ваше репо читатимуть PM (для нотатки про реалізації) і перевірка за SHA.

**Доказ:** URL вашого репо — для файлу здачі. Перший прогін CI (Actions → CI) має стартувати після створення. Якщо
прогону немає — Actions → CI → **Run workflow** або `gh workflow run ci.yml`. У ньому всі п'ять jobs, зокрема
`db-check` (на `main` теж). `db-deploy` зелений, але міграцій не застосовує: у лозі буде повідомлення про порожній
`SUPABASE_DB_URL`, і так має бути до кроку 3. Цей прогін знадобиться ruleset-у в кроці 6.

## 2. Supabase: організація, проєкт, Auth (30 хв) — трек A

База — лише ваша: її створюєте ви, і вона займає ваш безкоштовний слот.

1. **Організація.** Dashboard → New organization → назва `<team-slug>`, план **Free**.
2. **Проєкт** у цій організації: назва `<team-slug>`, регіон — найближчий. **Пароль бази** — довгий (≥ 20
   символів), лише з літер і цифр: тоді його не треба кодувати в рядку підключення (крок 3). Збережіть його в
   менеджері паролів.
3. **Auth.** У налаштуваннях Authentication вимкніть **Allow new users to sign up**, а в провайдері Email —
   **Confirm email** (орієнтовно Authentication → Sign In / Providers; точну назву перевірте в інтерфейсі).
   Публічний ключ лежить у JS-бандлі, тож з увімкненою реєстрацією будь-хто зареєструється у вашому застосунку
   сам. Вбудована пошта Supabase надсилає 2 листи на годину на весь проєкт і лише членам вашого проєкту Supabase,
   тому користувачів створює скрипт (крок 5), а не лист. `supabase/config.toml` шаблону має
   `[auth] enable_signup = false`, але це конфігурація CLI: у хмару вона не переноситься, тож перемикач у Dashboard
   обов'язковий.

Проєкт Free **засинає** після ~7 днів без активності. Відновлюєте його ви: Dashboard → Restore project (точну назву
кнопки перевірте в інтерфейсі). Перевіряйте перед кожним заняттям і перед здачею: ваш прод-URL відкриватимуть PM і
перевірка.

## 3. Environment `production` і секрет `SUPABASE_DB_URL` (15 хв) — трек A

Міграції в хмарну базу застосовує лише CI: job `db-deploy` після кожного merge у `main`. Йому потрібен рядок
підключення з паролем бази. Цей секрет має бачити тільки `main`, а не ваші робочі гілки: так незлита міграція з
гілки ніколи не потрапить у прод.

1. **Environment.** Settings → Environments → `production`. Перший прогін CI з кроку 1 міг уже створити його
   автоматично (job `db-deploy` на нього посилається) — тоді відкрийте його. Якщо Vercel (крок 4) створив
   `Production` з великої літери для своїх деплоїв, це те саме середовище: регістр у назвах GitHub не розрізняє.
   Налаштуйте його, нового не створюйте. Якщо середовища ще немає — **New environment** → `production` → Configure
   environment. Deployment branches and tags → **Selected branches and tags** → Add deployment branch or tag rule →
   `main`.
2. **Рядок підключення.** Supabase → кнопка **Connect** угорі проєкту → **Session pooler** (порт `5432`) → URI
   вигляду `postgresql://postgres.<ref>:[YOUR-PASSWORD]@<хост пулера>:5432/postgres`. Замініть `[YOUR-PASSWORD]`
   паролем бази. Потрібен саме Session pooler: пряме підключення на Free працює лише через IPv6, а раннери GitHub —
   IPv4. Transaction pooler (порт `6543`) для зміни схеми не годиться.
3. **Секрет environment.** У тому ж environment → Environment secrets → **Add environment secret** → Name
   `SUPABASE_DB_URL`, Value — рядок із пункту 2. Або в терміналі (значення команда спитає й ніде не надрукує):

   ```bash
   gh secret set SUPABASE_DB_URL --env production --repo <я>/<team-slug>
   ```

   Не кладіть його в Repository secrets: секрет репозиторію дістане будь-який workflow будь-якої гілки.

**Запасний шлях**, якщо `db-deploy` не підключається через пулер: офіційний варіант Supabase — `supabase link` +
`supabase db push` із трьома секретами в тому самому environment `production`. Це `SUPABASE_ACCESS_TOKEN` (токен з
обмеженими правами: Supabase → Account → Access Tokens, лише цей проєкт, право Project Settings → API Keys → API Key
Secrets — Read), `SUPABASE_DB_PASSWORD` і `SUPABASE_PROJECT_ID` (`<ref>`). Секрет `SUPABASE_DB_URL` тоді видаліть:
`db-deploy` сам перейде на цей шлях, у workflow нічого не змінюємо (README шаблону, розділ «Міграції в прод»).
Основним цей шлях не робимо: токени з обмеженими правами вийшли 06.10.2026, і з `supabase link` у них є відкрита
помилка.

## 4. Vercel: імпорт, дві змінні, Require Log In, адреси Auth (30 хв) — трек A

1. Vercel → **Add New → Project** → Import Git Repository → `<я>/<team-slug>` (дайте застосунку Vercel доступ лише
   до цього репо). Framework — Next.js, Node 24.x береться з `engines` у `package.json`.
2. **Environment Variables** для **Production і Preview** — рівно дві: `NEXT_PUBLIC_SUPABASE_URL` і
   `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (Supabase → Project Settings → API Keys, ключ `sb_publishable_…`).
   `SUPABASE_SECRET_KEY` у Vercel **ніколи** не додаємо: він обходить RLS, а застосунку першого зрізу не потрібен.
   **Не** підключайте Supabase через Marketplace-інтеграцію Vercel: вона створює окрему організацію й займає слот.
3. **Deploy** → прод-URL `https://<проєкт>.vercel.app/login` відкривається.
4. Settings → **Deployment Protection** → **Vercel Authentication**: вимкніть перемикач **Require Log In** → **Save**.
   На новому Hobby-проєкті він увімкнений; список «Standard Protection» поруч не чіпайте. Інакше PM, колеги й
   викладач не відкриють preview з ваших PR: на Hobby немає учасників команди, і без входу у ваш акаунт Vercel
   preview закрите. Чому це прийнятно лише для навчального проєкту і за яких трьох правил —
   [нижче](#ризики-відкритих-preview).
5. **Адреси Auth.** Supabase → Authentication → **URL Configuration**: Site URL — прод-URL Vercel; Redirect URLs —
   `http://localhost:3000/**` і `https://*-<slug-акаунта-vercel>.vercel.app/**`. Slug — остання частина адреси
   будь-якого preview: `<проєкт>-<хеш>-<slug>.vercel.app`.
6. Після будь-якої зміни змінних — Deployments → останній деплой → ⋯ → **Redeploy**: змінні діють лише для нових
   деплоїв.

Обмеження Hobby: лише некомерційне особисте використання — застосунок навчальний, дані вигадані, без брендингу
клієнта. Cron — щонайбільше раз на добу.

### Vercel: preview і прод

- **Preview.** На кожен PR бот Vercel пише коментар із preview-URL. Відкрийте його у вікні інкогніто: сторінка
  `/login` має відкритися без входу у Vercel.
- **Прод.** Після merge у `main` Vercel деплоїть прод-URL, а job `db-deploy` застосовує нові міграції. Якщо прод
  одразу після merge показує помилку бази, дочекайтеся завершення `db-deploy`.
- Vercel не є обов'язковою перевіркою ruleset-у: червоний деплой PR не блокує. Але зріз має працювати на вашому
  прод-URL (крок 11).

### Ризики відкритих preview

Ви вимкнули захист preview, щоб PM, колеги й викладач могли їх відкривати. Відтепер кожна адреса деплою — preview
гілки й URL кожного коміту — відкрита будь-кому, хто має посилання, а Vercel публікує ці посилання в коментарях до PR
у вашому публічному репо.

- **Preview працюють з тією самою базою, що й прод** (на Supabase Free окремої бази для preview немає): незавершений
  код будь-якої вашої гілки читає й пише ті самі дані.
- **Старі деплої лишаються доступними** за своїми адресами. Виправлення в базі (міграція) захищає всі деплої одразу,
  а виправлення лише в коді застосунку старих адрес не торкається. Небезпечний старий деплой видаліть у Vercel:
  Deployments → ⋯ → Delete (точну назву перевірте в інтерфейсі).
- **Сторінку входу кожного деплою може пробувати будь-хто**, тому паролі синтетичних користувачів робіть довгими й
  випадковими.

Тому відкриті preview безпечні лише за трьох правил: **секретного ключа немає у Vercel** (preview не обійде RLS),
**RLS і явні GRANT на кожній таблиці** (саме вони захищають дані, а не вхід у Vercel), **дані лише синтетичні**
(`*@example.test`, жодних справжніх клієнтів). У клієнтських проєктах агенції захист не вимикають: там потрібен
платний план із паролем або SSO.

## 5. Перша база, `users:demo`, локальний запуск (20–30 хв) — трек A

1. **Міграції й сид шаблону.** Actions → CI → **Run workflow** → Branch `main`, позначте `include_seed` → Run. Або:

   ```bash
   gh workflow run ci.yml --repo <я>/<team-slug> --ref main -f include_seed=true
   ```

   Job `db-deploy` має бути зеленим, а в лозі — застосована міграція шаблону
   `20261012000000_profiles_and_roles.sql` (точний текст виводу CLI перевірте в лозі). Сид у хмарі виконується
   **лише один раз**, пізніші довідкові дані додаються новою міграцією. `include_seed` після цього не вмикайте.
   Червоний job — перевірте пароль у рядку підключення і чи проєкт не на паузі. Не допомогло — запасний шлях із
   кроку 3.
2. **Локальні перевірки:**

   ```bash
   npm run lint
   npm run typecheck
   npm test
   ```

3. **`.env.local` і синтетичні користувачі.** `.env.local` — у `.gitignore`. Значення — з Supabase → Project
   Settings → API Keys:

   ```bash
   cp .env.example .env.local     # PowerShell: Copy-Item .env.example .env.local
   # заповніть: NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY (sb_publishable_…),
   # SUPABASE_SECRET_KEY (sb_secret_…), DEMO_USER_PASSWORD (довгий і випадковий)
   npm run users:demo     # admin@example.test, member1@example.test, member2@example.test
   npm run dev            # http://localhost:3000 → вхід як admin@example.test
   ```

   `SUPABASE_SECRET_KEY` обходить RLS. Він живе лише у вашому `.env.local` для `npm run users:demo`: ніколи у
   Vercel, ніколи з префіксом `NEXT_PUBLIC_`, ніколи в коді застосунку. Значень не вставляйте в чат з агентом:
   `.claude/settings.json` шаблону забороняє Claude Code читати `.env*`. У Cursor такої заборони немає, тож тут
   усе тримається на вашій дисципліні.
4. **Вхід на проді.** Прод-URL → вхід як `admin@example.test` тим самим паролем. Так ви перевіряєте змінні Vercel і
   адреси Auth.
5. **Свої тестові користувачі** — лише синтетичні. З потрібною роллю: запис `*@example.test` у
   `supabase/demo-users.json` у тому самому PR «Реалізація», що й міграція ролей зрізу (крок 11.1), і після merge
   знову `npm run users:demo`.
   Швидкий шлях без ролі: Supabase → Authentication → Users → **Add user** з автопідтвердженням (точну назву поля
   перевірте в інтерфейсі).
6. **Denylist команди.** PM надішле особистим повідомленням список справжніх назв. Збережіть його в
   `product/denylist.local.txt`: по одній назві на рядок, `#` — коментар, шлях уже в `.gitignore`. Тоді
   `npm run spec:check` ловить ці назви в `docs/`, `src/`, `supabase/` і `tests/` ще до push. Перевірка: перша
   команда має показати рядок `!! product/denylist.local.txt` (`!!` означає, що git ігнорує файл) і жодного рядка з
   `??`. Друга має показати «слів у denylist: N», де N — не менше кількості назв:

   ```bash
   git status --short --ignored product
   npm run spec:check
   ```

   Бачите `denylist.local.txt.txt`? Провідник Windows приховав розширення: увімкніть Провідник → Вигляд → Показати →
   Розширення імен файлів і перейменуйте файл. За бажанням той самий список (через кому) можна задати секретом
   репозиторію `SPEC_DENYLIST` (Settings → Secrets and variables → Actions → **New repository secret**). Тоді назви
   ловить і job `spec` у CI.
7. **Supabase MCP** з `.mcp.json` (лише читання; цей файл читає Claude Code, Cursor — ні): перед запуском агента
   задайте змінну `export SUPABASE_PROJECT_REF=<ref>` (PowerShell: `$env:SUPABASE_PROJECT_REF = "<ref>"`). `<ref>`
   візьміть з адреси Dashboard (`/project/<ref>`).

**Правила бази (з `AGENTS.md` шаблону).** Схему змінюємо лише міграціями через PR (крок 11.1). Жодних
`supabase db push` з ноутбука, DDL у SQL Editor чи Table Editor, `supabase db reset --linked`, `migration repair`,
`db push --include-all`. База ваша, і як Owner ви технічно можете змінити в Dashboard що завгодно, тож це правило
тримається лише на вас. Зміна поза `main` ламає наступний `db-deploy` («Remote migration versions not found…»), а
рубрика C зараховує лише міграції, що пройшли PR і `db-deploy`.

## 6. Ruleset `solo-main` і CodeRabbit (25 хв) — трек A

Порядок важливий: спершу CI пройшов на `main` (кроки 1 і 5), потім ruleset. Обов'язкові перевірки в ньому
прив'язані до джерела GitHub Actions, а GitHub приймає таку прив'язку, лише коли Actions уже нещодавно звітували
перевірки в цьому репо.

**Налаштування злиття.** Settings → General → Pull Requests: **Allow merge commits** і **Allow squash merging** мають
бути увімкнені (так за замовчуванням), **Allow rebase merging** можна вимкнути. Метод, вимкнений тут, не запрацює й
з ruleset-ом.

**Через `gh api`** (рекомендовано; JSON — [`materials/rulesets/solo-main.json`](../materials/rulesets/solo-main.json)):

```bash
curl -sLo ruleset.json https://raw.githubusercontent.com/koldovsky/2026-quitcode-06-specs-in-practice-hw/main/materials/rulesets/solo-main.json
gh api repos/<я>/<team-slug>/rulesets --method POST --input ruleset.json
rm ruleset.json
```

PowerShell: `curl.exe` замість `curl` (у Windows PowerShell 5.1 `curl` — це інша команда), `Remove-Item ruleset.json`
замість `rm`.

**Або через браузер:** Settings → Rules → Rulesets → New ruleset → **Import a ruleset** → файл `solo-main.json` (спершу
збережіть його за посиланням вище). Або New branch ruleset і вручну:

- Ruleset name — `solo-main`, Enforcement status — **Active**, Bypass list — **порожній**;
- Target branches → Add target → **Include default branch**;
- **Restrict deletions**, **Block force pushes**;
- **Require a pull request before merging**: Required approvals — `0`; Allowed merge methods — **Merge** і
  **Squash**, без Rebase;
- **Require status checks to pass** → Add checks → `spec`, `app`, `conformance`, `db-check` (джерело — GitHub
  Actions); **Require branches to be up to date before merging** — вимкнено → Create.

Точні назви полів перевірте в інтерфейсі. Якщо перевірка не знаходиться в пошуку, відкрийте Actions: прогони з
кроків 1 і 5 мають містити jobs саме з такими назвами. Те саме, якщо `gh api` відповідає помилкою про
`integration_id`: спершу зелений прогін CI на `main`, потім повторіть команду.

Чому саме так. PR обов'язковий навіть для одного розробника: кожна зміна проходить CI і CodeRabbit, а міграція
потрапляє в базу лише після merge. 0 approve — бо свій PR GitHub схвалити не дає. Перевірки без «up to date» — щоб
не оновлювати гілку після кожного merge іншої вашої гілки. Squash дає в `main` по одному коміту на задачу.

**CodeRabbit.** <https://github.com/apps/coderabbitai> → Install → ваш акаунт → Only select repositories →
`<team-slug>`. У публічних репо з < 10 зірок рев'ю не запускається само: після створення PR пишіть окремим
**коментарем** `@coderabbitai review`. Кількість рев'ю на годину обмежена, тож просіть його після групи виправлень, а
не після кожного push. Конфіг `.coderabbit.yaml` уже є в шаблоні. Серед іншого він попереджає, якщо PR не типу
«Оновлення вимог» чіпає `product/` чи `openspec/specs/`.

**Докази налаштування — перший PR під ruleset-ом.** Він же перевіряє ruleset. Прямих комітів у `main` не робимо й
тут:

```bash
git switch -c setup/bootstrap
```

Позначте пункти чекліста `docs/bootstrap.md` і допишіть докази **без жодного значення**: посилання на прогони CI і
`db-deploy`, назви змінних Vercel і де їх задано, прод-URL, знімки екрана в `docs/img/` (вимкнені «Allow new users to
sign up» і «Confirm email», Require Log In, сторінка ruleset-у). Знімки `spec:check` не читає, тож перевірте самі,
що на них немає назв клієнта, імен людей, email не з `example.test` і значень ключів.

```bash
git add docs
git commit -m "Setup evidence in docs/bootstrap.md"
git push -u origin setup/bootstrap
gh pr create --web
```

У шаблоні опису позначте тип «Реалізація», окремим коментарем — `@coderabbitai review`. **Перевірка ruleset-у:** поки
CI не зелений, кнопка merge неактивна. Потім **Squash and merge** → `git switch main` і `git pull`. Посилання на сам
цей PR і його preview допишіть у `docs/bootstrap.md` наступним PR (наприклад, разом із журналом, крок 9).

**Доказ:** злитий PR зі `docs/bootstrap.md`; сторінка ruleset-у або вивід `gh api repos/<я>/<team-slug>/rulesets`.

Наприкінці — посилання на своє репо й прод-URL у чат команди: вони знадобляться PM для нотатки про реалізації.

## 7. Зустріч команди: перший раунд питань до брифу (15–20 хв)

Цю розмову команда проводить сама, у день 0 або 1, ще до перших вимог PM. Зустріч призначає PM; матеріал — бриф
команди в [`materials/briefs/`](../materials/briefs/). Агента не запускаємо: спершу відчуйте таксономію руками.

1. **2 хв — PM:** мета зрізу і три ключові правила з брифу.
2. **4 хв — ви:** кожен називає одну діру — що неясно, категорію з одинадцяти (слайд 21 деки) і 2–3 варіанти поведінки.
3. **3 хв — PM:** відповідає або записує питання як відкрите.
4. **1 хв — підсумок:** одне питання, яке найбільше здивувало PM.

Не змогли прийти — напишіть свою діру в чат команди до кінця дня 1. Відкриті питання PM забирає в `proposal.md`, а
свої питання ви оформите як issues у репо PM, коли там з'являться перші вимоги (крок 9). Питання — про поведінку;
жодних назв клієнтів і реальних даних: репо публічні. Окремих балів за зустріч немає. Якщо зустріч уже відбулась до
15.10, цей крок пройдено.

## 8. Перші вимоги: `/spec-sync` (15–30 хв) — трек A

Коли PM зіллє перші вимоги в `main` свого репо (до кінця дня 2) і напише в чат команди «вимоги оновлено»,
закомітьте або сховайте (`git stash`) незавершені зміни: команда працює лише з чистим робочим деревом
(`git status --short` порожній). На свіжий `main` агент перейде сам.

У Claude Code: `/spec-sync <pm>/<team-slug>-spec` (або повна адреса `https://github.com/<pm>/<team-slug>-spec`). У
Cursor — `/spec-sync` і ту саму адресу в тому самому повідомленні. Адресу вказуєте лише першого разу, далі команда
бере її з шапки `docs/conformance.md`. Якщо немає ні адреси, ні шапки, агент спитає.

**Що робить агент.** Змінює лише `product/`, `openspec/specs/` і `docs/conformance.md`; єдиний виняток — п. 7, і
лише після вашого «так».

1. Знаходить репо PM у шапці `docs/conformance.md` — рядку на кшталт
   `Вимоги: https://github.com/<pm>/<team-slug>-spec · коміт <40 символів SHA> · 2026-10-15`. До першої синхронізації
   там стоїть `Вимоги: — (заповнює /spec-sync)`, і тоді агент бере адресу з вашого повідомлення.
2. Перевіряє, що робоче дерево чисте, і переходить на свіжий `main`. Потім перевіряє, що репо PM існує й публічне
   (`gh repo view`): якщо ні, зупиниться й попросить звірити адресу з чатом команди. Далі бере останній коміт `main`
   репо PM звичайним git, без тимчасових тек:

   ```bash
   git fetch --depth 1 https://github.com/<pm>/<team-slug>-spec main
   git rev-parse FETCH_HEAD                  # коміт вимог
   git log -1 --format=%cs FETCH_HEAD        # його дата
   ```

   Коміт той самий, що в шапці, — агент скаже «Вимоги актуальні» і зупиниться. У `main` PM ще немає вимог
   (`openspec/specs/`) — теж зупиниться: чекайте «вимоги оновлено». Інакше створить гілку `sync/<дата>` (наприклад,
   `sync/2026-10-15`; друга синхронізація за день — `sync/2026-10-15-2`).
3. Замінює вашу копію вимог версією PM:

   ```bash
   git rm -r -q --ignore-unmatch product openspec/specs
   git checkout FETCH_HEAD -- product openspec/specs
   ```

   `git rm` прибирає лише файли під git, тож ваш локальний `product/denylist.local.txt` лишається на місці.
4. Показує звіт: коміт вимог («було» → «стало»); нові, змінені й прибрані AC (рядки `#### Scenario: AC-…`) і
   вимоги; що змінилось у `product/`, нові записи `CHANGELOG.md` і рішення PM.
5. Оновлює `docs/conformance.md`:
   - шапку: адреса · коміт · дата;
   - для кожного нового AC додає рядок зі статусом дослівно `відкладено (нова вимога — чекає реалізації)`;
   - рядки прибраних AC видаляє й перелічує їх разом із вашими тестами й кодом, що досі згадують ці ID;
   - рядки змінених AC лишає й перелічує, бо їх треба перевірити (крок 12).
6. Запускає `npm run spec:validate`, `npm run spec:check` і `npm run ac:coverage` — ті самі перевірки, що jobs `spec`
   і `conformance` у CI.
7. Комітить «Оновлення вимог з <pm>/<team-slug>-spec@<перші 7 символів SHA>». Якщо ваш тест посилається на
   прибраний AC (`ORPHAN`) або у ваших файлах трапилось слово з нового `product/denylist.txt` PM, агент спитає, чи
   прибрати ID з назви тесту (і замінити слово) окремим комітом у цій самій гілці. Пушить гілку й відкриває PR
   (`gh pr create --web`) — **лише після вашого «так»**.

**PR «Оновлення вимог».**

1. У шаблоні опису позначте тип «Оновлення вимог» і вставте текст опису, який дав агент: рядок `Вимоги: …`, нові,
   змінені й прибрані AC, що змінилось у `product/`.
2. Перевірте вкладку Files changed: там лише `product/`, `openspec/specs/` і `docs/conformance.md` (плюс назви
   тестів, якщо ви погодились на п. 7).
3. Окремим коментарем — `@coderabbitai review`.
4. Дочекайтеся зелених `spec`, `app`, `conformance`, `db-check`. `conformance` зелений саме завдяки рядкам нових AC.
5. **Squash and merge** → `git switch main` і `git pull`.

**Ніколи не правте `product/` і `openspec/specs/` руками** — ні в цьому PR, ні в інших. Не згодні з вимогою чи
знайшли в ній помилку — ставте питання в репо PM (крок 9), і виправлення прийде у ваше репо наступним `/spec-sync`.
Ручна правка розводить вашу копію з `main` PM, а наступна синхронізація мовчки її перезапише. Якщо `spec` червоний
через текст PM (denylist, формат), файлів теж не правте: напишіть PM у чат команди й повторіть `/spec-sync` після
його виправлення.

**PM ще не злив вимог?** Чекайте його повідомлення, а тим часом читайте відкритий PR у репо вимог (крок 9) і готуйте
план (крок 10).

**Доказ:** злитий PR «Оновлення вимог» у вашому репо; шапка `docs/conformance.md` з адресою репо PM, комітом і
датою; зелений `conformance` на `main`.

## 9. Читання вимог і питання (1 год) — трек B

**Поки PM пише вимоги (дні 1–2),** читайте його відкритий PR у `<pm>/<team-slug>-spec` (вкладка Files changed) і
коментуйте рядки: це найдешевший момент для відповіді.

**Після першого `/spec-sync`** вимоги лежать і у вашому репо: `openspec/specs/` (вимоги й AC) і `product/` (глосарій,
ролі, рішення, CHANGELOG).

У Claude Code або Cursor: «Уточни вимоги скілом spec-clarify». Скіл бере репо PM із шапки `docs/conformance.md`,
а далі:

1. проходить 11 категорій таксономії і записує таблицю Ясно / Частково / Немає в розділ `## Скан за таксономією`
   вашого журналу `docs/spec-reading.md`;
2. ставить вам **по одному** питанню з варіантами й рекомендацією — ви вирішуєте, чи йде воно до PM;
3. видає готові тексти issues (або коментарів до наявних) і додає рядки в журнал.

**Власне око обов'язкове.** Перечитайте вимоги самі до запуску скіла й запишіть свої питання. У журналі має бути
видно, що знайшли ви, а що скіл (розділ «Що пропустив би без скіла / що скіл знайшов зайве»).

**Спершу перегляньте наявні питання** всієї команди в репо PM:

```bash
gh issue list --repo <pm>/<team-slug>-spec --label spec-question --state all
```

Або відкрийте сторінку Issues репо PM. Ваше питання вже є — допишіть коментар із новим випадком чи варіантом
поведінки, а не дубль (посилання на коментар — `…/issues/N#issuecomment-M`). Зараховуємо щонайменше 3 питання:
власні issues або змістовні коментарі з новим варіантом чи впливом у наявному issue колеги чи у відкритому PR перших
вимог PM (`…/pull/N#discussion_rM`). З них щонайменше 1 — власне issue.

**Issues — лише через вебформу:** `https://github.com/<pm>/<team-slug>-spec/issues/new/choose` → «Питання до
специфікації», вставте поля з тексту скіла. У поле «Коміт або дата вимог» — коміт із шапки `docs/conformance.md`.
Форма сама ставить мітку `spec-question` і тримає однаковий формат для всіх: варіанти поведінки (не реалізації),
рекомендований, вплив. **Не `gh issue create`:** у репо PM у вас немає права запису, тому мітку з командного рядка
GitHub не поставить, і питання загубиться серед інших issues.

Журнал `docs/spec-reading.md` — один файл у вашому репо:

- **таблиця скану** (11 категорій, статус Ясно / Частково / Немає, де саме). Перевірте, що всі 11 рядків на місці, а
  колонка «Де» вказує на конкретні вимоги чи AC, і виправте, з чим не згодні: рубрика B її перевіряє. Зразок —
  [`spec-reading.md` демо](../examples/leave-portal/2-clarify/spec-reading.md);
- колонка «Issue» — посилання `<pm>/<team-slug>-spec#N`;
- «Рішення (що зробили)» — ваше **робоче припущення**, поки PM не відповів;
- коли PM відповість — «Відповідь PM» і що змінили;
- «Власні припущення» — те, що ви вирішили самі без питання.

Зразок issue — [`issue-Q2.md`](../examples/leave-portal/2-clarify/issue-Q2.md). Журнал потрапляє в `main` PR-ом
«Реалізація»: окремим (гілка `docs/spec-reading`) або разом із першим PR задачі. Дописуйте його впродовж тижня.

**Доказ:** щонайменше 3 питання в репо PM (власні issues або коментарі з новим варіантом в issues чи в PR перших
вимог, з них щонайменше 1 — власне issue); `docs/spec-reading.md` у `main` вашого репо.

## 10. Власний план `impl-<зріз>` (30–45 хв) — трек C

Вимоги кажуть **що** зробити, а як — вирішуєте ви. План — ваша власна зміна OpenSpec у вашому репо:

```bash
git switch main
git pull
git switch -c plan/impl-<зріз>
npx openspec new change impl-<зріз>
```

1. У `openspec/changes/impl-<зріз>/.openspec.yaml` допишіть рядок `skip_specs: true`. Саме `skip_specs`: помилку в
   ключі OpenSpec мовчки ігнорує. Ваша зміна не змінює вимог PM.
2. Файли зміни:
   - `proposal.md` — коротко: який зріз і на якому коміті вимог (із шапки `docs/conformance.md`);
   - `design.md` — таблиці, міграції (GRANT + RLS + політики), server actions, сторінки. Кожне рішення — з посиланням
     на AC ID;
   - `tasks.md` — **вертикальні** задачі (таблиця/RLS + доменна функція + server action/сторінка + тест), кожна
     закінчується рядком `Done when: AC-…`.

   Задач — щонайменше дві, бо рубрика C рахує злиті PR задач. Задача — один AC **або одна область одного AC**
   (міграція й RLS, дія, UI). Правила для агента — у `openspec/config.yaml` (`rules: design`, `rules: tasks`),
   шаблони — `npx openspec instructions design --change impl-<зріз>` і
   `npx openspec instructions tasks --change impl-<зріз>`.
3. Рішення, яких вимоги свідомо не дають (модель даних, бібліотеки), записуйте як ADR у `docs/decisions/`.
4. `npx openspec validate impl-<зріз> --strict` і `npm run spec:check` → commit → push → PR (тип «Реалізація») →
   `@coderabbitai review` → зелений CI → **Squash and merge**.

Зразок — [`examples/leave-portal/3-impl/tasks.md`](../examples/leave-portal/3-impl/tasks.md) і
[`design.md`](../examples/leave-portal/3-impl/design.md). Задачі-issues не потрібні: задача — це пункт `tasks.md`,
і на кожну — одна гілка й один PR (крок 11).

**PM ще не злив перших вимог?** План можна почати з його відкритого PR. Тести з AC ID у назві зливайте лише після
кроку 8: до того `conformance` падає з `ORPHAN` (AC немає у вимогах вашого репо), і PR не злити.

## 11. Задачі: гілка → PR → CodeRabbit → merge (4–5 год) — трек C

Одна задача — одна гілка й один PR. Агенту давайте одну задачу за раз, а не всі вимоги. У Claude Code може
допомогти `/opsx:apply impl-<зріз>`.

```bash
git switch main
git pull
git switch -c task/<коротко>
```

### 11.1 Міграції — лише через PR

```bash
npx supabase migration new <назва>
# SQL: enable row level security; revoke all … from anon, authenticated; явні grant;
#      політика на кожну операцію to authenticated з (select auth.uid())
```

Шлях міграції: файл у PR → `db-check` у CI застосовує **всі** міграції й сид до одноразової бази й перевіряє
порядок відносно `main` → merge → `db-deploy` застосовує її до вашої бази. Локально без Docker міграцію не запустити:
її перевіряє `db-check`.

- **`OLDER`: мітка часу міграції ≤ найновішої на `main`.** Так буває, коли ви ведете дві гілки з міграціями, і іншу
  злили першою. Хмарна база такої міграції не прийме, тому `db-check` червоний і друкує готову команду `git mv` зі
  свіжою міткою. Виправлення: `git pull origin main` у своїй гілці, виконайте цю команду (SQL не змінюється),
  `git commit`, `git push`.
- **`db-deploy` FAIL «Found local migration files to be inserted before the last migration on remote database».**
  Два ваші PR з міграціями пройшли перевірки паралельно (перевірки не вимагають свіжої гілки) і злились у
  зворотному порядку. Виправлення — окремий PR: `git mv` старшого, ще не застосованого файлу (див. журнал
  `db-deploy`) на нову мітку без зміни SQL. `db-check` покаже попередження `RENAMED`, а не FAIL → merge → `db-deploy`
  застосує. `--include-all` — ніколи.
- **Застосовані міграції не редагуємо**, лише додаємо новий файл. Деструктивне (`drop`, перейменування колонок,
  `truncate`) — обережно: база одна для проду й усіх ваших preview, а старі деплої й відкриті гілки ще читають стару
  схему.
- **Кілька записів або перевірка + запис разом** (заявка з позиціями, запуск із копією задач, утримання слоту без
  перетину). У supabase-js немає транзакцій, кожен запит — окрема транзакція. Робіть це однією функцією Postgres
  через `supabase.rpc(…)`, як `submit_leave_request` у демо (`supabase/migrations/20261013000000_leave_requests.sql`):
  `security invoker`, або `security definer` лише в схемі `private` з `set search_path = ''`.
- **Довідкові дані.** Сид шаблону застосовано в кроці 5, а в хмарі сид виконується лише раз. Нові довідкові дані
  додавайте новою міграцією з `insert … on conflict do nothing`.
- **Типи бази** (`src/lib/supabase/database.types.ts`): готовий файл для схеми вашого PR — у Summary прогону
  `db-check`. Скопіюйте його в PR. З ноутбука в базу нічого не пушимо.

**Нові ролі** (замість `member`/`admin` шаблону) — окрема нова міграція в такому порядку:

```sql
alter table public.profiles drop constraint profiles_role_check;               -- 1) зняти старе правило
update public.profiles set role = 'employee' where role = 'member';            -- 2) перевести наявні дані
alter table public.profiles alter column role set default 'employee';          -- 3) новий default
alter table public.profiles
  add constraint profiles_role_check check (role in ('employee', 'manager', 'admin'));  -- 4) нове правило
```

Дані й default оновлюємо **до** нового `profiles_role_check`, інакше він упаде на рядках `member`, які вже створив
`npm run users:demo` у кроці 5. А старе правило знімаємо першим, бо воно не пускає нові значення в `update`. Ролі
тут — з демо, у вас — з `product/roles.md` PM. Нові ролі в `supabase/demo-users.json` пропишіть у тому самому PR:
старе `member` новий `profiles_role_check` уже не пропустить. Після merge і зеленого `db-deploy` запустіть
`npm run users:demo` — скрипт оновить ролі наявних користувачів.

### 11.2 Код, тести, матриця

1. **Тести.** Назва кожного тесту поведінки починається з AC ID: `it("AC-…-02: …")`. Тести — лише у файлах
   `*.test.ts` / `*.test.tsx` (у `src/` або `tests/`): `*.spec.ts` Vitest не запускає, і `ac:coverage` такий тест не
   рахує. Пропущені тести (`.skip`, `.todo`) AC не покривають, а тест без жодного `expect` Vitest вважає проваленим.
2. **Матриця.** У `docs/conformance.md` знайдіть рядок свого AC (його додав `/spec-sync` зі статусом
   `відкладено (нова вимога — чекає реалізації)`) і змініть його:
   - колонки «Код» і «Тест» — шляхи від кореня репо в бектиках (`src/…`, `tests/…`, `supabase/…`; номер рядка `:12`
     можна);
   - статус — дослівно `реалізовано` або `відкладено (причина)`.

   `ac:coverage` перевіряє, що ці файли є і що тест із цим AC лежить у вказаному файлі. Задача — лише частина AC
   (наприклад, міграція без UI)? Тест із AC ID для своєї частини (тест бази, доменної функції) додаєте, а статус
   лишаєте `відкладено (UI — задача 2.3)`. `реалізовано` ставить PR, що завершує AC. Рядки може оновити й команда
   `/conformance`. Шапку матриці (рядок «Вимоги: …») не чіпайте: її пише лише `/spec-sync`.
3. **Перевірки перед PR:**

   ```bash
   npm run lint
   npm run typecheck
   npm test
   npm run build          # у виводі має бути ƒ Proxy (Middleware)
   npm run ac:coverage
   npm run spec:check
   ```

### 11.3 PR задачі

```bash
git add -A
git commit -m "AC-…-02: <що зроблено>"
git push -u origin task/<коротко>
gh pr create --web
```

1. Опис PR: тип «Реалізація», пункт `tasks.md` (наприклад, «Задача 2.1») і AC, які покриває PR. Окремим
   коментарем — `@coderabbitai review`.
2. Відкрийте preview з коментаря Vercel і пройдіть свій AC руками під синтетичним користувачем.
3. На коментарі CodeRabbit відповідайте виправленням новим комітом у ту саму гілку або аргументом у треді. Він
   помічник, а не рецензент: остаточне рішення за вами, але аргумент має бути записаний.
4. Зелений CI → **Squash and merge** → `git switch main` і `git pull`. Якщо PR мав міграцію, дочекайтеся зеленого
   `db-deploy`.
5. **Останній PR зрізу, до merge:** `npx openspec archive impl-<зріз> -y` у тій самій гілці. Зміна має
   `skip_specs: true`, тож archive переносить її в `openspec/changes/archive/` і не чіпає `openspec/specs/`.
   Перевірте це: `git status` не має показувати змін у `openspec/specs/`.

**Відповіді PM ще немає?** Або реалізуйте своє робоче припущення з `docs/spec-reading.md` і назвіть його в PR, або
залиште AC зі статусом `відкладено (чекаю відповідь PM на Q3, <pm>/<team-slug>-spec#N)`.

**Знайшли помилку у вимогах?** Не правте `openspec/specs/` чи `product/`. Питання чи пропозиція змінити вимогу —
issue через форму репо PM (крок 9). Рішення й merge — за PM, а у ваше репо зміна прийде через `/spec-sync`.

**Доказ:** щонайменше 2 злиті PR задач у вашому репо зі зеленим CI, тестами з AC ID і рядками матриці. Міграції —
лише в PR, після merge `db-deploy` зелений. В останньому своєму PR задачі — коментар із записом екрана 1–2 хв: ваш
зріз на **вашому** прод-URL під синтетичним користувачем, без паролів у кадрі. Якщо PM злив перші вимоги пізніше за
кінець дня 3, для C достатньо одного злитого PR задачі (напишіть це у файлі здачі).

## 12. Оновлення вимог після уточнень (45 хв – 1 год) — трек D

До кінця дня 4 PM зливає в `main` уточнення — відповіді на питання, `product/decisions.md`, зміну з MODIFIED/ADDED —
і пише в чат команди «вимоги оновлено».

1. **Синхронізація.** Чисте робоче дерево → `/spec-sync` без адреси: репо PM уже записано в шапці. Агент сам перейде
   на свіжий `main`, створить гілку `sync/<дата>` і покаже звіт.
2. **Прочитайте звіт і зміни.** Запис PM у `product/CHANGELOG.md` і `product/decisions.md` пояснює, що змінилось і
   чому. Повний текст змін показує команда `git diff main -- openspec/specs product`, запущена в гілці
   синхронізації.
   - **Нові AC.** Агент уже додав їх рядки зі статусом `відкладено (нова вимога — чекає реалізації)`.
   - **Змінені AC** (той самий ID, інший текст). `ac:coverage` змін тексту не бачить: рядок лишається як був, і зі
     статусом `реалізовано` CI буде зелений, навіть коли код уже не відповідає вимозі. Звірте код і тест із новим
     текстом самі.
   - **Прибрані AC.** Агент видалив їх рядки й показав ваші тести та код, що досі згадують ці ID. Тест з ID
     прибраного AC у назві дає `ORPHAN`, і PR не пройде `conformance`. Тому агент запропонує прибрати ID з назви
     такого тесту окремим комітом у цій самій гілці; тіло тесту лишиться. Погодьтеся. Чи потрібен вам ще код
     прибраного AC, вирішіть окремим PR «Реалізація».
3. **PR «Оновлення вимог»** — як у кроці 8. В описі — текст від агента; кожен змінений AC перелічіть із результатом
   звірки: «код відповідає» або «виправлю в PR #N». Зелений CI → **Squash and merge**.
4. **Реалізуйте або відкладіть** щонайменше один новий чи змінений AC:
   - **реалізуйте** PR-ом «Реалізація» (крок 11): тест з AC ID, рядок `реалізовано`; для зміненого AC — виправлені
     код і тест;
   - або **відкладіть** із посиланням на рішення PM (issue в репо PM чи запис у `product/decisions.md`), наприклад
     `відкладено (рішення PM: <pm>/<team-slug>-spec#N — реалізація в наступному зрізі)`, і поясніть у PR, чому.
5. **Журнал.** У `docs/spec-reading.md` заповніть «Відповідь PM» і «Рішення» для своїх питань.

**PM не злив уточнень до кінця дня 4?** Вас за це не карають. На день 6 синхронізуйте те, що є в `main` PM, і
зробіть цей крок із тим, що прийшло: PR `/spec-sync` того дня зараховуємо за D навіть без нових AC. Якщо агент
відповів «Вимоги актуальні», нового коміту в `main` PM немає, і PR не буде: напишіть PM у чат команди, а у файлі
здачі в полі «PR «Оновлення вимог» після уточнень PM» вкажіть рядок `/spec-sync <YYYY-MM-DD>: Вимоги актуальні`.
Його зараховуємо, коли коміт у рядку «Вимоги:» вашої матриці дорівнює останньому коміту `main` PM на цю дату.

**Доказ:** злитий PR «Оновлення вимог» після уточнень PM (у шапці `docs/conformance.md` — новий коміт вимог); рядки
нових AC; щонайменше один новий чи змінений AC реалізовано (PR задачі) або відкладено з посиланням на рішення PM.

## Оновлення шаблону

Цей розділ знадобиться, лише якщо викладач оголосить виправлення шаблону (10–15 хв, поза бюджетом). Виправлення
шаблону викладач оголошує в чаті курсу з SHA коміту. Не робіть `git merge` шаблону: «Use this template» починає нову
історію, і merge дає конфлікт у кожному файлі, що відрізняється. Натомість у своєму репо:

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

У шаблоні PR позначте тип «Оновлення шаблону» → зелений CI → **Squash and merge**.

## 13. Реалізації колег (необов'язково, без балів)

Одна вимога в кількох реалізаціях найкраще показує, де текст допускає різні прочитання. Посилання на репо колег —
у чаті команди.

- Відкрийте `docs/conformance.md` колеги: які AC реалізовано, які відкладено й чому.
- Знайдіть тест того самого AC (пошук за AC ID у репо колеги) і порівняйте, що саме він перевіряє.
- Пройдіть той самий сценарій на прод-URL колеги.

Знайшли розбіжність у прочитанні — це питання до PM (issue в репо PM, крок 9) і матеріал для його нотатки
«порівняння реалізацій». Чужий код у свій PR без посилання на джерело не переносьте
([README](../README.md#своя-робота-у-своєму-репо)).

## 14. Здача (15 хв)

Інструкція — [`submissions/README.md`](../submissions/README.md). SHA — коміт `main` **вашого** репо на момент
здачі:

```bash
git fetch origin
git rev-parse origin/main
```

Коміт вимог, з якими ви здаєте роботу, — у шапці `docs/conformance.md` на цьому SHA. Питання — посиланнями на issues
чи коментарі в репо PM.

У файлі здачі:

- роль — `dev`; «Ваше репо» — `https://github.com/<я>/<team-slug>`;
- «Репо вимог PM» — `https://github.com/<pm>/<team-slug>-spec`; «Прод-URL» — ваш прод на Vercel;
- у розділі «Розробник» — дослівно рядок «Вимоги:» з матриці на SHA, PR-и з кроків 8, 11 і 12, журнал і питання
  з кроку 9, зміна `impl-<зріз>` з кроку 10 і ваші AC зі статусами, як у матриці на SHA;
- розділ «PM» — одним рядком «—».

## Бонус E (+5, 1–2 год) — одне з двох

- **Бутстрап без шаблону.** В окремому порожньому репо попросіть агента зібрати той самий стек з нуля
  (Next.js 16.4 + Supabase + вхід email/пароль + CI). Порівняйте результат зі [списком пасток, які обходить
  шаблон](https://github.com/koldovsky/2026-quitcode-06-project-template#як-цей-шаблон-зібрано-з-нуля):
  де лежить `proxy.ts`, Cache Components, `setAll` із заголовками, `*_ANON_KEY` замість publishable, GRANT у
  міграції, `.env.example` у `.gitignore`, `next typegen` перед `tsc`. Результат — таблиця «пастка → що зробив
  агент → як помітили».
- **`/opsx:verify` + власний розбір.** Перед archive `impl-<зріз>` запустіть `/opsx:verify impl-<зріз>`. Для
  кожної знахідки (CRITICAL / WARNING / SUGGESTION) вирішіть: справжня проблема, хибна тривога чи смак — і чому.
  Підсумуйте, що verify знайшов такого, чого не ловлять `npm test` і `ac:coverage`, і де він помилився.

Звіт — у розділі «Бонус E» файлу здачі: посилання і до пів сторінки висновків.
