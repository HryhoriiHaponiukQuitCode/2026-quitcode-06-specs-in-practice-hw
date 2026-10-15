> **Зразок формату, не відповідь.** Копія з демо-репозиторію (портал відсутностей). Службові коментарі `<!-- … -->` переписано під процес курсу (вимоги PM — в issues чи `docs/` репо PM, розробник забирає їх командою `/requirements-sync`), решта нижче цього блоку — без змін.
> Оригінал: <https://github.com/koldovsky/2026-quitcode-06-leave-portal-demo/blob/demo-03-impl/docs/conformance.md>. PM команди leave-portal пише власні вимоги, а розробник — власну специфікацію, своїми словами — див. [README](../README.md).

# Матриця відповідності (AC → код → тест → статус)

<!--
Ваша матриця: один рядок на кожен AC з openspec/specs/** — вашої специфікації, перекладеної з вимог PM (requirements/,
знімок, який приносить команда агента /requirements-sync).
Рядок «Вимоги» під цим коментарем заповнює лише /requirements-sync: адреса репо вимог PM, джерело (issues чи docs) і
дата знімка, через « · ». Руками його не змінюйте.
Рядки AC з'являються в PR «Специфікація» разом зі сценаріями, зі статусом «відкладено (чекає реалізації)»; рядки
прибраних AC зникають у тому самому PR. Рядок свого AC ви змінюєте в PR «Реалізація» — у тому самому PR, що й код і
тест. Перебудувати автоматично: /conformance (Claude Code, Cursor).
Перевірка в CI: npm run ac:coverage — падає, якщо
  NO-ROW     AC є в специфікації, а рядка немає;
  MISSING    статус «реалізовано», а непропущеного тесту, назва якого починається з цього AC ID, у файлах
             *.test.ts / *.test.tsx немає (*.spec.ts і допоміжні файли Vitest не запускає — вони не рахуються);
  ORPHAN     тест посилається на AC, якого немає в специфікації (типово — PM прибрав AC, а ви переклали це REMOVED);
  BAD-STATUS статус не «реалізовано» і не «відкладено (причина)»;
  WRONG-REF  шляху `src/…`/`tests/…`/`supabase/…` з колонок «Код»/«Тест» немає або тест із цим AC в іншому файлі;
і попереджає, не падаючи, про AC з requirements/, яких ще немає в openspec/specs («вимогу PM ще не перекладено в
специфікацію»).

Приклад рядків (усередині коментаря — скрипт їх не читає):
| AC-LEAVE-02 | заявка на дві посади дає два записи main і additional | `src/app/requests/actions.ts` (createRequest) | `tests/requests.test.ts` | реалізовано |
| AC-LEAVE-04 | заблоковані дні не рахуються | — | — | відкладено (чекаю відповідь PM на Q2, <pm>/<team-slug>-spec#5) |
| AC-LEAVE-10 | діапазон лише із заблокованих днів | — | — | відкладено (чекає реалізації) |
-->

Версія специфікації: spec-v0.1

| AC | Що перевіряє | Код | Тест | Статус |
|---|---|---|---|---|
| AC-LEAVE-01 | заявка на одну посаду: одна заявка `pending`, один запис `main` | `src/lib/leave/submission.ts` (validateSubmission), `src/app/requests/actions.ts` (createLeaveRequest), `supabase/migrations/20261013000000_leave_requests.sql` (submit_leave_request) | `tests/leave/submission.test.ts` | реалізовано |
| AC-LEAVE-02 | позначені обидві посади → два записи `main` і `additional` в одній заявці (Q1: галочки) | `src/lib/leave/records.ts` (splitIntoRecords), `src/app/requests/new/page.tsx` | `tests/leave/submission.test.ts` | реалізовано |
| AC-LEAVE-03 | дата кінця раніша за дату початку → `INVALID_RANGE`, до бази не доходимо; check у базі | `src/lib/leave/submission.ts` (validateDates), `supabase/migrations/20261013000000_leave_requests.sql` (leave_requests_range_check) | `tests/leave/submission.test.ts` | реалізовано |
| AC-LEAVE-04 | заблокований 05.11 у заявці 04–06.11 не рахується: 2 дні з 3 (Q2: заявку приймаємо) | `src/lib/leave/blocked-days.ts` (countLeaveDays), `src/app/requests/page.tsx` | `tests/leave/blocked-days.test.ts` | реалізовано |
| AC-LEAVE-05 | запис іде в чергу менеджера підрозділу своєї посади (Q3); у базі — RLS за division_id | `src/lib/leave/routing.ts` (routeRecords), `supabase/migrations/20261013000000_leave_requests.sql` (leave_records_select_own_division_or_admin), `src/app/manage/page.tsx` | `tests/leave/routing.test.ts` | реалізовано |
| AC-LEAVE-06 | схвалення → `approved`; хто і коли ставить тригер set_decision_fields | `src/lib/leave/decisions.ts` (decide), `src/app/manage/actions.ts` (decideRecord), `supabase/migrations/20261013000000_leave_requests.sql` | `tests/leave/decisions.test.ts` | реалізовано |
| AC-LEAVE-07 | повернення без коментаря → `COMMENT_REQUIRED`; з коментарем → `needs_revision`; check у базі | `src/lib/leave/decisions.ts` (decide), `supabase/migrations/20261013000000_leave_requests.sql` (leave_records_comment_check) | `tests/leave/decisions.test.ts` | реалізовано |
| AC-LEAVE-08 | виправлена заявка на доопрацюванні знову `pending` у тій самій черзі | `src/lib/leave/decisions.ts` (resubmit), `src/app/requests/actions.ts` (resubmitLeaveRequest), `supabase/migrations/20261013000000_leave_requests.sql` (resubmit_records) | `tests/leave/decisions.test.ts` | реалізовано |
| AC-LEAVE-09 | доменне правило видимості; у базі те саме — політики RLS (перевірка трьома користувачами — RUNBOOK) | `src/lib/leave/access.ts` (canViewRecord), `supabase/migrations/20261013000000_leave_requests.sql` (політики select) | `tests/leave/access.test.ts` | реалізовано |
