```json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "allowJs": true,
    "jsx": "react",
    "sourceMap": true,
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "baseUrl": "./",
    "paths": {
      "components/*": [ "src/components/*" ],
      "components": [ "src/components/index.tsx" ]
    },
    "esModuleInterop": true
  },
  "exclude": [
    "node_modules",
    "src/assets",
    "unused",
    "modules",
    "public",
    "locales",
    "etc"
  ]
}

```