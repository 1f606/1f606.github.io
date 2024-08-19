
## 版本

```json
{
  "husky": "9.0.11",
  "lint-staged": "15.2.9"
}
```

## 配置

/.husky/pre-commit

```
npx lint-staged
```

/lint-staged.config.js

```javascript
export default {
  '*.{js,jsx,ts,tsx}': (fileNames) => [
    `eslint ${fileNames.join(' ')}`,
    'eslint --fix',
  ],
  '*.vue': (fileNames) => [
    `eslint ${fileNames.join(' ')}`,
    'eslint --fix',
    'vue-tsc --noEmit',
  ],
};
```
