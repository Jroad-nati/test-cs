# Daily — Vue todo app

A responsive, accessible Vue 3 application backed by [DummyJSON Todos](https://dummyjson.com/docs/todos).

## Develop

Requires Node.js 24 and npm.

```sh
npm ci
npm run dev
```

```sh
npm test
npm run build
npm run preview
```

Initial tasks load from `/todos/user/1`. New tasks use `POST /todos/add`; completion changes use `PUT /todos/:id`. DummyJSON simulates mutations and does not persist them. This application saves your list in browser localStorage, uses unique local IDs for additions, and handles completion of newly added tasks locally because they do not exist on the API. Data is specific to this browser, not a user account or cross-device sync. Clear the site's browser storage to reload the sample data.

## Deploy

The included GitHub Actions workflow tests, builds, and deploys `dist` to GitHub Pages on pushes to `main`. In repository Settings → Pages, set Source to **GitHub Actions**. Relative asset paths support deployment under the repository URL. No API keys or build secrets are required.
