## Linting tools in `~/workspace/lms/learn-ops-client`

There is only one linter: **ESLint**. There's no `.eslintrc*`, `.pylintrc`, `setup.cfg` or `pyproject.toml`, and no Python linting.

### ESLint (via `eslint-config-react-app` v7.0.1)

**Config location:** the `"eslintConfig"` key in `package.json`, which contains `{ "extends": "react-app" }`. The rules come from `node_modules/eslint-config-react-app/index.js` and `base.js`

**Command:** there's no `lint` script in `package.json`. ESLint runs automatically during `npm start` and `npm run build` through react-scripts. To run it on its own:

    cd ~/workspace/lms/learn-ops-client && npx eslint src --ext .js,.jsx,.ts,.tsx

**Rule categories and examples:**

| Category | Example rules |
|---|---|
| Possible errors (`error`) | `no-undef`, `no-restricted-glob`, `name` and similar), `react/jsx-no-undef`,`react/no-typos`, `react/require-render-return` |
| Best practices (`warn`) | `eqeqeq` (smart), `array-callback-return`, `default-case`, `no-eval`, `no-implied-eval`, `no-throw-literal`, `no-loop-func`,
`no-fallthrough`, `no-self-compare` |
| Unused or dead code (`warn`) | `no-unused-vars` (args ignored), `no-unused-expressions`, `no-unreachable`, `no-useless-escape`, `no-useless-concat`, `no-useless-constructor` |
| Style (`warn`) | `no-mixed-operators`, `dot-location`, `newg`, `strict: never`, `unicode-bom` |
| Imports | `import/first` (error), `import/no-amd` (error), tax` (error), `import/no-anonymous-default-export` (warn) |
| React and hooks (`warn`) | `react-hooks/exhaustive-deps`, `react/jsx-no-target-blank`, `react/no-direct-mutation-state`, `react/jsx-no-duplicate-props`, `react/jsx-pascal-case` |
| Accessibility (`jsx-a11y`, `warn`) | `alt-text`, `anchor-is, `aria-props`, `aria-role`, `heading-has-content`,
`iframe-has-title`, `no-access-key` |
| TypeScript override (`*.ts`, `*.tsx`) | Turns off the core `no-undef`, `no-redeclare`, `no-unused-vars` and `no-use-before-define`, and uses the `@typescript-eslint/*` versions instead. This repo has no TS files, so it has no effect. |

**Explicitly disabled rules**

There are no overrides in the config. All disables are inline

| Location | Disable | What it hides |
|---|---|---|
| `src/components/cohorts/StudentCardList.js:152` | `/* eslint-disable no-undef */` | `structuredClone` flagged as an undefined global, because the old react-app env doesn't know it. This is a block comment that is never re-enabled, so `no-undef` is off for the rest of the file. |
| `src/components/cohorts/StudentCapstoneList.js:55` | `/* esame as above, and also off for the rest of the file. |
| `src/components/utils/HumanDate.js:21` | `// eslint-disable-next-line` (all rules) | Most likely `no-useless-escape` for the `'\/'` in `date.replace(/-/g, '\/')`. |
| `src/components/dashboard/student/StudentDashboard.js:20` |` (all rules) | Most likely `no-unused-vars` for the`firstName` / `setFirstName` state. |

The two `no-undef` file-wide disables could also hide real uner down those files. Adding `/* global structuredClone */`would be a narrower fix.