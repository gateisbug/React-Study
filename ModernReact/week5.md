## Nextjs 시작
```shell
npx create-next-app@latest . --typescript
# What is your project named? my-app
# would you like to use Typescript? No / Yes
Would you like to use ESLint? No / Yes
Would you like to use Tailwind CSS? No / Yes
Would you like to use `src/` directory? No / Yes
Would you like to use App Router? (recommended) No / Yes
Would you like to customize the default import alias (@/*)? No / Yes
```
## Eslint
```shell
yarn add -d prettier
```

```json
// .prettierrc
{  
  "semi": false,  
  "singleQuote": true,  
  "jsxSingleQuote": true,  
  "trailingComma": "all",  
  "useTabs": false,  
  "tabWidth": 2,  
  "printWidth": 80,  
  "arrowParens": "always",  
  "bracketSameLine": false  
}
```

```json
// package.json
{
	"scripts": {
		// ...
		"format": "prettier --check --ignore-path .gitignore .",  
		"format:fix": "prettier --write --ignote-path .gitignore ."  
	}
}
```

```shell
yarn add -D eslint-config-airbnb eslint-config-airbnb-typescript @typescript-eslint/eslint-plugin @typescript-eslint/parser
yarn add -D eslint-plugin-import@^2.25.3 eslint-plugin-jsx-a11y@^6.5.1 eslint-plugin-react@^7.28.0 eslint-plugin-react-hooks@^4.3.0
yarn add -D eslint-config-prettier
```

```json
// .eslintrc.json
{  
  "extends": [  
    "next/core-web-vitals",  
    "airbnb",  
    "airbnb-typescript",  
    "prettier"  
  ],  
  "parserOptions": {  
    "project": "./tsconfig.json"  
  },  
  "rules": {  
    "sort-imports": "off",  
    "import/order": [  
      "error",  
      {  
        "groups": [  
          "builtin",  
          "external",  
          "internal",  
          "parent",  
          "sibling",  
          "index",  
          "object",  
          "type"  
        ],  
        "newlines-between": "always",  
        "alphabetize": {  
          "order": "asc",  
          "caseInsensitive": true  
        }  
      }  
    ],  
    "react/react-in-jsx-scope": 0,  
    "@typescript-eslint/no-unused-vars": 1,  
    "@typescript-eslint/no-floating-promises": 1,  
    "@typescript-eslint/class-literal-property-style": 1,  
    "@typescript-eslint/no-unused-expressions": 1  
  }  
}
```
## styled-components

```js
// next.config.js
module.exports = {
  compiler: {
    styledComponents: true,
  },
}
```

```tsx
// lib/registry.tsx
'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'
 
export default function StyledComponentsRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())
 
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })
 
  if (typeof window !== 'undefined') return <>{children}</>
 
  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  )
}
```

```tsx
// app/layout.tsx
import StyledComponentsRegistry from './lib/registry'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  )
}
```
다만 주의할 점으로는 위 예제처럼 최상위 레이아웃에서 styled-components의 registry를 사용하는 경우, 자식 노드에서는 server components를 사용할 수 없기 때문에 개별 페이지의 레이아웃에 포함시키는 것이 나을 듯 하다.