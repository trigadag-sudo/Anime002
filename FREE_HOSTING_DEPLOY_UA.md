# Безкоштовний деплой (MVP): Hikka UA Rebirth

## Рекомендований стек (безкоштовні тарифи)
- **Vercel (Hobby)** — хостинг Next.js (frontend + API routes).
- **Neon** — PostgreSQL.
- **Upstash Redis** — rate limiting та кеш.

## Кроки
1. Створіть БД у Neon і скопіюйте `DATABASE_URL`.
2. У Vercel імпортуйте GitHub-репозиторій.
3. Додайте всі змінні з `.env.example` у Vercel Project Settings → Environment Variables.
4. Запустіть Prisma міграції (через CI/CD або одноразово локально):
   - `pnpm prisma migrate deploy`
   - `pnpm prisma db seed` (за потреби)
5. Налаштуйте домен `*.vercel.app` або власний домен.
6. Перевірте `/sitemap.xml`, `/robots.txt`, авторизацію та API history/stream.

## Обмеження free-tier
- Встановіть rate limiting на `/api/stream/*` і `/api/torrent/*`.
- Увімкніть кешування зображень (через `next/image`).
- Обмежте частоту ETL sync (наприклад, 1–2 рази на день).

## Резервний варіант
- **Cloudflare Pages + Supabase** якщо потрібна інша інфраструктура.
