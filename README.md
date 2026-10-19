# [Admin Dashboard Template](https://github.com/salimi-my/admin-dashboard-template) &middot; [![Author Salimi](https://img.shields.io/badge/Author-Salimi-%3C%3E)](https://www.salimi.my)

A [Laravel](https://laravel.com) + [Inertia](https://inertiajs.com) + React starter kit for admin dashboards. It ships a floating sidebar and header layout, a users datatable with full CRUD, account settings, a dashboard with charts, and polished authentication pages.

## Features

- **App shell**: floating sidebar with groups and collapsible sub menus, floating header, breadcrumbs, footer, dark mode.
- **Users management**: server-driven datatable (search, filters, sorting, pagination, bulk delete), create / edit / delete dialogs, and password reset links. State lives in the query string, so pages are shareable and refresh-safe.
- **Roles**: `User` and `Admin` through the `App\Enums\UserRole` backed enum, enforced by `UserPolicy`.
- **Settings**: profile (with avatar upload), password, two-factor authentication and appearance.
- **Dashboard**: stat cards, signups chart, role distribution chart and recent users.
- **Authentication**: login, register, forgot / reset password, email verification and two-factor challenge, powered by [Laravel Fortify](https://laravel.com/docs/fortify).
- **UI library**: the full [shadcn/ui](https://ui.shadcn.com) set (radix-vega style) in `resources/js/components/ui`.
- **Typed routes**: [Wayfinder](https://github.com/laravel/wayfinder) generates TypeScript functions for every route, so URLs are never hardcoded.
- **Tooling**: Pest, Pint, Larastan (PHPStan), Laravel Boost, and a GitHub Actions workflow.

## Stack

| Layer           | Technology                       |
| --------------- | -------------------------------- |
| Backend         | Laravel 13, PHP 8.3+, Fortify    |
| Frontend        | Inertia v3, React 19, TypeScript |
| Styling         | Tailwind CSS v4, shadcn/ui       |
| Build           | Vite with Vite+ (`vp`)           |
| Database        | SQLite by default                |
| Package manager | **pnpm**                         |

## Requirements

- PHP 8.3 or newer (the project targets 8.5)
- [Composer](https://getcomposer.org)
- Node.js 22 or newer
- [pnpm](https://pnpm.io/installation)

This kit uses pnpm only. Do not use `npm` or `yarn`.

## Installation

### With the Laravel installer

The installer defaults to npm and deletes other lockfiles, so pass `--pnpm`:

```bash
laravel new my-app --using=salimi-my/admin-dashboard-template --pnpm
```

### With Composer

```bash
composer create-project salimi-my/admin-dashboard-template my-app
cd my-app
pnpm install
pnpm run build
```

### From a clone

```bash
git clone https://github.com/salimi-my/admin-dashboard-template.git my-app
cd my-app
composer setup
```

`composer setup` installs dependencies, creates `.env`, generates the app key, runs migrations, links storage, then installs and builds the frontend.

## Development

```bash
composer run dev
```

This starts the Laravel server, queue worker, log viewer and the Vite dev server together. Wayfinder regenerates the route functions automatically while Vite runs.

To generate them manually:

```bash
php artisan wayfinder:generate --with-form
```

### Demo data

```bash
php artisan db:seed
```

| Email                 | Role  | Password   |
| --------------------- | ----- | ---------- |
| `johndoe@example.com` | Admin | `P@$$w0rd` |
| `janedoe@example.com` | User  | `P@$$w0rd` |

It also creates 48 random users (password `password`). Change or remove the seeded accounts in `database/seeders/DatabaseSeeder.php` before deploying.

## Customizing

### Primary color

Change `--brand`, `--brand-foreground` and `--brand-hue` in `resources/css/app.css` (in both `:root` and `.dark`). Every other brand color is derived from them.

### Sidebar menu

Edit `navGroups` in `resources/js/lib/nav.ts`. Always build `href` from Wayfinder route functions.

```ts
{
    title: 'Content',
    items: [
        // A plain link
        { title: 'Dashboard', href: dashboard(), icon: LayoutGrid },
        // A collapsible item with sub menus
        {
            title: 'Posts',
            icon: FileText,
            items: [
                { title: 'All Posts', href: postsIndex() },
                { title: 'New Post', href: postsCreate() },
            ],
        },
        // Hidden from regular users
        { title: 'Users', href: usersIndex(), icon: Users, adminOnly: true },
    ],
}
```

The `Content` section (Posts, Media, Comments) ships as dummy pages that all render `resources/js/pages/content/placeholder.tsx`. The routes live at the bottom of the authenticated group in `routes/web.php`. Replace them with real controllers and pages, or delete them along with the menu entries.

### Roles

1. Add a case to `App\Enums\UserRole`. Keep the least privileged role first, because it is the default for new users.
2. Add its `label()`.
3. Add it to the `UserRole` type in `resources/js/types/auth.ts`.

Validation, filters and role selects read `UserRole::options()`, so they update automatically. Authorization lives in `App\Policies\UserPolicy` and the `adminOnly` nav flag. Do not check roles inline in controllers.

### Adding a datatable page

1. Build the controller like `UserController::index`. A Form Request validates `search`, filters, `sort`, `direction` and `per_page`, and the controller returns an API Resource collection plus the applied `filters`.
2. Use the shared pieces in `resources/js/components/table/*` with the `useTableParams` hook.
3. Add new filter keys to `Params` in `resources/js/types/table.ts` and to `paramsFromServer` / `paramsToQuery` in `resources/js/lib/table.ts`.
4. Start from `components/users/users-table.tsx` and add a sidebar item in `lib/nav.ts`.

## Project structure

```text
app/
  Enums/UserRole.php            Role enum
  Http/Controllers/             Dashboard, Users, Settings controllers
  Http/Requests/                Form Requests (users, settings)
  Http/Resources/UserResource.php
  Policies/UserPolicy.php
resources/js/
  components/ui/                shadcn/ui library
  components/table/             Reusable datatable pieces
  components/users/             Users table, forms and dialogs
  components/dashboard/         Dashboard cards and charts
  hooks/use-table-params.ts     Query-string driven table state
  layouts/                      App shell, settings and auth layouts
  lib/nav.ts                    Sidebar menu config
  pages/                        Inertia pages
routes/                         web.php and settings.php
```

## Scripts

| Command                | Purpose                                |
| ---------------------- | -------------------------------------- |
| `composer run dev`     | Run the full development environment   |
| `composer test`        | Pint check, PHPStan and the Pest suite |
| `composer lint`        | Fix PHP code style with Pint           |
| `composer types:check` | PHPStan analysis                       |
| `pnpm run build`       | Production frontend build              |
| `pnpm run check:fix`   | Format and lint the frontend           |
| `pnpm run types:check` | TypeScript type check                  |

## Using AI assistants

`AGENTS.md` and `.cursor/rules` describe this kit's conventions (pnpm only, Wayfinder for URLs, Inertia `<Form>` for forms, flash toasts). They are read by Cursor, Laravel Boost and other agents, so generated code follows the same patterns.

## Credits

Built on the official Laravel React starter kit, with UI components from [shadcn/ui](https://ui.shadcn.com).

## License

Released under the [MIT license](https://opensource.org/licenses/MIT).
